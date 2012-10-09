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
	
	#600秒钟不对FTP服务器进行任何操作，则断开该FTP连接
	idle_session_timeout=600
	
	#建立FTP数据连接的超时时间为120秒
	data_connection_timeout=120
	
	#不限制用户的连接数量
	max_clients=0
	#每个IP与FTP服务器同时建立连接数
	max_per_ip=100
	
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
		pam_service_name=vsftpd
	
	可能二：密码不正确。

* 550 Permission denied

	这个问题可能是多种原因造成的，最常见的是缺少
	
		write_enable=YES
		
	但在我的环境中，出现这个问题是因为服务端 iptables、客户端防火墙、vsftpd pasv 三者之间的配置造成的。参考 [iptables 中配置 vsftp 的访问](http://blog.csdn.net/moreorless/article/details/5289147)	
	
	* 主动模式下，客户连接 TCP/21，服务器通过 TCP/20 连接客户的随机端口。这种情况下，通过状态防火墙可以解决 
	
		iptables -A INPUT -m state --state NEW,RELATED,ESTABLISHED -j ACCEPT
	
	* 被动模式下，客户连接 TCP/21，客户再通过其他端口连接服务器的随机端口。因为服务器在被动模式下没有打开临时端口让 client 连过来，因此需要几个条件：
	
		* client 没有防火墙时，用主动模式连接即可。
		* server 没有防火墙时，用被动模式即可。
		* 双方都有防火墙时，vsftpd 设置被动模式高端口范围，server 打开那段范围，client 用被动模式连接即可。
		* 加载 ip_conntrack_ftp 模块，使 server 支持 connection tracking，支持临时打洞，client 用被动模式即可。
		* server 使用 ip_conntrack_ftp、client 使用 ip_conntrack_ftp 和 ip_nat_ftp，支持临时打洞和临时 NAT 穿越打洞，双方使用主动或被动模式均可。
		
因为我 client 和 server 都有防火墙，所以对前边的 vsftpd 设置稍做修改

	#pasv_enable=NO
	pasv_min_port=2222
	pasv_max_port=2322
	
修改 iptables 

	-A INPUT -p tcp --dport 2222:2322 -j ACCEPT
	
重启

	# service iptables restart
	# service vsftpd restart				

				
