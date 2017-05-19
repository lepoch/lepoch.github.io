---
title: 代码review工具
date: 2017-04-14T15:00:00.000Z
tags:
  - 新浪游戏
  - php
categories:
  - php
---

上周审核了几个同事的代码，发现代码规范用处太小。使用工具进行代码的审核势在必行。

<!-- MORE -->

 # 环境

LINUX 虚拟机 CentOS7

# 命令行代理，见另一篇 《[Linux代理](/Linux代理/)》

# [Rietveld](http://code.google.com/p/rietveld/)

谷歌出品工具，很犀利，免费开源。PYTHON + Django， 使用Google App Engine。

- [git](https://github.com/rietveld-codereview/rietveld)
- [Google Cloud SDK](https://cloud.google.com/sdk/docs/)

## 安装

```
wget https://storage.googleapis.com/appengine-sdks/featured/google_appengine_1.9.50.zip
```

# [Phabricator](https://www.phacility.com/phabricator/)

facebook出品。基于LAMP。<br>
[官方安装手册](https://secure.phabricator.com/book/phabricator/article/installation_guide/)

## 环境依赖

- [Apache](https://httpd.apache.org/download.cgi)
- [MySQL](https://www.mysql.com/downloads/)
- [PHP](http://php.net/downloads.php)

## yum 安装

这种会出现 [proc_open 调用系统命令权限问题](#proc问题)

### 安装环境

```
yum install -y httpd
yum install -y httpd-manual mod_ssl mod_perl mod_auth_mysql


wget http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm
rpm -ivh mysql-community-release-el7-5.noarch.rpm
yum install -y mysql-community-server
yum install -y mysql mysql-devel

yum install -y php php-mysql
yum install -y gd php-gd gd-devel php-xml php-common php-mbstring php-ldap php-pear php-xmlrpc php-imap
```

### 安装组件

复制下来直接执行，执行路径即安装路径 [.sh脚本](https://secure.phabricator.com/source/phabricator/browse/master/scripts/install/install_rhel-derivs.sh)

## 配置，[Configration Guide](https://secure.phabricator.com/book/phabricator/article/configuration_guide/)

- Apache
- MySQL

## 使用

- 打开页面，创建管理员帐号
- 左上角的 Setup Issues，一个一个解决

### [Arcanist](https://secure.phabricator.com/book/phabricator/article/arcanist_quick_start/)

安装这个可以使用lint等，进行代码检测。

依赖

- Install PHP.
- Install Git.

安装

```
mkdir somewhere/
cd somewhere/
git clone https://github.com/phacility/libphutil.git
git clone https://github.com/phacility/arcanist.git
```

添加到PATH

```
export PATH="$PATH:/somewhere/arcanist/bin/"
```

####

### [Diffusion](https://secure.phabricator.com/book/phabricator/article/diffusion/)

这是绑定SVN、GIT等版本控制软件的工具。

### [提交后的审查audit](https://secure.phabricator.com/book/phabricator/article/audit/)

Post-commit code review and auditing. Audits you are assigned to will appear here.

这里有一节《Audits in Small Teams》，中有[Herald rule](https://secure.phabricator.com/book/phabricator/article/herald/)，在More Applications菜单中能找到。

## 问题

### Authentication Failure

> This Phabricator install is not configured with any enabled authentication providers which can be used to log in. If you have accidentally locked yourself out by disabling all providers, you can use `phabricator/bin/auth recover <username>` to recover access to an administrative account.

这是管理员被锁上了，使用提示中的命令恢复下就好。

### proc问题

查看diffusion，报错如下：

> Command failed with error #128! COMMAND git for-each-ref --sort='-creatordate' --format='%(objectname)%01%(objecttype)%01%(refname)%01%(_objectname)%01%(_objecttype)%01%(subject)%01%(creator)' -- 'refs/' STDOUT (empty) STDERR fatal: Not a git repository (or any of the parent directories): .git

还有svn --non-interactive --no-auth-cache --username什么什么的错误。

跟踪代码发现，都是proc执行系统命令出现的权限错误。<br>
而直接用 php cli 模式执行那些命令，则正常。。<br>
因时间原因，我直接换了 lnmp 。

### ini_set include_path不生效

> Unable to load libphutil. Put libphutil/ next to phabricator/, or update your PHP 'include_path' to include the parent directory of libphutil/.

发现是在fpm配置中写的，删除之后就行。

> php_admin_value[include_path] = .:/usr/local/sinasrv2/lib/php


# [SonarQube](http://www.sonarqube.org/)
用于检测代码质量的开源平台，通过插件可对PHP,Java,C/C++,JavaScript等20多种语言进行多纬度质量检测。

## 依赖
- Java

## 安装
官方手册：<https://docs.sonarqube.org/display/SONARQUBE56/Documentation/ >
本次安装的是5.6.6 LTS，
P.S. 6.3 我这连MySQL失败

### SonarQube server
web可视化管理工具， 去<http://www.sonarsource.org/downloads/ >下载，unzip 解压。

```
wget https://sonarsource.bintray.com/Distribution/sonarqube/sonarqube-5.6.6.zip
unzip sonarqube-5.6.6.zip
mv sonarqube-5.6.6 /etc/sonarqube
/etc/sonarqube/bin/[OS]/sonar.sh console
```

国内得修改谷歌字体，不然很卡
```
sed -i 's/\(@import url(https:\/\/fonts.googleapis.com\/css\)/\/\/\1/g' /etc/sonarqube/web/js/*
```

配置MySQL数据库，因为是虚拟机，就直接[yum安装MySQL](/yum安装MySQL)l了。
```
CREATE DATABASE sonar CHARACTER SET utf8 COLLATE utf8_general_ci;
CREATE USER 'sonar' IDENTIFIED BY 'sonar';
GRANT ALL ON sonar.* TO 'sonar'@'%' IDENTIFIED BY 'sonar';
GRANT ALL ON sonar.* TO 'sonar'@'localhost' IDENTIFIED BY 'sonar';
FLUSH PRIVILEGES;
```
配置MySQL
```
[root@localhost bin]# cat /etc/my.cnf
[mysqld]
max_allowed_packet=10M
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
user=mysql
# Disabling symbolic-links is recommend to prevent assorted security risks
symbolic-links=0
# 修改默认的存储引擎为InnoDB
default-storage-engine=InnoDB
# 这个参数主要作用是缓存innodb表的索引，数据，插入数据时的缓冲
innodb_buffer_pool_size = 256M
# 配置查询缓存的大小
query_cache_size=128M
# 启动mysql高速缓存
query_cache_type=1

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```

修改配置
vim /etc/sonarqube/conf/sonar.properties
```
sonar.jdbc.username=sonar
sonar.jdbc.password=sonar
sonar.jdbc.url=jdbc:mysql://localhost:3306/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance
```

### [SonarQube Scanner](https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner)
检测工具
```
wget https://sonarsource.bintray.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-3.0.1.733-linux.zip
unzip sonar-scanner-cli-3.0.1.733-linux.zip
mv sonar-scanner-3.0.1.733-linux /etc/sonar-scanner

ln -s /etc/sonar-scanner/bin/sonar-scanner /bin/sonar-scanner

```

### 安装插件 [SonarPHP](https://docs.sonarqube.org/display/PLUG/SonarPHP)

https://sonarsource.bintray.com/Distribution/sonar-php-plugin/sonar-php-plugin-2.10.0.2087.jar



# 参考资料

- <https://secure.phabricator.com/book/phabricator/article/installation_guide/>

- <https://www.oschina.net/news/53172/phabricator-manual-cn>
