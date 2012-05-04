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

## 1. 在 managed.json 中增加新类型 organization

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
	--data '{ "name":"dev", "dn":"o=dev,ou=ideal,ou=people,dc=example,dc=com", "description":"ideal dev team" }' \
	http://openam.example.com:9090/openidm/managed/organization/dev
	
返回结果

	{"_id":"ideal","_rev":"0"}
	
查询新增加的 organization

	# curl \
	--header "X-OpenIDM-Username: openidm-admin" \
	--header "X-OpenIDM-Password: openidm-admin" \
	http://openam.example.com:9090/openidm/managed/organization/dev
	
返回结果

	{
	 "_rev":"0",
	 "_id":"dev",
	 "dn":"o=dev,ou=ideal,ou=people,dc=example,dc=com",
	 "description":"ideal dev team",
	 "name":"dev"
	}	

## 2. 配置 OpenIDM 同步

在 sync.json 中增加

	{
		"name" : "managedOrganization_hrdb",
		"source" : "managed/organization",
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
		"name" : "managedOrganization_ldap",
		"source" : "managed/organization",
		"target" : "system/ldap/organization",
		"properties" : [
			{
				"source" : "description",
				"target" : "description"
			},
			{
				"source" : "name",
				"target" : "o"
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

	"organization" :
	{
	   "$schema" : "http://json-schema.org/draft-03/schema",
	   "id" : "organization",
	   "type" : "object",
	   "nativeType" : "organization",
	   "properties" :
		  {
			 "preferredDeliveryMethod" :
				{
				   "type" : "string",
				   "nativeName" : "preferredDeliveryMethod",
				   "nativeType" : "string"
				},
			 "seeAlso" :
				{
				   "type" : "array",
				   "items" :
					  {
						 "type" : "string",
						 "nativeType" : "string"
					  },
				   "nativeName" : "seeAlso",
				   "nativeType" : "string"
				},
			 "x121Address" :
				{
				   "type" : "array",
				   "items" :
					  {
						 "type" : "string",
						 "nativeType" : "string"
					  },
				   "nativeName" : "x121Address",
				   "nativeType" : "string"
				},
			 "l" :
				{
				   "type" : "array",
				   "items" :
					  {
						 "type" : "string",
						 "nativeType" : "string"
					  },
				   "nativeName" : "l",
				   "nativeType" : "string"
				},
			 "o" :
				{
				   "type" : "array",
				   "items" :
					  {
						 "type" : "string",
						 "nativeType" : "string"
					  },
				   "required" : true,
				   "nativeName" : "o",
				   "nativeType" : "string"
				},
			 "businessCategory" :
				{
				   "type" : "array",
				   "items" :
					  {
						 "type" : "string",
						 "nativeType" : "string"
					  },
				   "nativeName" : "businessCategory",
				   "nativeType" : "string"
				},
			 "street" :
				{
				   "type" : "array",
				   "items" :
					  {
						 "type" : "string",
						 "nativeType" : "string"
					  },
				   "nativeName" : "street",
				   "nativeType" : "string"
				},
			 "postOfficeBox" :
				{
				   "type" : "array",
				   "items" :
					  {
						 "type" : "string",
						 "nativeType" : "string"
					  },
				   "nativeName" : "postOfficeBox",
				   "nativeType" : "string"
				},
			 "postalCode" :
				{
				   "type" : "array",
				   "items" :
					  {
						 "type" : "string",
						 "nativeType" : "string"
					  },
				   "nativeName" : "postalCode",
				   "nativeType" : "string"
				},
			 "st" :
				{
				   "type" : "array",
				   "items" :
					  {
						 "type" : "string",
						 "nativeType" : "string"
					  },
				   "nativeName" : "st",
				   "nativeType" : "string"
				},
			 "registeredAddress" :
				{
				   "type" : "array",
				   "items" :
					  {
						 "type" : "string",
						 "nativeType" : "string"
					  },
				   "nativeName" : "registeredAddress",
				   "nativeType" : "string"
				},
			 "postalAddress" :
				{
				   "type" : "array",
				   "items" :
					  {
						 "type" : "string",
						 "nativeType" : "string"
					  },
				   "nativeName" : "postalAddress",
				   "nativeType" : "string"
				},
			 "objectClass" :
				{
				   "type" : "array",
				   "items" :
					  {
						 "type" : "string",
						 "nativeType" : "string"
					  },
				   "nativeName" : "objectClass",
				   "nativeType" : "string",
				   "flags" :
					  [
						 "NOT_CREATABLE",
						 "NOT_UPDATEABLE"
					  ]
				},
			 "description" :
				{
				   "type" : "array",
				   "items" :
					  {
						 "type" : "string",
						 "nativeType" : "string"
					  },
				   "nativeName" : "description",
				   "nativeType" : "string"
				},
			 "internationaliSDNNumber" :
				{
				   "type" : "array",
				   "items" :
					  {
						 "type" : "string",
						 "nativeType" : "string"
					  },
				   "nativeName" : "internationaliSDNNumber",
				   "nativeType" : "string"
				},
			 "searchGuide" :
				{
				   "type" : "array",
				   "items" :
					  {
						 "type" : "string",
						 "nativeType" : "string"
					  },
				   "nativeName" : "searchGuide",
				   "nativeType" : "string"
				},
			 "physicalDeliveryOfficeName" :
				{
				   "type" : "array",
				   "items" :
					  {
						 "type" : "string",
						 "nativeType" : "string"
					  },
				   "nativeName" : "physicalDeliveryOfficeName",
				   "nativeType" : "string"
				},
			 "telexNumber" :
				{
				   "type" : "array",
				   "items" :
					  {
						 "type" : "string",
						 "nativeType" : "string"
					  },
				   "nativeName" : "telexNumber",
				   "nativeType" : "string"
				},
			 "teletexTerminalIdentifier" :
				{
				   "type" : "array",
				   "items" :
					  {
						 "type" : "string",
						 "nativeType" : "string"
					  },
				   "nativeName" : "teletexTerminalIdentifier",
				   "nativeType" : "string"
				},
			 "userPassword" :
				{
				   "type" : "array",
				   "items" :
					  {
						 "type" : "string",
						 "nativeType" : "JAVA_TYPE_BYTE_ARRAY"
					  },
				   "nativeName" : "userPassword",
				   "nativeType" : "JAVA_TYPE_BYTE_ARRAY"
				},
			 "dn" :
				{
				   "type" : "string",
				   "required" : true,
				   "nativeName" : "__NAME__",
				   "nativeType" : "string"
				},
			 "telephoneNumber" :
				{
				   "type" : "array",
				   "items" :
					  {
						 "type" : "string",
						 "nativeType" : "string"
					  },
				   "nativeName" : "telephoneNumber",
				   "nativeType" : "string"
				},
			 "destinationIndicator" :
				{
				   "type" : "array",
				   "items" :
					  {
						 "type" : "string",
						 "nativeType" : "string"
					  },
				   "nativeName" : "destinationIndicator",
				   "nativeType" : "string"
				},
			 "facsimileTelephoneNumber" :
				{
				   "type" : "array",
				   "items" :
					  {
						 "type" : "string",
						 "nativeType" : "string"
					  },
				   "nativeName" : "facsimileTelephoneNumber",
				   "nativeType" : "string"
				}
		  }
	}
	
重启 OpenIDM，执行	
	
	# curl \
	--header "X-OpenIDM-Username: openidm-admin" \
	--header "X-OpenIDM-Password: openidm-admin" \
	--request POST "http://openam.example.com:9090/openidm/sync?_action=recon&mapping=managedOrganization_hrdb"
	
	# curl \
	--header "X-OpenIDM-Username: openidm-admin" \
	--header "X-OpenIDM-Password: openidm-admin" \
	--request POST "http://openam.example.com:9090/openidm/sync?_action=recon&mapping=managedOrganization_ldap"
	
返回

	{"reconId":"acc2de0a-59ec-4537-9115-333be932aecd"}
	
这时查看 OpenDJ 和 MySQL，已经同步成功。

![](/images/2012-05-03-step-by-step-two-openidm.png)	