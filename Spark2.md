---
title: Spark local& stand-alone 配置
date: 2024-06-03 13:31:31
tags: Spark(local)，Spark(stand-alone)
---

# Spark Local环境部署

## 下载地址

https://dlcdn.apache.org/spark/spark-3.2.0/spark-3.2.0-bin-hadoop3.2.tgz



## 条件


- PYTHON 推荐3.8
- JDK 1.8



## Anaconda On Linux 安装

本次课程的Python环境需要安装到Linux(虚拟机)和Windows(本机)上

参见最下方, 附1: Anaconda On Linux 安装



## 解压

解压下载的Spark安装包

`tar -zxvf spark-3.2.0-bin-hadoop3.2.tgz -C /export/server/`



## 环境变量


配置Spark由如下5个环境变量需要设置


-  SPARK_HOME: 表示Spark安装路径在哪里 
-  PYSPARK_PYTHON: 表示Spark想运行Python程序, 那么去哪里找python执行器 
-  JAVA_HOME: 告知Spark Java在哪里 
-  HADOOP_CONF_DIR: 告知Spark Hadoop的配置文件在哪里 
-  HADOOP_HOME: 告知Spark  Hadoop安装在哪里 



这5个环境变量 都需要配置在: `/etc/profile`中
​

![](/2024/06/03/Spark2/1.jpg)


PYSPARK_PYTHON和 JAVA_HOME 需要同样配置在: `/root/.bashrc`中


![](/2024/06/03/Spark2/2.jpg)

## 上传Spark安装包


资料中提供了: `spark-3.2.0-bin-hadoop3.2.tgz`


上传这个文件到Linux服务器中


将其解压, 课程中将其解压(安装)到: `/export/server`内.

`tar -zxvf spark-3.2.0-bin-hadoop3.2.tgz -C /export/server/`


由于spark目录名称很长, 给其一个软链接:

`ln -s /export/server/spark-3.2.0-bin-hadoop3.2 /export/server/spark`
​

![](/2024/06/03/Spark2/3.jpg)
![](/2024/06/03/Spark2/4.jpg)


## 测试


### bin/pyspark

bin/pyspark 程序, 可以提供一个  `交互式`的 Python解释器环境, 在这里面可以写普通python代码, 以及spark代码
​

![](/2024/06/03/Spark2/5.jpg)


如图:


![](/2024/06/03/Spark2/6.jpg)


在这个环境内, 可以运行spark代码


图中的: `parallelize` 和 `map` 都是spark提供的API

`sc.parallelize([1,2,3,4,5]).map(lambda x: x + 1).collect()`
​

### WEB UI (4040)


Spark程序在运行的时候, 会绑定到机器的`4040`端口上.

如果4040端口被占用, 会顺延到4041 ... 4042...
![](/2024/06/03/Spark2/7.jpg)


4040端口是一个WEBUI端口, 可以在浏览器内打开:

输入:`服务器ip:4040` 即可打开:
![](/2024/06/03/Spark2/8.jpg)


打开监控页面后, 可以发现 在程序内仅有一个Driver


因为我们是Local模式, Driver即管理 又 干活.

同时, 输入jps
​

![](/2024/06/03/Spark2/9.jpg)


可以看到local模式下的唯一进程存在


这个进程 即是master也是worker


### bin/spark-shell - 了解


同样是一个解释器环境, 和`bin/pyspark`不同的是, 这个解释器环境 运行的不是python代码, 而是scala程序代码


```shell
scala> sc.parallelize(Array(1,2,3,4,5)).map(x=> x + 1).collect()
res0: Array[Int] = Array(2, 3, 4, 5, 6)
```


> 这个仅作为了解即可, 因为这个是用于scala语言的解释器环境



### bin/spark-submit (PI)


作用: 提交指定的Spark代码到Spark环境中运行


使用方法:


