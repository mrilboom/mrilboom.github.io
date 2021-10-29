---
title: 如何优雅地在PyCharm中调试代码
tags:
  - python
comments: true
hidden: false
date: 2021-09-13 14:17:55
---
***
最近有个朋友走上了编程的道路,之前劝了他好几回不要入坑,他不听,铁了心要来,于是乎教了他一些PyCharm里面DEBUG的方法,这篇博客也是为他而写的,我尽量从最基础的操作开始讲解,如何一步一步的调试代码,找出代码中的BUG.<!-- more -->
***
## 一年前的聊天记录

Comedian

你怎么把那个笔记打在黑白那个背景上的啊

<p align="right">Geronimo</p>
<p align="right">?</p>

Comedian

黑白的那个背景

<p align="right">Geronimo</p>

<p align="right">什么东西啊？</p>

Comedian

你上次给我看过那个

Comedian

{% asset_img codered.jpg  %}

Comedian

像这个

<p align="right">Geronimo</p>

<p align="right">这**</p>

<p align="right">Geronimo</p>
<p align="right">你是**吗</p>


<p align="right">Geronimo</p>
<p align="right">你以为是记笔记的软件呢</p>

Comedian

因为记在word里很sb

Comedian

全是错误提醒

Comedian

还自动大小写

<p align="right">Geronimo</p>
<p align="right">没事</p>



<p align="right">Geronimo</p>
<p align="right">你记在IDE里更**</p>




<p align="right">Geronimo</p>
<p align="right">写一行红一行</p>

*而一年后的今天,他竟然做到了：*

{% asset_img codered2.jpg %}

## 什么叫DEBUG

