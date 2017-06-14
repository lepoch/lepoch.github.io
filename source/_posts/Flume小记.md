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

```
# Name the components on this agent
agent_foo.sinks = k1
agent_foo.sources = r1
agent_foo.channels = c1



# Use a channel which buffers events in memory
agent_foo.channels.c1.type = memory
agent_foo.channels.c1.capacity = 1000
agent_foo.channels.c1.transactionCapacity = 100



agent_foo.sources.r1.type = TAILDIR
agent_foo.sources.r1.channels = c1
agent_foo.sources.r1.positionFile = /tmp/var/log/flume/taildir_position.json
agent_foo.sources.r1.filegroups = f1 f2
agent_foo.sources.r1.filegroups.f1 = /tmp/var/log/test1/example.log
agent_foo.sources.r1.headers.f1.headerKey1 = value1
agent_foo.sources.r1.filegroups.f2 = /tmp/var/log/test2/.*log.*
agent_foo.sources.r1.headers.f2.headerKey1 = value2
agent_foo.sources.r1.headers.f2.headerKey2 = value2-2
agent_foo.sources.r1.fileHeader = true



# Describe the sink
agent_foo.sinks.k1.type = avro
agent_foo.sinks.k1.channel = c1
agent_foo.sinks.k1.hostname = hadoop.maozhuar.com
agent_foo.sinks.k1.port = 4141

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
a1.sinks.k1.hdfs.path = hdfs://hadoop.maozhuar.com:9000//flume/events/%y-%m-%d-%S
a1.sinks.k1.hdfs.fileType = DataStream
a1.sinks.k1.hdfs.round = true
a1.sinks.k1.hdfs.roundValue = 10
a1.sinks.k1.hdfs.roundUnit = minute
a1.sinks.k1.hdfs.useLocalTimeStamp = true

a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100
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
