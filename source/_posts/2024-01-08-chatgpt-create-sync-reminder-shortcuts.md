---
title: 使用ChatGPT一步步实现iOS提醒事项与Notion双向同步
date: 2024-01-08 11:53:42
tags:
  - 快捷指令
  - ChatGPT
---
最近因为很多事情需要提醒，因为我是苹果手机，想到最便捷的方法就是提醒事项。但是要创建的提醒事项实在是太多了，手机上一个个操作又太费劲。我又是Notion的重度用户，于是就有了一个想法，能不能在Notion的database管理所有提醒事项，并自动同步到iOS的提醒事项；其次iOS创建的提醒事项也会自动同步到Notion的database，完成双向同步。因为我自己不太熟悉快捷指令，正好可以尝试一次用ChatGPT辅助完成这次的工作。以下是完整演示
<!--more-->

| ![提醒事项到Notion](img1.gif) | ![Notion同步提醒事项](img2.gif) |
|--------------------------|---------------------------|



### 开始

首先我把最初的想法告诉了ChatGPT：

> 我需要一个快捷指令，具体功能是可以双向同步Notion的database和iOS的提醒事项

显然iOS并没有直接的方法能完成双向同步，ChatGPT回复：

> 不幸的是，直接在iOS快捷指令中实现这样的功能是非常有限的，因为iOS快捷指令本身并不支持直接与Notion API交互，也不支持复杂的数据处理和同步逻辑。要实现这样的功能，你可能需要使用更高级的工具，比如编写一个自定义的应用或脚本，这可能需要编程知识。

### 实现单向创建

这时候我们就需要拆分需求了，我们可以先退一步，完成单向同步，也就是先从iOS提醒事项同步到Notion database，于是问ChatGPT：

> 不需要双向同步，需要实现点击快捷指令后弹出输入框，把输入的内容同时创建提醒事项和同步到notion的database即可

这时候ChatGPT就给出了可行的步骤：

> **获取Notion API密钥和数据库ID**
>
> - 在Notion中，创建一个集成并获取API密钥。- 找到你想同步的数据库，并获取其ID。### **创建快捷指令：**1. **打开iOS的“快捷指令”应用。**1. **创建一个新的快捷指令**：
> - 点击右上角的“+”按钮开始创建。1. **添加输入动作**：
> - 使用“询问输入”动作来弹出输入框。
> - 可以自定义输入框的提示文字，例如“请输入提醒内容”。1. **添加提醒事项动作**：
> - 使用“添加新提醒”动作。
> - 将“询问输入”的结果作为提醒内容。1. **设置Notion API请求**：
> - 使用“获取URL的内容”动作来发送POST请求到Notion API。
> - 设置URL为Notion API的地址（通常是**https://api.notion.com/v1/pages**）。
> - 在请求体中，按照Notion API的要求格式化输入内容。
> - 添加Headers，包括你的Notion API密钥和Content-Type。

我们先准备一个database，添加几个提醒事项最基本的属性

![notion database](img9.webp)
ChatGPT给了初步的步骤，里面最重要的部分就是Notion API，我们假设现在对Notion API一无所知，于是询问ChatGPT：

