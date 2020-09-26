# Rancher 安装

系统：六台 Centos7，fueltank1-6，内存 32 GB。用户： admin



---



## 系统准备

需要先安装 Docker，然后对 Docker 进行配置。以下每台机器都要做！

### Docker 安装

官方安装地址：https://docs.docker.com/install/linux/docker-ce/centos/

先卸载老的 docker：

```bash
$ sudo yum remove docker* containerd.io
```

安装依赖：

```bash
$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

下载安装 docker repo：

```bash
$ sudo rm /etc/yum.repos.d/docker-ce.repo
$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
$ sudo yum install docker-ce docker-ce-cli containerd.io -y
```

启动 Docker，并加入开机启动：

```bash
$ sudo systemctl enable docker
$ sudo systemctl start docker
```

自此安装完成，最好确保没有docker镜像和容器，我们需要一个全新的环境。

```bash
$ sudo docker ps
$ sudo docker images
$ sudo docker volume ls
$ sudo docker stop $(sudo docker ps -q)  # 停止全部容器
$ sudo docker rm `docker ps -a|grep Exited|awk '{print $1}'` # 删除所有停止的容器
$ # sudo docker volume rm $(sudo docker volume ls -q) #删除所有的数据卷
$ #sudo docker rmi --force $(sudo docker images -q) # 删除全部镜像,可不做
```



---



### 配置 Docker 及环境

这部分的文档：https://rancher.com/docs/rke/latest/en/os/

修改或创建 `/etc/docker/daemon.json`，加入以下字段，注意最终的文件格式是 JSON ！

```json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ],
  "group": "docker",
  "registry-mirrors": ["https://4vra6qzb.mirror.aliyuncs.com"]
}
```

然后将需要运行 docker 的用户（非root，我这里是 admin）加入到 `docker` 用户组，保证普通用户可以运行 docker 命令。

```bash
$ sudo usermod -aG docker admin
```

这里有个坑：更改用户组之后，需要命令行，才会生效！！！！

然后需要每台服务器之间配置免密登录，包括自己登录自己。

```bash
$ ssh-keygen
$ ssh-copy-id -i ~/.ssh/id_rsa.pub admin@fueltank-6
```



---



## 使用 RKE 安装 K8s

这部分的官方文档：https://rancher.com/docs/rke/latest/en/installation/

RKE是一个二进制命令，首先需要下载 RKE 的二进制包，下载地址：https://github.com/rancher/rke/releases/latest

rke 1.0.2版本坑较少，1.0.0 全是坑

选择一个最新的二进制包进行下载，下载到 fueltank1 上。

```bash
$ wget https://github-production-release-asset-2e65be.s3.amazonaws.com/108337180/2f213400-1098-11ea-8c49-220e077ae517?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20200114%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20200114T091349Z&X-Amz-Expires=300&X-Amz-Signature=0b5342e489afb01181c4b59a175e28d93dbe7a49f4276addf36045703edcbb33&X-Amz-SignedHeaders=host&actor_id=16862201&response-content-disposition=attachment%3B%20filename%3Drke_linux-amd64&response-content-type=application%2Foctet-stream
$ mv rke_linux-amd64 rke
$ chmod +x rke
$ sudo cp rke /usr/bin/rke
```

然后运行 `rke config --name cluster.yml` 生成配置，配置过程如下：

<img src="/Users/jiyouxu/Library/Application Support/typora-user-images/image-20200114181531859.png" alt="image-20200114181531859" style="zoom:50%;" />

<img src="/Users/jiyouxu/Library/Application Support/typora-user-images/image-20200114181621081.png" alt="image-20200114181621081" style="zoom:50%;" />

<img src="/Users/jiyouxu/Library/Application Support/typora-user-images/image-20200114181641999.png" alt="image-20200114181641999" style="zoom:50%;" />

配置完成后，会在当前目录生成一个 cluster.yml。

看一下 cluster.yml ，找一个镜像名字，手动拉一下镜像测试一下是否拉的通，防止镜像拉取失败，拉取失败就尴尬了。

实时证明 rancher 的服务没被墙！

然后开始部署 K8s ！开始祈祷吧！！！

```bash
$ rke up
```

第一次不成功，报错：

```
level=fatal msg="Failed to get job complete status for job rke-network-plugin-deploy-job in namespace kube-system"
```

在 github issue 里找到解决方案：https://github.com/rancher/rancher/issues/19713

Same issue , always need to run it twice to get the cluster created because the first run always fails with that error

他们说重新运行一下就行了，果然是这样！

最后的部署成功的消息如下：

```
level=fatal msg="Failed to get job complete status for job rke-network-plugin-deploy-job in namespace kube-system"
```

部署完成后，在本地会生成俩文件：

- `cluster.rkestate` 这个文件保存了全部的证书和私钥
- `kube_config_cluster.yml` 这个文件就是 k8s 集群的连接配置文件，helm 和 kubectl 都会利用这个文件，要把这个文件移动到默认位置：`~/.kube/config`

```bash
$ cp kube_config_cluster.yml ~/.kube/config
```

下一步安装 rancher



---



## 安装 Rancher

下载 helm 和 kubelet 的二进制文件，并放到环境变量中。

Kubectl: https://github.com/kubernetes/kubectl/releases

Helm: https://github.com/helm/helm/releases

helm 3 很恶心，各种报错不断，没办法，只能遇到一个错误解决一个。

还有，helm 3 删掉了tiller服务端，还要注意 helm 命令也需要加命名空间

添加 rancher 库：

```bash
$ helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
```

添加命名空间：

```bash
$ kubectl create namespace cattle-system
```

使用默认方式安装 rancher 前，需要先安装 `cert-manager` 来自动为 rancher 生成证书。



---



### cert-manager

安装 `[cert-manager](https://github.com/jetstack/cert-manager) `

这里注意一下，要安装 0.12.0 版本，0.9.1 版本坑特多。

```bash
# Install the CustomResourceDefinition resources separately
$ kubectl apply -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.12/deploy/manifests/00-crds.yaml

# Create the namespace for cert-manager
$ kubectl create namespace cert-manager

# Add the Jetstack Helm repository
$ helm repo add jetstack https://charts.jetstack.io

# Update your local Helm chart repository cache
$ helm repo update

# Install the cert-manager Helm chart
$ helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --version v0.12.0
```

过一会验证一下：

```bash
$ kubectl get pods --namespace cert-manager

NAME                                            READY   STATUS      RESTARTS   AGE
cert-manager-7cbdc48784-rpgnt                   1/1     Running     0          3m
cert-manager-webhook-5b5dd6999-kst4x            1/1     Running     0          3m
cert-manager-cainjector-3ba5cd2bcd-de332x       1/1     Running     0          3m
```

查看 helm 安装的库，记住加命名空间：

```bash
$ helm list --all-namespaces
```



---



### rancher

然后正式安装 rancher，注意，使用 rke 1.0.2 创建的集群，这里会安装 rancher 1.3.4，别安装 1.3.3 ，太坑了。

```bash
$ helm install rancher rancher-stable/rancher \
  --namespace cattle-system \
  --set hostname=fueltank.bbdops.com
```

安装完成后，验证：

```
$ kubectl get pods -n cattle-system
```



---



### 安装报错

下面记录一下安装 1.3.3 时报的错，1.3.4 可以略过。

这里出现`第一个报错`：

```
Error: Internal error occurred: failed calling webhook "issuers.admission.certmanager.k8s.io": the server could not find the requested resource
```

然后重新运行一遍，又报错：

```
Error: cannot re-use a name that is still in use
```

找解决方案：https://github.com/helm/helm/issues/4174

```bash
$ kubectl -n cattle-system get secrets
NAME                            TYPE                                  DATA   AGE
default-token-l95jz             kubernetes.io/service-account-token   3      38m
rancher-token-xwdmx             kubernetes.io/service-account-token   3      10m
sh.helm.release.v1.rancher.v1   helm.sh/release.v1                    1      10m
tls-rancher                     kubernetes.io/tls                     2      10m
```

然后挨个删除之：

```bash
[admin@fueltank-1 ~]$ kubectl -n cattle-system delete secret default-token-l95jz
secret "default-token-l95jz" deleted
[admin@fueltank-1 ~]$ kubectl -n cattle-system delete secret rancher-token-xwdmx
secret "rancher-token-xwdmx" deleted
[admin@fueltank-1 ~]$ kubectl -n cattle-system delete secret sh.helm.release.v1.rancher.v1
secret "sh.helm.release.v1.rancher.v1" deleted
[admin@fueltank-1 ~]$ kubectl -n cattle-system delete secret tls-rancher
secret "tls-rancher" deleted
```

再运行上面的 helm install，又报错：

```
Error: rendered manifests contain a resource that already exists. Unable to continue with install: existing resource conflict: kind: ServiceAccount, namespace: cattle-system, name: rancher
```

解决：

```
$ kubectl -n cattle-system get ServiceAccount
$ kubectl -n cattle-system delete ServiceAccount rancher
```

再运行上面的 helm install，又报错：

```
Error: rendered manifests contain a resource that already exists. Unable to continue with install: existing resource conflict: kind: ClusterRoleBinding, namespace: , name: rancher
```

解决：

```
$ kubectl -n cattle-system delete ClusterRoleBinding rancher
```

好了，到此为止，再运行 上边的 helm install ，又会出现第一个错误。

用的 helm 3.0.2，更新了28天了，这东西真蠢的可以，不会自动删除重置，非要手动删！垃圾

中间气的删命名空间，命名空间变成 `Terminating` 状态了，删不掉，这里给一下解决方法：

```
$ kubectl edit namespace cattle-system
```

然后在 vim 里边，把 finalizers 数组里的项全部删除，保存，命名空间就自动删除了。

但有时候这种方式也删不掉，可以使用这个解决方案：教程：https://stackoverflow.com/questions/52369247/namespace-stuck-as-terminating-how-do-i-remove-it

```bash
$ NAMESPACE=your-rogue-namespace
$ kubectl proxy &
$ kubectl get namespace $NAMESPACE -o json |jq '.spec = {"finalizers":[]}' >temp.json
$ curl -k -H "Content-Type: application/json" -X PUT --data-binary @temp.json 127.0.0.1:8001/api/v1/namespaces/$NAMESPACE/finalize
```



---



### 错误解决

下面来着重解决第一个错误：

```
Error: Internal error occurred: failed calling webhook "issuers.admission.certmanager.k8s.io": the server could not find the requested resource
```

报错说找不到这条资源，估计是 `cert-manager` 安装有误。

刚开始用的 0.9.1 ，现在切换 cert-manager 的版本为 `0.12.0` 。

切换方式：暴力删除命名空间：`cert-manager` 。

然后重装，期间遇到各种 helm 的资源已存在的错误，根据提示删除即可。这里 helm 的处理方式真恶心，必须手动删除！

cert-manager 版本切换到 `0.12.0` 后，再安装 rancher，一切就顺利了！

![image-20200115115801053](/Users/jiyouxu/Library/Application Support/typora-user-images/image-20200115115801053.png)

访问需要等一会。

等了好久也没等出来UI界面，一直loading。。。。

估计是POD之间网络不通，进去看看

```bash
$ kubectl -n ingress-nginx exec -it nginx-ingress-controller-z4vgh -- /bin/bash
```

错误是 499，ingress-nginx 访问其他主机上的 Pod 很慢，访问自己主机上的Pod的到挺快。

直接在主机上使用 Pod 的地址访问，也很快，一旦访问其他主机的 Pod 就不行了，等好长时间。

下面又从网上找到的几个命令，访问rancher的，记录下。

The initial account is `admin` / `admin`

```bash
# Login token good for 1 minute
LOGINTOKEN=`curl -k -s 'https://10.42.222.131/v3-public/localProviders/local?action=login' -H 'content-type: application/json' --data-binary '{"username":"admin","password":"admin","ttl":60000}' | jq -r .token`


# Create API key good forever
APIKEY=`curl -k -s 'https://10.42.222.131/v3/token' -H 'Content-Type: application/json' -H "Authorization: Bearer $LOGINTOKEN" --data-binary '{"type":"token","description":"for scripts and stuff"}' | jq -r .token`
echo "API Key: ${APIKEY}"

# Set server-url
curl -k -s 'https://10.42.222.131/v3/settings/server-url' -H 'Content-Type: application/json' -H "Authorization: Bearer $APIKEY" -X PUT --data-binary '{"name":"server-url","value":"https://your-rancher.com/"}'

```



---



## 最终解决

解决办法就是升级版本。。。

rke 使用 1.0.2，rancher 使用 2.3.4，helm 还是用的 3.0.2，kubelet 默认安装的是 1.17.0，cert-manager 使用的 0.12.0

全部删掉重装：

```bash
$ rke remove
$ rm ~/.kube/config
```

然后每台 node 都执行：

```bash
$ sudo docker stop $(sudo docker ps -q)  # 停止全部容器
```

OK，再下载新的 rke，再执行：

```bash
$ rke config
```

然后进入命令行配置阶段，注意，网络插件选 `flannel`，上一次选的 calico，难用 ，flannel 好用。

然后运行上面的安装 k8s 和 cert-manager 的步骤就行了。`rke up` 要运行两遍。

用新版本一个错误也不会遇到！Pod 之间访问也很快，不论是同主机还是不同主机之间的Pod，kubectl 命令执行也很快。

然后一切安装好之后，查看：

```bash
$ kubectl get pods --all-namespaces
```

没问题之后，把 `fueltank.bbdops.com` 加入到本地的 hosts 中：

```
10.28.109.5 fueltank-1 fueltank-1.cloud.bbdops.com fueltank.bbdops.com
```

`fueltank.bbdops.com` 可以映射到任意一台主机，因为每台主机上都运行有 `ingress-nginx` 。

还要注意的是，在每台 node 上，都要在 hosts 中把 fueltank.bbdops.com 映射好。

然后就可以在浏览器里访问 `https://fueltank.bbdops.com` 了，初始用户名和密码是 admin/admin，第一次进入界面会提示更改密码，我已经改好密码了。

这里注意一下，如果使用 curl 访问 rancher 服务，会返回一堆 json，这时看到的是 rancher 的 api 服务，当用浏览器访问时，看到的是 web 界面。



## 错误记录

某些容器一直处于 `ContainerCreating` 状态，这种情况下可以使用 describe 命令：

```bash
$ kubectl -n kube-system describe pod nginx-pod
```

可以看到以下错误

```
  Warning  FailedCreatePodSandBox  16m                   kubelet, fueltank-6  Failed to create pod sandbox: rpc error: code = Unknown desc = failed to set up sandbox container "afaa79e58645e17a787350d8899d7220c35d857c8aaf034487959b88f4b90bb9" network for pod "coredns-7c5566588d-w7657": networkPlugin cni failed to set up pod "coredns-7c5566588d-w7657_kube-system" network: failed to set bridge addr: "cni0" already has an IP address different from 10.42.4.1/24
```

解决方案：https://kubernetes.feisky.xyz/v/en/index/pod

```bash
$ sudo ip link set cni0 down
$ sudo brctl delbr cni0   
```

但报错说：

```
bridge cni0 is still up; can't delete it
```

再解决：https://unix.stackexchange.com/questions/62751/cannot-delete-bridge-bridge-br0-is-still-up-cant-delete-it

```bash
$ sudo ifconfig cni0 down
$ sudo brctl delbr cni0
```



### rancher agant运行错误

具体错误：

```
ERROR: https://fueltank.bbdops.com/ping is not accessible (The requested URL returned error: 404 Not Found)
```

解决：

```bash
# 配置 cattle-cluster-agent
kubectl -n cattle-system patch  deployments cattle-cluster-agent --patch '{
    "spec": {
        "template": {
            "spec": {
                "hostAliases": [
                    {
                        "hostnames":
                        [
                            "rancher.s1.com"
                        ],
                            "ip": "192.168.6.121"
                    }
                ]
            }
        }
    }
}'
# 配置 cattle-node-agent
kubectl -n cattle-system patch  daemonsets cattle-node-agent --patch '{
 "spec": {
     "template": {
         "spec": {
             "hostAliases": [
                 {
                     "hostnames":
                        [
                            "rancher.s1.com"
                        ],
                            "ip": "192.168.6.121"
                 }
             ]
         }
     }
 }
}'
```

