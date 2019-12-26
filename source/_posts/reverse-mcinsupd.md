---
title: Malware Analyse 4 --- stack trace
date: 2019-12-26 00:28:36
tags: reverse
categories: reverse
---
Sample:
{% asset_img sample.png sample%}  
<!--more-->
同[前文](https://subesp0x10.github.io/2019/12/25/reverse-bdreinit/)一样，修改了LoadLibrary后的返回地址，在技术上使用栈回溯，无需调用VirtualProtect

{% asset_img stack-trace1.png sample%}  
{% asset_img stack-trace2.png sample%}  

首先获取LoadLibrary的下一条指令的地址，然后通过栈回溯查找返回地址，如果匹配则修改为劫持函数的地址