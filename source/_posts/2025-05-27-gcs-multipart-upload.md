---
title: Google cloud storage multipart upload遇到的坑
date: 2025-05-27 16:06:42
tags:
  - Google Cloud
---

# 背景
最近在优化上传本地文件的功能，目前的实现是通过resumable upload，实现了串行分块上传，但是在上传大文件时的速度有点慢，不能充分利用上带宽，于是在调研了一波后，决定改成gcs提供的另一种multipart upload，可以并行上传多个分段，加速上传时间。
<!--more-->
# 前置
根据gcs的[文档](https://cloud.google.com/storage/docs/xml-api/post-object-multipart)指示，步骤是

1. 先init multipart upload，得到uploadId，

2然后再并行调用PUT接口上传分片，这一步会在response headers里返回Etag，需要把所有分片的Etag都保存下来，

3. 最后在complete接口里组装成一个xml body。根据文档提供的例子很快把第一版代码写好了。

# 坑1
文档里的例子Host的结构为`bucket.storage.googleapis.com` ，在浏览器上使用fetch调用会报错CORS，于是查看Host字段的说明，发现可以配置custom domain，于是在GCP里配置了load balance，在cloudflare里配置了dns，最后调用接口为`https://opus-clip-staging.opus.pro`，但是调用还是报错，这回不是CORS，而是404，错误信息是no such object in bucket opus-clip-staging.opus.pro，貌似GCS把我配置的全域名当作bucket名称了，这里也不清楚GCS是如何解析bucket名称的，神奇的是调用第二步的put接口时，GCS又能正确识别bucket，只有在调用第一步和第三步时会把bucket认错。这个错误目前还是有办法解决的，只需要在我们自己的后端服务上提供2个接口，接口里调用GCS的真正的接口，前端在第一步和第三步调用后端服务的接口即可。

# 坑2
在把接口搬到后端前，我打算先把complete接口调通，于是在Apifox里尝试调用接口如下

![](1.webp)
这里的xml body我尝试过删除全部行，去掉多余符号，都是报这个错误，谷歌搜了好多都搜不到这个错误，于是求助GCP的同事，看看他们有什么解决办法

# 解决
他们一开始提供了一个[博客](https://medium.com/@rosyparmar/google-cloud-storage-upload-a-large-file-using-curl-872dd1e4e323)，我尝试按着博客上的例子在curl里请求，最后成功调通了，于是决定按着博客的方式修改。首先修改调用接口host，改回`bucket.storage.googleapis.com` ，接着搜索GCS cors，搜到[官方文档](https://cloud.google.com/storage/docs/cors-configurations)的配置，一通配置下来，发现调用还是CORS，这时GCP的同事也没有其他解决办法，于是只好继续看文档，这时我在另一篇[文档](https://cloud.google.com/storage/docs/cross-origin)里发现了这样一段

![](2.webp)
大致意思是有些请求会先preflight，且preflight里会携带 requested headers，GCS会检查requested headers，如果检查不通过，就会直接拒绝，那么这里的检查目标又是什么呢？继续往下看，又出现一段话

![](3.webp)
原来是跟cors配置里的responseHeaders的值做检查，那这时候就需要看一下OPTIONS请求里的值到底是什么，于是点开Chrome network，点击上传

![](4.webp)
什么OPTIONS，哪来的OPTIONS？Chrome根本就不显示OPTIONS。ok那我们换个浏览器

![](5.webp)
![](6.webp)
可以看到在Access-Control-Request-Headers有2个值，ok那我们在cors config里把这2个值加上再试一次（这里还有个点要注意，Etag在response header里，想要在前端获取到header，需要Etag在response里的Access-Control-Expose-Headers里，但是cors config里并没有这个配置字段，仔细看了一下接口返回的Access-Control-Expose-Headers里有个authorize，跟cors config里配置的responseHeaders里的一样，于是抱着试一试的想法把Etag也加进去，发现是可以的。以及每个part不能小于5mb，否则在最后的complete接口会返回invalid argument错误）

![](7.webp)
完美通过


