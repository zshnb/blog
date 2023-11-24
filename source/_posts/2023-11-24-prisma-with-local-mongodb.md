---
title: 使用prisma连接本地mongodb
date: 2023-11-24 20:40:26
tags:
  - Mongodb
  - Prisma
  - Javascript
---

最近有个项目是用nest.js作为后端框架，数据库是mongodb，ORM框架选择了目前nest.js圈最流行的prisma。项目本身很简单，在本地开发调试的时候出现了一个问题。
当我调用`prismaService.save()`时，报错Transactions are not supported by this deployment，大意是mongodb的事务操作要求必须有replica，即使是在本地开发。
搜索之后找到一个[issue](https://github.com/prisma/prisma/issues/8266)，找到一个配置本地docker mongodb的replica参数
```text
1 - start a mongo container
docker run --name mongodb -d -p 27017:27017 mongo mongod --replSet rs0

2 - After your mongodb container is up and running, enter mongosh
docker exec -it mongodb mongosh

3 - Inside mongosh, initiate replica set
rs.initiate({_id: 'rs0', members: [{_id: 0, host: 'localhost:27017'}]})
```
配置完再执行事务就没问题了。