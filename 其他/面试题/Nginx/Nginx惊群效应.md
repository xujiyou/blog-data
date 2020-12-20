# Nginx 惊群效应

## 惊群效应

惊群简单来说就是多个进程或者线程在等待同一个事件，当事件发生时，所有线程和进程都会被内核唤醒。唤醒后通常只有一个进程获得了该事件并进行处理，其他进程发现获取事件失败后又继续进入了等待状态，在一定程度上降低了系统性能。

具体来说惊群通常发生在服务器的监听等待调用上，服务器创建监听socket，后fork多个进程，在每个进程中调用accept或者epoll_wait等待终端的连接。

多进程模式下服务端的socket编程一般有如下4种方式：

1，主进程首先调用listen创建监听TCP套接字，进行监听，接着调用阻塞accept等待连接请求，当有TCP连接请求到达时，阻塞accept会获得该连接请求并创建TCP连接，然后调用fork创建出多进程对该TCP连接进行handle。

这种方式需要不断地fork进程，会造成系统极大的性能损耗。

2，主进程首先调用listen创建监听TCP套接字，进行监听，接着调用fork创建多进程，子进程内部调用阻塞accept等待连接请求，当有TCP连接请求到达时，这些子进程全部被唤醒并抢占连接请求，抢占到的子进程会获得该连接请求并创建TCP连接，然后进行handle。抢占不到连接请求的进程会收到EAGAIN错误并在下次调用阻塞accept时再次挂起。

多个进程因为一个连接请求而被同时唤醒，称为**惊群效应**，在高并发情况下，大部分进程会无效地被唤醒然后因为抢占不到连接请求又重新进入睡眠，是会造成系统极大的性能损耗。

3，主进程首先调用listen创建监听TCP套接字，进行监听，接着调用epoll_create创建一个epfd和对应的红黑树，然后调用fork创建多进程，子进程内部调用epoll_wait等待同一个epfd的事件，当有TCP连接请求到达时，也就是这个epfd有事件发生时，这些子进程全部被唤醒并处理事件，只有一个子进程通过调用非阻塞accept抢占到该连接请求并创建TCP连接，然后进行handle。

这种方式有如下2个缺点：

- 同样会导致惊群效应
- 由于等待的是同一个epfd，假如A进程创建了TCP连接A，B进程创建了TCP连接B，当连接A发生读写事件时，可能B进程抢占了该事件，导致混乱。

4，主进程首先调用listen进行监听，接着调用fork创建多进程，子进程内部调用epoll_create创建各自的epfd和各自的红黑树，并调用epoll_wait等待同各自epfd的事件，这些子进程都将listen端口的监听事件放在自己的epfd中，当有TCP连接请求到达时，每个子进程的epfd的这个事件都会发生，这些子进程全部被唤醒并处理各自的这个事件，只有一个子进程通过调用非阻塞accept抢占到该连接请求并创建TCP连接，然后进行handle。

这种方式对比于第3种方式，不会出现进程与连接的不对应（创建TCP连接后的子进程会将该连接放在自己的epfd中而不是公共的epfd中），但在监听事件上仍然会出现惊群效应。

---

方式2的惊群效应称为accept惊群效应，方式3跟方式4则称为epoll惊群效应。

方式1需要不断地fork进程，基本上已经被抛弃使用了。

针对方式2的accept惊群效应，Linux 2.6版本给出了解决方案。

在Linux 2.6版本中，维护了一个等待队列，队列中的元素就是进程，非exclusive属性的元素会加在等待队列的前面，而exclusive属性的元素会加在等待队列的末尾，当子进程调用阻塞accept时，该进程会被打上WQ_FLAG_EXCLUSIVE标志位，从而成为exclusive属性的元素被加到等待队列中。当有TCP连接请求到达时，该等待队列会被遍历，非exclusive属性的进程会被不断地唤醒，直到出现第一个exclusive属性的进程，该进程会被唤醒，同时遍历结束。

只唤醒一个exclusive属性的进程，这也是exclusive的含义所在：互斥。

因为阻塞在accept上的进程都是互斥的（都是打上WQ_FLAG_EXCLUSIVE标志位），所以TCP连接请求到达时只会有一个进程被唤醒，从而解决了惊群效应。

虽然可以采用类似方式2的解决方案处理方式3的epoll惊群效应，但是该解决方案无法解决方式3中进程与连接不对应的问题。因为accept确实只需要任意一个进程就能够处理，通过互斥的方式就可以解决，而epoll处理的事件有连接请求事件，读写事件等，前者是任意一个进程就可以处理，但后者是需要在持有对应TCP连接的进程中才能处理。所以，一般也不会使用方式3。

