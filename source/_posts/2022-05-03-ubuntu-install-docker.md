---
title: Ubuntu安装Docker教程
date: 2022-05-03 21:10
tags:
- Docker
- Linux
---

# 背景
最近经常需要在云服务器上安装Docker，在安装过程中出现不少问题，遂记录下来供自己以及有需要的朋友查阅。
本教程适用于Ubuntu 18.04 Ubuntu 20.04 Ubuntu 22.04
<!--more-->
# 安装
1. 安装新版之前卸载旧版本
    ```shell
    sudo apt-get remove docker docker-engine docker.io containerd runc
    sudo rm -rf /var/lib/docker/ # 如果不打算留旧版本的镜像，容器等数据，可以删掉这个文件夹
    ```
2. 安装apt证书，更新源仓库，添加docker软件源
    ```shell
    sudo apt-get update

    sudo apt-get install \
        ca-certificates \
        curl \
        gnupg \
        lsb-release
        
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```
3. 安装docker
    ```shell
    sudo apt-get update
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
    ```
    此时docker已经装好，可以输入`sudo docker version`验证安装结果，若打印出版本说明安装成功，但是目前所有docker命令均需要root权限，也就是需要sudo，为了去掉docker命令的sudo，还需要接下来的操作
4. 去除sudo
    ```shell
    # 建立用户组（如果已经有可以忽略）
    sudo groupadd docker
    # 将当前用户加入docker用户组
    sudo usermod -aG docker $USER
    # 生效用户组改动
    newgrp docker
    # 重启docker
    sudo systemctl restart docker
    ```
5. 镜像加速

   在/etc/docker下新建daemon.json（有就不需要创建了）, 然后在文件里输入`{"registry-mirrors": ["https://registry.cn-hangzhou.aliyuncs.com", "https://registry.docker-cn.com"]}`，重启docker服务即可
