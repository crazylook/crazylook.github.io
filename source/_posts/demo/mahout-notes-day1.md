title: 'Mahout开发环境搭建及推荐系统初探'
date: 2014-06-18 15:55:57
tags:
- Mahout
categories:
- 那些年的小项目
---
Mahout开发环境搭建及推荐系统初探

# 1.用Maven构建Mahout项目
## 1.1 Maven介绍和安装

下载Maven：http://maven.apache.org/download.cgi

下载最新的xxx-bin.zip文件，在win上解压到 F:\ProgramFiles\maven-3.2.1
并把maven/bin目录设置在环境变量PATH：
{% img /images/2014-6-18/01.png 400 400 %}
然后，打开命令行输入mvn -version，我们会看到mvn命令的运行效果
```
Apache Maven 3.2.1 (ea8b2b07643dbb1b84b6d16e1f08391b666bc1e9; 2014-02-15T01:37:5
2+08:00)
Maven home: F:\ProgramFiles\maven-3.2.1\bin\..
Java version: 1.6.0_20, vendor: Sun Microsystems Inc.
Java home: F:\Program Files\Java\jdk1.6.0_20\jre
Default locale: zh_CN, platform encoding: GBK
OS name: "windows xp", version: "5.1", arch: "x86", family: "windows"
```

安装Eclipse的Maven插件: http://www.eclipse.org/m2e/


# 2.用Maven构建Mahout开发环境
## 2.1用Maven创建一个标准化的Java项目
```
D:\Code\workspace>mvn archetype:generate -DarchetypeGroupId=org.apache.maven.archetypes
-DgroupId=org.lhb.mymahout -DartifactId=myMahout -DpackageName=org.lhb.mymahout -Dversion=1.0-SNAPSHOT -DinteractiveMode=false
```

进入项目，执行mvn命令

```
D:\Code\workspace>cd myHadoop
D:\Code\workspace\myHadoop>mvn clean install
```

## 2.2导入项目到eclipse
我们创建好了一个基本的maven项目，然后导入到eclipse中。 这里我们最好已安装好了Maven的插件。
## 2.3增加hadoop依赖
这里我使用hadoop-1.0.3版本，修改文件：pom.xml
```
vi pom.xml

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>org.conan.mymahout</groupId>
  <artifactId>myMahout</artifactId>
  <packaging>jar</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>myMahout</name>
  <url>http://maven.apache.org</url>
<properties>
<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
<mahout.version>0.8</mahout.version>
</properties>

<dependencies>
<dependency>
<groupId>org.apache.mahout</groupId>
<artifactId>mahout-core</artifactId>
<version>${mahout.version}</version>
</dependency>
<dependency>
<groupId>org.apache.mahout</groupId>
<artifactId>mahout-integration</artifactId>
<version>${mahout.version}</version>
<exclusions>
<exclusion>
<groupId>org.mortbay.jetty</groupId>
<artifactId>jetty</artifactId>
</exclusion>
<exclusion>
<groupId>org.apache.cassandra</groupId>
<artifactId>cassandra-all</artifactId>
</exclusion>
<exclusion>
<groupId>me.prettyprint</groupId>
<artifactId>hector-core</artifactId>
</exclusion>
</exclusions>
</dependency>
</dependencies>
</project>
```

## 2.4下载依赖
```
mvn clean install
```

在eclipse中刷新项目:

## 2.5从Hadoop集群环境下载hadoop配置文件

core-site.xml
hdfs-site.xml
mapred-site.xml
```
vim core-site.xml

<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
  <property>
    <name>fs.default.name</name>
    <value>hdfs://Master.Hadoop:9000</value>
  </property>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/usr/hadoop/tmp</value>
  </property>
  <property>
    <name>io.sort.mb</name>
    <value>1024</value>
  </property>
</configuration>
```

```
vim hdfs-site.xml

<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <property>
    <name>dfs.data.dir</name>
    <value>/usr/hadoop/data</value>
  </property>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
  <property>
    <name>dfs.permissions</name>
    <value>false</value>
  </property>
</configuration>

```
```
vim mapred-site.xml

<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <property>
    <name>mapred.job.tracker</name>
    <value>hdfs://Master.Hadoop:9001</value>
  </property>
</configuration>
```

