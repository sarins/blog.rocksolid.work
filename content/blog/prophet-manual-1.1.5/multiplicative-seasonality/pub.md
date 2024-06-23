+++
title = 'Prophet 具备乘法性质的季节性'
subtitle = 'Prophet Multiplicative Seasonality'
brief = ''
date = 2024-06-10T00:00:05
categories = ['Prophet', 'Anaconda']
series = ['Prophet']
tags = ['Prophet', 'Anaconda', 'Docker', 'Python']
type = 'blog'
+++

- - -

By default Prophet fits additive seasonalities, meaning the effect of the seasonality is added to the trend to get the forecast. This time series of the number of air passengers is an example of when additive seasonality does not work:

默认情况下，Prophet 会拟合附加季节性，这意味着季节性的影响会添加到趋势中以获得预测。这个航空乘客人数的时间序列就是附加季节性不起作用的一个例子：

```r
# R
df <- read.csv('https://raw.githubusercontent.com/facebook/prophet/main/examples/example_air_passengers.csv')
m <- prophet(df)
future <- make_future_dataframe(m, 50, freq = 'm')
forecast <- predict(m, future)
plot(m, forecast)
```

```python
# Python
df = pd.read_csv('https://raw.githubusercontent.com/facebook/prophet/main/examples/example_air_passengers.csv')
m = Prophet()
m.fit(df)
future = m.make_future_dataframe(50, freq='MS')
forecast = m.predict(future)
fig = m.plot(forecast)
```

![multiplicative_seasonality_4_0.png](../multiplicative_seasonality_4_0.png)

This time series has a clear yearly cycle, but the seasonality in the forecast is too large at the start of the time series and too small at the end. In this time series, the seasonality is not a constant additive factor as assumed by Prophet, rather it grows with the trend. This is multiplicative seasonality.

Prophet can model multiplicative seasonality by setting `seasonality_mode='multiplicative'` in the input arguments:

这个时间序列具有明显的年度周期，但预测中的季节性在时间序列开始时太大，在结束时太小。在这个时间序列中，季节性不是 Prophet 假设的恒定加法因子，而是随着趋势而增长。这是乘法季节性。

Prophet 可以通过设置`seasonality_mode='multiplicative'`输入参数来模拟乘法季节性：

```r
# R
m <- prophet(df, seasonality.mode = 'multiplicative')
forecast <- predict(m, future)
plot(m, forecast)
```

```python
# Python
m = Prophet(seasonality_mode='multiplicative')
m.fit(df)
forecast = m.predict(future)
fig = m.plot(forecast)
```

![multiplicative_seasonality_7_0.png](../multiplicative_seasonality_7_0.png)

The components figure will now show the seasonality as a percent of the trend:

成分图现在将显示季节性占趋势的百分比：

```r
# R
prophet_plot_components(m, forecast)
```

```python
# Python
fig = m.plot_components(forecast)
```

![multiplicative_seasonality_10_0.png](../multiplicative_seasonality_10_0.png)

With `seasonality_mode='multiplicative'`, holiday effects will also be modeled as multiplicative. Any added seasonalities or extra regressors will by default use whatever `seasonality_mode` is set to, but can be overriden by specifying `mode='additive'` or `mode='multiplicative'` as an argument when adding the seasonality or regressor.

For example, this block sets the built-in seasonalities to multiplicative, but includes an additive quarterly seasonality and an additive regressor:

使用`seasonality_mode='multiplicative'`，假日效应也将被建模为乘法。任何添加的季节性或额外的回归量将默认使用`seasonality_mode`设置为的任何值，但可以在添加季节性或回归量时通过指定`mode='additive'`或作为参数来覆盖。`mode='multiplicative'`

例如，此块将内置季节性设置为乘性，但包括加性季度季节性和加性回归量：

```r
# R
m <- prophet(seasonality.mode = 'multiplicative')
m <- add_seasonality(m, 'quarterly', period = 91.25, fourier.order = 8, mode = 'additive')
m <- add_regressor(m, 'regressor', mode = 'additive')
```

```python
# Python
m = Prophet(seasonality_mode='multiplicative')
m.add_seasonality('quarterly', period=91.25, fourier_order=8, mode='additive')
m.add_regressor('regressor', mode='additive')
```

Additive and multiplicative extra regressors will show up in separate panels on the components plot. Note, however, that it is pretty unlikely to have a mix of additive and multiplicative seasonalities, so this will generally only be used if there is a reason to expect that to be the case.

加法和乘法额外回归量将在分量图上的单独面板中显示。但请注意，加法和乘法季节性混合出现的可能性很小，因此通常只有在有理由预期会出现这种情况时才会使用。