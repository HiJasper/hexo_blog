---
title: Git生成patch的方法(diff and format-patch)
date: 2015-02-20 16:03
tags: [Git,TECH]
show_copyright : true
---
Git 生成patch的方法（patch就是补丁）
	注:因为博主用的是linux系统,所以运行的环境均在linux下.

### 建立一个文件夹作为git的本地仓库:

``` bash
mkdir /home/jasper/test_for_git
cd test_for_git
git init(创建一个repository)
```
<!--more-->
### 给本地仓库创建一个资源,并写入"hello world"

``` bash
touch hello.txt
echo "hello world">>hello.txt
```

### 提交本地仓库:

``` bash
git add .
git commit -a -m "first commit"
```

### 创建一个本地branch,并且签出到新建的branch

``` bash
git branch jasper
git checkout jasper
```

### 在jasper的branch下面为master brach做一个patch(补丁)
#### 方法一
	用diff的方法生成的通用补丁,patch的文件以.diff结尾:
	1.在patch分支中更改hello.txt:
	echo "first diff patch" >> hello.txt
	2.然后将更改提交:
	git commit -a -m "first diff patch commit"
	3.生成对master branch的patch 并将生成的patch以想要的文件名放到/home/jasper路径下面:
	git diff master > /home/jasper/first_diff_patch.diff
	4.可以ls一下路径下面是否有生成的patch,LZ在/home/jasper下面可以看到自己生成的名为first_diff_patch.diff的patch
	5.签回master branch:
	git checkout master
	6.应用建立的patch:
	git apply /home/jasper/first_diff_patch.diff
	7.提交patch做的更改:
	git commit -a -m "master got the change from diff patch"
	然后你就会发现master已经更新到和jasper branch一样了

#### 方法二
	用format-patch 生成git专用的patch
	1.继续使用上面的例子,签出到jasper branch:
	git checkout jasper
	2.修改hello.txt:
	echo "first format-patch patch" >> hello.txt
	3.将更改提交:
	git commit -a -m "first format-patch commit"
	4.生成对master branch的patch，生成的路径是在当前文件夹下:
	git format-patch -M master //(-M指的是与那个分支比较)
	5.然后会在当前路径下发现一个名为0001-first-format-patch-patch-commit.patch的patch档案(档案名字前面是序号,
	  后面是提交的内容)
	6.签回master branch,并且应用patch:
	git checkout master
	git am 0001-first-format-patch-patch-commit.patch
	7.没有问题的话你就会发现master已经更新到和jasper一样了,然后请删除建立的分支

### 结语
OK,两种方法都可以建立patch,只不过format-patch建立的patch的内容要多于diff建立的(可以cat两个patch察看内容),但是diff使用的面要比format-patch要多.

linux下使用gitk方便查看仓库信息!!!
