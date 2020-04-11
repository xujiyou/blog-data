#  Istio crd （三） -  config

`config.istio.io` 中的 CRD 有点多，有 10 个。

```
$ kubectl api-resources --api-group=config.istio.io
NAME                  SHORTNAMES   APIGROUP          NAMESPACED   KIND
adapters                           config.istio.io   true         adapter
attributemanifests                 config.istio.io   true         attributemanifest
handlers                           config.istio.io   true         handler
httpapispecbindings                config.istio.io   true         HTTPAPISpecBinding
httpapispecs                       config.istio.io   true         HTTPAPISpec
instances                          config.istio.io   true         instance
quotaspecbindings                  config.istio.io   true         QuotaSpecBinding
quotaspecs                         config.istio.io   true         QuotaSpec
rules                              config.istio.io   true         rule
templates                          config.istio.io   true         template
```



## adapter

Mixer 适配器能够让 Istio 连接各种基础设施后端以完成类似指标和日志这样的功能。

Istio 1.5 版本之后，Mixer 的功能加入到了 Envoy 之中。

为学习适配器，我们可以自己来造一个简单的适配器。

