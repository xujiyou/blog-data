# 日常工作技术记录

分别包括 `编程语言`，`大数据`，`前端`，`数据储存`，`云原生`，`DevOps`，`Java`，`Linux` 这几个方面，另外还有其他的一些编程基础。



---



## 云原生

云原生是最近兴起的开源基金会，包含许多高质量的开源项目，大多依托于 Docker 及 Kubernetes。这些开源项目大多是 Golang 写的。

- Kubernetes

   [Kubernetes二进制安装.md](云原生/Kubernetes/Kubernetes二进制安装.md) 

   [kubeadm安装集群.md](云原生/Kubernetes/kubeadm安装集群.md) 

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

      [配置踩坑笔记.md](云原生/Kubernetes/Kubernetes配置系列/配置踩坑笔记.md) 

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
      
      - Operator

         [Kubernetes-Operator教程.md](云原生/Kubernetes/Kubernetes扩展机制/Operator/Kubernetes-Operator教程.md) 

         [operator-sdk教程.md](云原生/Kubernetes/Kubernetes扩展机制/Operator/operator-sdk教程.md) 

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

   - Kubernetes 认证及授权系列

     [Kubernetes证书生成.md](云原生/Kubernetes/Kubernetes认证及授权系列/Kubernetes证书生成.md) 

     [为Kubernetes搭建支持OpenId-Connect的身份认证系统.md](云原生/Kubernetes/Kubernetes认证及授权系列/为Kubernetes搭建支持OpenId-Connect的身份认证系统.md) 

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

   [Docker储存驱动.md](云原生/Docker/Docker储存驱动.md) 

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
  
  [containerd配置.md](云原生/containerd/containerd配置.md) 



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

   [istio安装配置.md](云原生/Istio/istio安装配置.md) 

   [Istio监控措施.md](云原生/Istio/Istio监控措施.md) 

   [istioctl使用方法.md](云原生/Istio/istioctl使用方法.md) 

   [动手构建一个adapter.md](云原生/Istio/动手构建一个adapter.md) 

   - 实践

      [Istio实现灰度发布.md](云原生/Istio/实践/Istio实现灰度发布.md) 
   
      [Istio流量治理.md](云原生/Istio/实践/Istio流量治理.md) 
      
      [Istio服务保护.md](云原生/Istio/实践/Istio服务保护.md) 
   
   - CRD

      [Istio-crd(一)-概述.md](云原生/Istio/CRD/Istio-crd(一)-概述.md) 

      [Istio-crd(二)-authentication.md](云原生/Istio/CRD/Istio-crd(二)-authentication.md) 

      [Istio-crd(三)-config.md](云原生/Istio/CRD/Istio-crd(三)-config.md) 

      [Istio-crd(四)-networking.md](云原生/Istio/CRD/Istio-crd(四)-networking.md) 

      [Istio-crd(五)-rbac.md](云原生/Istio/CRD/Istio-crd(五)-rbac.md) 

      [Istio-crd(六)-security.md](云原生/Istio/CRD/Istio-crd(六)-security.md) 

   - 架构
   
      [Poilt.md](云原生/Istio/架构/Poilt.md) 
   
   - 配置



