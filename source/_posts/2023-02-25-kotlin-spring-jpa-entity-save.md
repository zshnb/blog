---
title: Kotlin实体使用JPA的save自动设置id问题
date: 2023-02-24 15:21
tags:
- Kotlin
- Spring JPA
- Java
---

# 背景
最近打算把公司后端项目从Java迁移到Kotlin，在迁移JPA实体类的时候，用data class代替了Java定义的class，同时用var+默认值的方式改写了id的定义
<!--more-->
```kotlin
data class User(
    @Id
    @GenerateStrategy(xxxStrategy.class)
    var id: String = ""
)
```
改完后上线发现jpaRepository.save方法不会对实体参数自动设置id了，于是便开始debug

# 探寻
跟踪jpaRepository.save
```java
@Transactional
@Override
public <S extends T> S save(S entity) {

    Assert.notNull(entity, "Entity must not be null.");

    if (entityInformation.isNew(entity)) {
        em.persist(entity);
        return entity;
    } else {
        return em.merge(entity);
    }
}
```
发现有个isNew的判断，继续跟踪isNew
```java
public boolean isNew(T entity) {
    ID id = getId(entity);
    Class<ID> idType = getIdType();

    if (!idType.isPrimitive()) {
        return id == null;
    }

    if (id instanceof Number) {
        return ((Number) id).longValue() == 0L;
    }

    throw new IllegalArgumentException(String.format("Unsupported primitive id type %s!", idType));
}
```
默认实现是通过id类型判断，如果是number类型，判断是否为0，否则判断是否为null。
看到这思路已经出来大半了，上面data class定义的id类型是String，由于Kotlin语法要求，非空类型 设置了默认值，于是isNew先进入了第一个if，然后返回false，那么save方法就进入了merge的分支。
而Java版本没有设置默认值，所以默认为null，isNew返回true，区别就在这里。由于传入的实体已经存在id（虽然没有值）于是merge操作便不会给传入的实体赋值生成的id。
# 解决
问题都查清楚了，那要怎么解决这个问题呢
1. 使用number类型作为主键
2. 使用Kotlin可空类型作为主键
3. 不要依赖参数自动设置，强制使用返回值作为后续使用的实体