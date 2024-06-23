+++
title = 'Prophet 季节性、节日效应和回归量'
subtitle = 'Prophet Seasonality, Holiday Effects, And Regressors'
brief = ''
date = 2024-06-11T00:00:04
categories = ['Prophet', 'Anaconda']
series = ['Prophet']
tags = ['Prophet', 'Anaconda', 'Docker', 'Python']
type = 'blog'
+++

- - -

## Modeling Holidays and Special Events（模拟假期和特殊活动）

If you have holidays or other recurring events that you’d like to model, you must create a dataframe for them. It has two columns (`holiday` and `ds`) and a row for each occurrence of the holiday. It must include all occurrences of the holiday, both in the past (back as far as the historical data go) and in the future (out as far as the forecast is being made). If they won’t repeat in the future, Prophet will model them and then not include them in the forecast.

You can also include columns `lower_window` and `upper_window` which extend the holiday out to `[lower_window, upper_window]` days around the date. For instance, if you wanted to include Christmas Eve in addition to Christmas you’d include `lower_window=-1,upper_window=0`. If you wanted to use Black Friday in addition to Thanksgiving, you’d include `lower_window=0,upper_window=1`. You can also include a column `prior_scale` to set the prior scale separately for each holiday, as described below.

Here we create a dataframe that includes the dates of all of Peyton Manning’s playoff appearances:

如果您有想要建模的节假日或其他重复事件，则必须为它们创建一个数据框。它有两列（`holiday`和`ds`）和一行，用于表示每个节假日的发生。它必须包含节假日的所有发生，包括过去（追溯到历史数据）和未来（预测进行时）。如果它们将来不会重复，Prophet 将对它们进行建模，然后不将它们包含在预测中。

您还可以添加列`lower_window`和`upper_window`，将假期延伸到`[lower_window, upper_window]`日期前后的几天。例如，如果您希望除了圣诞节之外还包括平安夜，则应添加`lower_window=-1,upper_window=0`。如果您希望除了感恩节之外还包括黑色星期五，则应添加`lower_window=0,upper_window=1`。您还可以添加一列`prior_scale`来为每个假期分别设置先前的比例，如下所述。

这里我们创建一个数据框，其中包含佩顿·曼宁 (Peyton Manning) 所有季后赛出场日期：

```r
# R
library(dplyr)
playoffs <- data_frame(
  holiday = 'playoff',
  ds = as.Date(c('2008-01-13', '2009-01-03', '2010-01-16',
                 '2010-01-24', '2010-02-07', '2011-01-08',
                 '2013-01-12', '2014-01-12', '2014-01-19',
                 '2014-02-02', '2015-01-11', '2016-01-17',
                 '2016-01-24', '2016-02-07')),
  lower_window = 0,
  upper_window = 1
)
superbowls <- data_frame(
  holiday = 'superbowl',
  ds = as.Date(c('2010-02-07', '2014-02-02', '2016-02-07')),
  lower_window = 0,
  upper_window = 1
)
holidays <- bind_rows(playoffs, superbowls)
```

```python
# Python
playoffs = pd.DataFrame({
  'holiday': 'playoff',
  'ds': pd.to_datetime(['2008-01-13', '2009-01-03', '2010-01-16',
                        '2010-01-24', '2010-02-07', '2011-01-08',
                        '2013-01-12', '2014-01-12', '2014-01-19',
                        '2014-02-02', '2015-01-11', '2016-01-17',
                        '2016-01-24', '2016-02-07']),
  'lower_window': 0,
  'upper_window': 1,
})
superbowls = pd.DataFrame({
  'holiday': 'superbowl',
  'ds': pd.to_datetime(['2010-02-07', '2014-02-02', '2016-02-07']),
  'lower_window': 0,
  'upper_window': 1,
})
holidays = pd.concat((playoffs, superbowls))
```

Above we have included the superbowl days as both playoff games and superbowl games. This means that the superbowl effect will be an additional additive bonus on top of the playoff effect.

