---
title: Linux命令讲解 - ftp命令简介
date: 2015-02-02 18:03
tags: [Linux command,TECH]
show_copyright : true
---

## 连接ftp服务器

``` bash
ftp [ip_addr | hostname]
```
举个栗子：
``` bash
ftp 172.168.0.1
```
链接后会让你输入用户名和密码进行身份认证.
<!--more-->
## 下载文件

``` bash
get [ftp服务器文件路径] [本地路径]
mget [文件匹配规则]
```
get的命令是下载单个文件到指定本地路径.
mget则可以批量下载.

再举个栗子：
``` bash
ftp> mget *.txt //将服务器当前文件夹下的txt文档下载到本地当前路径
```

## 上传文件

``` bash
put [本地文件路径] [ftp服务器文件路径]
mput [文件匹配规则]
```
很容易理解,和get一样.只是需要注意的是,mput上传到服务器的路径是当前服务器路径,所以要想将文件上传到服务器指定路径的时候要先在ftp下cd的命令

## 退出链接

``` bash
ftp> bye
```

That's all,Thank you
