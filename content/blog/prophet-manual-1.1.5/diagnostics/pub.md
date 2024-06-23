+++
title = 'Prophet 诊断'
subtitle = 'Prophet Diagnostics'
brief = ''
date = 2024-06-10T00:00:09
categories = ['Prophet', 'Anaconda']
series = ['Prophet']
tags = ['Prophet', 'Anaconda', 'Docker', 'Python']
type = 'blog'
+++

- - -

## Cross validation（交叉验证）

Prophet includes functionality for time series cross validation to measure forecast error using historical data. This is done by selecting cutoff points in the history, and for each of them fitting the model using data only up to that cutoff point. We can then compare the forecasted values to the actual values. This figure illustrates a simulated historical forecast on the Peyton Manning dataset, where the model was fit to an initial history of 5 years, and a forecast was made on a one year horizon.

Prophet 包含时间序列交叉验证功能，可使用历史数据测量预测误差。这是通过选择历史记录中的截止点，然后为每个截止点使用截至该截止点的数据拟合模型来完成的。然后，我们可以将预测值与实际值进行比较。该图显示了 Peyton Manning 数据集上的模拟历史预测，其中模型适合 5 年的初始历史，并在一年的范围内进行预测。

![diagnostics_4_0.png](../diagnostics_4_0.png)

