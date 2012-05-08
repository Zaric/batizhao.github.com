---
layout: post
title: "统一身份管理和单点登录"
category: java
tags: 
- sso
- ldap
---
{% include JB/setup %}

![](/images/2012-05-08-identity-management.png)

## 1. 统一身份管理和单点登录平台

* 调用 OpenIDM REST API 管理组织、用户（CRUD，需要开发）。
* 通过 OpenIDM Core Service 推送数据到 OpenDJ（LDAP）、业务系统数据库。
* 通过 OpenAM 调用 OpenDJ 完成单点认证（SSO）。
* 如果控制要求不是很复杂，可以使用 OpenAM 完成统一的业务系统授权和访问控制。否则可以自己开发 OpenAM 授权插件或者业务系统自己实现。

## 2. 对于新的业务系统

* 业务系统需要有组织、用户表用来同步数据。
* 业务系统不需要单独的管理组织、用户模块，可以方便的使用统一的组织、用户数据。

## 3. 对于遗留业务系统

* 建立遗留业务系统和底层身份管理平台的映射关系。
* 屏蔽遗留业务系统的认证模块。
* 安装 OpenAM Agent 或者使用 SDK 开发简单的认证接口。可参考
[开发 OpenAM Spring Security 3 客户端](http://batizhao.github.com/java/2012/04/16/openam-integrate-with-spring-security-3/)
和
[开发 OpenAM Java 客户端](http://batizhao.github.com/java/2012/04/12/using-openam-develop-dlient-applications/)。
