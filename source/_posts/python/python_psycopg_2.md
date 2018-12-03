---
title: 使用Psycopg2高效更新数据(二)
date: 2018-12-03 23:00:28
tags: [Python, Psycopg2]
categories: python
toc: true
---



> python: 3.7
>
> pscopg2: 2.7



参考文档[Server side cursors](http://initd.org/psycopg/docs/usage.html#server-side-cursors)

*当执行一个数据库查询时，Pscopg cursor通常将查询到的所有数据返回给客户端，如果返回的数据过大，则将占用客户端大量的内存。因此，psycopg提供了一种成为`server side curosr`机制，每次返回可控制数量的数据。*

*Server side cursor是使用PostgreSQL的`DECLARE`命令创建，并经过`MOVE`、`FETCH`和`CLOSE`命令处理的。*

*Psycopg通过命名的cursors装饰server side cursor的，而命名cursor是通过对`cursor()`方法指定`name`参数而创建的。server side cursor允许用户在数据集中使用`scroll()`移动游标，并通过`fetchone()`和`fetchmany()`方法获取数据。*

* scrollable：控制游标是否可以向后移动
* itersize：控制每次可以获取多少条数据，默认是2000



*当命名cursor尝试在`commit()`方法执行后获取数据或者由一个`autocommit	`模式的connection创建命名cursor时都将导致一个错误。*

*使用示例：fetchone*

```python
import time
import datetime
from dateutil import tz

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
        with conn.cursor(name='server_cursor', cursor_factory=RealDictCursor) as cur:
            data = date_range()
            for k, v in data.items():
                sql = "SELECT id, score FROM review " \
                      "WHERE create_date >= %s AND create_date < %s AND score IS NULL;"
                cur.execute(sql, (v[0], v[1]))
                
                for c in cur:
                    print(c)


if __name__ == '__main__':
    now = time.time()
    connection()
    print('time is %d' % (time.time() - now))
```

*使用示例：fetchmany*

```python
import time
import datetime
from dateutil import tz

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
        with conn.cursor(name='server_cursor', cursor_factory=RealDictCursor) as cur:
            data = date_range()
            for k, v in data.items():
                sql = "SELECT id, score FROM review " \
                      "WHERE create_date >= %s AND create_date < %s AND score IS NULL;"
                cur.execute(sql, (v[0], v[1]))

                while True:
                    rows = cur.fetchmany(2000)
                    if len(rows) > 0:
                        values = []
                        for row in rows:
                            score = 5
                            values.append((row['id'], score))
                        sql = "UPDATE review SET score=data.score FROM (VALUES %s) AS data (id, score) " \
                              "WHERE review.id = data.id;"
                        execute_values(cur, sql, values, page_size=100)
                    else:
                        break


if __name__ == '__main__':
    now = time.time()
    connection()
    print('time is %d' % (time.time() - now))
```

