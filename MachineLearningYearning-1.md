---
title: Machine Learning Yearning 读书笔记
date: 2016-12-10
tags: [Machine Learning Yearning,读书笔记]
---

读了几章Andrew NG新书 **Machine Learning Yearning**(v0.5)，非常适合实战，本系列记录相应的读书笔记。

<!--more-->

# [Chapter 1~12](https://gallery.mailchimp.com/dc3a7ef4d750c0abfc19202a3/files/Machine_Learning_Yearning_V0.5_01.pdf "click to download") #

## Train/Dev/Test sets ##
1. 工程项目中，Train-set和Test-set的生成需和线上保持一致，单纯的随机抽样并不一定适合。
> e.g. 资讯推荐系统中，我们拿历史一个月的日志，随机抽样生成Train-set和Test-set，离线会得到很高的AUC指标，但是线上效果却不行。
> 分析原因，资讯推荐实际下发的大部分下发的物料都是新资讯，在历史日志里是找不到的。更合理的是拿前29天的数据作为Train-set，第30天的数据作为Test-set。

2. 使用Dev-set调参/模型选择，使用Test-set评估
> 这个点是我以前没有注意的，Dev-set和Test-set是用的同一份，很容易出现overfit Test-set的情况。只在特定的评估集上效果好，换一个评估集就不行了。
> 调参/模型选择只在Dev-set上进行，一定不能用Test-set，不然Test-set又变成Dev-set了，死循环了。
> 一旦出现overfit Dev-set的情况，就换一组Dev-set和Test-set。

3. Dev/Test sets 大小设置
> 下限：足够体现diff。不设上限，但是过大会造成浪费，影响迭代效率。
> e.g. 分类精度从90.0%提高到90.1%，则起码需要1000个样本才能体现diff。为了消除波动，可以加大到1W。但是没必要加大到1KW，影响迭代效率。

4. 考虑添加Train_Dev集，如下图所示。（个人思考，书中没写）

<img src="https://raw.githubusercontent.com/haorenhao/_posts/master/imgs/%E6%95%B0%E6%8D%AE%E9%9B%86%E5%88%92%E5%88%86.png" width ="700" alt="DataSet" align=center />



## One Single-Number Evaluation Metric ##
1. 单评估指标！多评估指标很难判断哪个模型更优。
> Precision+Recall --> F1 score
> Weigthed average

2. 约束条件 vs 优化目标
> 有些评估指标可以转化为约束条件：e.g.运行时间，模型大小。
> 拆解出约束条件后，再将多目标转化为单目标优化。

# [Chapter 13](https://gallery.mailchimp.com/dc3a7ef4d750c0abfc19202a3/files/Machine_Learning_Yearning_V0.5_02.pdf "click to download") #

## Basic Error Analysis ##
看case非常重要。找到系统瓶颈，寻求突破点。
> 反例：猫分类问题
>    当前错误率是10%，其中只有5%的错误分类是“狗”误判。花一个月时间解决狗误判问题，精度只从90%提升只90.5%。

Q&A：排序问题的case怎么归纳？<font color=red>//TODO</font>

# [Chapter 14](https://gallery.mailchimp.com/dc3a7ef4d750c0abfc19202a3/files/Machine_Learning_Yearning_V0.5_03.pdf "click to download") #

> in parallel
> iterative