# 连接外部 TCP 服务

```yaml
apiVersion: networking.istio.io/v1beta1
kind: ServiceEntry
metadata:
  name: zk-external
  namespace: s1-search
spec:
  hosts:
  - s1.cloud.bbdops.com
  addresses:
  - 10.28.109.27/32
  ports:
  - name: tcp
    number: 2181
    protocol: tcp
  location: MESH_EXTERNA
```

