---
layout: post
title: "在 RHEL6 上安装 OpenLDAP"
category: linux
tags: 
- linux
- ldap
---
{% include JB/setup %}

安装需要的包：

	# yum install openldap openldap-clients openldap-servers
	
启动服务：

	# service slapd start
	
修改密码：

	# slappasswd
	New password: 
	Re-enter new password:
	
参考 

* RHEL6 [官方文档](http://docs.redhat.com/docs/en-US/Red_Hat_Enterprise_Linux/6/html/Deployment_Guide/ch-Directory_Servers.html)。
* OpenLDAP [官方文档](http://www.openldap.org/doc/admin24/)。
