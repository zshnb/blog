---
title: 如何部署NestJS应用到Vercel
date: 2025-03-12 23:11:03
tags:
  - Nestjs
  - Vercel
---

最近在帮助一个客户开发后端程序，技术栈是Node.js+NestJS+Prisma+Planetscale，客户希望能部署到Vercel。而Vercel基本都是用来部署前端应用的，以前我也没有尝试过在Vercel部署后端应用，这次顺便记录一下部署过程，以及中间的几个坑。
<!--more-->
## 部署过程
### 配置Vercel
1. `touch vercel.json` 在项目根目录下新建配置文件

2. 在vercel.json里写入以下内容，builds和routes的src目录根据自己项目配置修改
    ```
    {
        "version": 2,
        "builds": [
            {
            "src": "dist/main.js",
            "use": "@vercel/node"
            }
        ],
        "routes": [
            {
            "src": "/(.*)",
            "dest": "dist/main.js"
            }
        ]
    }

    ```

3. `npm i -g vercel`安装vercel命令行

4. `vercel login` 登录vercel，这一步基本上输入邮箱即可

5. `npm run build` 打包应用

6. `vercel --prod` 开始部署，注意如果是第一次部署，会有选项让你选择，类似下图
   ![](img1.webp)

7. 最后就可以在vercel的dashboard里看到应用了。

## 踩坑
### prisma连接不上
项目的数据库是planetscale的云数据库，在prisma的[文档](https://www.prisma.io/docs/accelerate/getting-started)里提到连接planetscale要用DIRECT_DATABASE_URL，实测这样还是会报错，必须还是用DATABASE_URL，且格式是mysql://才行。

### prisma缓存
部署成功后在vercel的日志里看到

`Prisma has detected that this project was built on Vercel, which caches dependencies.
This leads to an outdated Prisma Client because Prisma's auto-generation isn't triggered.
To fix this, make sure to run the prisma generate command during the build process.`

看了官方文档的[解决方案](https://www.prisma.io/docs/orm/more/help-and-troubleshooting/help-articles/vercel-caching-issue)，才知道需要在npm的build脚本前面加`prisma generate` ，也就是

```
"scripts" {
   "build": "prisma generate && <actual-build-command>"
 }

```
### swagger
项目配置了swagger，本地运行完全没问题，部署到vercel后swagger页面打不开了，一看network全是404，猜测是vercel的机制导致swagger找不到生成的静态文件了，解决办法如下：

```
SwaggerModule.setup('/swagger', app, swaggerDocument, {
    customCssUrl:
      'https://cdnjs.cloudflare.com/ajax/libs/swagger-ui/4.15.5/swagger-ui.min.css',
    customJs: [
      'https://cdnjs.cloudflare.com/ajax/libs/swagger-ui/4.15.5/swagger-ui-bundle.js',
      'https://cdnjs.cloudflare.com/ajax/libs/swagger-ui/4.15.5/swagger-ui-standalone-preset.js',
    ],
});

```

