---
layout: post
title: "在 RHEL6 上安装 OpenDJ"
category: linux
tags: 
- linux
- ldap
- sso
---

官网"OpenDJ is a extension of the Sun Microsystems' initiated OpenDS project and offers a fully supported product for it."
接下来的这几篇文章会介绍 ForgeRock Open Platform 的 OpenDJ、OpenIDM、OpenAM 三个产品的安装、配置，
以及如何使用他们来搭建企业用户管理、访问认证的基础平台。

确认 Java 环境，否则请参考[在 RHEL6 上安装 Java](/linux/2012/03/29/install-java-on-rhel6/)

	# java -version
	java version "1.6.0_31"
	Java(TM) SE Runtime Environment (build 1.6.0_31-b04)
	Java HotSpot(TM) 64-Bit Server VM (build 20.6-b01, mixed mode)

下载

	# wget http://download.forgerock.org/downloads/opendj/2.4.5/OpenDJ-2.4.5.zip
	
安装
	
	# cp OpenDJ-2.4.5.zip /opt
	# cd /opt
	# unzip OpenDJ-2.4.5.zip
	# mv OpenDJ-2.4.5 opendj
	# cd opendj
	
	如果在图形界面
	# ./setup
	
	这里以 shell 安装为例
	# ./setup --cli
	
	OpenDJ 2.4.5
	安装程序正在初始化，请稍候...
	
	您希望将哪些内容用作目录服务器的初始超级用户 DN？ [cn=Directory Manager]: 
	请提供用于初始超级用户的密码: password
	请重新输入密码以进行确认: password
	
	您希望目录服务器使用哪个端口接受来自 LDAP 客户端的连接？ [389]: 1389
	
	您希望管理连接器在哪个端口上接受连接？ [4444]: 
	是否要在服务器中创建基 DN？ (yes / no) [yes]: 
	
	提供目录数据的基 DN: [dc=example,dc=com]: 
	用于填充数据库的选项:
	
		1)  仅创建基条目
		2)  将数据库保留为空
		3)  从 LDIF 文件中导入数据
		4)  加载自动生成的样例数据
	
	输入选项 [1]: 
	
	是否要启用 SSL？ (yes / no) [no]: 
	
	是否要启用 StartTLS？ (yes / no) [no]: 
	
	是否要在完成配置时启动服务器？ (yes / no) [yes]: 
	
	
	安装摘要
	=============
	LDAP 侦听器端口: 389
	管理连接器端口:   4444
	LDAP 安全访问:  已禁用
	超级用户 DN:    cn=Directory Manager
	目录数据:       创建新的基 DN dc=example,dc=com。
				基 DN 数据: 仅创建基条目 (dc=example,dc=com)
	
	在完成配置时启动服务器
	
	
	您希望执行哪些操作？
	
		1)  使用上面的参数设置服务器
		2)  再次提供安装参数
		3)  打印等效的非交互命令行
		4)  取消并退出
	
	输入选项 [1]: 

	请参见 /tmp/opends-setup-1826847628910167129.log 以了解有关此操作的详细日志。
	
	正在配置目录服务器 ..... 完成。
	正在创建基条目 dc=example,dc=com ..... 完成。
	正在启动目录服务器 ................
	
停止

	# bin/stop-ds
	
启动

	# bin/start-ds
	
启动控制面板

	# bin/control-panel
	
加入到 service（以 root 运行）

	# bin/create-rc-script -f /etc/init.d/opendj	
	
加入到 service（以 userName 运行）

	# bin/create-rc-script -f /etc/init.d/opendj -u userName	

查看 service 启动设置，345 已经生效，Reboot 之后 OpenDJ 自动启动
	
	# chkconfig --list|grep opendj

	opendj         	0:关闭	1:关闭	2:关闭	3:启用	4:启用	5:启用	6:关闭
	
运行

	# service opendj { start | stop | restart }				
	
更详细的安装文档，请参照 [官方文档](http://opendj.forgerock.org/doc/install-guide/index/preface.html) 或者
[在 CentOS6 上安装 OpenDJ（GUI）](/linux/2012/07/13/install-opendj-on-centos6-with-gui/)。	