---
title: Next.js搭配Filepond实现上传文件操作
date: 2024-03-25 21:23:01
tags:
  - React
  - Next.js
---

最近自己在做一个产品，[CreateDeep - AI创作师](https://createdeep.com/)，产品需要实现上传文件的功能，找了一圈React生态中的上传组件，发现[Filepond](https://pqina.nl/filepond/docs/)无论是外观还是功能，都做的很不错。于是就决定用它了。
<!--more-->
Filepond本身是一个纯粹的Javascript库，不是专门为React做的，只不过它提供了一个React集成版，按照官网的教程，实现起来非常快

```
import React from 'react'
import { FilePond, registerPlugin } from 'react-filepond'
import 'filepond/dist/filepond.min.css'

const FileUploadButton = ({
  className,
  onFinishParse,
}: FileUploadButtonProps) => {
  return (
    <FilePond
      server={{
        process: '/api/upload',
        fetch: null,
        revert: null,
      }}
      allowFileMetadata={true}
      allowMultiple={true}
      maxParallelUploads={1}
      maxFiles={5}
      acceptedFileTypes={['application/pdf']}
      labelIdle="拖拽或点击上传文件"
      className={className}
      labelFileProcessingComplete="上传完毕"
      labelFileProcessing="上传中"
      onprocessfiles={() => {
        console.log('process files')
      }}
      onprocessfilestart={(file) => {
        console.log(`file ${file.filename} process start`)
      }}
      onprocessfile={(error, file) => {
        if (!error) {
          console.log(`file ${file.filename} process finished`)
        }
      }}
    />
  )
}

export default FileUploadButton


```
出来效果大概长这样。

![效果](img1.webp)
Filepond组件支持`acceptedFileTypes` ，一开始加上这个属性发现不生效，经过排查发现原来需要添加[File type validation](https://pqina.nl/filepond/docs/api/plugins/file-validate-type/)，代码如下

```
import FilePondPluginFileValidateType from 'filepond-plugin-file-validate-type'
// 然后在组件代码里加上
registerPlugin(FilePondPluginFileValidateType)

```
Filepond的server参数配置的是后端接口，默认请求参数是formData，第一个是一个json对象，第二个是binary数据。我在调试过程中发现无法从请求中获取到文件名，为此在Github仓库里开了一个[issue](https://github.com/pqina/react-filepond/issues/242)询问。同时找到了其他解决办法，Filepond支持自定义配置server，然后在server里自行完成请求的发送。

```
server={{
  process: (fieldName, file, _, load, error, progress, abort) => {
    const formData = new FormData()
    formData.append(fieldName, file, file.name)

    const request = new XMLHttpRequest()
    request.open('POST', '/api/upload')

    request.upload.onprogress = (e) => {
      progress(e.lengthComputable, e.loaded, e.total)
    }

    request.onload = function () {
      if (request.status >= 200 && request.status < 300) {
        const json = JSON.parse(request.response)
        const response = {
          ...json,
          filename: file.name,
        }
        onFinishParse(response)
        load(response)
      } else {
        error('parse pdf filed')
      }
    }
    request.send(formData)

    return {
      abort: () => {
        request.abort()
        abort()
      },
    }
  },
  // process: '/api/upload',
  fetch: null,
  revert: null,
}}

```
通过这种方式既可以拿到请求文件名，又可以拿到响应的内容。

Filepond默认会在组件尾部加上水印标识，官方表示因为是开源产品，所以希望大家能捐助。那么如果我们想要去掉它该怎么做呢？也很简单。首先在同级目录下新建`index.css` ，加入以下代码

```
.filepond--credits {
  display: none;
}

```
然后在组件开头`import ‘./index.css’` ，尾部的水印就被去掉了。
