---
title: Flume小记
date: 2017-06-13 15:00:00
tags:
  - Hadoop
  - Flume
categories:
    - Hadoop
    - 小记
---

Flume小记
<!-- MORE -->
环境：腾讯云CentOS 7 x64  
目标：安装Flume 1.7


# 安装  
## java 1.8  
[Hadoop小记](Hadoop小记)

## Flume 1.7
```
# wget http://apache.fayea.com/flume/1.7.0/apache-flume-1.7.0-bin.tar.gz
wget http://10.210.0.8/apache-flume-1.7.0-bin.tar.gz
tar -xzf apache-flume-1.7.0-bin.tar.gz
mv apache-flume-1.7.0-bin /usr/local/

cd /usr/local/apache-flume-1.7.0-bin
mkdir etc
```
## 配置
### client使用exec source 

vim config/client.conf
```
# Name the components on this agent
client.sinks = k1
client.sources = r1
client.channels = c1



# Use a channel which buffers events in memory
client.channels.c1.type = memory
client.channels.c1.capacity = 1000
client.channels.c1.transactionCapacity = 100



client.sources.r1.type = TAILDIR
client.sources.r1.channels = c1
client.sources.r1.positionFile = /data/www/flume/taildir_position.json
client.sources.r1.filegroups = f1
client.sources.r1.filegroups.f1 = /data/www/htdocs/v3.maozhuar.com/storage/maozhua/testing/weight/log-.*
client.sources.r1.headers.f1.headerKey1 = value1
client.sources.r1.fileHeader = true



# Describe the sink
client.sinks.k1.type = avro
client.sinks.k1.channel = c1
client.sinks.k1.hostname = hadoop.maozhuar.com
client.sinks.k1.port = 4141

```

运行
```
bin/flume-ng agent --conf conf --conf-file conf/client_testing_dev.conf --name client -Dflume.root.logger=INFO,console
```

### 接收端使用  
使用hdfs，得先把hadoop中得jar包复制到flume/lib/中
hadoop/share/hadoop/common/*.jar
hadoop/share/hadoop/common/lib/*.jar
hadoop/share/hadoop/hdfs/hadoop-hdfs-*.jar

```
# Name the components on this agent
a1.sinks = k1
a1.sources = r1
a1.channels = c1

a1.sources.r1.type = avro
a1.sources.r1.channels = c1
a1.sources.r1.bind = 0.0.0.0
a1.sources.r1.port = 4141

a1.sinks.k1.type = hdfs
a1.sinks.k1.channel = c1
a1.sinks.k1.hdfs.path = hdfs://hadoop.maozhuar.com:9000/flume/events/%y-%m-%d/
a1.sinks.k1.hdfs.useLocalTimeStamp = true

a1.sinks.k1.hdfs.filePrefix=maozhua
a1.sinks.k1.hdfs.minBlockReplicas=1
a1.sinks.k1.hdfs.fileType=DataStream
a1.sinks.k1.hdfs.writeFormat=Text
a1.sinks.k1.hdfs.rollInterval=600
a1.sinks.k1.hdfs.rollSize=0
a1.sinks.k1.hdfs.rollCount=0
a1.sinks.k1.hdfs.idleTimeout=0

a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100
```
运行
```
bin/flume-ng agent --conf conf --conf-file conf/.conf --name a1 -Dflume.root.logger=INFO,console
```

#### 问题：hdfs文件乱码

解决：
```
a1.sinks.k1.hdfs.fileType=DataStream
```

#### 问题：Expected timestamp in the Flume event headers, but it was null  
解决1，每行开头有时间戳
解决2
```
a1.sinks.k1.hdfs.useLocalTimeStamp = true
```

#### 问题：没秒钟产生一个hdfs文件
解决
```
a1.sinks.k1.type = hdfs
a1.sinks.k1.channel = c1
a1.sinks.k1.hdfs.path = hdfs://hadoop.maozhuar.com:9000/flume/events/%y-%m-%d/
a1.sinks.k1.hdfs.useLocalTimeStamp = true

a1.sinks.k1.hdfs.filePrefix=maozhua
a1.sinks.k1.hdfs.minBlockReplicas=1
a1.sinks.k1.hdfs.fileType=DataStream
a1.sinks.k1.hdfs.writeFormat=Text
a1.sinks.k1.hdfs.rollInterval=600
a1.sinks.k1.hdfs.rollSize=0
a1.sinks.k1.hdfs.rollCount=0
a1.sinks.k1.hdfs.idleTimeout=0
```

# 参考
[【Flume】【源码分析】flume中sink到hdfs，文件系统频繁产生文件，文件滚动配置不起作用？](http://blog.csdn.net/simonchi/article/details/43231891)
