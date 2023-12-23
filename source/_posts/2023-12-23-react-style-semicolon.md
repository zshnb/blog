---
title: React样式陷阱：解决多余分号导致的背景样式问题
date: 2023-12-23 21:37:24
tags:
  - React
  - Javascript
---

在开发[人生进度表](https://lifetime.zshnb.com/)期间，我遇到了一个有趣且富有教育意义的前端开发问题。这个项目需要在一个格子下同时显示多个里程碑，比如使用不同的颜色来区分不同的内容。2个颜色呈现上下分布，而4个颜色则形成四宫格，如下图所示： ![四宫格样式示例](img.png)
<!--more-->
### 实现过程

我决定通过`background`属性的`linear-gradient`来实现这一需求。以下是实现逻辑的代码：

```
switch (props.backgroundColor.length) {
  case 1: {
    return {
      backgroundColor: props.backgroundColor[0]
    }
  }
  case 2: {
    const colors = props.backgroundColor.map(it => `${it} 50%`).join(',')
    return {
      background: `linear-gradient(to top, ${colors})`
    }
  }
  case 3: {
    const colors = props.backgroundColor
    return {
      background: `linear-gradient(to top, ${colors[0]} 33.33%, ${colors[1]} 33.33%, ${colors[1]} 66.66%, ${colors[2]} 66.66%)`
    }
  }
  case 4: {
    const colors1 = buildLinearGradient(props.backgroundColor[0], props.backgroundColor[1])
    const colors2 = buildLinearGradient(props.backgroundColor[2], props.backgroundColor[3])
    return {
      background: `linear-gradient(to right, ${colors1}), linear-gradient(to right, ${colors2});`,
      backgroundSize: '100% 50%',
      backgroundPosition: 'center top, center bottom',
      backgroundRepeat: 'no-repeat'
    }
  }
}
```

### **遇到的问题**

一开始，2个和3个颜色的情况都能正常显示，但在添加第4个颜色时，样式没有变成预期的四宫格。经过检查，我发现除了`background`外，其他属性都正确应用了。我最初怀疑是`colors1/2`变量生成有误，但经过验证，它们是正确的。

### **问题的发现与解决**

我花了大约一个小时调试，最终发现原来的`background`属性值末尾多了一个分号。我原本是直接从浏览器的开发者工具中复制的代码，不慎将分号也复制了进来。这个小小的分号阻止了背景颜色的正确渲染。

### **原因剖析**

在React中，样式通常以JavaScript对象的形式定义。在这种格式中，每个属性（如`backgroundColor`）对应一个值（如`'blue'`），且不需要像CSS那样以分号结尾。在我的代码中，`background`属性的值是一个模板字符串，其中不应包含分号。这个多余的分号使得整个属性值变成了无效的CSS语法，浏览器因此无法正确解析这个属性。

### **结论**

这个经历提醒我，在前端开发中，即使是最小的细节也可能导致意想不到的问题。