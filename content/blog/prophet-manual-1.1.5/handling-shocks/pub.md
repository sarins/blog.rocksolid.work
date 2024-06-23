+++
title = 'Prophet 应对冲击'
subtitle = 'Prophet Handling Shocks'
brief = ''
date = 2024-06-10T00:00:10
categories = ['Prophet', 'Anaconda']
series = ['Prophet']
tags = ['Prophet', 'Anaconda', 'Docker', 'Python']
type = 'blog'
+++

- - -

```python
# Python
%matplotlib inline
from prophet import Prophet
import pandas as pd
from matplotlib import pyplot as plt
import logging
logging.getLogger('prophet').setLevel(logging.ERROR)
import warnings
warnings.filterwarnings("ignore")

plt.rcParams['figure.figsize'] = 9, 6
```

As a result of the lockdowns caused by the COVID-19 pandemic, many time series experienced “shocks” during 2020, e.g. spikes in media consumption (Netflix, YouTube), e-commerce transactions (Amazon, eBay), whilst attendance to in-person events declined dramatically.

Most of these time series would also maintain their new level for a period of time, subject to fluctuations driven by easing of lockdowns and/or vaccines.

Seasonal patterns could have also changed: for example, people may have consumed less media (in total hours) on weekdays compared to weekends before the COVID lockdowns, but during lockdowns weekday consumption could be much closer to weekend consumption.

In this page we’ll explore some strategies for capturing these effects using Prophet’s functionality:

1. Marking step changes / spikes due to COVID events as once-off.
2. Sustained changes in behaviour leading to trend and seasonality changes.

由于 COVID-19 大流行造成的封锁，许多时间序列在 2020 年经历了“冲击”，例如媒体消费（Netflix、YouTube）、电子商务交易（亚马逊、eBay）激增，而现场活动的参加人数急剧下降。

这些时间序列中的大多数也将在一段时间内保持其新的水平，但会受到封锁放松和/或疫苗接种引起的波动的影响。

季节性模式也可能发生了变化：例如，在新冠疫情封锁之前，人们在工作日消费的媒体时间（总小时数）可能比周末少，但在封锁期间，工作日的消费量可能更接近周末的消费量。

在此页面中，我们将探讨使用 Prophet 的功能捕捉这些效果的一些策略：

1. 将由于 COVID 事件导致的步骤变化/峰值标记为一次性。
2. 行为的持续变化导致趋势和季节性的变化。

- - -

## Case Study - Pedestrian Activity（案例研究——行人活动）

