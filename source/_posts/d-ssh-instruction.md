---
title: 【文档小记】SSH 使用方法
tags:
  - SSH
  - Linux
toc: true
languages:
  - zh-CN
categories:
  - 文档小记
comments: false
cover: false
date: 2023-10-22 18:01:53
---

记录 SSH 一些使用方法

<!-- more -->

> 记录如何使用 SSH RSA 私钥登录服务器，包括 Windows、Linux 客户端操作以及 Linux 服务端操作。 

## Client 端

### Windows

* 安装 `OpenSSH Client` 和 `OpenSSH Server`

* Powershell（管理员）中输入：  
  
  ```bash
  Start-Service sshd
  ```

* 命令行中输入：  
  
  ```bash
  ssh Frank@localhost
  ```
  
  测试连接成功，但需要输入密码。

* 通过 RSA 私钥登录：
  
  ```bash
  ssh-keygen -t rsa
  ```
  
  在 `~/.ssh` 目录下得到 `id_rsa.pub` 公钥和 `id_rsa` 私钥。

* 将 `id_rsa.pub` 中的内容复制到 Server 端的 `~/.ssh/authorized_keys` 文件中。提供一个可行的方法：

  ```bash
  # 1. 直接通过 SSH 连接 Server 端
  ssh <user>@<hostname>

  # 2. echo 的方式加入到 ~/.ssh/authorized_keys 文件中
  echo <content_of_id_rsa_pub> >> ~/.ssh/authorized_keys
  ```

* 在 `.ssh/` 目录下新建文本文件 `config`，内容如下： 
  
  ```bash
  Host <host>
  HostName <hostname> # ip address or domain
  User <user>
  PreferredAuthentications publickey
  IdentityFile <.ssh/id_rsa>
  ```

* 验证连接：
  
  ```bash
  ssh <host>
  ```
  
  当所有操作正确完成后，连接成功。


### Linux
* 所有操作与 Windows 几乎相同，在复制 `id_rsa.pub` 公钥的时候，还可以通过以下方式：

  ```bash
  ssh-copy-id -i <~/.ssh/id_rsa.pub> <user>@<hostname>
  ```

---

## Server 端

> 要注意的是 `.ssh/` 目录下的文件权限，通过 `chmod` 命令修改。

* 安装 `openSSH`：
  
  ```bash
  sudo apt install openssh-server
  ```
  
  启动：  
  
  ```bash
  /etc/init.d/ssh start
  ```

* 使用 `vim` 打开 `/etc/ssh/sshd_config`，添加以下内容：
  
  ```bash
  AuthorizedKeysFile .ssh/authorized_keys 
  
  PubkeyAuthentication yes
  ```

* 通过 `ssh-keygen` 工具生成 `.ssh` 目录。

* 如果 `.ssh` 目录下没有 `authorized_keys` 文件，创建一个：
  
  ```bash
  touch ~/.ssh/authorized_keys
  ```

* 将 `服务器自己的公钥` 和 `远程登录机器对应的公钥` 复制到 `authorized_keys` 文件中：
  
  ```bash
  cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
  ```

* 修改目录权限：
  
  ```bash
  # ～: 750 
  # ~/.ssh: 700 
  # ~/.ssh/*: 600 
  # ~/.ssh/config: 700
  
  chmod 700 ~/.ssh
  chmod 600 .ssh/authorized_keys
  ```

* 理论上可以运行成功。

---

## 管理会话 —— screen

管理会话，SSH 时断开连接也不中断正在运行的进程，重新连接 SSH 后可以恢复。

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