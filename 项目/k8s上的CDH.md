# k8s 上的 CDH



## 第一步，要创建一个管理端。

界面使用 Vue.js 构建，后端通过 golang 构建，要通过 k8s 的 api 部署大数据组件并实时查看大数据组件状态。

后端应该是一个 kubernetes operator，需要自定义一堆 CRD，一个 CRD 对应一个大数据组件，这个 operator 监听这些 CRD，一旦有所改变，应该重新部署组件。

管理端本身是有状态的，但是这个状态是存储在 CRD 实例中。

除了监听这些 CRD 之外。这个 operator 还应该对外提供 Rest API 服务，供前端调用。

前端发出部署或修改组件配置的请求后，后端实际上是对 CRD 实例进行创建或更改，CRD实例更改后，会自动触发大数据组件的重新部署。



#### 建立 zookeeper 镜像

自己建一个 zookeeper 镜像，版本是 3.6.1。

下载二进制包，解压，在包内执行以下命令：

```bash
$ cp conf/zoo_sample.cfg conf/zoo.cfg
```

测试单机版启动：

```bash
$ ./bin/zkServer.sh start-foreground
```

这里要在前台启动，因为在容器中，是需要在前台启动的。

Dockerfile 如下：

```dockerfile
FROM ubuntu:16.04

COPY jdk1.8.0 /opt/jdk1.8.0
COPY zookeeper /opt/zookeeper

ENV JAVA_HOME=/opt/jdk1.8.0
ENV ZOOKEEPER_HOME=/opt/zookeeper
ENV PATH=${PATH}:${JAVA_HOME}/bin:${ZOOKEEPER_HOME}/bin

ENTRYPOINT ["zkServer.sh","start-foreground"]
```

把 jdk 和 zookeeper 的二进制文件拷贝到镜像中了，这种虽然镜像大一点，但是不需要使用 `apt-get install -y` 下载东西。

build.sh ：

```bash
#!/bin/bash
#保证 docker login 已经执行过
# docker -H tcp://fueltank-1:2376 login --username=552003271@qq.com registry.cn-chengdu.aliyuncs.com
docker -H tcp://fueltank-1:2376 build -t registry.cn-chengdu.aliyuncs.com/bbd-image/drift-zookeeper:v0.0.6  ./
docker -H tcp://fueltank-1:2376 push registry.cn-chengdu.aliyuncs.com/bbd-image/drift-zookeeper:v0.0.6
```





#### 创建项目

将项目命名为 `drift`，`operator-sdk` 版本为 `v0.18.0`

```bash
$ cd $HOME/vgo/
$ operator-sdk new drift --repo=github.com/xujiyou/drift.git
```

创建期间需要科学上网，会自动下载依赖。创建的项目是用 Go Mudules 组织的，可以直接使用 Goland 打开。

添加CRD：

```bash
$ operator-sdk add api --api-version=app.drift.com/v1alpha1 --kind=ZooKeeper
```

修改 CRD 的内容：

```go
type ZooKeeperSpec struct {
	Size int32 `json:"size"`
	ClientPort int32 `json:"clientPort"`
}

type ZooKeeperStatus struct {
	Nodes []string `json:"nodes"`
}
```

使用 `operator-sdk generate k8s` 命令重新生成 CRD，但是这个版本的 `operator-sdk` 好像有 bug，不会自动修改 CRD，需要手动修改，如下：

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: zookeepers.app.drift.com
spec:
  group: app.drift.com
  names:
    kind: ZooKeeper
    listKind: ZooKeeperList
    plural: zookeepers
    singular: zookeeper
  scope: Namespaced
  versions:
  - name: v1alpha1
    schema:
      openAPIV3Schema:
        description: ZooKeeper is the Schema for the zookeepers API
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation
              of an object. Servers should convert recognized schemas to the latest
              internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this
              object represents. Servers may infer this from the endpoint the client
              submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            description: ZooKeeperSpec defines the desired state of ZooKeeper
            type: object
            properties:
              size:
                type: integer
              clientPort:
                type: integer
          status:
            description: ZooKeeperStatus defines the observed state of ZooKeeper
            type: object
            properties:
              nodes:
                type: array
                items:
                  type: string
        type: object
    served: true
    storage: true
    subresources:
      status: {}
