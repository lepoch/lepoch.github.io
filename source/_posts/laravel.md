---
title: laravel
date: 2017-05-24 15:04:07
tags:
 - PHP

categories:
 - PHP
 - laravel

---

laravel 笔记
<!-- MORE -->

# homestead
这是个vagrant使用的盒子，其中包含开发使用的各种环境。
## 添加 yar 扩展
登陆到虚拟机后
```
sudo su
apt-get update
apt-get install -y libcurl3-dev

wget http://pecl.php.net/get/yar-2.0.2.tgz
tar -zxf yar-2.0.2.tgz
cd yar-2.0.2
phpize
./configure
make && make install

echo "extension=yar.so" >> /etc/php/7.1/fpm/php.ini
/etc/init.d/php7.1-fpm reload


yum install autoconf -y
cd /usr/local/serv/src
wget http://pecl.php.net/get/redis-3.1.2.tgz
tar -zxf redis-3.1.2.tgz
cd redis-3.1.2
phpize
./configure
make && make install

echo "extension=redis.so" >> /etc/php/7.1/fpm/php.ini
/etc/init.d/php7.1-fpm reload
```
