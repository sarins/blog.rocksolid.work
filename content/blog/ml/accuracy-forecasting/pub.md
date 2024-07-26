+++
title = '精度（准确性）预测'
subtitle = 'Accuracy Forecating'
brief = ''
date = 2022-05-01T00:00:04
categories = ['Machine Learning', 'Math', 'Plot', 'Algorithm']
series = ['Machine Learning', 'Algorithm']
tags = ['Python', 'Machine Learning', 'Math', 'Plot', 'Algorithm', 'Accuracy', 'Forecating']
type = 'blog'
draft = true
+++

---

## 1. GBDT算法

GBDT（Gradient Boosting Decision Tree），全名叫梯度提升决策树，是一种迭代的决策树算法，又叫 MART（Multiple Additive Regression Tree），它通过构造一组弱的学习器（树），并把多颗决策树的结果累加起来作为最终的预测输出。该算法将决策树与集成思想进行了有效的结合。

![1](../1.png)

---

### 1.1. Boosting核心概念

- Boosting方法训练基分类器时采用串行的方式，各个基分类器之间有依赖。它的基本思路是将基分类器层层叠加，每一层在训练的时候，对前一层基分类器分错的样本，给予更高的权重。测试时，根据各层分类器的结果的加权得到最终结果。
- Bagging 与 Boosting 的串行训练方式不同，Bagging 方法在训练过程中，各基分类器之间无强依赖，可以进行并行训练。

![2](../2.png)

#### 1.1.1. TODO
```bash

```
