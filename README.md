# 日常工作技术记录

分别包括 `编程语言`，`大数据`，`前端`，`数据储存`，`云原生`，`DevOps`，`Java`，`Linux` 这几个方面，另外还有其他的一些编程基础。



---



## 云原生

云原生是最近兴起的开源基金会，包含许多高质量的开源项目，大多依托于 Docker 及 Kubernetes。这些开源项目大多是 Golang 写的。

- Kubernetes

   [Kubernetes二进制安装.md](云原生/Kubernetes/Kubernetes二进制安装.md) 

    [核心组件运行原理.md](云原生/Kubernetes/核心组件运行原理.md) 

    [Kubernetes-Leader选举.md](云原生/Kubernetes/Kubernetes-Leader选举.md) 

    [k8s网络原理.md](云原生/Kubernetes/k8s网络原理.md) 

    [共享储存原理.md](云原生/Kubernetes/共享储存原理.md) 

    [深入分析集群安全机制.md](云原生/Kubernetes/深入分析集群安全机制.md) 

    [pause容器作用.md](云原生/Kubernetes/pause容器作用.md) 

    [Secret.md](云原生/Kubernetes/Secret.md) 

    [Kubernetes-Ingress-Controller.md](云原生/Kubernetes/Kubernetes-Ingress-Controller.md) 

    [K8sFAQ.md](云原生/Kubernetes/K8sFAQ.md) 

    [二进制部署Flannel.md](云原生/Kubernetes/二进制部署Flannel.md) 

    [kubectl查看资源对象.md](云原生/Kubernetes/kubectl查看资源对象.md) 

    [List-Watch原理.md](云原生/Kubernetes/List-Watch原理.md) 

    [kube-controller-manager内置的控制器.md](云原生/Kubernetes/kube-controller-manager内置的控制器.md) 

   - Kubernetes 配置系列

      [kube-apiserver配置详解.md](云原生/Kubernetes/Kubernetes配置系列/kube-apiserver配置详解.md) 

      [kube-controller-manager配置详解.md](云原生/Kubernetes/Kubernetes配置系列/kube-controller-manager配置详解.md) 

      [kube-scheduler配置详解.md](云原生/Kubernetes/Kubernetes配置系列/kube-scheduler配置详解.md) 

      [kube-proxy配置详解.md](云原生/Kubernetes/Kubernetes配置系列/kube-proxy配置详解.md) 

      [kubelet配置详解.md](云原生/Kubernetes/Kubernetes配置系列/kubelet配置详解.md) 

      [kubectl命令行参考.md](云原生/Kubernetes/Kubernetes配置系列/kubectl命令行参考.md) 

   - Kubernetes 源码系列

      [Kubernetes源码系列(一)-源码结构.md](云原生/Kubernetes/Kubernetes源码系列/Kubernetes源码系列(一)-源码结构.md) 

      [Kubernetes源码系列(二)-kubectl源码分析.md](云原生/Kubernetes/Kubernetes源码系列/Kubernetes源码系列(二)-kubectl源码分析.md) 

      [Kubernetes源码系列(三)-kube-proxy源码分析.md](云原生/Kubernetes/Kubernetes源码系列/Kubernetes源码系列(三)-kube-proxy源码分析.md) 

      [Kubernetes源码系列(四)-kubelet源码分析.md](云原生/Kubernetes/Kubernetes源码系列/Kubernetes源码系列(四)-kubelet源码分析.md) 

      [Kubernetes源码系列(五)-kube-scheduler源码分析.md](云原生/Kubernetes/Kubernetes源码系列/Kubernetes源码系列(五)-kube-scheduler源码分析.md) 

      [Kubernetes源码系列(六)-kube-controller-manager源码分析.md](云原生/Kubernetes/Kubernetes源码系列/Kubernetes源码系列(六)-kube-controller-manager源码分析.md) 

      [Kubernetes源码系列(七)-kube-apiserver源码分析.md](云原生/Kubernetes/Kubernetes源码系列/Kubernetes源码系列(七)-kube-apiserver源码分析.md) 

   - Kubernetes 扩展机制

      - CNI

         [CNI工作机制.md](云原生/Kubernetes/Kubernetes扩展机制/CNI/CNI工作机制.md) 

         [cnitool使用方式.md](云原生/Kubernetes/Kubernetes扩展机制/CNI/cnitool使用方式.md) 

         [手写一款CNI插件.md](云原生/Kubernetes/Kubernetes扩展机制/CNI/手写一款CNI插件.md) 

      - CRI

         [CRI工作机制.md](云原生/Kubernetes/Kubernetes扩展机制/CRI/CRI工作机制.md) 

         [CRI-Tools.md](云原生/Kubernetes/Kubernetes扩展机制/CRI/CRI-Tools.md) 

         [CRI与OCI的关系.md](云原生/Kubernetes/Kubernetes扩展机制/CRI/CRI与OCI的关系.md) 

         [CRI-API.md](云原生/Kubernetes/Kubernetes扩展机制/CRI/CRI-API.md) 

      - CSI

         [CSI工作方式.md](云原生/Kubernetes/Kubernetes扩展机制/CSI/CSI工作方式.md) 

   - Kubernetes API

      [Kubernetes-API访问方式.md](云原生/Kubernetes/Kubernetes-API/Kubernetes-API访问方式.md) 

      [swagger.json位置.md](云原生/Kubernetes/Kubernetes-API/swagger.json位置.md) 

      [Kubernetes-API学习(一)-介绍.md](云原生/Kubernetes/Kubernetes-API/Kubernetes-API学习(一)-介绍.md) 

      [Kubernetes-API学习(二)-核心API.md](云原生/Kubernetes/Kubernetes-API/Kubernetes-API学习(二)-核心API.md) 

      [Kubernetes-API学习(三)-apps.md](云原生/Kubernetes/Kubernetes-API/Kubernetes-API学习(三)-apps.md) 

      [Kubernetes-API学习(四)-batch.md](云原生/Kubernetes/Kubernetes-API/Kubernetes-API学习(四)-batch.md) 

      [Kubernetes-API学习(五)-RDBC.md](云原生/Kubernetes/Kubernetes-API/Kubernetes-API学习(五)-RDBC.md) 

      [Kubernetes-API学习(六)-admissionregistration.md](云原生/Kubernetes/Kubernetes-API/Kubernetes-API学习(六)-admissionregistration.md) 

      [Kubernetes-API学习(七)-APIService.md](云原生/Kubernetes/Kubernetes-API/Kubernetes-API学习(七)-APIService.md) 

      [Kubernetes-API学习(八)-networking.md](云原生/Kubernetes/Kubernetes-API/Kubernetes-API学习(八)-networking.md) 

      [Kubernetes-API学习(九)-CRD.md](云原生/Kubernetes/Kubernetes-API/Kubernetes-API学习(九)-CRD.md) 

      [Kubernetes-API学习(十)-autoscaling.md](云原生/Kubernetes/Kubernetes-API/Kubernetes-API学习(十)-autoscaling.md) 

      [Kubernetes-API学习(十一)-authentication.md](云原生/Kubernetes/Kubernetes-API/Kubernetes-API学习(十一)-authentication.md) 

      [Kubernetes-API学习(十二)-RuntimeClass.md](云原生/Kubernetes/Kubernetes-API/Kubernetes-API学习(十二)-RuntimeClass.md) 

      [Kubernetes-API学习(十三)-policy.md](云原生/Kubernetes/Kubernetes-API/Kubernetes-API学习(十三)-policy.md) 

      [Kubernetes-API学习(十四)-scheduling.md](云原生/Kubernetes/Kubernetes-API/Kubernetes-API学习(十四)-scheduling.md) 

      [Kubernetes-API学习(十五)-certificates.md](云原生/Kubernetes/Kubernetes-API/Kubernetes-API学习(十五)-certificates.md) 

      [Kubernetes-API学习(十六)-coordination.md](云原生/Kubernetes/Kubernetes-API/Kubernetes-API学习(十六)-coordination.md) 

      [Kubernetes-API学习(十七)-storage.md](云原生/Kubernetes/Kubernetes-API/Kubernetes-API学习(十七)-storage.md) 

   - 动手操作

      [部署Zookeeper.md](云原生/Kubernetes/动手操作/部署Zookeeper.md) 

      [创建PV及PVC.md](云原生/Kubernetes/动手操作/创建PV及PVC.md) 

      [从私有仓库拉取镜像.md](云原生/Kubernetes/动手操作/从私有仓库拉取镜像.md) 

      [公开外部IP地址以访问集群中应用程序.md](云原生/Kubernetes/动手操作/公开外部IP地址以访问集群中应用程序.md) 

      [将Pod分配给节点.md](云原生/Kubernetes/动手操作/将Pod分配给节点.md) 

      [配置存活、就绪和启动探测器.md](云原生/Kubernetes/动手操作/配置存活、就绪和启动探测器.md) 

      [配置Pod初始化.md](云原生/Kubernetes/动手操作/配置Pod初始化.md) 

      [配置Pod的服务质量.md](云原生/Kubernetes/动手操作/配置Pod的服务质量.md) 

      [配置Pod使用投射卷作存储.md](云原生/Kubernetes/动手操作/配置Pod使用投射卷作存储.md) 

      [配置Pod以使用卷进行存储.md](云原生/Kubernetes/动手操作/配置Pod以使用卷进行存储.md) 

      [使用扩展进行并行处理.md](云原生/Kubernetes/动手操作/使用扩展进行并行处理.md) 

      [使用ConfigMap来配置Redis.md](云原生/Kubernetes/动手操作/使用ConfigMap来配置Redis.md) 

      [使用ConfigMap配置Pod.md](云原生/Kubernetes/动手操作/使用ConfigMap配置Pod.md) 

      [使用CronJob运行自动化任务.md](云原生/Kubernetes/动手操作/使用CronJob运行自动化任务.md) 

      [使用PodPreset将信息注入Pods.md](云原生/Kubernetes/动手操作/使用PodPreset将信息注入Pods.md) 

      [使用Secret安全地分发凭证.md](云原生/Kubernetes/动手操作/使用Secret安全地分发凭证.md) 

      [示例：使用PV部署WordPress和MySQL.md](云原生/Kubernetes/动手操作/示例：使用PV部署WordPress和MySQL.md) 

      [示例：使用Redis部署PHP留言板应用程序.md](云原生/Kubernetes/动手操作/示例：使用Redis部署PHP留言板应用程序.md) 

      [通过环境变量将Pod信息呈现给容器.md](云原生/Kubernetes/动手操作/通过环境变量将Pod信息呈现给容器.md) 

      [通过文件将Pod信息呈现给容器.md](云原生/Kubernetes/动手操作/通过文件将Pod信息呈现给容器.md) 

      [为节点发布扩展资源-为容器分派扩展资源.md](云原生/Kubernetes/动手操作/为节点发布扩展资源-为容器分派扩展资源.md) 
   
      [为容器的生命周期事件设置处理函数.md](云原生/Kubernetes/动手操作/为容器的生命周期事件设置处理函数.md) 
   
      [为容器设置启动时要执行的命令及其入参.md](云原生/Kubernetes/动手操作/为容器设置启动时要执行的命令及其入参.md) 
   
      [在Pod中的容器之间共享进程命名空间.md](云原生/Kubernetes/动手操作/在Pod中的容器之间共享进程命名空间.md) 

  

