---
title: 《Spark HA & Yarn 配置》
date: 2024-06-03 13:37:58
tags: Spark(HA)，Spark(Yarn)
---

# Spark StandAlone HA 环境搭建


## 步骤

> 前提: 确保Zookeeper 和 HDFS 均已经启动



先在`spark-env.sh`中, 删除: `SPARK_MASTER_HOST=node1`


原因: 配置文件中固定master是谁, 那么就无法用到zk的动态切换master功能了.


在`spark-env.sh`中, 增加:


```shell
SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=ZOOKEEPER -Dspark.deploy.zookeeper.url=node1:2181,node2:2181,node3:2181 -Dspark.deploy.zookeeper.dir=/spark-ha"
# spark.deploy.recoveryMode 指定HA模式 基于Zookeeper实现
# 指定Zookeeper的连接地址
# 指定在Zookeeper中注册临时节点的路径
```


将spark-env.sh 分发到每一台服务器上


```shell
scp spark-env.sh node2:/export/server/spark/conf/
scp spark-env.sh node3:/export/server/spark/conf/
```


停止当前StandAlone集群


```shell
sbin/stop-all.sh
```


启动集群:


```shell
# 在node1上 启动一个master 和全部worker
sbin/start-all.sh

# 注意, 下面命令在node2上执行
sbin/start-master.sh
# 在node2上启动一个备用的master进程
```

![](/2024/06/03/Spark3/14.jpg)
![](/2024/06/03/Spark3/15.jpg)

## master主备切换


提交一个spark任务到当前`alive`master上:


```shell
bin/spark-submit --master spark://node1:7077 /export/server/spark/examples/src/main/python/pi.py 1000
```


在提交成功后, 将alivemaster直接kill掉

不会影响程序运行:
![](/2024/06/03/Spark3/16.jpg)
当新的master接收集群后, 程序继续运行, 正常得到结果.


> 结论 HA模式下, 主备切换 不会影响到正在运行的程序.
>
> 最大的影响是 会让它中断大约30秒左右.





# Spark On YARN 环境搭建

## 部署

确保:


- HADOOP_CONF_DIR
- YARN_CONF_DIR



在spark-env.sh 以及 环境变量配置文件中即可
​

## 连接到YARN中


### bin/pyspark


```shell
bin/pyspark --master yarn --deploy-mode client|cluster
# --deploy-mode 选项是指定部署模式, 默认是 客户端模式
# client就是客户端模式
# cluster就是集群模式
# --deploy-mode 仅可以用在YARN模式下
```


> 注意: 交互式环境 pyspark  和 spark-shell  无法运行 cluster模式



### bin/spark-shell


```shell
bin/spark-shell --master yarn --deploy-mode client|cluster
```


> 注意: 交互式环境 pyspark  和 spark-shell  无法运行 cluster模式



### bin/spark-submit (PI)


```shell
bin/spark-submit --master yarn --deploy-mode client|cluster /xxx/xxx/xxx.py 参数
```


## spark-submit 和 spark-shell 和 pyspark的相关参数


参见: 附2
​



# 附1 Anaconda On Linux 安装 (单台服务器)


## 安装


上传安装包:


上传: 资料中提供的`Anaconda3-2021.05-Linux-x86_64.sh`文件到Linux服务器上


安装:

`sh ./Anaconda3-2021.05-Linux-x86_64.sh`
![](/2024/06/03/Spark3/17.jpg)
![](/2024/06/03/Spark3/18.jpg)
![](/2024/06/03/Spark3/19.jpg)
![](/2024/06/03/Spark3/20.jpg)
![](/2024/06/03/Spark3/21.jpg)
输入yes后就安装完成了.

安装完成后, `退出finalshell 重新进来`:
![](/2024/06/03/Spark3/22.jpg)


看到这个Base开头表明安装好了.


base是默认的虚拟环境.
​

## 国内源

如果你安装好后, 没有出现base, 可以打开:/root/.condarc这个文件, 追加如下内容:

```shell
channels:
  - defaults
show_channel_urls: true
default_channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
custom_channels:
  conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  msys2: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  bioconda: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  menpo: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  pytorch: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  simpleitk: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
```


# 附2 spark-submit和pyspark相关参数


客户端工具我们可以用的有:


- bin/pyspark: pyspark解释器spark环境
- bin/spark-shell: scala解释器spark环境
- bin/spark-submit: 提交jar包或Python文件执行的工具
- bin/spark-sql: sparksql客户端工具



这4个客户端工具的参数基本通用.


以spark-submit 为例:


`bin/spark-submit --master spark://node1:7077 xxx.py`


