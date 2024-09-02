---
title: 【学习笔记-存储】NVMeoF（二）：NVMe over RDMA 环境部署
tags:
  - 存储
  - NVMe
toc: true
languages:
  - zh-CN
categories:
  - 学习笔记
  - 存储
comments: false
cover: false
date: 2024-08-25 16:42:26
---

NVMe over RDMA 环境部署记录。

<!-- more -->

> 前提虚拟机环境要求：  
> 1. Ubuntu 22.04  
> 2. Ubuntu 虚拟机的磁盘配置为 `NVMe`  
> 3. clone 为两份虚拟机，一份为 `Host` 端，一份为 `Target` 端

## 1. 自动部署软件栈 sh 脚本

以 root 权限运行：

```bash
sudo sh ./rdma-setup.sh
```

```bash
# rdma-setup.sh

#!/bin/bash
# @CS0522
# rdma setup
# run with root
# DOING:
# 1. Install tools
# 2. Setup rdma & nvme

# install 
apt-get install vim open-vm-tools open-vm-tools-desktop net-tools nvme-cli fio

# rdma user space tools
apt-get install libibverbs1 ibverbs-utils librdmacm1 libibumad3 ibverbs-providers rdma-core

# rdma test tool
apt-get install perftest

# get local IP address
local_ip=`ifconfig -a | grep inet | grep -v 127.0.0.1 | grep -v inet6 | awk '{print $2}' | tr -d "addr:"`
echo local_ip="${local_ip}"

# get device name
net_dev=`ifconfig | grep -w BROADCAST | awk '{print $1}' | sed 's/://g'`
echo net_dev="${net_dev}"

# modprobe RXE NIC
modprobe rdma_rxe
rdma link add rxe_0 type rxe netdev ${net_dev}

# modprobe nvme
modprobe nvmet
modprobe nvmet-rdma
modprobe nvme-rdma

# 1. config nvme subsystem
subsys_name="nvme-rdma-test"
echo subsys_name="${subsys_name}"
mkdir /sys/kernel/config/nvmet/subsystems/"${subsys_name}"
cd /sys/kernel/config/nvmet/subsystems/"${subsys_name}"

# 2. allow any host to be connected to this target
echo 1 > attr_allow_any_host

# 3. create a namespace，example: nsid=10
nsid=10
echo nsid="${nsid}"
mkdir namespaces/"${nsid}"
cd namespaces/"${nsid}"

# 4. set the path to the NVMe device
echo -n /dev/nvme0n1> device_path
echo 1 > enable

# 5. create the following directory with an NVMe port
portid=1
echo portid="${portid}"
mkdir /sys/kernel/config/nvmet/ports/"${portid}"
cd /sys/kernel/config/nvmet/ports/"${portid}"

# 6. set ip address to traddr
echo "${local_ip}" > addr_traddr

# 7. set rdma as a transport type，addr_trsvcid is unique.
echo rdma > addr_trtype
echo 4420 > addr_trsvcid

# 8. set ipv4 as the Address family
echo ipv4 > addr_adrfam

# 9. create a soft link
ln -s /sys/kernel/config/nvmet/subsystems/"${subsys_name}" /sys/kernel/config/nvmet/ports/"${portid}"/subsystems/"${subsys_name}"

# 10. Check dmesg to make sure that the NVMe target is listening on the port
dmesg -T | grep "enabling port"

# 11. output info < ip/port>
#  XXX  nvmet_rdma: enabling port 1 (192.168.225.131:4420)

# 12. check the status of nvme
lsblk | grep nvme
# nvme0n1
```

运行结果：

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/n-nvmeof-02/run-sh.png)


## 2. 手动部署——RDMA

### 2.1. 确认内核是否支持

```bash
cat /boot/config-$(uname -r) | grep RXE

# output:
# CONFIG_RDMA_RXE=m

# if not m or y, then cannot support RXE
```

### 2.2. 安装环境依赖、工具

```bash
apt-get install vim open-vm-tools open-vm-tools-desktop net-tools nvme-cli fio

apt-get install libibverbs1 ibverbs-utils librdmacm1 libibumad3 ibverbs-providers rdma-core

apt-get install perftest
```

