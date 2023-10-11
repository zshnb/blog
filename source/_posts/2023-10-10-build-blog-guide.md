---
title: Hexo博客搭建指南
date: 2023-10-10 18:14:10
tags:
- hexo
---

# 背景
目前我个人的博客是托管在Github Pages下，用的也是Github提供的免费域名，但每个程序员都有个梦想，就是有一个属于自己的博客。
于是就决定趁着这段时间不忙，把Github Pages上的博客迁移到自己的服务器上，同时把这个过程记录下来，也能帮助到想要拥有自己博客的人。
<!--more-->
# 需求清单
这里先简单陈列一下，搭建一个个人博客所需要的所有东西
1. 云服务器，用于运行博客网站
2. 域名，用于用户可以方便地访问你的博客，你也不想访问别人网站的时候，还要记住毫无规律的IP吧
3. 备案（可选），如果服务器在大陆，那么这个是逃不了的，没有备案分分钟网站关停
4. Github账号，用于存储博客

接下来按照优先等级一个个详细介绍
# 博客仓库
## 初始化
第一步我们先要有一个博客仓库，用来储存所有的文章，这里我选择的是[Hexo博客框架](https://hexo.io/)，是最近比较热门的博客框架。Hexo可以选择网站的主题，
这里我选择的是[nexT主题](https://theme-next.js.org/docs/getting-started/)，也是Hexo框架里非常热门的主题。
按照Hexo文档里的提示，我们执行
1. `npm install -g hexo-cli`，安装hexo命令行工具。注意Hexo要求系统已经安装Node.js和Git，如果没有安装过，
可以移步各自的官方文档进行安装。
2. 接着执行`hexo init xxx-blog`，生成hexo项目
3. `cd xxx-blog && npm install`，然后用任意编辑器打开xxx-blog文件夹，你会看到这样的目录![](img1.png)，说明博客仓库初始化成功了
4. `hexo s`，看到控制台打印的![](img3.png)，说明博客运行成功了，访问localhost:4000就能看到博客页面了
5. `hexo g && hexo d`，部署博客到Github仓库，这里hexo会把生成的静态文件push到另一个分支，默认是gh-pages，因此我们需要分清楚，
hexo的main分支是存放我们的md文章，以及hexo的配置，gh-pages分支是存放hexo generate后的html等静态文件，用于页面展示的。

## 基础配置
接着需要对hexo进行一些配置，hexo的配置文件在./_config.yml，下面列举几个比较重要的配置，其他配置都可以参考官方文档
1. [网站基础信息配置](https://hexo.io/docs/configuration#Site)，这里需要配置网站标题，作者等信息
2. [静态资源配置](https://hexo.io/docs/asset-folders)，这里配置文章里引用的图片
3. [Git部署配置](https://hexo.io/docs/one-command-deployment)，这里配置博客部署发布的Git信息
如果需要 **标签(tags)**、**关于(about)**、**搜索(search)** 页面，需要在source下面新建文件夹，然后新建index.md文件，以标签为例子
```markdown
// 路径：./source/tags/index.md
---
title: tags
date: 2023-09-13 15:35:31
type: tags
---
```

## 主题配置
nexT官方推荐的配置文件是./config.next.yml，如果懒得看官方文档，可以直接使用我这份[配置文件](https://github.com/zshnb/blog/blob/main/_config.next.yml)

## SEO配置
参考[官方文档](https://theme-next.js.org/docs/theme-settings/seo)进行基础配置，接着配置各个搜索引擎的收录，这方面的资料很多，可以参考[这篇](https://blog.juanertu.com/archives/9013c8d8)

# 上线
有了博客仓库后，接下来要做的事就是把网站上线到互联网，让大家访问了
## 云服务器
国内可以选购腾讯云，阿里云，华为云等，一般博客不需要很好的配置，1核2G足以，这里我用的是腾讯云。购买成功后，需要将自己的ssh 公钥加进服务器里。
本地公钥文件在~/.ssh/id_rsa.pub里，然后在腾讯云云服务器里绑定
![](img4.png)
创建成功后在密钥列表里选择刚才创建的密钥，绑定服务器即可
## 域名
最好在你购买的云服务器厂商里注册，绑定解析什么的比较方便，后缀推荐.com|.org，最好不要选择.cn，注册好之后在域名服务商里添加解析
![](img2.png)
## 部署
通过ssh登录服务器后
1. 首先安装nginx`sudo apt update && sudo apt install nginx`
接着启动nginx`sudo systemctl start nginx && sudo systemctl enable nginx`。nginx的配置文件在/etc/nginx下
2. 配置https证书，这里我们使用免费的[Let's encrypt](https://certbot.eff.org/instructions?ws=nginx&os=ubuntufocal)，注意如果在这个步骤遇到了域名无法访问的问题，请先往下解决备案后，再回来生成证书。
按照Let's encrypt的教程生成证书后，会自动在nginx/sites-available目录下创建以域名为名称的配置文件，如下
    ```text
    server {

        server_name blog.zshnb.com;

        root /home/ubuntu/blog;
        index index.html;
        location ~* ^.+\.(jpg|jpeg|gif|png|ico|css|js|pdf|txt){
                root /home/ubuntu/blog;
        }

        listen [::]:443 ssl; # managed by Certbot
        listen 443 ssl; # managed by Certbot
        ssl_certificate /etc/letsencrypt/live/blog.zshnb.com/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/blog.zshnb.com/privkey.pem; # managed by Certbot
        include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

    }
    server {
        if ($host = blog.zshnb.com) {
            return 301 https://$host$request_uri;
        } # managed by Certbot


        listen 80;
        listen [::]:80;

        server_name blog.zshnb.com;
        return 404; # managed by Certbot
    }
    ```
    配置完毕后执行`sudo nginx -s reload`
3. 在服务器个人目录下拉取博客仓库，切换到gh-pages分支，然后便可以访问博客首页了，这里如果域名还未生效，可以通过访问ip的形式查看。
# 备案
因为众所周知的原因，部署在国内服务器的网站，想要合法公开，必须经过工信部备案，如果部署在海外的服务器，可以跳过这一步
1. 云服务厂商备案，这里我以腾讯云为例子，你也可以根据自己购买的云服务厂商说明来做。
首先登录腾讯云，进入[备案页面](https://console.cloud.tencent.com/beian)，点击新增服务，填写好页面要求的信息，这里注意几个点，
   1. 网站标题不要包含博客，个人姓名，容易过不了审核
   2. 在审核期间把服务器关了
腾讯云的审核大概1天就会通过，通过后会提交工信部管局审核，大约需要5个工作日，备案成功后会在腾讯云的备案里看到备案号，后面会用到
2. 公安局备案，登录[备案网站](https://beian.mps.gov.cn/#/)，按照提示填写信息提交即可，备案成功后同样会有备案号
3. 在nexT配置里添加备案信息，[参考文档](https://theme-next.js.org/docs/theme-settings/footer#Site-Beian-Information)

# 自动化
按照上面的步骤，在备案通过后，博客已经成功上线了，不过这里还存在一个问题，当我们发布新文章时，我们先要在本地写好文章，然后提交GitHub，然后本地执行`hexo g && hexo d`，
然后ssh登录服务器，更新gh-pages，这样才能同步发布最新文章到线上，那么我们能不能把这个过程自动化呢？也就是要做到当我们提交文章到GitHub后，有什么办法能自动部署hexo，且自动更新服务器上
的gh-pages分支。答案当然是有的，这里我们用GitHub action来完成这一自动化过程。
大概的思路如下
1. 把仓库clone下来
2. 安装nodejs，进入仓库目录`npm install`
3. 执行`hexo g && hexo d`
4. ssh服务器，更新gh-pages分支

这里我们需要准备一个新的ssh密钥对，用于action服务器提交GitHub和ssh服务器，一切准备完毕后，在根目录下新建./.github/workflows/deploy.yml，内容如下
```yaml
name: Deploy blog
on:
  push:
    branches:
      - main
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: checkout repo
        uses: actions/checkout@v2
      - name: prepare node env
        uses: actions/setup-node@v1
        with:
          node-version: "18.18.0"
      - uses: webfactory/ssh-agent@v0.7.0
        with:
          ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}
      - name: set github profile
        run: |
          git config --global user.name 'zshnb'
          git config --global user.email 'a857681664@gmail.com'
      - name: hexo generate
        run: |
          npm i -g hexo
          npm i
          hexo clean && hexo g && hexo d
      - name: ssh remove server
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          port: 22
          script: |
            cd blog
            git pull origin gh-pages --rebase
            sudo nginx -s reload
```
action里用到的secrets在仓库的settings -> secrets and variables -> actions里添加
![](img5.png)

# 总结
至此一个个人博客就搭建完毕了，过程虽然比较多，但每个步骤都不算难，只是中间需要等待备案，比较磨人