```shell
Usage: spark-submit [options] <app jar | python file | R file> [app arguments]
Usage: spark-submit --kill [submission ID] --master [spark://...]
Usage: spark-submit --status [submission ID] --master [spark://...]
Usage: spark-submit run-example [options] example-class [example args]

Options:
  --master MASTER_URL         spark://host:port, mesos://host:port, yarn,
                              k8s://https://host:port, or local (Default: local[*]).
  --deploy-mode DEPLOY_MODE   部署模式 client 或者 cluster 默认是client
  --class CLASS_NAME          运行java或者scala class(for Java / Scala apps).
  --name NAME                 程序的名字
  --jars JARS                 Comma-separated list of jars to include on the driver
                              and executor classpaths.
  --packages                  Comma-separated list of maven coordinates of jars to include
                              on the driver and executor classpaths. Will search the local
                              maven repo, then maven central and any additional remote
                              repositories given by --repositories. The format for the
                              coordinates should be groupId:artifactId:version.
  --exclude-packages          Comma-separated list of groupId:artifactId, to exclude while
                              resolving the dependencies provided in --packages to avoid
                              dependency conflicts.
  --repositories              Comma-separated list of additional remote repositories to
                              search for the maven coordinates given with --packages.
  --py-files PY_FILES         指定Python程序依赖的其它python文件
  --files FILES               Comma-separated list of files to be placed in the working
                              directory of each executor. File paths of these files
                              in executors can be accessed via SparkFiles.get(fileName).
  --archives ARCHIVES         Comma-separated list of archives to be extracted into the
                              working directory of each executor.

  --conf, -c PROP=VALUE       手动指定配置
  --properties-file FILE      Path to a file from which to load extra properties. If not
                              specified, this will look for conf/spark-defaults.conf.

  --driver-memory MEM         Driver的可用内存(Default: 1024M).
  --driver-java-options       Driver的一些Java选项
  --driver-library-path       Extra library path entries to pass to the driver.
  --driver-class-path         Extra class path entries to pass to the driver. Note that
                              jars added with --jars are automatically included in the
                              classpath.

  --executor-memory MEM       Executor的内存 (Default: 1G).

  --proxy-user NAME           User to impersonate when submitting the application.
                              This argument does not work with --principal / --keytab.

  --help, -h                  显示帮助文件
  --verbose, -v               Print additional debug output.
  --version,                  打印版本

 Cluster deploy mode only(集群模式专属):
  --driver-cores NUM          Driver可用的的CPU核数(Default: 1).

 Spark standalone or Mesos with cluster deploy mode only:
  --supervise                 如果给定, 可以尝试重启Driver

 Spark standalone, Mesos or K8s with cluster deploy mode only:
  --kill SUBMISSION_ID        指定程序ID kill
  --status SUBMISSION_ID      指定程序ID 查看运行状态

 Spark standalone, Mesos and Kubernetes only:
  --total-executor-cores NUM  整个任务可以给Executor多少个CPU核心用

 Spark standalone, YARN and Kubernetes only:
  --executor-cores NUM        单个Executor能使用多少CPU核心

 Spark on YARN and Kubernetes only(YARN模式下):
  --num-executors NUM         Executor应该开启几个
  --principal PRINCIPAL       Principal to be used to login to KDC.
  --keytab KEYTAB             The full path to the file that contains the keytab for the
                              principal specified above.

 Spark on YARN only:
  --queue QUEUE_NAME          指定运行的YARN队列(Default: "default").
```




# 附3 Windows系统配置Anaconda

## 安装

打开资料中提供的:Anaconda3-2021.05-Windows-x86_64.exe文件,或者去官网下载:[https://www.anaconda.com/products/individual#Downloads]
​

打开后,一直点击`Next`下一步即可:
![image.png](/2024/06/03/Spark3/23.jpg)
![image.png](/2024/06/03/Spark3/24.jpg)
如果想要修改安装路径, 可以修改
![image.png](/2024/06/03/Spark3/25.jpg)
不必勾选
![image.png](/2024/06/03/Spark3/26.jpg)
最终点击Finish完成安装

打开开始菜单, 搜索Anaconda
![image.png](/2024/06/03/Spark3/27.jpg)
出现如图的程序, 安装成功.

打开 `Anaconda Prompt`程序:
![image.png](/2024/06/03/Spark3/28.jpg)
出现`base`说明安装正确.



## 配置国内源

Anaconda默认源服务器在国外, 网速比较慢, 配置国内源加速网络下载.
​

打开上图中的 `Anaconda Prompt`程序:
执行:
`conda config --set show_channel_urls yes`
​

然后用记事本打开:
`C:\Users\用户名\.condarc`文件, 将如下内容替换进文件内,保存即可:

```shell
channels:
  - defaults
show_channel_urls: true
default_channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
custom_channels:
  conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  msys2: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  bioconda: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  menpo: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  pytorch: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  simpleitk: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
```


## 创建虚拟环境


```shell
# 创建虚拟环境 pyspark, 基于Python 3.8
conda activate base 
conda create -n pyspark python=3.8

# 切换到虚拟环境内
conda activate pyspark

# 在虚拟环境内安装包
pip install pyhive pyspark jieba -i https://pypi.tuna.tsinghua.edu.cn/simple 

pip install pyspark==3.2.0 -i https://pypi.tuna.tsinghua.edu.cn/simple
```

