---
layout: post
title: "在 Parallels for Mac 上安装 RHEL6"
category: linux
tags: 
- linux 
- VM
---

昨天想测试一下 RHEL6 上的 lvs 和 piranha，所以在自己的 Mac 上使用 Parallels 安装了两个虚拟机。
在安装过程中，遇到了一些问题，记录一下安装过程。

设置 Parallels VM

![Parallels VM](/images/RHEL-00.jpg)

选择 Install system with basic video driver

![](/images/RHEL-01.jpg)

这一步，选择 Skip，是验证安装镜像完整性的。跳过之后选择语言、存储、分区。

![](/images/RHEL-02.jpg)

在自定义安装的服务器类型这一步，有基本，数据库，WEB，虚拟主机，最小化安装等好多种选择。
这里根据需要做出自己的选择，会影响到包的安装。我在这里选择桌面（否则安装完成后是命令行界面）。
下边的弹性存储、负载平衡器、高可用全部勾选。如果还希望增加其它组件，可以选择“现在自定义”。

![](/images/RHEL-03.jpg)

在自定义安装包这一步，可以自由选择需要安装的组件。在上一步做的设置会影响到这一步的默认选中状态。
这里不建议选择安装：Apache、Tomcat、JDK 等组件，最好自己从官网下载最新版本。

![](/images/RHEL-04.jpg)

在此之后就可以喝杯咖啡，重启系统了。


