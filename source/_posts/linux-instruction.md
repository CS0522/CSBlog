---
title: 【文档小记】Linux 使用方法
tags:
  - Linux
  - Ubuntu
toc: true
languages:
  - zh-CN
categories:
  - 文档小记
comments: false
cover: false
date: 2023-10-22 18:01:26
---

记录 Linux(Ubuntu) 一些使用方法

<!-- more -->

## Ubuntu 环境配置

> [Ubuntu 备份与恢复](https://blog.csdn.net/sinat_27554409/article/details/78227496)  
> [VSCode 创建启动器](https://blog.csdn.net/weixin_39289876/article/details/118484127)

### C++, Java 环境安装配置
  ```bash
  sudo apt-get install g++
  
  sudo apt-get install gdb
  
  sudo apt install g++ gdb make ninja-build rsync zip
  
  sudo apt install openjdk-11-jdk
  
  java -version
  
  # JAVA_HOME变量配置 
  echo "JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64/bin/java" >> ~/.bashrc
  echo $JAVA_HOME
  ```

### 配置root密码
  ```bash
  sudo passwd
  ```

### 中文包  
  ```bash
  sudo apt install language-pack-zh-hans
  sudo update-locale LANG=zh_CN.UTF-8
  ```

### Surface-Linux
> [Surface安装Ubuntu指南](https://blog.csdn.net/weixin_43858776/article/details/109351218)  

<details>
<summary>点击查看</summary>

  ```bash
  wget -qO - https://raw.githubusercontent.com/linux-surface/linux-surface/master/pkg/keys/surface.asc \
  | gpg --dearmor | sudo dd of=/etc/apt/trusted.gpg.d/linux-surface.gpg
  
  echo "deb [arch=amd64] https://pkg.surfacelinux.com/debian release main" \
  | sudo tee /etc/apt/sources.list.d/linux-surface.list
  
  sudo apt update
  
  # if errors
  sudo add-apt-repository ppa:gpxbv/apt-urlfix
  sudo apt-get update
  sudo apt install apt
  
  sudo apt install linux-image-surface    linux-headers-surface iptsd libwacom-surface
  
  sudo systemctl enable iptsd
  
  sudo apt install linux-surface-secureboot-mok
  
  sudo update-grub
  ```
</details>

### 删除桌面回收站、用户文件图标
  ```bash
  gsettings set org.gnome.shell.extensions.desktop-icons  show-trash false
  gsettings set org.gnome.shell.extensions.desktop-icons  show-home false
  ```

### 添加监视器 system monitor
  ```bash
  sudo add-apt-repository ppa:fossfreedom/indicator-sysmonitor
  sudo apt-get install -y indicator-sysmonitor
  ```

### sudo 不输入密码
  ```bash
  sudo vim /etc/sudoers
  
  # 文末添加下句: 
  frank ALL=(ALL:ALL) NOPASSWD: ALL
  ```

## 软件安装

### 数据库
<details>
<summary>点击查看</summary>

  ```bash
  sudo apt install mysql-server
  
  sudo apt install mysql-client
  
  sudo apt install libmysqlclient-dev
  
  sudo vim /etc/mysql/debian.cnf
  
  mysql -u debian-sys-maint -p
  
  flush privileges;
  
  alter user 'root'@'localhost' identified with   caching_sha2_password by '123456frank'
  
  flush privileges
  
  quit
  
  service mysql restart
  ```
</details>

### 其他软件安装
<details>
<summary>点击查看</summary>

  ```bash
  # deb package
  sudo dpkg -i *.deb
  
  # vim
  sudo apt install gedit vim
  
  # 主菜单编辑软件
  sudo apt install alacarte
  
  # gnome-tweak-tool
  sudo apt install gnome-tweak-tool
  
  # copyQ
  sudo add-apt-repository ppa:hluk/copyq
  sudo apt update
  sudo apt install copyq
  
  # indicator stickynotes
  sudo add-apt-repository ppa:umang/indicator-stickynotes
  sudo apt-get update
  sudo apt-get install indicator-stickynotes
  
  # jetbrains (idea & clion)
  sudo snap install intellij-idea-ultimate --classic
  sudo snap install clion --classic
  
  # vscode
  sudo apt-get install wget gpg
  wget -qO- https://packages.microsoft.com/keys/microsoft.    asc | gpg --dearmor > packages.microsoft.gpg
  sudo install -D -o root -g root -m 644 packages.    microsoft.gpg /etc/apt/keyrings/packages.microsoft.gpg
  sudo sh -c 'echo "deb [arch=amd64,arm64,armhf   signed-by=/etc/apt/keyrings/packages.microsoft.gpg]   https://packages.microsoft.com/repos/code stable main"    > /etc/apt/sources.list.d/vscode.list'
  rm -f packages.microsoft.gpg
  
  sudo apt install apt-transport-https
  sudo apt update
  sudo apt install code
  
  # ms-edge
  curl https://packages.microsoft.com/keys/microsoft. asc | gpg --dearmor > microsoft.gpg
  sudo install -o root -g root -m 644 microsoft.gpg /etc/ apt/trusted.gpg.d/
  sudo sh -c 'echo "deb [arch=amd64] https://packages.microsoft.com/repos/edge stable main" > /etc/apt/sources.list.d/microsoft-edge-dev.list'
  sudo rm microsoft.gpg
  
  sudo apt update
  sudo apt install microsoft-edge-dev
  
  # systemback
  sudo sh -c 'echo "deb [arch=amd64] http://mirrors.bwbot.org/ stable main" > /etc/apt/sources.list.d/systemback. list'
  sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key 50B2C005A67B264F
  sudo apt-get update
  sudo apt-get install systemback

  # 通过 nvm 安装 nodejs npm
  curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash
  nvm install node

  # anaconda3
  wget -P /tmp https://repo.anaconda.com/archive/Anaconda3-2020.02-Linux-x86_64.sh
  sudo gedit ~/.bashrc
  # .bashrc
  export PATH="/home/用户名/anaconda3/bin:$PATH"
  source ~/.bashrc
  ```
</details>