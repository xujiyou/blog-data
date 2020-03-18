# Kubernetes API 学习(九) - CRD

CRD 是 API 资源的扩展点，全称是 CustomResourceDefinition

```bash
$ kubectl api-resources --api-group=apiextensions.k8s.io
NAME                        SHORTNAMES   APIGROUP               NAMESPACED   KIND
customresourcedefinitions   crd,crds     apiextensions.k8s.io   false        CustomResourceDefinition
```

代码在 `staging/src/k8s.io/apiextensions-apiserver/pkg/apis/apiextensions/types.go` 中

对象结构：

```go
type CustomResourceDefinition struct {
   metav1.TypeMeta
   metav1.ObjectMeta

   // Spec describes how the user wants the resources to appear
   Spec CustomResourceDefinitionSpec
   // Status indicates the actual state of the CustomResourceDefinition
   Status CustomResourceDefinitionStatus
}
```

CustomResourceDefinitionSpec:

```go
type CustomResourceDefinitionSpec struct {
   // 指定新资源属于哪个组
   Group string
   // 过期了，请使用 Versions
   Version string
   // 可以用哪些名字描述这个 CRD，详细请看下面的 CustomResourceDefinitionNames
   Names CustomResourceDefinitionNames
   // CRD 的作用域，Cluster 或 Namespaced
   Scope ResourceScope
   // 验证
   // Validation describes the validation methods for CustomResources
   // Optional, the global validation schema for all versions.
   // Top-level and per-version schemas are mutually exclusive.
   // +optional
   Validation *CustomResourceValidation
   // 子资源
   // Subresources describes the subresources for CustomResource
   // Optional, the global subresources for all versions.
   // Top-level and per-version subresources are mutually exclusive.
   // +optional
   Subresources *CustomResourceSubresources
   // 版本列表
   // Versions is the list of all supported versions for this resource.
   // If Version field is provided, this field is optional.
   // Validation: All versions must use the same validation schema for now. i.e., top
   // level Validation field is applied to all of these versions.
   // Order: The version name will be used to compute the order.
   // If the version string is "kube-like", it will sort above non "kube-like" version strings, which are ordered
   // lexicographically. "Kube-like" versions start with a "v", then are followed by a number (the major version),
   // then optionally the string "alpha" or "beta" and another number (the minor version). These are sorted first
   // by GA > beta > alpha (where GA is a version with no suffix such as beta or alpha), and then by comparing
   // major version, then minor version. An example sorted list of versions:
   // v10, v2, v1, v11beta2, v10beta3, v3beta1, v12alpha1, v11alpha2, foo1, foo10.
   Versions []CustomResourceDefinitionVersion
   // 指定 CRD 的字段
   AdditionalPrinterColumns []CustomResourceColumnDefinition

   // 版本之间的转换规则设置
   Conversion *CustomResourceConversion

   // preserveUnknownFields禁止修剪OpenAPI架构中未指定的对象字段。
   // 始终保留元数据中的apiVersion、kind、metadata和已知字段。
   // v1beta默认为true，v1默认为false。
   PreserveUnknownFields *bool
}
```

CustomResourceDefinitionVersion

```go
type CustomResourceDefinitionVersion struct {
   // Name is the version name, e.g. “v1”, “v2beta1”, etc.
   Name string
   // Served is a flag enabling/disabling this version from being served via REST APIs
   Served bool
   // Storage flags the version as storage version. There must be exactly one flagged
   // as storage version.
   Storage bool
   // Schema describes the schema for CustomResource used in validation, pruning, and defaulting.
   // Top-level and per-version schemas are mutually exclusive.
   // Per-version schemas must not all be set to identical values (top-level validation schema should be used instead)
   // This field is alpha-level and is only honored by servers that enable the CustomResourceWebhookConversion feature.
   // +optional
   Schema *CustomResourceValidation
   // Subresources describes the subresources for CustomResource
   // Top-level and per-version subresources are mutually exclusive.
   // Per-version subresources must not all be set to identical values (top-level subresources should be used instead)
   // This field is alpha-level and is only honored by servers that enable the CustomResourceWebhookConversion feature.
   // +optional
   Subresources *CustomResourceSubresources
   // AdditionalPrinterColumns are additional columns shown e.g. in kubectl next to the name. Defaults to a created-at column.
   // Top-level and per-version columns are mutually exclusive.
   // Per-version columns must not all be set to identical values (top-level columns should be used instead)
   // This field is alpha-level and is only honored by servers that enable the CustomResourceWebhookConversion feature.
   // NOTE: CRDs created prior to 1.13 populated the top-level additionalPrinterColumns field by default. To apply an
   // update that changes to per-version additionalPrinterColumns, the top-level additionalPrinterColumns field must
   // be explicitly set to null
   // +optional
   AdditionalPrinterColumns []CustomResourceColumnDefinition
}
```

