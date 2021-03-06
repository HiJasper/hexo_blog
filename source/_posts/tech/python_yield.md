---
title: Python中关键字yield有什么作用
date: 2016-06-10 17:03
tags: [拿来主义,Python,TECH]
---
本文转自[Python中关键字yield有什么作用?](https://taizilongxu.gitbooks.io/stackoverflow-about-python/content/1/README.html)
***
例如下面这段代码:
``` python
def node._get_child_candidates(self, distance, min_dist, max_dist):
    if self._leftchild and distance - max_dist < self._median:
        yield self._leftchild
    if self._rightchild and distance + max_dist >= self._median:
        yield self._rightchild
```
下面是调用它:
``` python
result, candidates = list(), [self]
while candidates:
    node = candidates.pop()
    distance = node._get_dist(obj)
    if distance <= max_dist and distance >= min_dist:
        result.extend(node._values)
    candidates.extend(node._get_child_candidates(distance, min_dist, max_dist))
return result
```
<!--more-->
当`_get_child_candidates`方法被调用的时候发生了什么?是返回一个列表?还是一个元祖?它还能第二次调用吗?后面的调用什么时候结束?

为了理解yield有什么用,首先得理解`generators`,而理解`generators`前还要理解`iterables`
## Iterables

当你创建了一个列表,你可以一个一个的读取它的每一项,这叫做`iteration`:
``` python
>>> mylist = [1, 2, 3]
>>> for i in mylist:
...    print(i)
1
2
3
```
Mylist是可迭代的.当你用列表推导式的时候,你就创建了一个列表,而这个列表也是可迭代的:
``` python
>>> mylist = [x*x for x in range(3)]
>>> for i in mylist:
...    print(i)
0
1
4
```
所有你可以用在`for...in...`语句中的都是可迭代的:比如`lists,strings,files...`因为这些可迭代的对象你可以随意的读取所以非常方便易用,但是你必须把它们的值放到内存里,当它们有很多值时就会消耗太多的内存.

## Generators

生成器也是迭代器的一种,但是你只能迭代它们一次.原因很简单,因为它们不是全部存在内存里,它们只在要调用的时候在内存里生成:
``` python
>>> mygenerator = (x*x for x in range(3))
>>> for i in mygenerator:
...    print(i)
0
1
4
```
生成器和迭代器的区别就是用()代替[],还有你不能用`for i in mygenerator`第二次调用生成器:首先计算0,然后会在内存里丢掉0去计算1,直到计算完4.
## Yield

Yield的用法和关键字return差不多,下面的函数将会返回一个生成器:
``` python
>>> def createGenerator():
...    mylist = range(3)
...    for i in mylist:
...        yield i*i
...
>>> mygenerator = createGenerator() # 创建生成器
>>> print(mygenerator) # mygenerator is an object!
<generator object createGenerator at 0xb7555c34>
>>> for i in mygenerator:
...     print(i)
0
1
4
```
在这里这个例子好像没什么用,不过当你的函数要返回一个非常大的集合并且你希望只读一次的话,那么它就非常的方便了.
要理解Yield你必须先理解当你调用函数的时候,函数里的代码并没有运行.函数仅仅返回生成器对象,这就是它最微妙的地方:-)
然后呢,每当for语句迭代生成器的时候你的代码才会运转.

现在,到了最难的部分:
当for语句第一次调用函数里返回的生成器对象,函数里的代码就开始运作,直到碰到yield,然后会返回本次循环的第一个返回值.所以下一次调用也将运行一次循环然后返回下一个值,直到没有值可以返回.
一旦函数运行并没有碰到yeild语句就认为生成器已经为空了.原因有可能是循环结束或者没有满足if/else之类的.

对于你的代码的解释
生成器:

``` python
# 这里你创建node方法的对象将会返回一个生成器
def node._get_child_candidates(self, distance, min_dist, max_dist):

  # 这里的代码你每次使用生成器对象的时候将会调用

  if self._leftchild and distance - max_dist < self._median:
      yield self._leftchild

  if self._rightchild and distance + max_dist >= self._median:
      yield self._rightchild

  # 如果代码运行到这里,生成器就被认为变成了空的
```
调用:
``` python
# 创建空列表和一个当前对象索引的列表
result, candidates = list(), [self]

# 在candidates上进行循环(在开始只保含一个元素)
while candidates:

    # 获得最后一个condidate然后从列表里删除
    node = candidates.pop()

    # 获取obj和candidate的distance
    distance = node._get_dist(obj)

    # 如果distance何时将会填入result
    if distance <= max_dist and distance >= min_dist:
        result.extend(node._values)

    candidates.extend(node._get_child_candidates(distance, min_dist, max_dist))

return result
```
这段代码有几个有意思的地方:

