---
layout: post
title: 在 RHEL6 上安装 phpLDAPAdmin
category: linux
tags:
- linux
- ldap
---

安装 php：

	# yum install php php-ldap
	
配置 Apache：

    # vim /etc/httpd/conf/httpd.conf

加入如下设置：

    AddType application/x-httpd-php .php
    AddType application/x-httpd-php-source .phps

安装 phpLDAPAdmin：

	# tar xzvf phpldapadmin-1.2.2.tgz
	# mv phpldapadmin-1.2.2 phpldapadmin
	# cp -R phpldapadmin /var/www/html

打开 phpLDAPAdmin 配置文件：

	# cd /var/www/html/phpldapadmin/config
	# cp config.php.example config.php
	# vim config.php

找到以下内容，去掉注解，修改参数：

	$servers->setValue('server','host','127.0.0.1');
	$servers->setValue('server','port',389); 
	$servers->setValue('server','base',array('dc=my-domain,dc=com')); #olcSuffix
	$servers->setValue('login','auth_type','cookie');
	$servers->setValue('login','bind_id','cn=Manager,dc=my-domain,dc=com'); #olcRootDN
	
修改配置文件：

	# vim /etc/openldap/slapd.d/cn=config/olcDatabase={2}bdb.ldif

修改 olcSuffix,olcRootDN，对应上边的配置，增加 olcRootPW（登录密码）。
	
打开防火墙的 389 端口，启动 OpenLDAP：	
	
	# service slapd start
	
打开防火墙的 80 端口，启动 Apache：

	# service httpd start

访问 http://localhost/phpldapadmin
