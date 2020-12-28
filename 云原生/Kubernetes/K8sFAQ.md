# K8S FAQ

执行 `kubelet get nodes`时出现错误：

```
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

或者：

```
error: the server doesn't have a resource type "nodes"
```

这时需要执行以下命令：

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

这些命令用来生成一个默认的 kubeconfig 配置文件，文件内容如下：

````yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN5RENDQWJDZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRFNU1Ea3dNekEzTXpZMU9Wb1hEVEk1TURnek1UQTNNelkxT1Zvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTjVUCkZ0VmpDekpxaUowazNkOWRnVlpVL1JERnZkeldzN25obnE1dHY1QXFFS2NxK0lKdVdpYWU2MmtzQjErUnozUzcKb1NwTWlZdDlYc21oM3MwdnZ6YVlJNy9FY0dCZmpHYlNGaFkwVEZqajR6aEFZR0ozLzZkd0RmWTkwUUMxSzBMSQpWaUYxeEwyNzk4L0ROZkVGM2dvWUczaVhTZDhIa2JxaHMya2RCbDhESHdEa1NsamRBdGNTNzd4Ui9YVG16Q0c3ClRwYy8ydC9SV2NON3orZU13ZW0wbWptR1hvbGVKRzJndjlFT1huejJnTWlSSGFsV3phbHZzQ0tzaG1BSm5TUXEKOXN5b1dibW9kc2Y2MDc3MUNCN0p6WHlnVnlQK3U1ckJmNUhwQmU1U3hmWjUyOG1ramhhNWoyR0IzQXMyTHRsMgp1b0tuMDhqTko5d3htcHlXbTZNQ0F3RUFBYU1qTUNFd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFKK1JJS3lnTjJtTTVkKzAvVDlPK2hxcHQ0OGkKZ09GK3EzSlRlQWtwQUxvYm5JSXIvZEc5WTJrL244cFdlNU1JdjhRY2FxTTRyS29Fd0FSYm8wejJFYmF4L3NsawpoMDVVbXgvT3I5WGd0TjRCNWZiaHgrT0tmYUxGd0FPUGdWMWZ2WW5HUy9IVHp1T0xUc3QxMm8ySEZwYzlpNlEwCkJyNWVEWnBXdU02eVpXRmJ6RGM3M2ZiNmY0VGdERUhlZ1k0YWZ1VzAvSzZ1bytaREx0RkcyRSt2TTdUdHpnZSsKdTRrNHlnWkF2bWVENk1Ed1JOYzM0eEV6MDVZd1hweTZHbDVCcmpoclhDa0JoaEpXNVdMOU8vS0JUR2xJTGJMbApLSHVsWkxUKzFONTNRSVhQcU5YcWhaQnkrMy96NnQxT1d6YnMzdXFyejluN3IxTy9JTStVNWxycGxKOD0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    server: https://172.20.20.229:8443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM4akNDQWRxZ0F3SUJBZ0lJRHVHbWpEdEduNmN3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB4T1RBNU1ETXdOek0yTlRsYUZ3MHlNREE1TVRBd056TTBNekZhTURReApGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1Sa3dGd1lEVlFRREV4QnJkV0psY201bGRHVnpMV0ZrCmJXbHVNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQXlaMFovcGFjZVAwSnI4VUUKa21XcWIrR25hclplaTAveUlJeFQ0b1RNNkdsNjdvUkdDZlpURGxEOWxFWURhbFVZdm5pZjRBMVJmOTJ6Y3VidgppMlk1b1oyZ29PWXFzN3YvT1VsVEdyQ1dBQVJubnIzVVVvaWNueUQyUnhYeVEzMFBWSWlyeEdKemt5d2xPL0JvClQzTzkrUDlJRkpPZzZCMk5hdlBlWGtOZVBpeFZkNTE3UUs1a21Wc0Q0TmhicVF5cmJZc1pFVE1vbmd1N1dTN3QKVnRSR0xQRU91cFYrZ09qNWtwemdZL2VQOExjMTZEbzRSSTVqMGJ4OEdRejlzR2x4N1lTTVRISDBhb2J0SmtTegpFSUE4bUhPeCtiQ2gzelhLaWRtVUg5UnRnWmwrSTVWeGNpaUdIVi9LS0F6azIvTzBwNDRid3BZRktOM0w5WW84CjN4TjFzd0lEQVFBQm95Y3dKVEFPQmdOVkhROEJBZjhFQkFNQ0JhQXdFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUgKQXdJd0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFOSU9Eb3JvOXBtYThOQ25ab3JvUXg1ZzRxd1F5ZTgvSVQvbApBV25ERktiMFo5VDVKbTZ1UU1JTXlYWW51QjBkRWNjVjRHTXhOaHI0WlVtSk9kS1k0WE5UbllwaVpEcmNVbjVFClVrYWV6em1SZGtjYXBkbWtVSGxLa0IrN1h3ZGl1THBQcUk4T3VXVGJsS3BoSWZ6M3RyTi9xYngzS1RpNEhHWkcKWjI3SWVrNnZZdDJwb1FpeitpNW5FaVpYRFIzdjc1bDA2SFQ3OUVNNnZMdnBlakRwblBIZElkRlJBVWdJSUd4UwppNU1XdE1GZktpcE1oaWFGK2R2Zmp2bmsyMWx2TzlJMm1rOVpvcGpkWXkzY1ZzZzd3UFZ0TEVEeGFHajlBMHBuClRGM2NNcGppUXQzcEtaM2VoUXNINm42REJCSFJvY3d0UG9sYVYxWlZyZ29RcWY0SDcrTT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcEFJQkFBS0NBUUVBeVowWi9wYWNlUDBKcjhVRWttV3FiK0duYXJaZWkwL3lJSXhUNG9UTTZHbDY3b1JHCkNmWlREbEQ5bEVZRGFsVVl2bmlmNEExUmY5MnpjdWJ2aTJZNW9aMmdvT1lxczd2L09VbFRHckNXQUFSbm5yM1UKVW9pY255RDJSeFh5UTMwUFZJaXJ4R0p6a3l3bE8vQm9UM085K1A5SUZKT2c2QjJOYXZQZVhrTmVQaXhWZDUxNwpRSzVrbVZzRDROaGJxUXlyYllzWkVUTW9uZ3U3V1M3dFZ0UkdMUEVPdXBWK2dPajVrcHpnWS9lUDhMYzE2RG80ClJJNWowYng4R1F6OXNHbHg3WVNNVEhIMGFvYnRKa1N6RUlBOG1IT3grYkNoM3pYS2lkbVVIOVJ0Z1psK0k1VngKY2lpR0hWL0tLQXprMi9PMHA0NGJ3cFlGS04zTDlZbzgzeE4xc3dJREFRQUJBb0lCQUNBZ1M0TVk1c2dVc2hWegpGSDVyZXRRbkpmRklMQnFRMjZrNkV5ZldOM3lWU2tSMWlWK1BBNzhWUXNMOHdSQ1JqTWJWRzh5czhwNm9haTdXCkE3ZTN5MWtvYis4VG5oeFR5YUNNUVpUUUxLYkdET2pyb01paUFpc05LcEU3T3dac0NDUlZQdUdsT250cUhtakYKcnlseDdRU2ZVUklPVUNhTWh2dFM3czBnZVFUNDdMcXZiZitWWS9CdkZkbitvU1RFTVlLUXp1VWE5VnRMc2tNVwpYRC9RM251R0x2QjRDcktiSW5FalJ6Z2JZSElKMG1oSDhrZ2t0WlZvcEJzU3hqZkhUMUMvejRGNEwyR09QVktVCk1VeEM1d04rV2VWUC9pUHZQcFBCZTcvZ3c3OWRWTXcxaE9HTWFHRHk3RkdzL01mYVlZamZOd0JOZnFmcG5USjkKa3kxYk1ua0NnWUVBekdNTFpRajBFZERkWE9RNjVmTFpJek1sT1RXUkRiSXpaYmthMGc2L2hvK1hqWi9QdDhXMgpsTy9DQ0F3Z1JrSnM2WGoxUEtnZ3dWRFUwdW41c2g1bWNDak9nS1JTOWRKZDVlUFRYMVZYS21Ec0NqSzdmYnh5CmRMNFNIcUwwNHBxcVk5VVlIbmZ2aCs5Q2dyN1JSaXN4Vk13OFNJN2FQdktXamlxU0M0TGF0czBDZ1lFQS9JYkgKRWNGOHRQRVBtaERnV3U3RmUxMDVmaVdyVDZVTUdDeUhWcmxJcjdoTWVkdGhOelhSOUNpN212L2xBSXVZcUxQeQorRG5WUXlRZEkwdVdqNGpCekJzU0ticURvTldtbDVwRGliTXArTEFualFRZmNQYnNLYmhEdFVuNE1EaTEreWtaCkR6NVhESU1ZM0I5OHQ4djNSQ1pRYmtvbTAvcWF3Z3NnMkdCQjNuOENnWUVBd0NzRHhLeEZaeGJkZXdCdnpGS28KREN1RGZTVzdTNGhZUVBWb25VWVdsL3NjZ0tGWTJTNEJQRW10UW9tOE1yTXoyZFRMcDR0Z3VNSTZTRkNMWUFpcgpRaHRzQlpIN0dudi9veTJ4U0hwaDZVdVZ3d1R1T2d0Y0JoM0x4WmhyN1QrRW96YnhaWHhZNzVOckVxazg5TitaCmsyUXY3Znk2Z09MdjRaMXZFWG1vRUtVQ2dZRUFwNmI3UndpRU9NVEtMT2tEYXB1WE5KM2g2NlFxcGdmWGpiMFMKWlR0QnpKZTQvalh0eHUvT1lpRWczSGtEbW1jVGhQMWpVL1ZoWnQvMUVGZkFyNjZGcTNKVmpxcXJkUDRqU2djNgp5NUxOVExQMnJpS01sVHo1OFlES1F2UEcrSXpPRk45bUtiNmpvRVR4SGtNeFUvendQcWlKUVkrdFU1TFBhQUJuCllBQVgreFVDZ1lCa1NtK3hnZi9XZVVINXhhdXErdEVrTms0SHlOM29sNXA1UEJLZ3FOSkswSUd6OUZwNDdKQzQKQWdRNXlLQ0UyWE1xZHE3bjIvSkpEOG1UOFFrekVORlhDUGlTQ1FPUWd0NlN4bFM5M2x3VHRmZldvTUk4QmRXTgp3cXNxY3JVK1pYWWd2UTdqRDlWUU1DemJyQ0I1WjRMN3ZoOTcvN3ZNTExTVEtJamg0dXFJNEE9PQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=
````



## Kubectl 自动补全

CentOS:

```bash
$ yum install bash-completion -y
$ source /usr/share/bash-completion/bash_completion
$ source <(kubectl completion bash)
$ echo "source <(kubectl completion bash)" >> ~/.bashrc
```

MacOS:

```
$ brew install bash-completion
$ kubectl completion bash > $(brew --prefix)/etc/bash_completion.d/kubectl
```

调用brew info命令，根据提示将指令添加到~/.bash_profile中：

```
10:31:07-jiyouxu@XuJiyou:~$ brew info bash-completion
bash-completion: stable 1.3 (bottled)
Programmable completion for Bash 3.2
https://salsa.debian.org/debian/bash-completion
Conflicts with:
  bash-completion@2 (because Differing version of same formula)
