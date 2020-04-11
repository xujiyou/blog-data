# Istio crd （四） - networking

`networking.istio.io` 中的 CRD 是 Istio 中最重要的 CRD。

```
$ kubectl api-resources --api-group=networking.istio.io
NAME               SHORTNAMES   APIGROUP              NAMESPACED   KIND
destinationrules   dr           networking.istio.io   true         DestinationRule
envoyfilters                    networking.istio.io   true         EnvoyFilter
gateways           gw           networking.istio.io   true         Gateway
serviceentries     se           networking.istio.io   true         ServiceEntry
sidecars                        networking.istio.io   true         Sidecar
virtualservices    vs           networking.istio.io   true         VirtualService
```

