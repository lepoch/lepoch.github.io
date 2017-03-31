---
title: worker-dockerfile
date: 2017-03-31T18:48:00.000Z
tags:
  - docker
  - 环境
categories:
  - docker
---

用docker搭建开发环境

<!-- MORE -->

公司、家里多台电脑、多个系统之间的环境搭建实在麻烦，写个dockerfile。

# 环境

LINUX 虚拟机 CentOS7

```
# Version 0.1

# 基础镜像
FROM centos:7

# 维护者信息
MAINTAINER ll-xzx@qq.com

# 镜像操作命令
RUN yum update -y
RUN yum install -y wget gcc

###############################nginx 1.8.1################################
RUN wget http://nginx.org/download/nginx-1.8.1.tar.gz
RUN tar -zxf nginx-1.8.1.tar.gz

RUN useradd -d /usr/local/nginx/ -s /sbin/nologin www

##安装地址
RUN mkdir -p  /usr/local/nginx/
RUN mkdir -p  /usr/local/nginx/etc/
RUN mkdir -p  /usr/local/nginx/var/logs/
RUN mkdir -p  /usr/local/nginx/var/nginx/
RUN mkdir -p  /usr/local/nginx/var/run/
RUN mkdir -p  /usr/local/nginx/sbin/

RUN yum install -y pcre* openssl* zlib zlib-devel

RUN cd nginx-1.8.1 && \
./configure --prefix=/usr/local/nginx/ --sbin-path=/usr/local/nginx/sbin/ --conf-path=/usr/local/nginx/etc/nginx.conf --pid-path=/usr/local/nginx/var/logs/nginx/nginx.pid --error-log-path=/usr/local/nginx/var/logs/error.log --http-log-path=/usr/local/nginx/var/logs/access.log --user=www --group=www --with-http_ssl_module && \
make && make install

COPY nginx/etc /usr/local/nginx/etc/

################################supervisor##################################
RUN yum install python-setuptools -y
RUN easy_install supervisor
COPY supervisord.conf /etc/supervisord.conf

# 容器启动命令
CMD ["/usr/bin/supervisord"]

# 暴漏80端口
EXPOSE 80
```



```

################################mysql##################################

RUN yum install -y mysql mysql-devel
RUN wget http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm
RUN rpm -ivh mysql-community-release-el7-5.noarch.rpm
RUN yum install -y mysql-community-server

# 初始化
RUN mysql_install_db
RUN mysqld --user=root &





################################php 5.5.4################################################
mkdir -p /data/www/logs/
mkdir -p /data/www/userupload
mkdir -p /data/www/phpsession
mkdir -p /data/www/phpcache/
mkdir -p /usr/local/php

cd ~
wget http://cn.php.net/distributions/php-5.6.30.tar.gz
tar -zxf php-5.6.30.tar.gz


yum install -y pcre* openssl*  libxml2 libxml2-devel gdbm-devel gd-devel libjpeg libjpeg-devel libpng libpng-devel xmlrpc-c xmlrpc-c-devel freetype freetype-devel  readline-devel bzip2 bzip2-devel  libXpm-devel libX11-devel gmp-devel libxslt-devel

cd php-5.6.30 && \
./configure --prefix=/usr/local/php --exec-prefix=/usr/local/php --with-mysql --with-mysql-sock --with-mysqli --enable-fpm  --enable-soap --with-libxml-dir --with-openssl --with-mhash --with-pcre-regex --with-sqlite3 --with-zlib --enable-bcmath --with-iconv --with-bz2 --enable-calendar --with-curl --with-cdb --enable-dom --enable-exif --enable-fileinfo --enable-filter --with-pcre-dir --enable-ftp --with-gd --with-openssl-dir --with-jpeg-dir --with-png-dir --with-zlib-dir --with-freetype-dir --enable-gd-native-ttf --enable-gd-jis-conv --with-gettext --with-gmp --with-mhash --enable-json --enable-mbstring=all --disable-mbregex --disable-mbregex-backtrack --with-libmbfl --with-onig --enable-pdo --with-pdo-mysql --with-zlib-dir --with-pdo-sqlite --with-readline --enable-session --enable-shmop --enable-simplexml --enable-sockets  --enable-sysvmsg --enable-sysvsem --enable-sysvshm --enable-wddx --with-libxml-dir --with-xsl --enable-zip --enable-mysqlnd-compression-support --with-pear --enable-mbregex  && \
make && make install

ln -s /usr/local/php/bin/php* /usr/bin


#启动 fast cgi

/usr/local/php/sbin/php-fpm -y /usr/local/php/etc/web3/default/php-fpm.conf -c /usr/local/php/lib/php.ini -p /usr/local/php/etc/web3/default/

```


# 参考
http://www.cnblogs.com/ivictor/p/4837750.html
