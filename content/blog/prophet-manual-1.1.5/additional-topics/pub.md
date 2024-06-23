+++
title = 'Prophet 附加内容'
subtitle = 'Prophet Additional Topics'
brief = ''
date = 2024-06-10T00:00:11
categories = ['Prophet', 'Anaconda']
series = ['Prophet']
tags = ['Prophet', 'Anaconda', 'Docker', 'Python']
type = 'blog'
+++

- - -

## Saving models（保存模型）

It is possible to save fitted Prophet models so that they can be loaded and used later.

In R, this is done with `saveRDS` and `readRDS`:

可以保存适合的 Prophet 模型，以便以后加载和使用。

在 R 中，这是通过`saveRDS`和完成的`readRDS`：

```r
# R
saveRDS(m, file="model.RDS")  # Save model
m <- readRDS(file="model.RDS")  # Load model
```

In Python, models should not be saved with pickle; the Stan backend attached to the model object will not pickle well, and will produce issues under certain versions of Python. Instead, you should use the built-in serialization functions to serialize the model to json:

在 Python 中，不应使用 pickle 保存模型；附加到模型对象的 Stan 后端无法很好地进行 pickle，并且在某些版本的 Python 下会产生问题。相反，您应该使用内置的序列化函数将模型序列化为 json：

```python
# Python
from prophet.serialize import model_to_json, model_from_json

with open('serialized_model.json', 'w') as fout:
    fout.write(model_to_json(m))  # Save model

with open('serialized_model.json', 'r') as fin:
    m = model_from_json(fin.read())  # Load model
```

The json file will be portable across systems, and deserialization is backwards compatible with older versions of prophet.

json 文件将可跨系统移植，并且反序列化与旧版本的 prophecy 向后兼容。

- - -

## Flat trend（趋势平缓）

For time series that exhibit strong seasonality patterns rather than trend changes, or when we want to rely on the pattern of exogenous regressors (e.g. for causal inference with time series), it may be useful to force the trend growth rate to be flat. This can be achieved simply by passing `growth='flat'` when creating the model:

对于表现出强烈季节性模式而非趋势变化的时间序列，或者当我们想要依赖外生回归变量的模式（例如，使用时间序列进行因果推断）时，强制趋势增长率持平可能会很有用。这可以通过在`growth='flat'`创建模型时传递以下内容来实现：

```r
# R
m <- prophet(df, growth='flat')
```

```python
# Python
m = Prophet(growth='flat')
```

Below is a comparison of counterfactual forecasting with exogenous regressors using linear versus flat growth.

以下是使用线性增长与平坦增长的外生回归量的反事实预测的比较。

```python
# Python
import pandas as pd
from prophet import Prophet
import matplotlib.pyplot as plt

import warnings
warnings.simplefilter(action='ignore', category=FutureWarning)

regressor = "location_4"
target = "location_41"
cutoff = pd.to_datetime("2023-04-17 00:00:00")

df = (
    pd.read_csv(
        "https://raw.githubusercontent.com/facebook/prophet/main/examples/example_pedestrians_multivariate.csv", 
        parse_dates=["ds"]
    )
    .rename(columns={target: "y"})
)
train = df.loc[df["ds"] < cutoff]
test = df.loc[df["ds"] >= cutoff]
```

```python
# Python
def fit_model(growth):
    m = Prophet(growth=growth, seasonality_mode="multiplicative", daily_seasonality=15)
    m.add_regressor("location_4", mode="multiplicative")
    m.fit(train)
    preds = pd.merge(
        test,
        m.predict(test),
        on="ds",
        how="inner"
    )
    mape = ((preds["yhat"] - preds["y"]).abs() / preds_linear["y"]).mean()
    return m, preds, mape
```

```python
# Python
m_linear, preds_linear, mape_linear = fit_model("linear")
```

```python
01:19:58 - cmdstanpy - INFO - Chain [1] start processing
01:19:58 - cmdstanpy - INFO - Chain [1] done processing
```

```python
# Python
m_flat, preds_flat, mape_flat = fit_model("flat")
```

```python
01:19:58 - cmdstanpy - INFO - Chain [1] start processing
01:19:58 - cmdstanpy - INFO - Chain [1] done processing
```

```python
# Python
m_linear.plot_components(preds_linear);
```

