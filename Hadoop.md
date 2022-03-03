# 1、Hadoop入门

版本信息：hadoop-3.1.0

## 1.1集群配置

NameNode 和 SecondaryNameNode 不要装在同一台服务器
ResourceManager也很耗内存不要和NameNode SecondaryNameNode配置在一台机器上

**配置文件说明**

core-site.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
 <!-- 指定 NameNode 的地址 -->
 <property>
 <name>fs.defaultFS</name>
 <value>hdfs://hadoop102:8020</value>
 </property>
 <!-- 指定 hadoop 数据的存储目录 -->
 <property>
 <name>hadoop.tmp.dir</name>
 <value>/opt/module/hadoop-3.1.3/data</value>
 </property>
 <!-- 配置 HDFS 网页登录使用的静态用户为 root -->
 <property>
 <name>hadoop.http.staticuser.user</name>
 <value>root</value>
 </property>
</configuration>
```

hdfs-site.xml 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
<!-- nn web 端访问地址-->
<property>
 <name>dfs.namenode.http-address</name>
 <value>hadoop102:9870</value>
 </property>
<!-- 2nn web 端访问地址-->
 <property>
 <name>dfs.namenode.secondary.http-address</name>
 <value>hadoop104:9868</value>
 </property>
</configuration>
```

yarn-site.xml

```sql
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
 <!-- 指定 MR 走 shuffle -->
 <property>
 <name>yarn.nodemanager.aux-services</name>
 <value>mapreduce_shuffle</value>
 </property>
 <!-- 指定 ResourceManager 的地址-->
 <property>
 <name>yarn.resourcemanager.hostname</name>
 <value>hadoop103</value>
 </property>
 <!-- 环境变量的继承 -->
 <property>
 <name>yarn.nodemanager.env-whitelist</name>
 
<value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CO
NF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAP
RED_HOME</value>
 </property>
</configuration>
```

apred-site.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
<!-- 指定 MapReduce 程序运行在 Yarn 上 -->
 <property>
 <name>mapreduce.framework.name</name>
 <value>yarn</value>
 </property>
</configuration>
```

**格式化**

第一次启动需要格式化 hdfs namenode -format

### **常用端口**

| 端口名称                       | Hadoop2.x | Hadoop3.x      |
| ------------------------------ | --------- | -------------- |
| NameNode 内存端口              | 8020/9000 | 8020/9000/9820 |
| HDFS NameNode 对用户的查询端口 | 50070     | 9870           |
| MapReduce Yarn任务端口         | 8088      | 8088           |
| 历史服务器                     | 19888     | 19888          |

### **常用配置文件**

​    2.x core-site.xml hdfs-site.xml yarn-site.xml mapred-site.xml workers
​        MapReduce 计算+资源调度
​        HDFS 数据存储
​        Common 辅助工具
​    3.x core-site.xml hdfs-site.xml yarn-site.xml mapred-site.xml salve
​        MapReduce 计算
​        Yarn 资源调度
​        HDFS 数据存储
​        Common 辅助工具

### 配置历史服务器

```xml
vim mapred-site.xml

<!-- 历史服务器端地址 -->
<property>
 <name>mapreduce.jobhistory.address</name>
 <value>hadoop102:10020</value>
</property>
<!-- 历史服务器 web 端地址 -->
<property>
 <name>mapreduce.jobhistory.webapp.address</name>
 <value>hadoop102:19888</value>
</property>
```

### 开启日志功能

```xml
vim yarn-site.xml
<!-- 开启日志聚集功能 -->
<property>
 <name>yarn.log-aggregation-enable</name>
 <value>true</value>
</property>
<!-- 设置日志聚集服务器地址 -->
<property> 
 <name>yarn.log.server.url</name> 
 <value>http://hadoop102:19888/jobhistory/logs</value>
</property>
<!-- 设置日志保留时间为 7 天 -->
<property>
 <name>yarn.log-aggregation.retain-seconds</name>
 <value>604800</value>
</property>
```

### 集群脚本

**myhadoop.sh**

```sh
#!/bin/bash
if [ $# -lt 1 ]
then
 echo "No Args Input..."
 exit ;
fi
case $1 in
"start")
 echo " =================== 启动 hadoop 集群 ==================="
 echo " --------------- 启动 hdfs ---------------"
 ssh hadoop102 "/opt/module/hadoop-3.1.3/sbin/start-dfs.sh"
 echo " --------------- 启动 yarn ---------------"
ssh hadoop103 "/opt/module/hadoop-3.1.3/sbin/start-yarn.sh"
 echo " --------------- 启动 historyserver ---------------"
 ssh hadoop102 "/opt/module/hadoop-3.1.3/bin/mapred --daemon start 
historyserver"
;;
"stop")
 echo " =================== 关闭 hadoop 集群 ==================="
 echo " --------------- 关闭 historyserver ---------------"
 ssh hadoop102 "/opt/module/hadoop-3.1.3/bin/mapred --daemon stop 
historyserver"
 echo " --------------- 关闭 yarn ---------------"
 ssh hadoop103 "/opt/module/hadoop-3.1.3/sbin/stop-yarn.sh"
 echo " --------------- 关闭 hdfs ---------------"
 ssh hadoop102 "/opt/module/hadoop-3.1.3/sbin/stop-dfs.sh"
;;
*)
 echo "Input Args Error..."
;;
esac
chmod +x myhadoop.sh 赋予脚本权限
```

**jpsall**

```sh
#!/bin/bash
for host in hadoop102 hadoop103 hadoop104
do
 echo =============== $host ===============
 ssh $host jps 
