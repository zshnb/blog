---
title: Git系列（2）分支策略
date: 2022-09-05 21:10
tags:
- Git
---

# 分支约定

- Git提供了创建分支的功能，但是没有详细说明如何使用
- 需要有一个基于团队协作的最佳实践去避免错误和混淆
- 帮助新成员快速理解协作流程

下面来具体说明。
<!--more-->
## 基于主线分支开发，短时分支策略

- 只有极少数的分支
- 分支关联很小的提交
- 高质量的测试和QA
- 分支用后即删

## 基于阶段分支开发，长期存活分支策略

- 存在不同类型的分支
- 每个类型的分支有各自的用途
- 存在长期分支，比如master，release
- 开发分支往往不会直接在长期分支上直接提交，而是通过merge/rebase的方式

## GitHub Flow

- 只存在一个长期存活分支（master）
- 对于新代码，基于master分支创建新分支，用后即删

## Git Flow

- 存在master和develop2个长期存活分支
- 对于新代码，基于master分支创建新分支，开发结束先合并develop分支，再把develop分支合并进master分支
