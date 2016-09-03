---
title: Python编写Phidget Relay控制
date: 2016-08-30 19:21
tags: [Python,Phidget,TECH]
show_copyright : true
---
这几天收拾的时候发现了Phidget，这还是上一个案子客户给的。型号是1014\_2，记得是当时拿来做压力测试用的。
过太久都忘了怎么用了，所以再拾遗一下。
因为重装`Ubuntu 16.04`的系统，所以就从头装一遍phidget在Linux平台上的运行环境。
可参考[Phidget官网](http://www.phidgets.com/docs/Language_-_Python#Follow_the_Examples)，这里只是大体说一下。

# Linux环境的搭建
#### 安装libusb的开发库
``` bash
	sudo apt-get install libusb-1.0-0-dev
```
#### 安装Phidget库
点击[下载路径](http://www.phidgets.com/downloads/libraries/libphidget.tar.gz)下载Phidget库。
解压后，依次执行:
``` bash
	./configure 
	make 
	sudo make install
```
<!--more-->
# 检查Phidget库的安装
#### Software:
下载官方提供的[示例](http://www.phidgets.com/downloads/examples/phidget21-c-examples.tar.gz)
解压后编译`HelloWorld.c`
``` bash
gcc HelloWorld.c -o HelloWorld -lphidget21
```
运行HelloWorld
``` bash
jasper@jasper:~/phidget/phidget21-c-examples-2.1.8.20151217$ sudo ./HelloWorld 
Opening...
Phidget Simple Playground (plug and unplug devices)
Press Enter to end anytime...
Hello Device Phidget InterfaceKit 0/0/4, Serial Number: 381507
Goodbye Device Phidget InterfaceKit 0/0/4, Serial Number: 381507

Closing...
```
正常情况下，伴随着phidget设备的插入/拔出，能对应的看到Hello/Goodbye的信息。

#### Hardware:
使用`kernal log`查看插入/拔出信息:
插入:
``` bash
jasper@jasper:~$ dmesg | tail
......
[17239.182460] usb 2-1.4: new low-speed USB device number 13 using ehci-pci
......
```
拔出:
``` bash
jasper@jasper:~$ dmesg | tail
......
[17262.852520] usb 2-1.4: USB disconnect, device number 13
......
```
正常情况下插入时用到ehci-pci的device number会在拔出时disconnect掉。

# Phidget的python套件安装
下载并解压phidget的[python套件](http://www.phidgets.com/downloads/libraries/PhidgetsPython.zip)
``` bash
jasper@jasper:~/phidget/PhidgetsPython$ sudo python setup.py install
```
可以下载[官方示例](http://www.phidgets.com/downloads/examples/Python.zip)验证。
至此，Phidget在Linux平台上的Python环境就安装好了。

# Phidget的使用步骤
Python API可参考[官方API文档](http://www.phidgets.com/documentation/PythonDoc.zip)
#### Step 1:设备的初始化和打开
``` python
self.device = InterfaceKit()
self.device.openPhidget()
```

#### Step 2:等待phidget设备的接入
``` python
self.device.waitForAttach(10000)
```

#### Step 3:对设备的操作
``` python
self.device.setOutputState(output, 1)
```

#### Step 4:设备的关闭
``` python
self.device.closePhidget()
```

# 实际操作
#### 手头材料
phidget板: 型号是1014_2(实际上就是4个relay),能做的无非就是当作开关控制线路。
小米电风扇: 小米官网可购，19块好像。
USB线: 延长线更好，能省去不少麻烦。
Micro USB线: 10年以后的Phidget 1014_2采用了Micro USB连接。

#### 实操
因为只是为了控制电风扇的开关，所以USB的四条线我们只用电源正级和接地就好，因为用不到数据部分，所以D+/D-就不用管了。
将USB小的那头剪去，并扯出红线和黑线。将红线剪下一半。然后按照下图连接Phidget和风扇就好。这里在连风扇的USB时很麻烦，有延长线的话就简单多了，但是我这里就一条，舍不得浪费....接风扇的时候连USB长的那两条即可，中间那两个是数据传输用的。
![](https://coding.net/u/jasper_xu/p/web_pic/git/raw/master/phidget1.JPG)
连好后的总体效果如图:
![](https://coding.net/u/jasper_xu/p/web_pic/git/raw/master/phidget2.JPG)

控制代码如下:
``` python
import time

from Phidgets.PhidgetException import *
from Phidgets.Devices.InterfaceKit import *

class TestPhidget(object):
	def __init__(self):
		pass

	def __enter__(self):
		try:
			self.device = InterfaceKit()
			self.device.openPhidget()
			self.device.waitForAttach(10000)
			return self
		except PhidgetException, e:
			exit(1)

	def relayOn(self, output):
		self.device.setOutputState(output, 1)
		time.sleep(1)

	def relayOff(self, output):
		self.device.setOutputState(output, 0)
		time.sleep(1)

	def __exit__(self, e_t, e_v, t_b):
		try:
			self.device.closePhidget()
		except PhidgetException, e:    
			exit(1)

def test():
	import optparse

	parser = optparse.OptionParser()
	parser.add_option("-n",
		dest = "counts",
		action = "store",
		help = "Number of test counts"
	)
    
	(options, args) = parser.parse_args()
	counts = int(options.counts)

	with TestPhidget() as testphidget:
		time.sleep(1)
		for i in range(0, counts):
			testphidget.relayOff(3)
			time.sleep(7)
			testphidget.relayOn(3)
			time.sleep(5)

if __name__ == '__main__':
	test()        

```
这些统统可以在[官方API文档](http://www.phidgets.com/documentation/PythonDoc.zip)里找得到。
效果就是控制风扇的开关。仅此而已(好无聊@@)

# 结语
想了解更多Phidget的信息可以查看[官方网址](http://www.phidgets.com/)。
想要了解`1014_2`原理的可以参考[这里](http://www.phidgets.com/docs/Mechanical_Relay_Primer),只要你学过高中物理，我觉得理解应该没有问题。
