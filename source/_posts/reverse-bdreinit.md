---
title: Malware Analyse 3 --- bdreninit dll hijacking
date: 2019-12-25 18:00:41
tags: reverse
categories: reverse
---
恶意程序样本：
{% asset_img sample.png sample%}  
利用bitdefender组件<font color=blue>bdreinit.exe</font>的DLL劫持
<!--more-->
### IDA
导出函数：
{% asset_img export1.png sample%}  
aPxbGs863函数跳转到GetMessageTime，没有意义
{% asset_img export2.png sample%}  
分析DllMain，发现只是将bdreinit.exe进程基址+0x2775偏移处的6个字节修改为
```asm
push offset sub_10001500
ret
```
</br>
{% asset_img virtualprotect.png sample%}  

### 动态调试
修改的代码刚好位于LoadLibraryW(L"log.dll")之后
{% asset_img modifycode1.png sample%}  
修改后：
{% asset_img modifycode2.png sample%}  
这样，DllMain执行完就会跳转到sub_10001500里面去了