+++
title = '梯度提升决策树'
subtitle = 'Gradient Boosting Decision Tree'
brief = ''
date = 2022-05-01T00:00:03
categories = ['Machine Learning', 'Math', 'Plot', 'Algorithm']
series = ['Machine Learning', 'Algorithm']
tags = ['Python', 'Machine Learning', 'Math', 'Plot', 'Algorithm', 'GBDT']
type = 'blog'
draft = true
+++

---

## 1. GBDT算法

GBDT（Gradient Boosting Decision Tree），全名叫梯度提升决策树，是一种迭代的决策树算法，又叫 MART（Multiple Additive Regression Tree），它通过构造一组弱的学习器（树），并把多颗决策树的结果累加起来作为最终的预测输出。该算法将决策树与集成思想进行了有效的结合。

![1](../1.png)

---

### 1.1. Boosting核心概念

- Boosting方法训练基分类器时采用串行的方式，各个基分类器之间有依赖。它的基本思路是将基分类器层层叠加，每一层在训练的时候，对前一层基分类器分错的样本，给予更高的权重。测试时，根据各层分类器的结果的加权得到最终结果。
- Bagging 与 Boosting 的串行训练方式不同，Bagging 方法在训练过程中，各基分类器之间无强依赖，可以进行并行训练。

![2](../2.png)

#### 1.1.1. Install
```bash
# Pip
pip install matplotlib
# Conda
conda install -c conda-forge matplotlib
conda install conda-forge::matplotlib
```

#### 1.1.2. Example
```python
# import matplotlib.pyplot as plt
from matplotlib import pyplot as plt
import numpy as np
%matplotlib inline # Inline mode

fig, ax = plt.subplots()             # Create a figure containing a single Axes.
ax.plot([1, 2, 3, 4], [1, 4, 2, 3])  # Plot some data on the Axes.
# plt.show()                           # Show the figure.
```

#### 1.1.3. Pie Example
```python
# import matplotlib.pyplot as plt
from matplotlib import pyplot as plt
import numpy as np
%matplotlib inline # Inline mode

fig, ax = plt.subplots()             # Create a figure containing a single Axes.
ax.pie([1,2,3], labels = ['A', 'B', 'C'], autopct='%1.1f%%')
```

---

### 1.2. Seaborn

Seaborn is a Python data visualization library based on matplotlib. It provides a high-level interface for drawing attractive and informative statistical graphics.

#### 1.2.1. Install
```bash
# Pip
pip install seaborn
# Conda
conda install conda-forge::seaborn
```

#### 1.2.2. Example
```python
from matplotlib import pyplot as plt
import numpy as np
import seaborn as sns
sns.set()  # Declearing using Seaborn Style
%matplotlib inline # Inline mode

fig, ax = plt.subplots()
ax.plot([1, 2, 3, 4], [1, 4, 2, 3])
```

---

## 2. Geography

---

### 2.1. Folium

Folium builds on the data wrangling strengths of the Python ecosystem and the mapping strengths of the Leaflet.js library. Manipulate your data in Python, then visualize it in a Leaflet map via Folium.

#### 2.1.1. Install
```bash
# Pip
pip install folium
# Conda
conda install conda-forge::folium
```

#### 2.1.2. Example
```python
import folium
m = folium.Map(location=(45.5236, -122.6750)) # Give a geo location
m # The map will be shown as below.
```

---

## 3. Database

---

### 3.1. Pymysql
```python
import pymysql.cursors
connection = pymysql.connect(host='host.docker.internal',
                             user='root',
                             password='6yhn*IK<',
                             database='reviewboard',
                             cursorclass=pymysql.cursors.DictCursor)

# df_db = pd.read_sql_table("data", connection)

with connection:
  with connection.cursor() as cursor:
    SQL = "SELECT dt, sum(sale_amount), sum(origin_amount) FROM sale_202402_shoe_202401_202404_sales_direct WHERE store_code = 'XG02' GROUP BY dt ORDER BY dt"
    cursor.execute(SQL)
    result = cursor.fetchall()
    print(result)
```

### 3.2. Sqlalchemy
```python
import sqlalchemy as sa
import pymysql
pymysql.install_as_MySQLdb()

SQL = "SELECT dt as ds, sum(sale_amount) as y FROM sale_202302_shoe_202301_202304_sales_direct WHERE store_code = 'S107' AND sku = 'ARMT013-4' GROUP BY dt ORDER BY dt;"
engine = sa.create_engine("mysql+mysqldb://root:6yhn*IK<@host.docker.internal/reviewboard")
df_db = pd.read_sql(sa.text(SQL), engine)
df_db.head()
```
