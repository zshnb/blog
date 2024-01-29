---
title: OpenAI - 新的嵌入式模型和API更新
date: 2024-01-29 15:56:48
tags:
  - OpenAI
---

OpenAI最近发了一篇[博客](https://openai.com/blog/new-embedding-models-and-api-updates)，主要介绍了他们最近发布的新模型，同时新模型还降低了API的价格，做到了加量的同时不加价，最后还给开发者们带来了官方的API密钥管理工具，很多开发者在此前都是在自己的应用里记录API的用量情况，现在也不需要了，直接登录OpenAI的后台就能看到。下面是博客的中文翻译版。



我们正在推出新的模型，降低 GPT-3.5 Turbo 的价格，并为开发者提供新的 API 密钥管理和 API 使用情况理解方法。新推出的模型包括：

- 两款新的嵌入式表示（Embedding）模型

- 更新后的 GPT-4 Turbo 预览模型

- 更新后的 GPT-3.5 Turbo 模型

- 更新后的文本审核模型

默认情况下，发送至 OpenAI API 的数据将不会用于训练或改善 OpenAI 模型。

## **新的嵌入式表示模型与更低的价格**
我们推出了两款新的嵌入式表示模型：体积更小、效率更高的 **text-embedding-3-small** 模型，以及体积更大、性能更强的 **text-embedding-3-large** 模型。

嵌入式表示是一组数字序列，它能代表自然语言或代码等内容中的概念。这种表示形式简化了机器学习模型和其他算法理解内容之间关系的难度，便于进行聚类或检索等任务。它们被广泛应用于 ChatGPT 和 Assistants API 的知识检索功能，以及许多检索增强生成（RAG）开发者工具中。

![](img1.webp)
**全新的小型文本嵌入模型**

**text-embedding-3-small** 是我们全新推出的高效嵌入模型，性能大幅优于其前代 **text-embedding-ada-002** 模型（于 2022 年 12 月发布）。

**性能更加强大。** 将 **text-embedding-ada-002** 与 **text-embedding-3-small** 进行比较，我们发现在多语言检索的常用基准测试 MIRACL 上，平均得分从 31.4% 提升至 44.0%，而在英语任务的常用基准测试 MTEB 上，平均得分从 61.0% 提升至 62.3%。

**价格大幅降低。** **text-embedding-3-small** 在效率上远超前一代模型 **text-embedding-ada-002**。因此，其价格相较于前代模型降低了 5 倍，从每千 Token 的 $0.0001 降至 $0.00002。

我们不会停用 **text-embedding-ada-002**，因此客户仍可选择继续使用旧版模型。

一个全新的大型文本嵌入模型：**text-embedding-3-large**

**text-embedding-3-large** 是我们推出的新一代大型嵌入模型，能够创建多达 3072 维的嵌入式表示。

**性能显著提升。** **text-embedding-3-large** 作为我们表现最佳的新模型，在 MIRACL 上的平均得分从 31.4% 提升至 54.9%，而在 MTEB 上的平均得分从 61.0% 提升至 64.6%。

|**评估基准**|**ada v2 版**|**text-embedding-3-small**|**text-embedding-3-large**|
|----|----|----|----|
|MIRACL 平均得分|31.4|44.0|54.9|
|MTEB 平均得分|61.0|62.3|64.6|

**text-embedding-3-large** 的定价为每千 Token $0.00013。

您可以在我们的 [嵌入式表示指南](https://platform.openai.com/docs/guides/embeddings) 中了解更多有关使用新嵌入模型的信息。

**对缩短嵌入式表示的原生支持**

使用较大的嵌入式表示（例如，在向量存储中用于检索），通常比小型表示消耗更多的计算资源、内存和存储空间。

我们的两款新嵌入模型均采用了一种技术，允许开发者在使用嵌入式表示时平衡性能和成本。具体来说，开发者可以通过调整 **dimensions** API 参数来缩短嵌入式表示的长度（即减少序列末端的数字），同时保持其表达概念的能力。例如，在 MTEB 基准测试上，缩短至 256 维的 **text-embedding-3-large** 嵌入式表示仍然能胜过 1536 维的未缩短 **text-embedding-ada-002** 嵌入。

|**评估基准**|**ada v2 版**|**text-embedding-3-small**|**text-embedding-3-small**|**text-embedding-3-large**|**text-embedding-3-large**|**text-embedding-3-large**|
|----|----|----|----|----|----|----|
|嵌入模型大小|1536|512|1536|256|1024|3072|
|MTEB 平均得分|61.0|61.6|62.3|62.0|64.1|64.6|

这种灵活性极高的使用方式，使得即便是在只支持最大 1024 维嵌入式表示的向量数据存储条件下，开发者也可以使用我们的最佳嵌入模型 **text-embedding-3-large**，通过将 **dimensions** 参数设置为 1024，从而将嵌入式表示从 3072 维缩短，以较小的向量大小换取一定的准确度。

## **其他新模型及更低价格**
**更新后的 GPT-3.5 Turbo 模型及降价**

下周，我们将推出新版 GPT-3.5 Turbo 模型 **gpt-3.5-turbo-0125**，这是过去一年内第三次降低 GPT-3.5 Turbo 的价格，旨在帮助客户扩大规模。新模型的输入价格降低了 50%，至每千 Token $0.0005，输出价格降低了 25%，至每千 Token $0.0015。这个模型还将有多项改进，包括更高的请求格式响应准确度，以及修复了之前影响非英语语言函数调用的文本编码问题的 [bug](https://community.openai.com/t/gpt-4-1106-preview-is-not-generating-utf-8/482839)。

使用固定 **gpt-3.5-turbo** 模型别名的客户将在该模型发布两周后自动从 **gpt-3.5-turbo-0613** 升级至 **gpt-3.5-turbo-0125**。

**更新后的 GPT-4 Turbo 预览版**

自 GPT-4 Turbo 发布以来，超过 70% 的 GPT-4 API 客户请求已转移到 GPT-4 Turbo，以便利用其更新的知识截止日期、更大的 128k 上下文窗口和更低价格。

今天，我们正在发布一个更新后的 GPT-4 Turbo 预览版模型 **gpt-4-0125-preview**。这个模型在执行任务（如代码生成）方面比之前的预览版更为彻底，目的是减少模型未能完成任务的“懒惰”情况。新模型还包含了针对非英语 UTF-8 生成的 bug 修复。

对于希望自动升级至新版 GPT-4 Turbo 预览版的用户，我们还推出了一个新的模型名称别名 **gpt-4-turbo-preview**，始终指向我们最新的 GPT-4 Turbo 预览版模型。

我们计划在未来几个月内推出带有视觉功能的 GPT-4 Turbo 的通用可用版本。

**更新后的审核模型**

免费的 Moderation API 使开发者能够识别潜在的有害文本。作为我们持续安全工作的一部分，我们正在发布 **text-moderation-007**，这是我们迄今为止最健全的审核模型。**text-moderation-latest** 和 **text-moderation-stable** 的别名已更新，指向该模型。您可以通过我们的 [安全最佳实践指南](https://platform.openai.com/docs/guides/safety-best-practices) 了解更多关于构建安全 AI 系统的信息。

## **了解 API 使用情况和管理 API 密钥的新方法**
我们正在推出两项平台改进，以便开发者更好地了解他们的 API 使用情况并更有效地管理 API 密钥。

首先，开发者现在可以在 [API 密钥页面](https://platform.openai.com/api-keys) 上为 API 密钥分配权限。例如，可以将密钥设置为仅读取权限，用于支持内部跟踪仪表板，或限制其仅访问特定端点。

其次，通过 [开启跟踪](https://platform.openai.com/api-keys) 后，使用情况仪表板和导出功能现在可以展示基于 API 密钥的使用情况。这使得通过为每个功能、团队、产品或项目分配不同的 API 密钥，轻松查看各个层面的使用情况成为可能。

![](img2.webp)
在未来几个月，我们计划进一步改善开发者对 API 使用情况的查看和 API 密钥的管理能力，特别是在大型组织中。