- Docker

   [Docker基础.md](云原生/Docker/Docker基础.md) 

   [Docker 错误汇总.md](云原生/Docker/Docker错误汇总.md) 

   [Docker 更换储存位置.md](云原生/Docker/Docker更换储存位置.md) 

   [Docker 远程连接.md](云原生/Docker/Docker远程连接.md) 

   [Docker镜像相关.md](云原生/Docker/Docker镜像相关.md) 

   [Docker众产品笔记.md](云原生/Docker/Docker众产品笔记.md) 

   [Dockerfile语法.md](云原生/Docker/Dockerfile语法.md) 

   [Docker-CLIs.md](云原生/Docker/Docker-CLIs.md) 

   [动手理解Docker原理.md](云原生/Docker/动手理解Docker原理.md) 

   [Docker配置.md](云原生/Docker/Docker配置.md) 

   - Docker-API

      [Docker-API(一)-介绍.md](云原生/Docker/Docker-API/Docker-API(一)-介绍.md) 

      [Docker-API(二)-Containers.md](云原生/Docker/Docker-API/Docker-API(二)-Containers.md) 
   
      [Docker-API(三)-Images.md](云原生/Docker/Docker-API/Docker-API(三)-Images.md) 
      
      [Docker-API(四)-Network.md](云原生/Docker/Docker-API/Docker-API(四)-Network.md) 
      
      [Docker-API(五)-Volume.md](云原生/Docker/Docker-API/Docker-API(五)-Volume.md) 
      
      [Docker-API(六)-Exec.md](云原生/Docker/Docker-API/Docker-API(六)-Exec.md) 
      
      [Docker-API(七)-System.md](云原生/Docker/Docker-API/Docker-API(七)-System.md) 



