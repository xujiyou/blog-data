# 使用 CronJob 运行自动化任务

文档：https://kubernetes.io/zh/docs/tasks/job/automated-tasks-with-cron-jobs/

可以利用 [CronJobs](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs) 执行基于时间调度的任务。这些自动化任务和 Linux 或者 Unix 系统的 [Cron](https://en.wikipedia.org/wiki/Cron) 任务类似。

CronJobs 在创建周期性以及重复性的任务时很有帮助，例如执行备份操作或者发送邮件。CronJobs 也可以在特定时间调度单个任务，例如你想调度低活跃周期的任务。

```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

运行：

```
$ kubectl create -f ./cronjob.yaml
$ kubectl get cronjob hello
$ kubectl get jobs --watch
```

现在你已经看到了一个运行中的任务被 “hello” CronJob 调度。你可以停止监视这个任务，然后再次查看 CronJob 就能看到它调度任务：

```shell
$ kubectl get cronjob hello
NAME      SCHEDULE      SUSPEND   ACTIVE    LAST-SCHEDULE
hello     */1 * * * *   False     0         Mon, 29 Aug 2016 14:34:00 -0700
```

你应该能看到 “hello” CronJob 在 `LAST-SCHEDULE` 声明的时间点成功的调度了一次任务。有0个活跃的任务意味着任务执行完毕或者执行失败。

现在，找到最后一次调度任务创建的 Pod 并查看一个 Pod 的标准输出。请注意任务名称和 Pod 名称是不同的。

没次任务都会创建一个 POD ，所以这里的 POD 只有一条日志。

```shell
# Replace "hello-4111706356" with the job name in your system
$ pods=$(kubectl get pods --selector=job-name=hello-4111706356 --output=jsonpath={.items..metadata.name})

$ echo $pods
hello-4111706356-o9qcm

$ kubectl logs $pods
Mon Aug 29 21:34:09 UTC 2016
Hello from the Kubernetes cluster
```

当你不再需要 CronJob 时，可以用 `kubectl delete cronjob` 删掉它：

```shell
$ kubectl delete cronjob hello
cronjob "hello" deleted
```

