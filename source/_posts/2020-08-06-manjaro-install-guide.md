---
title: Manjaro安装指南
date: 2020-08-06 14:42:48
tags:
- Linux
- Manjaro
---

本文为个人安装并配置Manjaro系统的指南，可供后人安装学习指导用

<!--more-->

# 源设置
1. 首先更换源地址为国内地址，执行完会弹出一个框选择，一般选择前3个即可

   ```shell
   sudo pacman -Syy
   sudo pacman-mirrors -i -c China -m rank
   sudo pacman -Syyu	
   ```

2. 添加aur的源，aur就是各种第三方软件包的源，正是因为aur，arch linux才有如此多的软件可供下载

   ```shell
   编辑/etc/pacman.conf文件，加入下面的内容：
   [archlinuxcn]
   SigLevel = Optional TrustAll
   Server = https://mirrors.sjtug.sjtu.edu.cn/archlinux-cn/$arch
   ```

   执行`sudo pacman -Syu && sudo pacman -S archlinux-keyring && sudo pacman -Syyu`此时系统更新完毕，接下来需要安装必需软件



# 配置

## 输入法

1. 安装fcitx5：

   ```shell
   sudo pacman -S fcitx5-im fcitx5-chinese-addons fcitx5-pinyin-zhwiki
   ```

2. 设置环境变量，在`~/.pam_environment`文件（如果文件不存在就新建一个）末尾加上：

   ```shell
   GTK_IM_MODULE DEFAULT=fcitx
   QT_IM_MODULE  DEFAULT=fcitx
   XMODIFIERS    DEFAULT=\@im=fcitx
   SDL_IM_MODULE DEFAULT=fcitx
   export QT_IM_MODULE=fcitx5
   ```
   
3. 在`~/.xprofile`文件中添加
   ```shell
   export GTK_IM_MODULE=fcitx5
   export QT_IM_MODULE=fcitx5
   export XMODIFIERS="@im=fcitx5"
   ```
4. 安装系统依赖库 `pacman -S --needed base-devel`

5. 卸载iBus `pacman -Rs iBus`
   
6. 重启电脑后打开fcitx5的设置，添加输入法

## Git
1. git config --global credential.helper store
2. git config --global https.proxy 127.0.0.1:7890
3. git config --global user.name ""
4. git config --global user.email ""

### Shell

1. 安装oh my zsh `sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"`

2. 安装powerline shell

   ``` shell
   sudo pacman -S python-pip
   pip install powerline-shell
   ```

   然后在.zshrc文件中添加以下内容

   ```
   function powerline_precmd() {
       PS1="$(powerline-shell --shell zsh $?)"
   }
   
   function install_powerline_precmd() {
     for s in "${precmd_functions[@]}"; do
       if [ "$s" = "powerline_precmd" ]; then
         return
       fi
     done
     precmd_functions+=(powerline_precmd)
   }
   
   if [ "$TERM" != "linux" ]; then
       install_powerline_precmd
   fi
   ```

   执行 `source .zshrc`

   安装自动补全和高亮插件

    ```shell
     git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
     git clone https://github.com/zsh-users/zsh-syntax-highlighting ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
    ```

   编辑.zshrc，添加 `plugins=(zsh-autosuggestions zsh-syntax-highlighting)`

## Nvidia独显驱动

update: 2023年开始不需要在手动安装驱动了，系统安装完会自带

1. 查询可用驱动`mhwd -l -d --pci`
2. 安装驱动`sudo mhwd -i pci video-nvidia`
3. 重启

## 科学上网

### shadowsocks

1. 下载ShadowSocks-qt5

2. 添加代理服务器

3. 浏览器下载switchyOmega插件，设置代理地址端口为软件里配置的地址

4. 在/etc/profiles或者.zshrc里添加

   ```shell 
   export http_proxy="socks5://127.0.0.1:7890"
   export https_proxy="socks5://127.0.0.1:7890"
   ```
### Clash

5. 下载Clash for windows

6. 导入配置文件

7. 设置系统代理

8. 在/etc/profiles或者.zshrc里添加

   ```shell 
   export http_proxy="https://127.0.0.1:7890"
   export https_proxy="https://127.0.0.1:7890"
   ```

## 其他配置

### Gnome exteison

1. Gnome-tweaks里设置合盖不睡眠（如果硬件睡眠会导致无法唤醒）

2. Gnome-extension里设置dock栏位置和大小

3. Gnome-extension里开启places status indicator

4. Gnome-extension安装Clipboard Indicator

5. Gnome-extension开启ArcMenu

6. Gnome-extension安装Vitals

7. 禁用Ctrl + alt + left or right，与IDEA上一个光标快捷键冲突

   ```shell
   gsettings set "org.gnome.desktop.wm.keybindings" switch-to-workspace-left "['']"
   gsettings set "org.gnome.desktop.wm.keybindings" switch-to-workspace-right "['']"
   gsettings set "org.gnome.desktop.wm.keybindings" move-to-workspace-right "['']"
   gsettings set "org.gnome.desktop.wm.keybindings" move-to-workspace-left "['']"
   ```

8. tmux.conf里加上set -g escape-time 0，消除tmux里neovim切换模式时的延迟

## 常用软件

- Firefox（自带）
- JDK
- Docker
- ~~Electron-ssr~~ // 已不维护
- ShadowSocks-qt5
- Clash for windows（推荐）
- Typora
- VLC
- Telegram
- Jetbrains toolbox 
- Dbeaver
- Apifox
- Deepin-wine-wechat
- Chrome
- Albert
- Onedrive
- Wps
- ~~Flameshot~~
- tmux
