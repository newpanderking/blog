---
layout: post
title:  "SpringBoot启动报错 Failed to determine a suitable driver class"
date: 2019-07-22
categories: 技术
tags: spring-boot 
description: SpringBoot启动报错 "Failed to determine a suitable driver class"
---

####  spring boot 启动报错 "Failed to determine a suitable driver class"

- SpringBoot启动报错如下

```
Error starting ApplicationContext. To display the conditions report re-run your application with 'debug' enabled. 2019-05-06 21:27:18.275 ERROR 10968 --- [ main] o.s.b.d.LoggingFailureAnalysisReporter : 
*************************** 
APPLICATION FAILED TO START 
*************************** 

Description: Failed to configure a DataSource: 'url' attribute is not specified and no embedded datasource could be configured. 

Reason: Failed to determine a suitable driver class 

Action: 

Consider the following: 
		If you want an embedded database (H2, HSQL or Derby), please put it on the classpath. 
		If you have database settings to be loaded from a particular profile you may need to activate it (no profiles are currently active). 

Process finished with exit code 1

```

> 应用中没有使用DataSource, 但是在pom中依赖`mybatis-spring-boot-starter`，启动会触发springboot自动初始化加载DataSource。

#### 解决
- 方案一：pom中注释掉 `mybatis-spring-boot-starter` 依赖。

- 方案二：禁止启动类启动关于Datasource的加载
	- 在启动类的@SpringBootApplication加上

	```
	    @SpringBootApplication(exclude = { DataSourceAutoConfiguration.class, DataSourceTransactionManagerAutoConfiguration.class })
	```
	- 或者在application.properties里配置
	
	```
spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration
	```