```shell
# 语法
bin/spark-submit [可选的一些选项] jar包或者python代码的路径 [代码的参数]

# 示例
bin/spark-submit /export/server/spark/examples/src/main/python/pi.py 10
# 此案例 运行Spark官方所提供的示例代码 来计算圆周率值.  后面的10 是主函数接受的参数, 数字越高, 计算圆周率越准确.
```


对比

| 功能 | bin/spark-submit | bin/pyspark | bin/spark-shell |
| --- | --- | --- | --- |
| 功能 | 提交java\scala\python代码到spark中运行 | 提供一个`python`
解释器环境用来以python代码执行spark程序 | 提供一个`scala`
解释器环境用来以scala代码执行spark程序 |
| 特点 | 提交代码用 | 解释器环境 写一行执行一行 | 解释器环境 写一行执行一行 |
| 使用场景 | 正式场合, 正式提交spark程序运行 | 测试\学习\写一行执行一行\用来验证代码等 | 测试\学习\写一行执行一行\用来验证代码等 |



# Spark StandAlone环境部署

## 新角色 历史服务器


> 历史服务器不是Spark环境的必要组件, 是可选的.

> 回忆: 在YARN中 有一个历史服务器, 功能: 将YARN运行的程序的历史日志记录下来, 通过历史服务器方便用户查看程序运行的历史信息.



Spark的历史服务器, 功能: 将Spark运行的程序的历史日志记录下来, 通过历史服务器方便用户查看程序运行的历史信息.

搭建集群环境, 我们一般`推荐将历史服务器也配置上`, 方面以后查看历史记录
​

## 集群规划


课程中 使用三台Linux虚拟机来组成集群环境, 非别是:


node1\ node2\ node3


node1运行: Spark的Master进程  和 1个Worker进程


node2运行: spark的1个worker进程


node3运行: spark的1个worker进程


整个集群提供: 1个master进程 和 3个worker进程


## 安装


### 在所有机器安装Python(Anaconda)


参考 附1内容, 如何在Linux上安装anaconda


同时不要忘记 都创建`pyspark`虚拟环境 以及安装虚拟环境所需要的包`pyspark jieba pyhive`


### 在所有机器配置环境变量


参考 Local模式下 环境变量的配置内容


`确保3台都配置`


### 配置配置文件


进入到spark的配置文件目录中, `cd $SPARK_HOME/conf`


配置workers文件


```shell
# 改名, 去掉后面的.template后缀
mv workers.template workers

# 编辑worker文件
vim workers
# 将里面的localhost删除, 追加
node1
node2
node3
到workers文件内

# 功能: 这个文件就是指示了  当前SparkStandAlone环境下, 有哪些worker
```


配置spark-env.sh文件


```shell
# 1. 改名
mv spark-env.sh.template spark-env.sh

# 2. 编辑spark-env.sh, 在底部追加如下内容

## 设置JAVA安装目录
JAVA_HOME=/export/server/jdk

## HADOOP软件配置文件目录，读取HDFS上文件和运行YARN集群
HADOOP_CONF_DIR=/export/server/hadoop/etc/hadoop
YARN_CONF_DIR=/export/server/hadoop/etc/hadoop

## 指定spark老大Master的IP和提交任务的通信端口
# 告知Spark的master运行在哪个机器上
export SPARK_MASTER_HOST=node1
# 告知sparkmaster的通讯端口
export SPARK_MASTER_PORT=7077
# 告知spark master的 webui端口
SPARK_MASTER_WEBUI_PORT=8080

# worker cpu可用核数
SPARK_WORKER_CORES=1
# worker可用内存
SPARK_WORKER_MEMORY=1g
# worker的工作通讯地址
SPARK_WORKER_PORT=7078
# worker的 webui地址
SPARK_WORKER_WEBUI_PORT=8081

## 设置历史服务器
# 配置的意思是  将spark程序运行的历史日志 存到hdfs的/sparklog文件夹中
SPARK_HISTORY_OPTS="-Dspark.history.fs.logDirectory=hdfs://node1:8020/sparklog/ -Dspark.history.fs.cleaner.enabled=true"
```


