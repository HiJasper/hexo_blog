---
title: Python新式类和经典类的继承区别
date: 2015-12-12 19:03
tags: [Python,TECH]
show_copyright : true
---
## 写法区别
经典类写法:
``` python
class classA():
	pass
```

新式类写法:
``` python
class classB(object):
	pass
```

## 继承区别
经典类就好比是在好几个单元楼里面找一户人家,所以你的查找方法是从第一栋单元楼开始从最高一层一直往最低一层找,没找到的话再去下一栋找.这里每一栋就表示一个父类,所以经典类是深度优先
新式类就好比好几个单元合在一起,每一个垂直方向的所有的房间构成一个父类,所以这时候当你要找一户人家的时候,你就会一层一层的找,没一层都有一个父类的一个房间,你只有先确定同一级的每个父类的每个房间都没有找到你,你才能去下一层找
<!--more-->
举个经典类的栗子:
``` python
class classA():
	def sayhello(self):
		print "hello from class classA"

class classB():
	def sayhello(self):
		print "hello from class classB"

class A(classA, classB):
	pass

class B(classA, classB):
	def sayhello(self):
		print "hello from class B"

class C(A, B):
	pass

tt = C()

tt.sayhello()
```
得到的结果是"hello from class classA",所以继承的顺序为:C->A->classA,一直是在深度的方向延伸

还是那个栗子,只不过将它改成新式类的写法:
``` python
class classA(object):
	def sayhello(self):
		print "hello from class classA"

class classB(object):
	def sayhello(self):
		print "hello from class classB"

class A(classA, classB):
	pass

class B(classA, classB):
	def sayhello(self):
		print "hello from class B"

class C(A, B):
	pass

tt = C()

tt.sayhello()
```
得到的结果是"hello from class B",所以继承的顺序为:C->A->B,是先在广度上延伸,然后是深度上的延伸

## 结语
因为新旧类的区别,所以在多类继承的时候同名方法的处理要注意了

That's all,Thank you
