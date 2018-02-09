title: Hexo主题优化
date: 2014-05-17 11:14:41
tags:
- Hexo优化
categories:
- Hexo
---
Hexo主题优化

# 1.添加“多说”评论
1. 在[多说](http://duoshuo.com/)进行注册，获得通用代码。
2. 将通用代码粘贴到themes\light\layout\_partial\comment.ejs里面，如下：
```
<% if ( page.comments){ %>
<section id="comment">
通用代码
</section>
<% } %>
```
通用代码
{% img /images/2014-5-17/01.jpg 800 800 %}

# 2.添加分类、标签云widget
很简单，在themes/light/_config.yml中，添加如下：
```
widgets:
- category
- tagcloud
- tag
- links
- rss
```

# 3.导航栏添加”关于”
在source/about目录中 ``hexo new index``
到source/about/index.md编辑内容。
在themes/light/_config.yml中，添加如下：
```
menu:
  关于: /about
```

# 4.主页文章显示摘要
编辑md文件的时候，在要作为摘要的文字后面添加``<!--more-->``即可。

# 5.添加RSS
hexo提供了RSS的生成插件，需要手动安装和设置。步骤如下：
1.安装RSS插件到本地：``npm install hexo-generator-feed --save``
2.开启RSS功能：编辑hexo/_config.yml，添加如下代码：
```
feed:
    type: atom
    path: atom.xml
    limit: 20
 ```

3.在站点添加链接：
在themes/light/_config.yml中，编辑 ``rss: /atom.xml``
在themes/light/layout/_partial/header.ejs中，``<ul></ul>``之间，添加一样代码``<li><a href="/atom.xml">RSS</a></li>``

# 6.添加sitemap
我们使用hexo提供的插件，方法与添加RSS类似。
安装sitemap到本地：`npm install hexo-generator-sitemap --save`
开启sitemap功能：编辑hexo/_config.yml，添加如下代码：
```
sitemap:
    path: sitemap.xml
```
访问http://liuhongbin.jd-app.com/sitemap.xml
即可看到站点地图。不过，sitemap的初衷是给搜索引擎看的，为了提高搜索引擎对自己站点的收录效果，我们最好手动到google和百度等搜索引擎提交sitemap.xml。



<!--more-->
>**关于作者**
>- crazylook: pyhton,spark,ml,dl,recsys
>- blog: https://crazylook.github.io/
>- email: liuhongbin1990@foxmail.com
>
>**转载请注明出处：**
>https://crazylook.github.io/2014/05/17/hexo/theme-alteration/
