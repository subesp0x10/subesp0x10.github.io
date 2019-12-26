---
title: WinPcap的IP报文分片/重组问题
date: 2019-10-03 00:12:43
tags: note
categories: note
---

做XX项目的一个程序，需要A电脑上的一个程序发送UDP数据到X地址，在B电脑上使用WinPcap抓到这些数据包。


一开始发现自己的程序抓下来的包打印出来不对（字符串），但是Wireshark能正常打印。
```c
pcap_next_ex(handle, &header, &pkt_data);
```
查看pkt_data内存发现每隔一段地址就会插入一些奇怪的数据，计算开头到奇怪数据的间隔，得到1472，Google之，发现是MTU：

以太网MTU是1500字节，1500-IP头（20字节）-（UDP头）8字节=1472，搞清楚问题出在哪里了，下面就是解决问题。
<!--more-->


一、简单粗暴，每次发送的UDP数据不要超过1472，经测试可行。

二、尝试自己重组？

1.调用1次pcap_next_ex（），处理完pkt_data指向的所有数据？

此时header.len=1514，即这个包的大小是1514。而pkt_data指向的内存看似包含了所有数据，上面说到的”奇怪数据“，是22字节的未知数据+ether_header+ip_header+udp_header，不好处理

2.调试发现，假设IP包分成10块，那么pcap_next_ex（）就会调用10次，所以1次处理完是不行的。但是这10次又有一个坑：

第一次，pkt_data指向ether_header+ip_header+udp_header+data，其中ip包的fragment_offset为0

第二次，pkt_data指向ether_header+ip_header+data，那么UDP头哪儿去了？fragment_offset为1480/8，正确

。。。

So 看起来像是这样：

那22字节可能是WinPcap组包分隔符，其实第一次已经抓完了，后面的10次循环是根据分隔符遍历。

不知道怎么插入大图片，点击看图  
{% asset_link 1.png Wireshark %}  
{% asset_link 2.png Code %} 


