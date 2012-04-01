---
layout: post
title: "Cloud Foundry 快速开始"
category: 
tags:
- 云计算
---

前天去参加了 VMware 主办的 [Cloud Foundry 2012 上海开发者大会](http://cloudfoundry2012.csdn.net/shanghai.html)，
今天在自己机器上玩了一下，纪录一下过程和问题。
基本上可以按照 [Getting Started with Cloud Foundry](http://start.cloudfoundry.com/getting-started.html) 
的指引快速开发、部署自己的 App 到 Cloud Foundry。几个关键点纪录一下。

* *下载 Micro Cloud Foundry。*
这个文件比较大，但国内的下载速度不给力，下载了很多次都失败了。
后来发现是因为这个页面可能有 Session，Session 超时后导致下载失败。后来在 Chrome 上装了个叫 AutoReloader
的插件，用这个插件设置 15 分钟刷新一次页面，才下载成功。

* *安装和设置 Micro Cloud Foundry。* 参考 [Get the Micro Cloud Foundry VM Installed and Running](http://start.cloudfoundry.com/infrastructure/micro/installing-mcf.html) 
。但是在 `Enter Micro Cloud Foundry configuration token or offline domain name`
这一步必须要使用 `configuration token` 。

* *部署第一个 App。* 参考 [Create a Simple Ruby Application and Deploy to Cloud Foundry](http://start.cloudfoundry.com/frameworks/ruby/ruby-simple.html)
可以很快速的完成第一个 App。

* *虚拟机网络设置。* [Using the Micro Cloud Foundry Console](http://start.cloudfoundry.com/infrastructure/micro/using-mcf.html#using-micro-cloud-foundry-sts)
中有一段说明 `Switching between Networks`，但当我把默认的 NAT 改成 Bridge 后，直接 `vmc target` 这时是无效的，需要在宿主机上清除 DNS 缓存。具体的解决办法在
[Micro Cloud Foundry Trouble Shooting Help](http://support.cloudfoundry.com/entries/20332921-micro-cloud-foundry-trouble-shooting-help) 也有提到。在我的 Mac 上
执行 `dscacheutil -flushcache` 以后就可以了。


	
