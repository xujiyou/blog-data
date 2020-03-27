# Vault 使用示例

服务启动之后，就可以来运行一些示例了。

```bash
$ vault secrets list
$ vault write cubbyhole/one foo1=bar1 foo2=bar2
$ vault kv list cubbyhole/
$ vault list cubbyhole/
```

从 https://github.com/hashicorp/vault/issues/8531 这里找到的：

```bash
$ vault secrets enable -version=2 kv
$ vault kv put kv/landscape-master-password landscape-master-password=evenmoresecret
$ vim managelmp.cnf
path "kv/data/landscape-master-password" {
  capabilities = ["create", "read", "update", "delete", "list"]
}
$ vault policy write managelmp managelmp.cnf 

$ vault auth enable -path=authtest userpass
$ vault auth list
$ vault write auth/authtest/users/engler password=verysecret policies=managelmp

$ vault auth enable oidc

```

