---
tags:
  - Machine Learning
  - Paper
mathjax: true
comments: true
hidden: false
date: 2021-12-24 16:38:17
title: 论文理解 XiangBai——【CVPR2018】Multi-Oriented Scene Text Detection via Corner Localization and Region Segmentation
---
***
Multi-Oriented Scene Text Detection via Corner Localization and Region Segmentation
基于角点定位和区域分割的多向场景文本检测<!-- more -->
***

# Multi-Oriented Scene Text Detection via Corner Localization and Region Segmentation

目前场景文字检测（scene text detection ）基本分为两类：

* 将其作为目标检测来进行训练学习（文字扭曲、旋转的问题无法很好的解决）
* 将其直接作为文本分割的任务来进行（需要复杂的后处理）



该论文提出结合两个方法

从框的4个边角（左上，右上，左下，右下）出发

{% asset_img a.png %}

大致思路：

​	找出边角点，同时找出可能的边角区域（positionsensitive segmentation maps），两者进行结合对比，生成候选框，再由NMS进行细化处理

***

由此可以**解决文本倾斜的问题，文本旋转问题得到解决**，

（吹：）不管是单个文字还是单词还是句子，都可以分割的很好。



贡献：提出了一种全新的端到端方法，结合了目标检测和分割的理念。

提出了一种可以应对任意朝向的ROI池化方式

可以处理多种朝向的场景文本

SOTA



结合多方面，Related Work，

基于回归的文本检测： 关注的是点

基于分割的文本检测：用提出的Rotated Position-Sensitive Average ROI Pooling层处理

基于边角点的通用目标检测，敏感区域划分



网络设计：

{% asset_img b.png %}

全卷积网络进行  特征提取、边角点检测、敏感区域划分

backbone采用VGG-16

考虑到以下几个方面：

* 检测目标尺寸差异大（例如一个字和一大行文本）
* 文字背景复杂

FPN和DSSD在这两个方面做的挺不错，因此也采用了FPN/DSSD的结构来做处理

实现细节：

将vgg的全连接层（fc6、fc7）换为卷积层（conv6、conv7），然后在后面加了几层卷积层(conv8, conv9, conv10, conv11)以便扩大特征抽取的感受野

之后加了6个DSSD中的反卷积块，然后为了让模型适应多尺度数据，对一些模块的数据进行了级联处理

最后将产生的F3, F4, F7, F8, F9, F10 和 F11进行卷积和反卷积处理，为之后的corner detect进行准备

边角检测方面，每个角点都可能有4种情况，左上左下右上右下，而且一个点的位置可能是同时具有多个属性，比如一个点为A框的左上点，同时也是B框的右下点。角点检测输出两个分支，score分支和offset分支，其中score分支是用于检测区域内有无角点存在。score和offset分支输出分别为k × q × 2和k × q × 4，q为角点的类型数，一般都为矩形，所以为4.

标签生成：

旋转矩形的坐标如何确定？

给定4个顶点

* 左上左下的坐标，x坐标必须要小于右上右下的坐标
* 左上右上的坐标，y坐标必须要小于左下右下的坐标

这样4个点就能被确定好自己的分类位置


## 参考资料
[Multi-Oriented Scene Text Detection via Corner Localization and Region Segmentation](https://arxiv.org/abs/1802.08948)
<font size=4 face="幼圆" color='lightblue'>to be continue···</font>