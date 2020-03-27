# Vault 简介及入门

Vault 是一款 Secret 管理工具。

MacOS 安装：下载二进制文件，下载地址：https://www.vaultproject.io/downloads/

下载完成后放入 PATH 。

验证安装：

```bash
$ vault version
Vault v1.3.2
```

启动 UI

```bash
$ vault server -dev
```



## systemd 启动 vault

参见：https://learn.hashicorp.com/vault/operations/ops-deployment-guide

编写 service 文件：

```bash
$ sudo vim /usr/lib/systemd/system/vault.service
[Unit]
Description="HashiCorp Vault - A tool for managing secrets"
Documentation=https://www.vaultproject.io/docs/
Requires=network-online.target
After=network-online.target
ConditionFileNotEmpty=/etc/vault/config.hcl
StartLimitIntervalSec=60
StartLimitBurst=3

[Service]
User=admin
Group=admin
ProtectSystem=full
ProtectHome=read-only
PrivateTmp=yes
PrivateDevices=yes
SecureBits=keep-caps
AmbientCapabilities=CAP_IPC_LOCK
Capabilities=CAP_IPC_LOCK+ep
CapabilityBoundingSet=CAP_SYSLOG CAP_IPC_LOCK
NoNewPrivileges=yes
ExecStart=/usr/bin/vault server -config=/etc/vault/config.hcl
ExecReload=/bin/kill --signal HUP $MAINPID
KillMode=process
KillSignal=SIGINT
Restart=on-failure
RestartSec=5
TimeoutStopSec=30
StartLimitInterval=60
StartLimitIntervalSec=60
StartLimitBurst=3
LimitNOFILE=65536
LimitMEMLOCK=infinity

[Install]
WantedBy=multi-user.target
```

我这里使用 etcd 作为储存。编写配置文件：

```bash
$ sudo vim /etc/vault/config.hcl
storage "etcd" {
  address  = "https://fueltank-1:2379,https://fueltank-2:2379,https://fueltank-3:2379"
  etcd_api = "v3"
  ha_enabled = "true"
  tls_ca_file = "/etc/etcd/cert/ca.pem"
  tls_cert_file = "/etc/etcd/cert/etcd.pem"
  tls_key_file = "/etc/etcd/cert/etcd-key.pem"
}

listener "tcp" {
  address = "0.0.0.0:8200"
  cluster_address = "172.20.20.162:8201"
  tls_cert_file = "/etc/vault/cert/vault.pem"
  tls_key_file = "/etc/vault/cert/vault-key.pem"
  tls_client_ca_file = "/etc/kubernetes/cert/ca.pem"
}

api_addr = "https://172.20.20.162:8200"
cluster_addr = "https://172.20.20.162:8201"
ui = true
```

启动：

```bash
$ sudo systemctl enable vault.service
$ sudo systemctl start vault.service
```

服务启动后，要进行初始化：

```bash
$ vault operator init -ca-path=/etc/kubernetes/cert/ca.pem
Unseal Key 1: Xx4CWop5pr8l3CjPvRk2zm9Pi6q79HJR4K1Iv+oWE4HN
Unseal Key 2: ZvnAEmMjYZ1/o0WgtdFwD7CF7e/qV1ii3BTOgAMW/0uY
Unseal Key 3: s3BtR6kFiSb+H9KfZmU6NRAJPS8MzpPNQgdi3P/VIALU
Unseal Key 4: 5qD//UBLAswYYCxHNpWq3iMoTXt5S+dyl8SyO908aZ8q
Unseal Key 5: rZr9awWXYnLqv3MXrRPxp1zD0DakV5SACihzgcpGdEpC

Initial Root Token: s.lkTx7wVmlaCIYYE5I8u9hSBb

Vault initialized with 5 key shares and a key threshold of 3. Please securely
distribute the key shares printed above. When the Vault is re-sealed,
restarted, or stopped, you must supply at least 3 of these keys to unseal it
before it can start servicing requests.

Vault does not store the generated master key. Without at least 3 key to
reconstruct the master key, Vault will remain permanently sealed!

It is possible to generate new unseal keys, provided you have a quorum of
existing unseal keys shares. See "vault operator rekey" for more information.
```

生成的数据要保存好！！！然后进行解封：

```bash
$ vault operator unseal -ca-path=/etc/kubernetes/cert/ca.pem
```

执行三次，每次都输入一个上面的 Unseal Key。

因为上面的配置文件中，配置了 `ui = true` ，所以可以访问 `https://localhost:8200` 来访问 UI 界面，UI 界面也是需要先解封再输入上边的 root token



## vault 配置命令行自动提示

```bash
$ vault -autocomplete-install
$ complete -C /usr/bin/vault vault
```



## 配置 CA 环境变量

```bash
$ echo "export VAULT_CACERT=/etc/kubernetes/cert/ca.pem" >> ~/.bash_profile
$ source ~/.bash_profile
```



## 高可用模式

参考：https://learn.hashicorp.com/vault/operations/ops-vault-ha-consul

按照上边的配置再部署两个节点即可，解封后，使用 `vault status` 查看服务状态，会发现其中一个节点是 `active` 活动节点，两个节点是 `standby` 备用节点。