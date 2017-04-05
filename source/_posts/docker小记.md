---
title: docker小记
date: 2017-04-5T04:05:00.000Z
tags:
  - docker
  - 笔记
categories:
  - Linux
---

docker笔记

<!-- MORE -->

 # 容器运行后就退出，如何才能一直运行？比如centos:7镜像

```
docker run -itd centos:7 bash
```

# 创建镜像

## 本地镜像

```
docker commit -m "xxx" -a "lepoch" a4170f6b81b3 lepoch/worker:0.1
```

-m 注释<br>
-a 作者<br>
0b2616b0e5a8 容器ID<br>
ouruser/sinatra:v2 镜像id:tag

## Dockerfile

源码见 [worker-dockerfile](/worker-dockerfile)  

```
docker build -t lepoch/worker:0.01 .
```


# 删除所有非运行的容器， rm -f 为强制删除

```
docker rm `docker ps -aq`
```

# 挂载宿主机目录，并授权
```
docker run -d -p 8001:80 -v /worker/www/:/usr/local/nginx/html --privileged=true lepoch/worker:0.01
```