/usr/local/Cellar/bash-completion/1.3_3 (189 files, 607.9KB) *
  Poured from bottle on 2020-02-03 at 10:28:27
From: https://github.com/Homebrew/homebrew-core/blob/master/Formula/bash-completion.rb
==> Caveats
Add the following line to your ~/.bash_profile:
  [[ -r "/usr/local/etc/profile.d/bash_completion.sh" ]] && . "/usr/local/etc/profile.d/bash_completion.sh"

Bash completion has been installed to:
  /usr/local/etc/bash_completion.d
==> Analytics
install: 13,385 (30 days), 39,298 (90 days), 178,277 (365 days)
install-on-request: 12,452 (30 days), 36,576 (90 days), 163,668 (365 days)
build-error: 0 (30 days)
```

vim  ~/.bash_profile:

```
 [[ -r "/usr/local/etc/profile.d/bash_completion.sh" ]] && . "/usr/local/etc/profile.d/bash_completion.sh"
```



## kubectl exec 选择容器

当 Pod 中有多个容器时，可以使用下面的命令进入Pod 中的某一个容器：

参考：https://kubernetes.io/zh/docs/tasks/debug-application-cluster/get-shell-running-container/

```bash
$ kubectl exec -i -t prometheus-server-6f7c8bbb88-xzrr8 --container prometheus-server  /bin/sh -n prometheus
```



## 强制删除

```
$ kubectl delete pod kube-flannel-ds-amd64-ndlbs -n kube-system --force --grace-period=0
```



## Pod内权限错误

查看 pod 日志，出现错误：

```
mkdir: cannot create directory '/var/lib/zookeeper/data': Permission denied
```

在 pod 定义中加入：

```
securityContext:
  runAsUser: 1000
  fsGroup: 1000
