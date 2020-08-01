# Jenkins 安装

CentOS 7 安装方式如下，前提是保证有 JDK 环境：

```bash
$ sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
$ sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
$ sudo yum install jenkins
```

启动：

```bash
$ sudo systemctl enable jenkins
$ sudo systemctl start jenkins
```

查看状态：

```bash
$ sudo systemctl status jenkins
```

默认的配置文件是：`/etc/sysconfig/jenkins`

web界面地址默认是：http://localhost:8080/

初始密码默认在 `/var/lib/jenkins/secrets/initialAdminPassword` 中。用户名是 admin。

修改端口，在文件 `/etc/sysconfig/jenkins` 中：

```
JENKINS_PORT="18080"
```

在 `/var/lib/jenkins/hudson.model.UpdateCenter.xml ` 中，可以配置插件地址。

默认地址是：https://updates.jenkins.io/update-center.json

现在新版本修改插件地址还是有点麻烦的。

不过可以在 Jenkins 中可以设置代理。









