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

	cp -r samples/sample3/conf samples/sample3/tools .
	
在我使用的这个版本中，CreateScript.groovy 脚本中 `__ACCOUNT__` 这段内容的语法有一个错误，参数属性的最后一段应该是 `,` 这里写成了 `;` 。

用以下内容替换 sync.json 文件

	{
    "mappings" : [
        {
            "name" : "systemManagedOrganization_hrdb",
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
            "name" : "systemManagedUser_hrdb",
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
	--request POST "http://openam.example.com:9090/openidm/sync?_action=recon&mapping=systemManagedOrganization_hrdb"
	
	# curl \
	--header "X-OpenIDM-Username: openidm-admin" \
	--header "X-OpenIDM-Password: openidm-admin" \
	--request POST "http://openam.example.com:9090/openidm/sync?_action=recon&mapping=systemManagedUser_hrdb"
	
返回 JSON 结果

	{"reconId":"6073945c-60d8-4db3-97e5-e0ac0e5753d6"}
	
然后查看 MySQL 数据库 Users 和 Organizations ，数据已经成功同步。		

## 2. From OpenIDM To OpenDJ

以 samples/sample2 为模板（不要覆盖之前的 sync.json 文件），把新增加的 use,organization 同步到 OpenDJ 。

	# cp samples/sample2b/conf/provisioner.openicf-ldap.json conf
	# cp -r samples/sample2b/script .

在 sync.json 中增加
	
	{
		"name" : "managedUser_systemLdapAccounts",
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
			"source" : "target.dn = 'uid=' + source.userName + ',ou=People,dc=example,dc=com';"
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
	},
	{
		"name" : "managedOrganization_ldap",
		"source" : "managed/organization",
		"target" : "system/ldap/organizationalUnit",
		"properties" : [
			{
				"source" : "description",
				"target" : "description"
			},
			{
				"source" : "name",
				"target" : "ou"
			}
		],
		"onCreate" : {
			"type" : "text/javascript",
			"source" : "target.dn = 'ou=' + source.name + ',ou=People,dc=example,dc=com';"
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
	--request POST "http://openam.example.com:9090/openidm/sync?_action=recon&mapping=managedUser_systemLdapAccounts"
	
	# curl \
	--header "X-OpenIDM-Username: openidm-admin" \
	--header "X-OpenIDM-Password: openidm-admin" \
	--request POST "http://openam.example.com:9090/openidm/sync?_action=recon&mapping=managedOrganization_ldap"
	
返回

	{"reconId":"60a9fa62-00ca-4dff-a538-94f09c161fde"}
	
这时查看 OpenDJ，已经同步成功。	

## 3. 定时同步

除了可以调用 REST API 同步之外，OpenIDM 也提供了 Cron 的方式进行调度。可以分别为上边的 4 个接口建立 scheduler-recon 文件。
以 managedOrganization_ldap 为例

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