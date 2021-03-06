---
title: Python模块解析 - optparser模块
date: 2016-02-11 19:03
tags: [Python,TECH]
show_copyright : true
---
最近博主有用到`optparser`这个module,其对命令行的参数解析功能还是很强大的

## 举个栗子
``` python
    from optparse import OptionParser

    parser = OptionParser()
    parser.add_option("-f", "--file", dest="filename",
                     help="write report to FILE", metavar="FILE")
    parser.add_option("-q", "--quiet",
                     action="store_false", dest="verbose", default=True,
                     help="don't print status messages to stdout")

    (options, args) = parser.parse_args()
```
<!--more-->
## 注释
首先创建一个`OptionParser`的class,在原码中可以发现这个类是继承`OptionContainer`,所以`OptionContainer`的非重名方法`OptionParser`都可以使用,下文用的`add_option()`就是用的父类`OptionContainer`中的方法.
创建好类的对象后,调用的是`add_option()`对命令行参数进行定义.
最后通过parse_args()解析命令汗参数.

最后一步返回的是一个元组,元组的内容可以从原码中得到:

``` 
def parse_args: return self.check_values(values, args)-> def check_value: return (value,args)
```


value存的就是参数对应的值,不难发现,value是一个字典类型,由此我们也可以明白dest的作用就是作为字典的key值去索引对应的结果,举个例子:
假如命令行为:
``` 
xx.py -f test.txt
```
那么-f 后面的值就能通过`option.filename`去索引得到
args在原码中默认为`none`,他的作用是可以通过程式写入的方式控制命令行参数.

博主用的时候就是以原码中的这个例子开始的,当然其中很重要的一步就是方法`add_option()`中的参数理解：

	"-f","--file"和"-q""-quiet"的作用就是参数的名称,前者是后者的短参数名,运行的时候用-q或者--quiet的结果一样
	dest的作用上文已经说过,就是字典的key值.
	action的值从原码中看有"store","store_const","store_true","store_false"等:

		store的作用就是将命令汗参数后面的值赋给option对应的value,此为默认的方式.

		store_false和store_true的作用是当参数后面没有值的时候,如博主要查看程式的版本号,不需要在-v后面加上任何的参数,
		如果用store的话就会报错.store_false和store_true两个刚刚相反,当命令行参数中有当前参数时,那么store_false的
		方式得到的option.verbose为false,store_true相反

	default的作用就是给没有出现在命令行的参数赋值,如果命令行有此参数的话就会覆盖掉default的值
	type的作用是给参数的value定义类型,默认为string

help的作用很帅,用xx.py --help 或者xx.py -h的话,他会自动生成help文档,如上面的提到的例子得到的help文档结果：

``` 
            Usage: optparser_test.py [options]

            Options:
              -h, --help            show this help message and exit
              -f FILE, --file=FILE  write report to FILE
              -q, --quiet           don't print status messages to stdout
```

## 扩展
还有一点要提到的就是`optparser`有专门对version的提供了方便的方法,只要在`parser = OptionParser()中加入version`的值就好:
``` 
parser = OptionParser(version = "v1.0")
```
然后命令行:
``` 
xx.py --version
```

That's all,Thank you
