---
title: Swoole初识
date: 2017-05-22T16:55:00.000Z
tags:
  - Swoole
  - Laravel
  - 笔记
---

Swoole初识

<!-- MORE -->
## 背景  
可能是因为Laravel框架原因，Yar并发请求的性能并没有达到预期目标。  
需探索另外的道路。  

## 安装  
### Homestead环境  
```
sudo su
cd ~
wget https://github.com/swoole/swoole-src/archive/v2.0.7.tar.gz
tar -xzf v2.0.7.tar.gz
cd swoole-src-2.0.7/
phpize
./configure --enable-coroutine
make
make install


echo "extension=swoole.so" >> /etc/php/7.1/cli/php.ini
# echo "extension=swoole.so" >> /etc/php/7.1/fpm/php.ini
# /etc/init.d/php7.1-fpm reload
```