1. 一般的时候我们会在循环迭代一个列表的同时在列表中添加元素:-)尽管在有限循环里结束多少有一些危险,但也不失为一个简单的方法去遍历嵌套的数据.在这里candidates.extend(node._get_child_candidates(distance, min_dist, max_dist))将遍历生成器的每一个值,但是while循环中的condidates将不再保存已经遍历过的生成器对象，也就是说添加进condidates的生成器对象只会遍历一遍.
2. extend()是一个列表对象的方法,它可以把一个迭代对象添加进列表.

我们经常这么用:
``` python
>>> a = [1, 2]
>>> b = [3, 4]
>>> a.extend(b)
>>> print(a)
[1, 2, 3, 4]
```
但是在你给的代码里得到的是生成器,这样做的好处:
1. 你不需要读这个值两次
2. 你能得到许多孩子节点但是你不希望他们全部存入内存.

这种方法之所以能很好的运行是因为Python不关心方法的参数是不是一个列表.它只希望接受一个迭代器,所以不管是strings,lists,tuples或者generators都可以!这种方法叫做duck typing,这也是Python看起来特别cool的原因之一.但是这又是另外一个传说了,另一个问题~~

好了,看到这里可以打住了,下面让我们看看生成器的高级用法:
控制迭代器的穷尽
``` python
>>> class Bank(): # 让我们建个银行,生产许多ATM
...    crisis = False
...    def create_atm(self):
...        while not self.crisis:
...            yield "$100"
>>> hsbc = Bank() # 当一切就绪了你想要多少ATM就给你多少
>>> corner_street_atm = hsbc.create_atm()
>>> print(corner_street_atm.next())
$100
>>> print(corner_street_atm.next())
$100
>>> print([corner_street_atm.next() for cash in range(5)])
['$100', '$100', '$100', '$100', '$100']
>>> hsbc.crisis = True # cao,经济危机来了没有钱了!
>>> print(corner_street_atm.next())
<type 'exceptions.StopIteration'>
>>> wall_street_atm = hsbc.create_atm() # 对于其他ATM,它还是True
>>> print(wall_street_atm.next())
<type 'exceptions.StopIteration'>
>>> hsbc.crisis = False # 麻烦的是,尽管危机过去了,ATM还是空的
>>> print(corner_street_atm.next())
<type 'exceptions.StopIteration'>
>>> brand_new_atm = hsbc.create_atm() # 只能重新新建一个bank了
>>> for cash in brand_new_atm:
...    print cash
$100
$100
$100
$100
$100
$100
$100
$100
$100
...
```
它对于一些不断变化的值很有用,像控制你资源的访问.
Itertools,你的好基友

itertools模块包含了一些特殊的函数可以操作可迭代对象.有没有想过复制一个生成器?链接两个生成器?把嵌套列表里的值组织成一个列表?Map/Zip还不用创建另一个列表?

来吧import itertools

来一个例子?让我们看看4匹马比赛有多少个排名结果:
``` python
>>> horses = [1, 2, 3, 4]
>>> races = itertools.permutations(horses)
>>> print(races)
<itertools.permutations object at 0xb754f1dc>
>>> print(list(itertools.permutations(horses)))
[(1, 2, 3, 4),
 (1, 2, 4, 3),
 (1, 3, 2, 4),
 (1, 3, 4, 2),
 (1, 4, 2, 3),
 (1, 4, 3, 2),
 (2, 1, 3, 4),
 (2, 1, 4, 3),
 (2, 3, 1, 4),
 (2, 3, 4, 1),
 (2, 4, 1, 3),
 (2, 4, 3, 1),
 (3, 1, 2, 4),
 (3, 1, 4, 2),
 (3, 2, 1, 4),
 (3, 2, 4, 1),
 (3, 4, 1, 2),
 (3, 4, 2, 1),
 (4, 1, 2, 3),
 (4, 1, 3, 2),
 (4, 2, 1, 3),
 (4, 2, 3, 1),
 (4, 3, 1, 2),
 (4, 3, 2, 1)]
```
理解迭代的内部机制

迭代是可迭代对象(对应__iter__()方法)和迭代器(对应__next__()方法)的一个过程.可迭代对象就是任何你可以迭代的对象(废话啊).迭代器就是可以让你迭代可迭代对象的对象(有点绕口,意思就是这个意思)

