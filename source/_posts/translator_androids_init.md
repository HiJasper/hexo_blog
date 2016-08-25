---
title: Android init.rc的readme.txt翻译
date: 2015-07-25 16:03
tags: [Translator]
show_copyright : true
---
这是一篇自己胡乱翻译的,翻译的可能会有错误,但是以自己目前对kernel的了解只能做到这样了,希望以后自己会有所成长,然后再次看到这篇翻译的时候莞尔一笑.

## Android Init Language
Android Init语言包含四种类型的关键字:`Action`,`Commands`,`Services`和`Options`
以#开头的行是注释
`Action`和`Services`声名一个新的section,所有的`commands`和`options`都属于靠的最近的section.在第一个section之前的`commands`和`services`都被忽略
每一个`Actions`和`Services`都有独立的名字,如果有相同名字的`actions`和`services`被声名,那这个section将被当成一个错误忽略掉.

<!--more-->
### Actions
`Actions`就是一组commands的队列,它有一个用于决定何时执行此`action`的触发器,当`actions`的触发器满足条件的时候,此`actions`被添加到执行队列（除非该队列已经有了此action）
每一个被添加到队列中的`action`都是按顺序被移除队列,每一个此action中`command`都会被顺序执行.Init通过执行actions中的`commands`来处理其他活动（如设备创建/销毁,属性设置,重新启动和平进程)

`Actions`的形式如下:
``` code
on <trigger>
   <command>
   <command>
   <command>
```

### Services
`Services`是初始化加载和重启时候的程序,`Services`的格式如下:
``` code
service <name> <pathname> [ <argument> ]*
   <option>
   <option>
   ...
```

### Options
`Options`可以修改`services`的属性,它们决定init何时以何种方式执行service
``` code
critical
这是一个设备临界service,如果它在4分钟内存在超过4次,那该设备会reboot到recovery模式

disabled
有disabled参数的services不能用class start 的方式执行,只能通过start service name的方式执行.

setenv <name> <value>
就是设置环境变量

socket <name> <type> <perm> [ <user> [ <group> ] ]
在/dev/socket下创建一个unix domain类型的socket,并把描述符fd传给进程,type必须是"dgram", "stream" 
或者 "seqpacket".User和group默认为0.

user <username>
执行该service之前切换用户,目前默认是root.

group <groupname> [ <groupname> ]*
执行该service之前切换group,目前默认是root.

oneshot
当service存在,该service不会重启

class <name>
特殊的class name,可以通过class_start和class_stop同意启动和停止,默认是default.
（PS：注意上面提到过的disabled参数）

onrestart
service重启时执行一条command
```

### Triggers
触发器是一个字符串,它可以用来匹配特定类型的事件,并用来导致一个动作发生
``` code
boot
这是当init开始后的第一个触发器

<name>=<value>
当name被设置成指定的<value>的时候,这个触发器被触发.

device-added-<path>
device-removed-<path>
当一个设备节点被添加或者被移除的时候触发该触发器.

service-exited-<name>
当指定的services存在的时候触发该触发器
```

### Commands

``` code
exec <path> [ <argument> ]*
给定参数,执行path下的程序

export <name> <value>
设置环境变量,且设置的环境变量对所有进程有效

ifup <interface>
激活network指定接口

import <filename>
解析新的config文件,扩展当前的配置档案.

hostname <name>
设置host name

chdir <directory>
改变工作目录

chmod <octal-mode> <path>
改变文件的权限

chown <owner> <group> <path>
改变文件的ower和group

chroot <directory>
改变进程的根目录

class_start <serviceclass>
启动所有class name是指定class name的services(注意上面提到的disabled的参数)

class_stop <serviceclass>
停止所有class name为指定class name的services

domainname <name>
设置域名

insmod <path>
安装模块到指定path

mkdir <path> [mode] [owner] [group]
指定路径下,按照给定的mode,owner和group创建目录,如果没有指定,则创建的目录权限为755

mount <type> <device> <dir> [ <mountoption> ]*
挂载

setkey
待定

setprop <name> <value>
设置系统属性

setrlimit <resource> <cur> <max>
设置资源限制

start <service>
启动一个service,前提是该service还没有被启动

stop <service>
停止一个service,前提是该service正在running

symlink <target> <path>
创建一个符号链接

sysclktz <mins_west_of_gmt>
设置系统时间

trigger <event>
触发一个事件,可以从一个action去触发另外一个action

write <path> <string> [ <string> ]*
往打开的指定文件写字符串
```

### Properties
Init updates some system properties to provide some insight into what it's doing:
``` code
init.action 
等于当前执行的action的名字,如果没有没有则为空

init.command
等于当前被执行的command,如果没有则为空

init.svc.<name>
等于指定service的当前状态(包括"stopped", "running", "restarting")
```

That's all,Thank you
