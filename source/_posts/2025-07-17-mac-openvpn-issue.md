---
title: Mac的OpenVPN连接错误Error calling protect() method on socket
date: 2025-07-17 16:19:44
tags:
 -- OpenVPN
---

个人用的Mac M2Max电脑，最近连接OpenVPN一直报错Error calling protect() method on socket，网上搜了一下解决方案，只需要执行以下3条命令即可

```
sudo su

launchctl load -w /Library/LaunchDaemons/org.openvpn.client.plist

untill reboot // 这条执行报错也无所谓

```
最后重启一下OpenVPN即可
