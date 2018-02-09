title: 'Hive UDF函数开发'
date: 2014-06-10 10:55:57
tags:
- HiveUDF
categories:
- Hive
---
Hive学习笔记系列，Hive UDF函数开发

# 1.在eclipse中新建java project
然后新建class  package(com.jd.lhb)  name(UDFLower)

# 2.添加jar library
添加hadoop-core-1.0.3.jar   hive-exec-0.8.1.jar两个文件到project
build path

# 3.编写代码
```
package com.jd.lhb;

import org.apache.hadoop.hive.ql.exec.Description;
import org.apache.hadoop.hive.ql.exec.UDF;
import org.apache.hadoop.io.Text;

@Description(name="my_lower",
		value="_FUNC_(String) - transfer the upper string to lowwer",
		extended="Example:\n"
			+ " > select _FUNC_(String) from src;\n")
public class UDFLower extends UDF{

	/**
	 * @author lhb
	 * @param s
	 * 将大写的字符串s	transfer to小写的字符串
	 */
	public Text evaluate(final Text s){
		if(null==s){
			return null;
		}
		return new Text(s.toString().toLowerCase());
	}
}
```

# 4.编译打包文件为HiveUDF.jar

# 5.加载至hive中
```
hive> add jar /home/hadoop/udf/HiveUDF.jar
Added udf_hive.jar to class path
Added resource: udf_hive.jar
```

# 6.测试准备

## 6.1data.txt文件内容为
```
WHO
AM
I
HELLO
```

## 6.2创建测试数据表
hive> create table dual (info string);

## 6.3导入数据文件data.txt
```
hive> load data local inpath '/home/hadoop/data.txt' into table dual;
```

## 6.4创建udf函数
```
hive> create temporary function my_lower as 'com.afan.UDFLower';
```

# 7.测试
```
hive> select info from dual;
使用udf函数
hive> select my_lower(info) from dual;
```

<!--more-->
>**关于作者**
>- crazylook: pyhton,spark,ml,dl,recsys
>- blog: https://crazylook.github.io/
>- email: liuhongbin1990@foxmail.com
>
>**转载请注明出处：**
>https://crazylook.github.io/2014/06/10/hive/notes-chapter13-udf/