- Envoy

  [Envoy入门.md](云原生/Envoy/Envoy入门.md) 
  
  [Envoy术语.md](云原生/Envoy/Envoy术语.md) 
  
  [Envoy代理自己的服务.md](云原生/Envoy/Envoy代理自己的服务.md) 
  
  [Envoy-xDS实现动态配置.md](云原生/Envoy/Envoy-xDS实现动态配置.md) 
  
  [Envoy二进制构建及安装.md](云原生/Envoy/Envoy二进制构建及安装.md) 
  
  [Envoy配置文件详解.md](云原生/Envoy/Envoy配置文件详解.md) 
  
  [Envoy命令行选项.md](云原生/Envoy/Envoy命令行选项.md) 
  
  - Envoy - API
  
    [Envoy-API(一)-概述.md](云原生/Envoy/Envoy-API/Envoy-API(一)-概述.md) 
  
    [Envoy-API(二)-Boorstrap.md](云原生/Envoy/Envoy-API/Envoy-API(二)-Boorstrap.md) 
  
    [Envoy-API(三)-Listeners.md](云原生/Envoy/Envoy-API/Envoy-API(三)-Listeners.md) 
  
  - Envoy 安全
  
    [Envoy-TLS学习.md](云原生/Envoy/Envoy安全/Envoy-TLS学习.md) 
  
  - xDS
  
    [Envoy-xDS实现动态配置.md](云原生/Envoy/xDS/Envoy-xDS实现动态配置.md) 
  
    [EDS.md](云原生/Envoy/xDS/EDS.md)  
  
    [RDS.md](云原生/Envoy/xDS/RDS.md) 
  
    [RTDS.md](云原生/Envoy/xDS/RTDS.md) 
  
    [SDS.md](云原生/Envoy/xDS/SDS.md) 
  
    [VHDS.md](云原生/Envoy/xDS/VHDS.md) 
  
  - Envoy 配置
  
    - HTTP
  
      - HTTP 过滤器 
  
         [错误注入.md](云原生/Envoy/Envoy配置/HTTP/HTTP过滤器/错误注入.md) 
  
         [外部认证.md](云原生/Envoy/Envoy配置/HTTP/HTTP过滤器/外部认证.md) 
  
         [自适应并发过滤器.md](云原生/Envoy/Envoy配置/HTTP/HTTP过滤器/自适应并发过滤器.md) 
  
         [Buffer.md](云原生/Envoy/Envoy配置/HTTP/HTTP过滤器/Buffer.md) 
  
         [CORS.md](云原生/Envoy/Envoy配置/HTTP/HTTP过滤器/CORS.md) 
  
         [gRPC到HTTP1.1桥接.md](云原生/Envoy/Envoy配置/HTTP/HTTP过滤器/gRPC到HTTP1.1桥接.md) 
  
         [Gzip.md](云原生/Envoy/Envoy配置/HTTP/HTTP过滤器/Gzip.md) 
  
         [Header-To-Metadata.md](云原生/Envoy/Envoy配置/HTTP/HTTP过滤器/Header-To-Metadata.md) 
  
         [IP-Tagging.md](云原生/Envoy/Envoy配置/HTTP/HTTP过滤器/IP-Tagging.md) 
  
         [JWT.md](云原生/Envoy/Envoy配置/HTTP/HTTP过滤器/JWT.md) 
  
         [lua.md](云原生/Envoy/Envoy配置/HTTP/HTTP过滤器/lua.md) 
  
         [tap.md](云原生/Envoy/Envoy配置/HTTP/HTTP过滤器/tap.md) 
  
      - HTTP 连接管理
  
         [流量转移与分流.md](云原生/Envoy/Envoy配置/HTTP/HTTP连接管理/流量转移与分流.md) 
  
         [路由匹配.md](云原生/Envoy/Envoy配置/HTTP/HTTP连接管理/路由匹配.md) 
  
         [Header处理.md](云原生/Envoy/Envoy配置/HTTP/HTTP连接管理/Header处理.md) 
  
         [Header清理.md](云原生/Envoy/Envoy配置/HTTP/HTTP连接管理/Header清理.md) 
  
         [HTTP-Runtime.md](云原生/Envoy/Envoy配置/HTTP/HTTP连接管理/HTTP-Runtime.md) 
  
         [HTTP统计信息.md](云原生/Envoy/Envoy配置/HTTP/HTTP连接管理/HTTP统计信息.md) 
  
    - Listeners
  
       [http_inspector.md](云原生/Envoy/Envoy配置/Listeners/Listener-Filters/http_inspector.md) 
  
       [tls_inspector.md](云原生/Envoy/Envoy配置/Listeners/Listener-Filters/tls_inspector.md) 
  
       [Direct-response.md](云原生/Envoy/Envoy配置/Listeners/Network-Filters/Direct-response.md) 
  
       [echo.md](云原生/Envoy/Envoy配置/Listeners/Network-Filters/echo.md) 
  
       [kafka.md](云原生/Envoy/Envoy配置/Listeners/Network-Filters/kafka.md) 
  
       [mongo.md](云原生/Envoy/Envoy配置/Listeners/Network-Filters/mongo.md) 
  
       [mysql.md](云原生/Envoy/Envoy配置/Listeners/Network-Filters/mysql.md) 
  
       [redis.md](云原生/Envoy/Envoy配置/Listeners/Network-Filters/redis.md) 
  
       [ssl-client.md](云原生/Envoy/Envoy配置/Listeners/Network-Filters/ssl-client.md) 
  
       [zookeeper.md](云原生/Envoy/Envoy配置/Listeners/Network-Filters/zookeeper.md) 
  
    - Upstream
  
       [断路器.md](云原生/Envoy/Envoy配置/Upstream/断路器.md) 
  
       [HTTP健康检查.md](云原生/Envoy/Envoy配置/Upstream/HTTP健康检查.md) 
  
       [TCP健康检查.md](云原生/Envoy/Envoy配置/Upstream/TCP健康检查.md) 
  
    
  
  