- containerd

  [containerd介绍.md](云原生/containerd/containerd介绍.md) 



- etcd

  [etcd-demo.md](云原生/Etcd/etcd-demo.md) 

  [etcd动态发现.md](云原生/Etcd/etcd动态发现.md) 

  [Metrics.md](云原生/Etcd/Metrics.md) 

  [etcd配置详解.md](云原生/Etcd/etcd配置详解.md) 

  [etcdctl详解.md](云原生/Etcd/Etcdctl详解.md) 

  [etcd-golang客户端.md](云原生/Etcd/etcd-golang客户端.md) 

  [etcd中的各种版本.md](云原生/Etcd/etcd中的各种版本.md) 

  - etch - API

    [Etcd-Api(一)-介绍.md](云原生/Etcd/etcd-API/Etcd-Api(一)-介绍.md) 

    [Etcd-API(二)-KV.md](云原生/Etcd/etcd-API/Etcd-API(二)-KV.md) 

    [Etcd-API(三)-Watch.md](云原生/Etcd/etcd-API/Etcd-API(三)-Watch.md) 

    [Etcd-API(四)-Lease.md](云原生/Etcd/etcd-API/Etcd-API(四)-Lease.md) 

    [Etcd-API(五)-Cluster.md](云原生/Etcd/etcd-API/Etcd-API(五)-Cluster.md) 

    [Etcd-API(六)-Maintenance.md](云原生/Etcd/etcd-API/Etcd-API(六)-Maintenance.md) 

    [Etcd-API(七)-Auth.md](云原生/Etcd/etcd-API/Etcd-API(七)-Auth.md) 



