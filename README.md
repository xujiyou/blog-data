# 日常工作技术记录

分别包括 `编程语言`，`大数据`，`前端`，`数据储存`，`云原生`，`DevOps`，`Java`，`Linux` 这几个方面，另外还有其他的一些编程基础。



---



## 云原生

云原生是最近兴起的开源基金会，包含许多高质量的开源项目，大多依托于 Docker 及 Kubernetes。这些开源项目大多是 Golang 写的。

- Kubernetes

   [核心组件运行原理.md](云原生/Kubernetes/核心组件运行原理.md) 

   [深入分析集群安全机制.md](云原生/Kubernetes/深入分析集群安全机制.md) 

   [k8s网络原理.md](云原生/Kubernetes/k8s网络原理.md)

   [共享储存原理.md](云原生/Kubernetes/共享储存原理.md) 

   [pause容器作用.md](云原生/Kubernetes/pause容器作用.md) 

   [kubectl命令行参考.md](云原生/Kubernetes/kubectl命令行参考.md) 

   [Kubernetes-Operator教程.md](云原生/Kubernetes/Kubernetes-Operator教程.md) 

   

   [创建PV及PVC.md](云原生/Kubernetes/创建PV及PVC.md) 

   [从私有仓库拉取镜像.md](云原生/Kubernetes/从私有仓库拉取镜像.md) 

   [公开外部IP地址以访问集群中应用程序.md](云原生/Kubernetes/公开外部IP地址以访问集群中应用程序.md) 

   [将 Pod 分配给节点.md](云原生/Kubernetes/将Pod分配给节点.md) 

   [配置 Pod 初始化.md](云原生/Kubernetes/配置Pod初始化.md) 

   [配置 Pod 的服务质量.md](云原生/Kubernetes/配置Pod的服务质量.md) 

   [配置 Pod 使用投射卷作存储.md](云原生/Kubernetes/配置Pod使用投射卷作存储.md) 

   [配置 Pod 以使用卷进行存储.md](云原生/Kubernetes/配置Pod以使用卷进行存储.md) 

   [配置存活、就绪和启动探测器.md](云原生/Kubernetes/配置存活、就绪和启动探测器.md) 

   [使用 ConfigMap 配置 Pod.md](云原生/Kubernetes/使用ConfigMap配置Pod.md) 

   [使用 CronJob 运行自动化任务.md](云原生/Kubernetes/使用CronJob运行自动化任务.md) 

   [使用 PodPreset 将信息注入 Pods.md](云原生/Kubernetes/使用PodPreset将信息注入Pods.md) 

   [使用 Secret 安全地分发凭证.md](云原生/Kubernetes/使用Secret安全地分发凭证.md) 

   [使用扩展进行并行处理.md](云原生/Kubernetes/使用扩展进行并行处理.md) 

   [使用ConfigMap来配置Redis.md](云原生/Kubernetes/使用ConfigMap来配置Redis.md) 

   [示例：使用 Persistent Volumes 部署 WordPress 和 MySQL.md](云原生/Kubernetes/示例：使用PV部署WordPress和MySQL.md) 

   [示例：使用 Redis 部署 PHP 留言板应用程序.md](云原生/Kubernetes/示例：使用Redis部署PHP留言板应用程序.md) 

   [通过环境变量将Pod信息呈现给容器.md](云原生/Kubernetes/通过环境变量将Pod信息呈现给容器.md) 

   [通过文件将Pod信息呈现给容器.md](云原生/Kubernetes/通过文件将Pod信息呈现给容器.md) 

   [为节点发布扩展资源-为容器分派扩展资源.md](云原生/Kubernetes/为节点发布扩展资源-为容器分派扩展资源.md) 

   [为容器的生命周期事件设置处理函数.md](云原生/Kubernetes/为容器的生命周期事件设置处理函数.md) 

   [为容器设置启动时要执行的命令及其入参.md](云原生/Kubernetes/为容器设置启动时要执行的命令及其入参.md) 

   [在 Pod 中的容器之间共享进程命名空间.md](云原生/Kubernetes/在Pod中的容器之间共享进程命名空间.md) 

   [K8sFAQ.md](云原生/Kubernetes/K8sFAQ.md) 

   [kubectl查看资源对象.md](云原生/Kubernetes/kubectl查看资源对象.md) 

   [Secret.md](云原生/Kubernetes/Secret.md) 

   

