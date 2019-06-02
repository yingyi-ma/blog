---
title: SrpingBoot+Jooq整合+反向工程(自动生产代码)
date: 2019-04-25 16:54:18
tags: 
    - Java 
    - 框架
keywords:
description:
---

#### 前言
>SpringBoot版本：2.1.4  [demo](https://start.spring.io/starter.zip?fakeusernameremembered=&fakepasswordremembered=&type=maven-project&language=java&bootVersion=2.1.4.RELEASE&baseDir=demo&groupId=com.example&artifactId=demo&name=demo&description=Demo+project+for+Spring+Boot&packageName=com.example.demo&packaging=jar&javaVersion=1.8&inputSearch=)
对应的Jooq版本：3.11.10（不用设置，Springboot默认获取对应的jooq版本${jooq.version}）


#### 步骤
>1. 已安装mysql，并创建数据库、数据表
>
>2. idea 导入Springboot demo项目
>3. pom.xml 配置
>4. JooqConfig.xml 配置
>5. mvn compile 执行，生成代码



创建数据库
```
create database testjooq;
use testjooq;
```


数据库表
```

-- ----------------------------
-- Table structure for role
-- ----------------------------
DROP TABLE IF EXISTS `role`;
CREATE TABLE `role` (
  `id` bigint(20) NOT NULL,
  `role_name` varchar(32) DEFAULT NULL,
  `remark` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- ----------------------------
-- Table structure for user
-- ----------------------------
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `email` varchar(255) DEFAULT NULL COMMENT '邮箱',
  `mobile` varchar(255) DEFAULT NULL COMMENT '手机号码',
  `username` varchar(255) DEFAULT NULL COMMENT '用户名',
  `password` varchar(255) DEFAULT NULL COMMENT '密码',
  `company` varchar(255) DEFAULT NULL COMMENT '公司单位',
  `address` varchar(255) DEFAULT NULL COMMENT '地址',
  `role_id` int(1) DEFAULT NULL COMMENT '角色',
  `del_flag` varchar(1) DEFAULT NULL COMMENT '删除标识：0-未删除,1-删除',
  `edit_time` datetime DEFAULT NULL COMMENT '创建时间',
  `del_time` datetime DEFAULT NULL COMMENT '删除时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=13 DEFAULT CHARSET=utf8;
SET FOREIGN_KEY_CHECKS=1;

```



pom.xml 配置
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>springboot-jooq</groupId>
	<artifactId>springboot-jooq</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>springboot-jooq</name>
	<description>Demo project for Spring Boot</description>


	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.4.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
	</properties>


	<dependencies>
    	<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-jooq</artifactId>
		</dependency>

		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<version>1.16.18</version>
		</dependency>

		<!-- 阿里巴巴fastjson，解析json视图 -->
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>fastjson</artifactId>
			<version>1.2.15</version>
		</dependency>
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>druid</artifactId>
			<version>1.1.3</version>
		</dependency>
		<dependency>
			<groupId>org.jooq</groupId>
			<artifactId>jooq-meta</artifactId>
		</dependency>
		<!--<dependency>-->
			<!--<groupId>org.jooq</groupId>-->
			<!--<artifactId>jooq-meta-extensions</artifactId>-->
			<!--<version>3.8.9</version>-->
		<!--</dependency>-->
		<dependency>
			<groupId>org.jooq</groupId>
			<artifactId>jooq-codegen</artifactId>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
			</plugin>

			<plugin>
				<groupId>org.jooq</groupId>
				<artifactId>jooq-codegen-maven</artifactId>
				<version>${jooq.version}</version>
				<executions>
					<execution>
						<goals>
							<goal>generate</goal>
						</goals>
					</execution>
				</executions>
				<dependencies>
					<dependency>
						<groupId>mysql</groupId>
						<artifactId>mysql-connector-java</artifactId>
						<version>${mysql.version}</version>
					</dependency>
				</dependencies>
				<configuration>
					<configurationFile>src/main/resources/JooqConfig.xml</configurationFile>
				</configuration>
			</plugin>
		</plugins>
	</build>


</project>

```


JooqConfig.xml 配置
```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<configuration>
    <jdbc>
        <driver>com.mysql.cj.jdbc.Driver</driver>
        <url>jdbc:mysql://localhost:3306/testjooq?serverTimezone=GMT</url>
        <user>root</user>
        <password>root</password>
    </jdbc>
    <generator>
        <!-- 代码生成器 -->
        <name>org.jooq.codegen.JavaGenerator</name>
        <database>
            <!--下面这两行是为view也生成代码的关键-->
            <!--force generating id'sfor everything in public schema, that has an 'id' field-->
            <syntheticPrimaryKeys>public\..*\.id</syntheticPrimaryKeys>
            <!--name for fake primary key-->
            <overridePrimaryKeys>override_primmary_key</overridePrimaryKeys>
            <name>org.jooq.meta.mysql.MySQLDatabase</name>
            <!--include和exclude用于控制为数据库中哪些表生成代码-->
            <includes>.*</includes>
            <!--<excludes></excludes>-->
            <!--数据库名称-->
            <inputSchema>testjooq</inputSchema>
        </database>

        <generate>
            <!--生成dao和pojo-->
            <daos>true</daos>
            <pojos>true</pojos>
            <!--把数据库时间类型映射到java 8时间类型-->
            <javaTimeTypes>true</javaTimeTypes>
            <!--<interfaces>true</interfaces>-->
            <!--不在生成的代码中添加spring注释，比如@Repository-->
            <springAnnotations>false</springAnnotations>
        </generate>

        <target>
            <!--生成代码文件的包名及放置目录-->
            <packageName>com.generator</packageName>
            <directory>src/main/java</directory>
        </target>
    </generator>
</configuration>
```


执行 mvn compile 自动生成代码