- gRPC

  [Java构建gRPC服务.md](云原生/gRPC/Java构建gRPC服务.md) 

  [Protobuf语法.md](云原生/gRPC/Protobuf语法.md) 

  [Golang构建gRPC服务.md](云原生/gRPC/Golang构建gRPC服务.md) 

  [gRPC流式传输.md](云原生/gRPC/gRPC流式传输.md) 

  [gRPC认证.md](云原生/gRPC/gRPC认证.md) 

  [grpc-gateway使用.md](云原生/gRPC/grpc-gateway使用.md) 

  [gRPC使用sock文件进行通信.md](云原生/gRPC/gRPC使用sock文件进行通信.md) 
  
  [gRPC拦截器.md](云原生/gRPC/gRPC拦截器.md) 



- Helm

   [Helm入门文档.md](云原生/Helm/Helm入门文档.md) 

   [Helm命令.md](云原生/Helm/Helm命令.md) 

   [Helm Chart Template手册.md.md](云原生/Helm/Helm-Chart-Template手册.md) 

   [Helm最佳实践.md](云原生/Helm/Helm最佳实践.md) 

   

- Pulumi

   [Pulumi入门.md](云原生/Pulumi/Pulumi入门.md) 

   

- Istio

   [Istio安装教程.md](云原生/Istio/Istio安装教程.md) 

   [Istio Hello-world 教程.md](云原生/Istio/Istio之Hello-world教程.md) 

   [Istio 错误记录.md](云原生/Istio/Istio错误记录.md) 

   

- Jaeger

   [Jaeger 入门.md](云原生/jaeger/Jaeger入门.md) 

   [Jaeger-Operator.md](云原生/jaeger/Jaeger-Operator.md) 

   

- Rancher

   [Rancher安装.md](云原生/Rancher/Rancher安装.md) 

   

- Rook

   [Rook安装.md](云原生/Rook/Rook安装.md) 

   [Rook-Ceph 块储存.md](云原生/Rook/Rook-Ceph块储存.md) 

   [Rook-Ceph 对象储存.md](云原生/Rook/Rook-Ceph对象储存.md) 

   [Deph mgr 密码.md](云原生/Rook/Deph-mgr密码.md) 

   

- Telepresence

   [Telepresence安装及入门.md](云原生/Telepresence/Telepresence安装及入门.md) 



- CoreDNS

    [测试k8s集群中安装的CoreDNS.md](云原生/CoreDNS/测试k8s集群中安装的CoreDNS.md) 

    

- Vitess

  

- Knative

   [Knative安装.md](云原生/Knative/Knative安装.md) 

   

- Kubeless

   [Kubeless入门.md](云原生/Kubeless/Kubeless入门.md) 

   

- Vault

   [Vault简介及入门.md](云原生/Vault/Vault简介及入门.md) 

   [部署vault-operator.md](云原生/Vault/部署vault-operator.md) 

   

