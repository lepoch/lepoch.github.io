---
title: Hive小记
date: 2017-06-13 17:26:00
tags:
  - Hive
categories:
  - Hadoop
  - 小记
---

Hive小记
<!-- MORE -->
环境：腾讯云CentOS 7 x64  
目标：安装、使用Hadoop Hive


# 安装  
## 安装 OpenJDK 8
[Hadoop小记](Hadoop小记)

## 安装 Hive  
下载网址：[http://mirror.bit.edu.cn/apache/hive/]
```
wget http://10.210.0.8/apache-hive-2.1.1-bin.tar.gz
tar -xzf apache-hive-2.1.1-bin.tar.gz
mv apache-hive-2.1.1-bin /usr/local/


( cat <<EOF

export HIVE_HOME=/usr/local/apache-hive-2.1.1-bin
export PATH=$HIVE_HOME/bin:$PATH

EOF
) >> /etc/profile

source /etc/profile

```

## 设置 [metastore](https://cwiki.apache.org/confluence/display/Hive/AdminManual+MetastoreAdmin)

### conf/hive-site.xml
```
<configuration>
   <property>
      <name>javax.jdo.option.ConnectionURL</name>
      <value>jdbc:mysql://localhost/metastore?createDatabaseIfNotExist=true</value>
      <description>metadata is stored in a MySQL server</description>
   </property>
   <property>
      <name>javax.jdo.option.ConnectionDriverName</name>
      <value>com.mysql.jdbc.Driver</value>
      <description>MySQL JDBC driver class</description>
   </property>
   <property>
      <name>javax.jdo.option.ConnectionUserName</name>
      <value>hive</value>
      <description>user name for connecting to mysql server</description>
   </property>
   <property>
      <name>javax.jdo.option.ConnectionPassword</name>
      <value>hivepassword</value>
      <description>password for connecting to mysql server</description>
   </property>
</configuration>
```
### [jdbc](https://dev.mysql.com/downloads/connector/j/)
```
wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.42.zip
unzip -o mysql-connector-java-5.1.42.zip
mv mysql-connector-java-5.1.42/mysql-connector-java-5.1.42-bin.jar $HIVE_HOME/lib/
```

### 启动
问题：
```
Unable to instantiate org.apache.hadoop.hive.ql.metadata.SessionHiveMet
```

解决：
```
# 初始化
bin/schematool -dbType mysql -initSchema
```

# 使用
```
CREATE EXTERNAL TABLE MYTEST6 (d STRING) 
STORED AS TEXTFILE
LOCATION '/flume/events/17-06-14';

LOAD DATA INPATH '/flume/events/y/*' INTO TABLE MYTEST;

```

## 问题：递归
```
set hive.mapred.supports.subdirectories=true;
set mapreduce.input.fileinputformat.input.dir.recursive=true;
```

# 参考
- [官方文档](http://hadoop.apache.org/docs/r2.8.0/hadoop-project-dist/hadoop-common/SingleCluster.html#Execution)
- [Hadoop 伪分布式下更换默认hadoop.tmp.dir路径](http://blog.csdn.net/telnetor/article/details/6993336)
- [关于Hive Metastore模式杂谈](http://www.cognoschina.net/Article/122347)
- [搭建Hive所遇过的坑](http://www.jianshu.com/p/d9cb284f842d)
