---
title: 升级python
date: 2017-03-24 17:00:00
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
yum install -y zlib1g-dev zlib zlib-devel openssl-devel

wget https://www.python.org/ftp/python/2.7.11/Python-2.7.11.tgz
tar -zxf Python-2.7.11.tgz
cd Python-2.7.11



# 安装
mkdir /usr/local/python-2.7.11
./configure --prefix=/usr/local/python-2.7.11

# moudle @TODO
sed -i '/#zlib zlibmodule.c/s/.//' Modules/Setup
sed -i '/#_ssl _ssl.c/,+2{s/#//}' Modules/Setup


make && make install
```

# 扩展
## _sqlite
`yum install sqlite-devel -y`
重新编译安装


# 替换默认的 python 命令  __慎重__
因新装的python module会导致yum等系统命令问题。
```
mv /bin/python /bin/python_old  
mv /usr/bin/python /usr/bin/python_old  

ln -s /usr/local/python-2.7.11/bin/python2.7 /bin/python
ln -s /usr/local/python-2.7.11/bin/python2.7 /usr/bin/python
```

## 重装pip，已知所需module：zlib, ssl
```
wget https://bootstrap.pypa.io/get-pip.py
python get-pip.py
```

## yum 修复
### 第一种，指向yum到原来的python， 这种简单些
```
sed -i '1s/$/&_old/' /usr/bin/yum
sed -i '1s/$/&_old/' /usr/libexec/urlgrabber-ext-down
```
### 第二种，[重装yum](http://jingyan.baidu.com/article/ed2a5d1f5a9fbe09f6be17ea.html)