- Fluentd

   [Fluentd安装.md](云原生/Fluentd/Fluentd安装.md) 

   [Fluentd配置.md](云原生/Fluentd/Fluentd配置.md) 

   

- Prometheus

   [Prometheus入门.md](云原生/Prometheus/Prometheus入门.md) 

   [Prometheus概念.md](云原生/Prometheus/Prometheus概念.md) 

   [Prometheus教程.md](云原生/Prometheus/Prometheus教程.md) 

   [Prometheus配置.md](云原生/Prometheus/Prometheus配置.md) 

   [Prometheus查询.md](云原生/Prometheus/Prometheus查询.md) 

   [Prometheus储存.md](云原生/Prometheus/Prometheus储存.md) 

   [Prometheus集群.md](云原生/Prometheus/Prometheus集群.md) 

   [Prometheus可视化.md](云原生/Prometheus/Prometheus可视化.md) 

   [Prometheus安全.md](云原生/Prometheus/Prometheus安全.md) 

   [Prometheus-exporters.md](云原生/Prometheus/Prometheus-exporters.md) 

   [Prometheus告警.md](云原生/Prometheus/Prometheus告警.md) 

   [Prometheus客户端.md](云原生/Prometheus/Prometheus客户端.md) 

   [Prometheus架构.md](云原生/Prometheus/Prometheus架构.md) 

   

- Grafana

   [Grafana入门.md](云原生/Grafana/Grafana入门.md) 

   

- Calico

   [calicoctl 使用方法.md](云原生/Calico/calicoctl使用方法.md) 



- Operator

   [Kubernetes-Operator教程.md](云原生/Operator/Kubernetes-Operator教程.md) 





---



## 大数据

CDH 开始收费了，怎么说。。。

- 爬虫

   [Scrapy入门教程.md](大数据/爬虫/Scrapy入门教程.md) 

   [Splash使用教程.md](大数据/爬虫/Splash使用教程.md) 

- Flink

   [Flink-Table教程.md](大数据/Flink/Flink-Table教程.md) 

- Flume

   [Flume入门教程.md](大数据/Flume/Flume入门教程.md) 

- HBase

   [HBase入门教程.md](大数据/HBase/HBase入门教程.md) 

- HDFS

- Hive

   [Java连接Hive.md](大数据/Hive/Java连接Hive.md) 

   [Hive错误处理.md](大数据/Hive/Hive错误处理.md) 

- Kafka

   [Kafka快速入门.md](大数据/Kafka/Kafka快速入门.md) 

- Oozie

   [Oozie入门.md](大数据/Oozie/Oozie入门.md) 

- Spark

   [Spark入门教程.md](大数据/Spark/Spark入门教程.md) 

   [Spark RDD及编程接口的学习.md](大数据/Spark/Spark-RDD及编程接口的学习.md) 

   [Spark-Streaming学习指南.md](大数据/Spark/Spark-Streaming学习指南.md) 

   [Spark SQL入门教程.md](大数据/Spark/Spark-SQL入门教程.md) 

- YARN

   [YARN入门教程.md](大数据/YARN/YARN入门教程.md) 

   [YARN命令.md](大数据/YARN/YARN命令.md) 

   [YARN架构.md](大数据/YARN/YARN架构.md) 

- ZooKeeper

   [ZooKeeper介绍.md](大数据/ZooKeeper/ZooKeeper介绍.md) 

   [ZooKeeper使用入门.md](大数据/ZooKeeper/ZooKeeper使用入门.md) 

   [ZooKeeper开发手册.md](大数据/ZooKeeper/ZooKeeper开发手册.md) 

   [ZooKeeper管理手册.md](大数据/ZooKeeper/ZooKeeper管理手册.md) 



---



## 数据储存

数据储存包括数据库技术，也包含一些分布式储存技术。

- Elasticsearch

   [ES入门教程.md](数据存储/Elasticsearch/ES入门教程.md) 

   [ES之RESTAPIs.md](数据存储/Elasticsearch/ES之RESTAPIs.md) 

   [ES聚合.md](数据存储/Elasticsearch/ES聚合.md) 

   [ES之Mapping.md](数据存储/Elasticsearch/ES之Mapping.md) 

   [ES之QueryDSL.md](数据存储/Elasticsearch/ES之QueryDSL.md) 

   [ES之QueryDSL二.md](数据存储/Elasticsearch/ES之QueryDSL二.md) 

   [ES开启安全认证.md](数据存储/Elasticsearch/ES开启安全认证.md) 

