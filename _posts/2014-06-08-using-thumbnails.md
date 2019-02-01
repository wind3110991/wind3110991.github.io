---
layout:     post
title:      HTTPS，为用户带上安全【套】
date:       2018-12-31 02:32:18
summary:    简析HTTPS
categories: https
thumbnail: jekyll
tags:
 - https
 - tcp/ip
---

![Thumper](https://wildtanzaniasafari.com/wp-content/uploads/2018/12/HTTPS.png)


### 一、国际惯例——概念

我们还是要给一个基本概念：  

HTTP 协议（HyperText Transfer Protocol，超文本传输协议）：是客户端浏览器或其他程序与Web服务器之间的应用层通信协议。  
HTTPS 协议（HyperText Transfer Protocol over Secure Socket Layer）：可以理解为HTTP+SSL/TLS， 即 HTTP 下加入 SSL 层，HTTPS 的安全基础是 SSL，因此加密的详细内容就需要 SSL，用于安全的 HTTP 数据传输。

周所周知，在登录网页时，https不就是比http多了一个s吗？但是一个简单的字母，则隐藏了太多的东西与知识点在背后。

### 二、图解

我们在使用http协议时，HTTP直接与TCP通信，当使用SSL时，则需要先与SSL通信，然后再由SSL和TCP通信。 如下：

![Thumper](http://ww1.sinaimg.cn/large/afce444dgy1fzqx9fnlpbj20c9068jrg.jpg)

对于加密算法我不做过多赘述，我将其按方式分两种：
1、共享密钥加密：加密与解密使用同一个密钥 
2、公开密钥（非对称密钥）：公开密钥使用一对非对称密钥。一把叫私有密钥，一把公有密钥；  

### 三、谈谈实现 

那我们现在来想想，我们分别用上述两种方式来实现https，会有什么好处和坏处呢？

#### 1、共享密钥加密

比如，王二狗要把自己的银行卡密码告诉杨铁柱：

![Thumper](http://ww1.sinaimg.cn/large/afce444dgy1fzqxpms2dvj20br0bi75m.jpg)
![Thumper](http://ww1.sinaimg.cn/large/afce444dgy1fzqxq1an8gj20ck09pq4h.jpg)


这个不用说，显然傻子都知道不行，在http通信过程中，加密和解密都会用到同一个密钥【喵喵喵】，只要找到对应加密的方式，我们是可能解析出其真正的内容的。
如果密钥被攻击者截取获得，此时加密就失去了意义。  

![Thumper](http://ww1.sinaimg.cn/large/afce444dgy1fzqxra7rrcj20bf0am0tx.jpg)

#### 2、非对称密钥

![Thumper](http://ww1.sinaimg.cn/large/afce444dgy1fzqxukcxx2j20bv0b4gmo.jpg)

what？那你现在还想把密钥直接告诉所有人咯？  
非也非也，非对称加密又称共享密钥加密，使用一对非对称的密钥，一把叫做`私有密钥`，另一把叫做`公有密钥`；公钥加密只能用私钥来解密，私钥加密只能用公钥来解密。常见的公钥加密算法有：RSA、ElGamal、背包算法、Rabin（RSA的特例）、迪菲－赫尔曼密钥交换协议中的公钥加密算法、椭圆曲线加密算法）。

![Thumper](http://ww1.sinaimg.cn/large/afce444dgy1fzqxy69tpcj20ci0bbad4.jpg)

哦~ 那这个听起来蛮靠谱的嘛嘻嘻！我王二狗再也不用怕密码被偷听了！