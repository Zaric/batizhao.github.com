---
layout: post
title: "进阶使用 OpenIDM 【一】"
category: java
tags: 
- sso
- ldap
---
{% include JB/setup %}

## 1. From OpenIDM To MySQL

以 samples/sample3 为模板，把新增加的 use,organization 分别同步到 MySQL 的 Users,Organizations 表。
如果你看过 [使用 OpenIDM RESTful API](http://localhost:4000/java/2012/04/20/using-openidm-with-rest-api/) ,
这时在 managedobjects 表中应该有两条数据。

managed/user

	{
	 "_rev":"0",
	 "_id":"joe",
	 "email":["joe@example.com"],
	 "description":"My first user",
	 "familyName":"smith",
	 "userName":"joe",
	 "givenName":"joe",
	 "displayName":"Felicitas Doe"
	}

managed/organization
	
	{
	 "_rev":"0",
	 "_id":"shanghai",
	 "dn":"o=shanghai,dc=example,dc=com",
	 "description":"shanghai",
	 "name":"shanghai"
	}

执行

	# cp -r samples/sample3/conf samples/sample3/tools .
	
在我使用的这个版本中，CreateScript.groovy 脚本中 `__ACCOUNT__` 这段内容的语法有一个错误，参数属性的最后一段应该是 `,` 这里写成了 `;` 。

用以下内容替换 sync.json 文件

	{
    "mappings" : [
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
            "name" : "managedUser_hrdb",
            "source" : "managed/user",
            "target" : "system/hrdb/account",
            "properties" : [
                {
                    "source" : "email",
                    "target" : "email"
                },
                {
                    "source" : "userName",
                    "target" : "uid"
                },
                {
                    "source" : "familyName",
                    "target" : "lastName"
                },
                {
                    "source" : "givenName",
                    "target" : "firstName"
                },
                {
                    "source" : "displayName",
                    "target" : "fullName"
                },
                {
                    "source" : "description",
                    "target" : "organization"
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
        }
    ]}
	
provisioner.openicf-scriptedsql.json 文件中只需要修改 configurationProperties 这段内容

	"configurationProperties" : {
        ...
        "password" : "Your MySQL password",
        ...
        "createScriptFileName" : "/opt/openidm/tools/CreateScript.groovy",
        "testScriptFileName" : "/opt/openidm/tools/TestScript.groovy",
        "searchScriptFileName" : "/opt/openidm/tools/SearchScript.groovy",
        "deleteScriptFileName" : "/opt/openidm/tools/DeleteScript.groovy",
        "updateScriptFileName" : "/opt/openidm/tools/UpdateScript.groovy",
        "syncScriptFileName" : "/opt/openidm/tools/SyncScript.groovy"
    }
    
重启 OpenIDM 后，执行

	# curl \
	--header "X-OpenIDM-Username: openidm-admin" \
	--header "X-OpenIDM-Password: openidm-admin" \
	--request POST "http://openam.example.com:9090/openidm/sync?_action=recon&mapping=managedOrganization_hrdb"
	
	# curl \
	--header "X-OpenIDM-Username: openidm-admin" \
	--header "X-OpenIDM-Password: openidm-admin" \
	--request POST "http://openam.example.com:9090/openidm/sync?_action=recon&mapping=managedUser_hrdb"
	
返回 JSON 结果

	{"reconId":"6073945c-60d8-4db3-97e5-e0ac0e5753d6"}
	
然后查看 MySQL 数据库 Users 和 Organizations ，数据已经成功同步。		

## 2. From OpenIDM To OpenDJ

以 samples/sample2 为模板（不要覆盖之前的 sync.json 文件），把新增加的 use,organization 同步到 OpenDJ 。

	# cp samples/sample2b/conf/provisioner.openicf-ldap.json conf
	# cp -r samples/sample2b/script .

在 sync.json 中增加
	
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
	},
	{
		"name" : "managedUser_ldap",
		"source" : "managed/user",
		"target" : "system/ldap/account",
		"correlationQuery" : {
			"type" : "text/javascript",
			"file" : "script/ldapBackCorrelationQuery.js"
		},
		"properties" : [
			{
				"source" : "givenName",
				"target" : "givenName"
			},
			{
				"source" : "familyName",
				"target" : "sn"
			},
			{
				"source" : "displayName",
				"target" : "cn"
			},
			{
				"source" : "userName",
				"target" : "uid"
			},
			{
				"source" : "description",
				"target" : "description"
			},
			{
				"source" : "email",
				"target" : "mail"
			}
		],
		"onCreate" : {
			"type" : "text/javascript",
			"source" : "target.dn = 'uid=' + source.userName + ',o=shanghai,dc=example,dc=com';"
		},
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
	--request POST "http://openam.example.com:9090/openidm/sync?_action=recon&mapping=managedOrganization_ldap"
	
	# curl \
	--header "X-OpenIDM-Username: openidm-admin" \
	--header "X-OpenIDM-Password: openidm-admin" \
	--request POST "http://openam.example.com:9090/openidm/sync?_action=recon&mapping=managedUser_ldap"		
	
返回

	{"reconId":"60a9fa62-00ca-4dff-a538-94f09c161fde"}
	
这时查看 OpenDJ，已经同步成功。	

## 3. 定时同步

除了可以调用 REST API 同步之外，OpenIDM 也提供了 Cron 的方式进行调度。可以分别为上边的 4 个接口建立 scheduler-recon 文件。
以 managedOrganizationUnit_ldap 为例

	# cp samples/sample2/conf/scheduler-recon.json conf
	# mv conf/scheduler-recon.json conf/scheduler-recon_managedOrganization_ldap.json 

打开文件，enabled 修改为 true，schedule 和 mapping 修改为相应的配置
	
	{
		"enabled" : true,
		"type": "cron",
		"schedule": "1 * * * * ?",
		"invokeService": "sync",
		"invokeContext": {
			"action": "reconcile",
			"mapping": "managedOrganization_ldap"
		}
	}
	
重启 OpenIDM 后，	Organization 和 User 会分别同步到 OpenDJ 和 MySQL。

如果需要停止调度任务，除了修改 scheduler-recon.json 文件，可能还需要删除 configobjects 表中的 org.forgerock.openidm.scheduler 相关纪录。