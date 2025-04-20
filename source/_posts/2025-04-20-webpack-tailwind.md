---
title: 使用webpack配置tailwindcss和postcss
date: 2025-04-20 21:58:47
tags:
  - TailwindCSS
  - Webpack
---

最近在写一个谷歌浏览器插件，使用这个[脚手架](https://github.com/lxieyang/chrome-extension-boilerplate-react?tab=readme-ov-file)作为基础，在基础之上打算配置使用tailwindcss，由于脚手架用的是webpack，于是需要用postcss作为loader，于是安装了tailwindcss，postcss，postcss-loader，autoprefix，然后在webpack里配置使用postcss，启动后报错
<!--more-->
> It looks like you're trying to use `tailwindcss` directly as a PostCSS plugin. The PostCSS plugin has moved to a separate package, so to continue using Tailwind CSS with PostCSS you'll need to install `@tailwindcss/postcss` and update your PostCSS configuration
>

错误信息还是比较清晰的，网上搜索了一下，解决方法是把postcss.config.js文件里的改成：

```
module.exports = {
  plugins: {
    "@tailwindcss/postcss": {},
    autoprefixer: {},
  },
}

```
同时把postcss依赖删除就好了。
