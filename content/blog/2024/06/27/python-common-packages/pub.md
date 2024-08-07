+++
title = 'Python 常用程序包'
subtitle = 'Python Common Packages'
brief = ''
date = 2024-06-27T00:00:01
categories = ['Python', 'Math', 'Plot', 'Geo', 'Map']
series = ['Python Language']
tags = ['Python', 'Math', 'Plot', 'Geo', 'Map', 'Seaborn', 'Folium']
type = 'blog'
+++

---

## 1. Plot

---

### 1.1. Matplotlib

Matplotlib is a comprehensive library for creating static, animated, and interactive visualizations.

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
%matplotlib inline # Inline mode

fig, ax = plt.subplots()             # Create a figure containing a single Axes.
ax.plot([1, 2, 3, 4], [1, 4, 2, 3])  # Plot some data on the Axes.
# plt.show()                           # Show the figure.
```

#### 1.1.3. Pie Example
```python
# import matplotlib.pyplot as plt
from matplotlib import pyplot as plt
%matplotlib inline # Inline mode

fig, ax = plt.subplots()             # Create a figure containing a single Axes.
ax.pie([1,2,3], labels = ['A', 'B', 'C'], autopct='%1.1f%%')
```

#### 1.1.4 Chinese support
```bash
# Install font
sudo apt install fonts-wqy-zenhei

# Checkout font localtion, the font will be located at /usr/share/fonts/turetype/....
fc-list :lang=zh
# https://transfonter.org/ttc-unpack convert(unpacking) ttc to ttf.
```

```python
# Checkout matplotlib font locations
# Example /opt/conda/envs/ml-py310/lib/python3.10/site-packages/matplotlib/mpl-data/matplotlibrc
# Font location at /opt/conda/envs/ml-py310/lib/python3.10/site-packages/matplotlib/mpl-data/fonts/ttf
import matplotlib
print(matplotlib.matplotlib_fname())
```

```bash
# Copy ttf file into Font location
cp xxxx.ttf (some python site-packages)/matplotlib/mpl-data/fonts/ttf
```

```python
# Try the Chinese character correctly display or not
import matplotlib.pyplot as plt
fig, ax = plt.subplots()
ax.text(
    .5, .5, "There are 几个汉字 in between!",
    family=['DejaVu Sans', 'WenQuanYi Zen Hei'],
    ha='center'
)
```

```python
# Clean cache of matplotlib
import shutil
import matplotlib

shutil.rmtree(matplotlib.get_cachedir())
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
