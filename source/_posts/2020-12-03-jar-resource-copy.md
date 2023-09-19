---
title: 如何复制Jar包内资源到其他地方
date: 2020-12-03 21:20
tags:
- Kotlin
---

# 背景

最近一直在做自己的代码生成器项目，遇到一个需求，需要把线上运行Jar包里resources文件夹下的某些文件夹按照原本的文件结构，复制到Jar包外的另一处位置。
<!--more-->

# 解决

一开始在IDEA里开发很简单，调用`FileUtils.copyDirectory(src, dest)`就解决了，本地调试运行也很顺利，文件都按照预想的复制过去了，接下来准备打包部署到服务器上试一下，这一试就试出问题了，直接抛了异常如下
**class path resource xxx cannot be resolved to absolute file path because it does not reside in the file system**
搜索了一下，才知道Jar包里的文件是不能被当作文件系统里的文件来处理的，而只能用流的方式去处理。但试了一圈也没发现有什么简单的方法可以把Jar包下某个文件夹的流复制到Jar包外，
不过倒是让我找到一个方法`FileUtils.copyURLToFile(url, file)`，按着这个方法想到了一个思路，**不关心文件夹，而是把文件夹下的文件独立地通过流复制到目的地。**
spring本身有许多方便的方法去处理ClassPathResource，通过ClassPathResource可以拿到对应的InputStream，于是便有了以下思路

1. 获取Jar包某些文件夹下所有文件的流
2. 筛选需要复制的流
3. 复制
```kotlin
    // 通过spring提供的方法，用匹配路径字符串获取需要复制的resources
    val resourceResolver = PathMatchingResourcePatternResolver()
    val resources = resourceResolver.getResources("/templates/layui/**")
    resources.filter { ReUtil.isMatch(".*?\\.[a-zA-Z]*?", it.filename!!) } // 进一步筛选需要复制哪些文件，同时过滤掉文件夹路径的resource
        .forEach {
            val url = it.url
            val filePath = url.path.substring(url.path.indexOf("layui") + 5)
            if (!filePath.contains("page")) {
                val destination = File("${pathConstant.resourcesDirPath()}/templates/$filePath")
                FileUtils.copyURLToFile(url, destination) // 复制
            }
        }
```

最后打包测试，一切OK。
