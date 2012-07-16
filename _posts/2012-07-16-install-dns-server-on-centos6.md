---
layout: post
title: "在 CentOS6 上安装 bind（DNS Server）"
category: linux
tags: 
- linux
- dns
---
{% include JB/setup %}

## 1. DNS 安装

方式一：从官方下载最新的 Release 版本编译安装（生产环境推荐，后边的配置步骤也会以这种方式说明，和 yum 安装的路径不同）

在安装之前需要先安装 gcc

	# yum install gcc.x86_64 gcc-c++.x86_64 gcc-objc++.x86_64
	
还需要有 openssl

	# yum install openssl openssl-devel
	
下载并安装

	# wget http://ftp.isc.org/isc/bind9/9.9.1-P1/bind-9.9.1-P1.tar.gz
	# tar -zxvf bind-9.9.1-P1.tar.gz
	# cd bind-9.9.1-P1
	# ./configure --enable-largefile --enable-threads --prefix=/usr/local/named
	# make
	# make install		

查询版本号
	
	# /usr/local/named/sbin/named -v
	----
	BIND 9.9.1-P1	

方式二：使用 yum 安装

	# yum -y install bind* caching-nameserver
	
查询版本号

	# named -v
	BIND 9.8.2rc1-RedHat-9.8.2-0.10.rc1.el6		
	
## 2. DNS 配置

安装 RNDC，让其管理 bind

	# cd /usr/local/named/etc
	# /usr/local/named/sbin/rndc-confgen > /usr/local/named/etc/rndc.conf
	# tail -n10 rndc.conf |head -n9 |sed -e s/#\//g > named.conf	
	
更新 Internet 根服务器地址
		
	# cd /usr/local/named/
	# wget ftp://ftp.internic.org/domain/named.root	
		
配置 named.conf 文件，这是 bind 的主配置文件，最终的内容	
	
	# mkdir -p /usr/local/named/data
	# cd etc
	# vim named.conf
	
刚才	tail 命令时已经把 rndc.conf 的一部分内容加进来，现在再在前边加入以下内容

	options {
        directory "/usr/local/named";
        pid-file "named.pid";
        listen-on port 53 {any;};
        allow-query {any;};
        dump-file "/usr/local/named/data/cache_dump.db";
        statistics-file "/usr/local/named/data/named_stats.txt";
        forward only;               //增加转发功能
        forwarders {
                8.8.8.8;
        };

	};
	
	zone "."  IN {
        type hint;
        file "named.root";
	};
	
	zone "localhost" IN {
			 type master;
			 file "localhost.zone";
			 allow-update { none; };
	};
	
	zone "0.0.127.in-addr.arpa" IN {
			 type master;
			 file "localhost.rev";
			 allow-update { none; };
	};
	
	zone "dev.org" IN {
			type master;
			file "dev.org.zone";
	};
	
	zone "247.4.10.in-addr.arpa" IN {
			type master;
			file "10.4.247.zone";			
	};

生成域名相应的配置文件
	
	# cd /usr/local/named
	
localhost 正向解析文件
	
	# vim localhost.zone
	----
	$TTL 3600
	@    IN SOA  @    root (
                  20100923       ;serial (d. adams)
                  3H             ;refresh
                  15M            ;retry
                  1W             ;expiry
                  3600)          ;minimum
     IN NS   @
     IN A    127.0.0.1
    
localhost 反向解析文件

	# vim localhost.rev
	----
	$TTL 3600
	@   IN SOA   localhost.   root.localhost. (
				 20100923      ; serial
				 3600          ; refresh every hour
				 900           ; retry every 15 minutes
				 3600000       ; expire 1000 hours
				 3600)         ; minimun 1 hour
		 IN NS  localhost.
	1    IN PTR localhost.   
	
dev.org 正向解析文件

	# vim dev.org.zone
	----
	$TTL 86400
	@       IN SOA  dns.dev.org root (
											0       ; serial
											1D      ; refresh
											1H      ; retry
											1W      ; expire
											3H )    ; minimum
	@       IN      NS      dns.dev.org.
	cas     IN      A      10.4.247.20
	dns     IN      A       10.4.247.20
	ldap     IN      A       10.4.247.20
	
