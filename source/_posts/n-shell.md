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

## 基本语法

详见 [Shell 教程 | 菜鸟教程](https://www.runoob.com/linux/linux-shell.html)

---

## 使用小结

### 运行

```bash
# 1. 调用 sh 命令（不推荐）
sh ./setup_rdma.sh

# 2. 修改权限为可执行
sudo chmod 777 ./setup_rdma.sh
./setup_rdma.sh
```

### 多行注释

```sh
:<<EOF
这是多行注释
EOF
```

### 特殊变量

| 特殊变量名 | 作用 |
| :--: | :--: |
| `$0` | 脚本的名称 |
| `$1`, `$2`, ..., `${10}`, ... | 传递给脚本的参数 |
| `$#` | 传递给脚本的参数数量 |
| `$?` | 上一个命令的退出状态 |
| `$*` | 以一个单字符串显示所有向脚本传递的参数。如 "$*"，输出 "$1 $2 … $n"的形式输出所有参数。|

### 拼接字符串

```sh
your_name="runoob"
# 使用双引号拼接
greeting="hello, "$your_name" !"
greeting_1="hello, ${your_name} !"
```

### 获取字符串长度

```sh
string="abcd"
echo ${#string}     # 输出 4
echo ${#string[0]}  # 输出 4
```

### 提取子字符串

```sh
string="runoob is a great site"
echo ${string:1:4} # 输出 unoo
```

### 查找子字符串

```sh
string="runoob is a great site"
echo `expr index "$string" io`  # 输出 4
```

### 关联数组（类似于 map）

```sh
declare -A site
site["google"]="www.google.com"
site["runoob"]="www.runoob.com"
site["taobao"]="www.taobao.com"

# 使用
echo ${site["runoob"]}
```

### 获取数组所有元素和键

```sh
# @ 或 * 号获取所有元素
# ! 获取所有键
my_array[0]=A
my_array[1]=B
my_array[2]=C
my_array[3]=D

echo "数组的元素为: ${my_array[@]}"
echo "数组的键为: ${!my_array[@]}"
```

### 获取数组长度

```sh
echo "数组元素个数为: ${#my_array[@]}"
```

### 算数运算

`expr` 命令。

表达式和运算符之间要有空格，完整的表达式要被 \` \` 包含。

```sh
num=5

# 自增
num=`expr ${num} + 1`

# 自减
num=`expr ${num} - 1`
```

### echo 开启转义

```sh
echo -e "OK! \n" # -e 开启转义
echo "It is a test"

echo -e "OK! \c" # -e 开启转义 \c 不换行
echo "It is a test"
```

### if...else...fi 判断

括号和里面的条件之间一定要有空格。

```sh
# -gt, -lt, -ne, -eq
# -z: 是否等于 0
if [ $a -eq $b ]; then
   echo "a 等于 b"
elif [ $a -gt $b ]; then
   echo "a 大于 b"
elif [ $a -lt $b ]; then
   echo "a 小于 b"
else
   echo "没有符合的条件"
fi
```

遇到逻辑运算符或 `==, !=, >, <, <=, >=` 时，需要双括号。

```sh
if [[ $a -eq $b ]] && [[ -z $b ]]; then 
    ...
elif [[ ... ]] || [[ ... ]]; then
    ...
fi
```

### while 循环

```sh
while(( ${curr_node}<=${node_num} )); do
    ...
done
```

### 函数内局部变量

`local` 关键字。

```sh
function func() {
    local var1=1
}
```

### Here 文档

Shell 中的一种特殊的重定向方式，用来将输入重定向到一个交互式 Shell 脚本或程序。比如 ssh 等。

`ENDSSH` 分隔符可以自定义，分隔符之间的命令则是需要执行的部分。可以传入参数。

```sh
ssh ${ssh_arg} ${cloudlab_username}@${hostname} << ENDSSH
            sudo su
            cd ${workspace_dir}
            git clone --branch v${spdk_version} ${spdk_repo} ./spdk-${spdk_version} && cd ./spdk-${spdk_version}
            git submodule update --init
            rm -rf ./.git && git init
            exit
ENDSSH

# 这里 res 获取 ssh 执行后的输出结果
# 其中命令中的 $（不是需要传入外部参数）需要转义
res=$(
    ssh ${ssh_arg} ${cloudlab_username}@${hostname} << ENDSSH
            ifconfig | grep -A 1 '${mtu}' | grep 'inet' | awk '{print \$2}'
ENDSSH
    )
```

### awk 截取参数

`-F` 参数可以指定分隔符。

```sh
# 指定分隔符为 ':'
awk -F: '{print $2}'
```