```

之后添加 Controller：

```bash
$ operator-sdk add controller --api-version=app.drift.com/v1alpha1 --kind=ZooKeeper
```

修改代码：

```go
func (r *ReconcileZooKeeper) Reconcile(request reconcile.Request) (reconcile.Result, error) {
	reqLogger := log.WithValues("Request.Namespace", request.Namespace, "Request.Name", request.Name)
	reqLogger.Info("Reconciling ZooKeeper")

	// Fetch the ZooKeeper instance
	zooKeeperInstance := &appv1alpha1.ZooKeeper{}
	err := r.client.Get(context.TODO(), request.NamespacedName, zooKeeperInstance)
	if err != nil {
		if errors.IsNotFound(err) {
			// Request object not found, could have been deleted after reconcile request.
			// Owned objects are automatically garbage collected. For additional cleanup logic use finalizers.
			// Return and don't requeue
			return reconcile.Result{}, nil
		}
		// Error reading the object - requeue the request.
		return reconcile.Result{}, err
	}


	fmt.Printf("%+v\n", zooKeeperInstance)
	// Define a new Pod object
	deployment := newDeploymentForCR(zooKeeperInstance)

	// Set ZooKeeper instance as the owner and controller
	if err := controllerutil.SetControllerReference(zooKeeperInstance, deployment, r.scheme); err != nil {
		return reconcile.Result{}, err
	}

	// Check if this Pod already exists
	found := &appsv1.Deployment{}
	err = r.client.Get(context.TODO(), types.NamespacedName{Name: deployment.Name, Namespace: deployment.Namespace}, found)
	if err != nil && errors.IsNotFound(err) {
		reqLogger.Info("Creating a new Deployment", "Deployment.Namespace", deployment.Namespace, "Deployment.Name", deployment.Name)
		err = r.client.Create(context.TODO(), deployment)
		if err != nil {
			return reconcile.Result{}, err
		}

		return reconcile.Result{}, nil
	} else if err != nil {
		reqLogger.Error(err, "Failed to get Deployment")
		return reconcile.Result{}, err
	}

	reqLogger.Info("Updating a Deployment", "Deployment.Namespace", deployment.Namespace, "Deployment.Name", deployment.Name)
	err = r.client.Update(context.TODO(), deployment)
	if err != nil {
		reqLogger.Error(err, "Failed to update Deployment", "Deployment.Namespace", deployment.Namespace, "Deployment.Name", deployment.Name)
		return reconcile.Result{}, err
	}
	reqLogger.Info("Updated a Deployment", "Deployment.Namespace", deployment.Namespace, "Deployment.Name", deployment.Name)

	podList := &corev1.PodList{}
	listOpts := []client.ListOption{
		client.InNamespace(zooKeeperInstance.Namespace),
		client.MatchingLabels(map[string]string{
			"app": zooKeeperInstance.Name,
		}),
	}
	if err = r.client.List(context.TODO(), podList, listOpts...); err != nil {
		reqLogger.Error(err, "Failed to list pods", "ZooKeeper.Namespace", zooKeeperInstance.Namespace, "ZooKeeper.Name", zooKeeperInstance.Name)
		return reconcile.Result{}, err
	}
	podNames := getPodNames(podList.Items)

	// Update status.Nodes if needed
	if !reflect.DeepEqual(podNames, zooKeeperInstance.Status.Nodes) {
		zooKeeperInstance.Status.Nodes = podNames
		err := r.client.Status().Update(context.TODO(), zooKeeperInstance)
		if err != nil {
			reqLogger.Error(err, "Failed to update ZooKeeper status")
			return reconcile.Result{}, err
		}
		reqLogger.Info("Updated ZooKeeper status", "Pod.Namespace", deployment.Namespace, "Pod.Name", deployment.Name)
	}

	return reconcile.Result{}, nil
}

