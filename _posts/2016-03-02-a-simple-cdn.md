---
layout: post
title: 【转】搭建一个简单的cdn
keywords: 架构,cdn
category : 架构
tags : [架构]
---

我们时常提到“智能云分发”一词，觉得高深莫测，但其基本原理与最早期提出的 CDN 是一样的，并未发生本质上的改变。

可最早期的 CDN 又是什么样子呢？

实际上，它并没有那么神秘、也没那么复杂。我们可以粗略的认为它就是一个基于Origin Pull 或 Push CDN的内容缓存平台。

对于最早期的CDN平台来说，因为其仅仅通过有限的静态选项来代理内容，已经无法适应故障或者针对当前全局网络状况进行动态调整，所以此类早期的 CDN 已经离我们远去。

但是我们仍然可以通过亲自动手来构建一个小型的CDN 系统，以了解其工作原理。

注：请在实验室及测试而非生产环境下完成以下各步骤，也不要将该实验系统用于生产环境中，因为目前即使生产环境下的常规 CDN 也远比该示例先进得多。

搭建一个简易CDN需要几步？仅需3步！不信您请看～

###1.安装Nginx Web服务器

目前大部分Linux版本均预装了Nginx，我们仅举一下在Ubuntu下面的安装方法，请运行命令：

sudo apt-get install nginx

###2.配置虚拟机

在Web服务器安装完毕后，接下来应配置虚拟主机。您可以在Ubuntu的/etc/nginx/conf.d/或 /etc/nginx/sites-enabled/中配置与编辑，以便实施快速测试。

请记得关闭防火墙，并让网络在随后的测试中能够访问您的测试服务器。

如需完整的配置示例文件，请访问该链接地址：

https://github.com/abshkd/dumbcdn/blob/master/nginx-cdn.conf
将该配置放入Nginx目录内。

创建目录/tmp/cache并给予Nginx服务/用户写入许可，并重新加载Nginx服务。如果操作正确的话，服务将正常重启。随后可以在测试各个选项的过程中编辑该文件，并重新加载服务以应用相关更改。

###3.设定配置文件

下面我们将介绍配置中的主要命令行含义

####3.1 缓存目录设定

先介绍首行：

proxy_cache_path /tmp/cache levels=1:2 keys_zone=my_cache:100m max_size=1g inactive=60m;
该命令设定了Nginx用来存储缓存文件的目录以及其能够创建的子目录级数：

keys_zone 确定了存放内存对象的共享存储区。此处，我们还将用于缓存的最大存储空间设定为100 MB，并将磁盘缓存设定为1 GB。

inactive 的设置可能使某些人感到困惑，这里的意思是说，如果缓存对象在60分钟内未得到访问，它将被移除，为正在被访问的新缓存文件腾出空间，这点请牢记。

它与您发送给浏览器并使用 HTTP 304 缓冲的内容无关，只是一个维护特性。将在下文中加以解释。

####3.2 设定主机名

您可以定义多个主机名，各主机名之间用空格加以分隔。

{% highlight XML %}
listen [::]:80;
listen:80;
#edit your actual site name 
server_name www.example.com;
{% endhighlight %}

上述信息只是为了说明web服务器所采用的主机名，以及在哪些端口上接收。此处，我选择在80端口上接收所有的IPv4 与 IPv6。您可将www.example.com替换为任意站点，甚至可以替换为搭载站点的本地网络主机，以便供您进行尝试。当然，它也可以是实际的站点！

####3.3 Origin 即是王道

{% highlight XML %}
location / {
    proxy_pass http://origin.example.com;
    proxy_set_header Host $host;
    proxy_set_header  True-Client-IP  $remote_addr;

}
{% endhighlight %}

毫无意外， Origin主机名可以是一个IP。我可以将LAN IP或我的虚拟机IP作为origin。除了server_name，其他的都可以随便使用 。逻辑上，您只是创建了一个循环。Nginx非常智能，足以指出您所犯的错误。

proxy_pass命令告诉web服务器将匹配location “/“（它是站点的根路径）的所有请求以及其中未在别处定义的任意路径发送至何处。

proxy_set_header命令将其他数据头发送至Forward Host。Host将设定正确的Host数据头，$host是内嵌变量，相当于我们在第1部分放入的server_name。只要远程服务器能够对其响应，则可以一直指定静态名称。

在我们所熟悉的特殊True-Client-IP数据头中，我们增加的第2个数据头用于将实际Client IP发送至origin。

切记，我们实际的origin将隐藏于我们简单的CDN服务器中，因此它不会知道请求来自代理。我们可以添加该数据头，用于测试。

这条规则主要用于说明Nginx代理将所有请求发送至origin，并不太复杂。

####3.4 定义静态对象列表

{% highlight XML %}
location ~* .(jpg|png|gif|jpeg|webp|css|mp3|wav|swf|mov|doc|pdf|xls|ppt|docx|pptx|xlsx)
$ {...
{% endhighlight %}

最后一个程序块为我们第2节中默认的“root”规则增加了一条。它拥有许多我所添加的可行命令。任意文件扩展的情况均匹配该规则，从而执行整个程序块。我将解释主要的命令行，其他命令可以在您的测试中加以研究。

proxy_cache_valid 200 301 7d;
对于 HTTP 状态为200 OK 或 301 永久重定向的响应，缓存期为7天。在解释首行配置含义时，曾提及inactive=60m，并对其进行扩充。因此，我们说funny.gif 文件之所以得到缓存是因为符合各项标准。

理想的情况下，web服务器命令浏览器也缓存7天，简单的CDN在存储器内将其保存7天，但条件是每隔60分钟“至少”被访问一次，否则将被清除。它可能在1个小时后被弹出，因此您将继续看到来自它的请求。

proxy_cache_min_uses 2;
在考虑缓存对象时至少需要两次访问。如果某个文件仅被访问一次，则将仅得到origin的服务。

proxy_cachemy_cache;
该命令定义了我们在第1个配置命令行中指明的存储区。请勿与Cache key混淆。

###4. 运行

为了进行测试，您只需要做一番“伪装”！一旦重新加载web服务器，即可在系统中使用web浏览器编辑hosts文件，将www.example.com指向您的简单CDN IP地址，并多次载入浏览器。

检查测试服务器中的/tmp/cache目录，您将在其中发现新的目录与文件。这些都是磁盘缓存文件！

这是一种学习缓存与CDN基础知识的不错方法。

它虽比不上某些智能云分发平台的功能水平，但能够让您访问希望评估的任意站点，并针对尚未公开访问的测试应用查找异常行为。当然，您也可以添加一系列选项/目录。Nginx具有高度可编程特性，如果您喜欢脚本处理，它还提供Lua扩展（测试版）。

###5. 不足之处

您将开始意识到简单型CDN服务器存在多种限制，不仅局限于该实验室配置，其中我还忽略了一些内容，例如无缓存的cookies。

我编写了Nginx配置，但未在系统内实际运行，您可以通过nginx–test选项加以运行。如果您发现了漏洞，请告诉我，或者在Github上面发送pull 请求。
