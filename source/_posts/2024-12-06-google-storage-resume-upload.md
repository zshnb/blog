---
title: Google cloud storage resume upload的坑
date: 2024-12-06 17:00:15
tags:
  - Google Cloud
---

最近在做一个需求，网站需要支持用户上传本地视频文件做自动剪辑，因为公司用的云服务是Google cloud，自然选择接入Google cloud storage，于是经过一番调研，选择了resumable upload这个方案。在正式接入前端前，我打算先跟着官网文档，把curl的部分走通。
<!--more-->
![](1.webp)


一开始按照文档提示，去掉了—data-binary和content-length用curl测试，发现接口没有任何返回值

![](2.webp)
![](7.webp)
然后尝试在前端用fetch测试，一直报错400 parseError，这里强烈吐槽谷歌浏览器，400错误的response里是不会显示任何返回的数据的，非常不友好，下图是Firefox测试返回的400错误信息

![](3.webp)
经过一番搜索还是无法找到问题，于是用[google playground](https://developers.google.com/oauthplayground/?code=4/0AbUR2VN02NRYuxRTt0IY8Pl_X2s-fxDr7KhhpYf2ckHQxwIuJH4Z0Z_F_edLp7fNhZWIvw&scope=https://www.googleapis.com/auth/devstorage.read_write)工具测试upload接口，发现接口成功返回了

![](4.webp)
仔细对比了一下，发现playground里用的是http1.1，于是我把curl也改成了http1.1，发现接口也调用成功了

![](5.webp)
但浏览器又不能强制用http1.1，于是继续研究，经过不断测试，比如改contentType，去掉contentLength，都不行，最后发现playground里有个`content-length: 0` ，抱着试一试的心态测试了一下，居然可以了

![](6.webp)
想不到浓眉大眼的谷歌写的文档也有这么多坑。
