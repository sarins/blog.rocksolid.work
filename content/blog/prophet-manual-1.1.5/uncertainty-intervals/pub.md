+++
title = 'Prophet 不确定性区间'
subtitle = 'Prophet Uncertainty Intervals'
brief = ''
date = 2024-06-10T00:00:06
categories = ['Prophet', 'Anaconda']
series = ['Prophet']
tags = ['Prophet', 'Anaconda', 'Docker', 'Python']
type = 'blog'
+++

- - -

By default Prophet will return uncertainty intervals for the forecast `yhat`. There are several important assumptions behind these uncertainty intervals.

There are three sources of uncertainty in the forecast: uncertainty in the trend, uncertainty in the seasonality estimates, and additional observation noise.

默认情况下，Prophet 将返回预测的不确定区间`yhat`。这些不确定区间背后有几个重要的假设。

预测中的不确定性来源有三个：趋势的不确定性、季节性估计的不确定性和额外的观察噪声。

- - -

## Uncertainty in the trend（趋势的不确定性）

The biggest source of uncertainty in the forecast is the potential for future trend changes. The time series we have seen already in this documentation show clear trend changes in the history. Prophet is able to detect and fit these, but what trend changes should we expect moving forward? It’s impossible to know for sure, so we do the most reasonable thing we can, and we assume that the *future will see similar trend changes as the history*. In particular, we assume that the average frequency and magnitude of trend changes in the future will be the same as that which we observe in the history. We project these trend changes forward and by computing their distribution we obtain uncertainty intervals.

One property of this way of measuring uncertainty is that allowing higher flexibility in the rate, by increasing `changepoint_prior_scale`, will increase the forecast uncertainty. This is because if we model more rate changes in the history then we will expect more in the future, and makes the uncertainty intervals a useful indicator of overfitting.

The width of the uncertainty intervals (by default 80%) can be set using the parameter `interval_width`:

预测中最大的不确定性来源是未来趋势变化的可能性。我们在本文档中已经看到的时间序列显示了历史中的明显趋势变化。Prophet 能够检测和拟合这些变化，但我们应该期待未来出现什么样的趋势变化呢？我们不可能确切知道，所以我们会尽我们所能，做最合理的事情，我们假设未来*将会看到与历史类似的趋势变化*。特别是，我们假设未来趋势变化的平均频率和幅度将与我们在历史中观察到的相同。我们将这些趋势变化投射到未来，并通过计算它们的分布来获得不确定性区间。

这种测量不确定性的方法的一个特性是，通过增加 允许利率具有更高的灵活性，`changepoint_prior_scale`这将增加预测的不确定性。这是因为如果我们在历史中模拟更多的利率变化，那么我们就会预期未来会有更多变化，并使不确定性区间成为过度拟合的有用指标。

不确定区间的宽度（默认为 80%）可以使用以下参数设置`interval_width`：

```r
# R
m <- prophet(df, interval.width = 0.95)
forecast <- predict(m, future)
```

```python
# Python
forecast = Prophet(interval_width=0.95).fit(df).predict(future)
```

Again, these intervals assume that the future will see the same frequency and magnitude of rate changes as the past. This assumption is probably not true, so you should not expect to get accurate coverage on these uncertainty intervals.

再次强调，这些区间假设未来利率变化的频率和幅度与过去相同。这一假设可能并不正确，因此您不应期望在这些不确定区间内获得准确的覆盖范围。

- - -

## Uncertainty in seasonality（季节性的不确定性）

By default Prophet will only return uncertainty in the trend and observation noise. To get uncertainty in seasonality, you must do full Bayesian sampling. This is done using the parameter `mcmc.samples` (which defaults to 0). We do this here for the first six months of the Peyton Manning data from the Quickstart:

默认情况下，Prophet 只会返回趋势和观测噪声中的不确定性。要获得季节性的不确定性，您必须进行完整的贝叶斯抽样。这是使用参数`mcmc.samples`（默认为 0）完成的。我们在这里对 Quickstart 中 Peyton Manning 数据的前六个月执行此操作：

```r
# R
m <- prophet(df, mcmc.samples = 300)
forecast <- predict(m, future)
```

```python
# Python
m = Prophet(mcmc_samples=300)
forecast = m.fit(df, show_progress=False).predict(future)
```

This replaces the typical MAP estimation with MCMC sampling, and can take much longer depending on how many observations there are - expect several minutes instead of several seconds. If you do full sampling, then you will see the uncertainty in seasonal components when you plot them:

这将用 MCMC 采样取代典型的 MAP 估计，并且根据观测次数的不同，可能需要更长的时间 - 预计需要几分钟而不是几秒钟。如果您进行完整采样，那么在绘制季节性成分时，您将看到其中的不确定性：

```r
# R
prophet_plot_components(m, forecast)
```

```python
# Python
fig = m.plot_components(forecast)
```

![uncertainty_intervals_11_0.png](../uncertainty_intervals_11_0.png)

You can access the raw posterior predictive samples in Python using the method `m.predictive_samples(future)`, or in R using the function `predictive_samples(m, future)`.

您可以使用方法在 Python 中访问原始后验预测样本`m.predictive_samples(future)`，或者使用函数在 R 中访问`predictive_samples(m, future)`。