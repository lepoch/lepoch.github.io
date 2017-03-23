---
title: 升级python
date: 2017-03-23 17:00:00
tags:
 - Linux
 - CentOS

categories:
 - Linux


---

升级CentOS 自带的 python 2.7.5 到 2.7.11

<!-- MORE -->
# 环境
LINUX 虚拟机
CentOS 7

# 升级 [python](https://www.python.org/)

```
# 先装 zlib
yum install -y zlib1g-dev zlib zlib-devel

wget https://www.python.org/ftp/python/2.7.11/Python-2.7.11.tgz
tar -zxf Python-2.7.11.tgz
cd Python-2.7.11



# 安装
mkdir /usr/local/python-2.7.11
./configure --prefix=/usr/local/python-2.7.11

# moudle @TODO
sed -i '/#zlib zlibmodule.c/s/.//' Modules/Setup
sed -i '/#_ssl _ssl.c \\/s/.//' Modules/Setup


make && make install
```

# 替换默认的 python 命令  __慎重__
因新装的python module会导致yum等系统命令问题。
```
mv /bin/python /bin/python_old
mv /usr/bin/python /usr/bin/python_old

ln -s /usr/local/python-2.7.11/bin/python2.7 /bin/python
ln -s /usr/local/python-2.7.11/bin/python2.7 /usr/bin/python
```

## 重装pip
```
wget https://bootstrap.pypa.io/get-pip.py
python get-pip.py
```

## 修改 yum
```
sed -i '1s/$/&_old/' /usr/bin/yum
sed -i '1s/$/&_old/' /usr/libexec/urlgrabber-ext-down
```
