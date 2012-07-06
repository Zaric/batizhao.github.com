---
layout: post
title: "自定义 CAS 主题"
category: java 
tags: 
- sso
- cas
---
{% include JB/setup %}

这篇主要讲 CAS 自定义登录页面，主题名称叫作：cas-theme-twitter-bootstrap，是以 twitter bootstrap 为基础。

## 在 classes 目录下增加属性文件

cas-theme-twitter-bootstrap.properties 配置主题的路径

	standard.custom.css.file=themes/cas-theme-twitter-bootstrap/css/bootstrap.css

twitter-bootstrap-views.properties 配置 jsp 的路径，这里只修改了登录页面。

	casLoginView.url=/WEB-INF/view/jsp/twitter-bootstrap/casLoginView.jsp

## 修改 cas.properties

指向新的属性文件，名称和上边的文件名对应。

	cas.themeResolver.defaultThemeName=cas-theme-twitter-bootstrap
	cas.viewResolver.basename=twitter-bootstrap-views

## 新建对应的目录

* themes/cas-theme-twitter-bootstrap 放入 css 和 image。
* WEB-INF/view/jsp/twitter-bootstrap 根据默认的 jsp 修改相应的代码。

如果需要修改界面文字，从 target 下边复制 messages_zh_CN.properties 文件。这些文件在 mvn package 后会
相应的增加或者替换到 CAS 的 war 包中。在开发环境可以使用 mvn tomcat6:run 直接进行测试。
完整代码在 [Github](https://github.com/batizhao/custom-cas/tree/master/server-ldap)。