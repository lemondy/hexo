title: windows 10下快速运行 gyee 教程之WSL
date: 2019-06-23 00:00:59
tags: [yeeco, 公链, wsl, 教程]
---
本篇文章主要介绍的是在windows 10下快速运行yeeco 公链进行测试。
Windows Subsystem for Linux（简称WSL）是一个为在Windows 10上能够原生运行Linux二进制可执行文件（ELF格式）的兼容层。它是由微软与Canonical公司合作开发，目标是使纯正的Ubuntu 14.04 "Trusty Tahr"映像能下载和解压到用户的本地计算机，并且映像内的工具和实用工具能在此子系统上原生运行。

<!-- more -->

> 上面摘抄自[百度百科WSL](https://baike.baidu.com/item/wsl/20359185?fr=aladdin)

## 第一步：确认系统版本
Windows 10 下查看系统设置-关于，在Windows规格下面可以查看系统版本号。确保系统版本号大于 1703（我的版本是 1809）。
更多关于如何查看[Windows 10 系统版本](http://baijiahao.baidu.com/s?id=1599865729643161638&wfr=spider&for=pc)

## 第二步：打开 wsl（Windows Subsystem for Linux） 开关

右击我的电脑，进入：控制面板 - 程序 - 启用或关闭 Windows 功能 - 开启 适用于 Linux 的 Windows 子系统

![open-windows-wsl](/images/yeeco/windows-open-wsl.png)

## 第三步：打开 Windows Store，安装ubuntu
打开 Microsoft Store，然后选择你喜爱的 Linux 分发。例如 Ubuntu，选择安装想要的linux发行版本
搜索框内，输入 Ubuntu，选择版本然后安装，我这里选择的是 18.04 的版本（推荐这个版本）
![windows-app-store](/images/yeeco/windows-app-store.png)

![windows-app-install](/images/yeeco/windows-app-install.png)
这里安装完成后，我们可以再windows 的启动菜单栏里面看到如下的图标，点击该图标
![start-ubuntu](/images/yeeco/start-ubuntu.png)
首次启动的时候需要输入ubuntu系统的用户名和密码，密码输入的时候窗口中是不显示的，创建完成后，我们可以关闭这个窗口了。

![ubuntu-new-user](/images/yeeco/ubuntu-new-user.png)

## 第四步：启动 Ubuntu 系统
1.在开始菜单中，启动 Ubuntu 系统：打开两个ubuntu系统窗口(点击这个两次)
![ubuntu-new-user](/images/yeeco/start-ubuntu.png)

2.下载gyee客户端：
在第一个窗口中从上倒下依次执行如下命令：

```shell
cd ~ && mkdir gyee && cd gyee
wget https://github.com/lemondy/gyee/releases/download/v0.5.3/gyee.linux_x64.20190614.zip
unzip gyee.linux_x64.20190614.zip
./gyee
```

执行成功的情况下我们可以看到如下的输出
![gyee-run-log](/images/yeeco/gyee-run-log.png)

在第二个窗口中执行如下命令:

```shell
cd ~/gyee && ./gyee attach
```

这个执行成功后，我们可以看到如下的交互窗口，在这个交互的窗口中我们就可以执行账户的创建，块信息查询，转账等操作了。
![gyee-attach-run](/images/yeeco/gyee-attach-run.png)

Yeeco链中提供的更多命令可以查看：https://shimo.im/docs/yXlbTf1U12IPiyRq/read