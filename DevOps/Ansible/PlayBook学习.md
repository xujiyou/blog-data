# PlayBook 学习

先写个 demo，命名为 ansible-test.yaml：

```yaml
---
- hosts: fueltank-1,fueltank-2,fueltank-3
  name: test
  remote_user: admin
  tasks:
    - name: test task
      command: mkdir /tmp/xujiyou-test
```

执行：

```bash
$ ansible-playbook ansible-test.yaml -f 10
```

效果不错哦。