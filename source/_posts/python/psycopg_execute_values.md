---
title: 使用Psycopg2高效更新数据
date: 2018-11-30 20:56:28
tags: [Python, Psycopg2]
categories: python
toc: true
---


> Python: 3.7
>
> Psycopg: 2.7



> 最近要对Postgresql数据库某表中的几百万条数据进行计算并更新某字段的值，在此期间使用过协程+aiopg，7分钟更新2000条数据，速度太慢；后来查看Psycopg2文档发现了一个高效的方法。



**安装Psycopg **

```bash
pip install psycopg2
```

**文档中关于高效执行的描述：[Fast execution helpers](http://initd.org/psycopg/docs/extras.html#fast-exec).**



**psycopg2.extras.execute_values(*cur*, *sql*, *argslist*, *template=None*, *page_size=100*)**

*使用该方法可以快速批量更新，其中函数中的page_size参数默认为100，表示每个statement包含的最大条目数，如果传过来的argslist长度大于page_size,则该函数最多执行len(argslist)/page_size + 1次。*

*示例程序*

```python
# coding: utf-8

import time
import datetime

from psycopg2 import connect
from psycopg2.extras import RealDictCursor, execute_values


DB_HOST = 'localhost'
DB_PORT = 5432
DB_USERNAME = 'root'
DB_PASSWORD = '123456'
DB_NAME = 'test'


def date_range():
    now = datetime.datetime.utcnow()
    start_at = datetime.datetime(2018, 1, 1, 0, 0, 0, tzinfo=tz.gettz("utc"))
    end_at = now - datetime.timedelta(days=180)

    data = {
            0: [start_at, end_at]
            }
    return data


def connection():
    with connect(database=DB_NAME,
                 host=DB_HOST,
                 port=DB_PORT,
                 user=DB_USERNAME,
                 password=DB_PASSWORD) as conn:
        with conn.cursor(cursor_factory=RealDictCursor) as cur:
            data = date_range()
            for k, v in data.items():
                for i in range(50):
                    sql = "SELECT id, score FROM review " \
                          "WHERE create_date >= %s AND create_date < %s AND score IS NULL limit 2000;" 
                    cur.execute(sql, (v[0], v[1]))

                    values = []
                    for row in cur:
                        score = 5 
                        values.append((row['id'], score))
                        sql = "UPDATE review SET score=data.score FROM (VALUES %s) AS data (id, score) " \
                              "WHERE review.id = data.id;"
                    execute_values(cur, sql, values, page_size=100)
                    print(cur.query)


if __name__ == '__main__':
    now = time.time()
    connection()
    print('time is %d' % (time.time() - now))


```