[**The Prophet paper**](https://peerj.com/preprints/3190.pdf) gives further description of simulated historical forecasts.

This cross validation procedure can be done automatically for a range of historical cutoffs using the `cross_validation` function. We specify the forecast horizon (`horizon`), and then optionally the size of the initial training period (`initial`) and the spacing between cutoff dates (`period`). By default, the initial training period is set to three times the horizon, and cutoffs are made every half a horizon.

The output of `cross_validation` is a dataframe with the true values `y` and the out-of-sample forecast values `yhat`, at each simulated forecast date and for each cutoff date. In particular, a forecast is made for every observed point between `cutoff` and `cutoff + horizon`. This dataframe can then be used to compute error measures of `yhat` vs. `y`.

Here we do cross-validation to assess prediction performance on a horizon of 365 days, starting with 730 days of training data in the first cutoff and then making predictions every 180 days. On this 8 year time series, this corresponds to 11 total forecasts.

[**Prophet 论文**](https://peerj.com/preprints/3190.pdf)对模拟历史预测进行了进一步的描述。

可以使用该函数自动对一系列历史截止点进行交叉验证`cross_validation`。我们指定预测范围（`horizon`），然后可选地指定初始训练期的大小（`initial`）和截止日期之间的间隔（`period`）。默认情况下，初始训练期设置为范围的三倍，并且每半个范围进行一次截止。

的输出`cross_validation`是一个数据框，其中包含每个模拟预测日期和每个截止日期的真实值`y`和样本外预测值。具体来说，对和`yhat`之间的每个观察点进行预测。然后可以使用此数据框计算与的误差度量。`cutoffcutoff + horizonyhaty`

我们在这里进行交叉验证，以评估 365 天范围内的预测效果，从第一个截止点的 730 天训练数据开始，然后每 180 天进行一次预测。在这个 8 年的时间序列中，这相当于总共 11 次预测。

```r
# R
df.cv <- cross_validation(m, initial = 730, period = 180, horizon = 365, units = 'days')
head(df.cv)
```

```python
# Python
from prophet.diagnostics import cross_validation
df_cv = cross_validation(m, initial='730 days', period='180 days', horizon = '365 days')
```

```python
# Python
df_cv.head()
```

|  | DS | YHAT | YHAT_LOWER | YHAT_UPPER | Y | CUTOFF |
| --- | --- | --- | --- | --- | --- | --- |
| 0 | 2010-02-16 | 8.959678 | 8.470035 | 9.451618 | 8.242493 | 2010-02-15 |
| 1 | 2010-02-17 | 8.726195 | 8.236734 | 9.219616 | 8.008033 | 2010-02-15 |
| 2 | 2010-02-18 | 8.610011 | 8.104834 | 9.125484 | 8.045268 | 2010-02-15 |
| 3 | 2010-02-19 | 8.532004 | 7.985031 | 9.041575 | 7.928766 | 2010-02-15 |
| 4 | 2010-02-20 | 8.274090 | 7.779034 | 8.745627 | 7.745003 | 2010-02-15 |

In R, the argument `units` must be a type accepted by `as.difftime`, which is weeks or shorter. In Python, the string for `initial`, `period`, and `horizon` should be in the format used by Pandas Timedelta, which accepts units of days or shorter.

Custom cutoffs can also be supplied as a list of dates to the `cutoffs` keyword in the `cross_validation` function in Python and R. For example, three cutoffs six months apart, would need to be passed to the `cutoffs` argument in a date format like:

在 R 中，参数`units`必须是 接受的类型`as.difftime`，即周或更短。在 Python 中， 、 和 的字符串`initial`应`period`采用`horizon`Pandas Timedelta 使用的格式，该格式接受天或更短的单位。

自定义截止时间也可以作为日期列表提供给Python 和 R 中函数`cutoffs`的关键字`cross_validation`。例如，相隔六个月的三个截止时间需要以`cutoffs`日期格式传递给参数，例如：

```r
# R
cutoffs <- as.Date(c('2013-02-15', '2013-08-15', '2014-02-15'))
df.cv2 <- cross_validation(m, cutoffs = cutoffs, horizon = 365, units = 'days')
```

```python
# Python
cutoffs = pd.to_datetime(['2013-02-15', '2013-08-15', '2014-02-15'])
df_cv2 = cross_validation(m, cutoffs=cutoffs, horizon='365 days')
```

The `performance_metrics` utility can be used to compute some useful statistics of the prediction performance (`yhat`, `yhat_lower`, and `yhat_upper` compared to `y`), as a function of the distance from the cutoff (how far into the future the prediction was). The statistics computed are mean squared error (MSE), root mean squared error (RMSE), mean absolute error (MAE), mean absolute percent error (MAPE), median absolute percent error (MDAPE) and coverage of the `yhat_lower` and `yhat_upper` estimates. These are computed on a rolling window of the predictions in `df_cv` after sorting by horizon (`ds` minus `cutoff`). By default 10% of the predictions will be included in each window, but this can be changed with the `rolling_window` argument.

该`performance_metrics`实用程序可用于计算预测性能（`yhat`、`yhat_lower`和与`yhat_upper`相比`y`）的一些有用统计数据，作为与截止点距离（预测的未来距离）的函数。计算的统计数据包括均方误差 (MSE)、均方根误差 (RMSE)、平均绝对误差 (MAE)、平均绝对百分比误差 (MAPE)、中位数绝对百分比误差 (MDAPE) 以及`yhat_lower`和估计的覆盖率。这些是在按范围（减）排序`yhat_upper`后，根据预测的滚动窗口计算得出的。默认情况下，每个窗口将包含 10% 的预测，但可以使用参数进行更改。`df_cvdscutoffrolling_window`

```r
# R
df.p <- performance_metrics(df.cv)
head(df.p)
```

```python
# Python
from prophet.diagnostics import performance_metrics
df_p = performance_metrics(df_cv)
df_p.head()
```

|  | HORIZON | MSE | RMSE | MAE | MAPE | MDAPE | SMAPE | COVERAGE |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | 37 days | 0.493764 | 0.702683 | 0.504754 | 0.058485 | 0.049922 | 0.058774 | 0.674052 |
| 1 | 38 days | 0.499522 | 0.706769 | 0.509723 | 0.059060 | 0.049389 | 0.059409 | 0.672910 |
| 2 | 39 days | 0.521614 | 0.722229 | 0.515793 | 0.059657 | 0.049540 | 0.060131 | 0.670169 |
| 3 | 40 days | 0.528760 | 0.727159 | 0.518634 | 0.059961 | 0.049232 | 0.060504 | 0.671311 |
| 4 | 41 days | 0.536078 | 0.732174 | 0.519585 | 0.060036 | 0.049389 | 0.060641 | 0.678849 |

Cross validation performance metrics can be visualized with `plot_cross_validation_metric`, here shown for MAPE. Dots show the absolute percent error for each prediction in `df_cv`. The blue line shows the MAPE, where the mean is taken over a rolling window of the dots. We see for this forecast that errors around 5% are typical for predictions one month into the future, and that errors increase up to around 11% for predictions that are a year out.

交叉验证性能指标可以用 进行可视化`plot_cross_validation_metric`，这里显示的是 MAPE。点表示 中每个预测的绝对百分比误差`df_cv`。蓝线表示 MAPE，其中平均值取自滚动的点窗口。对于这个预测，我们看到，对于未来一个月的预测，误差约为 5% 是典型的，而对于一年后的预测，误差会增加到 11% 左右。

```r
# R
plot_cross_validation_metric(df.cv, metric = 'mape')
```

```python
# Python
from prophet.plot import plot_cross_validation_metric
fig = plot_cross_validation_metric(df_cv, metric='mape')
```

![diagnostics_17_0.png](../diagnostics_17_0.png)

The size of the rolling window in the figure can be changed with the optional argument `rolling_window`, which specifies the proportion of forecasts to use in each rolling window. The default is 0.1, corresponding to 10% of rows from `df_cv` included in each window; increasing this will lead to a smoother average curve in the figure. The `initial` period should be long enough to capture all of the components of the model, in particular seasonalities and extra regressors: at least a year for yearly seasonality, at least a week for weekly seasonality, etc.

可以使用可选参数 更改图中滚动窗口的大小`rolling_window`，该参数指定每个滚动窗口中使用的预测比例。默认值为 0.1，对应于`df_cv`每个窗口中包含的行数的 10%；增加此值将导致图中的平均曲线更平滑。周期`initial`应该足够长，以捕获模型的所有组成部分，特别是季节性和额外的回归量：年度季节性至少为一年，每周季节性至少为一周，等等。

- - -

## Parallelizing cross validation（并行交叉验证）

Cross-validation can also be run in parallel mode in Python, by setting specifying the `parallel` keyword. Four modes are supported

- `parallel=None` (Default, no parallelization)
- `parallel="processes"`
- `parallel="threads"`
- `parallel="dask"`

For problems that aren’t too big, we recommend using `parallel="processes"`. It will achieve the highest performance when the parallel cross validation can be done on a single machine. For large problems, a [**Dask**](https://dask.org/) cluster can be used to do the cross validation on many machines. You will need to [**Install Dask**](https://docs.dask.org/en/latest/install.html) separately, as it will not be installed with `prophet`.

交叉验证也可以在 Python 中以并行模式运行，通过设置指定关键字`parallel`。支持四种模式

- `parallel=None`（默认，无并行化）
- `parallel="processes"`
- `parallel="threads"`
- `parallel="dask"`

对于不太大的问题，我们建议使用`parallel="processes"`。当可以在单台机器上进行并行交叉验证时，它将实现最高性能。对于大型问题，可以使用[**Dask**](https://dask.org/)集群在多台机器上进行交叉验证。您需要单独[**安装Dask**](https://docs.dask.org/en/latest/install.html)，因为它不会随 一起安装`prophet`。

```python
from dask.distributed import Client

client = Client()  # connect to the cluster

df_cv = cross_validation(m, initial='730 days', period='180 days', horizon='365 days', parallel="dask")
```

- - -

## Hyperparameter tuning（超参数调节）

Cross-validation can be used for tuning hyperparameters of the model, such as `changepoint_prior_scale` and `seasonality_prior_scale`. A Python example is given below, with a 4x4 grid of those two parameters, with parallelization over cutoffs. Here parameters are evaluated on RMSE averaged over a 30-day horizon, but different performance metrics may be appropriate for different problems.

交叉验证可用于调整模型的超参数，例如`changepoint_prior_scale`和`seasonality_prior_scale`。下面给出了一个 Python 示例，其中包含这两个参数的 4x4 网格，并在截止点上并行化。这里根据 30 天内平均的 RMSE 评估参数，但不同的性能指标可能适用于不同的问题。

```python
# Python
import itertools
import numpy as np
import pandas as pd

param_grid = {  
    'changepoint_prior_scale': [0.001, 0.01, 0.1, 0.5],
    'seasonality_prior_scale': [0.01, 0.1, 1.0, 10.0],
}

# Generate all combinations of parameters
all_params = [dict(zip(param_grid.keys(), v)) for v in itertools.product(*param_grid.values())]
rmses = []  # Store the RMSEs for each params here

# Use cross validation to evaluate all parameters
for params in all_params:
    m = Prophet(**params).fit(df)  # Fit model with given params
    df_cv = cross_validation(m, cutoffs=cutoffs, horizon='30 days', parallel="processes")
    df_p = performance_metrics(df_cv, rolling_window=1)
    rmses.append(df_p['rmse'].values[0])

# Find the best parameters
tuning_results = pd.DataFrame(all_params)
tuning_results['rmse'] = rmses
print(tuning_results)
```

```python
    changepoint_prior_scale  seasonality_prior_scale      rmse
0                     0.001                     0.01  0.757694
1                     0.001                     0.10  0.743399
2                     0.001                     1.00  0.753387
3                     0.001                    10.00  0.762890
4                     0.010                     0.01  0.542315
5                     0.010                     0.10  0.535546
6                     0.010                     1.00  0.527008
7                     0.010                    10.00  0.541544
8                     0.100                     0.01  0.524835
9                     0.100                     0.10  0.516061
10                    0.100                     1.00  0.521406
11                    0.100                    10.00  0.518580
12                    0.500                     0.01  0.532140
13                    0.500                     0.10  0.524668
14                    0.500                     1.00  0.521130
15                    0.500                    10.00  0.522980
```

```python
# Python
best_params = all_params[np.argmin(rmses)]
print(best_params)
```

```python
{'changepoint_prior_scale': 0.1, 'seasonality_prior_scale': 0.1}
```

Alternatively, parallelization could be done across parameter combinations by parallelizing the loop above.

The Prophet model has a number of input parameters that one might consider tuning. Here are some general recommendations for hyperparameter tuning that may be a good starting place.

或者，可以通过并行化上述循环来实现跨参数组合的并行化。

Prophet 模型有许多输入参数可能需要调整。以下是一些超参数调整的一般建议，可能是一个不错的起点。

- - -

**Parameters that can be tuned（可调整的参数）**

- `changepoint_prior_scale`: This is probably the most impactful parameter. It determines the flexibility of the trend, and in particular how much the trend changes at the trend changepoints. As described in this documentation, if it is too small, the trend will be underfit and variance that should have been modeled with trend changes will instead end up being handled with the noise term. If it is too large, the trend will overfit and in the most extreme case you can end up with the trend capturing yearly seasonality. The default of 0.05 works for many time series, but this could be tuned; a range of [0.001, 0.5] would likely be about right. Parameters like this (regularization penalties; this is effectively a lasso penalty) are often tuned on a log scale.
- `changepoint_prior_scale`：这可能是影响最大的参数。它决定了趋势的灵活性，特别是趋势在趋势变化点的变化程度。如本文档所述，如果该值太小，趋势将欠拟合，本应使用趋势变化建模的方差最终将由噪声项处理。如果该值太大，趋势将过拟合，在最极端的情况下，您最终可能会得到捕捉年度季节性的趋势。默认值 0.05 适用于许多时间序列，但可以对其进行调整；范围 [0.001, 0.5] 可能差不多。像这样的参数（正则化惩罚；这实际上是套索惩罚）通常在对数尺度上进行调整。
- `seasonality_prior_scale`: This parameter controls the flexibility of the seasonality. Similarly, a large value allows the seasonality to fit large fluctuations, a small value shrinks the magnitude of the seasonality. The default is 10., which applies basically no regularization. That is because we very rarely see overfitting here (there’s inherent regularization with the fact that it is being modeled with a truncated Fourier series, so it’s essentially low-pass filtered). A reasonable range for tuning it would probably be [0.01, 10]; when set to 0.01 you should find that the magnitude of seasonality is forced to be very small. This likely also makes sense on a log scale, since it is effectively an L2 penalty like in ridge regression.
- `seasonality_prior_scale`：此参数控制季节性的灵活性。同样，较大的值允许季节性适应较大的波动，较小的值会缩小季节性的幅度。默认值为 10，基本上不应用正则化。这是因为我们很少在这里看到过度拟合（由于它是用截断的傅里叶级数建模的，因此存在固有的正则化，因此它本质上是低通滤波）。调整它的合理范围可能是 [0.01, 10]；当设置为 0.01 时，您应该会发现季节性的幅度被强制为非常小。这在对数尺度上可能也是有意义的，因为它实际上是像岭回归中的 L2 惩罚。
- `holidays_prior_scale`: This controls flexibility to fit holiday effects. Similar to seasonality_prior_scale, it defaults to 10.0 which applies basically no regularization, since we usually have multiple observations of holidays and can do a good job of estimating their effects. This could also be tuned on a range of [0.01, 10] as with seasonality_prior_scale.
- h`olidays_prior_scale`：这控制适应节假日效应的灵活性。与 seasonality_prior_scale 类似，它默认为 10.0，这基本上不应用正则化，因为我们通常对节假日有多个观察，并且可以很好地估计它们的影响。这也可以像 seasonality_prior_scale 一样在 [0.01, 10] 范围内进行调整。
- `seasonality_mode`: Options are [`'additive'`, `'multiplicative'`]. Default is `'additive'`, but many business time series will have multiplicative seasonality. This is best identified just from looking at the time series and seeing if the magnitude of seasonal fluctuations grows with the magnitude of the time series (see the documentation here on multiplicative seasonality), but when that isn’t possible, it could be tuned.
- s`easonality_mode`：选项为 [ `'additive'`, `'multiplicative'`]。默认值为`'additive'`，但许多业务时间序列将具有乘性季节性。最好的识别方法是查看时间序列，看看季节性波动的幅度是否随着时间序列的幅度而增长（请参阅此处有关乘性季节性的文档），但如果这不可能，则可以进行调整。

- - -

**Maybe tune?（是否需要调整）**

- `changepoint_range`: This is the proportion of the history in which the trend is allowed to change. This defaults to 0.8, 80% of the history, meaning the model will not fit any trend changes in the last 20% of the time series. This is fairly conservative, to avoid overfitting to trend changes at the very end of the time series where there isn’t enough runway left to fit it well. With a human in the loop, this is something that can be identified pretty easily visually: one can pretty clearly see if the forecast is doing a bad job in the last 20%. In a fully-automated setting, it may be beneficial to be less conservative. It likely will not be possible to tune this parameter effectively with cross validation over cutoffs as described above. The ability of the model to generalize from a trend change in the last 10% of the time series will be hard to learn from looking at earlier cutoffs that may not have trend changes in the last 10%. So, this parameter is probably better not tuned, except perhaps over a large number of time series. In that setting, [0.8, 0.95] may be a reasonable range.
- c`hangepoint_range`：这是允许趋势变化的历史记录比例。默认值为 0.8，即历史记录的 80%，这意味着模型将无法拟合时间序列最后 20% 的任何趋势变化。这是相当保守的，以避免在时间序列的最后阶段过度拟合趋势变化，因为没有足够的剩余空间来很好地拟合它。在人工参与的情况下，这一点可以很容易地通过视觉识别：人们可以非常清楚地看到预测在最后 20% 的表现是否不好。在完全自动化的设置中，不那么保守可能会有好处。使用如上所述的截断交叉验证可能无法有效地调整此参数。通过查看可能在最后 10% 没有趋势变化的早期截断，很难了解模型从时间序列最后 10% 的趋势变化中概括的能力。因此，除非在大量时间序列中，否则最好不要调整此参数。在该设置下，[0.8, 0.95] 可能是一个合理的范围。

- - -

**Parameters that would likely not be tuned（可能不会调整的参数）**

- `growth`: Options are ‘linear’ and ‘logistic’. This likely will not be tuned; if there is a known saturating point and growth towards that point it will be included and the logistic trend will be used, otherwise it will be linear.
- `growth`：选项为“线性”和“逻辑”。这可能不会进行调整；如果存在已知的饱和点并且朝着该点增长，则会将其包括在内并使用逻辑趋势，否则将是线性的。
- `changepoints`: This is for manually specifying the locations of changepoints. None by default, which automatically places them.
- `changepoints`：用于手动指定变更点的位置。默认情况下为无，即自动放置变更点。
- `n_changepoints`: This is the number of automatically placed changepoints. The default of 25 should be plenty to capture the trend changes in a typical time series (at least the type that Prophet would work well on anyway). Rather than increasing or decreasing the number of changepoints, it will likely be more effective to focus on increasing or decreasing the flexibility at those trend changes, which is done with `changepoint_prior_scale`.
- `n_changepoints`：这是自动放置的变更点的数量。默认值 25 应该足以捕捉典型时间序列中的趋势变化（至少是 Prophet 可以很好地处理的类型）。与其增加或减少变更点的数量，不如更有效地专注于增加或减少这些趋势变化的灵活性，这可以通过 来完成`changepoint_prior_scale`。
- `yearly_seasonality`: By default (‘auto’) this will turn yearly seasonality on if there is a year of data, and off otherwise. Options are [‘auto’, True, False]. If there is more than a year of data, rather than trying to turn this off during HPO, it will likely be more effective to leave it on and turn down seasonal effects by tuning `seasonality_prior_scale`.
- `yearly_seasonality`：默认情况下（“自动”），如果有一年的数据，则将年度季节性打开，否则关闭。选项为 ['auto'、True、False]。如果有一年以上的数据，与其在 HPO 期间尝试关闭此功能，不如将其保持打开状态并通过调整来降低季节性影响，这样可能更有效`seasonality_prior_scale`。
- `weekly_seasonality`: Same as for `yearly_seasonality`.
- `daily_seasonality`: Same as for `yearly_seasonality`.
- `holidays`: This is to pass in a dataframe of specified holidays. The holiday effects would be tuned with `holidays_prior_scale`.
- `holidays`：这是传入指定节假日的数据框。节假日效果将通过 进行调整`holidays_prior_scale`。
- `mcmc_samples`: Whether or not MCMC is used will likely be determined by factors like the length of the time series and the importance of parameter uncertainty (these considerations are described in the documentation).
- `mcmc_samples`：是否使用 MCMC 可能取决于时间序列的长度和参数不确定性的重要性等因素（这些考虑因素在文档中描述）。
- `interval_width`: Prophet `predict` returns uncertainty intervals for each component, like `yhat_lower` and `yhat_upper` for the forecast `yhat`. These are computed as quantiles of the posterior predictive distribution, and `interval_width` specifies which quantiles to use. The default of 0.8 provides an 80% prediction interval. You could change that to 0.95 if you wanted a 95% interval. This will affect only the uncertainty interval, and will not change the forecast `yhat` at all and so does not need to be tuned.
- `interval_width`：Prophet`predict`返回每个组件的不确定性区间，例如预测的`yhat_lower`和。这些是作为后验预测分布的分位数计算的，并指定要使用的分位数。默认值 0.8 提供 80% 的预测区间。如果您想要 95% 的区间，可以将其更改为 0.95。这只会影响不确定性区间，不会改变预测，因此不需要进行调整。`yhat_upperyhatinterval_widthyhat。`
- `uncertainty_samples`: The uncertainty intervals are computed as quantiles from the posterior predictive interval, and the posterior predictive interval is estimated with Monte Carlo sampling. This parameter is the number of samples to use (defaults to 1000). The running time for predict will be linear in this number. Making it smaller will increase the variance (Monte Carlo error) of the uncertainty interval, and making it larger will reduce that variance. So, if the uncertainty estimates seem jagged this could be increased to further smooth them out, but it likely will not need to be changed. As with `interval_width`, this parameter only affects the uncertainty intervals and changing it will not affect in any way the forecast `yhat`; it does not need to be tuned.
- `uncertainty_samples`：不确定性区间是根据后验预测区间的分位数计算的，后验预测区间是用蒙特卡罗抽样估计的。此参数是要使用的样本数（默认为 1000）。预测的运行时间与此数字呈线性关系。减小此值将增加不确定性区间的方差（蒙特卡罗误差），增大此值将减少该方差。因此，如果不确定性估计看起来不均匀，可以增加此值以进一步平滑它们，但可能不需要更改。与 一样`interval_width`，此参数仅影响不确定性区间，更改它不会以任何方式影响预测`yhat`；不需要对其进行调整。