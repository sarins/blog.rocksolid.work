+++
title = 'Prophet 快速开始'
subtitle = 'Prophet quick start'
brief = ''
date = 2024-06-10
categories = ['Prophet', 'Anaconda']
series = ['Prophet']
tags = ['Prophet', 'Anaconda', 'Docker', 'Python']
type = 'blog'
+++

- - -

## Python API

Prophet follows the `sklearn` model API. We create an instance of the `Prophet` class and then call its `fit` and `predict` methods.

The input to Prophet is always a dataframe with two columns: `ds` and `y`. The `ds` (datestamp) column should be of a format expected by Pandas, ideally YYYY-MM-DD for a date or YYYY-MM-DD HH:MM:SS for a timestamp. The `y` column must be numeric, and represents the measurement we wish to forecast.

As an example, let’s look at a time series of the log daily page views for the Wikipedia page for [**Peyton Manning**](https://en.wikipedia.org/wiki/Peyton_Manning). We scraped this data using the [**Wikipediatrend**](https://cran.r-project.org/package=wikipediatrend) package in R. Peyton Manning provides a nice example because it illustrates some of Prophet’s features, like multiple seasonality, changing growth rates, and the ability to model special days (such as Manning’s playoff and superbowl appearances). The CSV is available [**here**](https://github.com/facebook/prophet/blob/main/examples/example_wp_log_peyton_manning.csv).

First we’ll import the data:

Prophet 遵循`sklearn`模型 API。我们创建类的实例`Prophet`，然后调用它的`fit`方法`predict`。

Prophet 的输入始终是一个包含两列的数据框：`ds`和`y`。`ds`（日期戳）列应为 Pandas 所需的格式，理想情况下，日期为 YYYY-MM-DD，时间戳为 YYYY-MM-DD HH:MM:SS。该`y`列必须是数字，代表我们希望预测的测量值。

举个例子，让我们看一下[**Peyton Manning**](https://en.wikipedia.org/wiki/Peyton_Manning)的 Wikipedia 页面的每日页面浏览量的时间序列。
我们使用R 中的[**Wikipediatrend**](https://cran.r-project.org/package=wikipediatrend)包抓取了这些数据。Peyton Manning 提供了一个很好的例子，因为它说明了 Prophet 的一些功能，例如多种季节性、不断变化的增长率以及模拟特殊日子的能力（例如 Manning 的季后赛和超级碗出场）。
CSV 可[**在此处**](https://github.com/facebook/prophet/blob/main/examples/example_wp_log_peyton_manning.csv)获取。

首先我们导入数据：

```python
# Python
import pandas as pd
from prophet import Prophet
```

```python
# Python
df = pd.read_csv('https://raw.githubusercontent.com/facebook/prophet/main/examples/example_wp_log_peyton_manning.csv')
df.head()
```

|  | DS | Y |
| --- | --- | --- |
| 0 | 2007-12-10 | 9.590761 |
| 1 | 2007-12-11 | 8.519590 |
| 2 | 2007-12-12 | 8.183677 |
| 3 | 2007-12-13 | 8.072467 |
| 4 | 2007-12-14 | 7.893572 |

We fit the model by instantiating a new `Prophet` object. Any settings to the forecasting procedure are passed into the constructor. Then you call its `fit` method and pass in the historical dataframe. Fitting should take 1-5 seconds.

我们通过实例化一个新对象来拟合模型`Prophet`。预测过程的任何设置都会传递到构造函数中。然后调用其`fit`方法并传入历史数据框。拟合应该需要 1-5 秒。

```python
# Python
m = Prophet()
m.fit(df)
```

Predictions are then made on a dataframe with a column `ds` containing the dates for which a prediction is to be made. You can get a suitable dataframe that extends into the future a specified number of days using the helper method `Prophet.make_future_dataframe`. By default it will also include the dates from the history, so we will see the model fit as well.

然后对数据框进行预测，其中有一列`ds`包含要进行预测的日期。您可以使用辅助方法获取延伸到未来指定天数的合适数据框`Prophet.make_future_dataframe`。默认情况下，它还将包含历史记录中的日期，因此我们也将看到模型拟合。

```python
# Python
future = m.make_future_dataframe(periods=365)
future.tail()
```

|  | DS |
| --- | --- |
| 3265 | 2017-01-15 |
| 3266 | 2017-01-16 |
| 3267 | 2017-01-17 |
| 3268 | 2017-01-18 |
| 3269 | 2017-01-19 |

The `predict` method will assign each row in `future` a predicted value which it names `yhat`. If you pass in historical dates, it will provide an in-sample fit. The `forecast` object here is a new dataframe that includes a column `yhat` with the forecast, as well as columns for components and uncertainty intervals.

该`predict`方法将为每一行分配`future`一个预测值，并将其命名为`yhat`。如果您传入历史日期，它将提供样本内拟合。`forecast`此处的对象是一个新的数据框，其中包含`yhat`预测列以及组件和不确定区间的列。

```python
# Python
forecast = m.predict(future)
forecast[['ds', 'yhat', 'yhat_lower', 'yhat_upper']].tail()
```

|  | DS | YHAT | YHAT_LOWER | YHAT_UPPER |
| --- | --- | --- | --- | --- |
| 3265 | 2017-01-15 | 8.212625 | 7.456310 | 8.959726 |
| 3266 | 2017-01-16 | 8.537635 | 7.842986 | 9.290934 |
| 3267 | 2017-01-17 | 8.325071 | 7.600879 | 9.072006 |
| 3268 | 2017-01-18 | 8.157723 | 7.512052 | 8.924022 |
| 3269 | 2017-01-19 | 8.169677 | 7.412473 | 8.946977 |

You can plot the forecast by calling the `Prophet.plot` method and passing in your forecast dataframe.

`Prophet.plot`您可以通过调用该方法并传入预测数据框来绘制预测图。

```python
# Python
fig1 = m.plot(forecast)
```

![fig1](../output_12_0.png)

If you want to see the forecast components, you can use the `Prophet.plot_components` method. By default you’ll see the trend, yearly seasonality, and weekly seasonality of the time series. If you include holidays, you’ll see those here, too.

如果您想查看预测成分，可以使用该`Prophet.plot_components`方法。默认情况下，您将看到时间序列的趋势、年度季节性和每周季节性。如果您包括假期，您也会在这里看到这些。

```python
# Python
fig2 = m.plot_components(forecast)
```

![fig2](../output_14_0.png)

An interactive figure of the forecast and components can be created with plotly. You will need to install plotly 4.0 or above separately, as it will not by default be installed with prophet. You will also need to install the `notebook` and `ipywidgets` packages.

可以使用 plotly 创建预测和组件的交互式图表。您需要单独安装 plotly 4.0 或更高版本，因为它不会默认与 prophecy 一起安装。您还需要安装`notebook`和`ipywidgets`包。

```python
# Python
from prophet.plot import plot_plotly, plot_components_plotly

plot_plotly(m, forecast)
```

```python
# Python
plot_components_plotly(m, forecast)
```

More details about the options available for each method are available in the docstrings, for example, via `help(Prophet)` or `help(Prophet.fit)`.

有关每种方法可用选项的更多详细信息请参阅文档字符串，例如通过`help(Prophet)`或`help(Prophet.fit)`。

- - -

## R API

In R, we use the normal model fitting API. We provide a `prophet` function that performs fitting and returns a model object. You can then call `predict` and `plot` on this model object.

在 R 中，我们使用常规模型拟合 API。我们提供了一个`prophet`执行拟合并返回模型对象的函数。然后您可以在此模型对象上调用`predict`和。`plot`

```r
# R
library(prophet)
```

```r
R[write to console]: Loading required package: Rcpp

R[write to console]: Loading required package: rlang
```

First we read in the data and create the outcome variable. As in the Python API, this is a dataframe with columns `ds` and `y`, containing the date and numeric value respectively. The ds column should be YYYY-MM-DD for a date, or YYYY-MM-DD HH:MM:SS for a timestamp. As above, we use here the log number of views to Peyton Manning’s Wikipedia page, available [**here**](https://github.com/facebook/prophet/blob/main/examples/example_wp_log_peyton_manning.csv).

首先，我们读入数据并创建结果变量。与 Python API 一样，这是一个包含列和的数据框`ds`，`y`分别包含日期和数值。ds 列应为 YYYY-MM-DD（表示日期）或 YYYY-MM-DD HH:MM:SS（表示时间戳）。
与上文一样，我们在此使用 Peyton Manning 维基百科页面的浏览量对数，可[**在此处**](https://github.com/facebook/prophet/blob/main/examples/example_wp_log_peyton_manning.csv)查看。

```r
# R
df <- read.csv('https://raw.githubusercontent.com/facebook/prophet/main/examples/example_wp_log_peyton_manning.csv')
```

We call the `prophet` function to fit the model. The first argument is the historical dataframe. Additional arguments control how Prophet fits the data and are described in later pages of this documentation.

我们调用该`prophet`函数来拟合模型。第一个参数是历史数据框。其他参数控制 Prophet 如何拟合数据，并将在本文档的后面几页中描述。

```r
# R
m <- prophet(df)
```

Predictions are made on a dataframe with a column `ds` containing the dates for which predictions are to be made. The `make_future_dataframe` function takes the model object and a number of periods to forecast and produces a suitable dataframe. By default it will also include the historical dates so we can evaluate in-sample fit.

预测是在数据框上进行的，其中有一列`ds`包含要进行预测的日期。该`make_future_dataframe`函数采用模型对象和多个要预测的时间段并生成合适的数据框。默认情况下，它还将包括历史日期，以便我们可以评估样本内拟合度。

```r
# R
future <- make_future_dataframe(m, periods = 365)
tail(future)
```

```r
             ds
3265 2017-01-14
3266 2017-01-15
3267 2017-01-16
3268 2017-01-17
3269 2017-01-18
3270 2017-01-19
```

As with most modeling procedures in R, we use the generic `predict` function to get our forecast. The `forecast` object is a dataframe with a column `yhat` containing the forecast. It has additional columns for uncertainty intervals and seasonal components.

与 R 中的大多数建模程序一样，我们使用通用`predict`函数来获取预测。该`forecast`对象是一个数据框，其中有一列`yhat`包含预测。它还有用于不确定性区间和季节性成分的附加列。

```r
# R
forecast <- predict(m, future)
tail(forecast[c('ds', 'yhat', 'yhat_lower', 'yhat_upper')])
```

```r
             ds     yhat yhat_lower yhat_upper
3265 2017-01-14 7.818359   7.071228   8.550957
3266 2017-01-15 8.200125   7.475725   8.869495
3267 2017-01-16 8.525104   7.747071   9.226915
3268 2017-01-17 8.312482   7.551904   9.046774
3269 2017-01-18 8.145098   7.390770   8.863692
3270 2017-01-19 8.156964   7.381716   8.866507
```

You can use the generic `plot` function to plot the forecast, by passing in the model and the forecast dataframe.

您可以使用通用`plot`函数通过传入模型和预测数据框来绘制预测。

```r
# R
plot(m, forecast)
```

![plot(m, forecast)](../output_30_0.png)

You can use the `prophet_plot_components` function to see the forecast broken down into trend, weekly seasonality, and yearly seasonality.

您可以使用该`prophet_plot_components`函数查看细分为趋势、每周季节性和每年季节性的预测。

```r
# R
prophet_plot_components(m, forecast)
```

![prophet_plot_components(m, forecast)](../output_32_0.png)

An interactive plot of the forecast using Dygraphs can be made with the command `dyplot.prophet(m, forecast)`.

More details about the options available for each method are available in the docstrings, for example, via `?prophet` or `?fit.prophet`. This documentation is also available in the [**reference manual**](https://cran.r-project.org/web/packages/prophet/prophet.pdf) on CRAN.

可以使用以下命令使用 Dygraphs 绘制预测的交互式图表`dyplot.prophet(m, forecast)`。

有关每种方法可用选项的更多详细信息可在文档字符串中找到，例如通过`?prophet`或`?fit.prophet`。此文档也可在CRAN 上的[**参考手册**](https://cran.r-project.org/web/packages/prophet/prophet.pdf)中找到。