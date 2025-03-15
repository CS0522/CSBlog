---
title: 【文档小记】VSCode 运行、调试需要 sudo 权限程序
tags:
  - VSCode
  - CMake
  - Ubuntu
toc: true
languages:
  - zh-CN
categories:
  - 文档小记
comments: false
cover: false
date: 2025-03-15 21:47:56
---

简单记录 VSCode 运行、调试需要 sudo 权限的程序的 launch.json 配置。

<!-- more -->

## 在 etc/sudoers 中添加用户 sudo 无需密码

```bash
sudo vim /etc/sudoers

# 在以下位置中添加 frank 这一行
# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL
frank   ALL=(ALL:ALL) NOPASSWD:ALL
```

## 用户 home 目录下添加 gdbasroot 脚本

```bash
touch ~/gdbasroot
vim ~/gdbasroot

# 添加以下命令
sudo /usr/bin/gdb "$@"

# 保存退出

# 添加可执行权限
sudo chmod +x ~/gdbasroot
```

## 修改 launch.json 文件

修改调试路径 `miDebuggerPath`：

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "g++ - Build and debug active file",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/db/nofdb/table/build/sstable_manager_test",
            "args": [],
            // !! 添加以下属性 !!
            "linux": {
                "miDebuggerPath": "/home/frank/gdbasroot",
                "miDebuggerArgs": "sudo"
            },
            "stopAtEntry": false,
            "cwd": "${fileDirname}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": ""
        }
    ]
}
```
