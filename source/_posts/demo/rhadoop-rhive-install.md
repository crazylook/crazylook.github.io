title: RHadoop（rhdfs、rmr2、RHive）安装、配置
date: 2014-05-15 11:15:30
tags:
- RHadoop
- RHive
categories:
- 那些年的小项目
---
RHadoop（rhdfs、rmr2、RHive）安装、配置

# 1.系统及所需软件版本
　　服务器操作系统：CentOS 6.4
　　R语言版本：R-2.15.3.tar.gz
　　JDK版本：jdk-6u45-linux-i586.bin
　　Hadoop版本：hadoop-1.0.3.tar.gz
　　Hive版本：hive-0.9.0.tar.gz
　　MySQL版本：MySQL-client-5.5.21-1.linux2.6.i386.rpm
　　 MySQL-server-5.5.21-1.linux2.6.i386.rpm
　　rJava版本：rJava_0.9-5.tar.gz
　　RHadoop版本：rhdfs_1.0.5.tar.gz	rmr2_2.2.0.tar.gz	rhbase_1.1.1.tar.gz
　　下载地址：
https://github.com/RevolutionAnalytics/RHadoop/wiki/Downloads
# 2.依赖安装（R语言包、rJava包）
　　在安装之前需要在集群各个主机上逐个安装R语言包、rJava包，然后再进行Rhadoop的安装。具体安装步骤如下：
## 2.1安装R语言包
　　在编译R之前，需要通过yum安装以下几个程序：

```
	> # 检查R是否支持PNG等图形显示：
	> capabilities()
	jpeg      png
	TRUE     FALSE
	首先，退出R，然后安装一堆相关的包
	# yum install libpng libpng-devel libtiff libtiff-devel libjpeg-turbo libjpeg-turbo-devel

	# yum install gcc-gfortran
	否则报”configure: error: No F77 compiler found”错误

	# yum install gcc gcc-c++
	否则报”configure: error: C++ preprocessor “/lib/cpp” fails sanity check”错误

	# yum install readline-devel
	否则报”--with-readline=yes (default) and headers/libs are not available”错误

	# yum install libXt-devel
	否则报”configure: error: --with-x=yes (default) and X11 headers/libs are not available”错误

	然后下载源代码,编译
	# cd /home/admin
	# wget http://cran.rstudio.com/src/base/R-2/R-2.15.3.tar.gz

	# tar -zxvf R-2.15.3.tar.gz
	# cd /usr/R-2.15.3
	# ./configure --prefix=/usr --disable-nls  --enable-R-shlib --with-libpng --with-jpeglib --with-libtiff --with-x
	# ./configure --prefix=/usr --disable-nls --enable-R-shlib/** (后面两个选项--disable-nls --enable-R-shlib是为RHive的安装座准备，如果不安装RHive可以省去)*/
	# make clean
	# make
	# make install
	安装完毕后查看，得到R的安装路径为/usr/lib/R ，即后来要设置的R_HOME所在的目录。
```

## 2.2安装rJava包:
	版本：rJava_0.9-5.tar.gz
	在联网的情况下，可以进入R命令，安装rJava包：
```
	> install.packages("rJava")
	如果待安装机器不能上网，可以将源文件下载到本地，然后通过shell命令R CMD INSTALL ‘package_name’来安装：
	R CMD INSTALL "rJava_0.9-5.tar.gz"
	本教程的包，大部分都是都是本地安装的，只有Rserve等个别包。
```

	然后设置Java、Hadoop、R、Hive等相关环境变量（如果在搭建Cloudera Hadoop集群时已经设置好，做一下检查就OK）

	下面是RHadoop及RHive安装成功时/etc/profile中的环境变量配置情况
```
	#Java environment
	export JAVA_HOME=/usr/java/jdk1.6.0_45
	export CLASSPATH=.:$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
	export PATH=$PATH:$JAVA_HOME/bin:$JAVA_HOME/jre/bin

	# set Hadoop environment
	export HADOOP_HOME=/usr/hadoop
	export HADOOP_CONF_DIR=/usr/hadoop/conf
	export PATH=$PATH:$HADOOP_HOME/bin
	　　
	# set HADOOP_CMD environment
	export HADOOP_CMD=/usr/hadoop/bin/hadoop
	export HADOOP_STREAMING=/usr/hadoop/contrib/streaming/hadoop-streaming-1.0.3.jar

	# set Hive environment
	export HIVE_HOME=/usr/hive
	export PATH=$PATH:$HIVE_HOME/bin
	export CLASSPATH=$CLASSPATH:$HIVE_HOME/lib
	export RHIVE_DATA=/usr/lib/R/rhive/data
	　　
	#set R_HOME
	export R_HOME=/usr/lib/R
	export CLASSPATH=.:/usr/lib/R/library/rJava/jri
	export LD_LIBRARY_PATH=/usr/lib/R/library/rJava/jri
	export PATH=$PATH:$R_HOME/bin
	export RServe_HOME=/usr/lib/R/library/RServe
```

