---
title: ubuntu一步到位安装并连接MySQL
date: 2024-01-19 12:22:51
tags:
  - Linux
  - MySQL
---

## **第 1 步 - 更新系统**
确保我们的系统是最新的，要更新系统，请运行以下 [apt 命令](https://www.cyberciti.biz/faq/ubuntu-lts-debian-linux-apt-command-examples/)：

```
sudo apt update
sudo apt list --upgradable
sudo apt upgrade
```
<!--more-->
![更新系统](img1.webp)
![更新系统](img2.webp)
## **第 2 步 - 在 Ubuntu 22.04 LTS 上寻找 MySQL 8 服务器软件包**
我们可以使用 apt-cache 命令或 [apt 命令](https://www.cyberciti.biz/faq/ubuntu-lts-debian-linux-apt-command-examples/) 来查找 Ubuntu 22.04 LTS 上的 MySQL 服务器和客户端软件包。例如：apt-cache search mysql-server
系统会显示一系列可用选项，其中包括 Oracle MySQL 8.xx 和 MariaDB 10.x 的服务器和客户端软件包。

![搜索mysql包](img3.webp)
> 🔑
> mysql-server-8.0 与 mysql-server-core-8.0 软件包对比：
> - **mysql-server-8.0** – 这个包几乎在所有情况下都是必需的。它包含了 MySQL 数据库服务器的二进制文件、客户端和系统数据库的设置。- **mysql-server-core-8.0** – 这个包包含服务器的二进制文件，但不包括设置系统数据库所需的全部基础设施。因此，这个包更适合那些要设置 Linux 容器（如 Docker、LXD 等）且不需要所有额外组件（例如 mysql 客户端）的用户。



## **第 3 步 - 安装 MySQL 8 服务器软件包**
我们将在 Ubuntu 22.04 LTS 上安装 MySQL 服务器版本 8.0.28：
sudo apt install mysql-server-8.0

![安装mysql-server-8.0](img4.webp)
### **为根账户设置密码**
首先，设置根账户的密码，运行sudo mysql ，然后按照以下语法设置密码：

```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'My7Pass@Word_9_8A_zE';
exit

```
> 🔑
> **MySQL 8.xx 的关键配置文件和端口**
> - mysql.service，这是服务的名称。您可以使用以下 systemctl 命令来管理它
    > 	```
> 	sudo systemctl start mysql.service
> sudo systemctl stop mysql.service
> sudo systemctl restart mysql.service
> sudo systemctl status mysql.service
>
> 	```- /etc/mysql/ - MySQL 服务器的主要配置目录。- /etc/mysql/my.cnf - MySQL 数据库服务器的配置文件。编辑 .my.cnf ($HOME/.my.cnf) 文件来设置用户特定的选项。以下两个目录中的设置可以覆盖它：
> /etc/mysql/conf.d//etc/mysql/mysql.conf.d/- TCP/3306 端口 - TCP/3306 是 MySQL 服务器的默认网络端口，出于安全考虑，它绑定在 127.0.0.1 上，可以更改这个设置，之后就可以通过在 /run/mysqld/ 目录下设置的 localhost 套接字来访问 MySQL 服务器。

## **第 4 步 - 加强 MySQL 8 服务器的安全性**
默认情况下，服务器没有密码，且其他设置也需要调整。让我们运行以下命令来进行设置并加强服务器的安全性：sudo mysql_secure_installation

![设置mysql安全性](img5.webp)
## **第 5 步 - 设置 MySQL 服务器开机自启**
确保 MySQL 服务器 8 在系统启动时能自动启动，可以使用 systemctl 命令来实现：
sudo systemctl is-enabled mysql.service
如果尚未启用，使用以下命令来启用服务器：
sudo systemctl enable mysql.service
在 Ubuntu Linux 20.04 LTS 上，通过以下 systemctl 命令来检查 MySQL 8 服务器的状态：
sudo systemctl status mysql.service

![检查mysql服务状态](img6.webp)
## **第 6 步 - 启动/停止/重启 MySQL 服务器**
我们可以通过命令行在 Ubuntu 22.04 LTS 上控制 MySQL 服务器。如果服务器尚未运行，让我们先启动它：sudo systemctl start mysql.service

要停止 MySQL 服务器，请输入：sudo systemctl stop mysql.service

按照下面的方式来重启 MySQL 服务器：sudo systemctl restart mysql.service

我们还可以使用 journalctl 命令来查看 MySQL 服务的日志记录，方法如下：
sudo journalctl -u mysql.service -xe

## **第 7 步 - 登录 MySQL 8 服务器进行测试**
到目前为止，我们已经学习了如何在 Ubuntu 22.04 LTS 上安装、配置、加固安全性以及启动/停止 MySQL 服务器版本 8。接下来，是时候以 root（MySQL 管理员）用户身份登录了。登录语法如下：mysql -hlocalhost -uroot -p ，紧接着命令行会提示输入密码，回车即可进入。

![登录mysql](img7.webp)
## **第 8 步 - 配置 MySQL 8 服务器**
使用文本编辑器编辑 /etc/mysql/mysql.conf.d/mysqld.cnf 文件，设置字符集编码，慢sql，binlog等。

![配置mysql](img8.webp)
## 第9步 - 通过DataGrip连接MySQL
相信很多同学在服务器上安装完MySQL后，就直接开放了服务器的3306端口，为了让数据库管理工具可以直连。但是这个做法很危险，现在互联网上存在着大量机器人，无时无刻在扫大家服务器的3306端口，再恰巧你的数据库账户密码都很简单（root/admin），那么有一天当你登录数据库后就会发现被人清空了，惊不惊喜，所以这里我教大家一种更安全的连接数据库的方式。

### 安装DataGrip
去[Jetbrains官网下载](https://www.jetbrains.com/datagrip/)安装这个软件，如果能力允许可以支持一下正版

### 配置ssh-key
添加Data Source，选择SSH方式连接

![添加Data Source](img9.webp)
![添加SSH配置](img10.webp)
### 添加MySQL连接
返回Data Source页面，这里注意Host要填localhost，而不是服务器的ip地址，因为我们是通过SSH连接的，相当于在服务器本地用命令行连接数据库。最后输入用户名密码，测试连接是否正常。

![连接数据库](img11.webp)
大功告成。
