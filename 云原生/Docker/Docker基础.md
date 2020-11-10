# CentOS 安装 Docker

---

```bash
# uname -r
5.1.5-1.el7.elrepo.x86_64 # 要大于3.10
# yum update
# vim /etc/yum.repos.d/ghostcloud.repo
```

在 /etc/yum.repos.d/ghostcloud.repo 中写入

```
[dockerrepo]
name=Docker Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/$releasever
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg 

```

然后继续执行：

```bash
# yum install docker-engine
# service docker start #后台启动docker服务
# docker version #查看docker版本
```

`docker-engine` 在17.03版本后改名为 `docker-ee` 和 `docker-ce`

安装 `docker-ce`

```
# yum install -y yum-utils device-mapper-persistent-data lvm2 #依赖安装
# yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo #添加源
# #阿里云源
# yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# yum makecache fast #将软件包添加到本地缓存
# yum install docker-ce docker-ce-cli containerd.io
# systemctl restart docker # 重启docker
```

# Docker相关知识及命令

一些命令：

```
# docker run hello-world #运行一个hello，world项目
# docker images #查看本地有哪些镜像
# docker ps #查看本地正在运行的容器 -a参数可以显示已经死亡的容器

# docker info #查看docker的信息
# docker pull ubuntu #拉取ubuntu镜像
# docker search php #搜索相关镜像
# docker run -d -p 90:80 webdevops/php-apache #-p代表指定端口号，前面是宿主机的，后面是docker容器的端口号
```

Docker 依赖的 Linux 三大核心功能

- CGroups 用来限定一个进程的资源使用
- Namespace 用来划分不同的命名空间
- UnionFS 用来处理分层镜像


## 一个可交互的含Shell终端的容器

```
# docker run -i -t ubuntu /bin/bash
```

- -i:表示启动一个可交互的容器
- -t：表示使用pseudo-TTY，关联到容器的stdio和stdout
- /bin/bash: 表示启动容器时需要运行的命令
- 

## 运行一个长时间运行的容器

```
# docker run -d ubuntu /bin/sh -c "while true; do echo Hello World; sleep 1; done"
# docker logs <container_id> #查看容器日志
# docker ps #查看后台运行的容器
# docker kill <container_id> #根据容器ID杀死容器
```

- -d: 让容器在后台运行，不关联到shell上 


## 持久化容器

```
# docker exec -it 64a /bin/bash #在指定的容器中运行命令
```

在容器退出后，并不会更改镜像，因此，如果希望保存容器重的数据，就需要通过commit来保存成自己的镜像！！！

```
# docker commit 64a my-apache-php
```

## 镜像制作

获取镜像有三种方式：1，拉取镜像。2，把容器转换为镜像。3，使用Dockerfile生成镜像

## 根据Dockerfile编译镜像

```
# vim DockerFile
# docker build .
# docker images
# docker rmi 09a4 # 删除镜像
```

## docker run 的一些参数

- -d：对应的英文是detached，即把容器放到后台运行，当使用了-d参数后，如果想再次进入容器，可以使用 `docker attach <cid>` 的方式重新绑定到当前Shell，如果再次进入-d方式，则不能输入`ctrl + c`，而是使用`ctrl + pq`
- -it:把容器当道前台运行
- --name: 当创建容器时，可以指定容器的名字，名字的最大用途就是容器间通信
- --pid=host：容器与主机公用PID
- --uts：可以使容器和主机使用相同的hostname和domain，慎重使用。
- --ipc：支持进程间通信，可以和主机通信
- --dns=[]: 为容器设置自定义的DNS
- --net="bridge"：设置容器的网络模式
- --rm：彻底清除容器，包括容器的残留文件
- --device：支持容器访问某些设备
- --log-driver=json-file：设置日志的输入策略。

# 容器的网络

Docker 安装成功后，就会创建三种网络，可以通过`docker network ls`进行查看
这三种网络分别如下：

- bridge
- none
- host

创建容器时可以使用 `--net` 指定。对于birdge而言，默认是挂接在主机上的，
可以在主机上通过 `ifconfig` 查看到以docker命名的网卡设备。

通过 `docker network inspect bridge` 来查看bridge类型的容器的网络信息。
每次运行一个容器，都会在全局注册相关的网络信息。

除了默认的网络机制外，用户还可以创建自己的网络：桥接网络，Overlay网络，插件网络。

自定义桥接网络

