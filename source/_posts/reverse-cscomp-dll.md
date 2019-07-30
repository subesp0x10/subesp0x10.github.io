---
title: Malware Analyse:cscomp.dll
date: 2019-07-29 07:00:36
tags: Reverse
categories: Reverse
---
此样本包含3个文件：csc.exe，cscomp.dll以及cscomp.rom。其中，csc.exe是微软.net组件，Visual C# Command Line Compiler；cscomp.dll为恶意DLL，利用DLL劫持，使csc.exe启动时被调用；cscomp.rom是一个加密文件。
## 运行
{% asset_img run.png run%}  
<!--more-->
结合WireShark抓包，可知程序为IP-MAC扫描软件
## cscomp.dll分析
### IDA分析
DLL有3个导出函数  
{% asset_img export.png DLL Export %}  
其中CreateCompilerFactory，InMemoryCompile反编译为  
```return 1```

因此只分析GetMessageDll  
  
#### step1
检查系统时间，如果是2013年之前则退出  
{% asset_img check-time.png %} 
#### step2 
用此方法获取需要的函数  
{% asset_img search-function.png %}  
#### step3
VirtualAlloc分配内存，读取cscomp.rom内容，解密后通过call eax进入解密代码
{% asset_img procedure.png %}  
~~TODO：分析解密函数~~
### 动态调试
用x64dbg调试，打开文件后，File->Change Command Line改写参数为-s 127.0.0.1  
一路走到call eax处，对比发现  
cscomp.rom开始部分（esi）的数据没有变化，而从偏移0x5F06AD（eax）处完成了解密  
{% asset_img call-eax0.png %} 
{% asset_img call-eax1.png %} 
#### step4
跟进call eax  
{% asset_img step-in-call-eax.png %} 
xor dex,0之后跳转到代码末尾，进一步解密 
解密算法为，从E8 00400700指令的下一条指令开始，xor 0x72，共0x74000 bytes  
{% asset_img decrypt2.png %}  
解密完成后又一次call eax回到开头
{% asset_img 4D-5A.png %}  
看到4D，5A了，用Scylla插件把这部分内存DUMP出来（此处地址与前面不一样，是因为不小心跑飞了）  
{% asset_img mem-dump.png %}  
使用CFF Explorer查看，是一个DLL，只有一个导出函数Loader  
{% asset_img CFF-Explorer.png %}  
IDA查看Loader函数  
{% asset_img loader.png call Loader%}  
与meterpreter一样，是一个reflective loader，事先已计算好Loader函数与当前指令地址，call ebx处，即是call Loader







