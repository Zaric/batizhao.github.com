---
layout: post
title: "在 RHEL6 上安装 Apache2 "
category: linux
tags: 
- linux
- apache
- sso
---

最近在配置 OpenAM ，在参考[这篇文章](https://wikis.forgerock.org/confluence/display/openam/Add+Authentication+to+a+Website+using+OpenAM)
时遇到了一个 `[error] Certificate not found: 'Server-Cert'` 的问题。在尝试很多次之后无解，所以下载最新的 apache2.2.22 替换系统自带的 apache2.2.15，看能不能解决问题。

首先，卸载自带的 Apache2

	System - Administration - Add/Remove Sofeware - Web Services - Web Server
	
然后，安装 Apache2.2.22

	# wget http://apache.etoak.com//httpd/httpd-2.2.22.tar.gz
	# tar zxvf httpd-2.2.22.tar.gz
	# cd httpd-2.2.22
	# ./configure --prefix=/opt/apache2
	# make
	# make install
	
如果遇到 `configure: error: no acceptable C compiler found in $PATH`，参考[这篇文章](/linux/2012/03/28/install-gcc/)。

启动 Apache

	# /opt/apache2/bin/apachectl start
	
停止 Apache

	# /opt/apache2/bin/apachectl stop	
	
把 Apache 加入到系统服务

	# cp /opt/apache2/bin/apachectl /etc/rc.d/init.d/httpd
	
修改文件

	# cd /etc/rc.d/init.d/
	# vim httpd
	
	加入以下内容
	###
	# Comments to support chkconfig on RedHat Linux
	# chkconfig: 2345 90 90
	# description:http server
	###
	
启动 Apache

	# service httpd start
	
停止 Apache

	# service httpd stop	
	
加入到系统启动列表

	# chkconfig --add httpd
	
系统启动自动运行

	# chkconfig --level 345 httpd on	
	
	
	
	