&emsp;要说到DEBUG那肯定要知道BUG是什么,BUG英文意为虫子,而计算机中的BUG一词也源自一只小飞虫：1945年9月9日,格蕾丝使用的Mark Ⅱ出现故障格蕾丝使用的Mark Ⅱ出现故障,经过了近一天的检查,格蕾丝找到了故障的原因：继电器中有一只死掉的蛾子（那时的计算机系统用的还都是一个个继电器,机器也异常庞大）,后来,”[BUG](https://baike.baidu.com/item/bug/3353935)” 和”[DEBUG](https://baike.baidu.com/item/debug/825293)”  这两个本来普普通通的词汇成了计算机领域中特指莫明其妙的“错误”和“排除错误”的专用词汇而流传至今,而格蕾丝·赫柏也因此成了第一个发现“bug”的人.

## 如何找到bug
&emsp;&emsp;找bug有许多方式,这里只介绍PyCharm里的DEBUG的一些办法.
### 环境

* *python 3.6*
* PyCharm
* PyCharm中文语言包插件(IDE中安装)

&emsp;&emsp;我觉得IDE改语言为中文对于python初学者来说挺不错的,就算你英文很好我也建议你改用中文的语言设置.在学习python时没必要把时间浪费在花在理解英文意思上,阅读自己的母语总比阅读外语要来的效率高吧.同时自2020年5月份起,JetBrain已经加入了官方汉化,汉化的总体上的质量不错.

&emsp;&emsp;但是因为英文是原生语言,其中有些计算机中的名词并没有很统一的中文命名,而且有少数时候翻译会显得很生硬,这种情况确实是难以避免的.

### 打断点

先来看这一段代码

```python
A = [1196.167,136.667,1933.583,3650.667,255.0833,1392.157,2227.25]
B = [328.8636,549.0152,127.3485,1597.576,1198.712,10437.95,22699.32]
C = [325, 1831.5971209,725.0694,565.4167]
X = []
Y = []
Z = []
weight = 1e9
weight_min = weight
resA = [[]]
for num in A:
    resA += [i + [num] for i in resA]
for ls in resA:
    X.append(sum(ls))
A = X[1:]
result = []
resB = [[]]
for num in B:
    resB += [i + [num] for i in resB]
for ls in resB:
    Y.append(sum(ls))
B = Y[1:]
resC = [[]]
for num in C:
    resC += [i + [num] for i in resC]
for ls in resC:
    Z.append(sum(ls))
C = Z[1:]
D = {}
for a in A:
    for b in B:
        for c in C:
            if a+b+c >= 56400:
                weight = 1.2*a*0.6 + 1.1*b*0.66 + c*0.72
                if weight<weight_min:
                    weight_min = weight
                    result = [a,b,c]
                    D[weight]=result
print(weight_min)
print(result)

out = 5

print("****************")
for i in range(out):
    item = D.popitem()
    print(item[0])
    for i in range(len(A)):
        if item[1][0] == A[i]:
            print(resA[i + 1])
    for i in range(len(B)):
        if item[1][1] == B[i]:
            print(resB[i + 1])
    for i in range(len(C)):
        if item[1][2] == C[i]:
            print(resC[i + 1])
    print(f"**********")
```

这一段代码执行后报错了,PyCharm中的控制台输出如下:

{% asset_img debug1.jpg %}

它提示`popitem(): dictionary is empty`从字面上来看是字典为空出现的问题,那么哪里出了问题呢？前面说到在help.py这个文件中的第45行出了错,那我们就找找第45行中的错误,如果有经验的人一眼就可以看出问题所在了,但是对于新手来说可能会有一点困难,因此我们可以通过“打断点”的方式对程序进行调试(DEBUG).

&emsp;&emsp;打断点是指在程序中标记一个断点,在执行调试时,程序会在将要执行到断点所在那行的代码时停下,此时我们可以观察程序所保存的变量来确定程序出现BUG的原因.

&emsp;&emsp;以这个BUG为例,程序报错在第45行产生,那就在第45行打上断点：

{% asset_img breakpoint.gif %}

*点击此处即可添加断点,再次点击即可取消断点*

### 开始调试

打好断点后即可开始调试,可以通过右键PyCharm代码编辑栏选择“调试”来进行调试

{% asset_img debug.gif %}

等待程序执行到目标位置时,程序就会暂停并统计当前变量信息.

{% asset_img console.jpg %}

红框上的几个图标分别为：

步过：若函数A内存在子函数a时,不会进入子函数a内执行单步调试,而是把子函数a当作一个整体,一步执行.

步入：若函数A内存在子函数a时,会进入子函数a内执行单步调试.

单步执行我的代码：执行下一行但忽略libraries（导入库的语句）

步出：当目前执行在子函数a中时,选择该调试操作可以直接跳出子函数a,而不用继续执行子函数a中的剩余代码.并返回上一层函数.

运行到光标处：运行到鼠标光标所在的地方,在光标与当前断点中还存在断点或光标位置不可达（程序已经执行过的部分或if的另一个分支等情况）,则执行到下一个断点.

执行到这里,我们可以观察当前存在的变量,之前报错发现是"dictionary is empty"的错误,我们之前定义了字典为D,那就找D的变量,如果一时间找不到,则可以点击加号输入要监视的变量

{% asset_img addwatch.jpg %}

### 找到问题根源

找到D之后发现缺失D这个字典里面是空的,为什么是空的？那就找哪里修改了D,可以看到第37行我们对D进行了添加条目的操作,那么尝试在37行打断点,尝试进行调试：

{% asset_img ops.gif %}

此时发现我们的断点被直接跳过了,程序似乎”执行到”37行时并没有停下来列出消息,此时细想一下,程序有可能并没有执行到这一行,有可能是for一开始就跳出循环了,也有可能是if的条件为false了,所以直接跳过了这个地方,这时候我们又得判断一下是哪条语句导致的问题,那就在整个大循环开始的时候打上一个端点,然后一步步单步执行看程序的执行路径

{% asset_img step.gif %}

可以看到程序在总是在执行到` if a+b+c >= 56400:`这一行之后就直接进入了下一步循环,并没有步入,这时候我们就可以提出一个合理的猜测：是否`a+b+c >= 56400`永远为假呢？结合自己写的代码可以简易推出确实如此,写代码的时候有可能是数据录入错误,或者是条件表达式写错了.

### 评估表达式工具

{% asset_img calc.jpg %}

可以看到在各种调试按钮边上还有一个不起眼的计算器样子的图案,那就是评估表达式工具,它可以在调试过程中执行自定义的代码行（段）,比如想要在下面这段代码中判断if条件表达式的信息

```python
# a=1420,b=220,c=300
...
if (a+b+c <= 2000) & (c < 200):
    flag = True
...
```

可以通过计算条件表达式来进行判断

{% asset_img eval.gif %}

这个表达式其实就是额外执行了一段代码,如果在里面对变量进行赋值操作的话,也会生效

{% asset_img change.gif %}

因此注意一些函数,比如pop（）函数,它可以看到栈中最后的一个元素,在执行时该元素也在栈中被剔除了,因此不要用到这些会影响到变量的函数.

{% asset_img pop.gif %}

### 修改变量

有时候我们会有修改程序运行过程中变量的需求,我们此时就可以通过上面评估表达式进行变量赋值操作来修改变量,同时也可以通过直接右键变量设置值来进行修改

{% asset_img change2.gif %}

### CTRL + 左键点击

有时候代码太过冗长,不容易找到某个变量的定义和修改所在的位置,我们即可通过按住`ctrl`然后鼠标单击变量名进行查找

{% asset_img fing.gif %}

该功能在调试外也适用,如果点击的是一些导入库或者是python的一些内置函数或类型,那么PyCharm也会转到相应的定义页面,一般来说这些函数或者类下面都会有

{% asset_img exl.gif %}

## 最后说一说

&emsp;&emsp;无论如何,要找到BUG,就得看懂程序,理清其中的逻辑,逻辑都没有弄清楚就想找到并解决问题,怎么可能呢?用各种外部库函数的时候也要多去看看文档,看看源码,可能就像当年格蕾丝一样,在上百行上千行的代码中找了一整天的bug,就是由一只小小的“飞蛾”引起的.


## 参考资料

[醒了的追梦人|Pycharm Debug调试](https://blog.csdn.net/qq_33472146/article/details/90606359)

