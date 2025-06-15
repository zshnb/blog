---
title: 把telegram变成自己的私人图床
date: 2025-06-15 22:18:10
tags:
  - Cloudflare
---

最近看到一个[开源项目](https://github.com/xiyewuqiu/new-lmage)，通过把Cloudflare和电报结合，打造成属于自己的私人图床，效果如下
<!--more-->
![](img1.webp)
它的优势是免费，无限空间，永不过期，下面教大家如何配置。

### 前置条件
1. Cloudflare账号

1. 电报账号



### 步骤
cloudflare配置

1. 首先克隆[开源项目](https://github.com/xiyewuqiu/new-lmage)到本地

1. 进入项目目录，命令行执行`npm install`

1. 执行`npx wrangler login` ，会自动跳转浏览器打开cloudflare网页，确认授权即可

1. 执行`npx wrangler kv namespace create "img_url"` 和`npx wrangler kv namespace create "users"`，记下执行后命令行打印的两个id
   ![](img2.webp)



电报配置

1. 搜索BotFather，输入/newbot，创建完bot后记录下bot token
   ![](img3.webp)

1. 创建一个channel，把上一部创建的bot添加进channel，这里搜索的时候最好填入完整的bot info里的username
   ![](img4.webp)

1. 再在channel里添加一个get_id_bot，然后在channel里触发get_id_bot的功能，获取到channel的id，其格式是-100后面随机的数字



最后需要修改项目根目录下wrangler.toml文件，把前面记录的几个id分别填进去

![](img5.webp)
填完之后执行`npm run deploy` ，然后进入cloudflare的compute workers，就能看到正在运行的网站了，点击visit即可访问。

![](img6.webp)


