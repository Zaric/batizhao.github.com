---
layout: post
title: "在 CentOS6 上安装 OpenDJ（GUI）"
category: linux
tags: 
- linux
- ldap
- sso
---
{% include JB/setup %}

之前有一篇有讲到[在 RHEL6 上安装 OpenDJ](http://batizhao.github.com/linux/2012/03/29/install-opendj-directory-server-on-rhel6/)，
主要是讲通过命令行的方式安装，这次讲一下通过图形界面安装。

## 服务器设置
这一步主要是 host name 的设置。要确保这个域名可以被解析，否则会抛出 javax.naming.CommunicationException: 0.0.0.0:4444 的异常。
临时解决办法是在 hosts 中增加一条纪录 127.0.0.1   idams

![](/images/2012-07-13-install-opendj-on-centos6-with-gui-1.png)

## 复制选项
如果是第一台 Server，选择 This will be a stand alone server

![](/images/2012-07-13-install-opendj-on-centos6-with-gui-2.png)

## 初始化数据
输入你的 Base DN

![](/images/2012-07-13-install-opendj-on-centos6-with-gui-3.png)

## JVM 设置
生产环境分配 2G 以上内存

![](/images/2012-07-13-install-opendj-on-centos6-with-gui-4.png)

## 所有配置

![](/images/2012-07-13-install-opendj-on-centos6-with-gui-5.png)

## 安装完成
如果前边的 host name 有问题，会卡在这一步很长时间，查看 log ，会看到第一步提到的那个异常，这时其实已经安装成功，Cancel 即可，Server 已经启动。

![](/images/2012-07-13-install-opendj-on-centos6-with-gui-6.png)

## 控制面板

这时点击 Launch Control Panel，进入到 administrator 界面

![](/images/2012-07-13-install-opendj-on-centos6-with-gui-7.png)

## 其它问题

如果 host name 有问题，你在 local server 是无法使用 Control Panel 的

![](/images/2012-07-13-install-opendj-on-centos6-with-gui-8.png)
![](/images/2012-07-13-install-opendj-on-centos6-with-gui-9.png)

这时可以使用远程的方式控制，在另外一台主机启动 Control Panel

![](/images/2012-07-13-install-opendj-on-centos6-with-gui-10.png)
![](/images/2012-07-13-install-opendj-on-centos6-with-gui-11.png)

在远程登录之前，不要忘记设置防火墙

	# vim /etc/sysconfig/iptables
	
	-A INPUT -m state --state NEW -m tcp -p tcp --dport 389 -j ACCEPT
	-A INPUT -m state --state NEW -m tcp -p tcp --dport 4444 -j ACCEPT
	
	# service iptables restart
