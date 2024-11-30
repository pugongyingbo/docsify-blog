---
title: 使用hugo创建博客
date: 2017-03-06
keywords: hugo
desc:
tags: hugo
author: ""
---

#### Hello Hugo in windows

> 在网上找的教程都是其他系统的，没有Windows的，准备自己试一次，其实很简单

* Hugo官方主页：[https://gohugo.io/](https://gohugo.io/) 

* 从GitHub下载匹配自己系统的版本

* 解压缩，然后例如放在C:/Hugo/bin下，把.exe名字改成hugo.exe

* 添加到系统变量中（我的是windows）

这个时候应该可以在你的cmd命令行里执行hugo了，如果出来一堆命令之类的说明安装成功。新建个文件夹Site,我们在这new site myblog，会新建一个文件夹叫myblog，然后clone喜欢的模板 

<code>

    $ mkdir themes
    $ cd themes
    $ git clone https://github.com/spf13/hyde.git 

<code/>

* 对配置文件config.toml进行修改

<code>

	baseurl = "http://example.com/"
	languageCode = "en-us"
	title = "My Blog"
	theme = "hyde"
	[params]
    	description = "这里是个人博客"
<code/>

* 这个时候可以新建你的文章了（在post下)

<code>

    $ hugo new post/test.md 
<code/>

* 在目录下运行 hugo server 打开浏览器输入localhost：1313，就可以看见你的博客了

* 然后可以将其挂在giuhub page上

## 关于部署

* 假设你需要部署在 GitHub Pages上，首先在GitHub上创建一个Repository，命名为：(必须是你的用户名)×××.github.io

* 在站点根目录执行 Hugo 命令生成最终页面：

<code>

    $ hugo --theme=hyde --baseUrl="http://coderzh.github.io/"
<code/>

 
*　如果一切顺利，所有静态页面都会生成到 public 目录，将pubilc目录里所有文件 push 

<code>
    
    $ cd public
    $ git init
    $ git remote add origin https://github.com/coderzh/coderzh.github.io.git
    $ git add -A
    $ git commit -m "first commit"
    $ git push -u origin master
<code/>


#### 参考的文档

* 官方主页

* Hugo中文文档: [http://www.gohugo.org/](http://www.gohugo.org/)；

* 参考: [http://blog.coderzh.com/2015/08/29/hugo/](http://blog.coderzh.com/2015/08/29/hugo/)；

* 参考：[http://tonybai.com/2015/09/23/intro-of-gohugo/](http://tonybai.com/2015/09/23/intro-of-gohugo/)；