dev.org 反向解析文件

	# vim 10.4.247.zone
	----
	$TTL 86400
	@       IN SOA  dns.dev.org root (
											0       ; serial
											1D      ; refresh
											1H      ; retry
											1W      ; expire
											3H )    ; minimum
	@       IN      NS      dns.dev.org.
	cas     IN      A       10.4.247.20
	dns     IN      A       10.4.247.20
	ldap     IN     A       10.4.247.20
	20      IN      PTR     cas.dev.org.
	20      IN      PTR     dns.dev.org.
	20      IN      PTR     ldap.dev.org.

## 3. 测试

启动bind

	# /usr/local/named/sbin/named -gc /usr/local/named/etc/named.conf &
	----
	16-Jul-2012 10:47:03.896 ----------------------------------------------------
	16-Jul-2012 10:47:03.896 BIND 9 is maintained by Internet Systems Consortium,
	16-Jul-2012 10:47:03.896 Inc. (ISC), a non-profit 501(c)(3) public-benefit 
	16-Jul-2012 10:47:03.896 corporation.  Support and training for BIND 9 are 
	16-Jul-2012 10:47:03.896 available at https://www.isc.org/support
	16-Jul-2012 10:47:03.896 ----------------------------------------------------
	16-Jul-2012 10:47:03.896 adjusted limit on open files from 4096 to 1048576
	16-Jul-2012 10:47:03.896 found 16 CPUs, using 16 worker threads
	16-Jul-2012 10:47:03.896 using 16 UDP listeners per interface
	16-Jul-2012 10:47:03.897 using up to 4096 sockets
	16-Jul-2012 10:47:03.903 loading configuration from '/usr/local/named/etc/named.conf'
	16-Jul-2012 10:47:03.903 reading built-in trusted keys from file '/usr/local/named/etc/bind.keys'
	16-Jul-2012 10:47:03.904 using default UDP/IPv4 port range: [1024, 65535]
	16-Jul-2012 10:47:03.904 using default UDP/IPv6 port range: [1024, 65535]
	16-Jul-2012 10:47:03.905 listening on IPv4 interface lo, 127.0.0.1#53
	16-Jul-2012 10:47:03.911 listening on IPv4 interface em1, 10.4.247.20#53
	16-Jul-2012 10:47:03.916 generating session key for dynamic DNS
	16-Jul-2012 10:47:03.916 sizing zone task pool based on 5 zones
	16-Jul-2012 10:47:03.919 set up managed keys zone for view _default, file 'managed-keys.bind'
	16-Jul-2012 10:47:03.922 command channel listening on 127.0.0.1#953
	16-Jul-2012 10:47:03.922 ignoring config file logging statement due to -g option
	16-Jul-2012 10:47:03.922 managed-keys-zone: loaded serial 0
	16-Jul-2012 10:47:03.923 zone 0.0.127.in-addr.arpa/IN: loaded serial 20100923
	16-Jul-2012 10:47:03.924 zone localhost/IN: has no NS records
	16-Jul-2012 10:47:03.924 zone localhost/IN: not loaded due to errors.
	16-Jul-2012 10:47:03.924 zone 247.4.10.in-addr.arpa/IN: loaded serial 0
	16-Jul-2012 10:47:03.924 zone dev.org/IN: loaded serial 0
	16-Jul-2012 10:47:03.925 all zones loaded
	16-Jul-2012 10:47:03.925 running

修改本机的 DNS 设置（如果修改 resolv.conf 的话重启以后会失效）
	
	#vim /etc/sysconfig/network-scripts/ifcfg-em1
	DNS1=10.4.247.20

安装 nslookup 工具                	

	# yum install bind-utils
	# nslookup
	> dns.dev.org
	Server:		10.4.247.20
	Address:	10.4.247.20#53
	
	Name:	dns.dev.org
	Address: 10.4.247.20
	> cas.dev.org
	Server:		10.4.247.20
	Address:	10.4.247.20#53
	
	Name:	cas.dev.org
	Address: 10.4.247.20
	> ldap.dev.org
	Server:		10.4.247.20
	Address:	10.4.247.20#53
	
	Name:	ldap.dev.org
	Address: 10.4.247.20
	> 10.4.247.20
	Server:		10.4.247.20
	Address:	10.4.247.20#53
	
	20.247.4.10.in-addr.arpa	name = dns.dev.org.
	20.247.4.10.in-addr.arpa	name = ldap.dev.org.
	20.247.4.10.in-addr.arpa	name = cas.dev.org.
	
最后需要把 bind 加入到启动项，随操作系统一起启动

	# cd /etc/rc.d
	# vim rc.local
	
在最后添加
	
	# /usr/local/named/sbin/named -gc /usr/local/named/etc/named.conf &				 