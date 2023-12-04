---
title: 使用vaadin接入pocivoice的坑
date: 2023-11-06 12:04:54
tags:
  - Java
  - Vaadin
  - Picovoice
---

最近接到一个需求，客户想要用[vaadin](https://vaadin.com/)（一个用Java构建web的框架）构建一个页面，然后接入[picovoice](https://picovoice.ai/)，客户想要一个demo，能支持在网页上语音唤醒，音频识别，意图分析。
<!--more-->
客户给了我一篇[博客](https://vaadin.com/blog/always-listening-voice-commands-for-vaadin-web-applications)参考，于是按照博客里的步骤，1. 用vaadin创建一个空应用。2. 接入语音唤醒。很快代码就写完了，运行后点击按钮发现报错'pv_porcupine_init' failed with status INVALID_ARGUMENT。尝试了谷歌搜索后，并没有找到有价值的信息。又去官方Github仓库issue里搜索，也搜不到类似的内容。搜索无果后尝试点开porcupine.pv文件，发现文件开头有版本信息

![](img1.png)

同时发现博客里的porcupine-web依赖版本是2.1.16，猜测依赖和模型版本需要一致，于是把porcupine-web版本改成2.2.0后运行，成功

接着按照picovoice的[官方文档](https://picovoice.ai/docs/quick-start/rhino-web/)指示，下载rhino_params.pv模型文件，和自定义的context模型文件，运行后报了和上面一样的INVALID_ARGUMENT错误。这次有经验了，查看pv文件版本是2.2.0，context模型版本是3.0.0，2个模型文件的版本居然不一致。很显然context模型版本是无法改的，毕竟是从他们提供的网页下载的，于是只能从pv文件下手。在经过仔细寻找后，从他们的Github仓库中找到了一个叫v3.0的pull request，点进去后找到rhino_params.pv文件，下载查看，确实是3.0.0版本

![](img2.png)

于是再次运行，结果又是另一个错误。

Uncaught (in promise) RhinoRuntimeError: Initialization failed:

[0] License is expired.

[1] Failed to allocate, out of memory.

[2] Failed to load context with OUT_OF_MEMORY.

这个错误实在找不到解决方法，于是只好在他们的Github仓库里开了[issue](https://github.com/Picovoice/rhino/issues/631)询问，期间不断尝试，包括使用官方例子提供的模型运行也不行。折腾了好久，突发奇想通过隐私窗口运行居然成功了。

![](img3.png)

picovoice的web模型是使用wasm的技术，存在indexDB里的，猜测是最初运行了2.2.0版本的模型，导致数据已经存在indexDB里了，后面换成新模型后数据结构不兼容，才出现的这个错误。不过还是想说一下picovoice的speech-to-intent文档里模型版本不一致的问题，让新接入的用户很难排查。