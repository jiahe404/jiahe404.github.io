---
title: xgboost 特征重要性
tag:
  - xgboost
  - 特征重要性
categories:
  - 机器学习
  - tree
abbrlink: 1781693973
date: 2018-11-18 00:00:00
---

# 官方解释
Python中的xgboost可以通过get_fscore获取特征重要性，先看看官方对于这个方法的[说明](https://xgboost.readthedocs.io/en/latest/python/python_api.html):
>
>get_score(fmap='', importance_type='weight')
>
>Get feature importance of each feature. Importance type can be defined as:
>- ‘weight’: the number of times a feature is used to split the data across all trees.
- ‘gain’: the average gain across all splits the feature is used in.
- ‘cover’: the average coverage across all splits the feature is used in.
- ‘total_gain’: the total gain across all splits the feature is used in.
- ‘total_cover’: the total coverage across all splits the feature is used in.

看释义不直观，下面通过训练一个简单的模型，输出这些重要性指标，再结合释义进行解释。
<!-- more -->
# 代码实践
首先构造10个样例的样本，每个样例有两维特征，标签为0或1，二分类问题:
```python
import numpy as np

sample_num = 10
feature_num = 2

np.random.seed(0)
data = np.random.randn(sample_num, feature_num)
np.random.seed(0)
label = np.random.randint(0, 2, sample_num)
```

输出data和label:
```
# data:
array([[ 1.76405235,  0.40015721],
       [ 0.97873798,  2.2408932 ],
       [ 1.86755799, -0.97727788],
       [ 0.95008842, -0.15135721],
       [-0.10321885,  0.4105985 ],
       [ 0.14404357,  1.45427351],
       [ 0.76103773,  0.12167502],
       [ 0.44386323,  0.33367433],
       [ 1.49407907, -0.20515826],
       [ 0.3130677 , -0.85409574]])
# label:
array([0, 1, 1, 0, 1, 1, 1, 1, 1, 1])
```

训练，这里为了便于下面计算，将树深度设为3('max_depth': 3)，只用一棵树(num_boost_round=1):

```python
import xgboost as xgb

train_data = xgb.DMatrix(data, label=label)
params = {'max_depth': 3}
bst = xgb.train(params, train_data, num_boost_round=1)
```
输出重要性指标:
```python
for importance_type in ('weight', 'gain', 'cover', 'total_gain', 'total_cover'):
    print('%s: ' % importance_type, bst.get_score(importance_type=importance_type))
```
结果:
```
weight:  {'f0': 1, 'f1': 2}
gain:  {'f0': 0.265151441, 'f1': 0.375000015}
cover:  {'f0': 10.0, 'f1': 4.0}
total_gain:  {'f0': 0.265151441, 'f1': 0.75000003}
total_cover:  {'f0': 10.0, 'f1': 8.0}
```
画出唯一的一棵树图:
```python
xgb.to_graphviz(bst, num_trees=0)
```
![one](/img/blog/xgboost/tree.png)

下面就结合这张图，解释下各指标含义:

1. weight:  {'f0': 1, 'f1': 2}  
在所有树中，某特征被用来分裂节点的次数，在本例中，可见分裂第1个节点时用到f0，分裂第2，3个节点时用到f1，所以weight_f0 = 1, weight_f1 = 2。
2. total_cover:  {'f0': 10.0, 'f1': 8.0}  
第1个节点，f0被用来对所有10个样例进行分裂，之后的节点中f0没再被用到，所以f0的total_cover为10.0，此时f0 >= 0.855563045的样例有5个，落入右子树；  
第2个节点，f1被用来对上面落入右子树的5个样例进行分裂，其中f1 >= -0.178257734的样例有3个，落入右子树；  
第3个节点，f1被用来对上面落入右子树的3个样例进行分裂。  
总结起来，f0在第1个节点分裂了10个样例，所以total_cover_f0 = 10，f1在第2、3个节点分别用于分裂5、3个样例，所以total_cover_f1 = 5 + 3 = 8。total_cover表示在所有树中，某特征在每次分裂节点时处理(覆盖)的所有样例的数量。
3. cover:  {'f0': 10.0, 'f1': 4.0}
cover = total_cover / weight，在本例中，cover_f0 = 10 / 1，cover_f1 = 8 / 2 = 4.
4. total_gain:  {'f0': 0.265151441, 'f1': 0.75000003}
在所有树中，某特征在每次分裂节点时带来的总增益，如果用熵或基尼不纯衡量分裂前后的信息量分别为i0和i1，则增益为(i0 - i1)。
5. gain:  {'f0': 0.265151441, 'f1': 0.375000015}
gain = total_gain / weight，在本例中，gain_f0 = 0.265151441 / 1，gain_f1 = 75000003 / 2 = 375000015.

在平时的使用中，多用total_gain来对特征重要性进行排序。

# By The Way
构造xgboost分类器还有另外一种方式，这种方式类似于sklearn中的分类器，采用fit, transform形式训练模型:
```python
from xgboost import XGBClassifier

cls = XGBClassifier(base_score=0.5, booster='gbtree', colsample_bylevel=1,
       colsample_bytree=1, gamma=0, learning_rate=0.07, max_delta_step=0,
       max_depth=3, min_child_weight=1, missing=None, n_estimators=300,
       n_jobs=1, nthread=None, objective='binary:logistic', random_state=0,
       reg_alpha=0, reg_lambda=1, scale_pos_weight=1, seed=None,
       silent=True, subsample=1)
# 训练模型
# cls.fit(data, label)
```
采用下面的方式获取特征重要性指标:
```python
for importance_type in ('weight', 'gain', 'cover', 'total_gain', 'total_cover'):
    print('%s: ' % importance_type, cls.get_booster().get_score(importance_type=importance_type))
```
