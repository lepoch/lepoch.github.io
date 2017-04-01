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

# 源码

<https://github.com/lepoch/worker>

# 注意点：

- 每个RUN所在目录是独立的，遇到make之类的，用 && 拼接多条命令
- supervisor启动nginx出现重复启动时，可以修改nginx.conf，加入daemon off;
- RUN语句，有缓存

# 减少镜像体积

- 编译安装之后，最好删除安装包
- yum update -y 之后，加上 yum clean all， 并且使用同一 RUN 命令
- 一个RUN就有一层大小，得减少 RUN 个数

<!-- ################################mysql################################## RUN yum install -y mysql mysql-devel RUN wget http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm RUN rpm -ivh mysql-community-release-el7-5.noarch.rpm RUN yum install -y mysql-community-server # 初始化 RUN mysql_install_db RUN mysqld --user=root & -->

 # 参考

<http://www.cnblogs.com/ivictor/p/4837750.html>
