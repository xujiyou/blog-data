# etcd - golang 客户端

废话不多说，直接上代码：

```go
package main

import (
	"context"
	"crypto/tls"
	"crypto/x509"
	"fmt"
	"go.etcd.io/etcd/clientv3"
	"io/ioutil"
	"log"
	"time"
)

func main() {
	etcdCA, err := ioutil.ReadFile("./etcd/ca.pem")
	if err != nil {
		log.Fatal(err)
	}

	etcdClientCert, err := tls.LoadX509KeyPair("./etcd/etcd.pem", "./etcd/etcd-key.pem")
	if err != nil {
		log.Fatal(err)
	}

	rootCertPool := x509.NewCertPool()
	rootCertPool.AppendCertsFromPEM(etcdCA)

	cli, err := clientv3.New(clientv3.Config{
		Endpoints:   []string{"fueltank-1:2379", "fueltank-2:2379","fueltank-2:2379"},
		RetryDialer: nil,
		DialTimeout: 5 * time.Second,
		TLS: &tls.Config{
			RootCAs:      rootCertPool,
			Certificates: []tls.Certificate{etcdClientCert},
		},
	})
	if err == nil {
		resp,_ := cli.KV.Get(context.TODO(), "foo")
		for i := range resp.Kvs {
			fmt.Printf("resp value: %s", resp.Kvs[i].Value)
		}
	} else {
		_ = fmt.Errorf("err: %s", err)
	}
	defer cli.Close()
}
```

有双向认证，地址不要加 https:// 前缀。