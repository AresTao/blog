---
layout: post
title: 原来可以这样写博客--jekyll
keywords: jekyll, blog
category : jekyll
tags : [jekyll]
---


技术人员不爱写博客似乎无可厚非，专注于技术和实践，哪有心思写博客呢。但是，真正的思想的沉淀不只是放在脑子里和程序中，更需要写下来，一方面可以与别人分享、交流，另一方面在写的过程中思路也会得到纠正和完善。

技术人员普遍喜欢黑白窗口写代码那种hacker的感觉，所以他们鄙视在功能简陋的web页面去写东西，jekyll提供了另一种写博客的方式，能让博主像写程序一样去写自己的博客。广告到此为止，下面介绍一下如何搭建基于jekyll的博客。

我安装的环境是centos6.5 64位。
由于jekyll基于ruby，所以要先安装ruby、rubygems、nodejs、libyaml、libyaml-devel和make
<code>
# sudo yum install -y ruby ruby-devel rubygems make nodejs libyaml libyaml-devel
</code>
然后安装jekyll
<code>
# gem install jekyll
</code>

这时会报错，yum install安装的ruby版本是1.8.7。但是jekyll要求安装至少要1.9.2。所以就要用ruby源码安装，我安装的是1.9.3版本。
https://www.ruby-lang.org/en/news/2013/02/22/ruby-1-9-3-p392-is-released/可以下载。
<code>
# tar xvfz ruby-version.tar.gz
# cd ruby-version/
# ./configure 
# make 
</code>

这时会报错，应该是跟openssl有关，因为新版本的openssl去掉了一些东西。按照如下修改：
<code>
# vim ext/openssl/ossl_pkey_ec.c
</code>

![]({{ site.baseurl }}/images/jekyll1.jpg)
![]({{ site.baseurl }}/images/jekyll2.jpg)

重新make
然后make install即可。

接下来安装jekyll
在root下
#/usr/local/bin/gem install jekyll应该可以安装成功

好啦，jekyll安装好了。
基本用法：
<code>
# mkdir blog
# cd blog
# jeky new mysite
# cd mysite
</code>

该文件里已经默认生成了一些文件，_config.yml是配置文件，在文件中增加destination字段，指向apache或httpd的代码路径。

运行blog
jekyll build即可将编译好的代码拷贝到apache的代码路径下，默认/var/www/html/。
启动apache即可在浏览器中看到了

至此基于jekyll的静态博客系统就搭建好了，剩下的就自由发挥喽。

作为一个技术人员，应该让写博客成为一个习惯。厚积薄发！
