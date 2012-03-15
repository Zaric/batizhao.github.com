---
layout: post
title: 在 RHEL6 上安装 phpLDAPAdmin
category: linux
tags:
- linux
- ldap
---

配置 Apache：

{% highlight bash %}
# vim /etc/httpd/conf/httpd.conf
{% endhighlight %}

加入如下设置：

    AddType application/x-httpd-php .php
    AddType application/x-httpd-php-source .phps


安装 phpLDAPAdmin：

	# tar xzvf phpldapadmin-1.2.2.tgz
	# mv phpldapadmin-1.2.2 phpldapadmin
	# cp -R phpldapadmin /var/www/html

配置 phpLDAPAdmin：

	# cd /var/www/html/phpldapadmin/config
	# cp config.php.example config.php
	# vim config.php

	# $servers->setValue('server','host','127.0.0.1');
	# $servers->setValue('server','port',389);
	# $servers->setValue('server','base',array('dc=my-domain,dc=com'));
	# $servers->setValue('login','auth_type','cookie');
	# $servers->setValue('login','bind_id','cn=Manager,dc=my-domain,dc=com');

配置 openLDAP：

	# cd /etc/openldap/slapd.d/cn=config
	# vim olcDatabase={2}bdb.ldif

	增加属性：
	olcRootPW: 123456

	# service slapd restart

访问 http://localhost/phpldapadmin