# 3.安装RHadoop环境
（rhdfs、rmr2、rhbase、RHive）
## 3.1安装rhdfs包(仅安装在namenode上)：
```
	R CMD INSTALL "rhdfs_1.0.5.tar.gz"
	在/etc/profile中设置环境变量HADOOP_HOME、HADOOP_CON_DIR、HADOOP_CMD

	export HADOOP_HOME=/usr/hadoop
	export HADOOP_CONF_DIR=/usr/hadoop/conf
	export HADOOP_CMD=/usr/hadoop/bin/hadoop
	安装后调用rhdfs，测试安装：

	> library("rhdfs")
	Loading required package: rJava

	HADOOP_CMD=/usr/bin/hadoop

	Be sure to run hdfs.init()
```

## 3.2安装rmr2包（各个主机上都要安装）：
### 3.2.1安装其依赖的7个包
```
[root@master RHadoop-deps]# ls
digest_0.6.3.tar.gz    plyr_1.8.tar.gz     reshape2_1.2.2.tar.gz  stringr_0.6.2.tar.gz    functional_0.4.tar.gz  Rcpp_0.10.3.tar.gz  RJSONIO_1.0-3.tar.gz
执行安装：
[root@master RHadoop-deps]#  R CMD INSTALL "digest_0.6.3.tar.gz"
[root@master RHadoop-deps]#  R CMD INSTALL "plyr_1.8.tar.gz"
[root@master RHadoop-deps]#  R CMD INSTALL "reshape2_1.2.2.tar.gz"
[root@master RHadoop-deps]#  R CMD INSTALL "stringr_0.6.2.tar.gz"
[root@master RHadoop-deps]#  R CMD INSTALL "functional_0.4.tar.gz"
[root@master RHadoop-deps]#  R CMD INSTALL "Rcpp_0.10.3.tar.gz"
[root@master RHadoop-deps]#  R CMD INSTALL "RJSONIO_1.0-3.tar.gz"
```

　　如果未安装，或者7个包安装不全，安装程序会提示其所依赖的的包要安装。

### 3.2.2R CMD INSTALL "rmr2_2.2.0.tar.gz"
　　需要在/etc/profile中设置环境变量HADOOP_STREAMING

	```
　　export HADOOP_STREAMING=/usr/hadoop/contrib/streaming/hadoop-streaming-1.0.3.jar
	```

　　安装测试：
　　安装好rhdfs和rmr2两个包后，我们就可以使用R尝试一些Hadoop的操作了。

## 3.3测试RHadoop
### 3.3.1基本的hdfs的文件操作。
	查看hdfs文件目录
　　hadoop的命令：`hadoop fs -ls /user`
　　R语言函数：`hdfs.ls("/user/")`
　　
　　查看hadoop数据文件
　　hadoop的命令：`hadoop fs -cat /user/hadoop/input/RHadoop.txt`
　　R语言函数：`hdfs.cat("/user/hadoop/input/RHadoop.txt")`
### 3.3.2执行一个rmr2算法的任务
　　普通的R语言程序：
```
　　> small.ints = 1:10
　　> sapply(small.ints, function(x) x^2)
```

　　MapReduce的R语言程序：

```
　　> library("rhdfs")
　　> hdfs.init()
　　>hdfs.ls("/user/")
　　> library("rmr2")
　　> small.ints = to.dfs(1:10)
　　> mapreduce(input = small.ints, map = function(k, v) cbind(v, v^2))
```