- Jaeger

   [Jaeger 入门.md](云原生/jaeger/Jaeger入门.md) 

   [Jaeger-Operator.md](云原生/jaeger/Jaeger-Operator.md) 

   

- Rancher

   [Rancher安装.md](云原生/Rancher/Rancher安装.md) 

   [Ranche重置密码.md](云原生/Rancher/Ranche重置密码.md) 

   

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

   [Vault使用示例.md](云原生/Vault/Vault使用示例.md) 

   [Vault之OIDC认证方法.md](云原生/Vault/Vault之OIDC认证方法.md) 

   [Vault-Getting-Start.md](云原生/Vault/Vault-Getting-Start.md) 

   

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
   
   [calico网络配置.md](云原生/Calico/calico网络配置.md) 
   
   [calico-cni插件配置详解.md](云原生/Calico/calico-cni插件配置详解.md) 
   
   - 资源对象
   
      [FelixConfiguration.md](云原生/Calico/资源对象/FelixConfiguration.md) 
   
      [GlobalNetworkPolicy.md](云原生/Calico/资源对象/GlobalNetworkPolicy.md) 
   
      [GlobalNetworkSet.md](云原生/Calico/资源对象/GlobalNetworkSet.md) 
   
      [HostEndpoint.md](云原生/Calico/资源对象/HostEndpoint.md) 
   
      [IPPool.md](云原生/Calico/资源对象/IPPool.md) 
   
      [Node.md](云原生/Calico/资源对象/Node.md) 
   
      [WorkloadEndpoint.md](云原生/Calico/资源对象/WorkloadEndpoint.md) 



- Argo 

  [Argo介绍及入门.md](云原生/Argo/Argo介绍及入门.md) 







---



## 大数据

CDH 开始收费了，怎么说。。。

不用 CDH 和 HDP，自己动手搭建一个高可用的大数据集群，路线如下：

1. [ZooKeeper高可用部署.md](大数据/ZooKeeper/ZooKeeper高可用部署.md) 
2. [Hadoop高可用部署.md](大数据/HDFS/Hadoop高可用部署.md) 
3. [HBase高可用部署.md](大数据/HBase/HBase高可用部署.md) 
4. [Hive部署.md](大数据/Hive/Hive部署.md) 
5. [Spark高可用部署.md](大数据/Spark/Spark高可用部署.md) 
6. [Kafka高可用部署.md](大数据/Kafka/Kafka高可用部署.md) 
7. [Flink高可用部署.md](大数据/Flink/Flink高可用部署.md) 

下面按组件来学习。

- 爬虫

   [Scrapy入门教程.md](大数据/爬虫/Scrapy入门教程.md) 

   [Splash使用教程.md](大数据/爬虫/Splash使用教程.md) 



