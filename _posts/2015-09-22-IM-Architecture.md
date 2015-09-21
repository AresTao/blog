---
layout: post
title: --转载-- IM系统架构
keywords: 架构设计
category : 架构
tags : [架构设计]
---

对于一款移动产品，除了基本的功能性，我们更要考虑以下几点：电量、流量、网络。
电量：对于移动设备最大的瓶颈就是电量了。因为用户不可能随时携带电源，充电宝。所以必须考虑到电量问题。那就要检查我们工程是不是有后台运行，心跳包发送时间是不是合理。
流量：对于好多国内大部分用户来说可能还在使用2G，流量还是包月30M，那么我们必须站在广大用户角度来考虑问题了。一个包可以解决的就一个包。
![]({{site.baseurl}}/images/reducelight.png)
![]({{site.baseurl}}/images/state.png)
网络：这个也是IM最核心的内容了，我们要做到在任何网络下能顺畅聊天那就不容易了，好多公司都用的xmpp框架，如果在强网络环境下，xmpp完全没有问题。但是那种弱网络环境下xmpp就束手无策啦，用户体验就很垃圾了。个人觉得xmpp 可以玩玩，但是用来做真正的产品就差远了。如果遇到一个做IM的朋友张口闭口都说xmpp 的话，那么不用沟通了，肯定不是什么好产品。微信、QQ以前也曾用过xmpp，但是最后也放弃了xmpp，就知道xmpp有很多弊端了，还有就是报文太大，好臃肿，浪费流量。为了保证稳定，微信用了长链接和短链接相结合，例如：
1 、两个域名
微信划分了http模式（short链接）和 tcp 模式（long 链接），分别应对状态协议和数据传输协议
long.weixin.qq.com  dns check （112.64.237.188 112.64.200.218）
 short.weixin.qq.com  dns check  ( 112.64.237.186 112.64.200.240)
2 说明
2.1 short.weixin.qq.com  
 是HTTP协议扩展，运行8080 端口，http body为二进制（protobuf）。
 主要用途（接口）：
用户登录验证;
好友关系（获取，添加）；
消息sync (newsync)，自有sync机制；
获取用户图像；
用户注销；
行为日志上报。
朋友圈发表刷新
2.2  long.weixin.qq.com  
tcp 长连接， 端口为8080，类似微软activesync的二进制协议。
主要用途（接口）：
接受/发送文本消息；
接受/发送语音；
接受/发送图片；
接受/发送视频文件等。
 
 所有上面请求都是基于tcp长连接。在发送图片和视频文件等时，分为两个请求；第一个请求是缩略图的方式，第二个请求是全数据的方式。
 
2.2.1 数据报文方面
增量上传策略：
每次8k左右大小数据上传，服务器确认；在继续传输。
 
图片上传：
先传缩略图，传文本消息，再传具体文件
 
下载：
先下载缩略图， 在下载原图
下载的时候，全部一次推送。

3 附录
3.1  用户登录验证
POST /cgi-bin/micromsg-bin/auth HTTP/1.1
Accept: **
User-Agent: Mozilla/4.0
Content-Type: application/x-www-form-urlencoded
Host: short.weixin.qq.com
Content-Length: 174
 
3.3 消息sync (newsync)
POST /cgi-bin/micromsg-bin/newsync HTTP/1.1
Host: short.weixin.qq.com
User-Agent: Android QQMail HTTP Client
Cache-Control: no-cache
Connection: Keep-Alive
Content-Type: application/octet-stream
accept: **
Content-Length:  206
 
