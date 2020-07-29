---
title: Golang爬虫脚本
date: 2020-07-09 09:43:00
tags:
---

## bbd-online

bbd-online 用于登录BBD内网上网认证系统， 使用前提是安装有 Headless Chrome。

Headless Chrome 安装教程：https://wiki.wyyll.top:48989/blog/5e674034d7055a081f756a81

编译 Linux amd64 版本的二进制文件：

```bash
sh build.sh
```

生产的二进制文件是 bin 目录下的 `bbd-online`， 可以把它放到 Linux 系统中的 PATH 中。

使用方式：

```bash
bbd-online -h
bbd-online xujiyou 123456789
```

效果图：

![image](../../resource/image.png)

```go
package main

import (
	"context"
	"fmt"
	"github.com/chromedp/cdproto/page"
	"github.com/chromedp/chromedp"
	"github.com/spf13/cobra"
	"log"
	"time"
)

func main() {

	var rootCmd = &cobra.Command{
		Use:   "bbd-online <user> <password>",
		Short: "bbd-online 用于登录BBD内网上网认证系统",
		Long: `bbd-online 用于登录BBD内网上网认证系统
使用前提是安装有 Headless Chrome
教程：https://wiki.wyyll.top:48989/blog/5e674034d7055a081f756a81`,
		Args: cobra.ExactArgs(2),
		Run: func(cmd *cobra.Command, args []string) {
			flag := cmd.Flag("url")
			startAuth(args[0], args[1], flag.Value.String())
		},
	}

	rootCmd.Flags().StringP("url", "", "http://192.168.2.199:8099/portal/redirect/nacc/192.168.2.199", "指定登录地址")
	_ = rootCmd.Execute()
}

func startAuth(username string, password string, url string) {
	ctx, cancel := chromedp.NewContext(context.Background())
	defer cancel()

	//自动关闭alert对话框
	chromedp.ListenTarget(ctx, func(ev interface{}) {
		if ev, ok := ev.(*page.EventJavascriptDialogOpening); ok {
			fmt.Println("closing alert:", ev.Message)
			go func() {
				if err := chromedp.Run(ctx,
					page.HandleJavaScriptDialog(true),
				); err != nil {
					panic(err)
				}
			}()
		}
	})

	log.Println("欢迎使用BBD内网上网认证系统，登陆认证中请稍后 ...")

	err := chromedp.Run(ctx, downLine(url, username, password))
	if err != nil {
		log.Println("Note: 获取页面元素失败，请检查网络环境或目标资源可用性!!!")
		log.Fatal(err)
	}

	log.Println("登出上网认证!!!重新登录中...")
	time.Sleep(3 * time.Second)

	var arriveTime string
	var currentUser string
	err = chromedp.Run(ctx, onLine(url, &arriveTime, &currentUser, username, password))
	if err != nil {
		log.Println("Note: 获取页面元素失败，请检查网络环境或目标资源可用性!!!")
		log.Fatal(err)
	}

	log.Println("上网认证成功!!!")
	log.Println("您的上网有效期截止至：", arriveTime)
	log.Println("当前上网用户：", currentUser)

	time.Sleep(1 * time.Second)
}

func downLine(host string, username string, password string) chromedp.Tasks {

	return chromedp.Tasks{
		chromedp.Navigate(host),
		chromedp.WaitVisible(`#usr`, chromedp.ByID),
		chromedp.SetValue(`#usr`, username, chromedp.ByID),
		chromedp.SetValue(`#pwd`, password, chromedp.ByID),
		chromedp.Click(`#loginForm > div > .subbtn`, chromedp.ByID),
		chromedp.WaitVisible(`#downline`, chromedp.ByID),
		chromedp.Click(`#downline`, chromedp.ByID),
	}

}

func onLine(host string, arriveTime *string, currentUser *string, username string, password string) chromedp.Tasks {

	return chromedp.Tasks{
		chromedp.Navigate(host),
		chromedp.WaitVisible(`#usr`, chromedp.ByID),
		chromedp.SetValue(`#usr`, username, chromedp.ByID),
		chromedp.SetValue(`#pwd`, password, chromedp.ByID),
		chromedp.Click(`#loginForm > div > .subbtn`, chromedp.ByID),
		chromedp.WaitVisible(`#downline`, chromedp.ByID),

		chromedp.Text(`#arriveTime`, arriveTime, chromedp.ByID),
		chromedp.Text(`#authUser`, currentUser, chromedp.ByID),
	}

}

```

构建脚本：

```bash
#!/usr/bin/env bash

cd cmd/ && CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o ../bin/bbd-online main.go

scp ../bin/bbd-online bbders@192.168.98.131:/home/bbders

```

