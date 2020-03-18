# Kubernetes API 学习(十五) - certificates

证书管理：

```bash
$ kubectl api-resources --api-group=certificates.k8s.io
NAME                         SHORTNAMES   APIGROUP              NAMESPACED   KIND
certificatesigningrequests   csr          certificates.k8s.io   false        CertificateSigningRequest
```

源码位于 API 源码包的 certificates 包中。

用法参考：https://kubernetes.io/zh/docs/tasks/tls/managing-tls-in-a-cluster/

对象结构：

```go
type CertificateSigningRequest struct {
	metav1.TypeMeta `json:",inline"`
	// +optional
	metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

	// The certificate request itself and any additional information.
	// +optional
	Spec CertificateSigningRequestSpec `json:"spec,omitempty" protobuf:"bytes,2,opt,name=spec"`

	// Derived information about the request.
	// +optional
	Status CertificateSigningRequestStatus `json:"status,omitempty" protobuf:"bytes,3,opt,name=status"`
}
```

CertificateSigningRequestSpec

```go
type CertificateSigningRequestSpec struct {
   // CSR 数据  $(cat server.csr | base64 | tr -d '\n')
   Request []byte `json:"request" protobuf:"bytes,1,opt,name=request"`

   // 用途
   // allowedUsages specifies a set of usage contexts the key will be
   // valid for.
   // See: https://tools.ietf.org/html/rfc5280#section-4.2.1.3
   //      https://tools.ietf.org/html/rfc5280#section-4.2.1.12
   Usages []KeyUsage `json:"usages,omitempty" protobuf:"bytes,5,opt,name=usages"`

   // Information about the requesting user.
   // See user.Info interface for details.
   // +optional
   Username string `json:"username,omitempty" protobuf:"bytes,2,opt,name=username"`
   // UID information about the requesting user.
   // See user.Info interface for details.
   // +optional
   UID string `json:"uid,omitempty" protobuf:"bytes,3,opt,name=uid"`
   // Group information about the requesting user.
   // See user.Info interface for details.
   // +optional
   Groups []string `json:"groups,omitempty" protobuf:"bytes,4,rep,name=groups"`
   // Extra information about the requesting user.
   // See user.Info interface for details.
   // +optional
   Extra map[string]ExtraValue `json:"extra,omitempty" protobuf:"bytes,6,rep,name=extra"`
}
```

CertificateSigningRequestStatus：

```go
type CertificateSigningRequestStatus struct {
   // 状态列表
   Conditions []CertificateSigningRequestCondition `json:"conditions,omitempty" protobuf:"bytes,1,rep,name=conditions"`

   // 证书数据
   Certificate []byte `json:"certificate,omitempty" protobuf:"bytes,2,opt,name=certificate"`
}
```

创建：

```http
POST /apis/certificates.k8s.io/v1beta1/certificatesigningrequests
```

添加配置：

```http
PATCH /apis/certificates.k8s.io/v1beta1/certificatesigningrequests/{name}
```

修改：

```http
PUT /apis/certificates.k8s.io/v1beta1/certificatesigningrequests/{name}
```

删除：

```http
DELETE /apis/certificates.k8s.io/v1beta1/certificatesigningrequests/{name}
```

删除一堆：

```http
DELETE /apis/certificates.k8s.io/v1beta1/certificatesigningrequests
```

读取：

```http
GET /apis/certificates.k8s.io/v1beta1/certificatesigningrequests/{name}
```

读取列表：

```http
GET /apis/certificates.k8s.io/v1beta1/certificatesigningrequests
```

添加状态：

```http
PATCH /apis/certificates.k8s.io/v1beta1/certificatesigningrequests/{name}/status
```

读取状态：

```http
GET /apis/certificates.k8s.io/v1beta1/certificatesigningrequests/{name}/status
```

修改状态：

```http
PUT /apis/certificates.k8s.io/v1beta1/certificatesigningrequests/{name}/status
```