目前多进程模式下socket编程使用比较广泛的是方式4，Nginx就是使用这种方式。

针对方式4的epoll惊群效应问题，有如下3种解决方案：锁策略、SO_REUSEPORT、EPOLLEXCLUSIVE标志位



## 锁策略

基本思路是，子进程在进行epoll_ctl加入监听事件以及epoll_wait前，需要获得进程间的全局锁，获得锁的子进程才有资格通过监听获取连接请求并创建TCP连接，锁策略确保了只有1个子进程在处理TCP连接请求。

锁策略的伪代码如下：

```text
semop(...); // lock
epoll_wait(...);
accept(...);
semop(...); // unlock
... // manage the request
```

Nginx在1.11.3版本（2016-07-26）之前通过配置accept_mutex默认为on支持该解决方案。



## SO_REUSEPORT

在Linux 3.9版本引入了socket套接字选项SO_REUSEPORT，Linux 3.9版本之前，一个进程通过bind一个三元组（{<protocol>, <src_addr>, <src_port>}）组合之后，其他进程不能再bind同样的三元组，Linux 3.9版本之后，凡是传入选项SO_REUSEPORT且为同一个用户下（安全考虑）的socket套接字都可以bind和监听同样的三元组。内核对这些监听相同三元组的socket套接字实行负载均衡，将TCP连接请求均匀地分配给这些socket套接字。

这里的负载均衡基本原理为：当有TCP连接请求到来时，用数据包的（{<src_addr>, <src_port>}）作为一个hash函数的输入，将hash后的结果对SO_REUSEPORT套接字的数量取模，得到一个索引，该索引指示的数组位置对应的套接字便是要处理连接请求的套接字。

Nginx在1.9.1版本（2015-05-26）时支持了reuseport特征，在Nginx配置中的listen指令的端口号之后增加reuseport参数后，Nginx的各个worker进程就有自己各自的监听套接字，这些监听套接字监听相同的源地址和端口号组合。

使用方式如下：

```text
http {
     server {
          listen 80 reuseport;
          server_name  localhost;
          # ...
     }
}

stream {
     server {
          listen 12345 reuseport;
          # ...
     }
}
```

由于reuseport特征负载均衡在内核中的实现原理是按照套接字数量的hash，所以当Nginx进行reload，从reuseport升级为非reuseport，或者从多worker进程升级为少worker进程，都会有大幅度的性能下降。

### EPOLLEXCLUSIVE标志位

在Linux 4.5版本引入EPOLLEXCLUSIVE标志位（Linux 4.5, glibc 2.24），子进程通过调用epoll_ctl将监听套接字与监听事件加入epfd时，会同时将EPOLLEXCLUSIVE标志位显式传入，这使得子进程带上了exclusive属性，也就是互斥属性，跟Linux 2.6版本解决accept惊群效应的解决方案类似，不同的地方在于，当有监听事件发生时，唤醒的可能不止一个进程（见如下对EPOLLEXCLUSIVE标志位的官方文档说明中的“one or more”），这一定程度上缓解了惊群效应。

```text
# http://man7.org/linux/man-pages/man2/epoll_ctl.2.html
EPOLLEXCLUSIVE (since Linux 4.5)
              Sets an exclusive wakeup mode for the epoll file descriptor
              that is being attached to the target file descriptor, fd.
              When a wakeup event occurs and multiple epoll file descriptors
              are attached to the same target file using EPOLLEXCLUSIVE, one
              or more of the epoll file descriptors will receive an event
              with epoll_wait(2).  The default in this scenario (when
              EPOLLEXCLUSIVE is not set) is for all epoll file descriptors
              to receive an event.  EPOLLEXCLUSIVE is thus useful for avoid‐
              ing thundering herd problems in certain scenarios.
```

Nginx在1.11.3版本时采用了该解决方案，所以从该版本开始，配置accept_mutex默认为off。



## 总结

非IO复用的惊群在2.6内核就通过增加WQ_FLAG_EXCLUSIVE在内核中就行排他解决惊群了；epoll的惊群在3.10内核加了SO_REUSEPORT来解决惊群，但同时带来了不同的worker有的饥饿有的排队假死一样；4.5的内核增加EPOLLEXCLUSIVE在内核中直接将worker放在一个大queue，同时感知worker状态来派发任务更好滴解决了惊群，但是因为LIFO的机制导致在压力不大的情况下，任务主要派发给少数几个worker（能接受，压力大就会正常了）。



















