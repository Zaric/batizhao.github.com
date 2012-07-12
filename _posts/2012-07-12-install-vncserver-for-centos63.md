---
layout: post
title: "CentOS6.3 安装 vncserver"
category: linux 
tags: 
- linux
- vnc
---
{% include JB/setup %}

在 CentOS 安装好之后，如果想要通过图形界面访问远程主机，需要安装 vnc server。

先查看本机是否有安装vnc：

	# rpm -q vnc-server
	
如果没有安装

	# yum install vnc-server
	
把远程桌面的用户加入到配置文件中（这里的 `2` 和后边的端口有关系）

	# vim /etc/sysconfig/vncservers
 
	VNCSERVERS="2:root"
	VNCSERVERARGS[2]="-geometry 1024x768"
	
为配置的远程桌面用户设置密码（要在相应的帐号下边修改）

	# vncpasswd

启动 vnc server（这一步会生成 xstartup 文件）

	# service vncserver start
	
修改登录配置

	# cd ~/.vnc/   (/root/.vnc)
	# vim xstartup
	
把最后一行 `twm &` 注释，增加新的一行 `gnome-session &`。最终的文件：
	
	#!/bin/sh
	
	[ -r /etc/sysconfig/i18n ] && . /etc/sysconfig/i18n
	export LANG
	export SYSFONT
	vncconfig -iconic &
	unset SESSION_MANAGER
	unset DBUS_SESSION_BUS_ADDRESS
	OS=`uname -s`
	if [ $OS = 'Linux' ]; then
	  case "$WINDOWMANAGER" in
		*gnome*)
		  if [ -e /etc/SuSE-release ]; then
			PATH=$PATH:/opt/gnome/bin
			export PATH
		  fi
		  ;;
	  esac
	fi
	if [ -x /etc/X11/xinit/xinitrc ]; then
	  exec /etc/X11/xinit/xinitrc
	fi
	[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
	xsetroot -solid grey
	vncconfig -iconic &
	xterm -geometry 80x24+10+10 -ls -title "$VNCDESKTOP Desktop" &
	#twm &
	gnome-session &  

重启 vncserver

	# service vncserver restart
	
这时有可能还是不能访问，因为有防火墙。接下来配置防火墙

	# netstat -tunpl|grep vnc
	
	tcp  0  0 0.0.0.0:5902  0.0.0.0:*  LISTEN  6253/Xvnc           
	tcp  0  0 0.0.0.0:6002  0.0.0.0:*  LISTEN  6253/Xvnc           
	tcp  0  0 :::6002       :::*       LISTEN  6253/Xvnc 
	
	# vim /etc/sysconfig/iptables

	-A INPUT -m state --state NEW -m tcp -p tcp --dport 5902 -j ACCEPT
	-A INPUT -m state --state NEW -m tcp -p tcp --dport 6002 -j ACCEPT

	# service iptables restart	

访问	

	通过浏览器：vnc://192.168.1.102:5902
	通过客户端：192.168.1.102:2 有的客户端使用 192.168.1.102:6002
	
![](/images/2012-07-12-install-vncserver-for-centos63.png)	

 


