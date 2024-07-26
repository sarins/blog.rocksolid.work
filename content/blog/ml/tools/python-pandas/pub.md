+++
title = 'Python Pandas'
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

## 2. How to

---

### 2.1. Convert String to DateTime
```python
df_db['ds'] = pd.to_datetime(df_db['ds'], format='%Y%m%d')
```

### 2.2. Re-Index DateTime series
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

### 2.3. Convert Horizontal table as Vertical table

Some time data table looks like below, the values at columns which starts with "d_", convert the records as rows view.

| id | item_id | dept_id | cat_id | store_id | state_id | d_1 | d_2 | d_3 | d_4 | ... | d_1939 | d_1940 | d_1941 |
| - | - | - | - | - | - | - | - | - | - | - | - | - | - |
| HOBBIES_1_001_CA_1_evaluation | HOBBIES_1_001 | HOBBIES_1 | HOBBIES | CA_1 | CA | 0 | 0 | 0 | 0 | ... | 2 | 4 | 0 |
| HOBBIES_1_002_CA_1_evaluation | HOBBIES_1_002 | HOBBIES_1 | HOBBIES | CA_1 | CA | 0 | 0 | 0 | 0 | ... | 0 | 1 | 2 |
| HOBBIES_1_003_CA_1_evaluation | HOBBIES_1_003 | HOBBIES_1 | HOBBIES | CA_1 | CA | 0 | 0 | 0 | 0 | ... | 1 | 0 | 2 |
| HOBBIES_1_004_CA_1_evaluation | HOBBIES_1_004 | HOBBIES_1 | HOBBIES | CA_1 | CA | 0 | 0 | 0 | 0 | ... | 1 | 1 | 0 |
| HOBBIES_1_005_CA_1_evaluation | HOBBIES_1_005 | HOBBIES_1 | HOBBIES | CA_1 | CA | 0 | 0 | 0 | 0 | ... | 0 | 0 | 0 |


```python
# define key columns, which will be keep for each rows.
index_columns = ['id', 'item_id', 'dept_id', 'cat_id', 'store_id', 'state_id']
# use "melt" method, columns starts with "d_", will be convert as "d" column, and related values will be as "sales" column.
grid_df = pd.melt(train_df,
                  id_vars=index_columns,
                  var_name='d',
                  value_name='sales')
```

### 2.4. Reduce memory by change the numberic type (Depend on Numpy)
```python
def memory_optimizer(df, verbose=True):
    """
    Reduce memory by change the numberic type

    Parameters
    ----------
    df : DataFrame(pandas)
        the DataFrame instance which will be perform memory reducing operation.
    verbose : bool, default = True
        print verbose level log into default output or not, default is true
    """

    numerics = ['int16', 'int32', 'int64', 'float16', 'float32', 'float64']
    info_int8 = np.iinfo(np.int8)
    info_int16 = np.iinfo(np.int16)
    info_int32 = np.iinfo(np.int32)
    info_float16 = np.finfo(np.float16)
    info_float32 = np.finfo(np.float32)
    megabytes = 1 << 20
    origin_memory_usage = df.memory_usage().sum() / megabytes

    for col in df.columns: 
        ct = str(df[col].dtypes)
        if ct in numerics:
            col_min = df[col].min()
            col_max = df[col].max()
            if ct.startswith('int'):
                if col_min > info_int8.min and col_max < info_int8.max:
                    df[col] = df[col].astype(np.int8)
                elif col_min > info_int16.min and col_max < info_int16.max:
                    df[col] = df[col].astype(np.int16)
                elif col_min > info_int32.min and col_max < info_int32.max:
                    df[col] = df[col].astype(np.int32)
                else:
                    df[col] = df[col].astype(np.int64)
            else:
                if col_min > info_float16.min and col_max < info_float16.max:
                    df[col] = df[col].astype(np.float16)
                elif col_min > info_float32.min and col_max < info_float32.max:
                    df[col] = df[col].astype(np.float32)
                else:
                    df[col] = df[col].astype(np.float64)
    if verbose:
        optimized_memory_usage = df.memory_usage().sum() / megabytes
        print('Memory usage decreased to {:5.2f} Mb ({:.1f}% reduction)'.format(optimized_memory_usage, 100 * (origin_memory_usage - optimized_memory_usage) / origin_memory_usage))
    return df
```
