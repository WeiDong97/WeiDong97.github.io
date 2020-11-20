---
layout: post
title: Needles in a Haystack Tracking City-Scale Moving Vehicles From Continuously Moving Satellite
date: 2020-11-20 15:44:50
mathjax: true
categories: [论文阅读]
tags: [移动目标检测]
---

# 20201117-Needles in a Haystack: Tracking City-Scale Moving Vehicles From Continuously Moving Satellite

## 卫星视频移动目标检测问题

1. 目标移动并且很小，通常只有几个像素，并且没有纹理和颜色，最好的特征就是运动特征。

2. 卫星视频的帧覆盖了一个巨大范围并且提供一个动态场景。

   文章根据镜头与物体的距离将视频分为near-field, medium-field, far-field surveillance videos, and extremely far-field satellite videos[2] [3].卫星视频的处理更为困难，不仅是广阔视野还因为非常复杂的背景。

3. **背景称亚像素级不均匀运动**

   光流场如[4],[5]所示，背景的运动导致静止像素也在动，从而为运动目标检测造成妨碍。

## 方案概述

将每帧分解成两部分，原始图像和随机2D噪声信号图。采用随机分布拟合噪声，有利于分辨潜在的运动目标。

a local tactic is applied to address intra-variants within a frame and discern inter-variants between frames

然后是一种基于多形态线索的区域生长方法和判别方法可以消除其他噪声。

然后使用KF追踪车辆

## related work

B.移动目标检测和追踪

GMM[14-16], Vibe[17], Hierarchical convolutional features(HCF)[18,19]

GMM利用权重呼和的高斯分布来建模像素值随时间的变化，但是像素值变化可能不是服从于高斯分布，因此大多数情况下我们不能用一个确定的参数模型表示像素值变化

Vibe将不同时间不长的像素值认为是一个空间的样本，为了表示这些像素。

## Methodology

### Key concept：

==*detector*==: 集成了local tactic and noise modeling algorithm。

==*candidates*==：detector的输出，包括了true和some noise

==*discrimination*==：判别true和noise的程序，包括the proposed region growing and multi-morphological-cue based discrimination algorithms.

==*Hypotheses*==: discrimination 的输出，由true和few noise组成

==*state*==：a vector that includes **position, velocity, and acceleration** of a vehicle in a time step

==*track*==: a sequence of states of a vehicle in the temporal domain. 被一个独特ID标记并安排给一个KF

==*association*==：匹配算法，meets the minimum cost

==*prediction*==： KF推断出的当前位置

![image-20201117104616406](image-20201117104616406.png)

### Motion-Based Detection Using Local Noise Modeling

二值化的阈值不能固定，需要自适应

1) local tactic： 解决帧内的剧烈变化，在帧中沿垂直和水平方向实现2D光栅化，每个块设为30*30（emparically)

2) Detecting method:

*Step1*: 差分

*Step2*: 估计噪声分布

*Step3*：二值化

*Step4*：逻辑与

![image-20201117105120909](image-20201117105120909.png)

3)具体解释

***Step1:差分***

将帧看做2d信号，由原始光信号和附加的随机噪声组成
$$
G_i(x,y)=g_i(x,y)+n_i(x,y)
$$

> RGB->Grey

then,
$$
D_{i,i+k}(x,y)=|G_i(x,y)-G_{i+k}(x,y)|
=|n_i(x,y)-n_{i+k}(x,y)|
$$
***Step2:估计噪声分布***

根据直方图，heavy tail对应outlier，是target；拟合方法采用指数分布
$$
c_E(x;\lambda)=\begin{cases}
1-exp(-\lambda x), &\ x>0 \\
0,&x \leq \mbox{0}\end{cases}
$$
***Step3:******二值化***

根据[7]，引入一个预定义的概率值$p_{fa}$来推导阈值
$$
th=c_E^{-1}(1-p_{fa};\lambda)
$$
-1代表逆函数，这里设置参数$p_{fa}$为0.05，大于阈值th被认为是outlier

***Step4：逻辑与***

In addition to eliminating ambiguities, logical AND
also reduces the existing noises due to their random appearing.

### Region Growing and Multi-Morphological-Cue Based Discrimination

仍然可能有噪声，包括不规则噪声和规则噪声，不规则噪声是由剧烈光照变化或轻微偏差引起的，随机出现在一些连续帧中，**可以被KF逐步消除**

另一种是规则噪声，背景均匀移动，他是与移动目标混淆的重要一点。

用一个术语*pseudo motion*代称



算法：

1.车辆是2D时域中的一个奇异点，相反的，这些规则噪声的邻域内相似分布，如果*candidates*可以被链接到相似的邻域像素，那就可以根据形状区分，车辆是矩形，噪声是奇怪形状

邻域被定义为11*11

相似度度量使用高斯分布

被确定连通的像素的阈值应该是
$$
th_G^-=c_G^-(p_{fa}^-\mu,\sigma)\\
th_G^+=c_G^{-1}(p_{fa}^+\mu,\sigma)
$$
两个参数选定为0.005和1-0.005

2.区域生长后，采用一系列形态学特征来区分车辆目标和噪声。

这些特征包括（candidate直接叫做候选了）

*Area*：候选的面积

*Extent*：候选的像素与候选框像素比

*Major Axis Length*：如果一个椭圆与候选连接区域的标准化第二中心相同，那么椭圆半长轴定义为主轴长

*Eccentricity*：上述椭圆的离心率

面积和长轴衡量大小，范围和离心率衡量候选与矩形的相似性。

注意，视频的车当然是刚体

## Metric

***Precision***
$$
P=\frac{TP}{TP+FP}
$$
TP：检测出的，并且是目标

FP：检测出的，不是目标

FN：没检测出的目标

***Recall***
$$
R=\frac{TP}{TP+FN}
$$
***F1-score***
$$
F_1=2\frac{P*R}{P+R}
$$
***Jaccard similarity***
$$
J=\frac{TP}{TP+FP+FN}
$$
***MOTA***
$$
MOTA=1-\frac{\sum_i(FN_i+FP_i+IDSW_i)}{\sum_iGT_i}
$$
IDSW：ID switch

指标范围从负无穷到1，越大越好

***MOTP***
$$
MOTP=\frac{\sum_iIoU_i}{\sum_iM_i}
$$
从0到1

## Experiment

### Setup

#### dataset

#### Competitor：

GMM、ViBe、HCF，+KF

#### GT

考虑到工作量，标注了随机抽取的三个500*500区域

每10帧标记一下

其他帧的情况通过线性插值得到

使用Ground Truth Labeler App inMATLAB 2018a 