CustomResourceColumnDefinition

```go
type CustomResourceColumnDefinition struct {
   // name is a human readable name for the column.
   Name string
   // type is an OpenAPI type definition for this column.
   // See https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md#data-types for more.
   Type string
   // format is an optional OpenAPI type definition for this column. The 'name' format is applied
   // to the primary identifier column to assist in clients identifying column is the resource name.
   // See https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md#data-types for more.
   Format string
   // description is a human readable description of this column.
   Description string
   // priority is an integer defining the relative importance of this column compared to others. Lower
   // numbers are considered higher priority. Columns that may be omitted in limited space scenarios
   // should be given a higher priority.
   Priority int32

   // JSONPath is a simple JSON path, i.e. without array notation.
   JSONPath string
}
```

CustomResourceConversion:

```go
type CustomResourceConversion struct {
   // `strategy` specifies the conversion strategy. Allowed values are:
   // - `None`: The converter only change the apiVersion and would not touch any other field in the CR.
   // - `Webhook`: API Server will call to an external webhook to do the conversion. Additional information
   //   is needed for this option. This requires spec.preserveUnknownFields to be false.
   Strategy ConversionStrategyType

   // `webhookClientConfig` is the instructions for how to call the webhook if strategy is `Webhook`.
   WebhookClientConfig *WebhookClientConfig

   // ConversionReviewVersions is an ordered list of preferred `ConversionReview`
   // versions the Webhook expects. API server will try to use first version in
   // the list which it supports. If none of the versions specified in this list
   // supported by API server, conversion will fail for this object.
   // If a persisted Webhook configuration specifies allowed versions and does not
   // include any versions known to the API Server, calls to the webhook will fail.
   // +optional
   ConversionReviewVersions []string
}
```

CustomResourceDefinitionNames

```go
type CustomResourceDefinitionNames struct {
   // 复数形势，全小写
   Plural string
   // 单数形式，Kind 的小写版本
   Singular string
   // 简称列表
   ShortNames []string
   // 资源类型，驼峰规则
   Kind string
   // ListKind is the serialized kind of the list for this resource.  Defaults to <kind>List.
   ListKind string
   // Categories is a list of grouped resources custom resources belong to (e.g. 'all')
   // +optional
   Categories []string
}
```



CustomResourceDefinitionStatus:

```go
type CustomResourceDefinitionStatus struct {
   // Conditions indicate state for particular aspects of a CustomResourceDefinition
   Conditions []CustomResourceDefinitionCondition

   // AcceptedNames are the names that are actually being used to serve discovery
   // They may be different than the names in spec.
   AcceptedNames CustomResourceDefinitionNames

   // StoredVersions are all versions of CustomResources that were ever persisted. Tracking these
   // versions allows a migration path for stored versions in etcd. The field is mutable
   // so the migration controller can first finish a migration to another version (i.e.
   // that no old objects are left in the storage), and then remove the rest of the
   // versions from this list.
   // None of the versions in this list can be removed from the spec.Versions field.
   StoredVersions []string
}
```

创建：

```http
POST /apis/apiextensions.k8s.io/v1/customresourcedefinitions
```

添加配置：

```http
PATCH /apis/apiextensions.k8s.io/v1/customresourcedefinitions/{name}
```

修改：

```http
PUT /apis/apiextensions.k8s.io/v1/customresourcedefinitions/{name}
```

删除：

```http
DELETE /apis/apiextensions.k8s.io/v1/customresourcedefinitions/{name}
```

删除一堆：

```http
DELETE /apis/apiextensions.k8s.io/v1/customresourcedefinitions
```

读取：

```http
GET /apis/apiextensions.k8s.io/v1/customresourcedefinitions/{name}
```

读取列表：

```http
GET /apis/apiextensions.k8s.io/v1/customresourcedefinitions
```

添加状态：

```http
PATCH /apis/apiextensions.k8s.io/v1/customresourcedefinitions/{name}/status
```

读取状态：

```http
GET /apis/apiextensions.k8s.io/v1/customresourcedefinitions/{name}/status
```

修改状态：

```http
PUT /apis/apiextensions.k8s.io/v1/customresourcedefinitions/{name}/status
```























