# S系列·删除文件夹的几种姿势

S又称水，亦可读作Small，在日常工作学习过程中，偶尔会发现之前没有看见的、小的、有趣的操作，或许这些操作对于当下的问题解决并无意义，仍然想记录下来，或许能以单独写成一篇完整的文章，则作为流水账似的记下。

系列文章说明：

> S系列·<<文章名称>>

**平台：**

- windows 10.0

- python 3.8

- pywin32 227 （或者安装pypiwin32）

- send2trash  1.5.0

## 目的

将一个包含数据文件的目录删除。  

## 删除姿势

以下操作均在windows系统中操作。  

- 姿势一  

直接在电脑上右击该文件夹，选择删除，或者使用<key>Delete</key>按键删除文件，这种方法删除文件，会将文件转移至回收站，如果需要恢复，点击`还原`即可。  

- 姿势二  

在`python`中使用使用`os`模块：  

```python
import os

delete_dir = r'测试文件夹'
for r, d, f in os.walk(delete_dir, topdown=False):
    for files in f:
        os.remove(os.path.join(r, files)) # 删除文件
    os.removedirs(r)  # 删除文件夹，必须为空
```

使用`os.walk`遍历待删除文件的内容，默认`topdown=True`，先输出最外层的再输出内层，此时会先将非空的文件夹输出，而是用`os.removedirs`删除非空文件夹会尝试报错，将`topdown=False`，先将内层文件逐个输出并删除，最后再删除上层的文件夹，直至全部删除。  

- 姿势三  

`pathlib`模块：  

```python
from pathlib import Path

delete_dir = Path(r'测试文件夹')
# 删除所有文件
[i.unlink() for i in delete_dir.rglob('*') if i.is_file()]
# 删除包含的所有空文件夹
[i.rmdir() for i in delete_dir.rglob('*') if i.is_dir()]
# 删除当前文件夹
delete_dir.rmdir()
```

与`os`类似，将文件与文件夹分开删除，两次递归分别判断是否为文件或者目录，并按照对应删除方式删除，最后删除当前文件夹。  

- 姿势四  

`shutil`模块  

```python
from shutil import rmtree

delete_dir = r'测试文件夹'
rmtree(delete_dir)
```

`rmtree`递归返回该目录下所有文件及文件夹，并将其删除，原理同`姿势二`，以下为`rmtree`核心代码部分：  

![](https://s2.loli.net/2022/06/08/a2snMTy7fzVZlRE.png)  

![](https://s2.loli.net/2022/06/08/VshyKfRkiYOb89c.png)  

`rmtree`首先判断删除方式是否为安全删除，以安全删除方式为例，对递归结果进行判断，如果为文件夹，判断内层是否还有文件，如果还有进行递归，再对递归结果做删除操作，如果为文件，直接进行删除，此种方式相比`姿势二`有了更多对文件状态的判断。  

- send2trash+shell  

上述几种python方法删除文件会直接删除，不会经过回收站，如果想类似于姿势一的操作，可安装`send2trash`模块，将文件转移至回收站。  

`pip install send2trash`  

`pip install pywin32 (或者 pip install pypiwin32)`  

```python
import send2trash

delete_dir = r'测试文件夹'
send2trash.send2trash(delete_dir)
```

`win32com`是`pywin32`包中的一个模块，调用`shell`方法对回收站做处理。  

```python
from win32com.shell import shell, shellcon

def recyclebin_empty(confirm=True, show_progress=True, sound=True):
    flags = 0
    if not confirm:  # 提示框
        flags |= shellcon.SHERB_NOCONFIRMATION
    if not show_progress:  # 删除进度
        flags |= shellcon.SHERB_NOPROGRESSUI
    if not sound:  # 完成提示音
        flags |= shellcon.SHERB_NOSOUND
    shell.SHEmptyRecycleBin(None, None, flags)  

recyclebin_empty(False, False, False)
```

定义一个函数，设置默认会显示清空回收站提示框，显示进度及删除完提示音。两个函数相结合可以实现`姿势一`的操作过程。  

## 总结

本文通过几种姿势，对需要删除的文件进行删除操作，纯分享个人感悟，删除文件基本是有手就行，之前使用电脑偶然抽风，直接使用`shutil.rmtree`进行删除偶有卡顿，原以为这种删除方式很慢，遂上网搜索了一番，而后用`pathlib`模块却能够快速删除，正当我想分享此事时，我重启电脑再次运行，发现`shutil.rmtree`能较快删除，经几次测试，`shutil.rmtree`相比`pathlib`删除速度较快，为何那时删除时间较长原因未知。  

  

此山中烟雨，独留骚客文人。

--- 

<p align="right">2022.6.8留</>


