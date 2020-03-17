# Kubernetes API 学习（四）- Batch API

通过以下命令来查看有哪些资源对象：

```
$ kubectl api-resources --api-group=batch
NAME       SHORTNAMES   APIGROUP   NAMESPACED   KIND
cronjobs   cj           batch      true         CronJob
jobs                    batch      true         Job
```

只有两个。

源码位于 API 源码中的 batch 包。

## Job

对象结构：

```go
type Job struct {
	metav1.TypeMeta `json:",inline"`
	// Standard object's metadata.
	// More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
	// +optional
	metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

	// Specification of the desired behavior of a job.
	// More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
	// +optional
	Spec JobSpec `json:"spec,omitempty" protobuf:"bytes,2,opt,name=spec"`

	// Current status of a job.
	// More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
	// +optional
	Status JobStatus `json:"status,omitempty" protobuf:"bytes,3,opt,name=status"`
}
```

JobSpec:

```go
type JobSpec struct {

   // 指定作业在任何给定时间应运行的最大所需 Pod 数。
   // 当((.spec.completions - .status.successful) < .spec.parallelism)时，在稳定状态下运行的pod的实际数量将小于此数量，
   //当剩余工作小于最大并行度时。
   // More info: https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/
   Parallelism *int32 `json:"parallelism,omitempty" protobuf:"varint,1,opt,name=parallelism"`

   // 指定作业应使用的已成功完成 Pod 的所需数量。设置为nil意味着任何pod的成功都表示所有pod的成功，
   // 并允许并行性具有任何正值。设置为1意味着并行性被限制为1，而pod的成功表示作业的成功。
   // More info: https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/
   Completions *int32 `json:"completions,omitempty" protobuf:"varint,2,opt,name=completions"`

   // 指定相对于作业在系统尝试终止之前可能处于活动状态的开始时间的持续时间（秒）；值必须为正整数
   // +optional
   ActiveDeadlineSeconds *int64 `json:"activeDeadlineSeconds,omitempty" protobuf:"varint,3,opt,name=activeDeadlineSeconds"`

   // 指定将此作业标记为失败之前的重试次数。
   // Defaults to 6
   // +optional
   BackoffLimit *int32 `json:"backoffLimit,omitempty" protobuf:"varint,7,opt,name=backoffLimit"`

   // label 选择器
   Selector *metav1.LabelSelector `json:"selector,omitempty" protobuf:"bytes,4,opt,name=selector"`

   // manualSelector controls generation of pod labels and pod selectors.
   // Leave `manualSelector` unset unless you are certain what you are doing.
   // When false or unset, the system pick labels unique to this job
   // and appends those labels to the pod template.  When true,
   // the user is responsible for picking unique labels and specifying
   // the selector.  Failure to pick a unique label may cause this
   // and other jobs to not function correctly.  However, You may see
   // `manualSelector=true` in jobs that were created with the old `extensions/v1beta1`
   // API.
   // More info: https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/#specifying-your-own-pod-selector
   // +optional
   ManualSelector *bool `json:"manualSelector,omitempty" protobuf:"varint,5,opt,name=manualSelector"`

   // Pod 模版
   Template v1.PodTemplateSpec `json:"template" protobuf:"bytes,6,opt,name=template"`

   // ttlSecondsAfterFinished limits the lifetime of a Job that has finished
   // execution (either Complete or Failed). If this field is set,
   // ttlSecondsAfterFinished after the Job finishes, it is eligible to be
   // automatically deleted. When the Job is being deleted, its lifecycle
   // guarantees (e.g. finalizers) will be honored. If this field is unset,
   // the Job won't be automatically deleted. If this field is set to zero,
   // the Job becomes eligible to be deleted immediately after it finishes.
   // This field is alpha-level and is only honored by servers that enable the
   // TTLAfterFinished feature.
   // +optional
   TTLSecondsAfterFinished *int32 `json:"ttlSecondsAfterFinished,omitempty" protobuf:"varint,8,opt,name=ttlSecondsAfterFinished"`
}
```

JobStatus:

```go
type JobStatus struct {
   // 状态列表
   Conditions []JobCondition `json:"conditions,omitempty" patchStrategy:"merge" patchMergeKey:"type" protobuf:"bytes,1,rep,name=conditions"`

   // 开始时间
   StartTime *metav1.Time `json:"startTime,omitempty" protobuf:"bytes,2,opt,name=startTime"`

   // 完成时间
   CompletionTime *metav1.Time `json:"completionTime,omitempty" protobuf:"bytes,3,opt,name=completionTime"`

   // 活动的 Pod 数量
   Active int32 `json:"active,omitempty" protobuf:"varint,4,opt,name=active"`

   // 成功的 Pod 数量
   Succeeded int32 `json:"succeeded,omitempty" protobuf:"varint,5,opt,name=succeeded"`

   // 失败的 Pod 数量
   Failed int32 `json:"failed,omitempty" protobuf:"varint,6,opt,name=failed"`
}
```

创建：

```http
POST /apis/batch/v1/namespaces/{namespace}/jobs
```

添加配置：

```http
PATCH /apis/batch/v1/namespaces/{namespace}/jobs/{name}
```

修改：

```http
PUT /apis/batch/v1/namespaces/{namespace}/jobs/{name}
```

