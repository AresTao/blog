---
layout: post
title: kamailio使用认可的ca证书支持tls及wss
keywords: kamailio
category : 架构
tags : [架构]
---

kamailio是广泛使用的SIP proxy，支持udp/tcp/tls/ws/wss类型接入，并发性良好。
但是，网上对kamailio对于tls支持配置的文档很少，官网上的介绍就是简单的配置私有的ca证书，这种证书客户端一定要先同意才能连kamailio，对于在js中调用wss连接kamailio并不适用。这种调用场景不会弹出是否同意的框，也就意味着通过js访问不了。所以kamailio使用的证书必须要被浏览器默认认可。

比较好的方式是从starcom上申请免费又被浏览器认可的证书。
地址：[https://www.startssl.com/](https://www.startssl.com/)，这里需要说明，认可的证书有个要求，就是证书要对应一个域名，这个域名使用者要有管理权限，以test.com域名为例，starcom的域名认真方式是会往postmaster@test.com邮箱发送一封认证邮件，这意味着要为test.com新增一条A记录，mail.test.com，在对应的机器上搭建一个邮件服务器，推荐postfix，一般系统默认就有，网上找一下搭建的方案。

域名认证成功后，就可以添加ca证书了，免费的证书使用期是一年，假如域名为demo.test.com，那么要单独为这个域名申请一个证书。最后的服务一定要用这个域名，否则还是不认可的。

申请好的证书包括一个私钥.key文件，还有公钥文件，提供针对apache/IIS/Nginx/Other四种公钥。

kamailio用的是Other里的公钥。

假如得到的证书文件为：demo.test.com.key demo.test.com.crt。

找到kamailio配置文件路径etc/，将文件拷贝进来，在tls.cfg中存放tls的配置。把对应的证书文件进行替换。

启动kamailio会发现启动不了，把kamailio的debug调为0来输出错误信息，会看到读入private_key出错，unsupported cipher，不支持的密码。

原因是在starcom上申请的证书都是有密码的，即使用时该证书时要输入认证一下你对该证书的拥有权。但是kamailio是不支持这种证书的。

所以，要将证书的密码去掉。

执行 openssl rsa -in demo.test.com.key -out demo.test.com.key

重新启动，应该就OK了。

访问的时候一定记得用域名别用ip，否则跟私有证书一样了。

