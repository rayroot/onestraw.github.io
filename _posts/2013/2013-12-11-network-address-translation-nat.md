---
layout: single
author_profile: true
comments: true
title: NAT技术实现私有IP和公有IP通信
categories: [essay]
tags: [随笔]
---

仔细思考一个问题

> 学校或公司内部（使用私有IP）的电脑如何和Internet上的公用IP进行通信？  
  你用私有IP发送的数据包，google服务器怎么收到的？   
  你的私有IP主机如何收到google服务器的应答的？   

###网络地址转换 NAT 

Network Address Translation，简称NAT  
别名：网络掩蔽、IP掩蔽（IP masquerading）   

wiki的定义：  

> “是一种在IP封包通过路由器或防火墙时重写源IP地址或/和目的IP地址的技术”

使用NAT的目的是暂时解决IPv4地址不足。   

###私有IP和公用IP通信实例 

假设学校内部的电脑都是使用的私有IP地址，公用IP以google服务器为例。 

> 校园网的一个私有IP: 192.168.0.1   
  校园网的出口NAT网关的公用IP：210.14.146.7，网关的私有IP：192.168.0.254  
  google服务器的一个公用IP：74.125.128.105   

1.现在要从192.169.0.1发送一个请求数据包到google服务器  

该数据包源地址 192.169.0.1：8888   
目的地址 74.125.128.105:80  
本地主机发现目的IP 74.125.128.105不是本网段内，   

首先使用ARP协议获得192.168.0.254对应网卡的MAC地址，
将数据包发送到网关路由器

2.NAT网关收到该数据包后，转发是肯定的，因为源IP是私有IP，要进行NAT处理   

NAT网关路由器维护一个IP端口映射表——NAT表，表项格式如下   

> 私有IP：私有IP使用的端口 <==> NAT网关的公用IP：NAT网关临时分配的端口

所以数据在转换前，源地址会被替换成如下(端口号是我假定的一个)  

> 210.14.146.7:1234

目的地址不变

现在将其发向google服务器，中间经过再多路由器，也不会再改变源IP和目的IP

3.google服务器收到请求之后，返回一个应答数据包  

该数据包源地址 74.125.128.105: 80   
目的地址 210.14.146.7: 1234  

4.校园网的NAT网关收到应答数据包后   

解析目的IP及端口，在NAT映射表中查找对应的私有IP  
将应答数据包的目的地址替换成

> 192.169.0.1: 8888  

然后从网关的192.169.0.254对应的网卡向内网转发   
最后我们192.169.0.1电脑就收到的应答。   

###总结

从上面过程也可以看出，本来两个过程可以完成的通信，使用NAT需要4个过程，效率低下。  
当两个不同学校的同学之间聊QQ，它通过了QQ的服务器，相当于两个上述过程。  
IPv6从根本上解决了IP地址不足的问题，也就不需要NAT技术了。  

