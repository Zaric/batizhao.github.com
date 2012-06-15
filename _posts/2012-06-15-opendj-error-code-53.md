---
layout: post
title: "OpenDJ error code 53"
category: 
tags: 
- ldap
---
{% include JB/setup %}

By default, the OpenDJ LDAP directory server password policy is set to reject encrypted passwords.
So when adding or importing data with encrypted passwords, the server returns some error like this:

	LDAP: error code 53 – Pre-encoded passwords are not allowed for the password attribute userPassword

To allow pre-encoded passwords, the default password policy settings must be changed. This can be done using the dsconfig command line tool in advanced mode:

	$ dsconfig --advanced -p 4444 -h localhost -D "cn=directory manager" -X
	>>>> Specify OpenDS LDAP connection parameters
	Password for user 'cn=directory manager':
	>>>> OpenDS configuration console main menu
	What do you want to configure?
	1)   Access Control Handler          24)  Monitor Provider
	2)   Account Status Notification     25)  Network Group
	Handler
	3)   Administration Connector        26)  Network Group Criteria
	4)   Alert Handler                   27)  Network Group Request Filtering
	Policy
	5)   Attribute Syntax                28)  Network Group Resource Limits
	6)   Backend                         29)  Password Generator
	7)   Certificate Mapper              30)  Password Policy
	8)   Connection Handler              31)  Password Storage Scheme
	9)   Crypto Manager                  32)  Password Validator
	10)  Debug Target                    33)  Plugin
	11)  Entry Cache                     34)  Plugin Root
	12)  Extended Operation Handler      35)  Replication Domain
	13)  Extension                       36)  Replication Server
	14)  Global Configuration            37)  Root DN
	15)  Group Implementation            38)  Root DSE Backend
	16)  Identity Mapper                 39)  SASL Mechanism Handler
	17)  Key Manager Provider            40)  Synchronization Provider
	18)  Local DB Index                  41)  Trust Manager Provider
	19)  Local DB VLV Index              42)  Virtual Attribute
	20)  Log Publisher                   43)  Work Queue
	21)  Log Retention Policy            44)  Workflow
	22)  Log Rotation Policy             45)  Workflow Element
	23)  Matching Rule
	q)   quit
	Enter choice: 30
	>>>> Password Policy management menu
	What would you like to do?
	1)  List existing Password Policies
	2)  Create a new Password Policy
	3)  View and edit an existing Password Policy
	4)  Delete an existing Password Policy
	b)  back
	q)  quit
	Enter choice [b]: 3
	>>>> Select the Password Policy from the following list:
	1)  Default Password Policy
	2)  Root Password Policy
	c)  cancel
	q)  quit
	Enter choice [c]: 1
	>>>> Configure the properties of the Password Policy
	Property                                   Value(s)
	--------------------------------------------------------------------
	1)   account-status-notification-handler        -
	2)   allow-expired-password-changes             false
	3)   allow-multiple-password-values             false
	4)   allow-pre-encoded-passwords                false
	5)   allow-user-password-changes                true
	6)   default-password-storage-scheme            Salted SHA-1
	7)   deprecated-password-storage-scheme         -
	8)   expire-passwords-without-warning           false
	9)   force-change-on-add                        false
	10)  force-change-on-reset                      false
	11)  grace-login-count                          0
	12)  idle-lockout-interval                      0 s
	13)  last-login-time-attribute                  -
	14)  last-login-time-format                     -
	15)  lockout-duration                           0 s
	16)  lockout-failure-count                      0
	17)  lockout-failure-expiration-interval        0 s
	18)  max-password-age                           0 s
	19)  max-password-reset-age                     0 s
	20)  min-password-age                           0 s
	21)  password-attribute                         userpassword
	22)  password-change-requires-current-password  false
	23)  password-expiration-warning-interval       5 d
	24)  password-generator                         Random Password Generator
	25)  password-history-count                     0
	26)  password-history-duration                  0 s
	27)  password-validator                         -
	28)  previous-last-login-time-format            -
	29)  require-change-by-time                     -
	30)  require-secure-authentication              false
	31)  require-secure-password-changes            false
	32)  skip-validation-for-administrators         false
	33)  state-update-failure-policy                reactive
	?)   help
	f)   finish - apply any changes to the Password Policy
	c)   cancel
	q)   quit
	Enter choice [f]: 4
	>>>> Configuring the "allow-pre-encoded-passwords" property
	Indicates whether users can change their passwords by providing a
	pre-encoded value.
	This can cause a security risk because the clear-text version of the
	password is not known and therefore validation checks cannot be applied to
	it.
	Do you want to modify the "allow-pre-encoded-passwords" property?
	1)  Keep the default value: false
	2)  Change it to the value: true
	?)  help
	q)  quit
	Enter choice [1]: 2
	Press RETURN to continue
	>>>> Configure the properties of the Password Policy
	Property                                   Value(s)
	--------------------------------------------------------------------
	1)   account-status-notification-handler        -
	2)   allow-expired-password-changes             false
	3)   allow-multiple-password-values             false
	4)   allow-pre-encoded-passwords                true
	5)   allow-user-password-changes                true
	6)   default-password-storage-scheme            Salted SHA-1
	7)   deprecated-password-storage-scheme         -
	8)   expire-passwords-without-warning           false
	9)   force-change-on-add                        false
	10)  force-change-on-reset                      false
	11)  grace-login-count                          0
	12)  idle-lockout-interval                      0 s
	13)  last-login-time-attribute                  -
	14)  last-login-time-format                     -
	15)  lockout-duration                           0 s
	16)  lockout-failure-count                      0
	17)  lockout-failure-expiration-interval        0 s
	18)  max-password-age                           0 s
	19)  max-password-reset-age                     0 s
	20)  min-password-age                           0 s
	21)  password-attribute                         userpassword
	22)  password-change-requires-current-password  false
	23)  password-expiration-warning-interval       5 d
	24)  password-generator                         Random Password Generator
	25)  password-history-count                     0
	26)  password-history-duration                  0 s
	27)  password-validator                         -
	28)  previous-last-login-time-format            -
	29)  require-change-by-time                     -
	30)  require-secure-authentication              false
	31)  require-secure-password-changes            false
	32)  skip-validation-for-administrators         false
	33)  state-update-failure-policy                reactive
	?)   help
	f)   finish - apply any changes to the Password Policy
	c)   cancel
	q)   quit
	Enter choice [f]:
	The Password Policy was modified successfully
	Press RETURN to continue

The equivalent non interactive command is:

	$ dsconfig set-password-policy-prop \
	--policy-name "Default Password Policy" \
	--set allow-pre-encoded-passwords:true \
	--hostname localhost \
	--trustAll \
	--port 4444 \
	--bindDN "cn=directory manager" \
	--bindPassword ****** \
	--no-prompt