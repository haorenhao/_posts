---
title: 基于TensorFlow的Online learning
date: 2017-03-25
tags: [TensorFlow,Online learning,FTRL,gradient descent,adgrad]
---

最近几个月在项目中引入了基于TensorFlow的online learning，本文介绍下基本原理和实战中踩过的坑。

<!--more-->


# 1. TensorFlow #

主要看[官方文档和API](https://www.tensorflow.org/)就行，这里只简单记录下有意思的特性。

## Gradients computation ##
和大部分的DL库一样，采用链式求导，自带的OP中都已经包含了gradient的求解，我们在使用时，只需要搭建forward的求LOSS/likelihood的图就行，然后选择优化器去训练就行。

## Partial Execution ##
根据client指定的fetch，自动查找依赖，进行部分计算。一方面避免了无效的计算，另一方面实现了图的"多态"(例如，图中e和f可以分别是模型的训练和评估。)

## Parallel training ##
和其它的分布式计算一样，参数同步有异步和同步的两种形式。

## Queue ##
Queue可以作为一个缓冲区，实现数据预处理和模型训练的并行。

---
# 2. Online learning #

## overall structure ##

online learning是一个相对于offline learning的概念。
- 在offline learning中，图中的extract为ETL定期收集日志并提取样本；training为模型的定期训练batch training或者increment training。
- 在online learning中，图中的extract为实时样本流；training为模型的online training。据说凤巢已经做到了流式的训练样本，直接更新线上的model；我们的实现为流式的训练样本，定期dump模型然后推到线上reload。

## 实时样本流 ##
- 使用kafka实时收集服务器生成的pv log和客户端实时上报的click log.
- 使用storm流式的提取特征.
- label的确认:用户的反馈行为是延时的。我们的实现中采用延时确认方式，收集到pv log后先提取特征，然后等待15分钟(具体时长是label正确性和样本流时效性的tradeoff)，如果收集到对应的click log则生成正样本，否则生成负样本。在Twitter的一篇文章中，收集到pv log中马上产出一个负样本，收集到click log后拼接上pv log产出一个正样本，这样做引入了一些"错误的负样本"但提高了样本流的时效性。

## online training (FTRL based on TF) ##


## 收益分析 ##

# 3. 踩坑指南 #