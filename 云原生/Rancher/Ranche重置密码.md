# Rancher 重置密码

Rancher 的默认用户和密码是 admin/admin ，在第一次登录时会让修改密码。

如果忘记了密码，可以执行命令：

```
$ kubectl exec -it rancher-65bd74d789-748wj -n cattle-system -- reset-password
```

会生成一个随机密码，可以用这个随机密码登录，然后登录之后可以再改成自己想设置的密码。