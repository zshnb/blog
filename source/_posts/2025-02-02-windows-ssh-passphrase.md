---
title: Windows配置terminal记住ssh pass phrase
date: 2025-02-02 15:17:33
tags:
  - Windows
  - ssh
---

最近需要临时用Windows笔记本开发，在配置完Windows terminal后，发现一个问题，我的ssh key是带密码的，每次执行git操作都需要手动输入密码。Mac上可以很方便地通过Apple key chain保存，但Windows没有这么方便的工具，经过一番搜索，终于找到了替代方法。
<!--more-->
1. Win+r后输入services.msc，找到OpenSSH选项，设置运行和自动启动
   ![](img1.webp)

2. 在terminal里输入并执行`git config --global core.sshCommand C:/Windows/System32/OpenSSH/ssh.exe`

3. 在ssh目录的config配置文件里输入以下内容（通常位于C盘用户目录下.ssh文件夹
   ```
   Host *
   AddKeysToAgent yes
   IdentitiesOnly yes
   ```

4. 在terminal里输入并执行`ssh-add $HOME/.ssh/your_file_name` ，然后输入ssh key的pass phrase

最后关闭terminal就大功告成，之后打开terminal执行任何git命令不再需要输入pass phrase
