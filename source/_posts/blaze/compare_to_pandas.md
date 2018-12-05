---
title: Blaze(六):Pandas与Blaze比较
date: 2018-12-05 21:47:28
tags: [Python, Blaze]
categories: Blaze
toc: true
---

**导入和构造**

```python
# coding: utf-8

import numpy as np
import pandas as pd
from blaze import data, by

df = pd.DataFrame({'name': ['Alice', 'Bob', 'Joe', 'Bob'],
                   'amount': [100, 200, 300, 400],
                   'id': [1, 2, 3, 4],
                   })

df = data(df)
print(df.peek())
```

*输出：*

```bash
    name  amount  id
0  Alice     100   1
1    Bob     200   2
2    Joe     300   3
3    Bob     400   4
```
| Computaion              | Pandas                                                       | Blaze                                                        |
| ----------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Column Arithmetic       | df.amount * 2                                                | df.amount * 2                                                |
| Multiple Columns        | df[['id', 'amount']]                                         | df[['id', 'amount']]                                         |
| Selection               | df[df.amount > 300]                                          | df[df.amount > 300]                                          |
| Group By                | df.groupby('name').amount.mean() df.groupby(['name','id']).amount.mean() | by(df.name, amount=df.amount.mean()) by(merge(df.name, df.id), amount=df.amount.mean()) |
| Join                    | pd.merge(df, df2, on='name')                                 | join(df, df2, 'name')                                        |
| Map                     | df.amount.map(lambda x: x + 1)                               | df.amount.map(lambda x: x + 1, 'int64')                      |
| Relabel Columns         | df.rename(columns={'name': 'alias', 'amount': 'dollars'})    | df.relabel(name='alias', amount='dollars')                   |
| Drop duplicates         | df.drop_duplicates() df.name.drop_duplicates()               | df.distinct() df.name.distinct()                             |
| Reductions              | df.amount.mean() df.amount.value_counts()                    | df.amount.mean() df.amount.count_values()                    |
| Concatenate             | pd.concat((df, df))                                          | concat(df, df)                                               |
| Column Type Information | df.dtypes df.amount.dtype                                    | df.dshape df.amount.dshape                                   |

**Blaze可以简化一些常见的IO任务，并使其更具可读性，这些任务是希望使用pandas处理的。这些例子使用的是`odo`库。许多情况下，blaze可以处理超出内存大小的数据集，而这是pandas不能够容易处理的事情。**

```python
from odo import odo
```

| Operations                  | Pandas                                                       | Blaze                                    |
| --------------------------- | ------------------------------------------------------------ | ---------------------------------------- |
| Load directory of CSV files | df = pd.concat([pd.read_csv(filename)                 for filename in                 glob.glob('path/to/*.csv')]) | df = data('path/to/*.csv')               |
| Save result to CSV file     | df[df.amount < 0].to_csv('output.csv')                       | odo(df[df.amount < 0],     'output.csv') |
| Read from SQL database      | df = pd.read_sql('select * from t', con='sqlite:///db.db')  df = pd.read_sql('select * from t',      con=sa.create_engine('sqlite:///db.db')) | df = data('sqlite://db.db::t')           |

