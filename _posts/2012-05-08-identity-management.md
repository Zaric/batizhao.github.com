---
layout: post
title: "统一身份管理和单点登录"
category: java
tags: 
- sso
- ldap
---
{% include JB/setup %}

## 1. 方案一

* 调用 OpenIDM REST API 管理组织、用户（CRUD，需要开发）。
* 通过 OpenIDM Core Service 推送数据到 OpenDJ（LDAP）、业务系统数据库。
* 需要至少提供两台 Linux（RHEL6）主机，增加的系统服务有 OpenDJ、OpenIDM、OpenAM、MySQL、身份管理 Web 服务。
* OpenDJ 和 MySQL 在一台机器（做数据备份），其余在另外一台。OpenDJ 和 OpenAM 交叉 HA。

![](/images/2012-05-08-identity-management-1.png)

优点：

* 前期开发量比较小，见效快。
* 增加新的数据类型非常方便。
* 同步的功能很强大，不需要客户端开发同步的代码，只需要在服务端写配置文件。

缺点：

* 后期需要根据不同的业务系统在 OpenIDM 中进行数据映射（需要了解业务系统相关的数据结构）。
* 随着系统的加入，OpenIDM 的维护工作会越来越重。
* 不太方便第三方系统测试、调试同步机制（这个工作需要在服务端完成）。

## 2. 方案二

* 抛弃 OpenIDM，直接管理 OpenDJ 的数据（CRUD，需要开发）。
* 增加一个 MQ Server，当在 OpenDJ 中增加数据时，发一条 message 给 MQ Server（消息中组织、用户的数据用 JSON 封装）。
* 业务系统需要实现一个 Client，message 会被即时推送给业务系统（开发量非常小）。
* 业务系统收到消息后自行处理。如果处理中出现异常，建议把消息内容先存下来再稍后处理。
* 如果需要保证数据的一致性，在 MQ 中启动 acknowledgment。
* 服务端需要提供一个查询所有组织、用户的接口，用于系统第一次对接。
* 需要至少提供一到两台 Linux（RHEL6）主机，增加的系统服务有 OpenDJ、OpenAM、MQ、身份管理 Web 服务。
* OpenDJ 和 MQ 在一台机器，其余在另外一台。OpenDJ 和 OpenAM 交叉 HA。

![](/images/2012-05-08-identity-management-2.png)

优点：

* 灵活性增加。
* 服务端不用再考虑和了解业务系统的数据结构。
* 服务端和业务系统异步交互，服务端的后期工作大大降低。
* 不需要安装 OpenIDM 和 MySQL 两个服务。
* 因为不直接开放给业务系统调用（身份管理），几乎不用考虑服务端的性能。

缺点：

* 相对方案一，服务端和客户端开发量都增加。
* 服务端可能还需要开发一个 SDK 给客户端调用。
* 需要服务端自己开发管理 OpenDJ（LDAP） 的代码（相对通过调用 OpenIDM REST API 工作量要大）。
* 需要在服务端部署一个 MQ Server。
* 需要客户端实现一个 MQ Client ，并且自己处理组织、用户数据。

## 3. 都需要注意的

* 图中只关注了数据同步的数据流向，没有体现 App 和 SSO 的关系。
* 客户端需要有组织、用户表用来同步数据。
* 对于 OpenDJ（LDAP），OpenAM（SSO），一定要考虑 HA。要求高的最好在不同的机房。所以物理机至少是两台。
* 要考虑机器配置冗余，每个服务至少要分配 2-3G 内存。
* 需要有自己的 DNS Server。
* 如果对 SSO 的安全性要求很高，建议使用 HTTPS 协议，需要有 SSL 证书。自行签发的证书会影响用户体验。

## 4. 对于新的业务系统

* 不需要单独的管理组织、用户模块，可以方便的使用统一的组织、用户数据。
* 不需要实现用户认证功能，只需要和 OpenAM 集成。

## 5. 对于遗留业务系统

* 如果不能重新初始化组织、用户数据，那么需要建立遗留业务系统和底层身份管理平台的映射关系。
* 屏蔽遗留业务系统的认证模块。
* 如果权限控制要求不是很复杂，可以使用 OpenAM 完成统一的业务系统授权和访问控制。否则可以自己开发 OpenAM 授权插件或者业务系统自己实现。
* 安装 OpenAM Agent 或者使用 SDK 开发简单的认证接口。可参考
[开发 OpenAM Spring Security 3 客户端](http://batizhao.github.com/java/2012/04/16/openam-integrate-with-spring-security-3/)
和
[开发 OpenAM Java 客户端](http://batizhao.github.com/java/2012/04/12/using-openam-develop-dlient-applications/)。
