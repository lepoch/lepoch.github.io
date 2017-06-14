---
title: Haddop小记
date: 2017-06-12 15:00:00
tags:
  - Hadoop
categories:
  - Hadoop
  - 小记
---

Haddop小记
<!-- MORE -->
环境：腾讯云CentOS 7 x64  
目标：安装Hadoop 2.8.0


# 安装  
## 安装 OpenJDK 8
下载地址：[http://www.oracle.com/technetwork/java/javase/downloads/index.html]
修改 vim /etc/profile

```
wget http://10.210.0.8/server-jre-8u131-linux-x64.tar

tar -xf server-jre-8u131-linux-x64.tar
cp -r jdk1.8.0_131 /usr/local/
```

修改/etc/profile
```
( cat <<EOF

export JAVA_HOME=/usr/local/jdk1.8.0_131
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
export PATH=$JAVA_HOME/bin:$PATH
EOF
) >> /etc/profile

source /etc/profile

```

## 依赖
```
yum install -y ssh rsync
```

## 安装hadoop 2.8.0
```
wget https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-2.8.0/hadoop-2.8.0.tar.gz
tar -zxf hadoop-2.8.0.tar.gz
mv hadoop-2.8.0 /usr/local/

mkdir input
cp etc/hadoop/*.xml input
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.8.0.jar grep input output 'dfs[a-z.]+'
cat output/*
```

## 使用
### 伪分布式的hdfs目录  
修改 etc/hadoop/core-site.xml，加入
```
<property>  
        <name>hadoop.tmp.dir</name>  
        <value>/data/hadoop/data</value>  
        <description>A base for other temporary directories.</description>  
</property>  
```
### 其他
见[官方文档](http://hadoop.apache.org/docs/r2.8.0/hadoop-project-dist/hadoop-common/SingleCluster.html#Execution)


# 参考
- [http://blog.csdn.net/telnetor/article/details/6993336]