Once the table is created, holiday effects are included in the forecast by passing them in with the `holidays` argument. Here we do it with the Peyton Manning data from the [**Quickstart**](https://facebook.github.io/prophet/docs/quick_start.html):

上文中我们将超级碗比赛日既作为季后赛比赛也作为超级碗比赛。这意味着超级碗效应将成为季后赛效应之外的额外附加奖励。

创建表格后，通过将假日效应与参数一起传入，即可将其包含在预测中。这里我们使用[**Quickstart**](https://facebook.github.io/prophet/docs/quick_start.html)`holidays`中的 Peyton Manning 数据进行操作：

```r
# R
m <- prophet(df, holidays = holidays)
forecast <- predict(m, future)
```

```python
# Python
m = Prophet(holidays=holidays)
forecast = m.fit(df).predict(future)
```

The holiday effect can be seen in the `forecast` dataframe:

在数据框中可以看到节日效应`forecast`：

```python
# R
forecast %>% 
  select(ds, playoff, superbowl) %>% 
  filter(abs(playoff + superbowl) > 0) %>%
  tail(10)
```

```python
# Python
forecast[(forecast['playoff'] + forecast['superbowl']).abs() > 0][
        ['ds', 'playoff', 'superbowl']][-10:]
```

|  | DS | PLAYOFF | SUPERBOWL |
| --- | --- | --- | --- |
| 2190 | 2014-02-02 | 1.223965 | 1.201517 |
| 2191 | 2014-02-03 | 1.901742 | 1.460471 |
| 2532 | 2015-01-11 | 1.223965 | 0.000000 |
| 2533 | 2015-01-12 | 1.901742 | 0.000000 |
| 2901 | 2016-01-17 | 1.223965 | 0.000000 |
| 2902 | 2016-01-18 | 1.901742 | 0.000000 |
| 2908 | 2016-01-24 | 1.223965 | 0.000000 |
| 2909 | 2016-01-25 | 1.901742 | 0.000000 |
| 2922 | 2016-02-07 | 1.223965 | 1.201517 |
| 2923 | 2016-02-08 | 1.901742 | 1.460471 |

The holiday effects will also show up in the components plot, where we see that there is a spike on the days around playoff appearances, with an especially large spike for the superbowl:

节日效应也将在成分图中体现出来，我们可以看到在季后赛前后几天会出现一个峰值，超级碗的峰值尤其大：

```r
# R
prophet_plot_components(m, forecast)
```

```python
# Python
fig = m.plot_components(forecast)
```

![seasonality,_holiday_effects,_and_regressors_14_0.png](../seasonality,_holiday_effects,_and_regressors_14_0.png)

Individual holidays can be plotted using the `plot_forecast_component` function (imported from `prophet.plot` in Python) like `plot_forecast_component(m, forecast, 'superbowl')` to plot just the superbowl holiday component.

可以使用`plot_forecast_component`函数（从`prophet.plot`Python 导入）绘制单个假期，例如`plot_forecast_component(m, forecast, 'superbowl')`仅绘制超级碗假期部分。

- - -

## Built-in Country Holidays（内置国家假期）

You can use a built-in collection of country-specific holidays using the `add_country_holidays` method (Python) or function (R). The name of the country is specified, and then major holidays for that country will be included in addition to any holidays that are specified via the `holidays` argument described above:

您可以使用方法 (Python) 或函数 (R) 使用内置的国家/地区特定假日集合。指定国家/地区的名称，然后除了通过上述参数`add_country_holidays`指定的任何假日外，还将包括该国的主要假日：`holidays`

```r
# R
m <- prophet(holidays = holidays)
m <- add_country_holidays(m, country_name = 'US')
m <- fit.prophet(m, df)
```

```python
# Python
m = Prophet(holidays=holidays)
m.add_country_holidays(country_name='US')
m.fit(df)
```

You can see which holidays were included by looking at the `train_holiday_names` (Python) or `train.holiday.names` (R) attribute of the model:

您可以通过查看模型的`train_holiday_names`(Python) 或(R) 属性来查看包含了哪些假期：`train.holiday.names`

```r
# R
m$train.holiday.names
```

```r
 [1] "playoff"                     "superbowl"                  
 [3] "New Year's Day"              "Martin Luther King Jr. Day" 
 [5] "Washington's Birthday"       "Memorial Day"               
 [7] "Independence Day"            "Labor Day"                  
 [9] "Columbus Day"                "Veterans Day"               
[11] "Veterans Day (Observed)"     "Thanksgiving"               
[13] "Christmas Day"               "Independence Day (Observed)"
[15] "Christmas Day (Observed)"    "New Year's Day (Observed)"  
```

```python
# Python
m.train_holiday_names
```

```python
0                         playoff
1                       superbowl
2                  New Year's Day
3      Martin Luther King Jr. Day
4           Washington's Birthday
5                    Memorial Day
6                Independence Day
7                       Labor Day
8                    Columbus Day
9                    Veterans Day
10                   Thanksgiving
11                  Christmas Day
12       Christmas Day (Observed)
13        Veterans Day (Observed)
14    Independence Day (Observed)
15      New Year's Day (Observed)
dtype: object
```

The holidays for each country are provided by the `holidays` package in Python. A list of available countries, and the country name to use, is available on their page: https://github.com/vacanza/python-holidays/.

In Python, most holidays are computed deterministically and so are available for any date range; a warning will be raised if dates fall outside the range supported by that country. In R, holiday dates are computed for 1995 through 2044 and stored in the package as `data-raw/generated_holidays.csv`. If a wider date range is needed, this script can be used to replace that file with a different date range: https://github.com/facebook/prophet/blob/main/python/scripts/generate_holidays_file.py.

As above, the country-level holidays will then show up in the components plot:

Python 包提供了每个国家的节假日`holidays`。可用国家/地区列表以及要使用的国家/地区名称可在其页面上找到：https://github.com/vacanza/python-holidays/。

在 Python 中，大多数节假日都是确定性计算的，因此适用于任何日期范围；如果日期超出该国支持的范围，则会发出警告。在 R 中，节假日日期计算范围为 1995 年至 2044 年，并存储在包中`data-raw/generated_holidays.csv`。如果需要更宽的日期范围，可以使用此脚本将该文件替换为不同的日期范围：https://github.com/facebook/prophet/blob/main/python/scripts/generate_holidays_file.py。

如上所述，国家级假期将显示在组件图中：

```r
# R
forecast <- predict(m, future)
prophet_plot_components(m, forecast)
```

```python
# Python
forecast = m.predict(future)
fig = m.plot_components(forecast)
```

![seasonality,_holiday_effects,_and_regressors_24_0.png](../seasonality,_holiday_effects,_and_regressors_24_0.png)

- - -

## Holidays for subdivisions (Python) （Python细分假期）

We can use the utility function `make_holidays_df` to easily create custom holidays DataFrames, e.g. for certain states, using data from the [**holidays**](https://pypi.org/project/holidays/) package. This can be passed directly to the `Prophet()` constructor.

我们可以使用实用函数`make_holidays_df`轻松创建自定义假日 DataFrames，例如针对某些州，使用来自[**假日**](https://pypi.org/project/holidays/)包的数据。这可以直接传递给`Prophet()`构造函数。

```python
# Python
from prophet.make_holidays import make_holidays_df

nsw_holidays = make_holidays_df(
    year_list=[2019 + i for i in range(10)], country='AU', province='NSW'
)
nsw_holidays.head(n=10)
```

|  | DS | HOLIDAY |
| --- | --- | --- |
| 0 | 2019-01-26 | Australia Day |
| 1 | 2019-01-28 | Australia Day (Observed) |
| 2 | 2019-04-25 | Anzac Day |
| 3 | 2019-12-25 | Christmas Day |
| 4 | 2019-12-26 | Boxing Day |
| 5 | 2019-04-20 | Easter Saturday |
| 6 | 2019-04-21 | Easter Sunday |
| 7 | 2019-10-07 | Labour Day |
| 8 | 2019-06-10 | Queen's Birthday |
| 9 | 2019-08-05 | Bank Holiday |

```python
# Python
from prophet import Prophet

m_nsw = Prophet(holidays=nsw_holidays)
```

- - -

## Fourier Order for Seasonalities（季节性的傅里叶顺序）

Seasonalities are estimated using a partial Fourier sum. See [**the paper**](https://peerj.com/preprints/3190/) for complete details, and [**this figure on Wikipedia**](https://en.wikipedia.org/wiki/Fourier_series#/media/File:Fourier_Series.svg) for an illustration of how a partial Fourier sum can approximate an arbitrary periodic signal. The number of terms in the partial sum (the order) is a parameter that determines how quickly the seasonality can change. To illustrate this, consider the Peyton Manning data from the [**Quickstart**](https://facebook.github.io/prophet/docs/quick_start.html). The default Fourier order for yearly seasonality is 10, which produces this fit:

季节性是使用部分傅里叶和估计的。有关完整详细信息，请参阅[**论文**](https://peerj.com/preprints/3190/)以及[**维基百科上的此图**](https://en.wikipedia.org/wiki/Fourier_series#/media/File:Fourier_Series.svg)，以说明部分傅里叶和如何近似任意周期信号。部分和中的项数（阶数）是决定季节性变化速度的参数。为了说明这一点，请考虑[**Quickstart**](https://facebook.github.io/prophet/docs/quick_start.html)中的 Peyton Manning 数据。年度季节性的默认傅里叶阶数为 10，从而产生以下拟合：

```r
# R
m <- prophet(df)
prophet:::plot_yearly(m)
```

```python
# Python
from prophet.plot import plot_yearly
m = Prophet().fit(df)
a = plot_yearly(m)
```

![seasonality,_holiday_effects,_and_regressors_30_0.png](../seasonality,_holiday_effects,_and_regressors_30_0.png)

The default values are often appropriate, but they can be increased when the seasonality needs to fit higher-frequency changes, and generally be less smooth. The Fourier order can be specified for each built-in seasonality when instantiating the model, here it is increased to 20:

默认值通常是合适的，但当季节性需要适应更高频率的变化时，可以增加默认值，并且通常不太平滑。在实例化模型时，可以为每个内置季节性指定傅里叶阶数，这里将其增加到 20：

```r
# R
m <- prophet(df, yearly.seasonality = 20)
prophet:::plot_yearly(m)
```

```python
# Python
from prophet.plot import plot_yearly
m = Prophet(yearly_seasonality=20).fit(df)
a = plot_yearly(m)
```

![seasonality,_holiday_effects,_and_regressors_33_0.png](../seasonality,_holiday_effects,_and_regressors_33_0.png)

Increasing the number of Fourier terms allows the seasonality to fit faster changing cycles, but can also lead to overfitting: N Fourier terms corresponds to 2N variables used for modeling the cycle

增加傅里叶项的数量可以使季节性适应更快变化的周期，但也可能导致过度拟合：N 个傅里叶项对应用于建模周期的 2N 个变量

- - -

## Specifying Custom Seasonalities（指定自定义季节性）

Prophet will by default fit weekly and yearly seasonalities, if the time series is more than two cycles long. It will also fit daily seasonality for a sub-daily time series. You can add other seasonalities (monthly, quarterly, hourly) using the `add_seasonality` method (Python) or function (R).

The inputs to this function are a name, the period of the seasonality in days, and the Fourier order for the seasonality. For reference, by default Prophet uses a Fourier order of 3 for weekly seasonality and 10 for yearly seasonality. An optional input to `add_seasonality` is the prior scale for that seasonal component - this is discussed below.

As an example, here we fit the Peyton Manning data from the [**Quickstart**](https://facebook.github.io/prophet/docs/quick_start.html), but replace the weekly seasonality with monthly seasonality. The monthly seasonality then will appear in the components plot:

如果时间序列长度超过两个周期，Prophet 将默认拟合每周和每年的季节性。它还将拟合每日以下时间序列的每日季节性。您可以使用方法`add_seasonality`(Python) 或函数 (R) 添加其他季节性（每月、每季度、每小时）。

此函数的输入包括名称、季节性周期（以天为单位）和季节性的傅里叶阶数。作为参考，默认情况下，Prophet 对每周季节性使用傅里叶阶数 3，对每年季节性使用傅里叶阶数 10。可选输入是`add_seasonality`该季节性成分的先验尺度 - 下文将对此进行讨论。

举个例子，我们在这里拟合[**Quickstart**](https://facebook.github.io/prophet/docs/quick_start.html)中的 Peyton Manning 数据，但将每周季节性替换为每月季节性。然后，每月季节性将出现在组件图中：

```r
# R
m <- prophet(weekly.seasonality=FALSE)
m <- add_seasonality(m, name='monthly', period=30.5, fourier.order=5)
m <- fit.prophet(m, df)
forecast <- predict(m, future)
prophet_plot_components(m, forecast)
```

```python
# Python
m = Prophet(weekly_seasonality=False)
m.add_seasonality(name='monthly', period=30.5, fourier_order=5)
forecast = m.fit(df).predict(future)
fig = m.plot_components(forecast)
```

![seasonality,_holiday_effects,_and_regressors_36_0.png](../seasonality,_holiday_effects,_and_regressors_36_0.png)

- - -

## Seasonalities that depend on other factors（受其他因素影响的季节性）

In some instances the seasonality may depend on other factors, such as a weekly seasonal pattern that is different during the summer than it is during the rest of the year, or a daily seasonal pattern that is different on weekends vs. on weekdays. These types of seasonalities can be modeled using conditional seasonalities.

Consider the Peyton Manning example from the [**Quickstart**](https://facebook.github.io/prophet/docs/quick_start.html). The default weekly seasonality assumes that the pattern of weekly seasonality is the same throughout the year, but we’d expect the pattern of weekly seasonality to be different during the on-season (when there are games every Sunday) and the off-season. We can use conditional seasonalities to construct separate on-season and off-season weekly seasonalities.

First we add a boolean column to the dataframe that indicates whether each date is during the on-season or the off-season:

在某些情况下，季节性可能取决于其他因素，例如夏季的周季节性模式与一年中其他时间的季节性模式不同，或者周末与工作日的每日季节性模式不同。这些类型的季节性可以使用条件季节性进行建模。

考虑[**Quickstart**](https://facebook.github.io/prophet/docs/quick_start.html)中的 Peyton Manning 示例。默认的每周季节性假设每周季节性的模式全年相同，但我们预计每周季节性的模式在赛季开始（每周日都有比赛）和休赛期会有所不同。我们可以使用条件季节性来构建单独的赛季开始和休赛期每周季节性。

首先，我们向数据框添加一个布尔列，指示每个日期是在旺季还是淡季：

```r
# R
is_nfl_season <- function(ds) {
  dates <- as.Date(ds)
  month <- as.numeric(format(dates, '%m'))
  return(month > 8 | month < 2)
}
df$on_season <- is_nfl_season(df$ds)
df$off_season <- !is_nfl_season(df$ds)
```

```python
# Python
def is_nfl_season(ds):
    date = pd.to_datetime(ds)
    return (date.month > 8 or date.month < 2)

df['on_season'] = df['ds'].apply(is_nfl_season)
df['off_season'] = ~df['ds'].apply(is_nfl_season)
```

Then we disable the built-in weekly seasonality, and replace it with two weekly seasonalities that have these columns specified as a condition. This means that the seasonality will only be applied to dates where the `condition_name` column is `True`. We must also add the column to the `future` dataframe for which we are making predictions.

然后我们禁用内置的每周季节性，并将其替换为两个每周季节性，这些列指定为条件。这意味着季节性仅适用于列为的日期`condition_name`。`True`我们还必须将列添加到`future`我们要进行预测的数据框中。

```r
# R
m <- prophet(weekly.seasonality=FALSE)
m <- add_seasonality(m, name='weekly_on_season', period=7, fourier.order=3, condition.name='on_season')
m <- add_seasonality(m, name='weekly_off_season', period=7, fourier.order=3, condition.name='off_season')
m <- fit.prophet(m, df)

future$on_season <- is_nfl_season(future$ds)
future$off_season <- !is_nfl_season(future$ds)
forecast <- predict(m, future)
prophet_plot_components(m, forecast)
```

```python
# Python
m = Prophet(weekly_seasonality=False)
m.add_seasonality(name='weekly_on_season', period=7, fourier_order=3, condition_name='on_season')
m.add_seasonality(name='weekly_off_season', period=7, fourier_order=3, condition_name='off_season')

future['on_season'] = future['ds'].apply(is_nfl_season)
future['off_season'] = ~future['ds'].apply(is_nfl_season)
forecast = m.fit(df).predict(future)
fig = m.plot_components(forecast)
```

![seasonality,_holiday_effects,_and_regressors_42_0.png](../seasonality,_holiday_effects,_and_regressors_42_0.png)

Both of the seasonalities now show up in the components plots above. We can see that during the on-season when games are played every Sunday, there are large increases on Sunday and Monday that are completely absent during the off-season.

现在，上述成分图中显示了这两种季节性。我们可以看到，在每周日都有比赛的旺季，周日和周一的比赛数量会大幅增加，而在淡季则完全没有。

- - -

## Prior scale for holidays and seasonality（假期和季节性的优先尺度）

If you find that the holidays are overfitting, you can adjust their prior scale to smooth them using the parameter `holidays_prior_scale`. By default this parameter is 10, which provides very little regularization. Reducing this parameter dampens holiday effects:

如果你发现节假日过度拟合，你可以使用参数 调整先验尺度来平滑它们`holidays_prior_scale`。默认情况下，此参数为 10，这提供的正则化非常小。降低此参数可减弱节假日效应：

```r
# R
m <- prophet(df, holidays = holidays, holidays.prior.scale = 0.05)
forecast <- predict(m, future)
forecast %>% 
  select(ds, playoff, superbowl) %>% 
  filter(abs(playoff + superbowl) > 0) %>%
  tail(10)
```

```python
# Python
m = Prophet(holidays=holidays, holidays_prior_scale=0.05).fit(df)
forecast = m.predict(future)
forecast[(forecast['playoff'] + forecast['superbowl']).abs() > 0][
    ['ds', 'playoff', 'superbowl']][-10:]
```

|  | DS | PLAYOFF | SUPERBOWL |
| --- | --- | --- | --- |
| 2190 | 2014-02-02 | 1.206086 | 0.964914 |
| 2191 | 2014-02-03 | 1.852077 | 0.992634 |
| 2532 | 2015-01-11 | 1.206086 | 0.000000 |
| 2533 | 2015-01-12 | 1.852077 | 0.000000 |
| 2901 | 2016-01-17 | 1.206086 | 0.000000 |
| 2902 | 2016-01-18 | 1.852077 | 0.000000 |
| 2908 | 2016-01-24 | 1.206086 | 0.000000 |
| 2909 | 2016-01-25 | 1.852077 | 0.000000 |
| 2922 | 2016-02-07 | 1.206086 | 0.964914 |
| 2923 | 2016-02-08 | 1.852077 | 0.992634 |

The magnitude of the holiday effect has been reduced compared to before, especially for superbowls, which had the fewest observations. There is a parameter `seasonality_prior_scale` which similarly adjusts the extent to which the seasonality model will fit the data.

Prior scales can be set separately for individual holidays by including a column `prior_scale` in the holidays dataframe. Prior scales for individual seasonalities can be passed as an argument to `add_seasonality`. For instance, the prior scale for just weekly seasonality can be set using:

与之前相比，节日效应的幅度有所减小，尤其是超级碗，它的观测值最少。有一个参数`seasonality_prior_scale`可以同样调整季节性模型与数据的拟合程度。

通过在节日数据框中包含一列，可以分别为各个节日设置先验尺度`prior_scale`。各个季节性的先验尺度可以作为参数传递给`add_seasonality`。例如，可以使用以下方法设置仅针对每周季节性的先验尺度：

```r
# R
m <- prophet()
m <- add_seasonality(
  m, name='weekly', period=7, fourier.order=3, prior.scale=0.1)
```

```python
# Python
m = Prophet()
m.add_seasonality(
    name='weekly', period=7, fourier_order=3, prior_scale=0.1)
```

- - -

## Additional regressors（额外的回归量）

Additional regressors can be added to the linear part of the model using the `add_regressor` method or function. A column with the regressor value will need to be present in both the fitting and prediction dataframes. For example, we can add an additional effect on Sundays during the NFL season. On the components plot, this effect will show up in the ‘extra_regressors’ plot:

可以使用方法或函数将额外的回归量添加到模型的线性部分`add_regressor`。拟合和预测数据框中都需要存在具有回归量值的列。例如，我们可以在 NFL 赛季期间的周日添加额外的效果。在组件图上，此效果将显示在“extra_regressors”图中：

```r
# R
nfl_sunday <- function(ds) {
  dates <- as.Date(ds)
  month <- as.numeric(format(dates, '%m'))
  as.numeric((weekdays(dates) == "Sunday") & (month > 8 | month < 2))
}
df$nfl_sunday <- nfl_sunday(df$ds)

m <- prophet()
m <- add_regressor(m, 'nfl_sunday')
m <- fit.prophet(m, df)

future$nfl_sunday <- nfl_sunday(future$ds)

forecast <- predict(m, future)
prophet_plot_components(m, forecast)
```

```python
# Python
def nfl_sunday(ds):
    date = pd.to_datetime(ds)
    if date.weekday() == 6 and (date.month > 8 or date.month < 2):
        return 1
    else:
        return 0
df['nfl_sunday'] = df['ds'].apply(nfl_sunday)

m = Prophet()
m.add_regressor('nfl_sunday')
m.fit(df)

future['nfl_sunday'] = future['ds'].apply(nfl_sunday)

forecast = m.predict(future)
fig = m.plot_components(forecast)
```

![seasonality,_holiday_effects,_and_regressors_52_0.png](../seasonality,_holiday_effects,_and_regressors_52_0.png)

NFL Sundays could also have been handled using the “holidays” interface described above, by creating a list of past and future NFL Sundays. The `add_regressor` function provides a more general interface for defining extra linear regressors, and in particular does not require that the regressor be a binary indicator. Another time series could be used as a regressor, although its future values would have to be known.

[**This notebook**](https://nbviewer.jupyter.org/github/nicolasfauchereau/Auckland_Cycling/blob/master/notebooks/Auckland_cycling_and_weather.ipynb) shows an example of using weather factors as extra regressors in a forecast of bicycle usage, and provides an excellent illustration of how other time series can be included as extra regressors.

The `add_regressor` function has optional arguments for specifying the prior scale (holiday prior scale is used by default) and whether or not the regressor is standardized - see the docstring with `help(Prophet.add_regressor)` in Python and `?add_regressor` in R. Note that regressors must be added prior to model fitting. Prophet will also raise an error if the regressor is constant throughout the history, since there is nothing to fit from it.

The extra regressor must be known for both the history and for future dates. It thus must either be something that has known future values (such as `nfl_sunday`), or something that has separately been forecasted elsewhere. The weather regressors used in the notebook linked above is a good example of an extra regressor that has forecasts that can be used for future values.

One can also use as a regressor another time series that has been forecasted with a time series model, such as Prophet. For instance, if `r(t)` is included as a regressor for `y(t)`, Prophet can be used to forecast `r(t)` and then that forecast can be plugged in as the future values when forecasting `y(t)`. A note of caution around this approach: This will probably not be useful unless `r(t)` is somehow easier to forecast than `y(t)`. This is because error in the forecast of `r(t)` will produce error in the forecast of `y(t)`. One setting where this can be useful is in hierarchical time series, where there is top-level forecast that has higher signal-to-noise and is thus easier to forecast. Its forecast can be included in the forecast for each lower-level series.

Extra regressors are put in the linear component of the model, so the underlying model is that the time series depends on the extra regressor as either an additive or multiplicative factor (see the next section for multiplicativity).

也可以使用上面描述的“节假日”界面来处理 NFL 周日，方法是创建过去和未来的 NFL 周日列表。该`add_regressor`函数提供了一个更通用的界面来定义额外的线性回归量，特别是不需要回归量是二元指标。另一个时间序列可以用作回归量，尽管必须知道它的未来值。

[**This notebook**](https://nbviewer.jupyter.org/github/nicolasfauchereau/Auckland_Cycling/blob/master/notebooks/Auckland_cycling_and_weather.ipynb)展示了使用天气因素作为自行车使用量预测的额外回归量的示例，并很好地说明了如何将其他时间序列作为额外回归量。

该`add_regressor`函数具有可选参数，用于指定先验尺度（默认情况下使用假日先验尺度）以及回归量是否标准化 - 请参阅`help(Prophet.add_regressor)`Python 和`?add_regressor`R 中的文档字符串。请注意，必须在模型拟合之前添加回归量。如果回归量在整个历史中保持不变，Prophet 也会引发错误，因为没有任何内容可以拟合。

额外的回归量必须既有历史数据，也有未来数据。因此，它要么是具有已知未来值的东西（例如`nfl_sunday`），要么是已在其他地方单独预测的东西。上面链接的笔记本中使用的天气回归量是一个很好的例子，它具有可用于未来值的预测。

您还可以使用已用时间序列模型（例如 Prophet）预测过的另一个时间序列作为回归量。例如，如果`r(t)`作为的回归量`y(t)`，则可以使用 Prophet 进行预测`r(t)`，然后在预测时将该预测作为未来值插入`y(t)`。关于这种方法需要注意的是：除非`r(t)`比更容易预测，否则这种方法可能没有用`y(t)`。这是因为的预测错误`r(t)`会导致的预测错误`y(t)`。这种方法在分层时间序列中很有用，其中顶层预测具有更高的信噪比，因此更容易预测。它的预测可以包含在每个较低级别序列的预测中。

额外的回归量被放置在模型的线性分量中，因此底层模型是时间序列依赖于额外的回归量作为加法或乘法因子（有关乘法性，请参阅下一节）。

- - -

## Coefficients of additional regressors（附加回归量的系数）

To extract the beta coefficients of the extra regressors, use the utility function `regressor_coefficients` (`from prophet.utilities import regressor_coefficients` in Python, `prophet::regressor_coefficients` in R) on the fitted model. The estimated beta coefficient for each regressor roughly represents the increase in prediction value for a unit increase in the regressor value (note that the coefficients returned are always on the scale of the original data). If `mcmc_samples` is specified, a credible interval for each coefficient is also returned, which can help identify whether the regressor is meaningful to the model (a credible interval that includes the 0 value suggests the regressor is not meaningful).

要提取额外回归量的 beta 系数，请在拟合模型上使用效用函数`regressor_coefficients`（`from prophet.utilities import regressor_coefficients`在 Python 中，在 R 中）。每个回归量的估计 beta 系数大致表示回归量值每增加一个单位，预测值的增加量（请注意，返回的系数始终在原始数据的范围内）。如果指定，还会返回每个系数的可信区间，这有助于确定回归量对模型是否有意义（包含 0 值的可信区间表明回归量没有意义）。`prophet::regressor_coefficientsmcmc_samples`