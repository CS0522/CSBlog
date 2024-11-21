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
sudo apt install g++ gdb make build-essentialninja-build rsync zip
  
sudo apt install openjdk-11-jdk

java -version
  
# JAVA_HOME变量配置 
echo "JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64/binjava" >> ~/.bashrc
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
gsettings set org.gnome.shell.extensions.desktop-icons show-trash false
gsettings set org.gnome.shell.extensions.desktop-icons show-home false
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


### systemback

  ```bash
  # systemback
  sudo sh -c 'echo "deb [arch=amd64] http://mirrors.bwbot.org/ stable main" > /etc/apt/sources.list.d/systemback. list'
  sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key 50B2C005A67B264F
  sudo apt-get update
  sudo apt-get install systemback
  ```

### timeshift

```bash
sudo apt install timeshift
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

### VirtualBox

```bash
sudo apt install virtualbox
sudo apt install virtualbox-ext-pack
```

### EasyConnect

EasyConnect 在 Ubuntu 20.04 以上因为依赖问题无法正常打开。

```bash
# 安装软件
sudo dpkg -i ./EasyConnect.deb

# 查看安装路径
dpkg -L easyconnect
# /usr/share/sangfor/EasyConnect

# cd 
cd /usr/share/sangfor/EasyConnect
sudo su

# 查看依赖
ldd ./EasyConnect | grep pango

# 下载缺失依赖文件
# http://kr.archive.ubuntu.com/ubuntu/pool/main/p/pango1.0/
# libpangocairo-1.0-0_1.40.14-1_amd64.deb
# libpangoft2-1.0-0_1.40.14-1_amd64.deb
# libpango-1.0-0_1.40.14-1_amd64.deb

