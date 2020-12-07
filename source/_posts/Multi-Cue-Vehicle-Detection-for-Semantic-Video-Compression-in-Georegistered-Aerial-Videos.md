---
title: >-
  Multi-Cue Vehicle Detection for Semantic Video Compression in Georegistered
  Aerial Videos
mathjax: true
date: 2020-12-07 14:26:05
categories:
- [论文阅读]
- [传统方法]
- [深度学习方法]
tags:
- [融合]
- [2019]
---

既然传统方法和深度学习方法各有所长，那就把它们一起用！

<!--more-->

## 主要思想

![image-20201207142655003](image-20201207142655003.png)

将传统方法得到的运动目标检测结果和YOLO的检测结果进行决策级融合。

如下图所示，当同时被YOLO和运动目标检测检测出来时，可以认定是运动目标；当只被YOLO检测出来，可以认定为静止目标或者错误检测（类别错误）；当只被运动目标检测检测到，那么如果这个目标很小，可以看做是错误检测（运动的但不是车辆），如果目标很大，那可能是建筑物的那一片误检测。

![image-20201207142750566](image-20201207142750566.png)

## 一些细节

论文针对的是WAMI类型图像，与凝视卫星视频图像有差距，本文YOLO训练使用卫星数据Vehicle Detection in Aerial Imagery (VEDAI)
dataset，类别有nine vehicle classes (truck, camping car, tractor, boat, plane, pick-up, car, van and other），GSD为12.5cm，训练时将VEDAI的三个子类car、pick-up和van合并为一个组合的vehicle类；测试时使用目标数据，并将$2000\times2000$的数据裁剪为$500\times 500$

大小送入YOLO。