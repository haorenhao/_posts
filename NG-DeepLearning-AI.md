---
title: NG - Deep Learning AI 学习笔记
date: 2018-01-05
tags: [Deep Learning, AI, 读书笔记]
---

本文记录NG的 [ AI 公开课](https://www.deeplearning.ai) 读书笔记。

<!--more-->

# 正则化
1. L1/L2正则: 需要调超参。
2. dropout: 随机性，不可复现。
3. Data Augmentation: "fake training data"，可能引入nosie。
4. early stop: 非正交化。
5. Batch normalization: mini-batch 下计算 mean/variance 有一定噪声，算轻微正则。

# Bayes error
1. Bayes error: 理论最低误差，在人类擅长的领域（e.g. 图像识别），可以用 human error 来近似。 
2. avoidable error = training error -  Bayes error。
3. avoidable error 较大时，专注降低 bias。（e.g. bigger network）
4. avoidable error 较小时，专注降低 variance。(e.g. Regularization)

# CNN
1. Why convolution: 参数共享，稀疏连接。
2. 1x1 convolotion: Network in Network(FC in one slice), "shrink" in channle。

# BN
1. z = wx + b, BN 会将 z 值 normalize 然后 rescale，所以参数b没有意义，可以去除。
2. BN 降低当前层对前层的依赖（前层输出被 normalize 到相对稳定的分布）。
3. 在 mini-batch 下做 BN，mean/variance 是有噪声的，相当于加了正则。
4. 预测时两个方案确定 mean/variance：a.记录训练时所有的 mean/variance 计算无偏估计量，需要保留额外数据。b. exponential weighted sum 方式更新（类似 momentum ）。