- Flink

   [Flink高可用部署.md](大数据/Flink/Flink高可用部署.md) 

   [Flink-Table教程.md](大数据/Flink/Flink-Table教程.md) 



- Flume

   [Flume入门教程.md](大数据/Flume/Flume入门教程.md) 



- HBase

   [HBase高可用部署.md](大数据/HBase/HBase高可用部署.md) 

   [HBase入门教程.md](大数据/HBase/HBase入门教程.md) 
   
   [HBase架构.md](大数据/HBase/HBase架构.md) 
   
   - HBase配置
   
     [HBase配置概述.md](大数据/HBase/HBase配置/HBase配置概述.md) 
   
     [HBase默认配置.md](大数据/HBase/HBase配置/HBase默认配置.md) 



- HDFS

   [Hadoop高可用部署.md](大数据/HDFS/Hadoop高可用部署.md) 

   [HDFS常用操作.md](大数据/HDFS/HDFS常用操作.md) 
   
   [HDFS之web-ui.md](大数据/HDFS/HDFS之web-ui.md) 
   
   [HDFS原理详解.md](大数据/HDFS/HDFS原理详解.md) 



- Hive

   [Hive部署.md](大数据/Hive/Hive部署.md) 

   [Java连接Hive.md](大数据/Hive/Java连接Hive.md) 

   [Hive错误处理.md](大数据/Hive/Hive错误处理.md) 



- Kafka

   [Kafka高可用部署.md](大数据/Kafka/Kafka高可用部署.md) 

   [Kafka快速入门.md](大数据/Kafka/Kafka快速入门.md) 
   
   [Kafka主题.md](大数据/Kafka/Kafka主题.md) 
   
   [Kafka生产者.md](大数据/Kafka/Kafka生产者.md) 
   
   [Kafka消费者.md](大数据/Kafka/Kafka消费者.md) 



- Oozie

   [Oozie入门.md](大数据/Oozie/Oozie入门.md) 



- Spark

   [Spark高可用部署.md](大数据/Spark/Spark高可用部署.md) 

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

   [ZooKeeper高可用部署.md](大数据/ZooKeeper/ZooKeeper高可用部署.md) 

   [ZooKeeper使用入门.md](大数据/ZooKeeper/ZooKeeper使用入门.md) 

   [zkCli的使用.md](大数据/ZooKeeper/zkCli的使用.md) 
   
   [ZooKeeper之ACL认证.md](大数据/ZooKeeper/ZooKeeper之ACL认证.md) 
   
   

- Kerberos

- Sqoop



- HDP

   [HDP离线安装.md](大数据/HDP/HDP离线安装.md) 



- CDH



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

   [cephadm安装集群.md](数据存储/Ceph/cephadm安装集群.md) 

   [ceph-deploy安装集群.md](数据存储/Ceph/ceph-deploy安装集群.md) 

   - Ceph对象储存

     [用户管理.md](数据存储/Ceph/Ceph对象储存/用户管理.md) 
     
     [S3-API.md](数据存储/Ceph/Ceph对象储存/S3-API.md) 
     
   - Ceph块储存
   
     [Ceph块储存入门.md](数据存储/Ceph/Ceph块储存/Ceph块储存入门.md) 
   
     [Kubernetes使用Ceph块设备.md](数据存储/Ceph/Ceph块储存/Kubernetes使用Ceph块设备.md) 
   
     [rbd命令详解.md](数据存储/Ceph/Ceph块储存/rbd命令详解.md) 
   
     [Ceph块储存快照.md](数据存储/Ceph/Ceph块储存/Ceph块储存快照.md) 
   
     [Ceph块设备挂载.md](数据存储/Ceph/Ceph块储存/Ceph块设备挂载.md) 
   
   - Ceph文件系统
   
     [Ceph文件系统入门.md](数据存储/Ceph/Ceph文件系统/Ceph文件系统入门.md) 



- GlusterFS



