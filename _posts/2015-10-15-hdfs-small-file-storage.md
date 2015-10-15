---
layout: post
title: HDFS小文件处理解决方案总结
keywords: 架构设计
category : 架构
tags : [架构设计]
---

一、概述

手机图片或者像淘宝这样的网站中的产品图片特点：

（1）、大量手机用户同时在线，执行上传、下载、read等图片操作

（2）、文件数量较大，大小一般为几K到几十K左右

 

HDFS存储特点：

（1）      流式读取方式，主要是针对一次写入，多次读出的使用模式。写入的过程使用的是append的方式。

（2）      设计目的是为了存储超大文件，主要是针对几百MB，GB，甚至TB的文件

（3）      该分布式系统构建在普通PC机组成的集群上，大大降低了构建成本，并屏蔽了系统故障，使得用户可以专注于自身的操作运算。

  

HDFS与小图片存储的共通点和相悖之处：

（1）      都建立在分布式存储的基本理念之上

（2）      均要降低成本，利用普通的PC机构建系统集群

   

（1）      HDFS不适合大量小文件的存储，因namenode将文件系统的元数据存放在内存中，因此存储的文件数目受限于 namenode的内存大小。HDFS中每个文件、目录、数据块占用150Bytes。如果存放1million的文件至少消耗300MB内存，如果要存 放1billion的文件数目的话会超出硬件能力

（2）      HDFS适用于高吞吐量，而不适合低时间延迟的访问。如果同时存入1million的files，那么HDFS 将花费几个小时的时间。

（3）      流式读取的方式，不适合多用户写入，以及任意位置写入。如果访问小文件，则必须从一个datanode跳转到另外一个datanode，这样大大降低了读取性能。

    

二、HDFS文件操作流程

![]({{site.baseurl}}/images/hdfs1.gif)

reading：

![]({{site.baseurl}}/images/hdfs2.gif)

writing：

![]({{site.baseurl}}/images/hdfs3.gif)

三、HDFS自带的小文件存储解决方案

对于小文件问题，hadoop自身提供了三种解决方案：Hadoop Archive、 Sequence File 和 CombineFileInputFormat

（1）      Hadoop Archive

![]({{site.baseurl}}/images/hdfs4.gif)

归档为bar.har文件，该文件的内部结构为：

![]({{site.baseurl}}/images/hdfs5.gif)
          

创建存档文件的问题：

1、存档文件的源文件目录以及源文件都不会自动删除需要手动删除

2、存档的过程实际是一个mapreduce过程，所以需要需要hadoop的mapreduce的支持

3、存档文件本身不支持压缩

4、存档文件一旦创建便不可修改，要想从中删除或者增加文件，必须重新建立存档文件

5、创建存档文件会创建原始文件的副本，所以至少需要有与存档文件容量相同的磁盘空间

（2）      Sequence File

sequence file由一系列的二进制的对组成，其中key为小文件的名字，value的file content。

![]({{site.baseurl}}/images/hdfs6.gif)

创建sequence file的过程可以使用mapreduce工作方式完成

对于index，需要改进查找算法

          对小文件的存取都比较自由，也不限制用户和文件的多少，但是该方法不能使用append方法，所以适合一次性写入大量小文件的场景

（3）      CombineFileInputFormat

          CombineFileInputFormat是一种新的inputformat，用于将多个文件合并成一个单独的split，另外，它会考虑数据的存储位置。

该方案版本比较老，网上资料甚少，从资料来看应该没有第二种方案好。