删除：

````http
DELETE /apis/batch/v1/namespaces/{namespace}/jobs/{name}
````

删除一堆：

```http
DELETE /apis/batch/v1/namespaces/{namespace}/jobs
```

读取：

```http
GET /apis/batch/v1/namespaces/{namespace}/jobs/{name}
```

读取列表：

```http
GET /apis/batch/v1/namespaces/{namespace}/jobs
```

读取全部命名空间的：

```http
GET /apis/batch/v1/jobs
```

添加状态：

```http
PATCH /apis/batch/v1/namespaces/{namespace}/jobs/{name}/status
```

读取状态：

```http
GET /apis/batch/v1/namespaces/{namespace}/jobs/{name}/status
```

修改状态：

```http
PUT /apis/batch/v1/namespaces/{namespace}/jobs/{name}/status
```



## CronJob

定时任务目前是 beta 版本。

对象结构：

```go
type CronJob struct {
	metav1.TypeMeta `json:",inline"`
	// Standard object's metadata.
	// More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
	// +optional
	metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

	// Specification of the desired behavior of a cron job, including the schedule.
	// More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
	// +optional
	Spec CronJobSpec `json:"spec,omitempty" protobuf:"bytes,2,opt,name=spec"`

	// Current status of a cron job.
	// More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
	// +optional
	Status CronJobStatus `json:"status,omitempty" protobuf:"bytes,3,opt,name=status"`
}
```

CronJobSpec:

```go
type CronJobSpec struct {

   // Cron 格式的调度代码
   // The schedule in Cron format, see https://en.wikipedia.org/wiki/Cron.
   Schedule string `json:"schedule" protobuf:"bytes,1,opt,name=schedule"`

   // 如果作业因任何原因错过计划时间，则启动作业的可选截止时间（秒）。未执行的作业将被视为失败的作业。
   // +optional
   StartingDeadlineSeconds *int64 `json:"startingDeadlineSeconds,omitempty" protobuf:"varint,2,opt,name=startingDeadlineSeconds"`

   // 指定如何处理作业的并发执行。
   // 可选值有:
   // - "Allow" (default): 允许CronJobs并发运行;
   // - "Forbid": 禁止并发运行，如果上一次运行尚未完成，则跳过下一次运行；
   // - "Replace": 取消当前正在运行的作业并将其替换为新作业
   ConcurrencyPolicy ConcurrencyPolicy `json:"concurrencyPolicy,omitempty" protobuf:"bytes,3,opt,name=concurrencyPolicy,casttype=ConcurrencyPolicy"`

   // 此标志告诉控制器暂停后续执行，它不适用于已启动的执行。默认为false。
   // +optional
   Suspend *bool `json:"suspend,omitempty" protobuf:"varint,4,opt,name=suspend"`

   // Job 模版，其实就是一个 Job
   JobTemplate JobTemplateSpec `json:"jobTemplate" protobuf:"bytes,5,opt,name=jobTemplate"`

   // 要保留的成功完成作业数。
   // 这是用于区分显式零和未指定的指针。
   // Defaults to 3.
   // +optional
   SuccessfulJobsHistoryLimit *int32 `json:"successfulJobsHistoryLimit,omitempty" protobuf:"varint,6,opt,name=successfulJobsHistoryLimit"`

   // The number of failed finished jobs to retain.
   // This is a pointer to distinguish between explicit zero and not specified.
   // Defaults to 1.
   // +optional
   FailedJobsHistoryLimit *int32 `json:"failedJobsHistoryLimit,omitempty" protobuf:"varint,7,opt,name=failedJobsHistoryLimit"`
}
```

CronJobStatus:

```go
type CronJobStatus struct {
   // 正在运行的 Job
   Active []v1.ObjectReference `json:"active,omitempty" protobuf:"bytes,1,rep,name=active"`

   // 最后一个执行时间
   LastScheduleTime *metav1.Time `json:"lastScheduleTime,omitempty" protobuf:"bytes,4,opt,name=lastScheduleTime"`
}
```

创建：

```http
POST /apis/batch/v1beta1/namespaces/{namespace}/cronjobs
```

添加配置：

```http
PATCH /apis/batch/v1beta1/namespaces/{namespace}/cronjobs/{name}
```

修改：

```http
PUT /apis/batch/v1beta1/namespaces/{namespace}/cronjobs/{name}
```

删除：

```http
DELETE /apis/batch/v1beta1/namespaces/{namespace}/cronjobs/{name}
```

删除一堆：

```http
DELETE /apis/batch/v1beta1/namespaces/{namespace}/cronjobs
```

读取：

```http
GET /apis/batch/v1beta1/namespaces/{namespace}/cronjobs/{name}
```

读取列表：

```http
GET /apis/batch/v1beta1/namespaces/{namespace}/cronjobs
```

读取全部命名空间的：

```http
GET /apis/batch/v1beta1/cronjobs
```

添加状态：

```http
PATCH /apis/batch/v1beta1/namespaces/{namespace}/cronjobs/{name}/status
```

读取状态：

```http
GET /apis/batch/v1beta1/namespaces/{namespace}/cronjobs/{name}/status
```

修改状态：

```http
PUT /apis/batch/v1beta1/namespaces/{namespace}/cronjobs/{name}/status
```







