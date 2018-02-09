title: Python 正则表达式
date: 2015-02-06 15:15:31
tags:
- python
- 正则表达式
categories:
- python
---
Python 正则表达式的使用经验

# 1.Python正则表达式指南
http://www.cnblogs.com/huxi/archive/2010/07/04/1771073.html

# 2.正则表达式30分钟入门教程
http://www.jb51.net/tools/zhengze.html

# 3.示例
```{python}
pattern = re.compile(r'Regulator')
result=''
p=pattern.search("str")
if p:
    result = p.group()
print result
```


<!--more-->
>**关于作者**
>- crazylook: pyhton,spark,ml,dl,recsys
>- blog: https://crazylook.github.io/
>- email: liuhongbin1990@foxmail.com
>
>**转载请注明出处：**
>https://crazylook.github.io/2015/02/06/python/notes-Regulator/
