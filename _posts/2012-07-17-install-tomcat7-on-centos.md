---
layout: post
title: "在 CentOS6 上安装 Tomcat7"
category: linux 
tags: 
- linux
- centos
- tomcat
---
{% include JB/setup %}

## 1. 下载

	# wget http://mirror.bit.edu.cn/apache/tomcat/tomcat-7/v7.0.29/bin/apache-tomcat-7.0.29.tar.gz
	
## 2. 安装

	# tar -xzvf apache-tomcat-7.0.29.tar.gz
	# mv apache-tomcat-7.0.29 /opt/tomcat7
	# cd /opt/tomcat7
	# bin/startup.sh	
	
## 3. 配置

在生产环境用 root 是不安全的，所以

	# useradd -s /sbin/nologin tomcat
	# chown -R tomcat:tomcat /opt/tomcat7
	
做为 service，和操作系统一起启动

	# cd /opt/tomcat7/bin
	# tar -xzvf commons-daemon-native.tar.gz
	# cd commons-daemon-1.0.10-native-src/unix
	# ./configure 
	# make
	# cp jsvc ../..
	# cd ../..

在 daemon.sh 的注释后边，正文最开始增加下边五行内容
	
	# vim daemon.sh
	----
	# chkconfig: 2345 10 90 
	# description: Starts and Stops the Tomcat daemon. 
	
	JAVA_HOME=/usr/java/jdk1.6.0_31
	CATALINA_HOME=/opt/tomcat7
	CATALINA_OPTS="-Xms1024m -Xmx2048m -XX:PermSize=256m -XX:MaxPermSize=512m" 

增加到 service

	# cp daemon.sh /etc/init.d/tomcat
	# chkconfig --add tomcat

检查
	
	# chkconfig --list|grep tomcat
	tomcat         	0:关闭	1:关闭	2:启用	3:启用	4:启用	5:启用	6:关闭
	
打开端口

	# vim /etc/sysconfig/iptables
	----
	-A INPUT -m state --state NEW -m tcp -p tcp --dport 8080 -j ACCEPT
	
	# service iptables restart		
	
			
