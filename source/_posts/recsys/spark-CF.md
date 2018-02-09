title: Spark中实现ItemCF、UserCF
date: 2018-01-18 10:20:00
tags:
- ItemCF
categories:
- recsys

---
**前言**
以下为对实际生产环境中ItemCF、UserCF召回策略的总结

# 1. Spark中带分数的itemBaseCF
## 1.1 不考虑用户打分的差异性
```
1.item , (user,score)
2.item , list[(user,score)] ==> item , sqrt(|item|) --> sqrt_item
3.item , (user,score,sqrt_item) ==> user , list[(item,score,sqrt_item)]
4.cal_score  (item1,item2), item1_score * item2_score/sqrt_item1 * sqrt_item2          
5.group by add => (item1,item2) , similar_score   ---这就是余弦距离的计算方法sim=dot(d1,d2)/sqrt(|d1|)*sqrt(|d2|)
6.item1，list[(item2,similar_score)]
7.item1, list[topK]
```

## 1.2 考虑用户打分的差异性
```
-1.user , (item,score)
-2.user , list[(item,score)] ==> user, avg(score) --》 avg_score
-3.user , (item,score,avg_score) ==> user , (item,adj_score)
下面的score就是上面的adj_score
1.item , (user,score)
2.item , list[(user,score)] ==> item , sqrt(|item|) --> sqrt_item
3.item , (user,score,sqrt_item) ==> user , list[(item,score,sqrt_item)]
4.cal_score  (item1,item2), item1_score * item2_score/sqrt_item1 * sqrt_item2          
5.group by add => (item1,item2) , similar_score   ---这就是余弦距离的计算方法sim=dot(d1,d2)/sqrt(|d1|)*sqrt(|d2|)
6.item1，list[(item2,similar_score)]
7.item1, list[topK]
```

# 2. Spark中不带分数的itemBaseCF
```
1.item , user
2.item , list[user] ==> item , sqrt(item_cnt) --> sqrt_item
3.item , (user,sqrt_item) ==> user , list[(item,sqrt_item)]
4.cal_score  (item1,item2), 1 * 1/sqrt_item1 * sqrt_item2          
5.group by add => (item1,item2) , similar_score   ---这就是余弦距离的计算方法sim=d1_d2_cnt/sqrt(d1_cnt)*sqrt(d2_cnt)
6.item1，list[(item2,similar_score)]
7.item1, list[topK]
```
注:
1. 计算方法 只是把上面余弦距离的score全都置为1了
2. 本方法最终也就是是共现矩阵的余弦相似度计算方法 d1_d2_cnt/sqrt(d1_cnt*d1_cnt)

# 3. cal_score相似度 计算方法
```
def cal_score(line):
"""
Args: line:汇总的一个用户的评分行为，tuple格式为：(user, list[(item,score,sqrt_item)])
Returns: item间相似度，tuple格式为:(item1\1item2, similar_score)
"""
user,item = line
index1 = 0
while index1 < len(item):
    index2 = index1 + 1
    while index2 < len(item):

        if item[index1][0] == item[index2][0] :
            index2 += 1
            continue

        item1_score = float(item[index1][1])
        item1_score = float(item[index2][1])
        sqrt_item1 = float(item[index1][2])
        sqrt_item2 = float(item[index2][2])
        try:
            similar = item1_score*item2_score/(sqrt_item1*sqrt_item2)
            if item[index1][0] != None and item[index2][0] != None:
                 key = item[index1][0] + "\1" + item[index2][0]
                 yield (key, similar)
        except:
            pass
        index2 += 1
    index1 += 1
```


# 4. UserCF 和ItemCF原理相同

将user , list[(item,score,sqrt_item)] 换为 item , list[(user,score,sqrt_user)] 即可



<!--more-->
>**关于作者**
>- crazylook: pyhton,spark,ml,dl,recsys
>- blog: https://crazylook.github.io/
>- email: liuhongbin1990@foxmail.com
>
>**转载请注明出处：**
>https://crazylook.github.io/2018/01/18/recsys/spark-CF
