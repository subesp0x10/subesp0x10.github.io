---
title: SSH Tunnel
date: 2019-10-03 00:02:11
tags: Note
categories: Note
---

## 0x00 动态端口转发
ssh -fNDg 1080 -p 45678 root@x.x.x.x

-f 后台运行

-N 不执行远程指令

-D 动态端口转发

-g 监听0.0.0.0:1080，可内网共用

然后就可以用socks5连接本地127.0.0.1:1080了

## 0x01 本地端口转发
ssh -L 1081:x.x.x.x:3128 -fNg root@x.x.x.x -p 45678

意思是：本地的1081端口，通过x.x.x.x这台服务器再转到x.x.x.x的3128端口

需要先在远程Linux服务器上搭一个Squid代理，并设置允许来自x.x.x.x的连接：

acl localnet src x.x.x.x

http_access allow localnet

（此处或许有更好的方法）

最后设置代理服务器为127.0.0.1:1080，协议为http

## 0x02 ssh免密登录
ssh-keygen -t rsa

生成：

私钥文件id_rsa

公钥文件id_rsa.pub

scp id_rsa.pub root@x.x.x.x:/home/User/.ssh/

cat id_rsa.pub >> authorized_keys

chmod 600 authorized_keys