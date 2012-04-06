---
layout: post
title: "在 RHEL6 上安装 OpenAM"
category: 
tags: 
- linux
- sso
---

由于参加 Cloud Foundry 和清明假期，这个系列中断了几天。从今天开始继续。
官网“provides the community with a new home for Sun Microsystems' OpenSSO product.”
和 [CAS](http://www.jasig.org/cas) 类似，也是 SSO 的一个实现。本人也简单玩过 CAS，相对来讲，
OpenAM 提供了图形安装界面，CAS 基本都需要修改配置文件。并且 OpenAM 对应用系统的侵入性可能更小一些。 

这里使用了 [Nightly Build](http://download.forgerock.org/downloads/openam/openam_20120228.war)，因为
9 以前的版本默认还不支持 OpenDJ。具体的安装过程可以看 [Installing OpenAM Core Services](http://openam.forgerock.org/doc/install-guide/index/chap-install-core.html)，
这里只写一些关键点。

在安装之前，需要设置 JVM 参数，否则会报 OOM。主要是 PermSize 的问题。在 catalina.sh 中加入

	export JAVA_OPTS='-Xms1024m -Xmx2048m -XX:PermSize=256m -XX:MaxPermSize=256m'
	
准备 war 文件	

	# cp openam_20120228.war /opt/tomcat6/webapps
	# mv openam_20120228.war openam.war
	
在 OpenDJ 中建立 Base DN

![](/images/2012-04-05-install-openam-on-rhel6-1.png)
	
这里是测试环境，没有 DNS，所以需要修改 hosts。OpenDJ 和 OpenAM 都安装到 openam.example.com，OpenDJ 监听在 1389 端口。
强烈建议生产环境使用 DNS，因为在配置中需要多次用到域名属性。

这时可以启动 tomcat，访问 http://openam.example.com:8080/openam 进行配置。

在安装过程中，如果遇到错误，查看 install.log，尝试删除 OpenDJ 中两个 Base DN 下的所有子条目，并执行以下命令重新配置 OpenAM

	rm -rf $HOME/openam $HOME/.openssocfg
	
Step 2 	

![](/images/2012-04-05-install-openam-on-rhel6-2.png)

Step 3 这步发现一个 OpenAM 本身的问题，OpenAM 本身支持多国语言，在客户端系统是中文环境和英文环境下，	在 `Step 3` 中显示的内容稍有不同，
在中文环境下不显示 `OpenDJ` 选项(可能是 OpenAM 版本问题)。	

![](/images/2012-04-05-install-openam-on-rhel6-3.png)
	
Step 4

![](/images/2012-04-05-install-openam-on-rhel6-4.png)

OpenDJ

![](/images/2012-04-05-install-openam-on-rhel6-5.png)

配置完成后，使用 amadmin/password 登录系统。

如果要完成后边的 Sample，需要下载 [Example.ldif](http://mcraig.org/ldif/Example.ldif) 导入 OpenDJ。
还需要下载 [Nightly Build](http://download.forgerock.org/downloads/openam/openam_nightly_20120228.zip)，
在这个包中，会有一些附加工具和示例应用。