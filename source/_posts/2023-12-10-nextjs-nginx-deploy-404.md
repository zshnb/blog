---
title: Next.js部署nginx配置子路径访问404
date: 2023-12-10 18:07:34
tags:
  - React
  - Nginx
  - Next.js
---

周末没事干，正好被网上一个点子启发，做了个[人生进度可视化](https://zshnb.com/lifetime)的网页，网页功能比较简单，就是把每一天通过一个小格子显示在网页上，不同的阶段显示不同的颜色，同时也能切换时间单位。
![screenshots](img1.png)
<!--more-->
但是在部署的时候遇到点问题。一开始选择的是next.js的static export模式，导出静态html文件，在nginx里配置location如下
```text
location /lifetime {
    root /home/ubuntu/root
		index index.html
}

```
访问404，尝试了好久都找不到原因，遂放弃，改成next.js node server模式
按照官网的指导，重新配置nginx如下
```text
location /lifetime {
		proxy_set_header Host $http_host;
	  proxy_set_header X-Real-IP $remote_addr;
	  proxy_pass http://localhost:3001;
}

```
这次不仅404，连静态资源都访问不到了，一阵搜索后找到静态资源的配置，原来next在node模式下静态资源有不同的访问地址
```text
location /_next/static/ {
        alias /home/ubuntu/root/.next/static/;
        expires 365d;
        access_log off;
}

```
但网页访问还是404，突然想到会不会跟我配置访问链接有子路径有关系，因为官网的例子是没有子路径的，于是尝试搜索next.js nginx deploy with subpath，找到一个<u>回答</u>，原来是要在next.config.js 里配置basePath，因为根据nginx的配置，生成的访问next.js的链接是http://localhost:3001/lifetime，而默认next.js访问路径是没有前缀的，所以自然就404了。最后生效的配置如下
```text
location /lifetime/_next/static/ {
        alias /home/ubuntu/root/.next/static/;
        expires 365d;
        access_log off;
}
location /lifetime {
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_pass http://localhost:3001;
}

```
当然这里还有另一个解决办法是通过rewrite指令，修改proxy_pass后的url。