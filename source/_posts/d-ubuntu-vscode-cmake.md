---
title: 【文档小记】记录 VSCode + CMake 多可执行文件配置
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
date: 2024-05-06 12:24:38
---

记录 Ubuntu 下配置 VSCode + CMake 过程，存在多个可执行文件。

<!-- more -->

---

## 前期准备

* Ubuntu 22.04
* VSCode 1.83
* g++ 11.4
* cmake 3.22.1
* VSCode Extension
  * C/C++ 1.19.9
  * CMake 0.0.17
  * <font color = 'red'>不安装 CMake Tools</font>
* CMake 项目，其中包含多个可执行文件，存在 CMake 嵌套

## CMake 项目

### 目录结构

```
.
├── CMakeLists.txt
├── include
│   └── efanna2e
│       ├── distance.h
│       ├── dkm.hpp
│       ├── dkm.hpp.bak
│       ├── exceptions.h
│       ├── index.h
│       ├── index_nsg.h
│       ├── index_wadg.h
│       ├── lru_cache.h
│       ├── neighbor.h
│       ├── parameters.h
│       └── util.h
├── src
│   ├── CMakeLists.txt
│   ├── index.cpp
│   ├── index_nsg.cpp
│   ├── index_wadg.cpp
│   ├── lru_cache.cpp
├── tests
│   ├── CMakeLists.txt
│   ├── test_cal_top_K_precision.cpp
│   ├── test_nsg_index.cpp
│   ├── test_nsg_optimized_search.cpp
│   ├── test_nsg_search.cpp
│   └── test_wadg_search.cpp
```

存在 CMake 的多级嵌套，其中 `./tests` 文件夹中的源文件都存在 `main` 函数，需要分别编译为可执行文件。

### 编译命令

```bash
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
make -j
```

### 执行命令

```bash
test_wadg_search ../../dataset/sift/sift_base.fvecs ../../dataset/sift/sift_query.fvecs ../../nsg_graph/sift.nsg 200 200 ../../search_result/sift_200nn.ivecs

test_nsg_search ../../dataset/sift/sift_base.fvecs ../../dataset/sift/sift_query.fvecs ../../nsg_graph/sift.nsg 200 200 ../../search_result/sift_200nn.ivecs

test_cal_top_K_precision ../../search_result/sift_200nn.ivecs ../../dataset/sift/sift_groundtruth.ivecs 100
```


## VSCode 配置

编译可以使用 `CMake Tools` 插件，但是这样的话 `launch.json` 和 `tasks.json` 不起作用，所以打算手动配置。

### 加入头文件路径

配置 `c_cpp_properties.json` 文件。

```json
// .vscode/c_cpp_properties.json
{
    "configurations": [
        {
            "name": "Linux",
            "includePath": [
                "${workspaceFolder}/**",
                // 加入项目中的头文件
                "${workspaceFolder}/include"
            ],
            "defines": [],
            "cStandard": "c17",
            "cppStandard": "gnu++17",
            "intelliSenseMode": "linux-gcc-x64"
        }
    ],
    "version": 4
}
```

### 编译

配置 `tasks.json` 文件，自定义任务，使用[CMake 编译命令](#编译命令)编译程序。

```json
// .vscode/tasks.json
{
	  "version": "2.0.0",
	  "options": {
        // 指定工作目录
        "cwd": "${workspaceFolder}/build"
	  },
	  "tasks": [
      // 任务 1
		  {
		  	  "type": "shell",
		  	  "label": "cmake",
          // 命令
		  	  "command": "cmake",
          // 参数
		  	  "args": [
              // Release, Debug
              "-DCMAKE_BUILD_TYPE=Debug", ".."
		  	  ]
		  },
      // 任务 2
		  {
		  	  "label": "make",
		  	  "group": {
		  	  	"kind": "build",
		  	  	"isDefault": true
		  	  },
          // 命令
		  	  "command": "make",
          // 参数
		  	  "args": [
              "-j"
		  	  ]
		  },
		  {
		  	  "label": "Build",
		  	  // 依赖于上面两个 task 命令
		  	  "dependsOn": [
            // 与 label 一致
		  	  	"cmake",
		  	  	"make"
		  	  ]
		  }
	]
}
```

### 运行调试

配置 `launch.json` 文件，读取可执行文件。需要进行修改地方的是指定运行的文件，其次我们还可以在里面添加 build 任务，用于调试，添加[命令行参数](#执行命令)执行程序。

```json
{
    "version": "0.2.0",
    "configurations": [
        // 配置 1，运行可执行文件 1
        {
            // name 可自定义
            "name": "test_wadg_search",
            "type": "cppdbg",
            "request": "launch",
            // 需要运行的可执行文件
            "program": "${workspaceFolder}/build/tests/test_wadg_search",
            // 需要输入的命令行参数
            "args": [
                // data_file
                "./dataset/sift/sift_base.fvecs", 
                // query_file  
                "./dataset/sift/sift_query.fvecs", 
                // nsg_path 
                "./nsg_graph/sift.nsg", 
                // search_L 
                "200", 
                // search_K 
                "200", 
                // result_path
                "./search_result/sift_200nn.ivecs"
            ],
            "stopAtEntry": false,
            // 指定工作目录。如果参数包含文件路径，需要注意相对路径
            "cwd": "${workspaceFolder}",
            "environment": [],
            // 终端
            "externalConsole": false,
            "MIMode": "gdb",
            "preLaunchTask": "Build",
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                },
                {
                    "description": "将反汇编风格设置为 Intel",
                    "text": "-gdb-set disassembly-flavor intel",
                    "ignoreFailures": true
                }
            ]
        },
        // 配置 2，运行可执行文件 2
        {
            "name": "test_nsg_search",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/build/tests/test_nsg_search",
            "args": [
                // data_file
                "./dataset/sift/sift_base.fvecs",
                // query_file  
                "./dataset/sift/sift_query.fvecs", 
                // nsg_path 
                "./nsg_graph/sift.nsg", 
                // search_L 
                "200", 
                // search_K 
                "200", 
                // result_path
                "./search_result/sift_200nn.ivecs"
            ],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "preLaunchTask": "Build",
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                },
                {
                    "description": "将反汇编风格设置为 Intel",
                    "text": "-gdb-set disassembly-flavor intel",
                    "ignoreFailures": true
                }
            ]
        },
        // 配置 3，运行可执行文件 3
        {
            "name": "test_cal_top_K_precision",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/build/tests/test_cal_top_K_precision",
            "args": [
                "./search_result/sift_200nn.ivecs", 
                "./dataset/sift/sift_groundtruth.ivecs", 
                "100"
            ],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "preLaunchTask": "Build",
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                },
                {
                    "description": "将反汇编风格设置为 Intel",
                    "text": "-gdb-set disassembly-flavor intel",
                    "ignoreFailures": true
                }
            ]
        }
    ]
}
```

运行选择右上角的图标，选择相应的可执行文件配置运行即可。