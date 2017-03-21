---
title: linux科学上网
date: 2017-03-20 17:00:00
tags:
 - LINUX

categories:
 - LINUX


---

在搭建Rietveld时，碰到需要科学上网的情况。

这里使用 shadowsocks + privoxy

<!-- MORE -->
# 环境
LINUX 虚拟机
CentOS 7

# 方法1，使用宿主机的代理
```
export http_proxy=宿主机IP:1080
export https_proxy=宿主机IP:1080
export ftp_proxy=宿主机IP:1080
```


# 方法2，独立安装代理
## shadowsocks python版本

### shadowsocks

```
yum install python-pip
pip install shadowsocks

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
sslocal -c /etc/shadowsocks.json -d
```

### 问题
1. 然而报错 Exception: libsodium not found，发现是系统没有我指定的加密算法chacha20，所需 [libsodium](https://download.libsodium.org/doc/installation/)

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


## privoxy
```
yum install privoxy
echo "forward-socks5t   /               127.0.0.1:1080 ." >> /etc/privoxy/config
# 启动
pip install shadowsocks
# 设置代理
export http_proxy=http://127.0.0.1:8118
export https_proxy=http://127.0.0.1:8118
export ftp_proxy=http://127.0.0.1:8118
```

# 成功
```
curl https://google.com
```