### 3.2.3rmr2 WordCount实例：
```
　　#wordcunt示例
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

#4安装RHive（各个主机上都要安装）：
RHive是一种通过Hive高性能查询来扩展R计算能力的包。它可以在R环境中非常容易的调用HQL， 也允许在Hive中使用R的对象和函数。理论上数据处理量可以无限扩展的Hive平台，搭配上数据挖掘的利器R环境， 堪称是一个完美的大数据分析挖掘的工作环境。
## 4.1Rserve包的安装：
RHive依赖于Rserve，因此在安装R的要按照本文R的安装方式，即附带后面两个选项（--disable-nls --enable-R-shlib）
enable-R-shlib是将R作为动态链接库进行安装，这样像Rserve依赖于R动态库的包就可以安装了，但缺点会有20%左右的性能下降。
```
install.packages("Rserve")								（在线安装）
R CMD INSTALL /home/hadoop/Rserve_1.7-3.tar.gz 			（本地方式安装）
```

$R_HOME的目录下创建Rserv.conf文件，写入“remote enable''保存并退出。通过scp -r 命令将Master节点上安装好的Rserve包，以及Rserv.conf文件拷贝到所有slave节点下。当然在节点不多的情况下也可以分别安装Rserve包、创建Rserv.conf。

### 4.1.1查看进程
```
~ ps -aux|grep Rserve
```

### 4.1.2杀掉刚才的Rserve守护进程
```
~ kill -9 7142

scp -r /usr/lib/R/library/Rserve 192.168.1.151:/usr/lib/R/library/
scp -r /usr/lib/R/library/Rserv.conf 192.168.1.151:/usr/lib/R/library/

在所有节点启动Rserve
R CMD Rserve --RS-conf /usr/lib/R/library/Rserv.conf

在master节点上telnet（如果未安装，通过shell命令yum install telnet安装）所有slave节点:
yum install inetd
yum install libncurses.so.4
rpm -ivh /home/hadoop/telnet-0.10-31.i386.rpm

telnet Master.Hadoop 6311
显示Rsrv013QAP1则表示连接成功。
```

## 4.2RHive包的安装：
安装RHive_0.0-7.tar.gz，并在master和所有slave节点上创建rhive的data目录，并赋予读写权限（最好将$R_HOME赋予777权限）
```
[root@master admin]# R CMD INSTALL RHive_0.0-7.tar.gz
[root@master admin]# cd $R_HOME
[root@master R]# mkdir -p rhive/data
[root@master R]# chmod 777 -R rhive/data
master和slave中的/etc/profile中配置环境变量RHIVE_DATA=/usr/lib/R/rhive/data

export RHIVE_DATA=/usr/lib/R/rhive/data
通过scp命令将master节点上安装的RHive包拷贝到所有的slave节点下：

scp -r /usr/lib/R/library/RHive Slave1.Hadoop:/usr/lib/R/library/
查看hdfs文件下的jar是否有读写权限

hadoop fs -ls /rhive/lib
安装rhive后，hdfs的根目录并没有rhive及其子目录lib，这就需要自己建立，并将/usr/lib/R/library/RHive/java下的rhive_udf.jar复制到该目录

