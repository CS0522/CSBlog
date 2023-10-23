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

> This document shows how to set up SSH client (no-password login) on Windows and Linux and SSH server on Linux (Windows is complicated because of the authority of the `.ssh/*` files).  

## Client Side

### Windows

* Install `OpenSSH Client` and `OpenSSH Server` (not necessary)  

* Powershell (Admin) input   
  ```bash
  Start-Service sshd
  ```

* Command Prompt input   
  ```bash
  ssh Frank@localhost
  ```
  and connect successfully, but it needs password;

* For no-password, use Git to generate rsa key  
  ```bash
  ssh-keygen -t rsa
  ```
  and get files `id_rsa` and `id_rsa.pub` under dir `~/.ssh`

* Copy the content in the `id_rsa.pub` to  `~/.ssh/authorized_keys` which is on the server

* Create a file called `config` under the `.ssh/`, in the `config` file  
  ```bash
  Host [anynameuwant]
  HostName [host ip or domain]
  User [username on server]
  PreferredAuthentications publickey
  IdentityFile [.ssh/id_rsa]
  ```

* Finally, Command Prompt input  
  ```bash
  ssh [anynameuwant]
  ```
  and it will connect successfully without password if u have done all above correctly.  


### Linux
* Almost the same as Windows, but has a more convenient way to copy `id_rsa.pub`. Terminal input  
  ```bash
  ssh-copy-id -i [~/.ssh/id_rsa.pub] [user]@[server]
  ```


## Server Side
> The most essential thing is to pay attention to the authority of the dir `.ssh`. Use `chmod` command.

* Install `openSSH`  
  ```bash
  sudo apt install openssh-server
  ```
  and start it:  
  ```bash
  /etc/init.d/ssh start
  ```

* Use `vim` and open `/etc/ssh/sshd_config`, add following contents
  ```bash
    AuthorizedKeysFile .ssh/authorized_keys 
  
    PubkeyAuthentication yes
  ```

* Generate `.ssh` directory by using `ssh-keygen`

* If no `authorized_keys` file, generate one (and need to change authority in following steps)  
  ```bash
  touch ~/.ssh/authorized_keys
  ```

* 将`服务器自己的公钥` 和 `远程登录机器对应的公钥` 复制到 `authorized_keys` 文件中  
  ```bash
  cat ~/.ssh/[id_rsa name].pub >> ~/.ssh/authorized_keys
  ```

* Finally change authority  
  ```bash
  ～: 750,  
  `~/.ssh: 700,  
  `~/.ssh/*: 600,  
  `~/.ssh/config: 700,
  
  chmod 700 ~/.ssh
  chmod 600 .ssh/authorized_keys
  ```

* Theoretically it works