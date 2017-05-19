---
title: yum安装MySQL
date: 2017-03-24 00:00:00
tags:
 - MySQL

categories:
 - MySQL


---

yum 安装 MySQL

<!-- MORE -->
环境：
CentOS7
VBox

```
wget http://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm
yum localinstall -y mysql57-community-release-el7-7.noarch.rpm
yum install -y mysql-community-server
systemctl start mysqld.service

# 密码：
grep 'password' /var/log/mysqld.log | awk '{print $NF}'
mysql -u root -p`grep 'password' /var/log/mysqld.log | awk '{print $NF}'`

# 重设密码
set global validate_password_policy=0;
set global validate_password_length=4;
set password for 'root'@'localhost'=password('root');
flush privileges;

```
