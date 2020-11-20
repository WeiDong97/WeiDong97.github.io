---
title: GMM
mathjax: true
date: 2020-11-20 13:33:20
---



# GMM-Gaussian Mixture Model

## Definition

参数概率密度函数，用gaussian component densities的加权和表示。

GMM参数估计是从训练数据使用迭代期望最大化（EM）算法或最大后验（MAP）估计从一个训练有素的先验模型。

## Introduction

$$
p(\text{x}|\lambda)=\sum_{i=1}^M\omega_ig(\text{x}|\mu_i,\begin{matrix}\sum_i\end{matrix})
$$

where
$$
g(\text{x}|\mu_i,\begin{matrix}\sum_i\end{matrix})=\frac{1}{(2\pi)^{D/2}|\begin{matrix}\sum_i\end{matrix}|^{1/2}}\text{exp}\left\{-\frac{1}{2}(\text{x}-\mu_i)'\begin{matrix}\sum_i^{-1}\end{matrix}(\text{x}-\mu_i)\right\}
$$
with mean vextor $\mu_i$ and covariance matrix $\sum_i$, the weight satisfy the constraint that $\sum_{i=1}^M\omega_i=1$.

in the formular, the parameters can be denoted by
$$
\lambda=\{ \omega_i,\mu_i,\begin{matrix}\sum_i\end{matrix}\}\qquad i=1,2,...,M
$$
![pic1](1.png)

> Fig. 1. Comparison of distribution modeling. (a) histogram of a single cepstral coefficient from a 25 second utterance by a male speaker (b)maximum likelihood uni-modal Gaussian model (c) GMM and its 10 underlying component densities (d) histogram of the data assigned to the VQ centroid locations of a 10 element codebook.

