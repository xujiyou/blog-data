# CPU 占用高问题排查

排查顺序：

- 查消耗cpu最高的进程PID
- 根据PID查出消耗cpu最高的线程号
- 根据线程号查出对应的java线程，进行处理。



我这里有一套测试环境，可以用于排查问题所在。



1. 使用 `top -c`，再按下 P 来按照 CPU 使用率顺序来对进程排序，找到其中占用最高的 Java 程序的 PID 。
2. 使用 `top -Hp PID` 查看进程的线程，然后按下 P 来按照 CPU 使用率顺序来对线程排序，找到线程 ID，然后把 ID 转成 16 进制的。

3. 使用 `jstack -l 3033 > ./3033.stack` 导出运行时堆栈（注意这里的ID是进程ID），然后 vim 读取这个文件，搜索 16 进制的 线程ID 即可。



如果使用的是 Kubernetes ，可以使用 `kubectl top pod --all-namespaces --sort-by=cpu` 找到对应的 Pod，然后进入 Pod 进行操作！

