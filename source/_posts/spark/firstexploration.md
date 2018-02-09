title: Spark初探——关于分享Spark的一些感悟
date: 2014-06-24 12:20:31
tags:
- Spark
categories:
- Spark
---
本文主要用于分享Spark的一些感悟
<!--more-->
>**关于作者**
>- crazylook: pyhton,spark,ml,dl,recsys
>- blog: https://crazylook.github.io/
>- email: liuhongbin1990@foxmail.com
>
>**转载请注明出处：**
>https://crazylook.github.io/2014/06/24/spark-notes-firstexploration/

**前言**
之前偶尔遇到Spark、Shark这些名词，因为一直对Hadoop研究还有点粗浅，不愿意去接这触个号称更高一级别的Spark。这两天接触了Spark亚太研究院的一个免费公开课，开始揭开Spark的神秘面纱。
下面正式开始干货。

# 1. Spark为什么能够把Hadoop速度提高100倍以上
## 1.1 How fast to write with scala?
### 1.1.1 Spark的WordCount实例
```
val file = sc.textFile("hdfs://...")
val counts = file.flatMap(line => line.split(" ")).map(word => (word, 1)).reduceByKey(_ + _)
counts.saveAsTextFile(“hdfs://…")
```

顿时感觉scala编程很有R的味道（简洁、函数式编程）。本人很喜欢，做为本科数学系的人来说就是喜欢函数式编程。

### 1.1.2 对比一下Wordcount in Hadoop
{% img /images/2014-6-24/01.png 400 400 %}

我想我再也不愿意用Java写mapreduce程序了。

### 1.1.3 对比一下RHadoop写的Wordcount
```
input<- '/user/hadoop/input'
wordcount.mr<-mapreduce(
    input,
    map=function(k,v){
        key<-unlist(strsplit(v," "))
　　    keyval(key,1)
    }
　　,reduce=function(k,v){
        keyval(k,sum(v))
　　}
)
from.dfs(wordcount.mr)
```

其实RHadoop编程也很简洁，但是RHadoop还不是一个可用于生产环境的产品。

## 1.2MapReduce架构
{% img /images/2014-6-24/09.png 400 400 %}

## 1.3Why Hadoop so slow?
{% img /images/2014-6-24/06.png 400 400 %}

## 1.4Spark? So fast!

{% img /images/2014-6-24/07.png 400 400 %}
**More reasons!**
* DAG     有向无环图
* Scheduler   调度
{% img /images/2014-6-24/08.png 400 400 %}

* Lineage     血统(容错处理)

# 2. Spark的内核剖析
## 2.1 一些令人激动的图
### 2.1.1 上一个高大上的架构图
{% img /images/2014-6-24/02.png 400 400 %}

### 2.1.2 One stack to rule them all
用Spark，一个团队可以干Hadoop三个不同团队干的活。
{% img /images/2014-6-24/03.png 400 400 %}

### 2.1.3 Project  Goals
{% img /images/2014-6-24/04.png 400 400 %}

### 2.1.4 Codebase size
这个是一个让我发狂的东东，Spark主要内核代码只有2W行，这点燃了我研究Spark源码的欲望
{% img /images/2014-6-24/05.png 200 200 %}

## 2.2 Spark核心概念

### 2.2.1 (RDD)弹性分布数据集
* RDD(Resilient Distributed Dataset)是Spark的最基本抽象,是对分布式内存的抽象使用，实现了以操作本地集合的方式来操作分布式数据集的抽象实现。RDD是Spark最核心的东西，它表示已被分区，不可变的并能够被并行操作的数据集合，不同的数据集格式对应不同的RDD实现。RDD必须是可序列化的。RDD可以cache到内存中，每次对RDD数据集的操作之后的结果，都可以存放到内存中，下一个操作可以直接从内存中输入，省去了MapReduce大量的磁盘IO操作。这对于迭代运算比较常见的机器学习算法, 交互式数据挖掘来说，效率提升比较大。
* RDD的特点：
1. 它是在集群节点上的不可变的、已分区的集合对象。
2. 通过并行转换的方式来创建如（map, filter, join, etc）。
3. 失败自动重建。
4. 可以控制存储级别（内存、磁盘等）来进行重用。
5. 必须是可序列化的。
6. 是静态类型的。
* RDD的好处
1. RDD只能从持久存储或通过Transformations操作产生，相比于分布式共享内存（DSM）可以更高效实现容错，对于丢失部分数据分区只需根据它的lineage就可重新计算出来，而不需要做特定的Checkpoint。
2. RDD的不变性，可以实现类Hadoop MapReduce的推测式执行。
3. RDD的数据分区特性，可以通过数据的本地性来提高性能，这与Hadoop MapReduce是一样的。
4. RDD都是可序列化的，在内存不足时可自动降级为磁盘存储，把RDD存储于磁盘上，这时性能会有大的下降但不会差于现在的MapReduce。
* RDD的存储与分区
1. 用户可以选择不同的存储级别存储RDD以便重用。
2. 当前RDD默认是存储于内存，但当内存不足时，RDD会spill到disk。
3. RDD在需要进行分区把数据分布于集群中时会根据每条记录Key进行分区（如Hash 分区），以此保证两个数据集在Join时能高效。
* RDD的内部表示
在RDD的内部实现中每个RDD都可以使用5个方面的特性来表示：
1. 分区列表（数据块列表）
2. 计算每个分片的函数（根据父RDD计算出此RDD）
3. 对父RDD的依赖列表
4. 对key-value RDD的Partitioner【可选】
5. 每个数据分片的预定义地址列表(如HDFS上的数据块的地址)【可选】
* RDD的存储级别
RDD根据useDisk、useMemory、deserialized、replication四个参数的组合提供了11种存储级别：
```
    val DISK_ONLY = new StorageLevel(true, false, false)
    val DISK_ONLY_2 = new StorageLevel(true, false, false, 2)
    val MEMORY_ONLY = new StorageLevel(false, true, true)
    val MEMORY_ONLY_2 = new StorageLevel(false, true, true, 2)
    val MEMORY_ONLY_SER = new StorageLevel(false, true, false)
    val MEMORY_ONLY_SER_2 = new StorageLevel(false, true, false, 2)
    val MEMORY_AND_DISK = new StorageLevel(true, true, true)
    val MEMORY_AND_DISK_2 = new StorageLevel(true, true, true, 2)
    val MEMORY_AND_DISK_SER = new StorageLevel(true, true, false)
    val MEMORY_AND_DISK_SER_2 = new StorageLevel(true, true, false, 2)
```
* RDD定义了各种操作，不同类型的数据由不同的RDD类抽象表示，不同的操作也由RDD进行抽实现。

