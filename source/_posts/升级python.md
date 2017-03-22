---
title: 升级python
date: 2017-03-22 17:00:00
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

```
wget https://www.python.org/ftp/python/2.7.11/Python-2.7.11.tgz
tar -zxf Python-2.7.11.tgz
cd Python-2.7.11
mkdir /usr/local/python-2.7.11
./configure --prefix=/usr/local/python-2.7.11
make && make install

rm -f /bin/python
ln -s /usr/local/python-2.7.11/bin/python2.7 /bin/python

# 重装pip

```
