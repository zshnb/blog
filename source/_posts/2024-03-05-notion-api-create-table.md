---
title: Notion API踩坑之 - 创建Table
date: 2024-03-05 12:33:59
tags:
  - Notion
---

最近在做一个小工具 - 同步iOS备忘录到Notion。在做的过程中遇到一个场景是，需要把iOS备忘录的table同步到Notion的table，Notion API有提供table block的结构，但是写的非常不详细，根本没有给出一个可行的在page中创建table的demo，于是我只好通过各种尝试，最终成功通过API在page中创建了table。
<!--more-->
首先官方文档给出了table和table_row的结构

```
{
  "type": "table",
  "table": {
    "table_width": 2,
    "has_column_header": false,
    "has_row_header": false
  }
}
{
  "type": "table_row",
  "table_row": {
    "cells": [
      [
        {
          "type": "text",
          "text": {
            "content": "column 1 content",
            "link": null
          },
          "annotations": {
            "bold": false,
            "italic": false,
            "strikethrough": false,
            "underline": false,
            "code": false,
            "color": "default"
          },
          "plain_text": "column 1 content",
          "href": null
        }
      ]
    ]
  }
}

```
但是并没有给出table_row和table的关联关系，于是我先调用API尝试只添加table block，得到如下错误：`APIResponseError: body failed validation: body.children[3].table.children should be defined, instead was undefined.`

很明显table块里需要有children，并且大概率是table_row的容器。于是尝试把table_row加入，运行，成功了。