- MinIO

  [MinIO单机版启动.md](数据存储/MinIO/MinIO单机版启动.md) 

  [MinIO客户端的使用.md](数据存储/MinIO/MinIO客户端的使用.md) 

  [MinIO备份定期删除脚本.md](数据存储/MinIO/MinIO备份定期删除脚本.md) 



- MongoDB

   [MongoDB高可用部署.md](数据存储/MongoDB/MongoDB高可用部署.md) 



- MySQL

   [MySQL最新版本安装.md](数据存储/MySQL/MySQL最新版本安装.md) 

   [MySQL离线环境主从搭建.md](数据存储/MySQL/MySQL离线环境主从搭建.md) 

   [MySQL-binlog介绍.md](数据存储/MySQL/MySQL-binlog介绍.md) 

   [Mysql数据备份.md](数据存储/MySQL/Mysql数据备份.md) 

   [mysql-shell的使用.md](数据存储/MySQL/mysql-shell的使用.md) 

   - MySQL 管理

     [MySQL配置.md](数据存储/MySQL/MySQL管理/MySQL配置.md) 

     [MySQL系统库.md](数据存储/MySQL/MySQL管理/MySQL系统库.md) 

     [MySQL日志.md](数据存储/MySQL/MySQL管理/MySQL日志.md) 

     [MySQL数据目录.md](数据存储/MySQL/MySQL管理/MySQL数据目录.md) 

   - InnoDB

     

   



- Redis

  [Redis入门教程.md](数据存储/Redis/Redis入门教程.md) 

  [redis-cli使用方法.md](数据存储/Redis/redis-cli使用方法.md) 
  
  [Redis配置.md](数据存储/Redis/Redis配置.md) 
  
  [Redis主从复制.md](数据存储/Redis/Redis主从复制.md) 
  
  [Redis数据持久化.md](数据存储/Redis/Redis数据持久化.md) 
  
  [Redis高可用.md](数据存储/Redis/Redis高可用.md) 
  
  [Redis权限控制.md](数据存储/Redis/Redis权限控制.md) 
  
  - Redis 使用
  
     [Redis订阅发布.md](数据存储/Redis/Redis使用/Redis订阅发布.md) 
  
     [Redis分布式锁.md](数据存储/Redis/Redis使用/Redis分布式锁.md) 
  
     [Redis键过期.md](数据存储/Redis/Redis使用/Redis键过期.md) 
  
     [Redis事务.md](数据存储/Redis/Redis使用/Redis事务.md) 
  
     [Redis键空间通知.md](数据存储/Redis/Redis使用/Redis键空间通知.md) 



- Neo4j



---



## 编程语言

挑几门有前途的学一下

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
    
    [Python源码安装.md](编程语言/Python/Python源码安装.md) 



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

  [Gitlab安装.md](DevOps/Gitlab/Gitlab安装.md) 

  [Gitlab升级.md](DevOps/Gitlab/Gitlab升级.md) 
  
  [Gitlab-CI教程.md](DevOps/Gitlab/Gitlab-CI教程.md) 



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

- Ansible

  [Ansible安装.md](DevOps/Ansible/Ansible安装.md) 

  [PlayBook学习.md](DevOps/Ansible/PlayBook学习.md) 

  [Ansible安装k8s.md](DevOps/Ansible/Ansible安装k8s.md) 

- Git

   [git命令详解.md](DevOps/Git/git命令详解.md) 
   
   [详解.git目录.md](DevOps/Git/详解.git目录.md) 



---



## Java

 [ClassNotFoundException与NoClassDefFoundError.md](Java/ClassNotFoundException与NoClassDefFoundError.md) 

 [package-info的作用.md](Java/package-info的作用.md) 

- Gradle

- Maven

- JVM



---



## Linux

 [Linux中的各种信号.md](Linux/Linux中的各种信号.md) 

 [Linux处理僵尸进程.md](Linux/Linux处理僵尸进程.md) 

 [Cgroup实践.md](Linux/Cgroup实践.md) 

 [DNS配置文件.md](Linux/DNS配置文件.md) 

 [Linux创建用户并配置免密登录.md](Linux/Linux创建用户并配置免密登录.md) 

