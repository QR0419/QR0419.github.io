---
title: Spark基础环境配置
date: 2024-06-03 12:56:35
tags: 基础环境，JDK，Hadoop，Zookeeper
---

# 一、基础环境：

## 1、主机名：

| 序号 | 主机名 | IP Address    |
| ---- | ------ | ------------- |
| 01   | master | 192.168.30.10 |
| 02   | slave1 | 192.168.30.11 |
| 03   | slave2 | 192.168.30.12 |

## 2、配置hosts：（注：三台服务器同时配置）

![img](/2024/06/03/Spark1/clip_image002.jpg)

测试：（每台服务器都测试）

![img](/2024/06/03/Spark1/clip_image004.jpg)

![img](/2024/06/03/Spark1/clip_image006.jpg)

![img](/2024/06/03/Spark1/clip_image008.jpg)

 

## 3、安装NTP：（master做NTP服务器，slave1、slave2同步master）

（1）matser： yum -y install ntp 

![img](/2024/06/03/Spark1/clip_image010.jpg)

systemctl enable ntpd && systemctl start ntpd 

![img](/2024/06/03/Spark1/clip_image012.jpg)

三台服务器需执行以上命令。

Server:

![img](/2024/06/03/Spark1/clip_image014.jpg)

Client:

![img](/2024/06/03/Spark1/clip_image016.jpg)

![img](/2024/06/03/Spark1/clip_image018.jpg)

检查：（master、slave1、slave2）

![img](/2024/06/03/Spark1/clip_image020.jpg)

![img](/2024/06/03/Spark1/clip_image022.jpg)

![img](/2024/06/03/Spark1/clip_image024.jpg)

 

## 4、配置SSH免密码

### （1）所有节点分别生成私有密钥

ssh-keygen -t dsa -P '' -f /root/.ssh/id_dsa

![img](/2024/06/03/Spark1/clip_image026.jpg)

![img](/2024/06/03/Spark1/clip_image028.jpg)

![img](/2024/06/03/Spark1/clip_image030.jpg)

### （2）将公钥文件复制为authorized_keys文件

![img](/2024/06/03/Spark1/clip_image032.jpg)

### （3）mater主机连接自己，做SSH内环测试

![img](/2024/06/03/Spark1/clip_image034.gif)

### （4）slave1、slave2节点将master公钥文件复制本机（注：密码验证）

![img](/2024/06/03/Spark1/clip_image036.jpg)

![img](/2024/06/03/Spark1/clip_image038.jpg)

### （5）将master节点公钥文件追加各个节点（slave1、2）authorized_keys文件

![img](/2024/06/03/Spark1/clip_image040.jpg)

![img](/2024/06/03/Spark1/clip_image042.jpg)

### （6）master测试登录slave1、slave2（首次，需要“yes”确认连接）

### （7）同样操作，将slave1、2的id_dsa.pub传到master，追加authorized.keys

![img](/2024/06/03/Spark1/clip_image044.jpg)

## 5、安装、配置JDK

（1）mkdir -pv /usr/java 

（2）tar -zxvf jdk-8u171-linux-x64.tar.gz -C /usr/java/

![img](/2024/06/03/Spark1/clip_image046.jpg)

（3）vi /etc/profile

填写以下内容：

export JAVA_HOME=/usr/java/jdk1.8.0_171

export CLASSPATH=#JAVA_HOME/lib/

export PATH=$PATH:$JAVA_HOME/bin

export PATH JAVA_HOME CLASSPATH 

![img](/2024/06/03/Spark1/clip_image048.jpg)

生效环境变量，并查看jdk版本：

source /etc/profile 

java -version 

将master主机的jdk目录，分别拷贝slave1、slave2主机：

scp -r root@master:/usr/java  /usr/

将slave1、slave2配置jdk；并生效环境变量，查看jdk版本验证：

export JAVA_HOME=/usr/java/jdk1.8.0_171

export CLASSPATH=#JAVA_HOME/lib/

export PATH=$PATH:$JAVA_HOME/bin

export PATH JAVA_HOME CLASSPATH 

