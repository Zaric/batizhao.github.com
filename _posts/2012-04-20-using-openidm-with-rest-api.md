---
layout: post
title: "使用 OpenIDM RESTful API"
category: java
tags: 
- java
- sso
- ldap
---
{% include JB/setup %}

## 1. 使用默认的类型 user

在安装完成以后，使用下边的方法查看 OpenIDM 仓库的中所有用户

	# curl \
	--header "X-OpenIDM-Username: openidm-admin" \
	--header "X-OpenIDM-Password: openidm-admin" \
	http://openam.example.com:9090/openidm/managed/user/?_query-id=query-all-ids
	
返回的 JSON 显示结果为空。

	{"query-time-ms":1,"result":[]}
	
增加用户

	# curl \
	--header "X-OpenIDM-Username: openidm-admin" \
	--header "X-OpenIDM-Password: openidm-admin" \
	--request PUT \
	--data '{ "userName":"joe", "givenName":"joe", "familyName":"smith", "email":["joe@example.com"], "description":"My first user" }' \
	http://openam.example.com:9090/openidm/managed/user/joe
	
返回的 JSON 结果

	{"_id":"joe","_rev":"0"}
	
查询新增加的用户

	# curl \
	--header "X-OpenIDM-Username: openidm-admin" \
	--header "X-OpenIDM-Password: openidm-admin" \
	http://openam.example.com:9090/openidm/managed/user/joe
	
返回的 JSON 结果

	{
	 "_rev":"0",
	 "_id":"joe",
	 "email":["joe@example.com"],
	 "description":"My first user",
	 "familyName":"smith",
	 "userName":"joe",
	 "givenName":"joe"
	}

Java 代码在 [Github](https://github.com/batizhao/openam-java-sample/tree/master/idm-client) 。

## 2. 增加新的类型 organization

在 openidm/conf/managed.json 中增加

	{
        "name" : "organization"
    }

查询 organization
    
    # curl \
	--header "X-OpenIDM-Username: openidm-admin" \
	--header "X-OpenIDM-Password: openidm-admin" \
	http://openam.example.com:9090/openidm/managed/organization/?_query-id=query-all-ids
	
结果

	{"query-time-ms":6,"result":[]}
	
增加 organization

	# curl \
	--header "X-OpenIDM-Username: openidm-admin" \
	--header "X-OpenIDM-Password: openidm-admin" \
	--request PUT \
	--data '{ "name":"DEV", "description":"Dev Organization" }' \
	http://openam.example.com:9090/openidm/managed/organization/dev
	
返回结果

	{"_id":"dev","_rev":"0"}
	
查询新增加的 organization

	# curl \
	--header "X-OpenIDM-Username: openidm-admin" \
	--header "X-OpenIDM-Password: openidm-admin" \
	http://openam.example.com:9090/openidm/managed/organization/dev
	
返回结果

	{
	 "_rev":"0",
	 "_id":"dev",
	 "description":"Dev Organization",
	 "name":"DEV"
	}					