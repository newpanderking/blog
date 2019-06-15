---
layout: post
title:  "mvn deploy jar to remote repository"
date: 2019-06-15
categories: 技术
tags: mvn 
description: mvn deploy jar to remote repository 
---

###  mvn deploy jar to remote repository

> 目标：mvn工程中在本地执行mvn deploy 命令时可以直接jar包上传到远程仓库。

#### 步骤
- mvn的主pom文件中加入如下代码

```
<distributionManagement>
<repository>
<id>releases</id>
<name>internal releases</name>
<url>xxxxxxxxxxxxxxx</url>
</repository>
<snapshotRepository>
<id>snapshots</id>
<name>internal snapshot</name>
<url>xxxxxxxxxxxxxx</url>
</snapshotRepository>
</distributionManagement>
```

- setting.xml文件中配置如下

```
<servers>
<server>
<id>releases</id>
<username>userName</username>
<password>password</password>
</server>
<server>
<id>snapshots</id>
<username>userName</username>
<password>password</password>
</server>
</servers>
```


> 说明，在setting.xml文件中配置的<server>标签下的<id>标签内的内容，和mvn-pom文件中<repository>标签下的<id>标签同匹配。

- tips: mvn命令指定使用setting.xml文件时， mvn -s "/path/setting.xml" deploy/install/package
