---
title: Linux实用命令行工具
date: 2020-08-06 13:50
tags:
- Linux
---

# fzf
fzf是一个模糊搜索文件/文件夹的命令行工具，通过标准输入传入的内容，在交互式窗口输入搜索的关键字，即可高亮显示符合的文件名，用法如下
```shell
    find . -name "*.py" | fzf
```
<!--more-->
# shellcheck
shellcheck是一个用来检查sh文件语法的命令行工具，用法如下
```shell
    shellcheck example.sh
```

# tree
tree是一个以树型显示文件夹信息的命令行工具，用法如下
```shell
    tree dir
```

# tmux
tmux是终端会话复用工具

# tig
tig是一个在终端上可视化查看git各种状态，diff,文件的工具

# WindTerm
WindTerm是一个跨平台的终端工具

# jq
jq是一个命令行读写JSON工具

# riggrep
riggrep是一个性能更好的grep工具
