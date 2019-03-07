---
title: TPOT 调研
date: 2018-12-25
tags: [AutoML,TPOT,AutoML pipeline]
---

# 流程

<img src="https://raw.githubusercontent.com/EpistasisLab/tpot/master/images/tpot-ml-pipeline.png" width ="1000" alt="An example machine learning pipeline" align=center />

- preprocessing steps: missing value imputiation, scaling, PCA, feature selection, etc.
- machine learning algorithms: random forests, linear models, SVMs, etc.
- hyperparameters: for models and preprocessing steps
- ensemble or stack

<!--more-->

# 特性
- 基于异常算法。
- 计算量大，整体流程还是比较慢。
- 支持并行计算，单机多线程或 Dask 集群计算。
- 支持中途停止获取结果。
- 支持 warn start。
- 有一定随机性，多次 TPOT 输出的最优 pipeline 不一定一样，但性能差别不大(可通过 random_state 设置种子)。
- 数据集通过 CV 划分。
- 可以查看运行进度。
- 支持命令行运行。
- Built-in 常见 scoring functions，也支持自定义，接口很友好。
- Built-in configurations: Default、light、MDR、sparse，也可以自定义配置来限制参数空间。
- pipeline 提供 cache 功能，可避免重复计算。自定义 cache 目录时需手动删除。
- 搜索出的 pipeline 支持导出。

# 实验
## 示例（MNIST - 分类问题）
### 代码
`vi test.py`

```
from tpot import TPOTClassifier
from sklearn.datasets import load_digits
from sklearn.model_selection import train_test_split
from contextlib import contextmanager
import time

@contextmanager
def timer(title):
    t0 = time.time()
    yield
    print("{} - done in {:.0f}s".format(title, time.time() - t0))

with timer("tpot test"):
    digits = load_digits()
    X_train, X_test, y_train, y_test = train_test_split(digits.data, digits.target,
                                                        train_size=0.75, test_size=0.25)
    pipeline_optimizer = TPOTClassifier(generations=5, population_size=20, cv=5,
                                        random_state=42, verbosity=2)
    pipeline_optimizer.fit(X_train, y_train)
    print(pipeline_optimizer.score(X_test, y_test))
    pipeline_optimizer.export('tpot_exported_pipeline.py')
```

### 运行结果
<img src="https://raw.githubusercontent.com/haorenhao/_posts/master/tpot/test.png" width ="1000" alt="运行结果" align=center />

### exported pipeline
`vi tpot_exported_pipeline.py`

```
import numpy as np
import pandas as pd
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import train_test_split
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import PolynomialFeatures

# NOTE: Make sure that the class is labeled 'target' in the data file
tpot_data = pd.read_csv('PATH/TO/DATA/FILE', sep='COLUMN_SEPARATOR', dtype=np.float64)
features = tpot_data.drop('target', axis=1).values
training_features, testing_features, training_target, testing_target = \
            train_test_split(features, tpot_data['target'].values, random_state=42)

# Average CV score on the training set was:0.987447353168
exported_pipeline = make_pipeline(
    PolynomialFeatures(degree=2, include_bias=False, interaction_only=False),
    LogisticRegression(C=10.0, dual=False, penalty="l1")
)

exported_pipeline.fit(training_features, training_target)
results = exported_pipeline.predict(testing_features)
```

## 真实项目（流量预测 - 回归问题）
### 数据情况
总样本量为 42 万+，根据时间切分留最后 2 万+ 做为测试集。剩余 40 万+ 做为调研数据。

手动做好时序等特征，特征纬度为 400+ 连续值字段，50+ 离散值字段。

### 实验过程
- TPOT: (基于 sklearn 和 xgboost) 自动做模型和超参选择。
- 人工经验: lightgbm + 经验超参。
- hyperopt: lightgbm + tpe 超参搜索。

### 实验结论 (使用体验)
- 使用 MAPE 作为评估指标，TPOT低于人工经验，人工经验低于 hyperopt。(TPOT:人工经验:Hyperopt = 0.35:0.32:0.31）
- TPOT 搜索空间大 (重通用性)，收敛太慢。这块可以优化下，例如在排序等场景中，一般都是用 GBM、LR 和 DL，而 SVM，KNN 等效果往往不佳，所以没必要去搜。
- 当前版本需要手动 fillna。[issue 478](https://github.com/EpistasisLab/tpot/issues/478)
- 自定义 scoring 不能用 forkserver 做多线程并行（貌似是 joblib 的问题），速度较慢。[issue 664](https://github.com/EpistasisLab/tpot/issues/664)