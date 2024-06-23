+++
title = 'Prophet 趋势转折点'
subtitle = 'Prophet Trend Changepoints'
brief = ''
date = 2024-06-11T00:00:03
categories = ['Prophet', 'Anaconda']
series = ['Prophet']
tags = ['Prophet', 'Anaconda', 'Docker', 'Python']
type = 'blog'
+++

- - -

You may have noticed in the earlier examples in this documentation that real time series frequently have abrupt changes in their trajectories. By default, Prophet will automatically detect these changepoints and will allow the trend to adapt appropriately. However, if you wish to have finer control over this process (e.g., Prophet missed a rate change, or is overfitting rate changes in the history), then there are several input arguments you can use.

您可能已经在本文档前面的示例中注意到，实时序列的轨迹经常发生突然变化。默认情况下，Prophet 会自动检测这些变化点，并允许趋势进行适当调整。但是，如果您希望更好地控制此过程（例如，Prophet 错过了某个速率变化，或者在历史记录中过度拟合了速率变化），那么您可以使用多个输入参数。

- - -

## Automatic changepoint detection in Prophet（Prophet 中的自动转折点检测）

Prophet detects changepoints by first specifying a large number of *potential changepoints* at which the rate is allowed to change. It then puts a sparse prior on the magnitudes of the rate changes (equivalent to L1 regularization) - this essentially means that Prophet has a large number of *possible* places where the rate can change, but will use as few of them as possible. Consider the Peyton Manning forecast from the Quickstart. By default, Prophet specifies 25 potential changepoints which are uniformly placed in the first 80% of the time series. The vertical lines in this figure indicate where the potential changepoints were placed:

Prophet 通过首先指定允许速率变化的大量*潜在变化点来检测变化点。然后，它会对速率变化的幅度进行稀疏先验（相当于 L1 正则化）——这本质上意味着 Prophet 有大量可能*改变速率的位置，但会尽可能少地使用它们。考虑快速入门中的 Peyton Manning 预测。默认情况下，Prophet 指定 25 个潜在变化点，这些变化点均匀地放置在时间序列的前 80% 中。此图中的垂直线表示潜在变化点的位置：

![trend_changepoints_4_0.png](../trend_changepoints_4_0.png)

Even though we have a lot of places where the rate can possibly change, because of the sparse prior, most of these changepoints go unused. We can see this by plotting the magnitude of the rate change at each changepoint:

尽管有很多地方速率可能会发生变化，但由于稀疏先验，大多数这些变化点都未被使用。我们可以通过绘制每个变化点的速率变化幅度来看到这一点：

![trend_changepoints_6_0.png](../trend_changepoints_6_0.png)

The number of potential changepoints can be set using the argument `n_changepoints`, but this is better tuned by adjusting the regularization. The locations of the signification changepoints can be visualized with:

可以使用参数设置潜在变化点的数量`n_changepoints`，但最好通过调整正则化来调整。意义变化点的位置可以通过以下方式可视化：

```r
# R
plot(m, forecast) + add_changepoints_to_plot(m)
```

```python
# Python
from prophet.plot import add_changepoints_to_plot
fig = m.plot(forecast)
a = add_changepoints_to_plot(fig.gca(), m, forecast)
```

![trend_changepoints_9_0.png](../trend_changepoints_9_0.png)

By default changepoints are only inferred for the first 80% of the time series in order to have plenty of runway for projecting the trend forward and to avoid overfitting fluctuations at the end of the time series. This default works in many situations but not all, and can be changed using the `changepoint_range` argument. For example, `m = Prophet(changepoint_range=0.9)` in Python or `m <- prophet(changepoint.range = 0.9)` in R will place potential changepoints in the first 90% of the time series.

默认情况下，仅推断时间序列的前 80% 的变化点，以便有足够的空间来预测未来的趋势，并避免时间序列末尾的过度拟合波动。此默认设置在许多情况下都有效，但并非全部，可以使用参数进行更改`changepoint_range`。例如，`m = Prophet(changepoint_range=0.9)`在 Python 或`m <- prophet(changepoint.range = 0.9)`R 中，将潜在变化点放置在时间序列的前 90% 中。

- - -

## Adjusting trend flexibility（调整趋势灵活性）

If the trend changes are being overfit (too much flexibility) or underfit (not enough flexibility), you can adjust the strength of the sparse prior using the input argument `changepoint_prior_scale`. By default, this parameter is set to 0.05. Increasing it will make the trend *more* flexible:

如果趋势变化过度拟合（灵活性过高）或欠拟合（灵活性不足），则可以使用输入参数调整稀疏先验的强度`changepoint_prior_scale`。默认情况下，此参数设置为 0.05。增加它将使趋势*更加*灵活：

```r
# R
m <- prophet(df, changepoint.prior.scale = 0.5)
forecast <- predict(m, future)
plot(m, forecast)
```

```python
# Python
m = Prophet(changepoint_prior_scale=0.5)
forecast = m.fit(df).predict(future)
fig = m.plot(forecast)
```

![trend_changepoints_13_0.png](../trend_changepoints_13_0.png)

Decreasing it will make the trend *less* flexible:

降低它会使得趋势变得*不那么*灵活：

```r
# R
m <- prophet(df, changepoint.prior.scale = 0.001)
forecast <- predict(m, future)
plot(m, forecast)
```

```python
# Python
m = Prophet(changepoint_prior_scale=0.001)
forecast = m.fit(df).predict(future)
fig = m.plot(forecast)
```

![trend_changepoints_16_0.png](../trend_changepoints_16_0.png)

When visualizing the forecast, this parameter can be adjusted as needed if the trend seems to be over- or under-fit. In the fully-automated setting, see the documentation on cross validation for recommendations on how this parameter can be tuned.

在可视化预测时，如果趋势似乎过度拟合或欠拟合，则可以根据需要调整此参数。在全自动设置中，请参阅交叉验证文档，了解如何调整此参数的建议。

- - -

## Specifying the locations of the changepoints（指定转折点的位置）

If you wish, rather than using automatic changepoint detection you can manually specify the locations of potential changepoints with the `changepoints` argument. Slope changes will then be allowed only at these points, with the same sparse regularization as before. One could, for instance, create a grid of points as is done automatically, but then augment that grid with some specific dates that are known to be likely to have changes. As another example, the changepoints could be entirely limited to a small set of dates, as is done here:

如果您愿意，您可以使用参数手动指定潜在变化点的位置，而不是使用自动变化点检测`changepoints`。然后，只允许在这些点处进行斜率变化，并使用与之前相同的稀疏正则化。例如，可以像自动完成的那样创建一个点网格，然后用一些已知可能会发生变化的特定日期来扩充该网格。再举一个例子，变化点可以完全限制在一小组日期内，如下所示：

```r
# R
m <- prophet(df, changepoints = c('2014-01-01'))
forecast <- predict(m, future)
plot(m, forecast)
```

```python
# Python
m = Prophet(changepoints=['2014-01-01'])
forecast = m.fit(df).predict(future)
fig = m.plot(forecast)
```

![trend_changepoints_21_0.png](../trend_changepoints_21_0.png)