> 如何调用notion api，往database里新增一条记录？请搜索notion api文档(https://developers.notion.com/docs/working-with-databases)，给出具体步骤

ChatGPT回答：

> **确保你的Notion集成有足够的权限**：你的Notion集成必须具有更新内容的能力（update content capabilities）。你可以在Notion的"My integrations"仪表板中检查和更新这些设置。
>
> 1. **了解Notion数据库的结构**：了解你的Notion数据库的属性和它们的类型（如**title**, **rich_text**, **number**等），因为你需要在API请求中使用这些属性。1. **构建API请求**：使用POST请求至**https://api.notion.com/v1/pages**来创建一个新页面。请求的主体（body）应包含：
>
> - **parent**对象，其中包含数据库的ID。
> - **properties**对象，其中包含根据你的数据库属性配置的键值对。1. **发送请求**：使用你选择的编程语言或工具发送POST请求。请求头（Headers）中需要包含你的Notion API密钥和适当的内容类型（Content-Type）。

我们只需要配置好获取URL内容，剩下的就是根据Notion API文档，查阅具体properties结构调试即可。

![get url content](img3.webp)

### 实现反向同步

接下来让我们完成下一个，也就是从Notion database同步iOS提醒事项，老样子先问ChatGPT：

> 现在还需要实现另一个功能，需要能同步Notion的database里的内容到提醒事项里的快捷指令

回答：

> **准备工作：**
>
> 1. **获取Notion API密钥和数据库ID**：
>
> - 在Notion中创建一个集成，并获取API密钥。
> - 找到你想要同步的数据库，并获取其ID。**创建快捷指令：**
>
> 1. **打开iOS的“快捷指令”应用。**1. **创建一个新的快捷指令**：
>
> - 点击右上角的“+”按钮开始创建。1. **获取Notion数据库内容**：
> - 使用“获取URL的内容”动作来发起一个GET请求到Notion的API端点（通常是**https://api.notion.com/v1/databases/{database_id}/query**，将**{database_id}**替换为你的数据库ID）。
> - 在请求头（Headers）中添加你的Notion API密钥和适当的内容类型。
> - 解析返回的JSON以获取数据库中的记录。1. **遍历数据库记录**：
> - 使用“重复”或“循环”动作遍历解析后的数据库记录。
> - 提取你需要的信息，比如事项的标题、日期等。1. **为每条记录添加提醒事项**：
> - 在循环内部，使用“添加新提醒”动作。
> - 将提取的信息设置为提醒事项的内容。1. **测试快捷指令**：
> - 运行快捷指令并检查iOS提醒事项中是否正确添加了Notion数据库中的内容。

按照ChatGPT的答案我们先有一个大致思路

1. 调用Notion API获取database下所有pages

2. 遍历pages，对于每一条page记录，判断提醒事项有无相同事项列表和相同事项标题的记录

3. 如果没有，新建这条提醒事项

![create reminder](img4.webp)
这里有一个点要注意，快捷指令中json数组的索引和主流编程语言不一样，首先序号是从1开始，其次索引语法是`array.[index]`，而不是`array[index]` ，一开始获取title我写成了`properties.名称.title[0].text.content`一直报错，查了文档才知道。

运行后发现不对，提醒事项是添加了，但没有添加到对应列表里，因为提醒事项里并没有对应列表，也不会自动添加不存在的列表，于是需要我们额外做一步：检测page记录的事项列表属性，如果不存在于现有列表，则添加一个新列表

![create list reminder](img5.webp)
运行后会报错`com.apple.reminderkit error -1` 搜索了一下也没找到原因，猜测可能是快捷指令提醒事项的bug，无法用不存在的列表选项作为过滤条件。于是只能换一个思路，先获取全部提醒事项，然后获取全部列表，再判断列表中有没有page记录的事项列表属性

![create list reminder](img6.webp)
这种方式还是无法正常工作，因为List是列表类型，判断方式是全相等，而page记录的事项列表属性是字符串，肯定不会相等，所以我们需要把List转换成字符串，然后再判断是否包含，像这样

![create list reminder](img7.webp)
把添加列表步骤放在添加提醒事项前面，这样整个快捷指令就正常工作了。

剩下还有最后一个小细节：当Notion database里没有设置记录的提醒日期时，同步会报错提示没有对应日期。所以我们需要在添加提醒事项前增加一个小步骤，设置好提醒日期。当page记录不存在提醒日期时，设置提醒日期为明天，否则为page记录的提醒日期。这里同样可以问ChatGPT：

> 如何设置一个日期是明天的变量？

回答

> 1. **添加“当前日期”动作**：
>
> - 从动作列表中搜索并添加“当前日期”动作。这个动作会获取当前的日期和时间。1. **添加“调整日期”动作**：
> - 接着添加一个“调整日期”动作。这个动作可以用来修改日期和时间。
> - 将“调整日期”动作连接到“当前日期”动作的输出。1. **设置“调整日期”动作以获取明天的日期**：
> - 在“调整日期”动作的设置中，选择“添加”选项，并且设置时间增加的量。例如，设置为“1 天”来获取明天的日期。1. **（可选）格式化日期**：
> - 如果你需要特定的日期格式，可以添加一个“格式化日期”动作。
> - 在“格式化日期”动作中，选择你想要的日期格式。1. **将结果存储到变量**：
> - 使用“设置变量”动作来存储修改后的日期。
> - 给变量命名，例如“明天的日期”。1. **使用变量**：
> - 现在，你可以在快捷指令的后续步骤中使用这个变量。

![create reminder](img8.webp)
测试一下，大功告成。现在就剩最后一步自动化，我们期望每次打开提醒事项时，自动同步Notion database。于是我们需要在快捷指令的自动化中设置，当打开提醒事项APP时立即运行同步快捷指令，这样整个流程就都走通了。

### 总结

使用ChatGPT的这一套流程下来还是挺顺利的，从快捷指令各个步骤的选择，到Notion API的接入，ChatGPT都能给出正确的回答，只是在一些比较复杂的细节上还是无法一次性到位，需要更进一步询问ChatGPT修正错误，或者更换解决问题的思路。

最后分享一下这2个快捷指令

- [创建Notion提醒事项](https://www.icloud.com/shortcuts/6cb6dd4c684a477c8d1a5a19bba54866)

- [同步Notion提醒事项](https://www.icloud.com/shortcuts/73711bab1d07454a9c7bf4c6e8b95fc0)

本文由[NotionConverter](https://notionconverter.com)复制