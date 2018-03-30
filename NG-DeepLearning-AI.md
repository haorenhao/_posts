NG - Deep Learning AI 学习笔记

---
title: NG - Deep Learning AI 学习笔记
date: 2018-01-05
tags: [Deep Learning, AI, 读书笔记]
---

# 正则化
1. L1/L2正则: 需要调超参。
2. dropout: 随机性，不可复现。
3. Data Augmentation: "fake training data"，可能引入nosie。
4. early stop: 非正交化。
5. Batch normalization: mini-batch下计算mean/variance有一定噪声，算轻微正则。

# Bayes error
1. Bayes error: 理论最低误差，在人类擅长的领域（e.g. 图像识别），可以用human error来近似。 
2. avoidable error = training error -  Bayes error。
3. avoidable error较大时，专注降低bias。（e.g. bigger network）
4. avoidable error较小时，专注降低variance。(e.g. Regularization)

# CNN
1. Why convolution: 参数共享，稀疏连接。
2. 1x1 convolotion: Network in Network(FC in one slice), "shrink" in channle。

# BN
1. z = wx + b, BN会将z值normalize然后rescale，所以参数b没有意义，可以去除。
2. BN降低当前层对前层的依赖（前层输出被normalize到相对稳定的分布）。
3. 在mini-batch下做BN，mean/variance是有噪声的，相当于加了正则。
4. 预测时两个方案确定mean／variance：a.记录训练时所有的mean／variance计算无偏估计量，需要保留额外数据。b. exponential weighted sum方式更新（类似momentum）。