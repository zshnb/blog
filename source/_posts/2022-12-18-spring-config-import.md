---
title: SpringBoot import config属性加载顺序踩坑
date: 2022-12-18 21:10
tags:
- Spring
---

# 背景

SpringBoot2.4.0添加了`spring.config.import`配置项，可以在配置文件里导入其他配置文件，通常用来抽取一些所有profile都会使用的配置，比如公共服务器之类的，下面用一个demo项目演示一下。
<!--more-->
# 复现
首先有3个配置文件

```yaml
# application-base.yml
common:
  name: zsh
```

```yaml
# application-dev.yml
spring:
  config:
    import: application-base.yml
```

```yaml
# application-prod.yml
spring:
  config:
    import: application-base.yml
```

然后指定profile为dev运行，打印common.name配置的值

![结果](img1.png)

可以看到打印出了我们配置的值。

# 问题

很多时候我们希望在active profile里覆盖公共的一些配置，比如MySQL url之类的，这里我们覆盖common.name，把上面3个配置文件稍作修改

```yaml
# application-dev.yml
spring:
  config:
    import: application-base.yml

common:
  name: from dev
```

然后再次运行

![结果](img2.png)

奇怪的事情发生了，运行的结果并没有按照我们希望的输出"from dev"，依然还是"zsh"，这是为什么呢？我们来翻一下SpringBoot官方的说明：

> Imports can be considered as additional documents inserted just below the document that declares them. They follow the same top-down ordering as regular multi-document files: An import will only be imported once, no matter how many times it is declared.

粗看官方的解释，第一反应是SpringBoot会按照下面的方式转换导入的配置文件

```yaml
# application-dev.yml
# before
spring:
  config:
    import: application-base.yml

common:
  name: from dev
# after
spring:
  config:
    import: application-base.yml
common:
	name: zsh
common:
  name: from dev
```

按照上面的行为，最后打印出来的应该是"from dev"，但是实际结果好像是下面这样

```yaml
# application-dev.yml
# before
spring:
  config:
    import: application-base.yml

common:
  name: from dev
# after
spring:
  config:
    import: application-base.yml

common:
  name: from dev
  
common:
	name: zsh
```

于是我又去翻官方有关回答，找到了下面一段解释

> In  that example you only actually have one `spring.config.import`  statement. The second one in the properties file will replace the first.
>
> Imports will always happen in the order that they are specified, but the  granularity is the current document being processed. So in your example, it wouldn't matter if you had:
>
> ```
> spring.config.import=my.propertiesmy.property=value
> ```
>
> or
>
> ```
> my.property=valuespring.config.import=my.properties
> ```
>
> The result is the same. The `[my.property](http://disq.us/url?url=http%3A%2F%2Fmy.property%3A3zPwzcFT2x4NYuqlf_XVTdE2BeE&cuid=2935315)` value will be `anotherValue`.
>
> When working on the design, I liked to visualize this a deck of cards. As we read the files, we build up a stack of cards. The first card we process has a `spring.config.import` statement of `[my.properties](http://disq.us/url?url=http%3A%2F%2Fmy.properties%3AeHnV6OOS4iwu4hgYpanuM02s270&cuid=2935315)`. You go and fetch the "[my.properties](http://disq.us/url?url=http%3A%2F%2Fmy.properties%3AeHnV6OOS4iwu4hgYpanuM02s270&cuid=2935315)" card and place it on top. When you're done, there's a stack of cards  and the first one you find containing the property name you want will  win. I hope that helps, and doesn't make things more confusing :)

明确说明import config的配置会覆盖当前配置文件中的配置。

# 结论

如果你的项目存在多套配置文件，且有公共配置文件，注意千万不要把会被覆盖的配置写在公共配置文件内，不然其他引用了公共配置文件的配置文件，将无法覆盖此配置，因此如果某个配置存在被覆盖的可能，请将它单独写在每个启动时引用的配置文件内。
