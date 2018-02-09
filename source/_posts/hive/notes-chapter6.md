title: 'Hive学习笔记Chapter6——HQL:查询'
date: 2014-05-27 11:36:57
tags:
- Hive查询
categories:
- Hive
---
Hive学习笔记系列，HQL:查询。
# 1.select...from
## 1.1使用正则表达式来指定列
```
select symbol,`price.*` from stocks;
```

## 1.2算术运算符
```
A+B
A-B
A*B
A/B     A除以B
A%B     A除以B的余数
A&B     A和B按位取与
A|B     A和B按位取或
A^B     A和B按位取异或
~A      A按位取反
```

## 1.3使用函数
### 1.3.1数学函数
```
round(Double d)			返回d的BIGINT类型的近似值
round(Double d,INT n)		返回d的Double类型的,保留n位小数的近似值
floor(Double d)			返回<=d的最大BIGINT类型的近似值
ceil(Double d)/ceiling(Double d)		返回>=d的最小的BIGINT类型的近似值
rand() 	rand(Int seed)		返回一个Double类型的随机数，整数seed是随机因子。
exp(Double d)			返回e的d幂次方，返回的是个Double类型值
pow/power(Double d,Double p)	返回d的p次幂
sqrt(Double d)			计算d的平方根
bin(BIGINT i)			计算二进制值i的string类型值
hex(BIGINT i)			计算十六进制值i的string类型值
hex(string str)			计算十六进制表达的值str的string类型值
hex(BINARY b)			计算二进制值d的string类型值
unhex(string i)			hex(string i)的逆操作
conv(string num,Int from_base,Int to_base)	将string类型的num从from_base进制转化为to_base进制，并返回string类型的结果
pmod(INT i1,INT i2)		INT值i1对INT值i2取模，结果也是INT型
pmod(Double i1,Double i2)	Double值i1对Double值i2取模，结果也是Double型
e()				数学常数e
pi()				数学常数pi
```

### 1.3.2聚合函数
```
sum(DISTINCT col)		指定列排重后的总和
variance(col)/var_pop(col)	返回集合col的一组数值的方差
var_samp(col)			返回集合col的一组数值的样本方差
stddev_pop(col)			返回集合col的一组数值的标准偏差
stddev_samp(col)		返回集合col的一组数值的样本方差
covar_pop(col1,col2)	返回两组数值的协方差
covar_samp(col1,col2)	返回两组数值的样本协方差
corr(col1,col2)			返回两组数值的相关系数
percentile(BIGINT int_expr,p)			int_expr在p(范围是：[0,1])处对应的百分比，其中p是一个double型数值
percentile(BIGINT int_expr,ARRAY(p1[,p2]...))			int_expr在p1(范围是：[0,1])处对应的百分比，其中p1是一个double型数值
histogram_numeric(col,NB)	返回NB数量的直方图仓库数组，返回结果中的值x是中心，值y是仓库的高
collect_set(col)		返回集合col元素排重后的数组
```
注：可以通过设置属性hive.map.aggr的值为true来提高聚合的性能
```
hive> set hive.map.aggr=true;
```

### 1.3.3生成表函数
```
explode(ARRAY array)	返回0到多行结果，每行都对应输入的array数组中的一个元素
explode(MAP map)		返回0到多行结果，每行都对应一个map键-值对
explode(ARRAY<TYPE> a)		对于a中的每个元素，explode()会生成一行记录包含这个元素
inline(ARRAY <struct[,struct]>)		将结构体数组提取出来并插入到表中
json_tuple(String jsonStr,p1,p2,...,pn)		本函数可以接受多个标签名称，对输入的json字符串进行处理
parse_url_tuple(url,partname1,partname2,...,partnameN)		从url中解析出N个部分的信息
stack(INT n,col1,...,colM)		把M列转化为N行，每行有M/N个字段。其中n必须是常数
```

### 1.3.4其他内置函数

```
cast(<expr> as <type>)				将expr转化为type类型。
contact(BINARY s1,BINARY s2)			将二进制字符s1,s2拼接成一个字符串
contact(String s1,String s2)			将字符串s1,s2拼接成一个字符串
contact_ws(String separator,String s1,String s2)和contact类似，不过是使用指定的字符分隔开的
decode(BINARY bin,String charset)		使用指定的字符集charset将二进制值bin解码成字符串
encode(String src,String charset)		使用指定的字符集charset将字符串src编码成二进制值
find_in_set(String s,String commaSepa-ratedString)返回在以逗号分隔的字符串中S出现的位置
format_number(NUMBER x,INT d)			将数值x转换成‘#，###，###.##’格式的字符串，并保留d位小数。
in (val1,val2)					例如，test in (val1,val2),如果test值等于后面列表的任意值返回true
in_file(String s,String filename)		如果文件名filename的文件中有完整一行数据和字符串s完全相同true
instr(String str,String substr)			查找字符串中子字符串substr第一次出现的位置
length(String s)				计算字符s的长度
locate(String substr,String str [,INT pos])	查找字符串str中的pos位置后字符串substr第一次出现的位置
lpad(String s,INT len,String pad)		从左边开始对字符串S使用字符串pad进行填充，最终达到len长度为之。
ltrim(String s)			将字符串s前面出现的空格全部去除
parse_url(String url,String partname [,String key])	从url中抽取指定部分的内容
printf(String format,Obj ... args)		按照printf风格格式化输出输入的字符

```

<!--more-->
>**关于作者**
>- crazylook: pyhton,spark,ml,dl,recsys
>- blog: https://crazylook.github.io/
>- email: liuhongbin1990@foxmail.com
>
>**转载请注明出处：**
>https://crazylook.github.io/2014/05/27/hive/notes-chapter6/