[RAID实践.md](Linux/RAID实践.md) 

- Syslog

   [syslog-ng使用教程.md](Linux/syslog/syslog-ng使用教程.md) 

   

- Iptables

     [iptables教程.md](Linux/iptables/iptables教程.md) 



- Shell

    [Shell脚本语法记录.md](Linux/Shell/Shell脚本语法记录.md) 

   [常用命令.md](Linux/Shell/常用命令.md) 

   [错误记录.md](Linux/Shell/错误记录.md) 

   [删除脚本.md](Linux/Shell/删除脚本.md) 
   
   [一次网络命令实践.md](Linux/Shell/一次网络命令实践.md) 
   
   [内核升级.md](Linux/Shell/内核升级.md) 
   
   [iproute2命令详解.md](Linux/Shell/iproute2命令详解.md) 
   
- 命令详解

    [top命令详解.md](Linux/命令详解/top命令详解.md) 

    [lsof命令详解.md](Linux/命令详解/lsof命令详解.md) 
    
    [ps命令详解.md](Linux/命令详解/ps命令详解.md) 
    
    [netstat命令详解.md](Linux/命令详解/netstat命令详解.md) 
    
    [iostat命令详解.md](Linux/命令详解/iostat命令详解.md) 
    
    [screen命令详解.md](Linux/命令详解/screen命令详解.md) 
    
    [demsg命令.md](Linux/命令详解/demsg命令.md) 



- Systemd

  [systemd入门.md](Linux/systemd/systemd入门.md) 

  [systemctl命令详解.md](Linux/systemd/systemctl命令详解.md) 

  [journalctl命令详解.md](Linux/systemd/journalctl命令详解.md) 

  [其他命令.md](Linux/systemd/其他命令.md) 

  [service文件结构.md](Linux/systemd/service文件结构.md) 

  

- SELinux

  [SELinux入门.md](Linux/SELinux/SELinux入门.md) 



- YUM

   [yum命令详解.md](Linux/yum/yum命令详解.md) 

   [rpm命令详解.md](Linux/yum/rpm命令详解.md) 

   [repo文件格式.md](Linux/yum/repo文件格式.md) 

   [yum离线源的配置.md](Linux/yum/yum离线源的配置.md) 

   [yum配置文件.md](Linux/yum/yum配置文件.md) 

   [epel软件库的使用.md](Linux/yum/epel软件库的使用.md) 

- OpenSSL

   [openssl常用命令.md](Linux/OpenSSL/openssl常用命令.md) 

   [证书各个字段的含义.md](Linux/OpenSSL/证书各个字段的含义.md) 

- IPVS

- Keepalived

    [Keepalived入门教程.md](Linux/Keepalived/Keepalived入门教程.md) 

    [Keepalived配置.md](Linux/Keepalived/Keepalived配置.md) 
    
- HAProxy

    [HAProxy安装及入门.md](Linux/HAPorxy/HAProxy安装及入门.md) 

   





---



## 其他

- 计算机网络

- 设计模式

- 数据结构与算法

- 区块链

- TensorFlow

- 日常

   [chrome没有继续前往.md](其他/日常/chrome没有继续前往.md) 
   
   [MacOS触控板与鼠标的滚动方向.md](其他/日常/MacOS触控板与鼠标的滚动方向.md) 
   
   [Mac OS文件系统的附加属性@如何彻底删除.md](其他/日常/MacOS文件系统的附加属性@如何彻底删除.md) 
   
   [MacOS更改控制台图标大小.md](其他/日常/MacOS更改控制台图标大小.md) 



---



## 项目



 [大数据分析房价Demo.md](项目/大数据分析房价Demo.md) 

 [大数据分析知乎用户网络.md](项目/大数据分析知乎用户网络.md) 

 [Elasticsearch分析大乐透.md](项目/Elasticsearch分析大乐透.md) 

 [IDEA插件开发之 K8s-Client.md](项目/IDEA插件开发之K8s-Client.md) 