### 2.2.2 Lineage（血统）
* 利用内存加快数据加载,在众多的其它的In-Memory类数据库或Cache类系统中也有实现，Spark的主要区别在于它处理分布式运算环境下的数据容错性（节点实效/数据丢失）问题时采用的方案。为了保证RDD中数据的鲁棒性，RDD数据集通过所谓的血统关系(Lineage)记住了它是如何从其它RDD中演变过来的。相比其它系统的细颗粒度的内存数据更新级别的备份或者LOG机制，RDD的Lineage记录的是粗颗粒度的特定数据转换（Transformation）操作（filter, map, join etc.)行为。当这个RDD的部分分区数据丢失时，它可以通过Lineage获取足够的信息来重新运算和恢复丢失的数据分区。这种粗颗粒的数据模型，限制了Spark的运用场合，但同时相比细颗粒度的数据模型，也带来了性能的提升。
* RDD在Lineage依赖方面分为两种Narrow Dependencies与Wide Dependencies用来解决数据容错的高效性。Narrow Dependencies是指父RDD的每一个分区最多被一个子RDD的分区所用，表现为一个父RDD的分区对应于一个子RDD的分区或多个父RDD的分区对应于一个子RDD的分区，也就是说一个父RDD的一个分区不可能对应一个子RDD的多个分区。Wide Dependencies是指子RDD的分区依赖于父RDD的多个分区或所有分区，也就是说存在一个父RDD的一个分区对应一个子RDD的多个分区。对与Wide Dependencies，这种计算的输入和输出在不同的节点上，lineage方法对与输入节点完好，而输出节点宕机时，通过重新计算，这种情况下，这种方法容错是有效的，否则无效，因为无法重试，需要向上其祖先追溯看是否可以重试（这就是lineage，血统的意思），Narrow Dependencies对于数据的重算开销要远小于Wide Dependencies的数据重算开销。

### 2.2.3 容错
* 在RDD计算，通过checkpint进行容错，做checkpoint有两种方式，一个是checkpoint data，一个是logging the updates。用户可以控制采用哪种方式来实现容错，默认是logging the updates方式，通过记录跟踪所有生成RDD的转换（transformations）也就是记录每个RDD的lineage（血统）来重新计算生成丢失的分区数据。

### 2.2.4 资源管理与作业调度
* Spark对于资源管理与作业调度可以使用Standalone(独立模式)，Apache Mesos及Hadoop YARN来实现。 Spark on Yarn在Spark0.6时引用，但真正可用是在现在的branch-0.8版本。Spark on Yarn遵循YARN的官方规范实现，得益于Spark天生支持多种Scheduler和Executor的良好设计，对YARN的支持也就非常容易，Spark on Yarn的大致框架图。
{% img /images/2014-6-24/10.png 600 600 %}
* 让Spark运行于YARN上与Hadoop共用集群资源可以提高资源利用率。

### 2.2.5 Scala
* Spark使用Scala开发，默认使用Scala作为编程语言。编写Spark程序比编写Hadoop MapReduce程序要简单的多，SparK提供了Spark-Shell，可以在Spark-Shell测试程序。写SparK程序的一般步骤就是创建或使用(SparkContext)实例，使用SparkContext创建RDD，然后就是对RDD进行操作。如：
```
    val sc = new SparkContext(master, appName, [sparkHome], [jars])
    val textFile = sc.textFile("hdfs://.....")
    textFile.map(....).filter(.....).....
 ```
