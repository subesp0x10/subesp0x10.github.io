---
title: Malware Analyse 2 --- mpsvc
date: 2019-12-22 17:13:57
tags: reverse
categories: reverse
---
恶意程序样本：
{% asset_img sample.png sample%}  
利用签名程序微软Microsoft Malware Protection的DLL劫持
<!--more-->
## mpsvc.dll分析
### IDA分析
导出函数有<font color=blue>DllMain</font>和<font color=blue>ServiceCrtMain</font>
DllMain有些垃圾代码
{% asset_img dllmain.png dllmain%}  
实际上就是 
```c
return 1
```

ServiceCrtMain：
一段反调试代码
{% asset_img antidebug.png antidebug%}  
检查MsMpEng.exe模块地址+0xE730处的值是否为0x4cbeh，只有当前进程为MsMpEng.exe才能继续执行
### 动态调试
使用OllyDBG的call explort功能直接对DLL调试
{% asset_img callexport.png%}  
把反调试部分改为nop
{% asset_img nop.png%}  
进入之后，首先获取shellcode：读取当前目录下的mpsvc.ui文件。从开始到偏移0x217是一个字符串，\0之后是shellcode开始
使用strlen计算字符串的长度，返回shellcode起始地址
{% asset_img readfile.png%}  
进入shellcode
{% asset_img jump_to_shellcode.png%} 
很多花指令，遇到call一律F7
{% asset_img junkcode.png%}  
call $+5，取EIP
{% asset_img geteip.png%}  
一路经过很多call之后，最后终于call到shellcode末尾，开始寻找kernel32.dll以及所需的函数，搜索了VirtualAlloc，VirtualFree，VirtualProtect，RtlDecompressBuffer，memcpy
{% asset_img search_kernel32.png%}  
搜索函数之后，解密了一段长度为0x10的代码，解密的起始地址是调用当前函数的call指令的下一条指令
{% asset_img decrypt1.png%}  
{% asset_img decrypt2.png%}  
{% asset_img decrypt3.png%} 
转换成C语言
```c
DWORD x = *(DWORD*)begin;
DWORD a,b,c,d = x;
for(int i=0;i<16;i++)
{
    a = a + (a >> 3) - 1448498774;
    b = b + (b >> 5) - 909522486;
    c = -127 * c + 1465341783;
    d = -511 * d - 1987475063;
    begin[i] = (a + b + c + d) ^ *((char *)begin + i);
}
```
解密的4个字节，第1个再加0x10是第一次调用VirtualAlloc时的大小，第2个是第二次调用VirtualAlloc时的大小
第一次调用VirtualAlloc
{% asset_img virtualalloc1.png%}  
之后是同样的解密算法，解密同样的起始地址，解密长度为第1个VirtualAlloc的大小

第二次VirtualAlloc后，调用RtlDecompressBuffer解压，起始地址为之前解密地址+0x10
{% asset_img rtldecompressbuffer.png%}  
接下来就比较有意思了，实际上解压后已经是一个PE文件了，附近已经能够看到.text，.rdata字样
结合代码也可以看出这里开始就是PE加载的过程
{% asset_img peheader1.png%}  
{% asset_img peheader2.png%}  
PE加载过程就不仔细看了
最后可以把PE文件提取出来，修复PE头。大概看了一下，是个自带提权功能的DLL木马





