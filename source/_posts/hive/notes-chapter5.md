title: 'Hive学习笔记Chapter5——HQL:数据操作'
date: 2014-05-17 10:55:57
tags:
- Hive操作
categories:
- Hive
---
Hive学习笔记系列，HQL：数据操作。

# 1.装载本地数据
```
load data local inpath '/home/hadoop/hivedata' overwrite into table t_data;
```

# 2.装载HDFS上的数据
```
load data inpath '/user/hadoop/hivedata' overwrite into table t_data;
```

# 3.通过查询语句插入数据

## 3.1覆盖式插入
```
insert overwrite table t_data select * from t_data limit 2;
```

## 3.2追加式插入
```
insert into table t_data select * from t_data limit 2;
```

## 3.3变形
```
from t_data
insert overwrite table t_data
select * where name = 'lhb' limit 2;
```

# 4.通过查询语句创建表
```
create table t_data_new
select name,id from t_data;
```

# 5.导出数据

## 5.1Hadoop方式导出
```
hadoop fs -copytolocal /user/hive/warehouse/t_data ~/hive
```

## 5.2insert directory 方式导出
```
insert overwrite local directory '/home/hadoop/hive'
select * from t_data;
```

## 5.3insert directory 方式导出到HDFS
```
insert overwrite directory '/user/hadoop/hive'
select * from t_data;
```


<!--more-->
>**关于作者**
>- crazylook: pyhton,spark,ml,dl,recsys
>- blog: https://crazylook.github.io/
>- email: liuhongbin1990@foxmail.com
>
>**转载请注明出处：**
>https://crazylook.github.io/2014/05/17/hive/notes-chapter5/