# 对这三个文件进行解压缩，提取出 data.tra.xz -> usr -> lib -> x86_64-linux-gnu 下面的所有文件
# 将得到的 6 个文件全部复制到安装目录下
cp ./* /usr/share/sangfor/EasyConnect

# 启动 EasyConnect
```

### 安装 VMware 17.5.2 虚拟机

```bash
sudo dpkg -i ./*.bundle

# 编译 vmmon、vmnet 模块
sudo apt install gcc-12

# clone 补充模块
git clone https://github.com/mkubecek/vmware-host-modules

git checkout -t origin/workstation-17.5.1

sudo make
sudo make install 

sudo /etc/init.d/vmware start
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
  # 状态显示 Attached，但无法进入
  screen -D -r <session-id>
  
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

### 挂载、卸载 img 镜像

使用 `mount`、`umount` 命令。挂载 `nsg_server.img` 文件为例子。

```bash
# NSG_SERVER.img
# 1. 查看 NSG_SERVER.img 的分区情况，是为了看需要挂载哪个分区
fdisk ./NSG_SERVER.img
# result: 
# Disk NSG_SERVER.img：20 GiB，21474836480 字节，41943040 个扇区
# 单元：扇区 / 1 * 512 = 512 字节
# 扇区大小(逻辑/物理)：512 字节 / 512 字节
# I/O 大小(最小/最佳)：512 字节 / 512 字节
# 磁盘标签类型：gpt
# 磁盘标识符：D85F133D-8179-4557-86F6-6DF45C769770
# 设备             起点     末尾     扇区 大小 类型
# NSG_SERVER.img1  2048     4095     2048   1M BIOS 启动
# NSG_SERVER.img2  4096 41940991 41936896  20G Linux 文件系统

# 2. 挂载 .img2 分区到本机 /mnt/nsg_server
# 注意其中 offset = 4096 * 512，因为扇区大小为 512 字节
mkdir /mnt/nsg_server
sudo mount -o loop,offset=2097152 NSG_SERVER.img /mnt/nsg_server

# 3. 卸载 .img2 分区
sudo umount /mnt/nsg_server
```

### VirtualBox 挂载共享文件夹

在本地中创建共享的文件夹，我的共享文件夹为：`/mnt/nsg_server/home/icecream`，想要挂载到虚拟机中的 `/mnt/nsg_server` 中。

本地电脑中设置共享文件夹路径（如上）以及共享文件夹名称 `icecream`，注意**不要勾选自动挂载**，会出现权限问题；

虚拟机中：

```bash
# 创建挂载点
sudo mkdir /mnt/nsg_server

# 挂载命令
# mount -t vboxsf <共享文件夹名称> <挂载目录>
sudo mount -t vboxsf icecream /mnt/nsg_server/

# 开机自动挂载
sudo gedit /etc/sftab

# 文末添加
# <共享文件夹名称> < 挂载目录> vboxsf defaults 0 0
icecream /mnt/nsg_server/ vboxsf defaults 0 0
```

### vim 操作

```bash
# 显示行号
:set number 

# 删除单行
dd
:<num>d

# 删除多行
:<begin><end>d
```

### img、vdi 格式转换

通过 VirtualBox 的 VBoxChange 工具进行转换

```bash
# vdi to img
VBoxManage clonehd test.vdi test.img --format raw

# img to vdi 
VBoxManage convertfromraw test.img test.vdi --format vdi
```


### 彻底删除 Snap

```bash
# 1. 删掉所有的已经安装的 Snap 软件
for p in $(snap list | awk '{print $1}'); do
  sudo snap remove $p
done
# 直到出现：No snaps are installed yet. Try 'snap install hello-world'.

# 2. 删除 Snap 的 Core 文件
sudo systemctl stop snapd
sudo systemctl disable --now snapd.socket

for m in /snap/core/*; do
   sudo umount $m
done

# 3. 删除 Snap 的管理工具
sudo apt autoremove --purge snapd

# 4. 删除 Snap 的目录
rm -rf ~/snap
sudo rm -rf /snap
sudo rm -rf /var/snap
sudo rm -rf /var/lib/snapd
sudo rm -rf /var/cache/snapd
```


### 关闭 systemd-resolved 

自启动且会占用 53 端口。

```bash
# 停用 systemd-resolved 并取消开机自动启动
sudo systemctl disable systemd-resolved
sudo systemctl stop systemd-resolved

# 修改 NetworkManager 配置，让它能自动获取 dns
sudo vim /etc/NetworkManager/NetworkManager.conf

# 添加一行 dns=default
dns=default
# [main]
# plugins=ifupdown,keyfile
# dns=default
# [ifupdown]
# managed=false
# [device]
# wifi.scan-rand-mac-address=no

# 删除 /etc/resolv.conf
sudo unlink /etc/resolv.conf
sudo touch /etc/resolv.conf

# 重启 NetworkManager
sudo systemctl restart NetworkManager

# 查看 /etc/resolv.conf 中新的 dns
cat /etc/resolv.conf
```


### 重装网卡

重装网卡后还需要修改 DNS。

```bash
# 查看网卡名
ifconfig -a
# ens3

sudo dhclient <nic_name>
sudo ifconfig <nic_name>
```


### 设置 DNS

```bash
# 1. 修改 /etc/systemd/resolved.conf
sudo vim /etc/systemd/resolved.conf
# 修改：DNS=114.114.114.114 8.8.8.8

# 2. 重启 systemd-resolved
systemctl restart systemd-resolved
systemctl enable systemd-resolved
 
mv /etc/resolv.conf  /etc/resolv.conf.bak
ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf
```

### 网卡消失

问题：

```bash
sudo lshw -c network

# output:
# -network DISABLED
```

解决方案：

```bash
sudo service NetworkManager stop
sudo rm  /var/lib/NetworkManager/NetworkManager.state
sudo vi /etc/NetworkManager/NetworkManager.conf 
# 打开 .conf 文件后，将 managed=false 改为 managed=true
sudo service NetworkManager start
```

`ip link show` 命令可以查看所有网卡。


### 文件分割 sh 脚本

已知文件大小，利用 `dd` 命令

```sh
# Create split_file.sh: 
# 获取文件大小
size=$(stat -c%s "/dataset/bigann/bigann_learn.bvecs")

# 计算一半的大小
half=$((size / 2))

# 提取前半部分
dd if=filename of=part1.bvecs bs=1 count=$half

# 提取后半部分
dd if=filename of=part2.bvecs bs=1 skip=$half
```

### sftp 大文件断点续传

```bash
# sftp user@hostname
sftp root@hust-server

# cd 到指定目录
sftp> cd /root.....

# -a 表示使用增量续传的方式
sftp> get -a nsg_100G.img

# 上传
sftp> put -a nsg_100G.img
```

### 将命令放至后台

* `&` 放后台

    ```bash
    sleep 10s &
    ```

* `nohup` 命令

    ```bash
    nohup sleep 10s &
    ```

* `&>/dev/null &` 重定向

    输出重定向、完全挂后台

    ```bash
    ./build/bin/nvmf_tgt &>/dev/null &
    ```