- Docker

   [Docker基础.md](云原生/Docker/Docker基础.md) 

   [Docker 错误汇总.md](云原生/Docker/Docker错误汇总.md) 

   [Docker 更换储存位置.md](云原生/Docker/Docker更换储存位置.md) 

   [Docker 远程连接.md](云原生/Docker/Docker远程连接.md) 

   [Docker镜像相关.md](云原生/Docker/Docker镜像相关.md) 

    [Docker众产品笔记.md](云原生/Docker/Docker众产品笔记.md) 

    [Dockerfile语法.md](云原生/Docker/Dockerfile语法.md) 

    [Docker-CLIs.md](云原生/Docker/Docker-CLIs.md) 

    [Docker-Api.md](云原生/Docker/Docker-Api.md) 

- Helm

   [Helm入门文档.md](云原生/Helm/Helm入门文档.md) 

   [Helm命令.md](云原生/Helm/Helm命令.md) 

   [Helm Chart Template手册.md.md](云原生/Helm/Helm-Chart-Template手册.md) 

   [Helm最佳实践.md](云原生/Helm/Helm最佳实践.md) 

   

- Pulumi

   [Pulumi入门.md](云原生/Pulumi/Pulumi入门.md) 

   

- Istio

   [Istio安装教程.md](云原生/Istio/Istio安装教程.md) 

   [Istio UI.md](云原生/Istio/Istio-UI.md) 

   [Istio Hello-world 教程.md](云原生/Istio/Istio之Hello-world教程.md) 

   [Istio 错误记录.md](云原生/Istio/Istio错误记录.md) 

   

- Jaeger

   [Jaeger 入门.md](云原生/jaeger/Jaeger入门.md) 

   

- Rancher

   [Rancher安装.md](云原生/Rancher/Rancher安装.md) 

   

- Rook

   [Rook-Ceph 块储存.md](云原生/Rook/Rook-Ceph块储存.md) 

   [Rook-Ceph 对象储存.md](云原生/Rook/Rook-Ceph对象储存.md) 

   [Deph mgr 密码.md](云原生/Rook/Deph-mgr密码.md) 

   

- Telepresence

   [Telepresence安装及入门.md](云原生/Telepresence/Telepresence安装及入门.md) 

   

- gRPC

   [gRPC入门.md](云原生/gRPC/gRPC入门.md) 

    [Protobuf语法.md](云原生/gRPC/Protobuf语法.md) 

   

- Etcd

   

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

- Grafana

   [Grafana入门.md](云原生/Grafana/Grafana入门.md) 





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

- Shell

    [Shell脚本语法记录.md](Linux/Shell/Shell脚本语法记录.md) 

   [常用命令.md](Linux/Shell/常用命令.md) 

   [错误记录.md](Linux/Shell/错误记录.md) 

   [删除脚本.md](Linux/Shell/删除脚本.md) 
   
   [一次网络命令实践.md](Linux/Shell/一次网络命令实践.md) 



---



## 其他

- 计算机网络

- 设计模式

- 数据结构与算法

- 区块链

- OpenSSL

- TensorFlow



---



## 项目



 [大数据分析房价Demo.md](项目/大数据分析房价Demo.md) 

 [大数据分析知乎用户网络.md](项目/大数据分析知乎用户网络.md) 

 [Elasticsearch分析大乐透.md](项目/Elasticsearch分析大乐透.md) 

 [IDEA插件开发之 K8s-Client.md](项目/IDEA插件开发之K8s-Client.md) 