done
```

**xsync**

```sh
#1. 判断参数个数
if [ $# -lt 1 ]
then
 echo Not Enough Arguement!
 exit;
fi
#2. 遍历集群所有机器
for host in hadoop102 hadoop103 hadoop104
do
 echo ==================== $host ====================
 #3. 遍历所有目录，挨个发送
 for file in $@
 do
 #4. 判断文件是否存在
 if [ -e $file ]
 then
 #5. 获取父目录
 pdir=$(cd -P $(dirname $file); pwd)
 #6. 获取当前文件的名称
 fname=$(basename $file)
 ssh $host "mkdir -p $pdir"
 rsync -av $pdir/$fname $host:$pdir
 else
 echo $file does not exists!
 fi
 done
done
```

```sh
#1. 判断参数个数
if [ $# -lt 1 ]
then
 echo Not Enough Arguement!
 exit;
fi
#2. 遍历集群所有机器
for host in master slave1 slave2
do
 echo ==================== $host ====================
 #3. 遍历所有目录，挨个发送
 for file in $@
 do
 #4. 判断文件是否存在
 if [ -e $file ]
 then
 #5. 获取父目录
 pdir=$(cd -P $(dirname $file); pwd)
 #6. 获取当前文件的名称
 fname=$(basename $file)
 ssh $host "mkdir -p $pdir"
 rsync -av $pdir/$fname $host:$pdir
 else
 echo $file does not exists!
 fi
 done
done
```



### 集群时间同步

如果服务器在公网环境（能连接外网），可以不采用集群时间同步，因为服务器会定期 和公网时间进行校准；

如果服务器在内网环境，必须要配置集群时间同步，否则时间久了，会产生时间偏差， 导致集群执行任务时间不同步。

# 2、HDFS

## 2.1概述

HDFS（Hadoop Distributed File System），它是一个文件系统，用于存储文件，通过目 录树来定位文件；其次，它是分布式的，由很多服务器联合起来实现其功能，集群中的服务 器有各自的角色。

### 2.1.2 HDFS 优缺点

优点

- 高容错性，自动保存多个副本
- 适合处理大数据

缺点

- 不适合低延时数据访问
- 无法对高量小文件进行存储

**HDFS 组成架构**

- NameNode:就是Master
  - 管理HDFS的名称空间
  -  配置副本策略
  - 管理数据块(Block)映射信息
  - 处理客户端读写请求

- DataNode：salve
  - NameNode下达命令slave执行
  - 存储实际的数据块
  - 执行数据块的读/写操作

- Secondary NameNode:当NameNode挂掉不能替代NameNode
  - 辅助NameNode 分担工作量
  - 紧急恢复NameNode(部分)
- Client:客户端
  - 文件切分。文件上传HDFS的时候，Client将文件切分成一个一个的Block，然后进行上传；
  - 与NameNode交互，获取文件的位置信息；
  - 与DataNode交互，读取或者写入数据；
  - Client提供一些命令来管理HDFS，比如NameNode格式化；
  - Client可以通过一些命令来访问HDFS，比如对HDFS增删查改操作；

### 2.1.3HDFS 文件块大小（面试重点）

**HDFS文件块大小:**  **默认128M**

![hdfs块大小](E:\周钰航\笔记\img\hdfs块大小.png)

块的大小不能设置太小，也不能设置太大

- 太小，会增加寻址时间，程序一直在找块的开始位置；
- 太大，从磁盘传输数据的时间会明显大于定位这个块开 始位置所需的时间。导致程序在处理这块数据时，会非常慢。

**HDFS块的大小设置主要取决于磁盘传输速率**

## 2.2HDFS 的 Shell 操作（开发重点）

### 2.2.1基本语法

hadoop fs 具体命令 OR hdfs dfs 具体命令 两个是完全相同的。

跟Linux基本命令差不多，只是要加个-

#### 上传

1.moveFromLocal：从本地剪切粘贴到 HDFS

```
 hadoop fs -moveFromLocal ./shuguo.txt 
/sanguo
```

2.copyFromLocal：从本地文件系统中拷贝文件到 HDFS 路径去 等同于-put

```
 hadoop fs -copyFromLocal weiguo.txt 
/sanguo
```

3.-appendToFile：追加一个文件到已经存在的文件末尾

```
hadoop fs -appendToFile liubei.txt 
/sanguo/shuguo.txt
```

#### 下载

1.-copyToLocal==-get：从 HDFS 拷贝到本地

```
hadoop fs -copyToLocal 
/sanguo/shuguo.txt ./
```

## 2.3HDFS 的 API 操作

找到资料包路径下的 Windows 依赖文件夹，拷贝 hadoop-3.1.0 到非中文路径

配置 Path 环境变量。

**在 IDEA 中创建一个 Maven 工程 HdfsClientDemo，并导入相应的依赖坐标+日志添加**

```xml
<dependencies>
 <dependency>
 <groupId>org.apache.hadoop</groupId>
 <artifactId>hadoop-client</artifactId>
 <version>3.1.3</version>
 </dependency>
 <dependency>
 <groupId>junit</groupId>
 <artifactId>junit</artifactId>
 <version>4.12</version>
 </dependency>
 <dependency>
 <groupId>org.slf4j</groupId>
 <artifactId>slf4j-log4j12</artifactId>
 <version>1.7.30</version>
 </dependency>
</dependencies>

```

在项目的 src/main/resources 目录下，新建一个文件，命名为“log4j.properties”，在文件 中填入

```
log4j.rootLogger=INFO, stdout 
log4j.appender.stdout=org.apache.log4j.ConsoleAppender 
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout 
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n 
log4j.appender.logfile=org.apache.log4j.FileAppender 
log4j.appender.logfile.File=target/spring.log 
log4j.appender.logfile.layout=org.apache.log4j.PatternLayout 
log4j.appender.logfile.layout.ConversionPattern=%d %p [%c] - %m%n

```

客户端去操作 HDFS 时，是有一个用户身份的。默认情况下，HDFS 客户端 API 会从采 用 Windows 默认用户访问 HDFS，会报权限异常错误。所以在访问 HDFS 时，一定要配置 用户。

```
org.apache.hadoop.security.AccessControlException: Permission denied: 
user=56576, access=WRITE, 
inode="/xiyou/huaguoshan":atguigu:supergroup:drwxr-xr-x
```

**参数优先级**

（1）客户端代码中设置的值 >（2）ClassPath 下的用户自定义配置文 件 >（3）然后是服务器的自定义配置（xxx-site.xml）>（4）服务器的默认配置（xxx-default.xml）

## 2.4HDFS 的读写流程

![屏幕截图 2021-08-01 173439](E:\周钰航\笔记\img\屏幕截图 2021-08-01 173439.png)

（1）客户端通过 Distributed FileSystem 模块向 NameNode 请求上传文件，NameNode 检查目标文件是否已存在，父目录是否存在。（2）NameNode 返回是否可以上传。
（3）客户端请求第一个 Block 上传到哪几个 DataNode 服务器上。
（4）NameNode 返回 3 个 DataNode 节点，分别为 dn1、dn2、dn3。
（5）客户端通过 FSDataOutputStream 模块请求 dn1 上传数据，dn1 收到请求会继续调用dn2，然后 dn2 调用 dn3，将这个通信管道建立完成。
（6）dn1、dn2、dn3 逐级应答客户端。
（7）客户端开始往 dn1 上传第一个 Block（先从磁盘读取数据放到一个本地内存缓存），
			以 Packet 为单位，dn1 收到一个 Packet 就会传给 dn2，dn2 传给 dn3；dn1 每传一个 packet
			会放入一个应答队列等待应答。
（8）当一个 Block 传输完成之后，客户端再次请求 NameNode 上传第二个 Block 的服务器。（重复执行 3-7 步）。

## 2.5NNode 和 SNNode 工作机制

NameNode 中的元数据是存储在哪里的？

存放在内存中，在磁盘中备份元数据的 FsImage。

当在内存中的元数据更新时，如果同时更新 FsImage，就会导 致效率过低，但如果不更新，就会发生一致性问题，一旦 NameNode 节点断电，就会产生数 据丢失

引入 Edits 文件（只进行追加操作，效率很高）

每当元数据有更新或者添 加元数据时，修改内存中的元数据并追加到 Edits 中。这样，一旦 NameNode 节点断电，可 以通过 FsImage 和 Edits 的合并，合成元数据。

![NN工作机制](E:\周钰航\笔记\img\NN工作机制.png)

**第一阶段：NameNode 启动**

- 第一次启动 NameNode 格式化后，创建 Fsimage 和 Edits 文件。如果不是第一次启动，直接加载编辑日志和镜像文件到内存。

- 客户端对元数据进行增删改的请求。
- NameNode 记录操作日志，更新滚动日志。
- NameNode 在内存中对元数据进行增删改。

**第二阶段：Secondary NameNode 工作**

（1）Secondary NameNode 询问 NameNode 是否需要 CheckPoint。直接带回 NameNode 是否检查结果。

（2）Secondary NameNode 请求执行 CheckPoint。 

（3）NameNode 滚动正在写的 Edits 日志。

（4）将滚动前的编辑日志和镜像文件拷贝到 Secondary NameNode。 

（5）Secondary NameNode 加载编辑日志和镜像文件到内存，并合并。 

（6）生成新的镜像文件 fsimage.chkpoint。 

（7）拷贝 fsimage.chkpoint 到 NameNode。 

（8）NameNode 将 fsimage.chkpoint 重新命名成 fsimage。

### CheckPoint 时间设置

- **通常情况下，SecondaryNameNode 每隔一小时执行一次。**

[hdfs-default.xml]

```xml
<property>
 <name>dfs.namenode.checkpoint.period</name>
 <value>3600s</value>
</property>
```

- **一分钟检查一次操作次数，当操作次数达到 1 百万时，SecondaryNameNode 执行一次**。

```xml
<property>
<name>dfs.namenode.checkpoint.txns</name>
 <value>1000000</value>
<description>操作动作次数</description>
</property>
<property>
 <name>dfs.namenode.checkpoint.check.period</name>
 <value>60s</value>
<description> 1 分钟检查一次操作次数</description>
</property>
```

## 2.6DataNode

#### DataNode 工作机制

![DataNode](E:\周钰航\笔记\img\DataNode.png)

（1）一个数据块在 DataNode 上以文件形式存储在磁盘上，包括两个文件，一个是数据 本身，一个是元数据包括数据块的长度，块数据的校验和，以及时间戳。 

（2）DataNode 启动后向 NameNode 注册，通过后，周期性（6 小时）的向 NameNode 上 报所有的块信息。

- DN 向 NN 汇报当前解读信息的时间间隔，默认 6 小时；

```xml
<property>
<name>dfs.blockreport.intervalMsec</name>
<value>21600000</value>
<description>Determines block reporting interval in 
milliseconds.</description>
</property>
```

DATanode 每隔6小时向NameNode上传块信息是否正常 每隔3秒回复Datanode运行情况 超过10min30s不汇报就认为Datanode挂掉
数据校验 crc32
掉线时参
单节点启动 **hdfs --daemon start datanode**

#### 数据完整性

（1）当 DataNode 读取 Block 的时候，它会计算 CheckSum。 

（2）如果计算后的 CheckSum，与 Block 创建时值不一样，说明 Block 已经损坏。 

（3）Client 读取其他 DataNode 上的 Block。 

（4）常见的校验算法 crc（32），md5（128），sha1（160） 

（5）DataNode 在其文件创建后周期验证 CheckSum。

![数据校验](E:\周钰航\笔记\img\数据校验.png)

#### 掉线时限参数设置

TimeOut = 2 * dfs.namenode.heartbeat.recheck-interval + 10 * dfs.heartbeat.interval。 

而默认的dfs.namenode.heartbeat.recheck-interval 大小为5分钟，dfs.heartbeat.interval默认为3秒。

需要注意的是 hdfs-site.xml 配置文件中的 heartbeat.recheck.interval 的单位为毫秒， dfs.heartbeat.interval 的单位为秒。

```xml
<property>
 <name>dfs.namenode.heartbeat.recheck-interval</name>
 <value>300000</value>
</property>
<property>
 <name>dfs.heartbeat.interval</name>
 <value>3</value>
</property>
```

# 3、MapReduce

## 3.1MapReduce 概述

MapReduce 定义:自己处理业务相关代码 + 自身默认代码

MapReduce 是一个分布式运算程序的编程框架，是用户开发“基于 Hadoop 的数据分析应用”的核心框架。
MapReduce 核心功能是将用户编写的业务逻辑代码和自带默认组件整合成一个完整的分布式运算程序，并发运行在一个 Hadoop 集群上。

**优点:**
易于编程
良好扩展性：可以动态增加服务器，解决计算资源不够的问题
高容错性：任何一台机器挂掉，可以将任务转移到其他节点
适合海量数据计算(TB/PB)几千台服务器共同计算
**缺点:**
不擅长实时计算
不擅长流式计算
不擅长DAG有向无环图计算(相当于一个一个节点计算迭代运算)

**MapReduce 核心思想**: Map分任务 Reduce进行统计
        MrAppMaster:负责整个程序的过程调度及状态协调
        MapTask:负责Map阶段的整个数据流处理流程
        ReduceTask:负Reduce阶段的整个数据处理流程

| Java 类型 | Hadoop Writable 类型 |
| :-------: | :------------------: |
|  Boolean  |   BooleanWritable    |
|   Byte    |     ByteWritable     |
|    int    |     IntWritable      |
|   Float   |    FloatWritable     |
|  Double   |    DoubleWritable    |
|   Long    |     LongWritable     |
|  String   |         Text         |
|    Map    |     MapWritable      |
|   Array   |    ArrayWritable     |
|   Null    |     NullWritable     |

## 3.2MapReduce 编程规范

用户编写的程序分成三个部分：**Mapper、Reducer 和 Driver。**

**1．Mapper阶段**

（1）用户自定义的Mapper要继承自己的父类
（2）Mapper的输入数据是KV对的形式（KV的类型可自定义）
（3）Mapper中的业务逻辑写在map()方法中
（4）Mapper的输出数据是KV对的形式（KV的类型可自定义）
（5）map()方法（MapTask进程）对每一个<K,V>调用一次

**2．Reducer阶段**
（2）Reducer的输入数据类型对应Mapper的输出数据类型，也是KV
（3）Reducer的业务逻辑写在reduce()方法中
（4）ReduceTask进程对每一组相同k的<k,v>组调用一次reduce()方法
 **3．Driver阶段**
   相当于YARN集群的客户端，用于提交我们整个程序到YARN集群，提交的是
   封装了MapReduce程序相关运行参数的job对象

```java
//1 获取 job 对象
 Configuration conf = new Configuration();
 Job job = Job.getInstance(conf);
 //2 关联本 Driver 类
job.setJarByClass(FlowDriver.class);
 //3 关联 Mapper 和 Reducer
 job.setMapperClass(FlowMapper.class);
 job.setReducerClass(FlowReducer.class);
 
//4 设置 Map 端输出 KV 类型
 job.setMapOutputKeyClass(Text.class);
 job.setMapOutputValueClass(FlowBean.class);
 
//5 设置程序最终输出的 KV 类型
 job.setOutputKeyClass(Text.class);
 job.setOutputValueClass(FlowBean.class);
 
//6 设置程序的输入输出路径
 FileInputFormat.setInputPaths(job, new Path("D:\\inputflow"));
 FileOutputFormat.setOutputPath(job, new Path("D:\\flowoutput"));
 
//7 提交 Job
 boolean b = job.waitForCompletion(true);
 System.exit(b ? 0 : 1);
```



## 3.3hadoop序列化

从一台虚拟机内存传输到另一台虚拟机 没有办法直接传
首先先序列化:将内存数据转为字节码 再将字节码传输
自定义 bean 对象实现序列化接口:
	在企业开发中往往常用的基本序列化类型不能满足所有需求，比如在 Hadoop 框架内部
	传递一个 bean 对象，那么该对象就需要实现序列化接口。
	（1）必须实现 **Writable** 接口
	（2）反序列化时，需要反射调用空参构造函数，所以必须有空参构造
	（3）重写序列化与反序列化方法 且顺序必须相同 先写入的先反序列化
	（6）要想把结果显示在文件中，需要重写 toString()，可用"\t"分开，方便后续用。
	（7）如果需要将自定义的 bean 放在 key 中传输，则还需要实现 Comparable 接口，因为
				 MapReduce 框中的 Shuffle 过程要求对 key 必须能排序。

**Maven打包依赖**

```xml
<build>
 <plugins>
 <plugin>
 <artifactId>maven-compiler-plugin</artifactId>
 尚硅谷大数据技术之 Hadoop（MapReduce） 
—————————————————————————————
更多 Java –大数据 –前端 –python 人工智能资料下载，可百度访问：尚硅谷官网
 <version>3.6.1</version>
 <configuration>
 <source>1.8</source>
 <target>1.8</target>
 </configuration>
 </plugin>
 <plugin>
 <artifactId>maven-assembly-plugin</artifactId>
 <configuration>
 <descriptorRefs>
 <descriptorRef>jar-with-dependencies</descriptorRef>
 </descriptorRefs>
 </configuration>
 <executions>
 <execution>
 <id>make-assembly</id>
 <phase>package</phase>
 <goals>
 <goal>single</goal>
 </goals>
 </execution>
 </executions>
 </plugin>
 </plugins>
</build>
```

## 3.4MapReduce 框架原理

![MapReduce 框架原理](E:\周钰航\笔记\img\MapReduce 框架原理.png)

### InputFormat 数据输入

**切片与 MapTask 并行度决定机制**

1G 的数据，启动 8 个 MapTask，可以提高集群的并发处理能力。那么 1K 的数 据，也启动 8 个 MapTask，会提高集群性能吗？MapTask 并行任务是否越多越好呢？哪些因 素影响了 MapTask 并行度？

**数据块：**Block 是 HDFS 物理上把数据分成一块一块。数据块是 HDFS 存储数据单位。

**数据切片：**数据切片只是在逻辑上对输入进行分片，并不会在磁盘上将其切分成片进行 存储。数据切片是 MapReduce 程序计算输入数据的单位，<u>**一个切片会对应启动一个 MapTask**</u>

![切片0](E:\周钰航\笔记\img\切片0.png)

**Job 提交流程源码和切片源码详解**(以后看)

```java
waitForCompletion()
submit();
// 1 建立连接
connect();
// 1）创建提交 Job 的代理
new Cluster(getConfiguration());
// （1）判断是本地运行环境还是 yarn 集群运行环境
initialize(jobTrackAddr, conf); 
// 2 提交 job
submitter.submitJobInternal(Job.this, cluster)
// 1）创建给集群提交数据的 Stag 路径
Path jobStagingArea = JobSubmissionFiles.getStagingDir(cluster, conf);
// 2）获取 jobid ，并创建 Job 路径
JobID jobId = submitClient.getNewJobID();
// 3）拷贝 jar 包到集群
copyAndConfigureFiles(job, submitJobDir);
rUploader.uploadFiles(job, jobSubmitDir);
// 4）计算切片，生成切片规划文件
writeSplits(job, submitJobDir);
maps = writeNewSplits(job, jobSubmitDir);
input.getSplits(job);
// 5）向 Stag 路径写 XML 配置文件
writeConf(conf, submitJobFile);
conf.writeXml(out);
// 6）提交 Job,返回提交状态
status = submitClient.submitJob(jobId, submitJobDir.toString(),job.getCredentials());

```

![job](E:\周钰航\笔记\img\job.png)

### FileInputFormat 切片源码解析（input.getSplits(job)）

（1）程序先找到你数据存储的目录。
（2）开始遍历处理（规划切片）目录下的每一个文件
（3）遍历第一个文件ss.txt
 a）获取文件大小fs.sizeOf(ss.txt)
 b）计算切片大小
 computeSplitSize(Math.max(minSize 1,Math.min(maxSize ,blocksize)))=blocksize=128M = Long.MAXValue
 c）默认情况下，切片大小=blocksize 32M 128M
d）开始切，形成第1个切片：ss.txt—0:128M 第2个切片ss.txt—128:256M 第3个切片ss.txt—256M:300M
（每次切片时，都要判断切完剩下的部分是否大于块的1.1倍，不大于1.1倍就划分一块切片）
 e）将切片信息写到一个切片规划文件中
 f）整个切片的核心过程在getSplit()方法中完成
g）InputSplit只记录了切片的元数据信息，比如起始位置、长度以及所在的节点列表等。
（4）提交切片规划文件到YARN上，YARN上的MrAppMaster就可以根据切片规划文件计算开启MapTask个数。
                FileInputFormat 常见的接口实现类包括：TextInputFormat、KeyValueTextInputFormat、
                NLineInputFormat、CombineTextInputFormat 和自定义 InputFormat 等。 TextInputFormat 是默认的 FileInputFormat 实现类。按行读取每条记录
                CombineTextInputFormat 切片机制
                CombineTextInputFormat.setMaxInputSplitSize(job, 4194304);// 4m

### **TextInputFormat FileInputFormat 实现类**

在运行 MapReduce 程序时，输入的文件格式包括：基于行的日志文件、二进制 格式文件、数据库表等。那么，针对不同的数据类型，MapReduce 是如何读取这些数据的呢？

 FileInputFormat 常见的接口实现类包括：TextInputFormat、KeyValueTextInputFormat、 NLineInputFormat、CombineTextInputFormat 和自定义 InputFormat 等。

#### **TextInputFormat**(每个文件均是一个切片)

TextInputFormat 是默认的 FileInputFormat 实现类。按行读取每条记录。键是存储该行在整个文件中的起始字节偏移量， LongWritable 类型。值是这行的内容，不包括任何行终止 符（换行符和回车符），Text 类型。

#### **CombineTextInputFormat 切片机制**

框架默认的 TextInputFormat 切片机制是对任务按文件规划切片，不管文件多小，都会 是一个单独的切片，都会交给一个 MapTask，这样如果有大量小文件，就会产生大量的 MapTask，处理效率极其低下。

**应用场景：**

CombineTextInputFormat 用于小文件过多的场景，它可以将多个小文件从逻辑上规划到 一个切片中，这样，多个小文件就可以交给一个 MapTask 处理。

**虚拟存储切片最大值设置**

CombineTextInputFormat.setMaxInputSplitSize(job, 4194304);// 4m 

注意：虚拟存储切片最大值设置最好根据实际的小文件大小情况来设置具体的值。![CombineTextInputFormat](E:\周钰航\笔记\img\CombineTextInputFormat.png)

（1）虚拟存储过程： 将输入目录下所有文件大小，依次和设置的 setMaxInputSplitSize 值比较，如果不 大于设置的最大值，逻辑上划分一个块。如果输入文件大于设置的最大值且大于两倍， 那么以最大值切割一块；当剩余数据大小超过设置的最大值且不大于最大值 2 倍，此时 将文件均分成 2 个虚拟存储块（防止出现太小切片）。

 例如 setMaxInputSplitSize 值为 4M，输入文件大小为 8.02M，则先逻辑上分成一个 4M。剩余的大小为 4.02M，如果按照 4M 逻辑划分，就会出现 0.02M 的小的虚拟存储 文件，所以将剩余的 4.02M 文件切分成（2.01M 和 2.01M）两个文件。 

（2）切片过程： 

（a）判断虚拟存储的文件大小是否大于 setMaxInputSplitSize 值，大于等于则单独 形成一个切片。 （b）如果不大于则跟下一个虚拟存储文件进行合并，共同形成一个切片。

```java
// 如果不设置 InputFormat，它默认用的是 TextInputFormat.class
job.setInputFormatClass(CombineTextInputFormat.class);
//虚拟存储切片最大值设置 20m
CombineTextInputFormat.setMaxInputSplitSize(job, 20971520);
```

## MapReduce 工作流程

![MapReduce 工作流程1](E:\周钰航\笔记\img\MapReduce 工作流程1.png)

![MapReduce 工作流程2](E:\周钰航\笔记\img\MapReduce 工作流程2.png)

**注意：** 

（1）Shuffle 中的缓冲区大小会影响到 MapReduce 程序的执行效率，原则上说，缓冲区 越大，磁盘 io 的次数越少，执行速度就越快。 

（2）缓冲区的大小可以通过参数调整，参数：mapreduce.task.io.sort.mb 默认 100M。

##  Shuffle 机制

Map 方法之后，Reduce 方法之前的数据处理过程称之为 Shuffle。

![shuffle](E:\周钰航\笔记\img\shuffle.png)

先标记分区进入环形缓冲区(默认100M) 数据到达80%溢写 溢出之前先排序(快排) 对索引排序
溢写产生两个文件 1.索引 2.数据
Combiner为可选流程会聚合数据<at,(1,1)> --->分区合并 -->压缩写磁盘 --等待Reduce拉去
拉去后先尝试放内存 内存不够放磁盘 --> 排序分组

## Partition 分区

setNumReduceTasks(n); 设置Reduce数量

```java
public class HashPartitioner<K, V> extends Partitioner<K, V> {
                public int getPartition(K key, V value, int numReduceTasks) {
                return (key.hashCode() & Integer.MAX_VALUE) % numReduceTasks;
```

默认分区是根据key的hashCode对ReduceTasks个数取模得到的。用户没法控制哪个 key存储到哪个分区。

（1）自定义类继承Partitioner，重写getPartition()方法
（2）在Job驱动中，设置自定义Partitione job.setPartitionerClass(CustomPartitioner.class)
（3）自定义Partition后，要根据自定义Partitioner的逻辑设置相应数量的ReduceTask job.setNumReduceTasks(5);
如果设置NumTask个数多了 会产生多于空文件 浪费资源
少了的话会IO异常 有多少分区就有多少Reducer

### WritableComparable 排序

MapTask和ReduceTask均会对数据按 照key(key不能排序直接报错,默认字典排快排)进行排序。该操作属于
Hadoop的默认行为。任何应用程序中的数据均会被排序，而不管逻辑上是否需要。

默认排序是按照**字典顺序**排序，且实现该排序的方法是**快速排序。**

对于MApTask不会立马排序 当环形缓冲区阈值达到后才进行排序

对于ReduceTask，它从每个MapTask上远程拷贝相应的数据文件，如果文件大 小超过一定阈值，则溢写磁盘上，否则存储在内存中。如果磁盘上文件数目达到 一定阈值，则进行一次归并排序以生成一个更大文件；如果内存中文件大小或者 数目超过一定阈值，则进行一次合并后将数据溢写到磁盘上。当所有数据拷贝完 毕后，ReduceTask统一对内存和磁盘上的所有数据进行一次归并排序。

**1.部分排序(sort by)**
输出多个文件都会排序
 **2.全排序 (order by)**

输出结果只有一个文件 效率低 开发少用
**3.辅助排序：（GroupingComparator分组）**
在Reduce端对key进行分组。应用于：在接收的key为bean对象时，想让一个或几个字段相同（全部字段比较不相同）的key进入到同一个reduce方法时，可以采用分组排序。
**4.二次排序**
在自定义排序过程中，如果compareTo中的判断条件为两个即为二次排序。

**自定义排序 WritableComparable** 

原理分析 bean 对象做为 key 传输，需要实现 WritableComparable 接口重写 compareTo 方法，就可 以实现排序。

**Combiner 合并**

前提是不影响最终数据 map端运行解决数据倾斜

（1）Combiner是MR程序中Mapper和Reducer之外的一种组件。
（2）Combiner组件的父类就是Reducer。
（3）Combiner和Reducer的**区别在于运行的位置**
**Combiner是在每一个MapTask所在的节点运行**; Reducer是接收全局所有Mapper的输出结果；
（4）Combiner的意义就是对每一个MapTask的输出进行局部汇总，以减小网络传输量
（5）Combiner能够应用的前提是不能影响最终的业务逻辑，而且，Combiner的输出kv应该跟Reducer的输入kv类型要对应起来。

增加一个 WordCountCombiner 类继承 Reducer

```java
public class WordCountCombiner extends Reducer<Text, IntWritable, Text, 
IntWritable> {
private IntWritable outV = new IntWritable();
 @Override
 protected void reduce(Text key, Iterable<IntWritable> values, Context 
context) throws IOException, InterruptedException {
 int sum = 0;
 for (IntWritable value : values) {
 sum += value.get();
 }
 //封装 outKV
 outV.set(sum)
```

在 WordcountDriver 驱动类中指定 Combiner

```java
// 指定需要使用 combiner，以及用哪个类作为 combiner 的逻辑
job.setCombinerClass(WordCountCombiner.class);
```

## Reduce工作机制

![Reduce工作机制](E:\周钰航\笔记\img\Reduce工作机制.png)

（1）Copy 阶段：ReduceTask 从各个 MapTask 上远程拷贝一片数据，并针对某一片数 据，如果其大小超过一定阈值，则写到磁盘上，否则直接放到内存中。

2）Sort 阶段：在远程拷贝数据的同时，ReduceTask 启动了两个后台线程对内存和磁 盘上的文件进行合并，以防止内存使用过多或磁盘上文件过多。按照 MapReduce 语义，用 户编写 reduce()函数输入数据是按 key 进行聚集的一组数据。为了将 key 相同的数据聚在一 起，Hadoop 采用了基于排序的策略。由于各个 MapTask 已经实现对自己的处理结果进行了 局部排序，因此，ReduceTask 只需对所有数据进行一次归并排序即可。

（3）Reduce 阶段：reduce()函数将计算结果写到 HDFS 上。

### MapTask & ReduceTask 源码解析

## 数据清洗ETL

### **Map Join**

Map Join 适用于一张表十分小、一张表很大的场景。

提前处理业务逻辑，这样增加 Map 端业务，减少 Reduce 端数 据的压力，尽可能的减少数据倾斜。

-------------------------------------------------------------------------

在运行核心业务 MapReduce 程序之前，往往要先对数据进行清洗，清理掉不符合用户 要求的数据。清理的过程往往只需要运行 Mapper 程序，不需要运行 Reduce 程序。

**逻辑处理接口：**

Mapper  用户根据业务需求实现其中三个方法：map() setup() cleanup ()

- setup()，此方法被MapReduce框架仅且执行一次，在执行Map任务前，进行相关变量或者资源的集中初始化工作。若是将资源初始化工作放在方法map()中，导致Mapper任务在解析每一行输入时都会进行资源初始化工作，导致重复，程序运行效率不高！
- cleanup(),此方法被MapReduce框架仅且执行一次，在执行完毕Map任务后，进行相关变量或资源的释放工作。若是将释放资源工作放入方法map()中，也会导致Mapper任务在解析、处理每一行文本后释放资源，而且在下一行文本解析前还要重复初始化，导致反复重复，程序运行效率不高！

**Comparable 排序**

（1）当我们用自定义的对象作为 key 来输出时，就必须要实现 WritableComparable 接 口，重写其中的 compareTo()方法。 

（2）部分排序：对最终输出的每一个文件进行内部排序。 

（3）全排序：对所有数据进行排序，通常只有一个 Reduce。 

（4）二次排序：排序的条件有两个。

**逻辑处理接口**：

Reducer 用户根据业务需求实现其中三个方法：reduce() setup() cleanup () 

**输出数据接口：OutputFormat** 

（1）默认实现类是 TextOutputFormat，功能逻辑是：将每一个 KV 对，向目标文本文件 输出一行。

 （2）用户还可以自定义 OutputFormat。

# 4、Hadoop 数据压缩

## 概述

**压缩的好处和坏处**

压缩的优点：以减少磁盘 IO、减少磁盘存储空间。 

压缩的缺点：增加 CPU 开销。

压缩原则
（1）运算密集型的 Job，少用压缩
（2）IO 密集型的 Job，多用压缩

## MR 支持的压缩编码

| 压缩格式 | Hadoop 自带？ | 算法    | 文件扩展 名 | 是否可 切片 | 换成压缩格式后，原来的 程序是否需要修改 |
| -------- | ------------- | ------- | ----------- | ----------- | --------------------------------------- |
| DEFLATE  | 是            | DEFLATE | .deflate    | 否          | 和文本处理一样，不需要 修改             |
| Gzip     | 是            | DEFLATE | .gz         | 否          | 和文本处理一样，不需要 修改             |
| bzip2    | 是            | bzip2   | .bz2        | 是          | 和文本处理一样，不需要 修改             |
| LZO      | 否            | LZO     | .lzo        | 是          | 需要建索引，还需要指定 输入格式         |
| Snappy   | 是            | Snappy  | .snappy     | 否          | 和文本处理一样，不需要 修改             |

**压缩性能**

| 压缩算法 | 原始文件大小 | 压缩文件大小 | 压缩速度 | 解压速度 |
| -------- | ------------ | ------------ | -------- | -------- |
| gzip     | 8.3GB        | 1.8GB        | 17.5MB/s | 58MB/s   |
| bzip2    | 8.3GB        | 1.1GB        | 2.4MB/s  | 9.5MB/s  |
| LZO      | 8.3GB        | 2.9GB        | 49.3MB/s | 74.6MB/s |

**压缩方式选择**

 压缩方式选择时重点考虑：压缩/解压缩速度、压缩率（压缩后存储大小）、压缩后是否 可以支持切片。 

**Gzip 压缩** 

优点：压缩率比较高； 缺点：不支持 Split；压缩/解压速度一般； 

**Bzip2 压缩** 

优点：压缩率高；支持 Split； 缺点：压缩/解压速度慢。 

**Lzo 压缩** 

优点：压缩/解压速度比较快；支持 Split； 缺点：压缩率一般；想支持切片需要额外创建索引。

**Snappy 压缩** 

优点：压缩和解压缩速度快； 缺点：不支持 Split；压缩率一般； **压缩位置选择** 

压缩可以在 MapReduce 作用的任意阶段启用。

要在 Hadoop 中启用压缩，需要配置如下参数

![压缩](E:\周钰航\笔记\img\压缩.png)

# 5、YARN

YARN 主要由 ResourceManager、NodeManager、ApplicationMaster 和 Container 等组件构成

## Yarn 工作机制

**ResourceManager（RM**）主要作用如下
    （1）处理客户端请求
    （3）启动或监控ApplicationMaster
    （2）监控NodeManager
    （4）资源的分配与调度
 **NodeManager（NM）**主要作用如下
    （1）管理单个节点上的资源
    （2）处理来自ResourceManager的命令
    （3）处理来自ApplicationMaster的命令
**ApplicationMaster（AM）**作用如下
    （2）任务的监控与容错
    （1）为应用程序申请资源并分配内部的任务
**Container**:
        Container是Yarn中的资源抽象,它封装了某个节点上的多维度资源,如内存CPU磁盘网络

![Yarn工作机制](E:\周钰航\笔记\大数据\Hadoop\img\Yarn工作机制.png)

（1）MR 程序提交到客户端所在的节点。 

（2）YarnRunner 向 ResourceManager 申请一个 Application。 

（3）RM 将该应用程序的资源路径返回给 YarnRunner。

（4）该程序将运行所需资源提交到 HDFS 上。 

（5）程序资源提交完毕后，申请运行 mrAppMaster。 

（6）RM 将用户的请求初始化成一个 Task。 

（7）其中一个 NodeManager 领取到 Task 任务。 

（8）该 NodeManager 创建容器 Container，并产生 MRAppmaster

（9）Container 从 HDFS 上拷贝资源到本地。 

（10）MRAppmaster 向 RM 申请运行 MapTask 资源。 

（11）RM 将运行 MapTask 任务分配给另外两个 NodeManager，另两个 NodeManager 分 别领取任务并创建容器。 

（12）MR 向两个接收到任务的 NodeManager 发送程序启动脚本，这两个 NodeManager 分别启动 MapTask，MapTask 对数据分区排序。 

（13）MrAppMaster 等待所有 MapTask 运行完毕后，向 RM 申请容器，运行 ReduceTask。 

（14）ReduceTask 向 MapTask 获取相应分区的数据。 

（15）程序运行完毕后，MR 会向 RM 申请注销自己。

## Yarn 调度器和调度算法

目前，Hadoop 作业调度器主要有三种：FIFO、容量（Capacity Scheduler）和公平（Fair  Scheduler）。Apache Hadoop3.1.3 默认的资源调度器是 Capacity Scheduler。 CDH 框架默认调度器是 Fair Scheduler。 具体设置详见：yarn-default.xml 文件

```xml
<property>
 <description>The class to use as the resource scheduler.</description>
 <name>yarn.resourcemanager.scheduler.class</name>
<value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler</value>
</property>
```

###  先进先出调度器（FIFO）

![FiFO调度器](E:\周钰航\笔记\大数据\Hadoop\img\FiFO调度器.png)

优点：简单易懂； 

缺点：不支持多队列，生产环境很少使用；

### 容量调度器（Capacity Scheduler）

![2调度器](E:\周钰航\笔记\大数据\Hadoop\img\2调度器.png)

![容量调度器](E:\周钰航\笔记\大数据\Hadoop\img\容量调度器.png)

### 公平调度器（Fair Scheduler）

![公平调度器](E:\周钰航\笔记\大数据\Hadoop\img\公平调度器.png)

![缺额](E:\周钰航\笔记\大数据\Hadoop\img\缺额.png)

**公平调度器队列资源分配方式**

**FIFO策略**
公平调度器每个队列资源分配策略如果选择FIFO的话，此时公平调度器相当于上面讲过的容量调度器。
**Fair策略**
Fair 策略（默认）是一种基于最大最小公平算法实现的资源多路复用方式，默认情况下，每个队列内部采用该方式分配资源。这意味着，如果一个队列中有两个应用程序同时运行，则每个应用程序可得到1/2的资源；如果三个应用程序同时运行，则每个应用程序可得到1/3的资源。
具体资源分配流程和容量调度器一致；
（1）选择队列
（2）选择作业
（3）选择容器

**DRF策略**

DRF（Dominant Resource Fairness），我们之前说的资源，都是单一标准，例如只考虑内存（也是Yarn默认的情况）。但是很多时候我们资源有很多种，例如内存，CPU，网络带宽等，这样我们很难衡量两个应用应该分配的资源比例。

## Yarn 常用命令

### yarn application 查看任务

列出所有 Application

```
yarn application -list
```

根据 Application 状态过滤：yarn application -list -appStates （所有状态：ALL、NEW、 NEW_SAVING、SUBMITTED、ACCEPTED、RUNNING、FINISHED、FAILED、KILLED）

```
yarn application -list -appStates FINISHED
```

Kill 掉 Application

```
yarn application -kill 
application_1612577921195_0001
```

###  yarn logs

查询 Application 日志：yarn logs -applicationId 

```
yarn logs -applicationId application_1612577921195_0001
```

查询 Container 日志：yarn logs -applicationId  -containerId 

````
yarn logs -applicationId 
application_1612577921195_0001 -containerId 
container_1612577921195_0001_01_000001
````

### yarn applicationattempt 查看尝试运行的任务

列出所有 Application 尝试的列表：yarn applicationattempt -list 

```
yarn applicationattempt -list
application_1612577921195_0001
```

打印 ApplicationAttemp 状态：yarn applicationattempt -status 

```
yarn applicationattempt -status 
appattempt_1612577921195_0001_000001
```

### yarn container 查看容器

列出所有 Container：yarn container -list 

```
 yarn container -list  appattempt_1612577921195_0001_000001
```

打印 Container 状态：yarn container -status  

```
yarn container -status  container_1612577921195_0001_01_000001
```

注：只有在任务跑的途中才能看到 container 的状态

### yarn node 查看节点状态

列出所有节点：yarn node -list -all

### yarn rmadmin 更新配置

加载队列配置：yarn rmadmin -refreshQueues

yarn queue 查看队列

打印队列信息：yarn queue -status 

## Yarn 生产环境核心参数配置案例

注：调整下列参数之前尽量拍摄 Linux 快照，否则后续的案例，还需要重写准备集群。

1）需求：从 1G 数据中，统计每个单词出现次数。服务器 3 台，每台配置 4G 内存，4 核 CPU，4 线程。

 2）需求分析： 1G / 128m = 8 个 MapTask；1 个 ReduceTask；1 个 mrAppMaster 平均每个节点运行 10 个 / 3 台 ≈ 3 个任务（4 3 3）

修改 yarn-site.xml 配置参数如下：

```xml
<!-- 选择调度器，默认容量 -->
<property>
<description>The class to use as the resource scheduler.</description>
<name>yarn.resourcemanager.scheduler.class</name>
<value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.capaci
ty.CapacityScheduler</value>
</property>
<!-- ResourceManager 处理调度器请求的线程数量,默认 50；如果提交的任务数大于 50，可以
增加该值，但是不能超过 3 台 * 4 线程 = 12 线程（去除其他应用程序实际不能超过 8） -->
<property>
<description>Number of threads to handle scheduler 
interface.</description>
<name>yarn.resourcemanager.scheduler.client.thread-count</name>
<value>8</value>
</property>
<!-- 是否让 yarn 自动检测硬件进行配置，默认是 false，如果该节点有很多其他应用程序，建议
手动配置。如果该节点没有其他应用程序，可以采用自动 -->
<property>
<description>Enable auto-detection of node capabilities such as
memory and CPU.
</description>
<name>yarn.nodemanager.resource.detect-hardware-capabilities</name>
<value>false</value>
</property>
<!-- 是否将虚拟核数当作 CPU 核数，默认是 false，采用物理 CPU 核数 -->
<property>
<description>Flag to determine if logical processors(such as
hyperthreads) should be counted as cores. Only applicable on Linux
when yarn.nodemanager.resource.cpu-vcores is set to -1 and
yarn.nodemanager.resource.detect-hardware-capabilities is true.
</description>
<name>yarn.nodemanager.resource.count-logical-processors-ascores</name>
<value>false</value>
</property>
<!-- 虚拟核数和物理核数乘数，默认是 1.0 -->
<property>
<description>Multiplier to determine how to convert phyiscal cores to
vcores. This value is used if yarn.nodemanager.resource.cpu-vcores
is set to -1(which implies auto-calculate vcores) and
yarn.nodemanager.resource.detect-hardware-capabilities is set to true. 
The number of vcores will be calculated as number of CPUs * multiplier.
</description>
<name>yarn.nodemanager.resource.pcores-vcores-multiplier</name>
<value>1.0</value>
</property>
<!-- NodeManager 使用内存数，默认 8G，修改为 4G 内存 -->
<property>
<description>Amount of physical memory, in MB, that can be allocated 
for containers. If set to -1 and
yarn.nodemanager.resource.detect-hardware-capabilities is true, it is
automatically calculated(in case of Windows and Linux).
In other cases, the default is 8192MB.
</description>
<name>yarn.nodemanager.resource.memory-mb</name>
<value>4096</value>
</property>
<!-- nodemanager 的 CPU 核数，不按照硬件环境自动设定时默认是 8 个，修改为 4 个 -->
<property>
<description>Number of vcores that can be allocated
for containers. This is used by the RM scheduler when allocating
resources for containers. This is not used to limit the number of
CPUs used by YARN containers. If it is set to -1 and
yarn.nodemanager.resource.detect-hardware-capabilities is true, it is
automatically determined from the hardware in case of Windows and Linux.
In other cases, number of vcores is 8 by default.</description>
<name>yarn.nodemanager.resource.cpu-vcores</name>
<value>4</value>
</property>
<!-- 容器最小内存，默认 1G -->
<property>
<description>The minimum allocation for every container request at the
    RM in MBs. Memory requests lower than this will be set to the value of 
this property. Additionally, a node manager that is configured to have 
less memory than this value will be shut down by the resource manager.
</description>
<name>yarn.scheduler.minimum-allocation-mb</name>
<value>1024</value>
</property>
<!-- 容器最大内存，默认 8G，修改为 2G -->
<property>
<description>The maximum allocation for every container request at the 
RM in MBs. Memory requests higher than this will throw an
InvalidResourceRequestException.
</description>
<name>yarn.scheduler.maximum-allocation-mb</name>
<value>2048</value>
</property>
<!-- 容器最小 CPU 核数，默认 1 个 -->
<property>
<description>The minimum allocation for every container request at the 
RM in terms of virtual CPU cores. Requests lower than this will be set to 
the value of this property. Additionally, a node manager that is configured 
to have fewer virtual cores than this value will be shut down by the 
resource manager.
</description>
<name>yarn.scheduler.minimum-allocation-vcores</name>
<value>1</value>
</property>
<!-- 容器最大 CPU 核数，默认 4 个，修改为 2 个 -->
<property>
<description>The maximum allocation for every container request at the 
RM in terms of virtual CPU cores. Requests higher than this will throw an
InvalidResourceRequestException.</description>
<name>yarn.scheduler.maximum-allocation-vcores</name>
<value>2</value>
</property>
<!-- 虚拟内存检查，默认打开，修改为关闭 -->
<property>
<description>Whether virtual memory limits will be enforced for
containers.</description>
<name>yarn.nodemanager.vmem-check-enabled</name>
<value>false</value>
</property>
<!-- 虚拟内存和物理内存设置比例,默认 2.1 -->
<property>
<description>Ratio between virtual memory to physical memory when
setting memory limits for containers. Container allocations are
expressed in terms of physical memory, and virtual memory usage is 
allowed to exceed this allocation by this ratio.
</description>
<name>yarn.nodemanager.vmem-pmem-ratio</name>
<value>2.1</value>
</property>
```





# 6、常用命令

mapred --daemon start historyserver开启历史服务

*hdfs dfsadmin -safemode leave*退安全模式

netstat -alnp | grep 44444 查看端口占用情况

# 7、坑

```xml
 <property>
 	<name>dfs.permissions.enabled</name>
	<value>false</value>
</property>
```

关闭验证 hdfs-site.xml

实际不能用

./sbin/mr-jobhistory-daemon.sh start historyserver

必须内网ip

ID值
