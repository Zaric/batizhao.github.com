---
layout: post
title: "进阶使用 OpenIDM 【二】"
category: java
tags: 
- sso
- ldap
---
{% include JB/setup %}

在正式的企业场景中，组织一般具有父子关系。我们来看一下在这种情况下，OpenIDM 如何配置。

## 1. 增加新的类型 organizationUnit

在 openidm/conf/managed.json 中增加

	{
        "name" : "organizationUnit"
    }

查询 organizationUnit
    
    # curl \
	--header "X-OpenIDM-Username: openidm-admin" \
	--header "X-OpenIDM-Password: openidm-admin" \
	http://openam.example.com:9090/openidm/managed/organizationUnit/?_query-id=query-all-ids
	
结果

	{"query-time-ms":6,"result":[]}
	
增加 organizationUnit

	# curl \
	--header "X-OpenIDM-Username: openidm-admin" \
	--header "X-OpenIDM-Password: openidm-admin" \
	--request PUT \
	--data '{ "name":"ideal", "dn":"ou=ideal,o=shanghai,dc=example,dc=com", "description":"ideal company" }' \
	http://openam.example.com:9090/openidm/managed/organizationUnit/ideal
	
返回结果

	{"_id":"ideal","_rev":"0"}
	
查询新增加的 organizationUnit

	# curl \
	--header "X-OpenIDM-Username: openidm-admin" \
	--header "X-OpenIDM-Password: openidm-admin" \
	http://openam.example.com:9090/openidm/managed/organizationUnit/ideal
	
返回结果

	{
	 "_rev":"0",
	 "_id":"ideal",
	 "dn":"ou=ideal,o=shanghai,dc=example,dc=com",
	 "description":"ideal company",
	 "name":"ideal"
	}	

## 2. 配置 OpenIDM 同步

在 sync.json 中增加

	{
		"name" : "managedOrganizationUnit_hrdb",
		"source" : "managed/organizationUnit",
		"target" : "system/hrdb/organization",
		"properties" : [
			{
				"source" : "description",
				"target" : "description"
			},
			{
				"source" : "name",
				"target" : "name"
			}
		],
		"policies" : [
			{
				"situation" : "CONFIRMED",
				"action" : "UPDATE"
			},
			{
				"situation" : "FOUND",
				"action" : "UPDATE"
			},
			{
				"situation" : "ABSENT",
				"action" : "CREATE"
			},
			{
				"situation" : "AMBIGUOUS",
				"action" : "EXCEPTION"
			},
			{
				"situation" : "MISSING",
				"action" : "UNLINK"
			},
			{
				"situation" : "SOURCE_MISSING",
				"action" : "IGNORE"
			},
			{
				"situation" : "UNQUALIFIED",
				"action" : "IGNORE"
			},
			{
				"situation" : "UNASSIGNED",
				"action" : "IGNORE"
			}
		]
	},
	{
		"name" : "managedOrganizationUnit_ldap",
		"source" : "managed/organizationUnit",
		"target" : "system/ldap/organizationalUnit",
		"properties" : [
			{
				"source" : "description",
				"target" : "description"
			},
			{
				"source" : "name",
				"target" : "ou"
			},
			{
				"source" : "dn",
				"target" : "dn"
			}
		],
		"policies" : [
			{
				"situation" : "CONFIRMED",
				"action" : "UPDATE"
			},
			{
				"situation" : "FOUND",
				"action" : "LINK"
			},
			{
				"situation" : "ABSENT",
				"action" : "CREATE"
			},
			{
				"situation" : "AMBIGUOUS",
				"action" : "IGNORE"
			},
			{
				"situation" : "MISSING",
				"action" : "IGNORE"
			},
			{
				"situation" : "SOURCE_MISSING",
				"action" : "IGNORE"
			},
			{
				"situation" : "UNQUALIFIED",
				"action" : "IGNORE"
			},
			{
				"situation" : "UNASSIGNED",
				"action" : "IGNORE"
			}
		]
	}

