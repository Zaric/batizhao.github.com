---
layout: post
title: "Maven 自动化构建（dev, test, prod）"
category: maven
tags: 
- maven
---
{% include JB/setup %}


## 1. 需求描述

* 在项目构建时，需要根据环境的不同生成不同的安装包。不希望每次通过人工修改配置。
* 有非常多的不同的 prod 环境配置，100+。
* 在 Maven 的多模块项目中，需要有一个完整的 properties 来定义各个不同的环境，而不是分散在各个 Module 中。
* 不希望这些 properties 定义在 pom 中，而是需要独立出来，通过动态参数加载。
* 最终可以通过 mvn package -Pdev, mvn package -Ptest, mvn package -Pprod 完成各种环境的构建。

项目模块假设是以下的一个结构：

	<modules>
        <module>cas</module>
        <module>core</module>
        <module>web</module>
        <module>client</module>
    </modules>

其中，cas，web，client 是三个 war，core 是个 jar。要完成上述目标，还需要用到 properties-maven-plugin 这个插件，可以把需要定制的 properties 放到一个外部的文件，剥离 pom，并且做到动态加载配置。


## 2. maven-assembly-plugin

一开始想用 maven-assembly-plugin，assembly 功能很强大，可定制的程度很高。试了一下，也可以解决上边的需求。

pom

	<plugin>
	    <artifactId>maven-assembly-plugin</artifactId>
	    <configuration>
	        <descriptor>src/main/assemble/package.xml</descriptor>
	    </configuration>
	    <executions>
	        <execution>
	            <id>make-assembly</id>
	            <phase>package</phase>
	            <goals>
	                <goal>single</goal>
	            </goals>
	        </execution>
	    </executions>
	</plugin>

package.xml

	<assembly xmlns="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.2"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.2 http://maven.apache.org/xsd/assembly-1.1.2.xsd">

	    <id>distribution</id>
	    <formats>
	        <format>war</format>
	    </formats>
	    <includeBaseDirectory>false</includeBaseDirectory>
	    <fileSets>
	        <fileSet>
	            <directory>${project.build.directory}/${project.build.finalName}</directory>
	            <includes>
	                <include>**</include>
	            </includes>
	            <excludes>
	                <exclude>**/deploy*.xml</exclude>
	            </excludes>
	            <outputDirectory>/</outputDirectory>
	        </fileSet>
	        <fileSet>
	            <directory>${project.build.directory}/${project.build.finalName}</directory>
	            <includes>
	                <include>**/deploy*.xml</include>
	            </includes>
	            <filtered>true</filtered>
	            <outputDirectory>/</outputDirectory>
	        </fileSet>
	    </fileSets>
	</assembly>

但存在的问题如下：

* 如果使用了 assembly，那么如果在配置文件中有变量，你还是需要对 maven-war-plugin 的打包过程进行定制，否则会造成 assembly filter 起作用，war 打出的包中变量没有替换的情况。也是就是如果需要保证 mvn package 构建出的 war 是有效的，既需要定制 assembly，也需要定制 war。
* assembly 生成的文件名默认会附加 @id@ 在文件名上，好像没有配置的方法。

综上，在这个项目中，因为没有生成不同格式发布包的要求，只需要生成 war，所以，不考虑引入 assembly ，只对 maven-war-plugin 的构建过程进行定制。 


## 3. maven-war-plugin

在 root pom 中：

	<pluginManagement>
        <plugins>
            <plugin>
                <artifactId>maven-war-plugin</artifactId>
                <version>2.1.1</version>
            </plugin>
        </plugins>
    </pluginManagement>
    
    <profiles>
        <profile>
            <id>test</id>
            <properties>
                <profile>test</profile>
            </properties>
        </profile>
    </profiles>
    
    <properties>
    	<profile>dev</profile>
    </properties>
    
在 root src 下边增加 config 目录，把相关的配置文件放到下边：

* dev.properties
* test.properties
* prod1.properties
* prod2.properties
* …  

dev 内容：

	ldap.port=389
	ldap.host=ldap.dev.org
	ldap.searchbase=dc\=dev,dc\=org
	ldap.authdn=cn\=Directory Manager
	ldap.passwd=password
	
test 内容：

	ldap.port=389
	ldap.host=ldap.dev.org
	ldap.searchbase=dc\=test,dc\=org
	ldap.authdn=cn\=Directory Manager
	ldap.passwd=password	
	
在 cas module pom 中：

	<plugin>
        <artifactId>maven-war-plugin</artifactId>
        <configuration>
            <webResources>
            	<!-- 对需要修改参数的配置文件进行配置 filtering -->
                <resource>
                    <directory>src/main/webapp/WEB-INF</directory>
                    <includes>
                        <include>**/deploy*.xml</include>
                    </includes>
                    <targetPath>WEB-INF</targetPath>
                    <filtering>true</filtering>
                </resource>
            </webResources>
        </configuration>
    </plugin>
    <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>properties-maven-plugin</artifactId>
        <version>1.0-alpha-2</version>
        <executions>
            <execution>
                <phase>initialize</phase>
                <goals>
                    <goal>read-project-properties</goal>
                </goals>
                <configuration>
                    <files>
                    	<!-- 指向 root 目录中的配置文件，并且根据 profile 动态加载文件名 -->                    
                        <file>${basedir}/../src/config/${profile}.properties</file>
                    </files>
                </configuration>
            </execution>
        </executions>
    </plugin>
    
现在在 cas module 的根目录运行 mvn package 或者 mvn package -Ptest，可以看到需要修改的 ${ldap.host} 等已经被替换。之后只需要在每个 module 下边对相关的插件进行定制，就可以完成之前的目标。    

	


