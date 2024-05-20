---
title: 【文档小记】Anaconda3 使用方法
tags:
  - anaconda
toc: true
languages:
  - zh-CN
categories:
  - 文档小记
comments: false
cover: false
date: 2023-10-22 18:00:54
---

记录 Anaconda3 的使用方法

<!-- more -->

## Install
```bash
wget -P /tmp https://repo.anaconda.com/archive/Anaconda3-2020.02-Linux-x86_64.sh

sudo gedit ~/.bashrc
# .bashrc
export PATH="/home/<username>/anaconda3/bin:$PATH"

source ~/.bashrc
```

## Use
```bash
# set no auto activate
conda config --set auto_activate_base false

# create a new env
conda create -n <env_name>

# check all envs
conda info --envs

# activate the env
conda activate <env_name>

# deactivate env
conda deactivate <env_name>

# install package
conda install <package_name>

# list packages
conda list

# remove env
conda remove -n <env_name> --all

# set one env's python version
conda install python==3.8 -n "<env_name>"

# if pip failed
# Linux
sudo mkdir ~/.pip

sudo touch ~/.pip/pip.conf

sudo vim ~/.pip/pip.conf

# in pip.conf
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
[install]
trusted-host=mirrors.aliyun.com

```