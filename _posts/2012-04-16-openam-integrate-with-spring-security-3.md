---
layout: post
title: "开发 OpenAM Spring Security 3 客户端应用"
category: java
tags: 
- java
- sso
- ldap
---
{% include JB/setup %}

在开始 Maven 之前，需要先引入一个包，这个包的作用和原来的 Agent 的功能是一样的。这个包需要自己从[源码](http://sources.forgerock.org/browse/openam/trunk/opensso/extensions/spring2provider/provider?r=HEAD)编译，
`mvn install` 或者 `mvn deploy` 加入到自己的仓库中。这样在 pom.xml 中可以引入

	<dependency>
		<groupId>com.sun.identity.provider</groupId>
		<artifactId>opensso-springsecurity</artifactId>
		<version>0.2</version>
	</dependency>
	
确保 AMConfig.properties 和 applicationContext-security.xml 里的 OpenAM 相关配置正确。

	com.sun.identity.loginurl=http://openam.example.com:8080/openam/UI/Login
	
然后运行 `mvn package`，打包以后放到 Tomcat 运行。这里要确认没有配置 Tomcat Agent 
的全局 OpenAM Filter，这里也不需要在项目的 web.xml 中增加 Filter 配置。
完整的代码在 [Github](https://github.com/batizhao/openam-java-sample)。


	
	
