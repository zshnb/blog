---
title: MacOS 带passphrase的sshkey，Git命令不再输入密码
date: 2023-12-17 18:35:40
tags:
---

自从在Mac上换了一次sshkey，而且是带passphrase的，每次git操作都会提示要我输入密码，特别麻烦，于是在google上一阵搜索，找到一个解决方案。在.zshrc文件最开头输入

```bash
ssh-add --apple-use-keychain
```

这样只会在第一次命令时要求输入密码，之后都不需要输入密码了。