---
title: 关于双写缓存一致性的思考
date: 2021-10-04 08:30
tags:
- MySQL
- Redis
---

缓存数据库双写一致性一直是面试的一个高频问题，网上关于这个问题的文章也非常多，大家的观点都不一致。这几天在看了十几篇文章后，再加上一些自己的思考，决定写下来供大家一起讨论。

双写缓存一致性通常指的是1份数据要往缓存（Redis）和数据库（MySQL）里写，本质就是2个写的操作不是原子性的。因此我们可以从下面2个角度去思考
1. 在无法达到原子性的前提下，哪一步操作失败危害最低？在高并发下的情况哪一种又会更好？
2. 让2个写的操作原子性

下面我们分别展开，首先从第1点开始。

通用的数据库缓存读写模型大致是这样

```flow
read=>start: 读请求
exist=>condition: 缓存中有数据？
no=>inputoutput: 读取数据库数据并放入缓存
yes=>inputoutput: 返回缓存中数据
e=>end: 结束
read->exist
exist(no)->no->e
exist(yes)->yes->e

```



```flow
read=>start: 写请求
write_db=>inputoutput: 操作数据库
write_cache=>inputoutput: 操作缓存
e=>end: 结束
read->write_db
write_db->write_cache
write_cache->e
```

在写请求里的操作缓存有2种策略，删除或者更新。如果缓存的数据是很简单的计算结果，那可以选择更新，防止miss，如果缓存数据需要很复杂的计算，那可以选择删除，可以节省cpu资源，缺点是会有miss，目前绝大多数都会选择删除缓存，下文也默认使用删除缓存策略。

除了缓存操作策略，还有1个讨论的点是缓存的删除是在操作数据库前还是后。我们用开头提到的`哪一步操作失败危害最低？在高并发下的情况哪一种又会更好？`这2个问题分别讨论一下。

- 哪一步操作失败危害最低
    - 先删缓存失败，缓存里是旧数据，数据库里是新数据，缓存不一致，需要等待缓存过期或者下次触发缓存删除才能让数据一致
    - 操作数据库失败，数据库里是旧数据，缓存里是旧数据，缓存一致
    - 因此从这个角度看先操作数据库危害最低
- 高并发下
    - 先删除缓存。线程A发起写请求，删除了缓存，此时线程B发起读请求，读取数据库旧数据并放入缓存，线程A操作完数据库，缓存里为旧数据，缓存不一致
    - 先操作数据库。线程A发起写请求，线程B发起读请求，此时缓存刚好失效，线程B读取数据库旧数据，线程A操作完数据库后删除缓存，线程B将旧数据放入缓存，缓存不一致
    - 第2种情况发生的条件是数据库写请求要比读请求先完成，这种情况发生的概率是很小的，一般情况下数据库读肯定是比写要快，所以我认为先操作数据库优于先删除缓存

上面2个问题得出的都是先操作数据库优于先删除缓存，目前大多数人都是认为该方案较优。[代码实现](https://github.com/zshnb/interviewpractice/blob/master/src/main/java/com/zshnb/interviewpractice/cache_consistency/DelayedDeleteCacheStrategy.java)
同时对缓存key的删除失败情况，可以选择简单重试，也就是延迟双删，或者使用消息队列记录删除失败的key，待后续继续处理

接着讲第二点，让2个写操作原子性，有3种思路

1. 读写都加排他锁

   通过redis分布式锁，在读和写之前都要加锁，只有获取到锁才可以进行下一步操作，优点是简单，缺点是并发度较差。[代码实现](https://github.com/zshnb/interviewpractice/blob/master/src/main/java/com/zshnb/interviewpractice/cache_consistency/WithLockStrategy.java)

2. 订阅binlog

   写数据库操作后不执行删除缓存，通过另外的线程或者服务订阅binlog，一旦有缓存需求的表发生数据变动，删除缓存，优点是对业务无侵入，缺点是需要额外维护binlog服务。[代码实现](https://github.com/zshnb/interviewpractice/blob/master/src/main/java/com/zshnb/interviewpractice/cache_consistency/BinlogStrategy.java)

3. 标记失效

   借鉴volatile的思想，在数据库中新建一张`cache_info`表，有`cache_key`和`valid`列，分别表示缓存key名字和是否有效。写数据库操作后将缓存的`valid`设置为`false`，读缓存前先去查找缓存`key`对应的`valid`，如果是`false`表示缓存失效，需要重新计算缓存，如果是`true`则返回缓存值。[代码实现](https://github.com/zshnb/interviewpractice/blob/master/src/main/java/com/zshnb/interviewpractice/cache_consistency/DisableInDatabaseStrategy.java)

