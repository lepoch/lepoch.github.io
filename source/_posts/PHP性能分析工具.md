---
title: PHP性能分析工具
date: 2017-06-28 17:26:00
tags:
  - PHP
  - 工具
categories:
  - PHP
---

PHP性能分析工具
tideways：
xhgui：用于显示xhprof数据的图形界面。
<!-- MORE -->
环境：腾讯云CentOS 7 x64  
php 7.1


## 安装
### mongodb  
vi /etc/yum.repos.d/mongodb-org-3.2.repo 
```
[mongodb-org-3.2]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.2/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.2.asc
```

安装
```
yum -y install mongodb-org
```

启动
```
mkdir -p /data/mongodb/db
mongod --dbpath=/data/mongodb/db --fork --logpath=/data/mongodb/mongodb.log
```

### xhgui
#### 前置：mongodb扩展
```
# ubuntu
# yum install libssl openssl-devel  -y
# ln -s /usr/lib/x86_64-linux-gnu/libssl.so  /usr/lib
# pecl install mongodb


git clone https://github.com/mongodb/mongo-php-driver.git
cd mongo-php-driver
git submodule sync && git submodule update --init
phpize
./configure
make all -j 5
make install

```
在 fpm/php.ini和cli/php.ini里打开扩展，如
```
echo "extension=mongodb.so" >> /usr/local/serv/php/etc/php.ini
echo "extension=mongodb.so" >> /usr/local/serv/php/lib/php.ini

```
#### php install.php
php7.1没有mcrypt扩展，php install.php执行时会可能会出现问题，其引用的require修改成“>”版本即可
>Problem 1
>    - Installation request for slim/slim 2.3.1 -> satisfiable by slim/slim[2.3.1].
>    - slim/slim 2.3.1 requires ext-mcrypt * -> the requested PHP extension mcrypt is missing from your system.


#### 安装
```
git clone https://github.com/maxincai/xhgui.git profiler.maozhuar.com
cd profiler.maozhuar.com


```
#### 索引
```
mongo
```
```
use xhprof
db.results.ensureIndex( { 'meta.SERVER.REQUEST_TIME' : -1 } )
db.results.ensureIndex( { 'profile.main().wt' : -1 } )
db.results.ensureIndex( { 'profile.main().mu' : -1 } )
db.results.ensureIndex( { 'profile.main().cpu' : -1 } )
db.results.ensureIndex( { 'meta.url' : 1 } )
```

#### nginx配置
```
server {
    server_name profiler.maozhuar.com;
    root /data/www/htdocs/profiler.maozhuar.com/webroot;

    location ~ \.php {
        fastcgi_pass   localhost:9002;
    }
    try_files $uri $uri/ /index.php?$query_string;
}
```

### tideways
> 见该安装地址，选择系统安装方式`https://tideways.io/profiler/docs/setup/installation`


#### 配置
```
echo "extension=tideways.so" >> /usr/local/serv/php/etc/php.ini
echo "tideways.sample_rate=25" >> /usr/local/serv/php/etc/php.ini
echo "auto_prepend_file=/data/www/htdocs/xhgui/external/header.php" >> /usr/local/serv/php/etc/php.ini
```



## 参考
- [PHP性能被动分析工具之xhgui加tideways的安装实践](https://segmentfault.com/a/1190000007580819)
- [CentOS 7 yum方式快速安装MongoDB](http://blog.csdn.net/leshami/article/details/53171375)
