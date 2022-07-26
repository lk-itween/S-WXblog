# S系列·修改文件的时间属性

S又称水，亦可读作Small，在日常工作学习过程中，偶尔会发现之前没有看见的、小的、有趣的操作，或许这些操作对于当下的问题解决并无意义，仍然想记录下来，或许能以单独写成一篇完整的文章，则作为流水账似的记下。  

系列文章说明：  

> S系列·<<文章名称>>  

**平台：**  

- windows  10.0  

- python  3.8   

## 查看文件的时间属性

在windows电脑中，右击文件，再点击属性，能明显地看到创建时间，修改时间和访问时间。  

![](https://s2.loli.net/2022/05/30/5sCUzVg9DoYXBNL.png)  

![](https://s2.loli.net/2022/05/30/9JdEmAY516TDtKf.png)  

三个时间的含义如名称一样，最容易发生变动的为访问时间，只要对文件做了任何操作，都会发生变化，当对文件进行修改且保存了，修改时间就会发生变化，而创建时间是在文件创建时就已经设定了，可看到该文件中创建时间时间最早，单独对此文件左点右点是很难改变这个时间的。  

## 修改文件的时间属性

在python中可以用`os.stat`查看文件的基本属性，其中包含这三个时间属性。  

```python
import os

os.stat(r"C:\Users\admin\Desktop\1231.xlsx")
```

![](https://s2.loli.net/2022/05/30/tgHuKBPDopXjsWI.png)  

时间属性均是以时间戳的方式保存，如果尝试取出属性值修改，会产生报错，仅读属性不能修改。  

![](https://s2.loli.net/2022/05/30/Ld52nGw3NRIZQFi.png)  

`os.utime`可以帮助修改时间属性，但仅能修改访问时间和修改时间，且传入的时间参数应当为时间戳。

```python
from datetime import datetime

a_time = datetime(2024, 4, 9).timestamp()
m_time = datetime(2024, 5, 1).timestamp()
# 分别将访问时间和修改时间修改为2024-04-09和2024-05-01
os.utime(r"C:\Users\admin\Desktop\1231.xlsx", (a_time, m_time))
```

修改完后再次查看文件属性。  

![](https://s2.loli.net/2022/05/30/ZravsqiU8gl3ouY.png)  

虽然仅能看到修改时间按设想成功设置为2024-05-01，访问时间其实也是设置好了，但由于太灵活，在设置修改时间时，对文件进行了访问，从而设置成当前的操作时间。  

`os.utime(filename, (atime, mtime))`可形如这样，`atime`和`mtime`可不设定，默认为当前时间戳(s)，如传入，必须为时间戳浮点型形式，可设置成ns类型，其为整型数值。  

在os模块，无直接函数可以直接对文件创建时间进行修改。  

需要使用win32file模块调用文件句柄对创建时间进行修改，安装：`pip install pywin32`。  

```python
from win32file import CreateFile, SetFileTime, CloseHandle, GENERIC_READ, GENERIC_WRITE, OPEN_EXISTING
from datetime import datetime


def modifyFileTime(filepath, ctime, mtime, atime, format_str="%Y%m%d %H:%M:%S"):
    """
    用来修改任意文件的相关时间属性
    :param filepath: `str`， 文件路径
    :param ctime, mtime, atime: `str`， 时间格式：20210101 23:59:11
    :param format_str: `str`，默认为："%Y%m%d %H:%M:%S"
    return
    """
    # 创建文件句柄
    fh = CreateFile(filepath, GENERIC_READ | GENERIC_WRITE,
                    0, None, OPEN_EXISTING, 0, 0)
    createTimes = datetime.strptime(ctime, format_str)
    accessTimes = datetime.strptime(atime, format_str)
    modifyTimes = datetime.strptime(mtime, format_str)
    # 设置时间
    SetFileTime(fh, createTimes, accessTimes, modifyTimes)
    # 关闭句柄
    CloseHandle(fh)
```

使用这种方式可以直接对三个时间进行修改，`GENERIC_READ | GENERIC_WRITE`：可读可写。如不这样设置不能对文件进行读写修改。  

```python
modifyFileTime(r"C:\Users\admin\Desktop\1231.xlsx", '20240501 01:01:01', '20240501 02:01:01', '20240501 03:01:01')
```

分别对创建时间设置成2024年5月1号的1点、2点和3点。  

![](https://s2.loli.net/2022/05/30/fQIldDRMSN1b93C.png)  

都设置成功了。  

上面的方式使用的参数还挺多，在`os.utime`中可以修改访问时间和修改时间，只要再找一个模块修改创建时间就行，而`win32_setctime.setctime`恰好将上面的修改创建时间封装成了函数，再进一步组合，`parse`函数处理字符串型的时间可以减少对样式的考虑。  

```python
import os
from win32_setctime import setctime
from dateutil.parser import parse

def modifytime(filename, ctime, mtime, atime):
    ctime = parse(ctime).timestamp()
    atime = parse(atime).timestamp()
    mtime = parse(mtime).timestamp()

    setctime(filename, ctime)
    os.utime(filename, (atime, mtime))
```

使用这个函数将文件的创建时间、修改时间和访问时间分别设置为2024-05-02的10点、11点、12点。  

```python
modifytime(r"C:\Users\admin\Desktop\1231.xlsx", '20240502 10:00:00', '20240502 11:00:00', '20240502 12:00:00')
```

![](https://s2.loli.net/2022/05/30/PQvjDkW4MLp1RCG.png)  

## 结语

通过自身对小例子程序的实践，将文件的时间属性进行修改，对过程进行整理，仅做留存使用，不建议去修改文件的时间属性。  

---

<p align="right">2022.5.30留</p>