![additional_topics_16_0.png](../additional_topics_16_0.png)

```python
# Python
m_flat.plot_components(preds_flat);
```

![additional_topics_17_0.png](../additional_topics_17_0.png)

```python
# Python
fig, ax = plt.subplots(figsize=(11, 5))
ax.scatter(preds_linear["ds"], preds_linear["y"], color="black", marker=".")
ax.plot(preds_linear["ds"], preds_linear["yhat"], label=f"linear, mape={mape_linear:.1%}")
ax.plot(preds_flat["ds"], preds_flat["yhat"], label=f"flat, mape={mape_flat:.1%}")
plt.xticks(rotation=60)
ax.legend();
```

![additional_topics_18_0.png](../additional_topics_18_0.png)

In this example, the target sensor values can be mostly explained by the exogenous regressor (a nearby sensor). The model with linear growth assumes an increasing trend and this leads to larger and larger over-predictions in the test period, while the flat growth model mostly follows movements in the exogenous regressor and this results in a sizeable MAPE improvement.

Note that forecasting with exogenous regressors is only effective when we can be confident in the future values of the regressor. The example above is relevant to causal inference using time series, where we want to understand what `y` would have looked like for a past time period, and hence the exogenous regressor values are known.

In other cases – where we don’t have exogenous regressors or have to predict their future values – if flat growth is used on a time series that doesn’t have a constant trend, any trend will be fit with the noise term and so there will be high predictive uncertainty in the forecast.

在此示例中，目标传感器值主要由外生回归量（附近的传感器）解释。线性增长模型假设趋势呈上升趋势，这会导致测试期间的过度预测越来越大，而平坦增长模型主要遵循外生回归量的变动，这会导致 MAPE 显著改善。

请注意，使用外生回归量进行预测只有当我们对回归量的未来值有信心时才有效。上述示例与使用时间序列进行因果推断有关，我们想要了解`y`过去一段时间的情况，因此外生回归量值是已知的。

在其他情况下 - 我们没有外生回归量或必须预测它们的未来值 - 如果在没有恒定趋势的时间序列上使用平坦增长，则任何趋势都将与噪声项相适应，因此预测中会存在很高的预测不确定性。

- - -

## Custom trends（定制化趋势）