### 2.3. 获取 IP 地址、NIC 设备名

```bash
ifconfig -a

# output:
# ens33: ......
# inet: 192.168.159.140
# ......
# net_dev = ens33
# local_ip = 192.168.159.140
```

### 2.4. 配置 RXE 网卡

```bash
# 加载内核驱动
modprobe rdma_rxe

# 用户态配置
sudo rdma link add rxe_0 type rxe netdev ens33
# rxe_0: RDMA 设备名，可任意取名
# ens33: 网卡名，会发生变化，根据机器 ifconfig 的结果为准
```

### 2.5. RDMA 配置结果检验

```bash
rdma link

# example output: 
# link rxe_0/1 state ACTIVE physical_state LINK_UP netdev ens33

ibv_devinfo -d rxe_0
# example output: 
# hca_id: rxe_0
# ...
# port: 1
#       state: PORT_ACTIVE
#       ......
```


## 3. 手动部署——NVMe

### 3.1. 加载 NVMe 相关内核

```bash
modprobe nvmet
modprobe nvmet-rdma
modprobe nvme-rdma
```

### 3.2. 配置 nvme subsystem

```bash
# subsys_name 可替换
mkdir /sys/kernel/config/nvmet/subsystems/<subsys_name>
cd /sys/kernel/config/nvmet/subsystems/<subsys_name>
```

### 3.3. Allow any host to be connected to this target

```bash
echo 1 > attr_allow_any_host
```

### 3.4. Create a namespace，example: nsid=10

```bash
# nsid 可替换
mkdir namespaces/<nsid>
cd namespaces/<nsid>
```

### 3.5. Set the path to the NVMe device

```bash
echo -n /dev/nvme0n1> device_path
echo 1 > enable
```

### 3.6. Create the following directory with an NVMe port

```bash
# portid 可替换
mkdir /sys/kernel/config/nvmet/ports/<portid>
cd /sys/kernel/config/nvmet/ports/<portid>
```

### 3.7. Set ip address to traddr

```bash
# local_ip 为 ifconfig 中的结果
echo <local_ip> > addr_traddr
```

### 3.8. Set rdma as a transport type，addr_trsvcid is unique

```bash
echo rdma > addr_trtype
echo 4420 > addr_trsvcid
```

### 3.9. Set ipv4 as the Address family

```bash
echo ipv4 > addr_adrfam
```

### 3.10. create a soft link

```bash
# subsys_name、portid 可替换
ln -s /sys/kernel/config/nvmet/subsystems/<subsys_name> /sys/kernel/config/nvmet/ports/<portid>/subsystems/<subsys_name>
```

### 3.11. Check dmesg to make sure that the NVMe target is listening on the port

```bash
dmesg -T | grep "enabling port"

# output: 
# nvmet_rdma: enabling port 1 (192.168.225.131:4420)
```

### 3.12. Check the status of NVMe

```bash
lsblk | grep nvme
# nvme0n1
```


## 4. NVMe over RDMA 环境验证

**虚拟机重启后需要重新配置 RDMA 部分**

`Host` 端执行：

```bash
ib_send_bw -d rxe_0
```

`Target` 端执行：

```bash
ib_send_bw -d rxe_0 <server_ip>

# ib_send_bw -d rxe_0 192.168.159.141
```

`Target` 端执行后，两端的运行输出如下：

`Host` 端：

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/n-nvmeof-02/server-verify-rdma.png)

`Target` 端：

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/n-nvmeof-02/client-verify-rdma.png)


在 `Target` 端发现 `Host` 端的 NVMe 设备：

```bash
sudo nvme discover -t rdma -q <subsys_name> -a <server_ip> -s 4420
```

结果：

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/n-nvmeof-02/client-discover-nvmeof.png)

在 `Target` 端连接 `Host` 端的 NVMe 设备：

```bash
sudo nvme connect -t rdma -q <subsys_name> -n <subsys_name> -a <server_ip> -s 4420
```

结果：

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/n-nvmeof-02/client-connect-nvmeof.png)


断开连接：

```bash
sudo nvme disconnect -n <subsys_name>
```