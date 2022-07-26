# S系列·向文本文件新增数据并且不添加重复值

S又称水，亦可读作Small，在日常工作学习过程中，偶尔会发现之前没有看见的、小的、有趣的操作，或许这些操作对于当下的问题解决并无意义，仍然想记录下来，或许能以单独写成一篇完整的文章，则作为流水账似的记下。

系列文章说明：

> S系列·<<文章名称>>

**平台：**

- windows 10.0

- python 3.8

## 目的

向文本文件（可能不存在）新增一列数据并且文本文件中的数据不重复。  

## 处理方法

- 文本文件不存在  

```python
data = ['243', '122', '782', '577', '478', '334', '334', '738', '122', '112', '634']
```

需要保存的数据如上，其中字符串'122'和'334'均有重复值，可以在保存前进行去重处理。  

```python
data2 = list(set(data))
```

上述使用了`set`函数进行去重，如果需要保持原顺序，则可按以下方式进行排序，或者自拟函数进行去重。  

```python
data2.sort(key=data.index)
```

文件不存在可以使用`mode='w'`模式，可指定`encoding='utf-8'`编码进行保存。  

```python
with open('test.txt', 'w', encoding='utf-8') as f:
    f.write('\n'.join(data2) + '\n')
```

- 文本文件存在  

假设已经存在`test.txt`文件，且其内容如上，需要将新数据（如下）新增至文本文件中。  

```python
new = ['243', '122', '989', '989', '577', '159']
```

这里需要解决两个问题：  

1. 新增的数据本身需要去重  

2. 新增的数据与已有的数据也需要做去重处理  

观察来看，'989'是一个重复值，而'243', '122', '577'已经存在了，即只需要将一个'989'和'159'新增至`test.txt`中。  

```python
new = ['243', '122', '989', '989', '577', '159']

with open('test.txt', encoding='utf-8') as f:
    data_list = []
    r_data = f.readline()
    while r_data.strip():
        data_list.append(r_data.strip())
        r_data = f.readline()

new2 = list(set(new).difference(set(data_list)))
new2.sort(key=new.index)

with open('test.txt', 'a', encoding='utf-8') as f:
    f.write('\n'.join(new2) + '\n')
```

`第一个with open`先将文本文件的内容读取出来，使用逐行读取的方式，减少内存使用，并且在每行读取的时候去除换行符，若一次性读取仍然需要处理换行符，综合考虑下使用逐行读取的方式，再将需要新增的数据与已有数据进行判断仅在new中出现的数据，在`第二个with open`采用`mode='a'`方式将需要新增的数据写入至文本中。  

需要分开用两个open来进行读写，稍有麻烦，可以将mode设置为a+，在新增的模式上还能够读出数据。  

```python
new = ['243', '122', '989', '989', '577', '159', '777']

with open('test.txt', 'a+', encoding='utf-8') as f:
    f.seek(0)  # 将文件游标置于文件开头
    data_list = []
    r_data = f.readline()
    while r_data.strip():
        data_list.append(r_data.strip())
        r_data = f.readline()

    new2 = list(set(new).difference(set(data_list)))
    new2.sort(key=new.index)
    f.write('\n'.join(new2) + '\n')
```

跟上一方法对比，除open的数量减少为一个外，需要将对比去重的代码移至with open之内，使用'a'模式打开，会默认将文件的游标处于文本最后，使新增数据在末尾处新增，每次使用read，游标会后移，在使用readline时也是可以读到文件的最后，由于a模式游标在最后，直接使用read是无法将已有数据读出，需要使用seek，将游标置于文本开头处，这时再进行read就能按设想的方式读出已有的数据，再将需要新增的数据进行去重比较，写入。  

## 总结

如何将数据新增至已有文本中，并且不含重复数据，从文件不存在入手，逐步递增，最终使用a+模式写入，兼并了文件不存在的情况，将游标设置在开头，将已存在的数据顺利读取，再做去重处理，并未考虑其他因素，在代码设计方面可能存在纰漏。  

如凶猛鹰隼，桀骜不驯。  

---

<p align="right">2022.6.7留</p>