![img](/2024/06/03/Spark1/clip_image050.jpg)

# 二、zookeeper

## 1、修改主机名 与 IP地址映射设置（master、slave1、2都修改）

| vi /etc/hosts |        |              |
| ------------- | ------ | ------------ |
| 192.168.30.10 | master | master.root. |
| 192.168.30.11 | slave1 | slave1.root  |
| 192.168.30.12 | slave2 | slave2.root  |

 

![img](/2024/06/03/Spark1/clip_image052.jpg)

## 2、master节点执行操作：

### （1）创建zookeeper目录、解压解包；复制zoo.cfg：

![img](/2024/06/03/Spark1/clip_image054.jpg)

![img](/2024/06/03/Spark1/clip_image056.jpg)

或 egrep -v '(^#)' zoo_sample.cfg | tee -a zoo.cfg

### （2）配置zoo.cfg

![img](/2024/06/03/Spark1/clip_image058.jpg)

vi zoo.cfg 

tickTime=2000

initLimit=10

syncLimit=5

dataDir=/tmp/zookeeper

clientPort=2181

dataDir=/usr/zookeeper/zookeeper-3.4.10/zkdata

dataLogDir=/usr/zookeeper/zookeeper-3.4.10/zkdatalog

server.1=master:2888:3888

server.2=slave1:2888:3888

server.3=slave2:2888:3888

 

![555](/2024/06/03/Spark1/clip_image060.gif)

 

### （3）在zookeeper目录中，创建zkdata、zkdatalog两个目录

![img](/2024/06/03/Spark1/clip_image062.jpg)

### （4）进入zkdata目录，创建文件myid：

![img](/2024/06/03/Spark1/clip_image064.jpg)

### （5）将zookeeper目录分别slave1、slave2：

scp -r /usr/zookeeper/ root@slave1:/usr

![img](/2024/06/03/Spark1/clip_image066.jpg)

scp -r /usr/zookeeper/ root@slave2:/usr

![img](/2024/06/03/Spark1/clip_image068.jpg)

### （6）设置 myid

配置的 dataDir 指定的目录下面，创建一个 myid 文件，里面内容为一个数字，用来标识当前主机，conf/zoo.cfg 文件中配置的server.X 中 X 为什么数字，则 myid 文件中就输入这个数字。

slave1 中为 2

slave2 中为 3

![img](/2024/06/03/Spark1/clip_image070.jpg)

![img](/2024/06/03/Spark1/clip_image072.jpg)

### （7）配置环境变量，并启动zookeeper（注：master、slave1、slave2）：

export ZOOKEEPER_HOME=/usr/zookeeper/zookeeper-3.4.10

PATH=$PATH:$ZOOKEEPER_HOME/bin

![img](/2024/06/03/Spark1/clip_image074.jpg)

source /etc/profile

### （8）启动zookeeper集群

![img](/2024/06/03/Spark1/clip_image076.jpg)

# 三、安装Hadoop

## 1、创建/usr/hadoop目录、解压解包hadoop：（master节点操作）

![739a645a92f050b2f7621a0d2347516](/2024/06/03/Spark1/clip_image078.gif)

将hadoop-2.7.3.tar.gz 解压解包放置/usr/hadoop目录

tar -zxvf hadoop-2.7.3.tar.gz -C /usr/hadoop/

## 2、配置hadoop环境变量：（注：master、slave1、slave2均操作）

vi /etc/profile 

export HADOOP_HOME=/usr/hadoop/hadoop-2.7.3/

export CLASSPATH=$CLASSPATH:$HADOOP_HOME/lib

export PATH=$PATH:$HADOOP_HOME/bin

source /etc/profile 

![edea1819d194e4225e67db1d3242aca](/2024/06/03/Spark1/clip_image080.gif)

## 3、编辑hadoop环境配置文件hadoop-env.sh

路径：/usr/hadoop/hadoop-2.7.3/etc/hadoop

![d08312cfc2cba2d400f8a6d7ffffd7d](/2024/06/03/Spark1/clip_image082.gif)

输入内容：export JAVA_HOME=/usr/java/jdk1.8.0_171

