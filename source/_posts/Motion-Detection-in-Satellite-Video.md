---
title: Motion Detection in Satellite Video
mathjax: true
date: 2020-11-22 14:50:03
categories:
- [论文阅读]
- [传统方法]
tags:
- [传统方法]
- [移动目标检测]
- [背景差分法]
- [光流法]
- [2017]
---

完整介绍了一种背景差分法的流程，对伪运动目标和鬼影判断都有算法。

<!--more-->

## Introduction

背景差分的关键是采用合适的像素采样方法建立背景模型。根据前一帧检测的分割信息进行恰当的模型更新，以适应背景变化。[4]

文章归纳的挑战有：

1）由于卫星的运行、抖动等原因，卫星图像总有微小变化。变化分为两部分，全局变化和由视差引起的局部伪运动。

2）帧宽度很大，分辨率和对比度都较低，运动物体的尺寸很小且缺少特征信息。

因此，

1）对于大宽度动态场景，要对全局运动加以补偿，每一帧的背景应该是固定的。

2）建立准确而稳健的背景模型，并分割和提取小物体，另外针对经典的补偿方法需要加以改进。

## 算法流程

1.利用中间帧建立背景模型

2.估计真贱运动模型，并以此为基础做运动补偿，通过比较补偿好的帧与背景帧来得到目标的分割结果。

3.应用连接分量分析的方法提取潜在目标，根据检测到的分割信息和候选的伪运动区域更新背景模型，来处理下一帧。

![image-20201122152805262](image-20201122152805262.png)

## 具体各部分算法

### 3.1 改进ViBe背景模型

本文的背景模型用ViBe法，仅在一帧的基础上完成模型建立。

考虑到局部伪运动，对vibe模型进行调整，引入更新因子**a**，表示背景模型的更新频率。

### 3.2 全局运动补偿

本章对全局运动进行补偿。本文提出一种A global motion compensation method of blocked  forward-backward LK optical flow。

首先，对每个block使用uniform blocking processing方法提取出Good Feartures。

然后，利用光流跟踪跟踪Good Feature，实现了相关点集的匹配，再根据模型中相关点集估计每个block的运动模型。最后，收集整理变换结果完成整帧的运动补偿。

#### 图像分块和Good Feature提取

将参考帧打成$M\times N$块，每个小块记为$B_{mn}$，第m行第n列个块。然后使用[15]的Good Feature方法，用一个3,3滤波窗。最小因子和距离的取值可以设置得稍小，保证在低频弱特征的区域提取出足够多的好特征，且分布均匀。

![image-20201122154556809](image-20201122154556809.png)

#### 前后LK光流的点跟踪与匹配

理想情况下向前跟踪一步再回来应该回到同一个地方，但是实际上会有误差，这个误差就是要消掉或者说控制的。

![image-20201122155051112](image-20201122155051112.png)

![image-20201122155418650](image-20201122155418650.png)

#### 模型估计和运动补偿

匹配完成后，利用相关点估计block的帧间2D仿射模型参数并建立集合变换关系。同时，the MSACM-estimator Sample Consensus algorithm [16] 可以用来消除有误差的匹配点，进一步优化精度。2维帧间仿射参数矩阵如下：
$$
\begin{bmatrix}
a_1, a_2, 0\\
a_3, a_4, 0\\
a_5, a_6, 1\\
\end{bmatrix}
$$
然后，逆解的数字微分校正方法被使用[17]。一个空白图片被作为补偿帧建起来，根据变换关系，原补偿帧的小块就按照变换关系到空白帧上。这就完成了全局补偿。

![image-20201122160332829](image-20201122160332829.png)

#### 运动检测和分割

使用ViBe法建立背景模型后的运动检测就是那一套，当所有移动像素都分类玩，前北京的二值图像就获得了，在此基础上进行连通成分分析，将每个连通分量当做候选并进行分割。

#### 判断伪运动和背景模型更新

分割出的候选目标由3个主要部分构成，真实运动目标，视差伪目标和归目标。伪运动的判断目的在于找到后两种

首先，对于时差伪目标，选取准则如下：1）利用连通分量节点的坐标，提取目标的最小包围矩形。计算长宽比，如果长宽比大于3.5，则使该目标成为潜在的“伪运动”。2）  从目标候选区域提取边缘。确认靶标与边沿的重叠面积是否超过靶标面积的90%。如果是这样，那么目标就可以被视为潜在的“伪运动”，

根据这设置伪目标区域：将潜在“伪运动”目标的包络矩形向外扩展1个像素，同时向运动模型的平移矢量方向向外扩展10个像素，得到更新因子矩形A。

![image-20201122164026047](image-20201122164026047.png)

根据“鬼”目标的特点，确定了判断方法：记录运动目标第一次出现的位置。如果运动目标连续10帧静止，则视为潜在的“幽灵”目标

然后对背景进行更新，更新策略如下：每个像素背景模型都有一个更新因子a，每个像素背景模型的更新频率是1/a，

![image-20201122164318201](image-20201122164318201.png)

这种更新策略保证伪目标都进入背景中