```



## kubectl api-resources 出错

在执行`kubectl api-resources` 时报错：

```
error: unable to retrieve the complete list of server APIs: packages.operators.coreos.com/v1: the server is currently unable to handle the request
```

这是因为之前删东西没删干净造成的。

可以这样解决：

```
$ kubectl get apiservice
```

找出 AVAILABLE 为 false 的项





#### kubelet 目录数据删不掉

清理 kubelet 时有几个文件夹删不掉，是因为这几个目录被挂载了卷，使用 `df -h` 查看，卸载掉挂载的卷即可



## 一个 k8s 的 DNS 解析问题

一个有意思的问题。在每个 pod 中的 /etc/resolv.conf 中的内容如下：

```
nameserver 10.43.0.10
search default.svc.cluster.local svc.cluster.local cluster.local test.bbdops.com
options ndots:5
```

这个配置文件中 `options ndots:5` 的意思是：如果查询的域名中点号的数量小于 5 ，就会依次在域名后面带上 `default.svc.cluster.local`、`svc.cluster.local`、`cluster.local`、`test.bbdops.com` 来查询。

其中，`test.bbdops.com` 是宿主机的域名，可通过 `hostname -d` 查询宿主机的域名。

但偏偏在内网的 DNS 解析中，为 `test.bbdops.com`配置了泛域名解析，所以查询一般的公网域名时，其中的点号数量都小于 5，所以都被泛域名解析了。

解决方案：

1. 去掉DNS服务器的泛域名解析

2. 修改宿主机的 domain，然后重启宿主机。

3. Pod 中修改 DNS 配置，参考：https://kubernetes.io/zh/docs/concepts/services-networking/dns-pod-service/#pod-dns-config

   ```yaml
     dnsConfig:
       options:
         - name: ndots
           value: "3"
   ```

   



## 修改时区

Dockerfile 中加入以下命令可以让容器时区和宿主机时区同步：

```dockerfile
RUN rm -f /etc/localtime \
    && ln -sv /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo "Asia/Shanghai" > /etc/timezone
```

同时，部署文件要这么写：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: s1-system
  namespace: s1-test
  labels:
    app: s1-system
    version: v1.0
spec:
  ports:
    - port: 8891
      name: http
  selector:
    app: s1-system
    version: v1.0

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: s1-system
  namespace: s1-test
  labels:
    app: s1-system
    version: v1.0
spec:
  replicas: 3
  selector:
    matchLabels:
      app: s1-system
      version: v1.0
  template:
    metadata:
      labels:
        app: s1-system
        version: v1.0
    spec:
      imagePullSecrets:
        - name: aliyun-secret
      containers:
        - name: s1-system
          image: registry.cn-shanghai.aliyuncs.com/bbd-s1/s1-system:1.7
          ports:
            - containerPort: 8891
          env:
            - name: TZ
              value: Asia/Shanghai
          volumeMounts:
          - name: host-time
            mountPath: /etc/localtime
            readOnly: true
      volumes:
      - name: host-time
        hostPath:
          path: /etc/localtime
```













