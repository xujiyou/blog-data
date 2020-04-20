# Gitlab CI 教程

首先为项目配置 gitlab-runner

可以按照这个地址来安装：https://packages.gitlab.com/runner/gitlab-runner

```bash
$ curl -s https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh | sudo bash
$ sudo yum install gitlab-runner -y
```

安装完成后，进行注册：https://docs.gitlab.com/runner/register/index.html#gnulinux

```bash
$ sudo gitlab-runner register
```

具体过程如下：

![image-20200415160208230](/Users/jiyouxu/Documents/me/blog/resource/image-20200415160208230.png)

这里的地址和 token 要去项目中的 `setting` --->  `CI/CD`  ---> `Runners` 中获取。

配置完成后，`gitlab-runner` 会自己启动。。

手动启动，多启动一遍也没啥事：

```bash
$ sudo gitlab-runner start
```

查看状态：

```bash
$ sudo gitlab-runner status
```



## .gitlab-ci.yaml 文件

我这里的最简文件：

```yaml
# 定义 stages
stages:
  - test

# 定义 job
job1:
  stage: test
  script:
    - echo "I am job1" >> /home/gitlab-runner/job1.txt
    - echo "I am in test stage" >> /home/gitlab-runner/jjob1.txt
  tags:
    - fueltank
```

主要就是学习这个配置文件怎么写。



## 原理

gitlab-runner 一直在后台 pull 代码，一遇到 commit 就执行 `.gitlab-ci.yaml ` 这里面定义好的命令。