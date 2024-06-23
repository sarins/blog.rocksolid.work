+++
title = 'Prophet 非每日数据'
subtitle = 'Prophet Non-Daily Data'
brief = ''
date = 2024-06-10T00:00:08
categories = ['Prophet', 'Anaconda']
series = ['Prophet']
tags = ['Prophet', 'Anaconda', 'Docker', 'Python']
type = 'blog'
+++

- - -

## Sub-daily data（颗粒度更细的时间轴数据）

Prophet can make forecasts for time series with sub-daily observations by passing in a dataframe with timestamps in the `ds` column. The format of the timestamps should be YYYY-MM-DD HH:MM:SS - see the example csv [**here**](https://github.com/facebook/prophet/blob/main/examples/example_yosemite_temps.csv). When sub-daily data are used, daily seasonality will automatically be fit. Here we fit Prophet to data with 5-minute resolution (daily temperatures at Yosemite):

Prophet 可以通过在列中传入带有时间戳的数据框来预测具有亚日观测值的时间序列`ds`。时间戳的格式应为 YYYY-MM-DD HH:MM:SS - 请参阅[**此处**](https://github.com/facebook/prophet/blob/main/examples/example_yosemite_temps.csv)的示例 csv 。
当使用亚日数据时，每日季节性将自动拟合。在这里，我们将 Prophet 拟合到具有 5 分钟分辨率的数据（优胜美地的每日温度）：

```r
# R
df <- read.csv('https://raw.githubusercontent.com/facebook/prophet/main/examples/example_yosemite_temps.csv')
m <- prophet(df, changepoint.prior.scale=0.01)
future <- make_future_dataframe(m, periods = 300, freq = 60 * 60)
fcst <- predict(m, future)
plot(m, fcst)
```

```python
# Python
df = pd.read_csv('https://raw.githubusercontent.com/facebook/prophet/main/examples/example_yosemite_temps.csv')
m = Prophet(changepoint_prior_scale=0.01).fit(df)
future = m.make_future_dataframe(periods=300, freq='H')
fcst = m.predict(future)
fig = m.plot(fcst)
```

![non-daily_data_4_0.png](../non-daily_data_4_0.png)

The daily seasonality will show up in the components plot:

每日季节性将显示在成分图中：

```r
# R
prophet_plot_components(m, fcst)
```

```python
# Python
fig = m.plot_components(fcst)
```

![non-daily_data_7_0.png](../non-daily_data_7_0.png)

- - -

## Data with regular gaps（有规律的间隙数据）

Suppose the dataset above only had observations from 12a to 6a:

假设上述数据集仅包含从 12a 到 6a 的观测数据：

```r
# R
df2 <- df %>%
  mutate(ds = as.POSIXct(ds, tz="GMT")) %>%
  filter(as.numeric(format(ds, "%H")) < 6)
m <- prophet(df2)
future <- make_future_dataframe(m, periods = 300, freq = 60 * 60)
fcst <- predict(m, future)
plot(m, fcst)
```

```python
# Python
df2 = df.copy()
df2['ds'] = pd.to_datetime(df2['ds'])
df2 = df2[df2['ds'].dt.hour < 6]
m = Prophet().fit(df2)
future = m.make_future_dataframe(periods=300, freq='H')
fcst = m.predict(future)
fig = m.plot(fcst)
```

![non-daily_data_10_0.png](../non-daily_data_10_0.png)

The forecast seems quite poor, with much larger fluctuations in the future than were seen in the history. The issue here is that we have fit a daily cycle to a time series that only has data for part of the day (12a to 6a). The daily seasonality is thus unconstrained for the remainder of the day and is not estimated well. The solution is to only make predictions for the time windows for which there are historical data. Here, that means to limit the `future` dataframe to have times from 12a to 6a:

预测似乎相当糟糕，未来的波动比历史上看到的要大得多。这里的问题是，我们将每日周期拟合到只有部分时间（12a 到 6a）数据的时间序列中。因此，剩余时间的每日季节性不受限制，并且无法很好地估计。解决方案是仅对有历史数据的时间窗口进行预测。在这里，这意味着将数据框限制为`future`从 12a 到 6a 的时间：

```r
# R
future2 <- future %>% 
  filter(as.numeric(format(ds, "%H")) < 6)
fcst <- predict(m, future2)
plot(m, fcst)
```

```python
# Python
future2 = future.copy()
future2 = future2[future2['ds'].dt.hour < 6]
fcst = m.predict(future2)
fig = m.plot(fcst)
```

![non-daily_data_13_0.png](../non-daily_data_13_0.png)

The same principle applies to other datasets with regular gaps in the data. For example, if the history contains only weekdays, then predictions should only be made for weekdays since the weekly seasonality will not be well estimated for the weekends.

同样的原则也适用于数据中存在规律性间隙的其他数据集。例如，如果历史记录仅包含工作日，则应仅对工作日进行预测，因为无法很好地估计周末的每周季节性。

- - -

## Monthly data（月度数据）

You can use Prophet to fit monthly data. However, the underlying model is continuous-time, which means that you can get strange results if you fit the model to monthly data and then ask for daily forecasts. Here we forecast US retail sales volume for the next 10 years:

您可以使用 Prophet 来拟合月度数据。但是，底层模型是连续时间的，这意味着如果您将模型拟合到月度数据然后要求进行每日预测，您可能会得到奇怪的结果。这里我们预测了未来 10 年的美国零售额：

```r
# R
df <- read.csv('https://raw.githubusercontent.com/facebook/prophet/main/examples/example_retail_sales.csv')
m <- prophet(df, seasonality.mode = 'multiplicative')
future <- make_future_dataframe(m, periods = 3652)
fcst <- predict(m, future)
plot(m, fcst)
```

```python
# Python
df = pd.read_csv('https://raw.githubusercontent.com/facebook/prophet/main/examples/example_retail_sales.csv')
m = Prophet(seasonality_mode='multiplicative').fit(df)
future = m.make_future_dataframe(periods=3652)
fcst = m.predict(future)
fig = m.plot(fcst)
```

![non-daily_data_16_0.png](../non-daily_data_16_0.png)

This is the same issue from above where the dataset has regular gaps. When we fit the yearly seasonality, it only has data for the first of each month and the seasonality components for the remaining days are unidentifiable and overfit. This can be clearly seen by doing MCMC to see uncertainty in the seasonality:

这与上面的问题相同，数据集有规律地存在差距。当我们拟合年度季节性时，它只有每月第一天的数据，其余日子的季节性成分无法识别且过度拟合。通过进行 MCMC 可以清楚地看到季节性的不确定性：

```r
# R
m <- prophet(df, seasonality.mode = 'multiplicative', mcmc.samples = 300)
fcst <- predict(m, future)
prophet_plot_components(m, fcst)
```

```python
# Python
m = Prophet(seasonality_mode='multiplicative', mcmc_samples=300).fit(df, show_progress=False)
fcst = m.predict(future)
fig = m.plot_components(fcst)
```

```python
WARNING:pystan:481 of 600 iterations saturated the maximum tree depth of 10 (80.2 %)
WARNING:pystan:Run again with max_treedepth larger than 10 to avoid saturation
```

![non-daily_data_19_1.png](../non-daily_data_19_1.png)

The seasonality has low uncertainty at the start of each month where there are data points, but has very high posterior variance in between. When fitting Prophet to monthly data, only make monthly forecasts, which can be done by passing the frequency into `make_future_dataframe`:

季节性在每个月初有数据点时不确定性较低，但其间后验方差非常高。当将 Prophet 拟合到月度数据时，仅进行月度预测，这可以通过将频率传入来完成`make_future_dataframe`：

```r
# R
future <- make_future_dataframe(m, periods = 120, freq = 'month')
fcst <- predict(m, future)
plot(m, fcst)
```

```python
# Python
future = m.make_future_dataframe(periods=120, freq='MS')
fcst = m.predict(future)
fig = m.plot(fcst)
```

![non-daily_data_22_0.png](../non-daily_data_22_0.png)

In Python, the frequency can be anything from the pandas list of frequency strings here: https://pandas.pydata.org/pandas-docs/stable/user_guide/timeseries.html#timeseries-offset-aliases . Note that `MS` used here is month-start, meaning the data point is placed on the start of each month.

In monthly data, yearly seasonality can also be modeled with binary extra regressors. In particular, the model can use 12 extra regressors like `is_jan`, `is_feb`, etc. where `is_jan` is 1 if the date is in Jan and 0 otherwise. This approach would avoid the within-month unidentifiability seen above. Be sure to use `yearly_seasonality=False` if monthly extra regressors are being added.

在 Python 中，频率可以是此处 pandas 频率字符串列表中的任何内容：https://pandas.pydata.org/pandas-docs/stable/user_guide/timeseries.html#timeseries-offset-aliases 。请注意，`MS`这里使用的是月份开始，这意味着数据点位于每个月的开始处。

在月度数据中，年度季节性也可以用二元额外回归量来建模。具体来说，该模型可以使用 12 个额外回归量，如`is_jan`、`is_feb`等。其中`is_jan`，如果日期在一月，则为 1，否则为 0。这种方法可以避免上面看到的月内不可识别性。`yearly_seasonality=False`如果要添加每月额外回归量，请务必使用。

- - -

## Holidays with aggregated data（带汇总数据的节假日）

Holiday effects are applied to the particular date on which the holiday was specified. With data that has been aggregated to weekly or monthly frequency, holidays that don’t fall on the particular date used in the data will be ignored: for example, a Monday holiday in a weekly time series where each data point is on a Sunday. To include holiday effects in the model, the holiday will need to be moved to the date in the history dataframe for which the effect is desired. Note that with weekly or monthly aggregated data, many holiday effects will be well-captured by the yearly seasonality, so added holidays may only be necessary for holidays that occur in different weeks throughout the time series.

节假日效应适用于指定节假日的特定日期。对于按周或月频率汇总的数据，不在数据所用特定日期的节假日将被忽略：例如，在每周时间序列中，周一节假日是每个数据点都在周日的节假日。若要在模型中包含节假日效应，需要将节假日移至历史数据框中需要该效应的日期。请注意，对于按周或月汇总的数据，许多节假日效应将通过年度季节性得到很好的捕捉，因此可能只需要为时间序列中不同周发生的节假日添加节假日。