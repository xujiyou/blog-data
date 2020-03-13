# 通过文件将Pod信息呈现给容器

文档：https://kubernetes.io/zh/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/

此页面描述Pod如何使用DownwardAPIVolumeFile把自己的信息呈现给pod中运行的容器。DownwardAPIVolumeFile可以呈现pod的字段和容器字段。

有两种方式可以将Pod和Container字段呈现给运行中的容器：

- [环境变量](https://kubernetes.io/docs/tasks/configure-pod-container/environment-variable-expose-pod-information/)
- DownwardAPIVolumeFile

这两种呈现Pod和Container字段的方式都称为*Downward API*。

[`dapi-volume-resources.yaml`](https://raw.githubusercontent.com/kubernetes/website/master/content/zh/examples/pods/inject/dapi-volume-resources.yaml)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubernetes-downwardapi-volume-example-2
spec:
  containers:
    - name: client-container
      image: busybox:1.24
      command: ["sh", "-c"]
      args:
        - while true; do
            echo -en '\n';
            if [[ -e /etc/podinfo/cpu_limit ]]; then
              echo -en '\n'; cat /etc/podinfo/cpu_limit; fi;
            if [[ -e /etc/podinfo/cpu_request ]]; then
              echo -en '\n'; cat /etc/podinfo/cpu_request; fi;
            if [[ -e /etc/podinfo/mem_limit ]]; then
              echo -en '\n'; cat /etc/podinfo/mem_limit; fi;
            if [[ -e /etc/podinfo/mem_request ]]; then
              echo -en '\n'; cat /etc/podinfo/mem_request; fi;
            sleep 5;
          done;
      resources:
        requests:
          memory: "32Mi"
          cpu: "125m"
        limits:
          memory: "64Mi"
          cpu: "250m"
      volumeMounts:
        - mountPath: /etc/podinfo
          name: podinfo
          readOnly: false
  volumes:
    - name: podinfo
      downwardAPI:
        items:
          - path: "cpu_limit"
            resourceFieldRef:
              resource: limits.cpu
              containerName: client-container
              divisor: 1m
          - path: "cpu_request"
            resourceFieldRef:
              containerName: client-container
              resource: requests.cpu
              divisor: 1m
          - path: "mem_limit"
            resourceFieldRef:
              containerName: client-container
              resource: limits.memory
              divisor: 1Mi
          - path: "mem_request"
            resourceFieldRef:
              containerName: client-container
              resource: requests.memory
              divisor: 1Mi
```

```
$ kubectl apply -f dapi-volume-resources.yaml
$ kubectl get pods
$ kubectl logs kubernetes-downwardapi-volume-example-2
$ kubectl exec -it kubernetes-downwardapi-volume-example-2 -- sh
$ ls /etc/podinfo
```

[`dapi-volume.yaml`](https://raw.githubusercontent.com/kubernetes/website/master/content/zh/examples/pods/inject/dapi-volume.yaml)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubernetes-downwardapi-volume-example
  labels:
    zone: us-est-coast
    cluster: test-cluster1
    rack: rack-22
  annotations:
    build: two
    builder: john-doe
spec:
  containers:
    - name: client-container
      image: busybox
      command: ["sh", "-c"]
      args:
        - while true; do
          if [[ -e /etc/podinfo/labels ]]; then
          echo -en '\n\n'; cat /etc/podinfo/labels; fi;
          if [[ -e /etc/podinfo/annotations ]]; then
          echo -en '\n\n'; cat /etc/podinfo/annotations; fi;
          sleep 5;
          done;
      volumeMounts:
        - name: podinfo
          mountPath: /etc/podinfo
          readOnly: false
  volumes:
    - name: podinfo
      downwardAPI:
        items:
          - path: "labels"
            fieldRef:
              fieldPath: metadata.labels
          - path: "annotations"
            fieldRef:
              fieldPath: metadata.annotations
```

```
$ kubectl exec -it kubernetes-downwardapi-volume-example-2 -- sh
```

