+++
title = 'Prophet 异常值'
subtitle = 'Prophet Outliers'
brief = ''
date = 2024-06-10T00:00:07
categories = ['Prophet', 'Anaconda']
series = ['Prophet']
tags = ['Prophet', 'Anaconda', 'Docker', 'Python']
type = 'blog'
+++

- - -

There are two main ways that outliers can affect Prophet forecasts. Here we make a forecast on the logged Wikipedia visits to the R page from before, but with a block of bad data:

异常值主要通过两种方式影响 Prophet 预测。这里，我们根据之前记录的维基百科对 R 页面的访问情况进行预测，但使用了一组不良数据：

```r
# R
df <- read.csv('https://raw.githubusercontent.com/facebook/prophet/main/examples/example_wp_log_R_outliers1.csv')
m <- prophet(df)
future <- make_future_dataframe(m, periods = 1096)
forecast <- predict(m, future)
plot(m, forecast)
```

```python
# Python
df = pd.read_csv('https://raw.githubusercontent.com/facebook/prophet/main/examples/example_wp_log_R_outliers1.csv')
m = Prophet()
m.fit(df)
future = m.make_future_dataframe(periods=1096)
forecast = m.predict(future)
fig = m.plot(forecast)
```

![outliers_4_0.png](../outliers_4_0.png)

The trend forecast seems reasonable, but the uncertainty intervals seem way too wide. Prophet is able to handle the outliers in the history, but only by fitting them with trend changes. The uncertainty model then expects future trend changes of similar magnitude.

The best way to handle outliers is to remove them - Prophet has no problem with missing data. If you set their values to `NA` in the history but leave the dates in `future`, then Prophet will give you a prediction for their values.

趋势预测似乎合理，但不确定性区间似乎太宽。Prophet 能够处理历史中的异常值，但只能通过将其与趋势变化相拟合。然后，不确定性模型会预期未来趋势变化的幅度相似。

处理异常值的最佳方法是将其移除 - Prophet 不会处理缺失数据。如果您`NA`在历史记录中将其值设置为 但将日期保留为`future`，那么 Prophet 将为您提供其值的预测。

```r
# R
outliers <- (as.Date(df$ds) > as.Date('2010-01-01')
             & as.Date(df$ds) < as.Date('2011-01-01'))
df$y[outliers] = NA
m <- prophet(df)
forecast <- predict(m, future)
plot(m, forecast)
```

```python
# Python
df.loc[(df['ds'] > '2010-01-01') & (df['ds'] < '2011-01-01'), 'y'] = None
model = Prophet().fit(df)
fig = model.plot(model.predict(future))
```

![outliers_7_0.png](../outliers_7_0.png)

In the above example the outliers messed up the uncertainty estimation but did not impact the main forecast `yhat`. This isn’t always the case, as in this example with added outliers:

在上面的例子中，异常值扰乱了不确定性估计，但并未影响主要预测`yhat`。情况并非总是如此，就像在这个添加了异常值的例子中所展示的那样：

```r
# R
df <- read.csv('https://raw.githubusercontent.com/facebook/prophet/main/examples/example_wp_log_R_outliers2.csv')
m <- prophet(df)
future <- make_future_dataframe(m, periods = 1096)
forecast <- predict(m, future)
plot(m, forecast)
```

```python
# Python
df = pd.read_csv('https://raw.githubusercontent.com/facebook/prophet/main/examples/example_wp_log_R_outliers2.csv')
m = Prophet()
m.fit(df)
future = m.make_future_dataframe(periods=1096)
forecast = m.predict(future)
fig = m.plot(forecast)
```

![outliers_10_0.png](../outliers_10_0.png)

Here a group of extreme outliers in June 2015 mess up the seasonality estimate, so their effect reverberates into the future forever. Again the right approach is to remove them:

2015 年 6 月的一组极端异常值扰乱了季节性估计，因此它们的影响将永远回荡到未来。正确的方法是再次将其移除：

```r
# R
outliers <- (as.Date(df$ds) > as.Date('2015-06-01')
             & as.Date(df$ds) < as.Date('2015-06-30'))
df$y[outliers] = NA
m <- prophet(df)
forecast <- predict(m, future)
plot(m, forecast)
```

```python
# Python
df.loc[(df['ds'] > '2015-06-01') & (df['ds'] < '2015-06-30'), 'y'] = None
m = Prophet().fit(df)
fig = m.plot(m.predict(future))
```

![outliers_13_0.png](../outliers_13_0.png)