## 3、编辑core-site.xml（注：配置文件在当前路径下）

填写以下配置文件：

<configuration>

​    <property>

​        <name>fs.default.name</name>

​        <value>hdfs://master:9000</value>

​    </property>

​    <property>

​        <name>hadoop.tmp.dir</name>

​        <value>/usr/hadoop/hadoop-2.7.3/hdfs/tmp</value>

​        <description>A base for other temporary directories.</description>

​    </property>

​     <property>

​        <name>io.file.buffer.size</name>

​        <value>131072</value>

​    </property>

​    <property>

​        <name>fs.checkpoint.period</name>

​        <value>60</value>

​    </property>

​    <property>

​        <name>fs.checkpoint.size</name>

​        <value>67108864</value>

​    </property>

</configuration>

 

![img](/2024/06/03/Spark1/clip_image084.gif)

## 4、编辑yarn-site.xml：（注：配置文件路径在当前路径下）

<configuration>

​    <property>

​        <name>yarn.resourcemanager.address</name>

​        <value>master:18040</value>

​    </property>

​    <property>

​        <name>yarn.resourcemanager.scheduler.address</name>

​        <value>master:18030</value>

​    </property>

​    <property>

​        <name>yarn.resourcemanager.webapp.address</name>

​        <value>master:18088</value>

​    </property>

​    <property>

​        <name>yarn.resourcemanager.resource-tracker.address</name>

​        <value>master:18025</value>

​    </property>

​    <property>

​        <name>yarn.resourcemanager.admin.address</name>

​        <value>master:18141</value>

​    </property>

​    <property>

​        <name>yarn.nodemanager.aux-services</name>

​        <value>mapreduce_shuffle</value>

​    </property>

​    <property>

​        <name>yarn.nodemanager.auxservices.mapreduce.shuffle.class</name>

​        <value>org.apache.hadoop.mapred.ShuffleHandler</value>

​    </property>

<!-- Site specific YARN configuration properties -->

</configuration>

![img](/2024/06/03/Spark1/clip_image086.gif)

## 5、编写slave文件与 master文件：

![img](/2024/06/03/Spark1/clip_image088.gif)

## 6、配置hdfs-site.xml

<configuration>

​    <property>

​        <name>dfs.replication</name>

​        <value>2</value>

​    </property>

​     <property>

​        <name>dfs.namenode.name.dir</name>

​        <value>file:/usr/hadoop/hadoop-2.7.3/hdfs/name</value>

​        <final>true</final>

​    </property>

​    <property>

​        <name>dfs.datanode.data.dir</name>

​        <value>file:/usr/hadoop/hadoop-2.7.3/hdfs/data</value>

​        <final>true</final>

​    </property>

​    <property>

​        <name>dfs.namenode.secondary.http-address</name>

​        <value>master:9001</value>

​    </property>

​     <property>

​        <name>dfs.webhdfs.enabled</name>

​        <value>true</value>

​    </property>

​     <property>

​        <name>dfs.permissions</name>

​        <value>false</value>

​    </property>

</configuration>

![img](/2024/06/03/Spark1/clip_image090.gif)

## 7、编辑mapred-site.xml

将模板mapred-site.xml.template 复制为 mapred-site.xml

 cp mapred-site.xml.template mapred-site.xml

 

编写mapred-site.xml 填写以下内容：

​    <property>

​        <name>mapreduce.framework.name</name>

​        <value>yarn</value>

​    </property>

![img](/2024/06/03/Spark1/clip_image092.gif)

## 8、向slave1、slave2分发Hadoop目录：

scp -r /usr/hadoop root@slave1:/usr/

scp -r /usr/hadoop root@slave2:/usr/

## 9、master节点中格式化hadoop

**执行： hdfs namenode -format**

![img](/2024/06/03/Spark1/clip_image094.gif)

访问master节点  master:50070 （注：50070 是hdfs的Web管理页面）

![b3ff08f9843dc8f48f97ca6c52a3ed2](/2024/06/03/Spark1/clip_image096.jpg)

![222](/2024/06/03/Spark1/clip_image098.gif)