3.5 用户注销
POST /cgi-bin/micromsg-bin/iphoneunreg HTTP/1.1
Accept: */*
User-Agent: Mozilla/4.0
Cotent-Type: application/x-www-form-urlencoded
Host: short.weixin.qq.com
Content-Length: 271
 
3.6 行为日志上报
POST /cgi-bin/stackreport?version=240000a7&filelength=1258&sum=7eda777ee26a76a5c93b233eed504c7d&reporttype=1&username=jolestar HTTP/1.1
Content-Length: 736
Content-Type: binary/octet-stream
Host: weixin.qq.com
Connection: Keep-Alive
User-Agent: Apache-HttpClient/UNAVAILABLE (java 1.4)

从现在互联网的发展而言，IM和视频（包括IM里面视频通话）是一个方向，这些都应该成为互联网的基础设施，就像浏览器一样。现在IM还没有一个很好的解决方案，XMPP并不能很好地做到业务逻辑独立开来。从IM的本质来看，IM其实就是将一条消息从一个地方传输到另外一个地方，这个和TCP很像，为什么不实现一个高级点的TCP协议了，只是将TCP/IP里面的IP地址换成了一个类似XMPP的唯一ID而已，其他的很多细节都可以照搬TCP协议。有了这个协议之后，将业务逻辑在现有HTTP server的基础上做，例如发送语音和图片就相当于上传一个文件，服务器在处理完这个文件后就发一条特殊的IM消息。客户端收到这个IM消息后，按照IM消息里面url去HTTP server取语音文件和图片文件。将HTTP server和IM server打通之后，可以做很多事情。但将这个两个server合并在一块并不是一个好事，不然腾讯也不会有2005年的战略转型了。从现在的情况来看，应用除了游戏，都没怎么赚钱，现在能够承载赚钱业务的还是以web为主。IM不可以赚钱，但没有却是不行的，就像一个地方要致富，不修路是不行的道理一样。

上面说到了protobuf ，就简单介绍下：
JSON相信大家都知道是什么东西，如果不知道，那可就真的OUT了，GOOGLE一下去。这里就不介绍啥的了。
Protobuffer大家估计就很少听说了，但如果说到是GOOGLE搞的，相信大家都会有兴趣去试一下，毕竟GOOGLE出口，多属精品。
Protobuffer是一个类似JSON的一个传输协议，其实也不能说是协议，只是一个数据传输的东西罢了。
那它跟JSON有什么区别呢？
跨语言，这是它的一个优点。它自带了一个编译器，protoc，只需要用它进行编译，可以编译成JAVA、python、C++代码，暂时只有这三个，其他就暂时不要想了，然后就可以直接使用，不需要再写任何其他代码。连解析的那些都已经自带有的。JSON当然也是跨语言的，但这个跨语言是建立在编写代码的基础上。

陌陌设计
========
陌陌发展刚开始由于规模小，30-40W的连接数（包括Android后台长连接用户），也使用XMPP；由于XMPP的缺点：流量大（基于XML），不可靠（为传统固定网络设计，没有考虑WIFI/2G/3G/地铁/电梯等复杂网络场景），交互复杂（登陆需5-6次，尤其是TLS握手）；XMPP丢消息的根本原因：服务端和客户端处于“半关闭”状态，客户端假连接状态，服务端又收不到回执；Server端连接层和逻辑层代码没有解耦分离，常常重启导致不可用；

![]({{site.baseurl}}/images/momoserver.png)

消息中转：

![]({{site.baseurl}}/images/messagetransfer.png)

连接层：

![]({{site.baseurl}}/images/linklayer.png)

逻辑层：

![]({{site.baseurl}}/images/logiclayer.png)

通信协议设计：

![]({{site.baseurl}}/images/protocol1.png)

高效：弱网络快速的收发
可靠：不会丢消息
易于扩展
协议格式：

![]({{site.baseurl}}/images/protocol2.png)

redis协议：

![]({{site.baseurl}}/images/redis1.png)
![]({{site.baseurl}}/images/redis2.png)
![]({{site.baseurl}}/images/messagequeue.png)
![]({{site.baseurl}}/images/messagequeuetransfer.png)

消息协议：

![]({{site.baseurl}}/images/versionedmessagequeue.png)

![]({{site.baseurl}}/images/versionedmessagetransfer.png)

![]({{site.baseurl}}/images/versionedmessagetransfer2.png)

![]({{site.baseurl}}/images/lookup.png)

优化
连接层（参见通讯服务器组成）：只做消息转发，允许随时重启更新，设计原则简单/异步；单台压测试连接数70W；现状：1.5亿用户，月活5000W+，连接数1200W+；
逻辑层（参见通讯服务器组成）：用户会话验证即登陆、消息存取、异步队列
采用私有通讯协议，目标：高效，弱网络快速收发；可靠：不会丢消息；易于扩展；参考协议格式：REDIS协议；参见协议格式、基于队列的消息协议、基于队列的交互、基于版本号的消息协议、基于版本号的交互等；
核心的长连接只用于传输轻量的实时数据，图片、语音等都开新的TCP或HTTP连接；一切就绪后，最重要的就是监控，写一个APP查看所有的运营状态，每天观察；如何选择最优路线，即智能路由；
二、智能路由、连接策略：
多端口、双协议支持
应对移动网关代理的端口限制
支持TCP、HTTP两种协议
根据备选IP列表进行并发测速（IP+端口+协议）后端根据终端连接情况，定时更新终端的备选IP列表终端在连接空闲时上报测速数据，便于后端决策TCP协议不通，自动切换到http优先使用最近可用IP并发测速，根据终端所处的位置下发多组IP、PORT，只用IP，不用域名，手机上的DNS50%不准

负载均衡器(LVS...)的问题– 单点失效
单点性能瓶颈
负载均衡从客户端开始做起• 域名负载的问题
域名系统不可靠– 更新延迟大  

WNS（wireless network services）

![]({{site.baseurl}}/images/wns.png)

1解决移动互联网开发常见问题：
通道：数据交互、大数据上传、push
网络连接：大量长链接管理、链接不上、慢、多地分布
运营支撑：海量监控、简化问题定位
登录&安全：登录鉴权、频率控制
![]({{site.baseurl}}/images/secure.png)
![]({{site.baseurl}}/images/other.png)

移动互联网特点：
1、高延时：  信道建立耗时（ 高RTT）
2、低宽带、高丢包
3、多运营商（电信，移动，联通等）
4、复杂网络  
    －2G，3G，4G，wifi。cmwap，cmnet。。
    －网关限制：协议，端口
5、用户流动性大，上网环境复杂


WNS 性能指标：
![]({{site.baseurl}}/images/improve.png)
![]({{site.baseurl}}/images/improve2.png)
![]({{site.baseurl}}/images/basicdesign.png)
![]({{site.baseurl}}/images/scala.png)
1、开发时间：历史一年半
2、链接成功率－99.9%
3、极端网络环境下成功率－由于常见app
4、crash率 －0.02%（crash次数／登录用户数

微信后台系统架构
================

背景：
A、分布式问题收敛
  后台逻辑模块专注逻辑，快速开发
可能读取到过时的数据是个痛点
需要看到一致的数据
B、内部定义
  数据拥有两个以上的副本
如果成功提交了变更，那么不会再返回旧数据
推演：
1增加一个数据
2 序列号发生器，偏序
 约束：只能有一个client操作
client有解决冲突的能力
问题转移：client如何分布？
3 修改集群中一个制定的key的value
1）覆盖他
2）根据value的内容做修改
if value ＝ 1 then value ：＝2 
通用解法：
1）paxos算法
 工程难度
一切可控

分布式算法设计：
 2）quorum算法（2011）
再单个key上面运算
真是系统约束
类paxos方案，简化
为每次变更选举（by key）
算法过程
 提议／变更／同步／广播

![]({{site.baseurl}}/images/weixinarchitecture.png)

![]({{site.baseurl}}/images/weixinwrite.png)


 Replication & Sharding 
 权衡点
  自治，负载均衡，扩散控制
 replication－>relation

容灾抵消
同城（上海）多数派存活
三园区（独立供电，独立）


Sharding

一组KV6 为一个单位

1、人工阶段
 局部扩容，影响收敛9
2均匀分布 制定分段hash32 （string）
 翻倍扩容

3一致性哈希
具体实现？
1、业务侧快速开发
 存储需要提供强一致性
丰富的数据模型支持（结构化、类SQL／KV）
条件读，条件写

2 业务增长迅速，系统要能够方便的横向扩容
3设备故障／短时节点实效便成为常态，容灾自动化，主碑可写无需人工介入
4小数据


存储模型
 纯内存
Bitcask
小表系统
LSM－tree

![]({{site.baseurl}}/images/store.png)

![]({{site.baseurl}}/images/store2.png)

小表系统
1、解决放大问题
2、数据按变更聚集存储
3、Affected1
   ChangeTable
（1＋2+。。。。＋n－1+total）／n
4、分裂与合并
数据流动
1、自动化迁移
2、节点同时做代理
3、合并磁盘io

同步流量
1、数据vs 操作
2、幂等
3、保底策略

通信包量
 1、动态合并
    100K qps
    200% －10%
3、权衡与估算
设计要点
1、吞吐量
2、异步化
3、复杂度
4、libco
自动修复系统
1、不要让错误累计
2、全量扫描
bitcask 的一些变化
1、内存限制
2、全内存


