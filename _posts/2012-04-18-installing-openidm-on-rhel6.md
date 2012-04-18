---
layout: post
title: "在 RHEL6 上安装 OpenIDM"
category: linux
tags: 
- linux
- ldap
- sso
---
{% include JB/setup %}

确认 Java 环境，需要 update 24 以上版本。

	# java -version
	java version "1.6.0_31"
	Java(TM) SE Runtime Environment (build 1.6.0_31-b04)
	Java HotSpot(TM) 64-Bit Server VM (build 20.6-b01, mixed mode)

下载

	# wget http://download.forgerock.org/downloads/openidm/snapshot/openidm-2.1.0-SNAPSHOT.zip
	
安装

	# cp openidm-2.1.0-SNAPSHOT.zip /opt
	# cd /opt
	# unzip openidm-2.1.0-SNAPSHOT.zip
	
默认情况，OpenIDM 监听在 8080 和 8443 端口，这里因为和 OpenAM 用的同一台 Server，所以修改为 9090，9443。

	# cd openidm
	# vim conf/jetty.xml
	
如果在生产环境，需要替换默认的 OrientDB。这里替换为 MySQL。下载 [MySQL Connector/J](http://www.mysql.com/downloads/connector/j/)	 解压缩以后

	# cp mysql-connector-java-5.1.19-bin.jar /opt/openidm/bundle/
	# cd /opt/openidm/conf
	# mv repo.orientdb.json repo.orientdb.json.bak
	# cp ../samples/misc/repo.jdbc.json .
	# mysql -u root -p < /opt/openidm/db/scripts/mysql/openidm.sql
	# mysql -u root -p
	Enter password: 
	Welcome to the MySQL monitor.  Commands end with ; or \g.
	Your MySQL connection id is 4
	Server version: 5.1.52 Source distribution
	
	mysql> use openidm;
	Reading table information for completion of table and column names
	You can turn off this feature to get a quicker startup with -A
	
	Database changed
	mysql> show tables;
	+-------------------------+
	| Tables_in_openidm       |
	+-------------------------+
	| auditaccess             |
	| auditactivity           |
	| auditrecon              |
	| configobjectproperties  |
	| configobjects           |
	| genericobjectproperties |
	| genericobjects          |
	| internaluser            |
	| links                   |
	| managedobjectproperties |
	| managedobjects          |
	| objecttypes             |
	+-------------------------+
	12 rows in set (0.00 sec)
	
在启动 OpenIDM 之前，如果需要的话，修改 repo.jdbc.json

	# vim repo.jdbc.json
	"connection" : {
        "dbType" : "MYSQL",
        "jndiName" : "",
        "driverClass" : "com.mysql.jdbc.Driver",
        "jdbcUrl" : "jdbc:mysql://localhost:3306/openidm",
        "username" : "root",
        "password" : "",
        "defaultCatalog" : "openidm",
        "maxBatchSize" : 100,
        "maxTxRetry" : 5
    }
    
启动 OpenIDM

	# cd /opt/openidm
	# ./startup.sh
	Using OPENIDM_HOME:   /opt/openidm
	Using OPENIDM_OPTS:   -Xmx1024m
	Using LOGGING_CONFIG: -Djava.util.logging.config.file=/opt/openidm/conf/logging.properties
	Using boot properties at /opt/openidm/conf/boot/boot.properties
	OpenIDM version "2.1.0-SNAPSHOT" (revision: 1063)
	-> scr list
    Id   State          Name
	[  16] [active       ] org.forgerock.openidm.config.starter
	[   7] [active       ] org.forgerock.openidm.external.rest
	[  11] [active       ] org.forgerock.openidm.provisioner.openicf.connectorinfoprovider
	[   1] [active       ] org.forgerock.openidm.router
	[  18] [active       ] org.forgerock.openidm.scheduler
	[  13] [active       ] org.forgerock.openidm.restlet
	[   6] [unsatisfied  ] org.forgerock.openidm.external.email
	[  15] [unsatisfied  ] org.forgerock.openidm.repo.orientdb
	[   5] [active       ] org.forgerock.openidm.sync
	[   3] [active       ] org.forgerock.openidm.script
	[   2] [active       ] org.forgerock.openidm.scope
	[   9] [active       ] org.forgerock.openidm.http.contextregistrator
	[  17] [active       ] org.forgerock.openidm.config
	[   0] [active       ] org.forgerock.openidm.audit
	[  14] [active       ] org.forgerock.openidm.repo.jdbc
	[   4] [active       ] org.forgerock.openidm.managed
	[  12] [active       ] org.forgerock.openidm.provisioner.openicf
	[   8] [active       ] org.forgerock.openidm.authentication
	[  10] [active       ] org.forgerock.openidm.provisioner
	->
	
如果看到 	email 和 orientdb 是 unsatisfied，repo.jdbc 是 active 就成功了。如果有其它的 unsatisfied
检查 openidm/logs 或者查看 [Troubleshooting](http://openidm.forgerock.org/doc/integrators-guide/index.html#chap-troubleshooting)。

现在访问：http://openam.example.com:9090/system/console, 使用 admin/admin 登录控制台。

