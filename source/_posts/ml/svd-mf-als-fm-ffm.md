title: SVD=>MF=>ALS=>FM=>FFM
date: 2018-02-9 15:26:00
tags:
- FM
categories:
- ml

---
**前言**
以下为近期对矩阵分解到银子分解机研究的总结。本来想把公式都打进去，无奈对mathjax实在不熟，直接上图吧。


# 1. SVD
$R=PSV$
其中$S$为对角矩阵

# 2. MF(LMF)
$R=P^T*Q$

BasicMF:
loss:$ \sum_{i=1}^n $

{% img /images/20180209/01.png 600 800 %}



<!--more-->
>**关于作者**
>- crazylook: pyhton,spark,ml,dl,recsys
>- blog: https://crazylook.github.io/
>- email: liuhongbin1990@foxmail.com
>
>**转载请注明出处：**
>https://crazylook.github.io/2018/02/09/ml/svd-mf-als-fm-ffm