hadoop fs -put /usr/lib/R/library/RHive/java/rhive_udf.jar /rhive/lib
否则在测试rhive.connect()的时候会报没有/rhive/lib/rhive_udf.jar目录或文件的错误。
最后，在hive客户端（master、各slave均可）启动hive远程服务（rhive是通过thrift连接hiveserver的，需要要启动后台thrift服务）：
hive --service hiveserver  &
```

## 4.3RHive的使用及测试：
### 4.3.1启动RHive
```
　　在所有节点启动Rserve(远程模式启动)
　　R CMD Rserve --RS-enable-remote --RS-conf /usr/lib/R/library/Rserv.conf
　　telnet Master.Hadoop 6311
　　(
　　#查看进程
　　~ ps -aux|grep Rserve
　　kill -9 进程号	#杀死Rserve
　　#查看端口
　　~ netstat -nltp|grep Rserve
　　tcp     0      0 0.0.0.0:6311     0.0.0.0:*     LISTEN   7173/Rserve
　　0 0.0.0.0:6311，表示不限IP访问了。
　　）
　　
　　启动后台thrift服务
　　hive --service hiveserver &
　　
　　> library("RHive")
　　> rhive.init()
　　> rhive.connect("Master.Hadoop")
　　> rhive.list.tables()
　　>d<-rhive.query("select * from xp");
　　> class(d)
　　> rhive.close()
```

### 4.3.2RHive API
从HIVE中获得表信息的函数，比如
```
rhive.list.tables：获得表名列表，支持pattern参数(正则表达式)，类似于HIVE的show tables
rhive.desc.table：表的描述，HIVE中的desc table
rhive.exist.table：
```

### 4.3.3测试
```
library("RHive")
Loading required package: Rserve
This is RHive 0.0-7. For overview type ?.RHive?.
HIVE_HOME=/opt/cloudera/parcels/CDH-4.3.0-1.cdh4.3.0.p0.22/lib/hive
[1] "there is no slaves file of HADOOP. so you should pass hosts argument when you call rhive.connect()."
call rhive.init() because HIVE_HOME is set.
Warning message:
In file(file, "rt") :
  cannot open file '/etc/hadoop/conf/slaves': No such file or directory
rhive.connect()
13/06/17 20:32:33 WARN conf.Configuration: fs.default.name is deprecated. Instead, use fs.defaultFS
13/06/17 20:32:33 WARN conf.Configuration: fs.default.name is deprecated. Instead, use fs.defaultFS
```

表明安装成功，只是conf下面的slaves没有配置，在/etc/hadoop/conf中新建slaves文件，并写入各个slave的名称即可解决该警告。

### 4.4.4RHive的运行环境如下：
```
rhive.env()
Hive Home Directory : /usr/hive
Hadoop Home Directory : /usr/hadoop
Hadoop Conf Directory : /usr/hadoop/conf
No RServe
Connected HiveServer : 127.0.0.1:10000
Connected HDFS : hdfs://master:8020
```

### 4.4.5RHive简单应用
载入RHive包，令连接Hive，获取数据：
```
library(RHive)
rhive.connect(host = 'host_ip')
d <- rhive.query('select * from emp limit 1000')
class(d)
m <- rhive.block.sample(data_sku, percent = 0.0001, seed = 0)
rhive.close()
```

一般在系统中已经配置了host，因此可以直接rhive.connect()进行连接，记得最后要有rhive.close()操作。 通过HIVE查询语句，将HIVE中的目标数据加载至R环境下，返回的 d 是一个dataframe。

实际上，rhive.query的实际用途有很多，一般HIVE操作都可以使用，比如变更scheme等操作：

```
rhive.query('use scheme1')
rhive.query('show tables')
rhive.query('drop table emp')
```

但需要注意的是，数据量较大的情况需要使用rhive.big.query，并设置memlimit参数。

将R中的对象通过构建表的方式存储到HIVE中需要使用

```
rhive.write.table(dat, tablename = 'usertable', sep = ',')
```

而后使用join等HIVE语句获得相关建模数据。其实写到这儿，有需求的看官就应该明白了，这几项 RHive 的功能就足够 折腾些有趣的事情了。

注1：其他关于在HIVE中调用R函数，暂时还没有应用，未来更新。
注2：rhive.block.sample这个函数需要在HIVE 0.8版本以上才能执行。

# 5.Rstudio Server 安装记录
## 5.1配置动态链接库和环境变量
```
vi /etc/profile
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib/R/lib/
source /etc/profile
vi /usr/lib64/R/etc/Renviron
HADOOP_CMD=/usr/hadoop/bin/hadoop
HADOOP_HOME=/usr/hadoop
HADOOP_STREAMING=/usr/hadoop/contrib/streaming/hadoop-streaming-1.0.3.jar
HIVE_HOME=/usr/hive
RHIVE_DATA=/usr/lib/R/rhive/data
```

## 5.2安装相关库
确认以下动态链接库文件已安装 libcairo.so.2 libcrypto.so.6 libgfortran.so.1 libpango-1.0.so.0
libpangocairo-1.0.so.0 libssl.so.6 否则可以
```
yum install libcrypto.so.6 libgfortran.so.1 openssl098e-0.9.8e
```

## 5.3下载安装rstudio-server
http://download2.rstudio.org/rstudio-server-0.97.551-i686.rpm
```
rpm -Uvh --nodeps rstudio-server-0.97.551-i686.rpm
```

## 5.4使用rstudio-server
http://192.168.1.150:8787/


<!--more-->
>**关于作者**
>- crazylook: pyhton,spark,ml,dl,recsys
>- blog: https://crazylook.github.io/
>- email: liuhongbin1990@foxmail.com
>
>**转载请注明出处：**
>https://crazylook.github.io/2014/05/15/demo/rhadoop-rhive-install/
