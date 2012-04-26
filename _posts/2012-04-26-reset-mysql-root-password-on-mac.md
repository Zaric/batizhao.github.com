---
layout: post
title: "重置 MySQL root 密码"
category: mysql
tags: 
- mysql
---
{% include JB/setup %}

停止 MySQL 服务，运行

	# mysqld_safe --skip-grant-tables >/dev/null 2>&1 &
	# mysql -u root mysql
	mysql> update user set password = Password('new password') where User = 'root';
	mysql> flush privileges;
	mysql> exit;
	# killall mysqld;
	
启动 MySQL 服务，运行

	# mysql -r root -p
	Enter password:new password
	Welcome to the MySQL monitor.  Commands end with ; or \g.
	Your MySQL connection id is 402
	Server version: 5.5.16 MySQL Community Server (GPL)
	
	Copyright (c) 2000, 2011, Oracle and/or its affiliates. All rights reserved.
	
	Oracle is a registered trademark of Oracle Corporation and/or its
	affiliates. Other names may be trademarks of their respective
	owners.
	
	Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
	
	mysql>

	
	
	