func newDeploymentForCR(cr *appv1alpha1.ZooKeeper) *appsv1.Deployment {
	labels := map[string]string{
		"app": cr.Name,
	}
	return &appsv1.Deployment{
		ObjectMeta: metav1.ObjectMeta{
			Name:      cr.Name + "-deployment",
			Namespace: cr.Namespace,
			Labels:    labels,
		},
		Spec: appsv1.DeploymentSpec{
			Replicas: &cr.Spec.Size,
			Selector: &metav1.LabelSelector{
				MatchLabels: map[string]string{
					"app": cr.Name,
				},
			},
			Template: corev1.PodTemplateSpec{
				ObjectMeta: metav1.ObjectMeta{
					Labels: map[string]string{
						"app": cr.Name,
					},
				},
				Spec: corev1.PodSpec{
					Containers: []corev1.Container{
						{
							Name: "drift-zookeeper",
							Image: "registry.cn-chengdu.aliyuncs.com/bbd-image/drift-zookeeper:v0.0.6",
							Ports: []corev1.ContainerPort{
								{
									ContainerPort: cr.Spec.ClientPort,
								},
							},
						},
					},
					ImagePullSecrets: []corev1.LocalObjectReference{
						{
							Name: "docker-secret",
						},
					},
				},
			},
		},
	}
}

func getPodNames(pods []corev1.Pod) []string {
	var podNames []string
	for _, pod := range pods {
		log.Info(pod.Name)
		podNames = append(podNames, pod.Name)
	}
	return podNames
}
```

测试：

```bash
$ kubectl apply -f deploy/crds/app.drift.com_zookeepers_crd.yaml
$ operator-sdk run local --watch-namespace=default
```

创建实例：

```bash
$ kubectl apply -f deploy/crds/app.drift.com_v1alpha1_zookeeper_cr.yaml
```

我这里测试的创建实例和修改实例都是可以的。



#### 记录全局信息的 CRD

需要一个 CRD，记录了全局信息，比如集群所在命名空间，启用了哪些组件，初始化到了哪一步了，有没有完成等信息。

```bash
$ operator-sdk add api --api-version=app.drift.com/v1alpha1 --kind=DriftInit
$ operator-sdk add controller --api-version=app.drift.com/v1alpha1 --kind=DriftInit
$ kubectl apply -f deploy/crds/app.drift.com_driftinits_crd.yaml 
```

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: driftinits.app.drift.com
spec:
  group: app.drift.com
  names:
    kind: DriftInit
    listKind: DriftInitList
    plural: driftinits
    singular: driftinit
  scope: Cluster
  versions:
  - name: v1alpha1
    schema:
      openAPIV3Schema:
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation
              of an object. Servers should convert recognized schemas to the latest
              internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this
              object represents. Servers may infer this from the endpoint the client
              submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            properties:
              active:
                format: int32
                type: integer
              complete:
                type: boolean
              components:
                items:
                  type: string
                type: array
              currentPath:
                type: string
              namespace:
                type: string
              pvc:
                additionalProperties:
                  properties:
                    storage:
                      type: string
                    storageClass:
                      type: string
                  required:
                  - storage
                  - storageClass
                  type: object
                type: object
            required:
            - active
            - complete
            - components
            - currentPath
            - namespace
            type: object
          status:
            type: object
        type: object
    served: true
    storage: true
```





## 部署

```
helm install my-drift ./drift --namespace drift --set ingress.host=drift.test.bbdops.com
```

测试：

```
kafka-topics.sh --zookeeper zookeeper-cluster-0.zookeeper-cluster-headless-service:2181/kafka --create --topic one --replication-factor 3 --partitions 3
```



## Git

项目地址在：https://github.com/xujiyou-drift





