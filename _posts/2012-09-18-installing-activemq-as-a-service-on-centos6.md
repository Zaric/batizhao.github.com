---
layout: post
title: "在 CentOS6 上安装 ActiveMQ Service"
category: java
tags: 
- linux
- java
- jms
- activemq
---
{% include JB/setup %}

这里主要讲一下在 CentOS6 x64 上如何把 ActiveMQ5.5 添加到 Service 里自动启动。

打开配置文件

	# vim /opt/activemq	/bin/linux-x86-64/wrapper.conf

修改两个参数
	
	set.default.ACTIVEMQ_HOME=/opt/activemq
    set.default.ACTIVEMQ_BASE=/opt/activemq
    
建立一个软链接（用全路径）

	# ln -s /opt/activemq/bin/linux-x86-64/activemq /etc/init.d/activemq    
	
加入到启动项

	# chkconfig --add activemq
	
使用服务

	# service activemq start|stop|status	