To use a trend besides these three built-in trend functions (piecewise linear, piecewise logistic growth, and flat), you can download the source code from github, modify the trend function as desired in a local branch, and then install that local version. 
- [**This PR**](https://github.com/facebook/prophet/pull/1466/files) provides a good illustration of what must be done to implement a custom trend, as does [**This PR**](https://github.com/facebook/prophet/pull/1794) that implements a step function trend and [**This PR**](https://github.com/facebook/prophet/pull/1778) for a new trend in R.

要使用除这三个内置趋势函数（分段线性、分段逻辑增长和平坦）之外的趋势，您可以从 github 下载源代码，根据需要在本地分支中修改趋势函数，然后安装该本地版本。
- [**这个PR**](https://github.com/facebook/prophet/pull/1466/files)很好地说明了实现自定义趋势必须做什么；
- [**这个PR**](https://github.com/facebook/prophet/pull/1794)实现阶跃函数趋势；
- [**这个R中的PR**](https://github.com/facebook/prophet/pull/1778)新的趋势函数也很好地说明了这一点。

- - -

## Updating fitted models（更新已拟合的模型）

A common setting for forecasting is fitting models that need to be updated as additional data come in. Prophet models can only be fit once, and a new model must be re-fit when new data become available. In most settings, model fitting is fast enough that there isn’t any issue with re-fitting from scratch. However, it is possible to speed things up a little by warm-starting the fit from the model parameters of the earlier model. This code example shows how this can be done in Python:

预测的一个常见场景是拟合需要随着更多数据进入而更新的模型。Prophet 模型只能拟合一次，当有新数据可用时，必须重新拟合新模型。在大多数情况下，模型拟合速度足够快，因此从头开始重新拟合不会出现任何问题。但是，可以通过从早期模型的模型参数热启动拟合来加快速度。此代码示例展示了如何在 Python 中完成此操作：

```python
# Python
def warm_start_params(m):
    """
    Retrieve parameters from a trained model in the format used to initialize a new Stan model.
    Note that the new Stan model must have these same settings:
        n_changepoints, seasonality features, mcmc sampling
    for the retrieved parameters to be valid for the new model.

    Parameters
    ----------
    m: A trained model of the Prophet class.

    Returns
    -------
    A Dictionary containing retrieved parameters of m.
    """
    res = {}
    for pname in ['k', 'm', 'sigma_obs']:
        if m.mcmc_samples == 0:
            res[pname] = m.params[pname][0][0]
        else:
            res[pname] = np.mean(m.params[pname])
    for pname in ['delta', 'beta']:
        if m.mcmc_samples == 0:
            res[pname] = m.params[pname][0]
        else:
            res[pname] = np.mean(m.params[pname], axis=0)
    return res

df = pd.read_csv('https://raw.githubusercontent.com/facebook/prophet/main/examples/example_wp_log_peyton_manning.csv')
df1 = df.loc[df['ds'] < '2016-01-19', :]  # All data except the last day
m1 = Prophet().fit(df1) # A model fit to all data except the last day

%timeit m2 = Prophet().fit(df)  # Adding the last day, fitting from scratch
%timeit m2 = Prophet().fit(df, init=warm_start_params(m1))  # Adding the last day, warm-starting from m1
```

```python
1.33 s ± 55.9 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
185 ms ± 4.46 ms per loop (mean ± std. dev. of 7 runs, 10 loops each)
```

As can be seen, the parameters from the previous model are passed in to the fitting for the next with the kwarg `init`. In this case, model fitting was about 5x faster when using warm starting. The speedup will generally depend on how much the optimal model parameters have changed with the addition of the new data.

There are few caveats that should be kept in mind when considering warm-starting. First, warm-starting may work well for small updates to the data (like the addition of one day in the example above) but can be worse than fitting from scratch if there are large changes to the data (i.e., a lot of days have been added). This is because when a large amount of history is added, the location of the changepoints will be very different between the two models, and so the parameters from the previous model may actually produce a bad trend initialization. Second, as a detail, the number of changepoints need to be consistent from one model to the next or else an error will be raised because the changepoint prior parameter `delta` will be the wrong size.

可以看出，前一个模型的参数通过 kwarg 传递到下一个模型的拟合中`init`。在这种情况下，使用热启动时模型拟合速度大约快 5 倍。加速通常取决于最佳模型参数随着新数据的添加发生了多大的变化。

考虑热启动时，应牢记一些注意事项。首先，热启动可能适用于数据的小更新（如上例中增加一天），但如果数据发生较大变化（即增加了很多天），则可能比从头开始拟合更糟糕。这是因为当添加大量历史记录时，两个模型之间的变化点位置会有很大差异，因此前一个模型的参数实际上可能会产生错误的趋势初始化。其次，作为一个细节，从一个模型到下一个模型，变化点的数量需要保持一致，否则会引发错误，因为变化点先验参数的`delta`大小不正确。

- - -

## Minmax scaling (new in 1.1.5)（最小最大缩放比例 - 1.1.5 版新增）

Before model fitting, Prophet scales `y` by dividing by the maximum value in the history. For datasets with very large `y` values, the scaled `y` values may be compressed to a very small range (i.e. `[0.99999... - 1.0]`), which causes a bad fit. This can be fixed by setting `scaling='minmax'` in the Prophet constructor.

在模型拟合之前，Prophet`y`通过除以历史记录中的最大值来进行缩放。对于`y`值非常大的数据集，缩放后的`y`值可能会被压缩到非常小的范围（即），从而导致拟合不佳。这可以通过在 Prophet 构造函数中`[0.99999... - 1.0]`设置来修复。`scaling='minmax'`

```python
# Python
large_y = pd.read_csv(
    "https://raw.githubusercontent.com/facebook/prophet/main/python/prophet/tests/data3.csv", 
    parse_dates=["ds"]
)
```

```python
# Python
m1 = Prophet(scaling="absmax")
m1 = m1.fit(large_y)
```

```python
08:11:23 - cmdstanpy - INFO - Chain [1] start processing
08:11:23 - cmdstanpy - INFO - Chain [1] done processing
```

```python
# Python
m2 = Prophet(scaling="minmax")
m2 = m2.fit(large_y)
```

```python
08:11:29 - cmdstanpy - INFO - Chain [1] start processing
08:11:29 - cmdstanpy - INFO - Chain [1] done processing
```

```python
# Python
m1.plot(m1.predict(large_y));
```

![additional_topics_28_0.png](../additional_topics_28_0.png)

```python
# Python
m2.plot(m2.predict(large_y));
```

![additional_topics_29_0.png](../additional_topics_29_0.png)

- - -

## Inspecting transformed data (new in 1.1.5)（检查转换后的数据 - 1.1.5 版新增）

For debugging, it’s useful to understand how the raw data has been transformed before being passed to the stan fit routine. We can call the `.preprocess()` method to see all the inputs to stan, and `.calculate_init_params()` to see how the model parameters will be initialized.

对于调试来说，了解原始数据在传递给 stan fit 例程之前是如何转换的很有用。我们可以调用该`.preprocess()`方法来查看 stan 的所有输入，并`.calculate_init_params()`查看模型参数将如何初始化。

```python
# Python
df = pd.read_csv('https://raw.githubusercontent.com/facebook/prophet/main/examples/example_wp_log_peyton_manning.csv')
m = Prophet()
transformed = m.preprocess(df)
```

```python
# Python
transformed.y.head(n=10)
```

```python
0    0.746552
1    0.663171
2    0.637023
3    0.628367
4    0.614441
5    0.605884
6    0.654956
7    0.687273
8    0.652501
9    0.628148
Name: y_scaled, dtype: float64
```

```python
# Python
transformed.X.head(n=10)
```

|  | YEARLY_DELIM_1 | YEARLY_DELIM_2 | YEARLY_DELIM_3 | YEARLY_DELIM_4 | YEARLY_DELIM_5 | YEARLY_DELIM_6 | YEARLY_DELIM_7 | YEARLY_DELIM_8 | YEARLY_DELIM_9 | YEARLY_DELIM_10 | ... | YEARLY_DELIM_17 | YEARLY_DELIM_18 | YEARLY_DELIM_19 | YEARLY_DELIM_20 | WEEKLY_DELIM_1 | WEEKLY_DELIM_2 | WEEKLY_DELIM_3 | WEEKLY_DELIM_4 | WEEKLY_DELIM_5 | WEEKLY_DELIM_6 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | -0.377462 | 0.926025 | -0.699079 | 0.715044 | -0.917267 | 0.398272 | -0.999745 | 0.022576 | -0.934311 | -0.356460 | ... | 0.335276 | -0.942120 | 0.666089 | -0.745872 | -4.338837e-01 | -0.900969 | 7.818315e-01 | 0.623490 | -9.749279e-01 | -0.222521 |
| 1 | -0.361478 | 0.932381 | -0.674069 | 0.738668 | -0.895501 | 0.445059 | -0.995827 | 0.091261 | -0.961479 | -0.274879 | ... | 0.185987 | -0.982552 | 0.528581 | -0.848883 | -9.749279e-01 | -0.222521 | 4.338837e-01 | -0.900969 | 7.818315e-01 | 0.623490 |
| 2 | -0.345386 | 0.938461 | -0.648262 | 0.761418 | -0.871351 | 0.490660 | -0.987196 | 0.159513 | -0.981538 | -0.191266 | ... | 0.032249 | -0.999480 | 0.375470 | -0.926834 | -7.818315e-01 | 0.623490 | -9.749279e-01 | -0.222521 | -4.338837e-01 | -0.900969 |
| 3 | -0.329192 | 0.944263 | -0.621687 | 0.783266 | -0.844881 | 0.534955 | -0.973892 | 0.227011 | -0.994341 | -0.106239 | ... | -0.122261 | -0.992498 | 0.211276 | -0.977426 | 5.505235e-14 | 1.000000 | 1.101047e-13 | 1.000000 | 1.984146e-12 | 1.000000 |
| 4 | -0.312900 | 0.949786 | -0.594376 | 0.804187 | -0.816160 | 0.577825 | -0.955979 | 0.293434 | -0.999791 | -0.020426 | ... | -0.273845 | -0.961774 | 0.040844 | -0.999166 | 7.818315e-01 | 0.623490 | 9.749279e-01 | -0.222521 | 4.338837e-01 | -0.900969 |
| 5 | -0.296516 | 0.955028 | -0.566362 | 0.824157 | -0.785267 | 0.619157 | -0.933542 | 0.358468 | -0.997850 | 0.065537 | ... | -0.418879 | -0.908042 | -0.130793 | -0.991410 | 9.749279e-01 | -0.222521 | -4.338837e-01 | -0.900969 | -7.818315e-01 | 0.623490 |
| 6 | -0.280044 | 0.959987 | -0.537677 | 0.843151 | -0.752283 | 0.658840 | -0.906686 | 0.421806 | -0.988531 | 0.151016 | ... | -0.553893 | -0.832588 | -0.298569 | -0.954388 | 4.338837e-01 | -0.900969 | -7.818315e-01 | 0.623490 | 9.749279e-01 | -0.222521 |
| 7 | -0.263489 | 0.964662 | -0.508356 | 0.861147 | -0.717295 | 0.696769 | -0.875539 | 0.483147 | -0.971904 | 0.235379 | ... | -0.675656 | -0.737217 | -0.457531 | -0.889193 | -4.338837e-01 | -0.900969 | 7.818315e-01 | 0.623490 | -9.749279e-01 | -0.222521 |
| 8 | -0.246857 | 0.969052 | -0.478434 | 0.878124 | -0.680398 | 0.732843 | -0.840248 | 0.542202 | -0.948090 | 0.318001 | ... | -0.781257 | -0.624210 | -0.602988 | -0.797750 | -9.749279e-01 | -0.222521 | 4.338837e-01 | -0.900969 | 7.818315e-01 | 0.623490 |
| 9 | -0.230151 | 0.973155 | -0.447945 | 0.894061 | -0.641689 | 0.766965 | -0.800980 | 0.598691 | -0.917267 | 0.398272 | ... | -0.868168 | -0.496271 | -0.730644 | -0.682758 | -7.818315e-01 | 0.623490 | -9.749279e-01 | -0.222521 | -4.338837e-01 | -0.900969 |

10 rows × 26 columns

10 行 × 26 列

```python
# Python
m.calculate_initial_params(num_total_regressors=transformed.K)
```

```python
ModelParams(k=-0.05444079622224118, m=0.7465517318876905, delta=array([0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0.,
       0., 0., 0., 0., 0., 0., 0., 0.]), beta=array([0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0.,
       0., 0., 0., 0., 0., 0., 0., 0., 0.]), sigma_obs=1.0)
```

- - -

## External references（外部资源）

---

As we discuss in our [**2023 blog post on the state of Prophet**](https://medium.com/@cuongduong_35162/facebook-prophet-in-2023-and-beyond-c5086151c138), we have no plans to further develop the underlying Prophet model. If you’re looking for state-of-the-art forecasting accuracy, we recommend the following libraries:

- [**statsforecast**](https://github.com/Nixtla/statsforecast), and other packages from the Nixtla group such as [**hierarchicalforecast**](https://github.com/Nixtla/hierarchicalforecast) and [**neuralforecast**](https://github.com/Nixtla/neuralforecast).
- [**NeuralProphet**](https://neuralprophet.com/), a Prophet-style model implemented in PyTorch, to be more adaptable and extensible.

These github repositories provide examples of building on top of Prophet in ways that may be of broad interest:

- [**Forecastr**](https://github.com/garethcull/forecastr): A web app that provides a UI for Prophet.

---

正如我们在[2023 年关于 Prophet 现状的博客文章](https://translate.google.com/website?sl=en&tl=zh-CN&hl=zh-CN&client=webapp&u=https://medium.com/@cuongduong_35162/facebook-prophet-in-2023-and-beyond-c5086151c138)中所讨论的那样，我们没有进一步开发底层 Prophet 模型的计划。如果您正在寻找最先进的预测准确性，我们推荐以下库：

- [**statsforecast**](https://github.com/Nixtla/statsforecast)，以及来自 Nixtla 组的其他软件包，例如[**hierarchicalforecast**](https://github.com/Nixtla/hierarchicalforecast)和[**neuralforecast**](https://github.com/Nixtla/neuralforecast)。
- [**NeuralProphet**](https://neuralprophet.com/)，用 PyTorch 实现的 Prophet 风格模型，具有更强的适应性和可扩展性。

这些 GitHub 存储库提供了在 Prophet 之上构建的示例，这些示例可能引起了广泛的兴趣：

- [**Forecastr**](https://github.com/garethcull/forecastr) 为Prophet提供的Web UI应用程序。