title: 试用Hexo创建Blog
date: 2014-05-14 12:25:31
tags:
categories:
- Hexo
---
Hexo创建Blog
# 1.创建新文章

## 1.1进入目录

```{bash}
D:\Code\workspace\javascript\nodejs-hexo>cd nodejs-hexo
```

## 1.2启动hexo服务器
```{bash}
D:\Code\workspace\javascript\nodejs-hexo>hexo server
[info] Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.
```

# 2.创建新文章
```{bash}
D:\Code\workspace\javascript\nodejs-hexo>hexo new 2014-05-11-01.将hexo创建的blog部署到BAE上
[info] File created at D:\workspace\javascript\nodejs-hexo\source\_posts\2014-05-11-01.试用

hexo创建blog.md
```

这是**新的开始**，我用hexo创建了第一篇文章。
通过下面的命令，就可以创建新文章

# 3.发布到项目
静态化命令
```{bash}
D:\Code\workspace\javascript\nodejs-hexo>hexo generate
```

# 4.常用特殊语法
## 4.1代码块
 Swig语法
{% codeblock .compact http://underscorejs.org/#compact Underscore.js %}
.compact([0, 1, false, 2, ‘’, 3]);
=> [1, 2, 3]
{% endcodeblock %}

 Markdown语法

```{bash}
.compact([0, 1, false, 2, ‘’, 3]);
=> [1, 2, 3]
```

## 4.2图片
对于本地图片，需要在source目录下面新建一个目录images，然后把图片放到目录中。
Swig语法
{% img /images/myfirst/beijing.jpg 400 600 这是一张图片 %}
Markdown语法
![这是一张图片](/images/myfirst/beijing.jpg)

## 4.3链接
 Swig语法
{% link 洪彬Blog http://liuhongbin.duapp.com/ true 洪彬Blog %}

Markdown语法
[洪彬Blog](http://liuhongbin.duapp.com)

## 4.4引用
 Swig语法
{% blockquote Seth Godin http://sethgodin.typepad.com/seths_blog/2009/07/welcome-to-island-

marketing.html Welcome to Island Marketing %}
Every interaction is both precious and an opportunity to delight.
{% endblockquote %}

Markdown语法
> Every interaction is both precious and an opportunity to delight.

<!--more-->
>**关于作者**
>- crazylook: pyhton,spark,ml,dl,recsys
>- blog: https://crazylook.github.io/
>- email: liuhongbin1990@foxmail.com
>
>**转载请注明出处：**
>https://crazylook.github.io/2014/05/14/hexo/create-blog/
