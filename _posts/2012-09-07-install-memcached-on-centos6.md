---
layout: post
title: "在 CentOS6 上安装 memcached"
category: 
tags: 
- linux
- memcached
---
{% include JB/setup %}

# 1. 安装 libevent

	# yum list|grep libevent*
	
	libevent.x86_64                        1.4.13-4.el6                     
	libevent.i686                          1.4.13-4.el6                      
	libevent-devel.i686                    1.4.13-4.el6                         
	libevent-devel.x86_64                  1.4.13-4.el6                        
	libevent-doc.noarch                    1.4.13-4.el6                        
	libevent-headers.noarch                1.4.13-4.el6                     

	# yum install libevent.x86_64 libevent-devel.x86_64

# 2. 安装 memcached
	
	# wget http://memcached.googlecode.com/files/memcached-1.4.15.tar.gz
	# tar zxvf memcached-1.4.15.tar.gz
	# cd memcached-1.4.15
	# ./configure
	# make
	# make install

# 3. 使用 memcached

参数

	-p 监听端口
	-l 连接的IP地址,默认是本机
	-d start 启动 memecached 服务
	-d restart 重启
	-d stop|shutdown关闭服务
	-d install 安装
	-d uninstall 卸载
	-u 以身份运行仅在 root 下有效
	-m 最大内存使用，单位 MB，默认 64MB
	-M 内存耗尽时返回错误
	-c 最大同时连接数量,默认是 1024
	-f 块大小增长因为,默认是 1.25
	-n 最小分配空间, key + value + flags 默认48
	-h 显示帮助

启动
	
	# memcached -d -u root
	
验证

	＃ telnet localhost 11211
	Trying ::1...
	Connected to localhost.
	Escape character is '^]'.
	
查看当前状态

	＃ stats
	TAT pid 26427
	STAT uptime 2137
	STAT time 1346995532
	STAT version 1.4.15
	STAT libevent 1.4.13-stable
	STAT pointer_size 64
	STAT rusage_user 0.012998
	STAT rusage_system 0.023996
	STAT curr_connections 10
	STAT total_connections 14
	STAT connection_structures 12
	STAT reserved_fds 20
	STAT cmd_get 0
	STAT cmd_set 0
	STAT cmd_flush 0
	STAT cmd_touch 0
	STAT get_hits 0
	STAT get_misses 0
	STAT delete_misses 0
	STAT delete_hits 0
	STAT incr_misses 0
	STAT incr_hits 0
	STAT decr_misses 0
	STAT decr_hits 0
	STAT cas_misses 0
	STAT cas_hits 0
	STAT cas_badval 0
	STAT touch_hits 0
	STAT touch_misses 0
	STAT auth_cmds 0
	STAT auth_errors 0
	STAT bytes_read 68
	STAT bytes_written 2091
	STAT limit_maxbytes 67108864
	STAT accepting_conns 1
	STAT listen_disabled_num 0
	STAT threads 4
	STAT conn_yields 0
	STAT hash_power_level 16
	STAT hash_bytes 524288
	STAT hash_is_expanding 0
	STAT bytes 0
	STAT curr_items 0
	STAT total_items 0
	STAT expired_unfetched 0
	STAT evicted_unfetched 0
	STAT evictions 0
	STAT reclaimed 0
	END
	
远程使用需要打开 iptables 端口

	# vim /etc/sysconfig/iptables
	----
	-A INPUT -m state --state NEW -m tcp -p tcp --dport 11211 -j ACCEPT
	-A INPUT -m state --state NEW -m udp -p udp --dport 11211 -j ACCEPT  

	# service iptables restart   
