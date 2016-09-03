---
title: Python函数式编程 - zip| map| filter| reduce
date: 2016-07-13 19:21
tags: [Python,函数式编程,TECH]
show_copyright : true
---
Python中有很多函数式编程的方法,本文介绍的是最常见的几个函数
注:本文基于python2.7.不同的版本可能会导致差异

## Zip - 集资
标准格式`zip(seq[, seq[, seq]])`
举个栗子:
```python
a = (i for i in range(3))
b = [4, 5, 6]
c = (7, 8, 9)
abc = zip(a, b, c)
```
abc的结果为`[(0, 4, 7), (1, 5, 8), (2, 6, 9)]`
所以zip函数的作用已经很明显了,它就好比abc三家集资,一家出一个.
那当c家很富,a家没那么富怎么办呢?
```python
a = (i for i in range(2))
b = [4, 5, 6]
c = (7, 8, 9, 10)
abc = zip(a,b,c)
```
abc三家集资的结果为`[(0, 4, 7), (1, 5, 8)]`,说好的一家出一个,你家没有了,那我也就不出了(你看,代码也这么现实...)

之所以举的栗子不全是list,就是想说明一个小的注意点,zip接受的都是具有迭代的属性的参数(`__iter__`),这个后面会讲

<!--more-->

## Filter - 审查
标准格式`filter(func, seq)`
举个栗子:
```python
a = [1, 2, 3]
b = [4, 5, 6]
c = [7, 8, 9]
result = filter(lambda x:sum(x)>14, [a, b, c])
```
result为`[[4, 5, 6], [7, 8, 9]]`
现在有一个项目,但是这个项目要有个资产审查,要求资产满足lambda标准的企业才有资格参加招标(func是一个bool类型的函数)

## Map - 招标
标准格式`map(func, seq[, seq[, seq]])`
举个栗子:
```python
a = (i for i in range(3))
b = [4, 5, 6]
c = (7, 8, 9)
abc = map(lambda x,y,z: x+y+z, a, b, c)
```
abc的结果为`[11, 14, 17]`
还是这个招标的项目`lambda函数`需要招三家(函数的参数个数)来参与这个项目,这个项目分前中后三期的投入.
假如abc三家幸运的入选,并且三家都已经准备好了三个阶段的投入.所以前期a出0,b出4,c出7投入到lambda项目,项目前期返回收入11
依次类推,直到三家的投入全部结束,所以返还的收入就是`[11, 14, 17]`

假如c有钱(`c = [7,8,9,10,11]`),c说:老子有钱,我要追加投入. 那ab没钱怎么办?简单!!ab就诚实的说:老子没钱 就好了(`a = [0,1,2,None,None], b = [4,5,6,None,None]`)
所以lambda项目就收到到这样的投入`[None,None,10], [None,None,11]`,这里要注意处理这些None,因为不小心他就会报NoneType的问题
```python
a = (i for i in range(3))
b = [4, 5, 6]
c = (7, 8, 9, 10, 11)
abc = map(lambda x,y,z: str(x)+str(y)+str(z), a, b, c)
```
所以abc的结果为`['047', '158', '269', 'NoneNone10', 'NoneNone11']`
map接受的也都是具有迭代的属性的参数
注意,当func为None的时候,map的功能就类似与zip了.说到底map也就是先集资(zip)在干事(func),func不存在的时候就剩集资(zip)了

## Reduce - 二次投入
标准格式`reduce(func, seq[, init])`
举个栗子:
```python
a = (i for i in range(3))
result = reduce(lambda x,y: x+y, a)
```
result为3
reduce的工作流程为:
	1.`func(seq[0], seq[1])`                          ->seq的第一和第二元素应用到func
	2.`func(func(seq[0], seq[1]), seq[2])`            ->上一步func的结果作为func的第一个参数,seq第三个参数作为func的第二个元素
	......
	n.`func(...func(func(seq[0], seq[1]))...,seq[n])` ->上一步func的结果作为func的第一个参数,seq第n个参数作为func的第二个元素
下图可以很好的说明这个过程:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;![](/images/reduce.png)

以上是init不存在的时候,当init存在的时候,第一步的func的第一个参数为init,seq的第一个元素为func的第二参数,后面流程就一样了
这就好比把第一阶段的盈利当作二次投资的资本在进行投资


## 结语
在python中函数式编程能很大程度上节约我们的开发时间,利用好了就会是自己的一大利器
文中提到的`好比`在可能会有不当,毕竟我不是经济专业的....海涵

That's all,Thank you
