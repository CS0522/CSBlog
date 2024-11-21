---
title: 【文档小记】Git 使用方法
tags:
  - Git
  - GitHub
toc: true
languages:
  - zh-CN
categories:
  - 文档小记
comments: false
cover: false
date: 2023-10-22 18:01:13
---

记录 Git & GitHub 一些使用方法

<!-- more -->

## Git 基本操作

### SSH key  

1. 创建 SSH key

```bash
ssh-keygen -t rsa -C "812359326@qq.com"
```
2. 复制 key 到 Github SSH 设置中

3. 验证连接  

```bash
ssh -T git@github.com
```

### 配置用户名、email、默认文本编辑器  

```bash
git config --global user.name "Frank" 
git config --global user.email "812359326@qq.com"
git config --global core.editor gedit
```

### Check config  

```bash
git config --list
```

### 工作区、暂存库、版本库  
  
![Git工作区暂存库](https://www.runoob.com/wp-content/uploads/2015/02/1352126739_7909.jpg)

### 进入仓库、初始化 

```bash
git init
```

### 克隆仓库  
  
```bash
git clone git://github.com/CS0522/GitTest.git ./ideaTest
```

### Git基本操作命令  

![Git工作区暂存库](https://www.runoob.com/wp-content/uploads/2015/02/git-command.jpg)  

![Git工作区暂存库](https://img-blog.csdnimg.cn/20200312120348300.png)

### 提出更改，添加到暂存区  

```bash
git add ./src/ 
git add ./out/  
git add ./ideaTest.iml  
git add *
```

### 查看仓库当前的状态，显示有变更的文件  
  
```bash
git status
```

### 比较文件的暂存区和工作区的差异  
  
```bash
git diff
```

### 改动提交到 HEAD 版本库, 但还没到远端仓库  

```bash
git commit -m "test"
```

### 回退版本，可以指定退回某一次提交的版本  
  
```bash
git reset [--soft | --hard | --mixed] [HEAD]
git reset --hard HEAD~3
```
  
* --soft 参数用于回退到某个版本, 只进行对commit操作的回退，不影响工作区  

* --hard 参数撤销工作区中所有未提交，将暂存区与工作区都回到上一版本  

* --mixed 为默认，可以不用带该参数，重置暂存区与上一次的commit保持一致，工作区文件内容保持不变  

### 删除文件  
  
```bash
git rm [-- cached] [-f | -r] [file]
```
  
* 不带参数时，将文件从暂存区和*工作区中删除
  
* -- cached 文件仅从暂存区域移除，工作目录保留  

### 查看历史提交记录  

```bash
git log
```

### 添加远程仓库  
  
```bash
git remote add [alias] [url | server]
git remote add origin https://github.com/CS0522/GitTest.git
```

### 删除远程仓库、重命名远程仓库  

```bash
git remote rm [name] 
git remote rename [old_name] [new_name]
```

### 查看当前配置的远程仓库  

```bash
git remote -v
```

### 把远程仓库master分支下载到本地并存为temp分支  

```bash
git fetch origin master:temp
```

### 合并本地的temp分支和master分支  

```bash
git merge temp
```

### 拉取远程代码并合并  

```bash
git pull <远程主机名> <远程分支名>:<本地分支名> 
git pull origin master:brantest  
git pull origin master
```

### 推送代码至远程并合并  

```bash
git push <远程主机名> <本地分支名>:<远程分支名>  
git push origin master:master
git push origin master
```

## Git分支管理

### Git分支    
  
![Git分支](https://static.runoob.com/images/svg/git-brance.svg)

### 创建分支  

```bash
git branch [branchname]
```  

* 不带[branchname]时，列出所有分支

### 删除分支  

```bash
git branch -d [branchname]
```

### 切换分支  

```bash
git checkout [-b] [branchname]
```
  
  * -b 表示创建新的分支并切换到该分支下   
  
  * 切换分支的时候，Git 会用该分支的最后提交的快照替换你的工作目录的内容，所以多分支不需要多个目录

### 切换远程分支

列出远程所有分支：

```bash
git branch -r
```

拉取所有信息：

```bash
git fetch -a
```

切换到远程分支：
    
```bash
git checkout -t origin/<branch_name>
```

### 将temp分支和本地的master分支合并  

```bash
git merge temp [--allow-unrelated-histories]
```
* --allow-unrelated-histories 参数会允许合并无关历史，当远程版本远高于本地版本时，可能需要 

### Fork后本地项目同步他人更新

```bash
### 添加上游仓库
git remote add upstream git@github.com:Chtho1ly/WADG_ANN.git

### 拉取最新更改
git fetch upstream

### 切换分支并合并
git checkout master
git merge upstream/master

### 推送到自己的远程仓库
git push -u origin master
```




## Git可能会出现的问题

### 鉴权失败  
  
用户名和密码的登录方式失效，需要到 Dev Settings 设置 Token.

### 每次 pull/push 都要鉴权 
  
```bash
git config --global credential.helper store
```

### 无法合并  
  
本地版本太老，先远程下载分支，然后本地合并:  
  
```bash
git fetch origin master:temp  
git merge temp --allow-unrelated-histories
```

### 想要忽略某个文件(文件夹)  

使用 .gitignore 文件  
  
```bash
# 忽略 Holiday Homework  
Holiday Homework/  
  
# 忽略 *.class  
*.class  
```

### 已经上传至版本库后，想要忽略某个文件(文件夹)  

git清除本地缓存（改变成未track状态），然后再提交  

```bash
git rm -r --cached .
git add .
git commit -m 'update .gitignore'
git push origin master
```