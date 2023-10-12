---
title: layui文件上传进度条踩踩坑记
date: 2021-04-05 10:17
tags:
- Layui
- Javascript
---
# 背景
最近手里一个项目用到了layui，遇到了一个需求，上传文件的时候需要显示进度条，但是layui的上传模块没有提供进度条回调
<!--more-->
# 搜索
于是谷歌了一圈，发现一个大神魔改了layui的upload.js，加上了进度条回调功能 [访问原文](https://www.35youth.cn/644.html)，可惜原文的代码是不能直接复制使用的，于是我又找了一篇
[踩坑帖子](https://www.liangzl.com/get-article-detail-162868.html)，这篇文章修复了很多错误，可以使用魔改的upload.js上传文件了
(之前文章的写法连上传都会报错)，可惜最重要的进度条显示还是没有修复，于是我又在魔改的upload.js基础上进行了修复。这里我放上经我本人修复并测试的最终版本。(layui版本为2.5.5)，代码如下：
```html
<html>
<body>
<div class="layui-form-item">
    <label class="layui-form-label">上传信息</label>
    <div class="layui-upload">
        <button type="button" class="layui-btn layui-btn-normal" id="upload-people-btn">选择文件</button>
        <div class="layui-upload-list">
            <table class="layui-table">
                <thead>
                <tr>
                    <th>文件名</th>
                    <th>大小</th>
                    <th>上传进度</th>
                    <th>状态</th>
                    <th>操作</th>
                </tr>
                </thead>
                <tbody id="demoListPeople"></tbody>
            </table>
        </div>
        <button type="button" class="layui-btn" id="fileListActionPeople">开始上传</button>
    </div>
</div>
</body>
</html>
<script th:src="@{/lib/layui-v2.5.5/layui.js}" charset="utf-8"></script>
//这里要引入魔改后的js文件，下载地址在下面，同时注意layui.use中删掉引入的upload，引入的upload.js要在layui.js下面
<script th:src="@{/lib/layui-v2.5.5/lay/modules/upload.js}" charset="utf-8"></script>
```
```javascript
        let element = layui.element
        let demoListPeople = $('#demoListPeople')
        upload.render({
            elem: '#upload-people-btn',
            size: 102400, //限制文件大小，单位 KB
            url: '/import-people',
            accept: 'file',
            multiple: false,
            auto: false,
            bindAction: '#fileListActionPeople',
            // 第二篇文章xhr重复，而且progress回调也是错的
            xhr: function (index, e) {
                // e: xhr上传请求回调，返回loaded和
                $('#demoListPeople').find('.layui-progress ').each(function () {
                    let progressBarName = $(this).attr('lay-filter')
                    let percent = Math.floor((e.loaded / e.total) * 100)//计算百分比
                    element.progress(progressBarName, percent + '%')//设置页面进度条
                })
            },
            choose: function (obj) {
                let files = this.files = obj.pushFile() //将每次选择的文件追加到文件队列
                //读取本地文件
                obj.preview(function (index, file, result) {
                    let tr = $(`
                        <tr id="upload-${index}">
                            <td>${file.name}</td>
                            <td>${(file.size / 1024).toFixed(1)}kb</td>
                            <td>
                                <div class="layui-progress layui-progress-big" lay-filter="progress_${index}" lay-showPercent="true">
                                    <div class="layui-progress-bar" lay-percent="0%"></div>
                                </div>
                            </td>
                            <td>等待上传</td>
                            <td>
                                <div>
                                    <button class="layui-btn layui-btn-xs demo-reload-people layui-hide">重传</button>
                                    <button class="layui-btn layui-btn-xs layui-btn-danger demo-delete-people">删除</button>
                                </div>
                            </td>
                        </tr>
                    `)

                    //单个重传
                    tr.find('.demo-reload-people').on('click', function () {
                        obj.upload(index, file)
                    })

                    //删除
                    tr.find('.demo-delete-people').on('click', function () {
                        delete files[index] //删除对应的文件
                        tr.remove()
                    })
                    demoListPeople.append(tr)
                })
            },
            done: function (res, index, upload) {
                let tr = demoListPeople.find(`#upload-${index}`),
                    tds = tr.children()
                tds.eq(3).html('<span style="color: #5FB878;">上传成功</span>')
                tds.eq(4).html('') //清空操作
                layer.msg('上传成功')
            },
            error: function (index, upload) {
                let tr = demoListPeople.find('tr#upload-' + index)
                    , tds = tr.children()
                tds.eq(2).html('<span style="color: #FF5722;">上传失败</span>')
                tds.eq(3).find('.demo-reload').removeClass('layui-hide') //显示重传
            }
        })
```
最后出来结果如下：
![](img1.png)

最后附上魔改后的js: <a href="/2021-04-05-layui-upload-process/upload.js" target="_blank">附件1</a>