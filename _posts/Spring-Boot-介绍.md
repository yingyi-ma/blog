---
title: Spring Boot 介绍
date: 2019-04-26 17:42:56
tags: 
     - Java
     - 框架
keywords: springboot
description:
---


#### 前言
>&emsp;&emsp;Spring Boot是由Pivotal团队提供的全新框架，其设计目的是用来简化新Spring应用的初始搭建以及开发过程。该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置。通过这种方式，Spring Boot致力于在蓬勃发展的快速应用开发领域(rapid application development)成为领导者。


#### 核心功能

**1. 独立运行**　

Spring Boot可以以jar包形式独立运行，运行一个Spring Boot项目只需要通过java -jar xx.jar来运行就可以；

**2. 内嵌servlet容器**

spring boot自带了tomcat，jetty跟undertow，这样我们就无需以war包形式部署项目；

**3. 提供starter简化maven配置**

提供了一系列的starter pom来简化maven配置，看起来pom文件内容少了很多；

**4. 自动配置spring**

Spring Boot会根据在类路径中的jar包、类，为jar包里的类自动配置bean，这会极大地减少我们要使用的配置；当然，spring boot只是考虑了大部分场景，实际开发中仍会有需要我们自己配置的bean；

**5. 准生产的应用监控**

Spring Boot提供基于http、ssh跟telnet对运行时的项目进行监控；

**6. 无xml配置**

Spring 4.x提供了条件注解，在Spring Boot中可以不用任何xml即可实现spring的所有配置；原理参见@enable注解跟@import注解；


#### Demo
在线生成 [https://start.spring.io/](https://start.spring.io/)

