---
title: 代码review工具
date: 2017-03-20T15:00:00.000Z
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

# 参考资料

- <https://secure.phabricator.com/book/phabricator/article/installation_guide/>

- <https://www.oschina.net/news/53172/phabricator-manual-cn>