```
# docker network create --driver bridge mynet 
# docker network inspect mynet
# docker run --net=mynet --rm -it ubuntu
```

在同一桥接下，形成了一个私网，相互间是可以通信的，但是仅限于在同一台主机上，若要跨主机通信，就必须使用Overlay网络

Overlay网络

# 容器的数据

如果用户想将主机上的文件系统共享给容器使用，怎么办那？方式如下：

- 数据卷，将主机的卷mount进入容器
- 数据容器，将外部容器分享给容器

-----

可以使用在创建容器时使用 `-v` 来指定数据卷

```
# docker run -d -p --name datatest -v /webapp ubuntu
```

这样就在容器中创建了一个 `/webapp` 目录
使用 `docker inspect <cid>` 可以在 `Mounts` 中看到实际映射到主机中的地址

```
# docker run -i -t -v /root:/webapp ubuntu /bin/bash
```

这样可以定义主机中映射的目录

-----

使用数据型容器

由于容器本身可以包含文件系统，那么就可以把容器的卷分享给另外一个容器使用！

操作方式：
1，使用 `create` 创建一个包含数据卷的容器，注意是create，不是run，
run是create之后再start

```
# docker create -v /dbdata --name dbstore ubuntu
```

然后在另外一个容器中通过`--volumes-from` 来映射

```
# docker run --rm -it --volumes-from dbstore ubuntu
```

容器间是通过名字来标记的。

# 镜像和容器的储存结构

Docker为了节约空间及共享数据，会对镜像和容器进行分层，
不同的镜像可以分享相同的数据。

Docker为了加快容器的启动速度，在启动时，会在镜像上为容器分配一个可读写的数据层
，在容器中，数据的增删改都保存在这个容器层中，但是容器退出时，
并不会把这些数据保存到镜像，需要执行 `docker commit` 来从容器新建一个镜像。

Docker 支持多种不同的储存方式，每种储存方式保存镜像和容器的方法都不相同。

```
# docker history <cid> #查看镜像的分层
```

储存驱动用于管理这些镜像层，对外提供一个单一的文件系统。

容器在修改文件时，会将文件从镜像层复制到容器层。

# CentOS Docker 开启远程访问

```
# systemctl enable docker #开机启动docker，会创建/usr/lib/systemd/system/docker.service配置文件
# vim /usr/lib/systemd/system/docker.service
```

在配置文件中加入：

```
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock
```

然后执行：

```
# systemctl daemon-reload #重新加载配置文件
# systemctl restart docker #重启Docker
```

验证：

```
# netstat -lntp # 查看端口占用
# docker -H tcp://127.0.0.1:2375 images # 查看远程机器上的docker
# curl 127.0.0.1:2375/info #查看docker rest服务
```

对外开放端口：

```
# iptables -A INPUT -ptcp --dport 2375 -j ACCEPT
# yum install iptables-services
# systemctl enable iptables #开机启动服务
# service iptables save #保存规则
# cat /etc/sysconfig/iptables #查看规则
```

另外在服务器厂商那里要添加两条规则，对0.0.0.0/0和::/0都要加入规则

# 实战演练

先创建一个 `bridge` 网络用于容器上的服务互相调用

```
# docker network create -d bridge serrhub_network #只能用于本机的网络
# docker network inspect serrhub_network #查看网络状态
```

然后创建一个 `mongo` 容器，并且将它连接到上面创建的网络上

```
# docker pull mongo
# # --rm 用于在 docker kill 时干净的清除容器数据，-v 映射的数据卷将数据储存在本机中，--network 用于指定网络，--name 为容器命名，-d 指定了镜像
# docker run --rm -v /data/docker/db:/data/db --network=serrhub_network --name docker_mongodb -d mongo
# docker ps #查看正在运行容器是否有docker_mongodb
# docker network inspect serrhub_network #查看创建的网络状态，会发现mongo已经连接到网络上了
# mongo 172.18.0.2:27017 #验证连接，地址是从网络状态中得来的
```

下一步，通过 `Maven` 创建一个运行 `Spring boot` 项目的 docker 镜像

首先，`pom.xml` 加入以下代码

