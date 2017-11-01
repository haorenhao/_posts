---
title: 基于TensorFlow的Wide&Deep Learning //TODO
date: 2017-08-15
tags: [TensorFlow,W&D,Deep learning]
---

最近几个月在项目中引入了基于TensorFlow的Wide&Deep Learning，本文介绍下基本原理和实战中踩过的坑。

<!--more-->

# 1. Wide & Deep Learning #

> Wide Learning: 简单模型+复杂特征。例如，使用LR做模型，本质上是各特征维度的记忆(Memorization)，然后线性加权。它易于理解，可控。但是模型本身不具备泛化性（Generalization），通常需要复杂的特征工程来提高模型效果。
>
> Deep Learning: 复杂模型+简单特征。近几年ML界核武的存在，模型泛化能力（Generalization）强，不需要复杂的特征工程，依赖结构取胜，在很多beachmark上效果都很好。但缺少理论证明，解释性较差，模型不可控。

W&D思想很简答，就是Wide + Deep，