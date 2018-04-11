---
title: 基于TensorFlow的Wide&Deep Learning 
date: 2017-08-15
tags: [TensorFlow,W&D,Deep learning]
---

最近几个月在项目中引入了基于 TensorFlow 的 Wide & Deep Learning，本文介绍下基本原理和实战中踩过的坑。

<!--more-->

# 1. Wide & Deep Learning

> Wide Learning: 简单模型+复杂特征。例如，使用LR做模型，本质上是各特征维度的记忆(Memorization)，然后线性加权。它易于理解，可控。但是模型本身不具备泛化性（Generalization），通常需要复杂的特征工程来提高模型效果。
> 
> Deep Learning: 复杂模型+简单特征。近几年ML界核武的存在，模型泛化能力（Generalization）强，不需要复杂的特征工程，依赖结构取胜，在很多beachmark上效果都很好。但缺少理论证明，解释性较差，模型不可控。

如下图所示，W&D思想很简答很暴力，Wide + Deep = W&D，同时发挥两者的优势。

<img src="https://www.tensorflow.org/images/wide_n_deep.svg" width ="1000" alt="baseline click model" align=center />


## Joint Train
从上面的图看 W&D 和 ensemble 很像，但和 ensemble 有本质区别。ensemble 是子模型的参数是独立优化，W&D 是 Joint Training，参数是同时优化的。

这带来一个好处是 Wide 侧和 Deep 侧可以联合“互补”，相互去学习对方学不好的部分，例如：Deep 侧先学出“程序员都爱穿条纹衫”，Wide 侧发现大部分情况都是对的，但陈浩并不爱穿条纹衫，所以 Wide 侧只需记住陈浩不爱穿条纹衫，而不需要记其它程序员的情况。

## Wide 侧
在 W&D 里 Wide 侧结构就是一个 LR。
> 1. 记特例：如 Joint Train 节所示。
> 2. 去除 Bias: 拿 Position Bias 举例子，最简单的处理方式，就是在模型特征里添加 position 特征，训练时使用真实展现位置，预测时传入固定值。如果把 position 放在 Deep 侧，整个逻辑没法 make sense，输出完全不可控，肯定是不行的；如果放在 Wide 侧，则完全符合 Click Model - Mixture Hypothesis 假设。所以 Wide 侧另一个重要的作用就是用来去除 Bias。

## Deep 侧
在 W&D 里 Deep 侧使用的网络结构很简单，就是 MLP + Embedding。
> 1. MLP：带来非线性，让输入特征进行自动交叉组合。
> 2. Embedding: 通过反馈更新 Embedding Table，学习原始输入的行为相似性。例如: 如果 Item1 和 Item2 的反馈（点击）相似，那学出来的 Embedding 就是相似的，说明它们在受众上有一定相似性。最直接的好处是在 CTR 打分时，可以进行相互泛化。另外，我们可以把 Embedding 拉出来，离线建立 U2I2I (U2I: 通过 User 找到历史点击过的 Item，I2I: 通过 Embedding 找到相似的 Item) 的召回拉链。

# 2. TensorFlow W&D
W&D 作为 Google 的亲儿子，在[TensorFlow Model](https://github.com/tensorflow/models/tree/r1.4.0/official/wide_deep) 里有官方的实现。只有单机的版本，工业应用时通常要用到分布式版本。改动起来比较容易，可以自己写 PS 和 Worker 角色或者使用官方封装的 tf.contrib.learn.learn_runner，这里不展开叙述了。主要讲讲实践中遇到的问题吧。
## CPU vs GPU
官方的实现里，是使用 CPU，而且在[注释](https://github.com/tensorflow/models/blob/r1.4.0/official/wide_deep/wide_deep.py#L145)里显现的说了 CPU faster than GPU.

这里有个小坑，这里使用 CPU 是因为 Deep 侧网络比较简单（接入特征少+MLP小）。在实践中，我们通常要用一个会使用一个复杂的网络，使用 GPU 会更快（我们的项目实测，GPU 会更快）。

简单分析下原因，GPU 在 dense 计算（e.g. 矩阵乘积）时，能发挥并行的优势，所以当 FC 层成为瓶颈时会是更好的选择；但是在 Sparse 计算（e.g. embedding_lookup）时，所以当 Wide 侧或 Deep 侧最下层的 embedding 成为瓶颈时，CPU 反而更快。

## TF Serving 平滑切换
使用 TF Serving 做预估服务时，需要先定义好输入的 schema，在增减特征时，无法做到平滑切换。

但是在实际项目中，做线上实验时经常需要做特征的删减，如果无法做到平滑切换，则将实验分组和 Serving 分组耦合在一起，需要一起改动，工程成本较高。

我们的解决方法是自己写一个 KVparser 的 OP 编译到 TensorFlow 里面（不是我开发的），它的输入是一个 KV 的字符串，然后解析成多个 tensor 返回。这样输入就统一了，线上实验请求可以将请求字段的超集发送过来，通过 Serving 端加载不同的模型来做 ABTest。