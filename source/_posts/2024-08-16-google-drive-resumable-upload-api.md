---
title: Google Drive resumable upload api的坑
date: 2024-08-16 16:36:59
tags:
  - Javascript
  - Google drive
---

最近在开发一个ffmpeg生成短视频的程序，程序里需要对接Google drive，当程序生成视频成功后，需要把视频上传到drive。在Google的文档里上传总共有3个API，分别是

- single upload

- multiple upload

- resumable upload

<!--more-->
前面2个upload最大支持5MB的文件，而我这个程序生成的视频最小都有几十MB，所以只能选择最后一种upload。

按照[官方文档](https://developers.google.com/drive/api/guides/manage-uploads#resumable)的指南，resumable upload分为2个步骤：

1. 初始化upload，通过一个POST接口获取upload session url

1. 获取上一步的session url，使用PUT调用url，同时把文件blob传入

那么这里就遇到一个问题，文件名和文件夹信息从哪里传入？文档上压根没提到这回事。经过一番搜索，我发现了一个GitHub开源工具，包装好了upload API。于是立马看看人家的源码是怎么处理这个问题的。

功夫不负有心人，在[uploadUrl文件](https://github.com/overlookmotel/google-drive-uploader/blob/master/lib/uploadUrl.js#L49)里找到了需要的参数，测试一看又报错了，404 file not found。我又仔仔细细对了一遍folder id，发现没错啊。经过一番思索猜测可能还是少了一些关键参数，因为我要上传的folder是一个shared drive，而drive API里在list API部分专门有提到几个参数是关于shared，分别是`supportsAllDrives`，`corpora`，`includeItemsFromAllDrives`，经过各种组合尝试，最终发现需要在初始化upload的接口里加入`supportsAllDrives=true&corpora=drive`，最后测试终于上传成功了。


