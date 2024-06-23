+++
title = 'Prophet 饱和预测'
subtitle = 'Prophet Saturating Forecastsb'
brief = ''
date = 2024-06-10T00:00:02
categories = ['Prophet', 'Anaconda']
series = ['Prophet']
tags = ['Prophet', 'Anaconda', 'Docker', 'Python']
type = 'blog'
+++

- - -

## Forecasting Growth（预测增长）

By default, Prophet uses a linear model for its forecast. When forecasting growth, there is usually some maximum achievable point: total market size, total population size, etc. This is called the carrying capacity, and the forecast should saturate at this point.

Prophet allows you to make forecasts using a [**logistic growth**](https://en.wikipedia.org/wiki/Logistic_function) trend model, with a specified carrying capacity. We illustrate this with the log number of page visits to the [**R (programming language)**](https://en.wikipedia.org/wiki/R_%28programming_language%29) page on Wikipedia:

默认情况下，Prophet 使用线性模型进行预测。预测增长时，通常存在某个最大可实现点：总市场规模、总人口规模等。这称为承载能力，预测应在此点达到饱和。

Prophet 允许您使用具有指定承载能力的[**逻辑增长**](https://en.wikipedia.org/wiki/Logistic_function)趋势模型进行预测。我们通过对维基百科上[**R（编程语言）**](https://en.wikipedia.org/wiki/R_%2528programming_language%2529)页面的访问对数来说明这一点：

```r
#R
df <- read.csv('https://raw.githubusercontent.com/facebook/prophet/main/examples/example_wp_log_peyton_manning.csv')
```

```python
# Python
df = pd.read_csv('https://raw.githubusercontent.com/facebook/prophet/main/examples/example_wp_log_peyton_manning.csv')
```

We must specify the carrying capacity in a column `cap`. Here we will assume a particular value, but this would usually be set using data or expertise about the market size.

我们必须在列中指定承载能力`cap`。在这里我们将假设一个特定的值，但这通常会使用有关市场规模的数据或专业知识来设置。

```r
# R
df$cap <- 8.5
```

```python
# Python
df['cap'] = 8.5
```

The important things to note are that `cap` must be specified for every row in the dataframe, and that it does not have to be constant. If the market size is growing, then `cap` can be an increasing sequence.

We then fit the model as before, except pass in an additional argument to specify logistic growth:

需要注意的重要一点是，`cap`必须为数据框中的每一行指定，并且它不必是常量。如果市场规模不断增长，那么`cap`可以是一个递增序列。

然后，我们像以前一样拟合模型，但要传递一个额外的参数来指定逻辑增长：

```r
# R
m <- prophet(df, growth = 'logistic')
```

```python
m = Prophet(growth='logistic')
m.fit(df)
```

We make a dataframe for future predictions as before, except we must also specify the capacity in the future. Here we keep capacity constant at the same value as in the history, and forecast 5 years into the future:

我们像以前一样为未来预测制作一个数据框，只是我们还必须指定未来的容量。在这里，我们将容量保持在与历史相同的值，并预测未来 5 年的情况：

```r
# R
future <- make_future_dataframe(m, periods = 1826)
future$cap <- 8.5
fcst <- predict(m, future)
plot(m, fcst)
```

```python
# Python
future = m.make_future_dataframe(periods=1826)
future['cap'] = 8.5
fcst = m.predict(future)
fig = m.plot(fcst)
```

![fig = m.plot(fcst)](../saturating_forecasts_13_0.png)

The logistic function has an implicit minimum of 0, and will saturate at 0 the same way that it saturates at the capacity. It is possible to also specify a different saturating minimum.

逻辑函数具有隐式最小值 0，并且将在 0 处饱和，其饱和方式与在容量处饱和的方式相同。也可以指定不同的饱和最小值。

- - -

## Saturating Minimum（饱和最小值）

The logistic growth model can also handle a saturating minimum, which is specified with a column `floor` in the same way as the `cap` column specifies the maximum:

逻辑增长模型还可以处理饱和最小值，该最小值用列指定，其指定`floor`方式与`cap`列指定最大值的方式相同：

```r
# R
df$y <- 10 - df$y
df$cap <- 6
df$floor <- 1.5
future$cap <- 6
future$floor <- 1.5
m <- prophet(df, growth = 'logistic')
fcst <- predict(m, future)
plot(m, fcst)
```

```python
# Python
df['y'] = 10 - df['y']
df['cap'] = 6
df['floor'] = 1.5
future['cap'] = 6
future['floor'] = 1.5
m = Prophet(growth='logistic')
m.fit(df)
fcst = m.predict(future)
fig = m.plot(fcst)
```

![fig = m.plot(fcst)](../saturating_forecasts_16_0.png)

To use a logistic growth trend with a saturating minimum, a maximum capacity must also be specified.

要使用具有饱和最小值的逻辑增长趋势，还必须指定最大容量。