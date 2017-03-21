---
title: linux科学上网
date: 2017-03-20 17:00:00
tags:
 - LINUX

categories:
 - LINUX


---

在搭建Rietveld时，碰到需要科学上网的情况。

这里使用 [ss-redir](https://github.com/madeye/shadowsocks-libev) + iptables透明代理

<!-- MORE -->
# 环境
LINUX 虚拟机
CentOS 7

# shadowsocks-libev

## 安装
```
yum install epel-release -y
yum install gcc gettext autoconf libtool automake make pcre-devel asciidoc xmlto udns-devel libev-devel -y

#添加repo， 这里遇到$basearch是CPU指令集， 使用arch命令查看
( cat <<EOF
[librehat-shadowsocks]
name=Copr repo for shadowsocks owned by librehat
baseurl=https://copr-be.cloud.fedoraproject.org/results/librehat/shadowsocks/epel-7-`arch`/
type=rpm-md
skip_if_unavailable=True
gpgcheck=1
gpgkey=https://copr-be.cloud.fedoraproject.org/results/librehat/shadowsocks/pubkey.gpg
repo_gpgcheck=0
enabled=1
enabled_metadata=1
EOF
) > /etc/yum.repos.d/shadowsocks.repo

yum update
yum install -y shadowsocks-libev

# 配置
( cat <<EOF
{
    "server":"IP地址" `arch`,
    "server_port":端口,
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":"密码",
    "timeout":300,
    "method":"加密方式",
    "fast_open":false
}
EOF
) > /etc/shadowsocks.json

# 启动
sslocal -c /etc/shadowsocks.json
```
## 问题
然而报错 Exception: libsodium not found，发现是系统没有我指定的加密算法chacha20，所需 [libsodium](https://download.libsodium.org/doc/installation/)

```
yum install m2crypto gcc -y
wget -N --no-check-certificate  https://download.libsodium.org/libsodium/releases/libsodium-1.0.10.tar.gz
tar zfvx libsodium-1.0.10.tar.gz
cd libsodium-1.0.10
./configure
make && make install
(cat <<EOF
/lib
/usr/lib64
/usr/local/lib
EOF
) >> /etc/ld.so.conf
ldconfig
```

# iptables
@TODO
