---
layout:     post
title:      "快速搭建自己的github.io博客"
subtitle:   ""
date:       2018-01-03
author:     "吴煌辉"
header-img: "img/blog-header/SetupGithubPages-header.jpeg"
catalog: true
tags:
    - 教程
---


# 前言
作为懒癌患者的我，今天突然想搭一个自己的博客，于是百度了一下如何利用GitHub搭建自己的博客。(PS:不使用Google是因为懒得科学上网)

搭建博客总共两个步骤：

* 开通 GitHub Pages
* 选择一款 Jekyll 主题


# 开通 GitHub Pages

登录[Github](https://github.com)，如果你没账号请自行注册。新建一个Repository，注意 ***新建的Repository 名字必须为username.github.io，其中username为注册的账户名***。

<div style="text-align: center">
    <img src="https://pages.github.com/images/user-repo@2x.png" alt=""/>
</div>

之后我们要把博客的Repository clone到本地。在终端中敲入如下命令：
***git clone https://github.com/username/username.github.io***
也可以使用SourceTree等GitHub客户端工具。

这样就开通了你的博客，地址为：https://username.github.io。

#	选择 Jekyll 主题
GitHub Pages 默认使用 [Jekyll](https://jekyllrb.com/) 作为建站工具。我们刚开通的博客是空空如也的，选择一款 Jekyll [主题](https://github.com/Huxpro/huxpro.github.io)来装修我们的博客页面，下载解压，然后拷贝到clone的本地文件夹username.github.io中。

修改`_config.yml`文件：

```
# Site settings
title: 旧梦似流水
SEOTitle: 旧梦似流水的博客 | Wuhh Blog
header-img: img/home-bg.jpeg
email: 387703391@qq.com
description: "描述"
keyword: "iOS开发，心情"
url: "https://wuhh1991.github.io"              # your host, for absolute URL
baseurl: ""                             # for example, '/blog' if your blog hosted on 'host/blog'
```

然后把所有的文件都push到GitHub。到此我们的博客就搭建完了。

# 发布文章
搭建完博客，我们就可以开始写文章了。Jekyll 把文章都放在`_posts`目录下，并且要求要求文件名格式为：***YEAR-MONTH-DAY-title.MARKUP***，例如：2018-01-03-SetupGithubPages.md。推荐使用Markdown来编写文章，写好文章后push到GitHub就可以了。


# 本地查看自己的博客

* 安装 ruby

	Mac 安装命令 ：brew install ruby，其他系统请自行百度/Google。
	
	```
   $ brew install ruby
   ```

* 安装 Github pages

    ```
    $ gem install github-pages
    ```
	
* 开启 Jekyll 查看本地博客
	
	```
	$ cd username.github.io
	$ jekyll serve --watch
	```

启动 Jekyll 本地服务以后，在终端找到 `Server address: http://127.0.0.1:4000/`，在浏览器中输入***http://127.0.0.1:4000*** 即可。

# 最后
本文结合自身搭建 GitHub 博客经验，并参考了[如何快速搭建自己的github.io博客](https://keysaim.github.io/2017/08/15/how-to-setup-your-github-io-blog/), 同时感谢[Huxpro](https://github.com/Huxpro/huxpro.github.io)提供的`Jekyll`主题。

