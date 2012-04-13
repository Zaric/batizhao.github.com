---
layout: post
title: "开发 OpenAM Java 客户端应用"
category: java
tags: 
- java
- sso
- ldap
---
{% include JB/setup %}

在 Agent 安装完成之后，可以使用自带的 agentsample 应用登录。这里主要讲一下如何在 SSO
之后拿到 SSOToken，以及相关 Session 信息的获取。完整的代码在 [Github](https://github.com/batizhao/openam-java-sample)。

Agent 的安装在上一篇已经介绍，这里需要先配置一个 Policies，然后在客户端项目 web.xml 中加入

	<filter>
        <filter-name>Agent</filter-name>
        <display-name>Agent</display-name>
        <description>SJS Access Manager Tomcat Policy Agent Filter</description>
        <filter-class>com.sun.identity.agents.filter.AmAgentFilter</filter-class>
    </filter>

    <filter-mapping>
        <filter-name>Agent</filter-name>
        <url-pattern>/*</url-pattern>
        <dispatcher>REQUEST</dispatcher>
        <dispatcher>INCLUDE</dispatcher>
        <dispatcher>FORWARD</dispatcher>
        <dispatcher>ERROR</dispatcher>
    </filter-mapping>
    
在项目中获取 token 相关内容

	SSOTokenManager manager = SSOTokenManager.getInstance();
    SSOToken token = manager.createSSOToken(request);

    if (manager.isValidToken(token)) {
        java.security.Principal principal = token.getPrincipal();

        out.println("SSOToken Principal name: " + principal.getName());        
        out.println("<br />");
    }
    
分别部署两个相同的应用到 Tomcat，取名 Client1，Client2。访问任意应用，另外一个应用也自动登录。

![](/images/2012-04-12-using-openam-develop-dlient-applications.png)

## 参考
* [OpenAM 10.0.0 Developer's Guide](http://openam.forgerock.org/doc/dev-guide/index.html#chap-client-dev)
* [OpenAM 10.0.0 Installation Guide](http://openam.forgerock.org/doc/install-guide/index.html)
