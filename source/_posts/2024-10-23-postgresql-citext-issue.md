---
title: Postgres citext类型忽略大小写查询时遇到的坑
date: 2024-10-23 17:58:54
tags:
  - Postgresql
---

最近项目里有个需求是模糊查询某个字段时需要忽略大小写，一开始的实现很简单，用pg自带的LOWER函数，`WHERE LOWER(name) LIKE LOWER(’%’ || #{query.name} || ‘%’)` 但随着要修改的sql越来越多，这种方式容易遗漏，于是就想办法能不能在列上做一些事情，一劳永逸。
<!--more-->
经过一番搜索发现pg有一种[citext](https://www.postgresql.org/docs/current/citext.html)的列类型，支持忽略大小写搜索，于是马上尝试把列类型改成了citext，改完重启服务测试，居然查不到记录。怀疑是不是sql有问题，于是复制了mybatis打印的sql去数据库里执行。诡异的事情来了，sql的执行是能查到记录的，也就是说sql没问题，问题出在mybatis，或者说是jdbc层。

顺着这个思路搜索，找到一篇pg社区的[问题解答](https://www.postgresql.org/message-id/CAJFs0QC_nn5WxhrgMuXsK=WCc5JHvMmGk+zHoiwLz-EG7W2a4A@mail.gmail.com)，大概意思是

> 问题的核心出在jdbc在发送query到pg server时指定查询参数的类型，在prepareStatement.setString时jdbc会有个判断，设置查询参数为varchar还是unspecified。如果设置了varchar，那将会丢失citext的忽略大小写特性，pg会按照varchar类型的行为处理。
>

同时在上面的回答里还给了解决方案，在jdbc connection url的末尾加上`stringtype=unspecified` 。至于这个标记的作用，可以在pg的[这篇官方文档](https://jdbc.postgresql.org/documentation/use/#connection-parameters)里查到。


