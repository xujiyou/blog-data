# Git 提交触发 Jenkins 自动构建

在 Jenkins 中，需要安装 Gihub 插件，这里安装过程略过，配置好代理还是很好下载的。



## 获取 GitHub 的 Personal access token

在 Github 的 Setting -> Developer settings -> Personal access tokens 中，点击 Generate new token 按钮，然后配置如下：

![image-20200801141606574](../../resource/image-20200801141606574.png)

然后点击下面的按钮生成一个 Token，记住这个 Token，比如：228805e1c15e81ecc6107220db7b94fcaaf28ab3

**一定要保存，后面就看不到了**



## 配置 Jenkins

11bb51e647a4496aeea0c213fe2d58817a

在  Manage Jenkins -> Configure System 中配置 Github。如下：

![image-20200801142050988](../../resource/image-20200801142050988.png)

![image-20200801142125786](../../resource/image-20200801142125786.png)

















