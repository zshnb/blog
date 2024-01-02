---
title: Notion converter重构 - 解析与渲染
date: 2024-01-02 10:52:25
tags:
  - 产品
---

最近[Notion Converter](https://notionconverter.com/)完成了一次非常大的更新，彻底重构了底层的解析和渲染逻辑，这篇文章想和大家分享一下重构历程。同时我也把解析器开源在了[Github](https://github.com/zshnb/notion-dom-parser)。
<!--more-->
先大概介绍一下老版本的逻辑吧。因为我们一开始就不打算让用户走Notion集成的方式使用插件，会很麻烦，所以解析都是通过jQuery读取Notion页面的dom结构，然后根据class执行不同的渲染逻辑。代码结构大概是这样

```
for (let i = 0; i < $notionPageChildrenNodes.length; i++) {
  let $this = $($notionPageChildrenNodes[i]);
  const className = $this.attr("class");
  if (className.indexOf("notion-header-block") !== -1) {
    $wechatOutput.append(this.wechatRender.renderHeader($this));
  } else if (className.indexOf("notion-text-block") !== -1) {
		$wechatOutput.append(this.wechatRender.renderText($this));
	}
}

```
这种方式在大部分正常场景都工作的很好，但在一些复杂场景下，就无法解析了。

Notion的页面结构是非常自由的，你可以在段落里缩进段落，列表里写引用块，代码块，并列多个表格等等。这种无限嵌套的结构在老版本里是无法解析的，即使我在里面加了一些解析缩进块的逻辑，也做不到解析无限嵌套，同时这个逻辑也影响了复制为markdown的功能，于是就萌生了重构的念头。在老版本里为什么需要通过jQuery来解析渲染？因为需要获取到dom节点的内容。有没有不通过jQuery就能获取到dom节点内容的方法？有，浏览器原生的[DOMParser](https://developer.mozilla.org/en-US/docs/Web/API/DOMParser)。它接收html字符串，然后解析成dom树的结构返回，当然原生的DOMParser不是很友好，我选择了开源的[html-dom-parser](https://github.com/remarkablemark/html-dom-parser)，专注于解析html，并且优化了返回的数据结构。于是重构思路呼之欲出，即

1. 先通过html-dom-parser把Notion文章解析成中间态的数据结构
   ```
   export type DomNode = {
       type: string // h1, p, text...
       children: DomNode[]
       text: string // 如果type=text时的文本
   }

	```

1. 创建微信公众号和markdown渲染器，都通过读取解析的结果进行渲染，只不过一个渲染html，另一个渲染markdown

计算机领域有一句经典语录，程序=算法+数据结构，这么一看上面的思路完美符合这句语录

贴一下重构后的伪代码

```
function doParse(element) {
	const classes = element.attribs.class
    if (classes.indexOf('notion-header-block') !== -1) {
      const children = []
      this.parseHeader(element, children)
      return {
        type: 'h1',
        children
      }
    }
}

parseHeader(element, children) {
    for (const c of element.children) {
      if (c.type === 'text') {
        children.push(this.parseText(c))
      } else {
        if (c.attribs.hasOwnProperty('class')) {
          const classes = c.attribs.class
          if (classes.split(' ').some(it => this.notionBlockClasses.indexOf(it) !== -1)) {
            children.push(this.doParse(c))
          } else {
            this.parseHeader(c, children)
          }
        } else {
          this.parseHeader(c, children)
        }
      }
    }
  },

```
这里重点讲一下递归的思路。Notion里每一个块嵌套的dom层级都很深，加上无法通过jQuery直接查找层级获取数据，所以递归的思路就是遍历当前节点的子节点，如果是文本就解析。否则判断子节点是否有class，如果有，判断class是否在目标解析class集合里，这一步就是解析嵌套块的关键，通过这一步就实现了像标题嵌套链接，列表嵌套代码块的解析，以及无限层级缩进的解析。

![dom结构](img1.webp)
解析中最麻烦的是列表的解析，Notion的列表块不是用的原生ul和ol实现的，而且用的复杂的div布局，所以Notion的列表块序号可以做到非常自由，可以随意从任意数字开始，但是公众号里用的是原生的列表标签，所以我在解析完Notion的列表块后，必须把相邻列表块进行合并，转换成标准列表结构。同时还要记录列表项的层级，在渲染的时候设置对应层级的list-type

![Notion列表结构](img3.webp)
最后解析出来的结果如下

![解析结果](img2.webp)

