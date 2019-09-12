---
title: SSL
tags:
  - SSL
categories:
  - WEB安全
  - SSL
abbrlink: 5325
---

![](https://sslcity.com/wp-content/uploads/2019/03/ssl.png)

## 什么是SSL
* Secure Sockets Layer 安全套接层 
* 为Netscape所研发，用以保障在网络上的数据传输安全
* 一般通用的规格为40 bit安全标准
* SSL协议位于TCP/IP协议与各种应用层协议之间
* SSL协议分为两层：SSL记录协议（SSL Record Protocol）和 SSL握手协议 （SSL Handshake Protocol）。前者提供数据封装，压缩，加密等基本支持；后者用于传输之前双方的身份认证，协商加密算法，交换加密密钥等
* SSL记录协议处于表示层，SSL握手协议会话会话层。所以从网络层级上讲，SSL处于TCP/IP和应用层之间。TCP处于传输层，IP处于网络层。
![image](http://images.cnitblog.com/kb/1/201401/05125746-760ce000eb624a4f9a7d0d1d0b71ec0a.png)

## SSL协议的作用
* 认证用户和服务器，确保数据发送到之前的客户机和服务器
* 加密数据以防止数据中途被窃取
* 维护数据完整性，确保数据在传输过程中不被改变

## SSL协议的工作流程
服务器认证阶段：
* 客户端向服务器发送一个开始信息“Hello”以便开始一个新的会话连接；
* 服务器根据客户的信息确定是否需要生成新的主密钥，如需要则服务器在响应客户的“Hello”信息时将包含生成主密钥所需的信息；
* 客户根据收到的服务器响应信息，产生一个主密钥，并用服务器的公开密钥加密后传给服务器；
* 服务器恢复该主密钥，并返回给客户一个用主密钥认证的信息，以此让客户认证服务器。

## HTTPS和SSL的关系
* HTTPS是应用层协议（其实它并不是一种协议，准确的说），SSL也是应用层协议，但实际上工作在应用层和传输层之间，也就是说，HTTPS实际上是建立在SSL之上的HTTP协议

* SSL依靠证书来验证服务器的身份，并为浏览器和服务器之间的通信加密。

![image](https://zhangzifan.com/uploads/2016/12/HTTPS-HTTP-TLS-SSL.png)

## HTTPS
HTTPS(全称：Hyper Text Transfer Protocol over Secure Socket Layer)，是以安全为目标的HTTP通道，简单讲是HTTP的安全版。即HTTP下加入SSL层，HTTPS的安全基础是SSL，因此加密的详细内容就需要SSL。

![image](http://upload.chinaz.com/2017/0224/201702241048363689.jpg)

### HTTPS的工作原理：

*  客户端将它所支持的算法列表和一个用作产生密钥的随机数发送给服务器;
* 服务器从算法列表中选择一种加密算法，并将它和一份包含服务器公用密钥的证书发送给客户端;该证书还包含了用于认证目的的服务器标识，服务器同时还提供了一个用作产生密钥的随机数;
*  客户端对服务器的证书进行验证(有关验证证书，可以参考数字签名)，并抽取服务器的公用密钥;然后，再产生一个称作pre_master_secret的随机密码串，并使用服务器的公用密钥对其进行加密(参考非对称加/解密)，并将加密后的信息发送给服务器;
* 客户端与服务器端根据pre_master_secret以及客户端与服务器的随机数值独立计算出加密和MAC密钥(参考DH密钥交换算法)。

* 客户端将所有握手消息的MAC值发送给服务器;

* 服务器将所有握手消息的MAC值发送给客户端。

### HTTPS的优点与缺点
优点：

* 使用HTTPS协议可认证用户和服务器，确保数据发送到正确的客户机和服务器;

* HTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，要比http协议安全，可防止数据在传输过程中不被窃取、改变，确保数据的完整性。

* HTTPS是现行架构下最安全的解决方案，虽然不是绝对安全，但它大幅增加了中间人攻击的成本。

缺点：
* 相同网络环境下，HTTPS协议会使页面的加载时间延长近50%，增加10%到20%的耗电。此外，HTTPS协议还会影响缓存，增加数据开销和功耗。

* HTTPS协议的安全是有范围的，在黑客攻击、拒绝服务攻击、服务器劫持等方面几乎起不到什么作用。

* 最关键的，SSL 证书的信用链体系并不安全。特别是在某些国家可以控制 CA 根证书的情况下，中间人攻击一样可行。

* SSL的专业证书需要购买，功能越强大的证书费用越高。个人网站、小网站可以选择入门级免费证书。

* SSL 证书通常需要绑定 固定IP，为服务器增加固定IP会增加一定费用;

* HTTPS 连接服务器端资源占用高较高多，相同负载下会增加带宽和服务器投入成本;
* 

## SSL证书
SSL证书是数字证书的一种，类似于驾驶证、护照和营业执照的电子副本。因为配置在服务器上，也称为SSL服务器证书。SSL 证书就是遵守 SSL协议，由受信任的数字证书颁发机构CA，在验证服务器身份后颁发，具有服务器身份验证和数据传输加密功能。

SSL数字证书根据可信强度，大概可以分为以下几种：
* 域名型 SSL 证书（DV SSL）
* 企业型 SSL 证书（OV SSL）
* 增强型 SSL 证书（EV SSL）

证书可靠性(真实并可信)验证过程是一个信任链的验证过程，需要明确三点：
* 浏览器中预置了一些可信根证书；
* 证书的签名是由上级颁发者的私钥加密的；
* 信任链的验证终止与一个可信的证书。
![image](https://pic3.zhimg.com/50/v2-206007f16743a1d4e8963da6092a3586_hd.jpg)

## SSL免费证书申请及配置
* [Using Free SSL/TLS Certificates from Let’s Encrypt with NGINX](https://www.nginx.com/blog/free-certificates-lets-encrypt-and-nginx)

* [certbot使用说明](https://certbot.eff.org/docs/using.html#changing-a-certificate-s-domains)