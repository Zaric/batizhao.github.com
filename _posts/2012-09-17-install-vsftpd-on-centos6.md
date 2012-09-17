---
layout: post
title: "在 CentOS6 上安装 vsftpd"
category: linux
tags: 
- linux
- ftp
---
{% include JB/setup %}

vsftpd 是一款在 Linux 发行版中最受推崇的 FTP 服务器程序。特点是小巧轻快，安全易用。
vsftpd 的名字代表”very secure FTP daemon”, 安全是它的开发者 Chris Evans 考虑的首要问题之一。在这个 FTP 服务器设计开发的最开始的时候，高安全性就是一个目标。

在初次安装过程中遇到了很多问题，主要是关于帐号设置的问题，这里纪录一下。

# 1. 安装 vsftpd

通过 yum 安装

	# yum install vsftpd
	
设置开机自动启动	
	
	# chkconfig vsftpd on
	
启动服务

	# service vsftpd start
	Starting vsftpd for vsftpd:                                [  OK  ]
	
配置防火墙

	# vim /etc/sysconfig/iptables	
	
增加你需要开放的端口

	# -A INPUT -m state --state NEW -m tcp -p tcp --dport 21 -j ACCEPT
	
保存以后重启防火墙

	# service iptables restart
	
# 2. 配置 vsftpd

增加 ftpuser

	# useradd -g ftp -s /sbin/nologin ftpuser
	
设置密码

	# passwd ftpuser

修改配置文件
		
	# cd /etc/vsftpd/
	# mv vsftpd.conf vsftpd.conf.bak
	# vim vsftpd.conf
	
	#行结束不可以有空格
	#禁止匿名用户访问
	anonymous_enable=NO
	
	#允许登入者有写权限
	write_enable=YES
	
	#允许本地用户访问
	local_enable=YES
	
	#本地用户新增档案时的umask值
	local_umask=022
	
	#定义欢迎话语的字符串
	ftpd_banner=Welcome to FTP server.
	
	#启用上传/下载日志记录
	xferlog_enable=YES
	
	#日志文件所在的路径及名称
	xferlog_file=/var/log/vsftpd.log
	
	#将日志文件写成xferlog的标准格式
	xferlog_std_format=YES
	
	#在chroot_list中列出的用户不允许切换到家目录的上级目录
	chroot_list_enable=YES
	
	#如果chroot_local_user设置了YES，那么chroot_list_file  
	#设置的文件里，是不被chroot的用户(可以向上改变目录)  
	#如果chroot_local_user设置了NO，那么chroot_list_file  
	#设置的文件里，是被chroot的用户(无法向上改变目录)  
	chroot_local_user=YES
	chroot_list_file=/etc/vsftpd/chroot_list
	
	userlist_enable=YES
	userlist_deny=YES
	userlist_file=/etc/vsftpd/user_list
	
	#FTP服务器以standalone模式运行
	listen=YES
	
	#FTP服务器启用PORT模式
	port_enable=YES
	
	#禁用FTP服务器的PASV模式
	pasv_enable=NO
	
	#FTP服务器监听21端口
	listen_port=21
	
	#指定FTP服务器使用20端口进行数据传输
	connect_from_port_20=YES
	#FTP服务器数据传输端口为20
	ftp_data_port=20
	
	#600秒钟不对FTP服务器进行任何操作，则断开该FTP连接
	idle_session_timeout=600
	
	#建立FTP数据连接的超时时间为120秒
	data_connection_timeout=120
	
	#不限制用户的连接数量
	max_clients=0
	#每个IP只能与FTP服务器同时建立3个连接
	max_per_ip=3
	
	#PAM认证文件
	pam_service_name=vsftpd
	
	use_localtime=YES
	
	#支持ASCII模式
	ascii_upload_enable=YES
	ascii_download_enable=YES
	
重启

	# service vsftpd restart		

# 3. 问题

* 500 OOPS: cannot change directory:/home/ftpuser


		# getsebool -a|grep ftp
		# setsebool -P ftp_home_dir on
	
* 500 OOPS: could not read chroot() list file:/etc/vsftpd/chroot_list

	缺少	chroot_list 文件，需要手动建立。

		# vim chroot_list
	
* 530 Login incorrect.

	可能一：
	
		# vim vsftpd.conf
		增加：
		# pam_service_name=vsftpd
	
	可能二：密码不正确。

* 550 Permission denied

	这个问题在我的 Cyberduck for Mac 上有遇到，但是“命令行”和“FileZilla”是没问题的。

				