注意, 上面的配置的路径 要根据你自己机器实际的路径来写


在HDFS上创建程序运行历史记录存放的文件夹:


```shell
hadoop fs -mkdir /sparklog
hadoop fs -chmod 777 /sparklog
```


配置spark-defaults.conf文件


```shell
# 1. 改名
mv spark-defaults.conf.template spark-defaults.conf

# 2. 修改内容, 追加如下内容
# 开启spark的日期记录功能
spark.eventLog.enabled 	true
# 设置spark日志记录的路径
spark.eventLog.dir	 hdfs://node1:8020/sparklog/ 
# 设置spark日志是否启动压缩
spark.eventLog.compress 	true
```


配置log4j.properties 文件 [可选配置]


```shell
# 1. 改名
mv log4j.properties.template log4j.properties

# 2. 修改内容 参考下图
```

![](/2024/06/03/Spark2/10.jpg)

> 这个文件的修改不是必须的,  为什么修改为WARN. 因为Spark是个话痨
>
> 会疯狂输出日志, 设置级别为WARN 只输出警告和错误日志, 不要输出一堆废话.



### 将Spark安装文件夹  分发到其它的服务器上


```shell
scp -r spark-3.1.2-bin-hadoop3.2 root@node2:/export/server/
scp -r spark-3.1.2-bin-hadoop3.2 root@node3:/export/server/
```


不要忘记, 在node2和node3上 给spark安装目录增加软链接

`ln -s /export/server/spark-3.2.0-bin-hadoop3.2 /export/server/spark`


### 检查


检查每台机器的:


JAVA_HOME


SPARK_HOME


PYSPARK_PYTHON


等等 环境变量是否正常指向正确的目录


### 启动历史服务器

`sbin/start-history-server.sh`


### 启动Spark的Master和Worker进程


```shell
# 启动全部master和worker
sbin/start-all.sh

# 或者可以一个个启动:
# 启动当前机器的master
sbin/start-master.sh
# 启动当前机器的worker
sbin/start-worker.sh

# 停止全部
sbin/stop-all.sh

# 停止当前机器的master
sbin/stop-master.sh

# 停止当前机器的worker
sbin/stop-worker.sh
```


### 查看Master的WEB UI


默认端口master我们设置到了8080


如果端口被占用, 会顺延到8081 ...;8082... 8083... 直到申请到端口为止


可以在日志中查看, 具体顺延到哪个端口上:


`Service 'MasterUI' could not bind on port 8080. Attempting port 8081.`
​



![](/2024/06/03/Spark2/11.jpg)


### 连接到StandAlone集群


#### bin/pyspark


执行:


```shell
bin/pyspark --master spark://node1:7077
# 通过--master选项来连接到 StandAlone集群
# 如果不写--master选项, 默认是local模式运行


sc.parallelize([1,2,3,4,5]).map(lambda x: x + 1).collect()
```

![](/2024/06/03/Spark2/12.jpg)


#### bin/spark-shell


```shell
bin/spark-shell --master spark://node1:7077
# 同样适用--master来连接到集群使用
```


```scala
// 测试代码
sc.parallelize(Array(1,2,3,4,5)).map(x=> x + 1).collect()
```


#### bin/spark-submit (PI)


```shell
bin/spark-submit --master spark://node1:7077 /export/server/spark/examples/src/main/python/pi.py 100
# 同样使用--master来指定将任务提交到集群运行
```


### 查看历史服务器WEB UI


历史服务器的默认端口是: 18080


我们启动在node1上, 可以在浏览器打开:

`node1:18080`来进入到历史服务器的WEB UI上.
![](/2024/06/03/Spark2/13.jpg)

zookeeper