Scala是我接触的除了R又一个令我激动的语言，有机会要好好研究一下。

# 3.Spark集群案例解析，包含集群搭建

## 3.1 Standalone模式
* 为方便Spark的推广使用，Spark提供了Standalone模式，Spark一开始就设计运行于Apache Mesos资源管理框架上，这是非常好的设计，但是却带了部署测试的复杂性。为了让Spark能更方便的部署和尝试，Spark因此提供了Standalone运行模式，它由一个Spark Master和多个Spark worker组成，与Hadoop MapReduce1很相似，就连集群启动方式都几乎是一样。
* 以Standalone模式运行Spark集群

1. 下载Scala2.9.3，并配置SCALA_HOME
2. 下载Spark代码（可以使用源码编译也可以下载编译好的版本）这里下载 编译好的版本（http://spark-project.org/download/spark-0.7.3-prebuilt-cdh4.tgz）
3. 解压spark-0.7.3-prebuilt-cdh4.tgz安装包
4. 修改配置（conf/*） slaves: 配置工作节点的主机名 spark-env.sh：配置环境变量。
```
SCALA_HOME=/home/spark/scala-2.9.3
JAVA_HOME=/home/spark/jdk1.6.0_45
SPARK_MASTER_IP=spark1             
SPARK_MASTER_PORT=30111
SPARK_MASTER_WEBUI_PORT=30118
SPARK_WORKER_CORES=2 SPARK_WORKER_MEMORY=4g
SPARK_WORKER_PORT=30333
SPARK_WORKER_WEBUI_PORT=30119
SPARK_WORKER_INSTANCES=1
```
5. 把Hadoop配置copy到conf目录下
6. 在master主机上对其它机器做ssh无密码登录
7. 把配置好的Spark程序使用scp copy到其它机器
8. 在master启动集群
```
$SPARK_HOME/start-all.sh
```

## 3.2 yarn模式
* Spark-shell现在还不支持Yarn模式，使用Yarn模式运行，需要把Spark程序全部打包成一个jar包提交到Yarn上运行。目录只有branch-0.8版本才真正支持Yarn。
* 以Yarn模式运行Spark

1. 下载Spark代码.
`
git clone git://github.com/mesos/spark
`
2. 切换到branch-0.8
```
cd spark
git checkout -b yarn --track origin/yarn
```
3. 使用sbt编译Spark并
```
$SPARK_HOME/sbt/sbt
package
assembly
```
4. 把Hadoop yarn配置copy到conf目录下
5. 运行测试
```
 SPARK_JAR=./core/target/scala-2.9.3/spark-core-assembly-0.8.0-SNAPSHOT.jar \
./run spark.deploy.yarn.Client --jar examples/target/scala-2.9.3/ \
--class spark.examples.SparkPi --args yarn-standalone
```

## 3.3 使用Spark-shell
* Spark-shell使用很简单，当Spark以Standalon模式运行后，使用$SPARK_HOME/spark-shell进入shell即可，在Spark-shell中SparkContext已经创建好了，实例名为sc可以直接使用，还有一个需要注意的是，在Standalone模式下，Spark默认使用的调度器的FIFO调度器而不是公平调度，而Spark-shell作为一个Spark程序一直运行在Spark上，其它的Spark程序就只能排队等待，也就是说同一时间只能有一个Spark-shell在运行。
* 在Spark-shell上写程序非常简单，就像在Scala Shell上写程序一样。
```
    scala> val textFile = sc.textFile("hdfs://hadoop1:2323/user/data")
    textFile: spark.RDD[String] = spark.MappedRDD@2ee9b6e3

    scala> textFile.count() // Number of items in this RDD
    res0: Long = 21374

    scala> textFile.first() // First item in this RDD
    res1: String = # Spark
```

## 3.4 编写Driver程序
* 在Spark中Spark程序称为Driver程序，编写Driver程序很简单几乎与在Spark-shell上写程序是一样的，不同的地方就是SparkContext需要自己创建。如WorkCount程序如下：
```
import spark.SparkContext
import SparkContext._

object WordCount {
  def main(args: Array[String]) {
    if (args.length ==0 ){
      println("usage is org.test.WordCount <master>")
    }
    println("the args: ")
    args.foreach(println)

    val hdfsPath = "hdfs://hadoop1:8020"

    // create the SparkContext， args(0)由yarn传入appMaster地址
    val sc = new SparkContext(args(0), "WrodCount",
    System.getenv("SPARK_HOME"), Seq(System.getenv("SPARK_TEST_JAR")))

    val textFile = sc.textFile(hdfsPath + args(1))

    val result = textFile.flatMap(line => line.split("\\s+"))
        .map(word => (word, 1)).reduceByKey(_ + _)

    result.saveAsTextFile(hdfsPath + args(2))
  }
}
```

**附：一篇介绍Spark比较到位的Page**
http://tech.uc.cn/?p=2116
