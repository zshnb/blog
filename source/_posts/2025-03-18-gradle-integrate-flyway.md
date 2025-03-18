---
title: Springboot, Gradle, Postgresql接入flyway报错No database found to handle
date: 2025-03-18 21:00:08
tags:
  - SpringBoot
  - Gradle
  - Flyway
---

# 背景
项目是Springboot框架，使用gradle作为构建工具，数据库是postgresql。

今天在尝试接入flyway作为数据库迁移工具，按照[官方教程](https://documentation.red-gate.com/fd/postgresql-database-235241807.html)在build.gradle配置好后，运行flywayInfo报错No database found to handle，经过一番搜索找到了[这个PR](https://github.com/flyway/flyway/issues/3774)，需要在build.gradle里最开头加上

```
buildscript {
    dependencies {
        classpath("org.flywaydb:flyway-database-postgresql:10.13.0")
    }
}

```
更奇怪的是这个bug也一直没有修复，或者也没有在官方文档里加上修复方案，但PR却被关闭了

