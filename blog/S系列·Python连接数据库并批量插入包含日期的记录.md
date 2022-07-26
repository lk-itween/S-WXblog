# S系列·Python连接数据库并批量插入包含日期的记录

S又称水，亦可读作Small，在日常工作学习过程中，偶尔会发现之前没有看见的、小的、有趣的操作，或许这些操作对于当下的问题解决并无意义，仍然想记录下来，或许能以单独写成一篇完整的文章，则作为流水账似的记下。

系列文章说明：

> S系列·<<文章名称>>

**平台：**

- windows 10.0

- python 3.8

- oracle

- mysql

## 目的

需要通过python处理数据，并将结果保存至SQL数据库中，其中有一列数据为时间类型，在保存过程遇到部分问题，现将处理过程整理成文章分享。  

需要保存的数据类似于下方类型：  

```python
from datetime import datetime
import pandas as pd

df = pd.DataFrame({'time': datetime.now().replace(microsecond=0),
                   'idx': [80, 90]})
```

## 处理方法

- Oracle：本例连接方式采用jdk连接，具体操作过程可自行查阅资料。  

编写SQL语句，假设连接对象为`conn`，批量插入数据。  

```python
sql = "INSERT INTO Test_Table (Time, idx) VALUES(:1, :2)"

cursor = conn.cursor()  # 获取游标
try:
    cursor.executemany(sql, df.values.tolist())  # 将df数据插入数据库中
except Exception as e:
    conn.rollback()  # 如果插入失败，回滚
    print(f'插入失败, {str(e)}')
else:
    conn.commit()  # 插入成功，提交记录
finally:
    cursor.close()  # 关闭游标
```

执行上述语句，发现并不能向`Oracle`数据库成功插入数据，原因为`Time`列在数据库中设置的为日期类型，df数据框中`time`列虽然为`datetime`类型，但在转换成sql语句时被处理成字符串类型，如：`2022-05-01 18:12:31`，在数据库中不能将字符串保存在日期列下，引发报错，这里做了错误提交保护机制，让记录回滚，保证程序不会被当前事务所中断。  

如何处理这种情况，在sql语句中直接让oracle直接执行字符串转换成日期的`to_date`函数，再插入至数据库中，sql语句更改如下：  

```python
sql = "INSERT INTO Test_Table (Time, idx) VALUES(to_date(:1,'yyyy-mm-dd HH24:MI:SS'), :2)"
```

其中的日期格式要根据需要插入的字符串日期来设定，小时可设置成24小时制。  

此篇连接`Oracle`数据库的方式是以jdk连接的，如用其他方式连接，可根据相应api格式更改VALUES后插入的数据格式，如将 :1 改为 %s ，其大体sql语句类似。  

- Mysql：mysql.connector方式连接  

`pip install mysql-conncetor-python`  

导入方式：`import mysql.connector`    

具体连接方式可自行翻阅资料，与`pymysql`连接类似。

与`Oracle`略有不同为sql语句编写：  

```python
sql = "INSERT INTO Test_Table (time, idx) VALUES (%s, %s)"

cursor = conn.cursor()  # 获取游标
try:
    cursor.executemany(sql, df.values.tolist())  # 将df数据插入数据库中
except Exception as e:
    conn.rollback()  # 如果插入失败，回滚
    print(f'插入失败, {str(e)}')
else:
    conn.commit()  # 插入成功，提交记录
finally:
    cursor.close()  # 关闭游标
```

`Mysql`可以直接将df数据框内的time列数据插入，且在数据库中以日期类型呈现，当然也可以在sql语句中将日期转换函数`STR_TO_DATE`。

```python
sql = "INSERT INTO Test_Table (time, idx) VALUES (STR_TO_DATE(%s, '%Y-%m-%d %H:%i:%S'), %s)"
```

注意到sql语句中日期格式与python日期格式稍有不同，如果日期中包含毫秒，可在日期类型最后加上`.%f`帮助转换。  

## 总结

本文简单地将数据框数据通过使用python连接`Oracle`和`Mysql`数据库，根据数据库特点编写SQL语句，顺利将日期类型数据保存至数据库中，在执行过程中发现`Mysql`数据库在保存日期类型数据容忍度更高，允许日期列保存的数据为字符串类型，而`Oracle`需要通过函数将字符串转换为日期类型，不排除当前测试用数据库版本较低的可能原因。  

在任何时候都应该有继续探索的精神。

----

<p align="right">2022.6.23留</p>
