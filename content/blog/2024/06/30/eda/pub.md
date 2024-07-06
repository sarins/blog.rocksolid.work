+++
title = '探索性数据分析'
subtitle = 'Exploratory Data Analysis (EDA)'
brief = ''
date = 2024-06-30
categories = ['Machine Learning', 'Math']
series = ['Machine Learning']
tags = ['Machine Learning', 'Math', 'Statistics']
type = 'blog'
draft = true
+++

---

## 1. 什么是EDA (What is EDA)

探索性数据分析主要工作内容包括：数据清洗、基于统计学对数据进行描述（统计量、图表，通过这些手段呈现数据的基本情况。）、查看数据的分布情况（连续、离散）、查看主数据与其关联数据之间的关系。通过上述手段与方法让研究人员或开发人员能够对数据从整体上有一个较为全面的认知。

## 1.1. 探索性数据分析(EDA) 与 传统统计分析(Classical Analysis) 之间的区别

> **传统的统计分析** 通常是先假设样本服从某种分布，然后把数据套入假设模型再做分析。但由于多数数据并不能满足假设的分布，因此，传统统计分析结果常常不能让人满意。

> **探索性数据分析** 注重数据的真实分布，强调数据的可视化，使分析者能一目了然看出数据中隐含的规律，从而得到启发，以此帮助分析者找到适合数据的模型。“探索性”是指分析者对待解问题的理解会随着研究的深入不断变化。

## 1.2. EDA 与 Classical Analysis 工作步骤比较

> **应用传统统计分析方法的数据分析步骤**
> - 提出问题 Problem -> 准备数据 Data => 建模 Model => 分析 Analysis => 得出结论 Conclusions

> **应用探索性数据分析方法的数据分析步骤**
> - 提出问题 Problem => 准备数据 Data => 分析 Analysis => 建模 Model => 得出结论 Conclusions

---

## 2. EDA解决哪些问题？ (What problems the EDA will be resolved ?)

### 2.1. 数据检查 (Data checking)

- 是否有缺失值
- 是否有异常值
- 是否有重复值
- 样本是否均衡
- 是否需要抽样
- 变量是否需要转换
- 是否需要增加新的特征

### 2.2. 基于统计学初步分析 (Preliminary analysis data by Statistics)

#### 2.2.1. 连续型 Continuous

> **常见的描述统计量** 平均值(Mean) / 中位数(Median) / 众数(Mode) / 最小值(Min) / 最大值(Max) / 四分位数(Quartile) / 标准差(Standard Deviation)

> **常见图表** 频数分布表(Frequency Distribution Table) / 直方图(Histogram) / 箱线图(BoxPlot)
> ***
> ![1](../1.png)

#### 2.2.2. 无序离散型/类别型 (Nominal Discrete/Categorical)

> **常见的描述统计量** 变量分布情况(Distribution) / 变量频率占比(Frequency)

> **常见图表** 频数分布表(Frequency Distribution Table) / 柱图(BarPlot) / 饼图(PiePlot) / 韦恩图(VennPlot)
> ***
> ![2](../2.png)

#### 2.2.3. 有序离散型/类别型 (Ordinal Discrete/Categorical)

> **常见的描述统计量** 变量分布情况(Distribution) / 变量频率占比(Frequency)

> **常见图表** 频数分布表(Frequency Distribution Table) / 柱图(BarPlot)

### 2.3 变量之间的关系 (Relation between the variables)

#### 2.3.1. 连续型与连续型 (Continuous vs Continuous)

> 对于连续变量与连续变量之间的关系，通常通过散点图呈现。对于多个连续变量，通常使用散点图矩阵、相关系数矩阵、热图等呈现。
> ***
> ![3](../3.png)
> ***
> **线性关系量化指标** 皮尔逊相关系数(Pearson Correlation) / 斯皮尔曼相关系数(Spearman Correlation)
> ***
> **非线性关系量化指标** 互信息(Mutual Information)

#### 2.3.2. 离散型与离散型 (Categorical vs Categorical)

> 对于离散变量与离散变量之间的关系，可以通过交叉分组表(CrossTab)、复合柱形图、堆积柱形图、饼图等呈现。对于多个离散变量，可以使用网状图，通过各个要素之间是否有线条，以及线条的粗线来呈现是否有关系以及关系的强弱。
> ***
> ![4](../4.png)
> ***
> **量化指标** 卡方独立性检验(Cramér's V / Cramér's phi and denoted as φc)

#### 2.3.1. 连续型与离散型 (Continuous vs Categorical)

> 对于离散变量和连续变量之间的关系，可以使用直方图、箱线图、小提琴图呈现，将离散变量在图形中用不同的颜色显示，来直观地考察变量之间的关系。
> ***
> ![5](../5.png)
> ***
> **量化指标** 独立样本T检验中的T统计量和相应的p值（两个变量）/ 单因素方差分析中的η²（三个变量及以上）