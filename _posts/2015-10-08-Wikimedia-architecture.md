---
layout: post
title: Wikimedia架构
keywords: 架构设计
category : 架构
tags : [架构设计]
---

维基百科(WikiPedia.org )位列世界十大网站，目前排名第八位。这是开放的力量。

来点直接的数据：

    * 峰值每秒钟3万个 HTTP 请求
    * 每秒钟 3Gbit 流量, 近乎375MB
    * 350 台 PC 服务器

(数据来源 )
架构示意图如下
![]({{site.baseurl}}/images/wiki.png)

GeoDNS

"A 40-line patch for BIND to add geographical filters support to the existent views in BIND", 把用户带到最近的服务器。GeoDNS 在 WikiPedia 架构中担当重任当然是由 WikiPedia 的内容性质决定的--面向各个国家，各个地域。

负载均衡：LVS

WikiPedia 用 LVS 做负载均衡, 是章文嵩博士发起的项目,也算中国人为数不多的在开源领域的骄傲啦。LVS 维护的一个老问题就是监控了，维基百科的技术人员用的是 pybal.
图片服务器:Lighttpd

Lighttpd 现在成了准标准图片服务器配置了。不多说。
Wiki 软件: MediaWiki
对 MediaWiki 的应用层优化细化得快到极致了。用开销相对比较小的方法定位代码热点，参见实时性能报告，瓶颈在哪里，看这样的图树展示一目了然。另外一个十分值得重视的经验是，尽可能抛弃复杂的算法、代价昂贵的查询，以及可能带来过度开销的 MediaWiki 特性。

维基百科网站成功的第一关键要素就是 Cache 了。CDN(其实也算是 Cache) 做内容分发到不同的大洲、Squid 作为反向代理. 数据库 Cache 用 Memcached，30 台，每台 2G 。对所有可能的数据尽可能的Cache，但他们也提醒了 Cache 的开销并非永远都是最小的，尽可能使用，但不能过度使用。
数据库: MySQL

MediaWiki 用的DB 是 MySQL. MySQL 在 Web 2.0 技术上的常见的一些扩展方案他们也在使用。 复制、读写分离......应用在 DB 上的负载均衡

运营这样的站点，WikiPedia 每年的开支是 200 万美元，技术人员只有 6 个，惊人的高效。

source: [http://highscalability.com/wikimedia-architecture](http://highscalability.com/wikimedia-architecture)
