---
layout: post
title: 配置 RHEL6 本地源
category: linux
tags:
- linux
- yum
---

如果你没有注册或没有配置本地源的话，一般都会出现下面的情况：

	Loaded plugins: product-id, refresh-packagekit, security, subscription-manager
	Updating certificate-based repositories.
	Setting up Install Process
	Nothing to do

下面我们要以光盘为 yum 源，你也可以把光盘里面的文件 cp 到系统的某个目录里面：

	# mount /dev/cdrom /media
	mount: block device /dev/sr0 is write-protected, mounting read-only

备份 repo 文件：

	# cd /etc/yum.repos.d/
	# cp rhel-source.repo rhel-source.repo.bak

编辑 repo 文件:

	# vim rhel-source.repo

	内容如下：
	[InstallMedia] 
	name=local_yum 
	baseurl=file:///media 
	gpgcheck=0 
	enabled=1

执行清理缓存命令：

	# yum clean all


