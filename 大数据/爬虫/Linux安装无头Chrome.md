---
title: Linux安装无头Chrome
date: 2020-07-08 10:37:45
tags:
---

公司的网络需要登录，在 PC 上安装了一台没有图形界面的 CentOS 7，要实现上网，需要安装 无头版的 Chrome 浏览器。

安装方法可以上网找，比如这个：https://wiki.wyyll.top:48989/blog/5e674034d7055a081f756a81

PC 上现在是没有网络的，不能下载，但是 Chrome 的 RPM 包又一堆依赖，怎么办那，在另一台 **相同版本的 CentOS **上下载所有的 RPM 包：

```bash
$ yum -y install google-chrome-stable --downloadonly --downloaddir=./
```

然后把下载到的包传到 PC 上，使用命令来安装：

```bash
$ yum localinstall ./*.rpm
```

就可以了。

或者：

```bash
$ rpm -Uvh *.rpm --nodeps --force
```

版本查看：

```bash
$ google-chrome-stable --version
```



## 下载 ChromeDriver

ChromeDriver下载地址：https://share.weiyun.com/5T2zhRy

下载到的是一个 二进制文件，将它放入 PATH。



## BBD 上网脚本

```python
#!/usr/bin/env python
# _*_ coding: utf-8 _*_
# BBD办公区上网认证登入登出
# selenium headless test
# 环境依赖： python2.7.x、selenium模块、chromeDriver
# ChromeDriver下载地址：https://share.weiyun.com/5T2zhRy
import re
import string
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.wait import WebDriverWait
import time
class BBDOnlineAuth(object):
    def __init__(self, url, user, passwd, chromeDriverPath):
        print(u'欢迎使用BBD内网上网认证系统，登陆认证中请稍后 ...\n')
        self.url = url
        self.username = user
        self.password = passwd
        self.chromeExePath = chromeDriverPath
        # 配置chrome浏览器headless请求模式
        chromeOptions = webdriver.ChromeOptions()
        chromeOptions.add_argument("--headless")
        chromeOptions.add_argument("--no-sandbox") # linux root用户下运行代码需添加这一行，如果是linux 普通用户忽略;
        chromeOptions.add_argument("--disable-gpu")
        chromeOptions.add_argument("--window-size=1024x768")
        self.browser = webdriver.Chrome(chrome_options=chromeOptions, executable_path=self.chromeExePath)
    def autoFormInput(self):
        # 自动填充登陆认证页面的用户表单
        try:
            self.browser.get(self.url) # 打开当前会话目标页面
            formUsr = self.browser.find_element_by_id("usr")
            formPwd = self.browser.find_element_by_id("pwd")
            formUsr.send_keys(self.username)
            formPwd.send_keys(self.password)
            # formUsr.send_keys(Keys.ENTER)
            # formPwd.send_keys(Keys.ENTER)
        except Exception as e:
            # 如果在初始化登录页面失败的情况下，直接退出，不再执行后续步骤（页面打不开、或页面改版）
            print(u"Note: 获取页面元素失败，请检查网络环境或目标资源可用性!!!\n",e)
            self.browser.close()
            exit(1)
    def online(self):
        # 直接登入
        try:
            self.autoFormInput()
            print(u'准备提交表单信息...')
            print(self.browser.current_url)
            wait = WebDriverWait(self.browser, 2)
            wait.until(EC.presence_of_element_located((By.CLASS_NAME, "subbtn")))
            # 获取确认提交按钮并点击提交
            self.browser.find_element_by_xpath('//*[@id="loginForm"]/div[1]/div[4]').click()
            wait = WebDriverWait(self.browser, 3)
            wait.until(EC.presence_of_element_located((By.ID, "downline")))
            # 打印当前页面的内容
            # print(self.browser.page_source)
            cureentPageSource = self.browser.page_source
            # print(self.browser.title)  # headless mode of browser.
            # 打印上网凭证有效期和当前登录用户
            print(u"打印上网凭证有效期和当前登录用户... ...")
            print(string.join(re.findall('(.*)<span id="arriveTime">(.*)</span>', cureentPageSource)[0], ' '))
            print(string.join(re.findall('(.*)<span id="authUser">(.*)</span>', cureentPageSource)[0], ' '))
            time.sleep(1)
        except Exception as e:
            print(e)
        finally:
            self.browser.close()
    def downline(self):
        # 先登入再登出
        try:
            self.autoFormInput()
            wait = WebDriverWait(self.browser, 2)
            wait.until(EC.presence_of_element_located((By.CLASS_NAME, "subbtn")))
            # 获取确认提交按钮并点击提交
            self.browser.find_element_by_xpath('//*[@id="loginForm"]/div[1]/div[4]').click()
            print(u'登出上网认证登录!!!')
            print(self.browser.current_url)
            # print(browser.page_source)
            wait = WebDriverWait(self.browser, 3)
            wait.until(EC.presence_of_element_located((By.ID, "downline")))
            self.browser.find_element_by_id("downline").click()
            # 获取当前弹出对话框，并点击确认按钮，真正退出。
            self.browser.switch_to.alert.accept()
            time.sleep(2)
        except Exception as e:
            print(e)
            self.browser.close() # 如果有异常发生，则关闭webdriver实例（self.browser）
        finally:
            self.browser.refresh()
    def downlineAndOnlime(self):
        # pass
        print("to offline ...")
        self.downline()
        print("to online ...")
        self.online()
if __name__ == '__main__':
    destUrl = "http://192.168.2.199:8099/portal/redirect/nacc/192.168.2.199"  # BBD办公区域上网认证地址
    username = "xujiyou"  # 认证用户
    password = "12345678"  # 认证密码
    chromeExePath = "/usr/bin/chromedriver"
    BBDOnlineAuth(destUrl, username, password, chromeExePath).downlineAndOnlime()
```











