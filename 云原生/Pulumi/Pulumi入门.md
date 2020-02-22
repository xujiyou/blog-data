# Pulumi 入门

Pulumi 是一个库，用于替代 Yaml 方式管理软件，比如 Kubernetes 需要很多 Yaml 文件，管理维护这些 Yaml 很难，并且 Yaml 没有很强的语义，为解决这一痛点，可以考虑一下 Pulumi。

这里我们只关注 Kubernetes 。

官方文档：https://www.pulumi.com/docs/get-started/kubernetes/

首先安装（macOS）

```bash
$ brew install pulumi
```

目前，Pulumi 只支持 4 种语言 - TypeScript、JavaScript、Python、C#

这里我推荐使用 TypeScript，因为 TS 原生支持 Json ，写起来方便，并且是强类型，语法提示不会出错，方便编写重构。

在使用 Pulumi 前，还要在本地配置好 `kubectl` 和 kubernetes 的配置文件（默认是 ~/.kube/config）。

创建项目：

```bash
$ mkdir quickstart && cd quickstart
$ pulumi new kubernetes-typescript
```

创建完成后，可以使用 WebStorm 打开 quickstart 文件夹来查看代码。主要看 index.ts：

```typescript
import * as k8s from "@pulumi/kubernetes";

const appLabels = { app: "nginx" };
const deployment = new k8s.apps.v1.Deployment("nginx", {
    spec: {
        selector: { matchLabels: appLabels },
        replicas: 1,
        template: {
            metadata: { labels: appLabels },
            spec: { containers: [{ name: "nginx", image: "nginx" }] }
        }
    }
});
export const name = deployment.metadata.name;
```

看看创建 Deployment 部分，其实和写 Yaml 文件差不多的，但是 TS 有很强大的语义，可以随便关联。

看完代码后，在本地使用以下命令部署 Deployment：

```bash
$ pulumi up
```

待命令执行完毕后，可以看到 Deployment 已经部署好了：

```bash
$ kubectl get Deployment 
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
nginx-axwjuyog   1/1     1            1           22h
```

