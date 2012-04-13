---
layout: post
title: "安装 Tomcat Policy Agent"
category: 
tags:
- linux
- sso
- ldap
---

本文参照 [Installing the Apache Tomcat Policy Agent](http://openam.forgerock.org/doc/agent-install-guide/index.html#chap-apache-tomcat)，在开始之前必须停止 Tomcat。

## 在 OpenAM 中配置 Tomcat Agent

登录 OpenAM，Access Control - Top Level Realm - Agents - J2EE，在 Agent 下边点击按钮 New

	Name: tomcatAgent
	Password: 123456
	Configuration: Centralized
	Server URL: http://openam.example.com:8080/openam
	Agent URL: http://website.example.com:8080/agentapp

## 安装 Tomcat Policy Agent

创建密码文件（在需要配置 Agent 的机器）

	# umask 366
	# echo 123456 > /var/tmp/passwd

安装

	# wget http://download.forgerock.org/downloads/openam/j2eeagents/stable/3.0.3/tomcat_v6_agent_303.zip 
	# cp tomcat_v6_agent_303.zip /opt/tomcat6
	# cd /opt/tomcat6
	# unzip tomcat_v6_agent_303.zip
	# j2ee_agents/tomcat_v6_agent/bin/agentadmin --install
	
	Tomcat Server Config Directory : /opt/tomcat6/conf
	OpenSSO server URL : http://openam.example.com:8080/openam
	$CATALINA_HOME environment variable : /opt/tomcat6
	Tomcat global web.xml filter install : false
	Agent URL : http://website.example.com:8080/agentapp
	Agent Profile name : tomcatAgent
	Agent Profile Password file name : /var/tmp/passwd
	
拷贝示例程序

	# cp j2ee_agents/tomcat_v6_agent/etc/agentapp.war webapps
	# cp j2ee_agents/tomcat_v6_agent/sampleapp/dist/agentsample.war webapps
	
如果是同一个浏览器，先注销 OpenAM，启动 Tomcat，访问 agentsample，会被重定向到 OpenAM 的登录页面，这时使用 amadmin/password 登录是有问题的。

## 在 OpenAM 中配置 Policy

再次登录 OpenAM，Access Control - Top Level Realm - Subjects，在 User 下边点击 New，新建 4 个测试用户

	ID : user001
	First Name : User
	Last Name : One
	Full Name : User One
	Password : firstuser
	Confirm Password : firstuser
	User Status : Active
	
	ID : user002
	First Name : User
	Last Name : Two
	Full Name : User Two
	Password : seconduser
	Confirm Password : seconduser
	User Status : Active
	
	ID : user003
	First Name : User
	Last Name : Three
	Full Name : User Three
	Password : thirduser
	Confirm Password : thirduser
	User Status : Active
	
	ID : user004
	First Name : User
	Last Name : Four
	Full Name : User Four
	Password : fourthuser
	Confirm Password : fourthuser
	User Status : Active
	
![](/images/2012-04-06-installing-the-apache-tomcat-policy-agent-1.png)	

Access Control - Top Level Realm - Policies，点击 New Policy，在 Rules 下边点击 New，选择 URL Policy Agent

	Name: URLPolicy
	Resource Name: http://website.example.com:8080/agentsample/*
	Actions : Select and allow both GET and POST
	
在 Subjects	下边点击 New，选择 OpenAM Identity Subject（如果选择 Authenticated Users，不限制用户）

	Name: userAccess
	Exclusive : Not ticked
	
![](/images/2012-04-06-installing-the-apache-tomcat-policy-agent-2.png)	
	
在 New Policy 的 General 

	Name：samplePolicy
	
点击 Save.	

![](/images/2012-04-06-installing-the-apache-tomcat-policy-agent-3.png)

如果是同一个浏览器，先注销 OpenAM，访问 agentsample，会被重定向到 OpenAM 的登录页面，这时使用 user001，user002 可以正常登录，使用其它用户不可以。