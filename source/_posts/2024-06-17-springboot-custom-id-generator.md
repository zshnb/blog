---
title: SpringBoot JPA 自定义ID生成策略
date: 2024-06-17 17:41:56
tags:
  - Java
  - SpringBoot
---

在JPA中，我们是通过`@id`和`@GeneratedValue`来指定id主键生成策略的，比如：

```
@Id
@GeneratedValue(strategy = GenerationType.AUTO)
@Column(name = "id")
private String id;

```
<!--more-->
JPA内置提供了4种策略，分别是：

- **TABLE**：使用一个特定的数据库表格来保存主键。

- **SEQUENCE**：根据底层数据库的序列来生成主键，条件是数据库支持序列。

- **IDENTITY**：主键由数据库自动生成（主要是自动增长型）

- **AUTO**：主键由程序控制(也是默认的,在指定主键时，如果不指定主键生成策略，默认为AUTO)

当然，很多时候，这么几种策略并不够用，所以JPA还提供了扩展，允许我们自定义id生成策略，

具体使用就是多了一个`@GenericGenerator`注解，指定自定义名称以及策略，然后在`@GeneratedValue`中使用该策略，比如：

```
@Id
@GeneratedValue(generator  = "customId") // 这里注意generator里的值要和GenericGenerator的name一致
@GenericGenerator(name = "customId", strategy = "com.zshnb.config.CustomIdGenerator")
private String id;

```
```
public class CustomIdGenerator implements IdentifierGenerator{
   /*
     @param o: 表示当前保存的entity对象
   */
   @Override
    public Serializable generate(SessionImplementor sessionImplementor, Object o) throws HibernateException {
        String date = String.valueOf(new Date().getTime());
        return date.slice(4);
    }
}

```
如果这时候希望在`CustomIdGenerator`里和SpringBoot集成，比如调用其他component的方法生成id，有以下类：

```
@Component
public class IdGenerator {
    public String getId() {
        // 查数据库，生成自定义前缀的id
        return "";
    }
}

```
这时候我们希望能在`CustomIdGenerator`里注入`IdGenerator`，调用getId方法，但`CustomIdGenerator`是hibernate管理的，没办法参与Spring的IOC机制，有什么办法呢？可以通过以下方式解决，首先新建`ApplicationContextHolder`类

```
@Component
public class ApplicationContextHolder implements ApplicationContextAware {

    private static ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        synchronized (this) {
            if (ApplicationContextHolder.applicationContext == null) {
                ApplicationContextHolder.applicationContext = applicationContext;
            }
        }
    }

    public static <T> T getBean(Class<T> clazz) {
        return applicationContext.getBean(clazz);
    }

    public static <T> T getBean(String qualifier, Class<T> clazz) {
        return applicationContext.getBean(qualifier, clazz);
    }

```
然后修改CustomIdGenerator，获取需要注入的对象即可。

```
public class CustomIdGenerator implements IdentifierGenerator{
   /*
     @param o: 表示当前保存的entity对象
   */
   @Override
    public Serializable generate(SessionImplementor sessionImplementor, Object o) throws HibernateException {
        IdGenerator idGenerator = ApplicationContextHolder.getBean(IdGenerator.class);
        return idGenerator.getId();
    }
}

```