```
<properties>
    <docker.image.prefix>hacksoul</docker.image.prefix>
</properties>
<build>
    <plugins>
        <plugin>
            <groupId>com.spotify</groupId>
            <artifactId>dockerfile-maven-plugin</artifactId>
            <version>1.4.9</version>
            <configuration>
                <repository>${docker.image.prefix}/${project.artifactId}</repository>
                <buildArgs>
                 <JAR_FILE>target/${project.build.finalName}.jar</JAR_FILE>
                </buildArgs>
            </configuration>
        </plugin>

    </plugins>
</build>
```

需要在 Maven 目录下的 `conf/setting.xml` 的 `pluginGroups` 节点下
加入以下代码来信任 com.spotify 的插件：

```
<pluginGroup>com.spotify</pluginGroup> 
```

然后编写Dockerfile

```
# 基础镜像
FROM openjdk:8-jdk-alpine
# 对应pom.xml文件中的dockerfile-maven-plugin插件JAR_FILE的值
ARG JAR_FILE
# 复制打包完成后的jar文件到/opt目录下
COPY ${JAR_FILE} /opt/serrhub.jar
# 挂载目录到主机
VOLUME /tmp
VOLUME /var/log/serrhub
# 启动容器时执行,使用 /bin/sh -c 来执行会让日志重定向到挂载卷中的目录中
ENTRYPOINT ["/bin/sh", "-c", "java -jar /opt/serrhub.jar > /var/log/serrhub/serrhub.log"]
EXPOSE 8001
```

然后将代码通过git上传到服务端

在服务端执行以下脚本：

```
#!/bin/bash

#get source
git pull

#构建镜像
mvn package -Dmaven.test.skip=true dockerfile:build

#杀死老容器
docker kill hacksoul_serrhub

#运行新容器，并且连接到之前创建的网络，端口映射到宿主机可以让nginx来代理
#不指定tag的话，最新的镜像版本默认会是latest
docker run --rm -p 8002:8001 -v /var/log/serrhub:/var/log/serrhub --network=serrhub_network --name hacksoul_serrhub -d hacksoul/serrhub:latest
```

在Spring容器中，只需使用 `docker_mongodb` 来代替mongo地址即可，
这个名字是mongo容器的名字。

mongo容器是一个新库，需要迁移数据：

```
# # -u 用户名，-p 密码，-d 库名，-o 输出目录
# mongodump -h 127.0.0.1:27017 -u zhixi -p serrhub_zhixi -d serrhub -o /data/docker/db/old_db_data/mongo_dump
# # --authenticationDatabase 用于指定库名，--nsInclude 很关键，使用通配符来指定库下的所有集合，最后跟一个备份数据的目录名
# mongorestore -h 172.18.0.2:27017 -u zhixi -p serrhub_zhixi --authenticationDatabase serrhub --nsInclude 'serrhub.*' /data/docker/db/old_db_data/mongo_dump
```

# Dockerfile指令详解

第一条指令一定是FROM指令，FROM指定了基础镜像

MAINTAINER 设置镜像作者

RUN 指令会生成一个新容器，在容器中执行脚本，脚本执行完成后，会把该容器提交为一个中间镜像，
供后面的指令使用。

RUN 有一种hack方式，就是 `RUN ["/bin/sh", "-c", "echo $HOME"]` ,这个命令可以打印环境变量，
而 `RUN ["echo", "$HOME"]` 则不能。这里必须使用双引号！

Dockerfile 中只能有一条CMD指令。如果有多条，只有最后一条指令会生效。

LABEL 指令可以为镜像设置key-value值，最好用一个LABEL指令生成多个键值对，因为每个LABEL都会
生成一个镜像层，Docker规定最多有127个镜像层。`docker inspect` 可以查看镜像Label。

EXPOSE 设置镜像报漏端口，记录容器启动时监听哪些端口。如果想在宿主机上使用这些端口，
还需要在 RUN 时添加 -p 参数。

ENV 指令可以设置镜像中的环境变量。这些变量在整个编译周期有效，对运行中的容器时无效的。

ADD 指令可以将文件复制到镜像中，源文件默认在编译路径中找。

COPY 类似 ADD

ENTRYPOINT 指定容器入口程序，类似于 RUN 。

VOLUME 指定挂载卷。

USER 指定RUN，CMD，ENTRYPOINT的用户名或UID。

WORKDIR 指定了RUN，CMD，ENTRYPOINT，ADD，COPY的工作目录。

ARG 用于获取 `docker build` 中的参数

当父镜像生成子镜像时，会调用父镜像的 ONBUILD 指令。

SOPSIGNAL 指定了在容器退出时，Docker向进程发送的信号量。
