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
  sudo apt install g++ gdb make build-essential ninja-build rsync zip
  
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

---

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


### 安装 deb 软件包
  
  ```bash
  # deb package
  sudo dpkg -i *.deb
  ```
  
### vim
  
  ```bash
  # vim
  sudo apt install gedit vim
  ```

### 主菜单编辑软件
  
  ```bash
  # 主菜单编辑软件
  sudo apt install alacarte
  ```

### gnome-tweak-tool
  
  ```bash
  # gnome-tweak-tool
  sudo apt install gnome-tweak-tool
  ```
  
### copyQ
  
  ```bash
  # copyQ
  sudo add-apt-repository ppa:hluk/copyq
  sudo apt update
  sudo apt install copyq
  ```
  
### indicator & stickynotes

  ```bash
  # indicator stickynotes
  sudo add-apt-repository ppa:umang/indicator-stickynotes
  sudo apt-get update
  sudo apt-get install indicator-stickynotes
  ```
  
### jetbrains IDE

  ```bash
  # jetbrains (idea & clion)
  sudo snap install intellij-idea-ultimate --classic
  sudo snap install clion --classic
  ```
  
### vscode

  ```bash
  # vscode
  sudo dpkg -i code.deb
  ```
  
### edge
  
  ```bash
  # ms-edge
  sudo dpkg -i edge.deb
  ```

### systemback

  ```bash
  # systemback
  sudo sh -c 'echo "deb [arch=amd64] http://mirrors.bwbot.org/ stable main" > /etc/apt/sources.list.d/systemback. list'
  sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key 50B2C005A67B264F
  sudo apt-get update
  sudo apt-get install systemback
  ```

### nvm, node

  ```bash
  # 通过 nvm 安装 nodejs npm
  curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash
  nvm install node
  ```

### conda

  ```bash
  # anaconda3
  wget -P /tmp https://repo.anaconda.com/archive/Anaconda3-2020.02-Linux-x86_64.sh
  sudo gedit ~/.bashrc
  # .bashrc
  export PATH="/home/用户名/anaconda3/bin:$PATH"
  source ~/.bashrc
  ```

### screen

> 管理会话，SSH 时断开连接也不中断进程

  ```bash
  # 安装 screen
  sudo apt install screen
  
  # 新建 screen
  screen -S <name>
  
  # 进入 screen
  screen -r <name>
  
  # 退出当前 screen
  # 在当前 screen 下
  Ctrl+A，Ctrl+D  
  
  # 显示 screen list
  ​​​​​​​screen -ls
  
  # 删除指定 screen
  # 在当前 screen 下
  Ctrl+D
  # 不在当前 screen 下
  ​​​​​​​screen -S <name> -X quit
  ```



## 常用命令 / 解决方案

### 显示内存

    ```bash
    free
    free -m
    free -h
    ```

### screen

> 管理会话，SSH 时断开连接也不中断进程

  ```bash
  # 安装 screen
  sudo apt install screen
  
  # 新建 screen
  screen -S <name>
  
  # 进入 screen
  screen -r <name>
  
  # 退出当前 screen
  # 在当前 screen 下
  Ctrl+A，Ctrl+D  
  
  # 显示 screen list
  ​​​​​​​screen -ls
  
  # 删除指定 screen
  # 在当前 screen 下
  Ctrl+D
  # 不在当前 screen 下
  ​​​​​​​screen -S <name> -X quit
  ```


### 增加虚拟内存交换空间

未开启交换空间：

```bash
sudo su
# 查看内存使用情况
free -h

# 创建 swap 文件
# 单位 bs 为 G，大小 count 为 20，创建 20G 交换空间
dd if=/dev/zero of=/swapfile bs=1G count=20

# 激活 swap 文件
chmod 600 swapfile
mkswap swapfile

# 开启 swap
swapon swapfile

free -h
# 查看已开启的交换空间
swapon --show
```

已开启交换空间，重新修改 swap 大小：

```bash
# 查看内存使用情况
free -h

# 关闭指定 swap
swapoff /swapfile

# 重新分配
fallocate -l 30G /swapfile

chmod 600 swapfile
mkswap swapfile
swapon swapfile
```

### 动态库无法链接 cannot open shared object file

```bash
sudo vim /etc/ld.so.conf

# 在 ld.so.conf 中加入 *.so 的绝对路径，如
# /usr/local/lib

# 对配置文件 /etc/ld.so.conf 中定义的路径下的程序库重新建立必要的链接
sudo ldconfig
```

### 