- Ceph

- MongoDB

- MySQL

- PostgrerSQL

- Redis

- Neo4j



---



## 编程语言

挑几门有前途的学

- Rust

   [Cargo教程.md](编程语言/Rust/Cargo教程.md) 

   [Rust-WASM入门.md](编程语言/Rust/Rust-WASM入门.md) 

- Scala

   [Scala基础语法学习.md](编程语言/Scala/Scala基础语法学习.md) 

   [Scala语法深入学习.md](编程语言/Scala/Scala语法深入学习.md) 

- Dart

- JavaScript

- TypeScript

- Golang

    [Golang协程入门.md](编程语言/Golang/Golang协程入门.md) 

    [摆脱GOPATH.md](编程语言/Golang/摆脱GOPATH.md) 

    [Golang教程.md](编程语言/Golang/Golang教程.md) 

    [Go-FAQ.md](编程语言/Golang/Go-FAQ.md) 

    [Go命令详解.md](编程语言/Golang/Go命令详解.md) 

    [Golang类型转换.md](编程语言/Golang/Golang类型转换.md) 

    - Cobra

       [Cobra入门.md](编程语言/Golang/Cobra/Cobra入门.md) 

    - go-restful

       [go-restful入门.md](编程语言/Golang/go-restful/go-restful入门.md) 

- Swift

- Python

    [Python教程.md](编程语言/Python/Python教程.md) 



---



## 前端



- Flutter

   [Flutter 小技巧.md](前端/Flutter/Flutter小技巧.md) 

- Vue.js

- w3c

   [WebIDL语言.md](前端/w3c/WebIDL语言.md) 



---



## DevOps



- Gitlab

- Jenkins

- Kibana

   [Kibana入门教程.md](DevOps/Kibana/Kibana入门教程.md) 

   [Kibana发现页.md](DevOps/Kibana/Kibana发现页.md)

   [Kibana 错误记录.md](DevOps/Kibana/Kibana错误记录.md) 

- Logstash

   [LogStash入门教程.md](DevOps/Logstash/LogStash入门教程.md) 

   [设置并运行Logstash.md](DevOps/Logstash/设置并运行Logstash.md) 

   [LogStash配置详解.md](DevOps/Logstash/LogStash配置详解.md) 

- SlatStack



---



## Java

 [ClassNotFoundException与NoClassDefFoundError.md](Java/ClassNotFoundException与NoClassDefFoundError.md) 

 [package-info的作用.md](Java/package-info的作用.md) 

- Gradle

- Maven

- JVM



---



## Linux

- syslog

   [syslog-ng使用教程.md](Linux/syslog/syslog-ng使用教程.md) 

   

- iptables

     [iptables教程.md](Linux/iptables/iptables教程.md) 



- Shell

    [Shell脚本语法记录.md](Linux/Shell/Shell脚本语法记录.md) 

   [常用命令.md](Linux/Shell/常用命令.md) 

   [错误记录.md](Linux/Shell/错误记录.md) 

   [删除脚本.md](Linux/Shell/删除脚本.md) 
   
   [一次网络命令实践.md](Linux/Shell/一次网络命令实践.md) 
   
   [内核升级.md](Linux/Shell/内核升级.md) 
   
   [iproute2命令详解.md](Linux/Shell/iproute2命令详解.md) 



- systemd



- SELinux



- yum

  



---



## 其他

- 计算机网络

- 设计模式

- 数据结构与算法

- 区块链

- OpenSSL

- TensorFlow

- 日常

   [chrome没有继续前往.md](其他/日常/chrome没有继续前往.md) 
   
   [MacOS触控板与鼠标的滚动方向.md](其他/日常/MacOS触控板与鼠标的滚动方向.md) 
   
   [Mac OS X文件系统的附加属性@如何彻底删除.md](其他/日常/Mac OS X文件系统的附加属性@如何彻底删除.md) 
   
   



---



## 项目



 [大数据分析房价Demo.md](项目/大数据分析房价Demo.md) 

 [大数据分析知乎用户网络.md](项目/大数据分析知乎用户网络.md) 

 [Elasticsearch分析大乐透.md](项目/Elasticsearch分析大乐透.md) 

 [IDEA插件开发之 K8s-Client.md](项目/IDEA插件开发之K8s-Client.md) 











