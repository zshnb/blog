---
title: iOS同步高德地图收藏夹到Notion
date: 2024-05-29 21:55:06
tags:
  - 快捷指令
---

最近在小红书看到一位博主吐槽自己的高德地图收藏无法编辑，也无法查看，一点击收藏地点就会自动取消收藏，这可能是高德地图的bug，同时也让我有了一个想法，就是把高德地图的收藏地点全部同步到Notion中，有几个好处：
<!--more-->
1. 不用担心高德地图的bug导致收藏消失或者无法查看

1. 可以很方便地根据城市筛选收藏地点

1. 可以跟Notion其他页面联动

![高德收藏夹](img1.webp)
## 教程
1. 下载[快捷指令](https://www.icloud.com/shortcuts/40207921589145418bd7cf9611a19dd1)。

1. 复制[Notion模板](https://zhengsihua.notion.site/2753dd41aa284370b9b65b3933388b06?)到自己的workspace。
   ![ALT](img2.webp)

1. 在浏览器中访问刚才复制的模板页面，然后查看database，复制databaseId，记住databaseId的地址后面是带?v的，如果没看到，说明地址不对。
   ![ALT](img3.webp)
   ![ALT](img6.webp)

1. 访问[Notion integration页面](https://www.notion.so/my-integrations)，新建一个integration，之后点击show再点copy，复制secret key
   ![ALT](img4.webp)

1. 把前面复制的databaseId和secret key分别替换快捷指令里的databaseId和key
   ![词典](img5.webp)



大功告成。