修改 provisioner.openicf-ldap.json，在 “objectTypes” 中增加

	"organizationalUnit" : {
		"$schema" : "http://json-schema.org/draft-03/schema",
		"id" : "organizationalUnit",
		"type" : "object",
		"nativeType" : "organizationalUnit",
		"properties" : {
			"preferredDeliveryMethod" : {
				"type" : "string",
				"nativeName" : "preferredDeliveryMethod",
				"nativeType" : "string"
			},
			"l" : {
				"type" : "array",
				"items" : {
					"type" : "string",
					"nativeType" : "string"
				},
				"nativeName" : "l",
				"nativeType" : "string"
			},
			"businessCategory" : {
				"type" : "array",
				"items" : {
					"type" : "string",
					"nativeType" : "string"
				},
				"nativeName" : "businessCategory",
				"nativeType" : "string"
			},
			"street" : {
				"type" : "array",
				"items" : {
					"type" : "string",
					"nativeType" : "string"
				},
				"nativeName" : "street",
				"nativeType" : "string"
			},
			"postOfficeBox" : {
				"type" : "array",
				"items" : {
					"type" : "string",
					"nativeType" : "string"
				},
				"nativeName" : "postOfficeBox",
				"nativeType" : "string"
			},
			"postalCode" : {
				"type" : "array",
				"items" : {
					"type" : "string",
					"nativeType" : "string"
				},
				"nativeName" : "postalCode",
				"nativeType" : "string"
			},
			"st" : {
				"type" : "array",
				"items" : {
					"type" : "string",
					"nativeType" : "string"
				},
				"nativeName" : "st",
				"nativeType" : "string"
			},
			"registeredAddress" : {
				"type" : "array",
				"items" : {
					"type" : "string",
					"nativeType" : "string"
				},
				"nativeName" : "registeredAddress",
				"nativeType" : "string"
			},
			"postalAddress" : {
				"type" : "array",
				"items" : {
					"type" : "string",
					"nativeType" : "string"
				},
				"nativeName" : "postalAddress",
				"nativeType" : "string"
			},
			"objectClass" : {
				"type" : "array",
				"items" : {
					"type" : "string",
					"nativeType" : "string"
				},
				"nativeName" : "objectClass",
				"nativeType" : "string",
				"flags" : [
					"NOT_CREATABLE",
					"NOT_UPDATEABLE"
				]
			},
			"description" : {
				"type" : "array",
				"items" : {
					"type" : "string",
					"nativeType" : "string"
				},
				"nativeName" : "description",
				"nativeType" : "string"
			},
			"ou" : {
				"type" : "array",
				"items" : {
					"type" : "string",
					"nativeType" : "string"
				},
				"required" : true,
				"nativeName" : "ou",
				"nativeType" : "string"
			},
			"physicalDeliveryOfficeName" : {
				"type" : "array",
				"items" : {
					"type" : "string",
					"nativeType" : "string"
				},
				"nativeName" : "physicalDeliveryOfficeName",
				"nativeType" : "string"
			},
			"telexNumber" : {
				"type" : "array",
				"items" : {
					"type" : "string",
					"nativeType" : "string"
				},
				"nativeName" : "telexNumber",
				"nativeType" : "string"
			},
			"teletexTerminalIdentifier" : {
				"type" : "array",
				"items" : {
					"type" : "string",
					"nativeType" : "string"
				},
				"nativeName" : "teletexTerminalIdentifier",
				"nativeType" : "string"
			},
			"userPassword" : {
				"type" : "array",
				"items" : {
					"type" : "string",
					"nativeType" : "JAVA_TYPE_BYTE_ARRAY"
				},
				"nativeName" : "userPassword",
				"nativeType" : "JAVA_TYPE_BYTE_ARRAY"
			},
			"dn" : {
				"type" : "string",
				"required" : true,
				"nativeName" : "__NAME__",
				"nativeType" : "string"
			},
			"telephoneNumber" : {
				"type" : "array",
				"items" : {
					"type" : "string",
					"nativeType" : "string"
				},
				"nativeName" : "telephoneNumber",
				"nativeType" : "string"
			}
		}
	}
	
重启 OpenIDM，执行	
	
	# curl \
	--header "X-OpenIDM-Username: openidm-admin" \
	--header "X-OpenIDM-Password: openidm-admin" \
	--request POST "http://openam.example.com:9090/openidm/sync?_action=recon&mapping=managedOrganizationUnit_hrdb"
	
	# curl \
	--header "X-OpenIDM-Username: openidm-admin" \
	--header "X-OpenIDM-Password: openidm-admin" \
	--request POST "http://openam.example.com:9090/openidm/sync?_action=recon&mapping=managedOrganizationUnit_ldap"
	
返回

	{"reconId":"acc2de0a-59ec-4537-9115-333be932aecd"}
	
这时查看 OpenDJ 和 MySQL，已经同步成功。

![](/images/2012-05-03-step-by-step-two-openidm.png)	