For this case study we’ll use [**Pedestrian Sensor data from the City of Melbourne**](https://data.melbourne.vic.gov.au/Transport/Pedestrian-Counting-System-Monthly-counts-per-hour/b2ak-trbp). This data measures foot traffic from sensors in various places in the central business district, and we’ve chosen one sensor (`Sensor_ID = 4`) and aggregated the values to a daily grain.

The aggregated dataset can be found in the examples folder [**here**](https://github.com/facebook/prophet/tree/master/examples/example_pedestrians_covid.csv).

在本案例研究中，我们将使用来自[**墨尔本市的行人传感器数据**](https://data.melbourne.vic.gov.au/Transport/Pedestrian-Counting-System-Monthly-counts-per-hour/b2ak-trbp)。该数据通过中央商务区各个位置的传感器测量人流量，我们选择了一个传感器（`Sensor_ID = 4`）并将值汇总为每日粒度。

聚合数据集可在[**此处**](https://github.com/facebook/prophet/tree/master/examples/example_pedestrians_covid.csv)的示例文件夹中找到。

```python
# Python
df = pd.read_csv('https://raw.githubusercontent.com/facebook/prophet/main/examples/example_pedestrians_covid.csv')
```

```python
# Python
df.set_index('ds').plot();
```

![handling_shocks_4_0.png](../handling_shocks_4_0.png)

We can see two key events in the time series:

- The initial drop in foot traffic around 21 March 2020, which started to recover around 6 June 2020. This corresponds to the declaration of a pandemic by WHO and subsequent lockdowns mandated by the Victorian government.
- After some slow recovery, a second drop in foot traffic around July 9 2020, which began to recover around 27 October 2020. This corresponds to the “second wave” of the pandemic in metropolitan Melbourne.

There are also shorter periods of strict lockdown that lead to sudden tips in the time series: 5 days in February 2021, and 14 days in early June 2021.

我们可以在时间序列中看到两个关键事件：

- 客流量在 2020 年 3 月 21 日左右开始下降，并在 2020 年 6 月 6 日左右开始恢复。这与世界卫生组织宣布大流行以及维多利亚州政府随后强制实施封锁的时间相吻合。
- 经过一段缓慢的恢复后，2020 年 7 月 9 日左右客流量第二次下降，并于 2020 年 10 月 27 日左右开始恢复。这对应于墨尔本大都会区疫情的“第二波”。

也有一些较短的严格封锁期导致时间序列出现突然上升：2021 年 2 月为 5 天，2021 年 6 月初为 14 天。

- - -

## Default model without any adjustments（默认模型，无任何调整）

First we’ll fit a model with the default Prophet settings:

首先，我们将使用默认的 Prophet 设置来拟合模型：

```python
# Python
m = Prophet()
m = m.fit(df)
future = m.make_future_dataframe(periods=366)
forecast = m.predict(future)
```

```python
02:53:41 - cmdstanpy - INFO - Chain [1] start processing
02:53:41 - cmdstanpy - INFO - Chain [1] done processing
```

```python
# Python
m.plot(forecast)
plt.axhline(y=0, color='red')
plt.title('Default Prophet');
```

![handling_shocks_8_0.png](../handling_shocks_8_0.png)

```python
# Python
m.plot_components(forecast);
```

![handling_shocks_9_0.png](../handling_shocks_9_0.png)

The model seems to fit reasonably to past data, but notice how we’re capturing the dips, and the spikes after the dips, as a part of the trend component.

By default, the model assumes that these large spikes are possible in the future, even though we realistically won’t see something of the same magnitude within our forecast horizon (1 year in this case). This leads to a fairly optimistic forecast of the recovery of foot traffic in 2022.

该模型似乎与过去的数据合理契合，但请注意我们如何捕捉下降趋势以及下降趋势后的峰值作为趋势成分的一部分。

默认情况下，该模型假设未来可能会出现这些大幅增长，尽管我们实际上不会在预测期内（在本例中为 1 年）看到同样规模的峰值。这导致对 2022 年人流量恢复的预测相当乐观。

- - -

## Treating COVID-19 lockdowns as a one-off holidays（将新冠疫情封锁视为一次性假期）

To prevent large dips and spikes from being captured by the trend component, we can treat the days impacted by COVID-19 as holidays that will not repeat again in the future. Adding custom holidays is explained in more detail [**here**](https://facebook.github.io/prophet/docs/seasonality,_holiday_effects,_and_regressors.html#modeling-holidays-and-special-events). We set up a DataFrame like so to describe the periods affected by lockdowns:

为了防止趋势成分捕捉到大幅下降和上升，我们可以将受 COVID-19 影响的日子视为将来不会再重复的假期。[**此处**](https://facebook.github.io/prophet/docs/seasonality,_holiday_effects,_and_regressors.html#modeling-holidays-and-special-events)更详细地解释了如何添加自定义假期。我们设置了一个 DataFrame 来描述受封锁影响的时期：

```python
# Python
lockdowns = pd.DataFrame([
    {'holiday': 'lockdown_1', 'ds': '2020-03-21', 'lower_window': 0, 'ds_upper': '2020-06-06'},
    {'holiday': 'lockdown_2', 'ds': '2020-07-09', 'lower_window': 0, 'ds_upper': '2020-10-27'},
    {'holiday': 'lockdown_3', 'ds': '2021-02-13', 'lower_window': 0, 'ds_upper': '2021-02-17'},
    {'holiday': 'lockdown_4', 'ds': '2021-05-28', 'lower_window': 0, 'ds_upper': '2021-06-10'},
])
for t_col in ['ds', 'ds_upper']:
    lockdowns[t_col] = pd.to_datetime(lockdowns[t_col])
lockdowns['upper_window'] = (lockdowns['ds_upper'] - lockdowns['ds']).dt.days
lockdowns
```

|  | HOLIDAY | DS | LOWER_WINDOW | DS_UPPER | UPPER_WINDOW |
| --- | --- | --- | --- | --- | --- |
| 0 | lockdown_1 | 2020-03-21 | 0 | 2020-06-06 | 77 |
| 1 | lockdown_2 | 2020-07-09 | 0 | 2020-10-27 | 110 |
| 2 | lockdown_3 | 2021-02-13 | 0 | 2021-02-17 | 4 |
| 3 | lockdown_4 | 2021-05-28 | 0 | 2021-06-10 | 13 |

- We have an entry for each lockdown period, with `ds` specifying the start of the lockdown. `ds_upper` is not used by Prophet, but it’s a convenient way for us to calculate `upper_window`.
- `upper_window` tells Prophet that the lockdown spans for x days after the start of the lockdown. Note that the holidays regression is inclusive of the upper bound.

Note that since we don’t specify any future dates, Prophet will assume that these holidays will *not* occur again when creating the future dataframe (and hence they won’t affect our projections). This is different to how we would specify a recurring holiday.

- 我们为每个封锁期都有一个条目，其中`ds`指定了封锁的开始时间。Prophet`ds_upper`没有使用，但它是我们计算的一种方便的方法`upper_window`。
- `upper_window`告诉 Prophet 封锁开始后持续 x 天。请注意，假期回归包含上限。

请注意，由于我们没有指定任何未来日期，因此 Prophet 会在创建未来数据框时假定这些假期不会*再次*发生（因此它们不会影响我们的预测）。这与我们指定重复假期的方式不同。

```python
# Python
m2 = Prophet(holidays=lockdowns)
m2 = m2.fit(df)
future2 = m2.make_future_dataframe(periods=366)
forecast2 = m2.predict(future2)
```

```python
02:53:44 - cmdstanpy - INFO - Chain [1] start processing
02:53:45 - cmdstanpy - INFO - Chain [1] done processing
```

```python
# Python
m2.plot(forecast2)
plt.axhline(y=0, color='red')
plt.title('Lockdowns as one-off holidays');
```

![handling_shocks_15_0.png](../handling_shocks_15_0.png)

```python
# Python
m2.plot_components(forecast2);
```

![handling_shocks_16_0.png](../handling_shocks_16_0.png)

- Prophet is sensibly assigning a large negative effect to the days within the lockdown periods.
- The forecast for the trend is not as strong / optimistic and seems fairly reasonable.
- 先知明智地认为封锁期间的日子产生了巨大的负面影响。
- 对趋势的预测并不那么强烈/乐观，但似乎相当合理。

- - -

## Sense checking the trend（感知趋势）

In an environment when behaviours are constantly changing, it’s important to ensure that the trend component of the model is capturing to emerging patterns without overfitting to them.

The [**trend changepoints**](https://facebook.github.io/prophet/docs/trend_changepoints.html) documentation explains two things we could tweak with the trend component:

- The changepoint locations, which by default are evenly spaced across 80% of the history. We should be mindful of where this range ends, and extend the range (either by increasing the % or adding changepoints manually) if we believe the most recent data better reflects future behaviour.
- The strength of regularisation (`changepoint_prior_scale`), which determines how flexible the trend is; the default value is `0.05` and increasing this will allow the trend to fit more closely to the observed data.

We plot the trend component and changepoints detected by our current model below.

在行为不断变化的环境中，重要的是确保模型的趋势部分能够捕捉到新出现的模式而不会过度拟合。

[**趋势变化点**](https://facebook.github.io/prophet/docs/trend_changepoints.html)文档解释了我们可以使用趋势组件调整的两件事：

- 默认情况下，变化点位置均匀分布在 80% 的历史记录中。我们应该注意这个范围的结束位置，如果我们认为最新的数据能更好地反映未来的行为，则应扩展范围（通过增加百分比或手动添加变化点）。
- 正则化的强度（`changepoint_prior_scale`），它决定了趋势的灵活性；默认值为，`0.05`增加该值将使趋势更紧密地拟合观察到的数据。

我们在下面绘制了当前模型检测到的趋势成分和变化点。

```python
# Python
from prophet.plot import add_changepoints_to_plot
fig = m2.plot(forecast2)
a = add_changepoints_to_plot(fig.gca(), m2, forecast2)
```

![handling_shocks_18_0.png](../handling_shocks_18_0.png)

The detected changepoints look reasonable, and the future trend tracks the latest upwards trend in activity, but not to the extent of late 2020. This seems suitable for a best guess of future activity.

We can see what the forecast would look like if we wanted to emphasize COVID patterns more in model training; we can do this by adding more potential changepoints after 2020 and making the trend more flexible.

检测到的变化点看起来合理，未来趋势跟踪活动的最新上升趋势，但不会达到 2020 年末的程度。这似乎适合对未来活动的最佳猜测。

如果我们想在模型训练中更多地强调 COVID 模式，我们可以看到预测会是什么样子；我们可以通过在 2020 年之后添加更多潜在变化点并使趋势更加灵活来做到这一点。

```python
# Python
m3_changepoints = (
    # 10 potential changepoints in 2.5 years
    pd.date_range('2017-06-02', '2020-01-01', periods=10).date.tolist() + 
    # 15 potential changepoints in 1 year 2 months
    pd.date_range('2020-02-01', '2021-04-01', periods=15).date.tolist()
)
```

```python
# Python
# Default changepoint_prior_scale is 0.05, so 1.0 will lead to much more flexibility in comparison.
m3 = Prophet(holidays=lockdowns, changepoints=m3_changepoints, changepoint_prior_scale=1.0)
m3 = m3.fit(df)
forecast3 = m3.predict(future2)
```

```python
02:53:49 - cmdstanpy - INFO - Chain [1] start processing
02:53:52 - cmdstanpy - INFO - Chain [1] done processing
```

```python
# Python
from prophet.plot import add_changepoints_to_plot
fig = m3.plot(forecast3)
a = add_changepoints_to_plot(fig.gca(), m3, forecast3)
```

![handling_shocks_22_0.png](../handling_shocks_22_0.png)

We’re seeing many changepoints detected post-COVID, matching the various fluctuations from loosening / tightening lockdowns. Overall the trend curve and forecasted trend are quite similar to our previous model, but we’re seeing a lot more uncertainty because of the higher number of trend changes we picked up in the history.

We probably wouldn’t pick this model over the model with default parameters as a best estimate, but it’s a good demonstration of how we can incorporate our beliefs into the model about which patterns are important to capture.

我们看到，新冠疫情后出现了许多变化点，与放松/收紧封锁带来的各种波动相吻合。总体而言，趋势曲线和预测趋势与我们之前的模型非常相似，但由于我们在历史中发现的趋势变化数量较多，我们看到了更多的不确定性。

我们可能不会选择该模型而不是具有默认参数的模型作为最佳估计，但它很好地展示了我们如何将我们的信念融入到模型中，了解哪些模式是重要的。

- - -

## Changes in seasonality between pre- and post-COVID（疫情前后的季节性变化）

The seasonal component plots in the previous sections show a peak of activity on Friday compared to other days of the week. If we’re not sure whether this will still hold post-lockdown, we can add *conditional seasonalities* to the model. Conditional seasonalities are explained in more detail [**here**](https://facebook.github.io/prophet/docs/seasonality,_holiday_effects,_and_regressors.html#seasonalities-that-depend-on-other-factors).

First we define boolean columns in the history dataframe to flag “pre covid” and “post covid” periods:

前几节中的季节性成分图显示，与一周中的其他日子相比，周五的活动量达到峰值。如果我们不确定封锁结束后这种情况是否仍会持续，我们可以将*条件季节性*添加到模型中。条件季节性的详细解释在[**这里**](https://facebook.github.io/prophet/docs/seasonality,_holiday_effects,_and_regressors.html#seasonalities-that-depend-on-other-factors)。

首先，我们在历史数据框中定义布尔列来标记“新冠疫情之前”和“新冠疫情之后”时期：

```python
# Python
df2 = df.copy()
df2['pre_covid'] = pd.to_datetime(df2['ds']) < pd.to_datetime('2020-03-21')
df2['post_covid'] = ~df2['pre_covid']
```

The conditional seasonality we’re interested in modelling here is the day-of-week (“weekly”) seasonality. To do this, we firstly turn off the default `weekly_seasonality` when we create the Prophet model.

我们在此感兴趣的条件季节性是星期几（“每周”）的季节性。为此，我们`weekly_seasonality`在创建 Prophet 模型时首先关闭默认设置。

```python
# Python
m4 = Prophet(holidays=lockdowns, weekly_seasonality=False)
```

We then add this weekly seasonality manually, as two different model components - one for pre-covid, one for post-covid. Note that `fourier_order=3` is the default setting for weekly seasonality. After this we can run `.fit()`.

然后，我们手动添加这个每周季节性，作为两个不同的模型组件 - 一个用于新冠疫情前，一个用于新冠疫情后。请注意，这`fourier_order=3`是每周季节性的默认设置。之后我们可以运行`.fit()`。

```python
# Python
m4.add_seasonality(
    name='weekly_pre_covid',
    period=7,
    fourier_order=3,
    condition_name='pre_covid',
)
m4.add_seasonality(
    name='weekly_post_covid',
    period=7,
    fourier_order=3,
    condition_name='post_covid',
);
```

```python
# Python
m4 = m4.fit(df2)
```

```python
02:53:55 - cmdstanpy - INFO - Chain [1] start processing
02:53:56 - cmdstanpy - INFO - Chain [1] done processing
```

We also need to create the `pre_covid` and `post_covid` flags in the future dataframe. This is so that Prophet can apply the correct weekly seasonality parameters to each future date.

我们还需要在未来数据框中创建`pre_covid`和`post_covid`标志。这样 Prophet 就可以将正确的每周季节性参数应用于每个未来日期。

```python
# Python
future4 = m4.make_future_dataframe(periods=366)
future4['pre_covid'] = pd.to_datetime(future4['ds']) < pd.to_datetime('2020-03-21')
future4['post_covid'] = ~future4['pre_covid']
```

```python
# Python
forecast4 = m4.predict(future4)
```

```python
# Python
m4.plot(forecast4)
plt.axhline(y=0, color='red')
plt.title('Lockdowns as one-off holidays + Conditional weekly seasonality');
```

![handling_shocks_34_0.png](../handling_shocks_34_0.png)

```python
# Python
m4.plot_components(forecast4);
```

![handling_shocks_35_0.png](../handling_shocks_35_0.png)

Interestingly, the model with conditional seasonalities suggests that, post-COVID, pedestrian activity peaks on Saturdays, instead of Fridays. This could make sense if most people are still working from home and are hence less likely to go out on Friday nights. From a prediction perspective this would only be important if we care about predicting weekdays vs. weekends accurately, but overall this kind of exploration helps us gain insight into how COVID has changed behaviours.

有趣的是，具有条件季节性的模型表明，在新冠疫情之后，行人活动的高峰期在周六，而不是周五。如果大多数人仍在家工作，因此周五晚上不太可能外出，这可能是有道理的。从预测的角度来看，只有当我们关心准确预测工作日和周末时，这一点才重要，但总的来说，这种探索有助于我们深入了解新冠疫情如何改变了人们的行为。

- - -

## Further reading（进一步阅读）

A lot of the content in this page was inspired by this [**GitHub discussion**](https://github.com/facebook/prophet/issues/1416). We’ve covered a few low hanging fruits for tweaking Prophet models when faced with shocks such as COVID, but there are many other possible approaches as well, such as:

- Using external regressors (e.g. the lockdown stringency index). This would only be fruitful if we:
  - a have regressor data that aligns well (in terms of location) with the series we’re forecasting; 
  - b have control over or can predict the regressor much more accurately than the time series alone.
- Detecting and removing outlier data from the training period, or throwing away older training data completely. This might be a better approach for sub-daily time series that don’t have yearly seasonal patterns.

Overall though it’s difficult to be confident in our forecasts in these environments when rules are constantly changing and outbreaks occur randomly. In this scenario it’s more important to constantly re-train / re-evaluate our models and clearly communicate the increased uncertainty in forecasts.

本页中的许多内容都受到[**GitHub 讨论**](https://github.com/facebook/prophet/issues/1416)的启发。我们介绍了一些在面对 COVID 等冲击时调整 Prophet 模型的简单方法，但还有许多其他可能的方法，例如：

- 使用外部回归量（例如封锁严格度指数）。这只有在以下情况下才会有成效：
  - a 我们拥有与我们要预测的序列高度吻合（就位置而言）；
  - b 能够控制或比单独使用时间序列更准确地预测回归量。
- 检测并删除训练期间的异常数据，或完全丢弃较旧的训练数据。对于没有年度季节性模式的亚日时间序列，这可能是更好的方法。

总体而言，在规则不断变化且疫情随机发生的环境中，我们的预测很难令人信服。在这种情况下，更重要的是不断重新训练/重新评估我们的模型，并清楚地传达预测中增加的不确定性。