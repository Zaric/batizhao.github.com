---
layout: post
title: "在 RHEL6 上安装 Java"
category: linux
tags: 
- linux
- java
---

卸载老版本的 Java

	# java -version
	# rpm -qa|grep java
	# rpm -e --nodeps java-1.5.0-gcj-1.5.0.0-29.1.el6.x86_64
	
bin 安装
	
	# chmod a+x jdk-6u31-linux-x64.bin
	# cd /opt/
	# mkdir java
	# cd java
	# /home/bati/Downloads/jdk-6u31-linux-x64.bin
	
	Java(TM) SE Development Kit 6 successfully installed.
	......
	Press Enter to continue.....
	Done.
	
	安装到执行 bin 文件的目录。
	
rpm 安装

	# chmod a+x jdk-6u31-linux-x64-rpm.bin
	# ./jdk-6u31-linux-x64-rpm.bin
	
	安装到了 /usr/java
	
配置环境变量

	# vim /etc/profile
	
	增加以下内容
	export JAVA_HOME=/usr/java/jdk1.6.0_31
	export PATH=$JAVA_HOME/bin:$PATH

即时生效：

	# source /etc/profile
	# java -version 
