---
title: 【学习笔记】Linux Shell 脚本编程
tags:
  - Shell
  - Linux
toc: true
languages:
  - zh-CN
categories:
  - 学习笔记
  - Shell
comments: false
cover: false
date: 2024-10-16 19:56:02
---

项目中用到 Shell 脚本的编写，边学边用。

<!-- more -->

## 运行

```bash
# 1. 调用 sh 命令（不推荐）
sh ./setup_rdma.sh

# 2. 修改权限为可执行
sudo chmod 777 ./setup_rdma.sh
./setup_rdma.sh
```

## 格式

类似于 `python`，同为脚本语言，直接写命令就可以直接运行执行。

```sh
# xx.sh
#!/usr/bin/env bash

command1; command2

command3
command4
```

最重要的是注意空格，空格不正确会报错。

## 