---
layout: post
title: "在 RHEL6 上安装 gcc"
category: linux
tags: 
- linux
- gcc
---

默认情况下，RHEL 不会安装 gcc，在编译安装软件时会遇到 `configure: error: no acceptable C compiler found in $PATH` 的问题。

先查找名称：

	# yum list|grep gcc
	
安装：

	# yum install gcc.x86_64 gcc-c++.x86_64 gcc-objc++.x86_64
