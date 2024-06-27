+++
title = 'Python Pandas包'
subtitle = 'Python Pandas'
brief = ''
date = 2024-06-27T00:00:00
categories = ['Python', 'Pandas']
series = ['Python Language']
tags = ['Python', 'Math', 'Pandas']
type = 'blog'
+++

---

## 1. Install

---

```bash
# Pip
pip install pandas
# Conda
conda install -c conda-forge pandas
conda install conda-forge::pandas
```

---

## 2. Example

---

### 2.1. Convert String to DateTime
```python
df_db['ds'] = pd.to_datetime(df_db['ds'], format='%Y%m%d')
```

### 2.2. Refactoring DateTime series

```python
import pandas as pd
# By Merge with new datetime series
# df is incomplete series data, which contains a column 'ds' as datetime series. 
idx = pd.date_range(start = '2023-02-01', end = '2023-05-31', freq = '1D')
df_ds = pd.DataFrame({ 'ds': idx })
df_merged = pd.merge(df, df_ds, on = 'ds', how = 'outer').sort_values('ds')
# Fill 0 for Nan value of y2
df_merged['y2'] = df_merged['y'].fillna(value = 0)

# By Reindex origin ds column
idx = pd.date_range(start = '2023-02-01', end = '2023-05-31', freq = '1D')
df = df.set_index('ds') # set the DataFrame index by 'ds' column
df = df.reindex(idx, fill_value = 0) # reindex the DataFrame by idx(datetime series generated), and fill 0 for Nan value
df = df.reset_index() # reset the DataFrame index by number sequence
df.columns = ['ds', 'y'] # reset the DataFrame name

# For the perf reason, inplace should be use firstly.
df.set_index('ds', inplace = True)
df = df.reindex(idx, fill_value = 0)
df.reset_index(inplace = True)
df.columns = ['ds', 'y']
df.head()
```