{% img /images/2014-6-18/02.png 400 400 %}

## 2.6配置本地host，增加master的域名指向
```
vi c:/Windows/System32/drivers/etc/hosts

192.168.1.210 Master.Hadoop
```

# 3.用Mahout实现协同过滤userCF
## 3.1新建数据文件: item.csv
```
mkdir datafile
vi datafile/item.csv

1,101,5.0
1,102,3.0
1,103,2.5
2,101,2.0
2,102,2.5
2,103,5.0
2,104,2.0
3,101,2.5
3,104,4.0
3,105,4.5
3,107,5.0
4,101,5.0
4,103,3.0
4,104,4.5
4,106,4.0
5,101,4.0
5,102,3.0
5,103,2.0
5,104,4.0
5,105,3.5
5,106,4.0
```
数据解释：每一行有三列，第一列是用户ID，第二列是物品ID，第三列是用户对物品的打分。
## 3.2新建JAVA类：org.lhb.mymahout.UserCF.java
```
package org.lhb.mymahout;

import java.io.File;
import java.io.IOException;
import java.util.List;

import org.apache.mahout.cf.taste.common.TasteException;
import org.apache.mahout.cf.taste.impl.common.LongPrimitiveIterator;
import org.apache.mahout.cf.taste.impl.model.file.FileDataModel;
import org.apache.mahout.cf.taste.impl.neighborhood.NearestNUserNeighborhood;
import org.apache.mahout.cf.taste.impl.recommender.GenericUserBasedRecommender;
import org.apache.mahout.cf.taste.impl.similarity.EuclideanDistanceSimilarity;
import org.apache.mahout.cf.taste.model.DataModel;
import org.apache.mahout.cf.taste.recommender.RecommendedItem;
import org.apache.mahout.cf.taste.recommender.Recommender;
import org.apache.mahout.cf.taste.similarity.UserSimilarity;

/**
 * @author LHB
 *
 */
public class UserCF {

  /**
   * @param LHB
   * @throws IOException
   * @throws TasteException
   */
  final static int NEIGHBORHOOD_NUM=2;
  final static int RECOMMENDER_NUM=3;
  public static void main(String[] args) throws IOException, TasteException {
    // TODO Auto-generated method stub
    String file= "datafile/item.csv";
    //构建数据模型
    DataModel model=new FileDataModel(new File(file));
    //构建用户相似度模型 EuclideanDistanceSimilarity:欧式距离相似度
    UserSimilarity similar=new EuclideanDistanceSimilarity(model);
    //最近邻 NEIGHBORHOOD_NUM个最近邻
    NearestNUserNeighborhood neighbor=new NearestNUserNeighborhood(NEIGHBORHOOD_NUM,similar,model);
    //构建推荐模型
    Recommender recommender=new GenericUserBasedRecommender(model,neighbor,similar);

    LongPrimitiveIterator iter=model.getUserIDs();

    while(iter.hasNext()){
      Long uid=iter.nextLong();
      List<RecommendedItem> list=recommender.recommend(uid, RECOMMENDER_NUM);
      System.out.printf("uid:%s",uid);
      for(RecommendedItem item:list){
        System.out.printf("(%s,%f)",item.getItemID(),item.getValue());
      }
      System.out.println();
    }

  }

}
```
## 3.3运行程序 控制台输出:
```
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
uid:1(104,4.274336)(106,4.000000)
uid:2(105,4.055916)
uid:3(103,3.360987)(102,2.773169)
uid:4(102,3.000000)
uid:5
```
## 3.4推荐结果解读
向用户ID1，推荐前二个最相关的物品, 104和106
向用户ID2，推荐前二个最相关的物品, 但只有一个105
向用户ID3，推荐前二个最相关的物品, 103和102
向用户ID4，推荐前二个最相关的物品, 但只有一个102
向用户ID5，推荐前二个最相关的物品, 没有符合的


<!--more-->
>**关于作者**
>- crazylook: pyhton,spark,ml,dl,recsys
>- blog: https://crazylook.github.io/
>- email: liuhongbin1990@foxmail.com
>
>**转载请注明出处：**
>https://crazylook.github.io/2014/06/18/demo/mahout-notes-day1/
