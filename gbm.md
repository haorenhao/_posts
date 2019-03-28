---
title: GBM
date: 2018-01-29
tags: [GBM,xgboost,lightGBM]
---

本文主要记录下 xgboost 和 lightGBM 的学习笔记。

<!--more-->

# ID3、C4.5、CART
- 根据 info entropy gain (ID3)、info entropy gain ratio (C4.5)、gini (CART) 计算最优分裂。
- 节点均值作为节点打分。

# xgboost
- 节点打分不是简单平均，而是通过 Optimize the objective (Training Loss + Regularization) 计算。

> 每次学习新的树时，Training Loss = sum(loss(yi, yi(t-1) + ft(xi) + reg(ft)) + constant。其中argmin（Training loss）下的 ft 是要求的节点打分，可以理解为一个 1 * K（节点个数）的向量。
> 
> 已知树的结构，可通过将 loss(yi, yi(t-1) + ft(xi) 做二阶泰勒展开，reg(ft)通常包括叶子节点个数和节点打分，合并到一起后是个二次方程，很容易求 argmin 的 ft 的取值（需要计算每个样本的一阶和二阶导，每棵树只要计算一次）。

- 逐层遍历所有可能的结构，找到最优分裂(pre-sort-based and linear-scan)。
- 离散特征 one-hot。
- Pre-stopping vs Post-Prunning。

# lightGBM
- Gradient-based One-Side Sampling: 根据样本梯度大小采样（头部保留，尾部采样 + 加权）。
- Histogram bin: 直方图分桶（xgboost也支持直方图但没有 EFB）。
- EFB: 将互斥的特征（不同时取非空值）的特征集合作为一个slot，共用一个 Histogram。
- 默认 Leaf-wise（xgboost 也支持 Leaf-wise，但默认是level-wise），限制树最大深度缓解过拟合问题。
- Optimal Split for Categorical Features: 离散特征不用 one-hot 变成多个 slot。如果直接遍历，有 2^(k-1) - 1 划分方式。但通过累积 sum_gradient / sum_hessian 对直方图排序，然后做划分，时间复杂度为 O(k * log(k))。
- Feature Parallel: 只需要存 Histogram，所以可以每个 worker 节点保存所有的数据，所以不需要多次通讯，每个 worker 负责找一部分特征的最优划分，最后再比较下，选最优就行。
- Data Parallel: 网络通讯优化，使用 “Reduce Scatter” 同步不同的部分；分裂后只需对一个子节点统计直方图，另一个子节点可通过直方图求差（父节点-兄弟节点）得到。

# 参考
- ID3、C4.5、CART:《统计学习方法》
- xgboost: [paper](https://arxiv.org/pdf/1603.02754.pdf)、[ppt](https://homes.cs.washington.edu/~tqchen/pdf/BoostedTree.pdf)、[docs](https://xgboost.readthedocs.io/en/latest/)
- lightgbm: [paper](https://papers.nips.cc/paper/6907-lightgbm-a-highly-efficient-gradient-boosting-decision-tree.pdf)、[docs](https://lightgbm.readthedocs.io/en/latest)
