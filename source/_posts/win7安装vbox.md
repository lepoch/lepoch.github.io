---
title: win7安装vbox
date: 2017-03-31T21:14:00.000Z
tags:
  - vbox
  - 笔记
categories:
  - Linux
---

win7下安装virtual box笔记

<!-- MORE -->

# virtual box创建CentOS虚拟机
# win7下使用SSH登录虚拟机
## 使用NAT方式  
使用端口转发，把宿主机的端口，转到虚拟机上去， 如下图  
![img](/img/win7安装vbox/vbox_nat_setting.png)

# win7 与虚拟机的共享文件夹
## 安装增强功能
```
yum install -y gcc gcc-devel gcc-c++ gcc-c++-devel make kernel kernel-devel bzip2
mkdir /mnt/cdrom
mount /dev/cdrom /mnt/cdrom
cd /mnt/cdrom
./VBoxLinuxAdditions.run
```

然后就各种yes，回车了。

## 配置共享文件夹  
在虚拟机设置，共享文件夹，新增得到共享文件夹名称。
```
sudo mount -t vboxsf 共享文件夹名称（在设置页面设置的） 挂载的目录
```
