# S系列·Python中类的mro与继承关系（二）

S又称水，亦可读作Small，在日常工作学习过程中，偶尔会发现之前没有看见的、小的、有趣的操作，或许这些操作对于当下的问题解决并无意义，仍然想记录下来，或许能以单独写成一篇完整的文章，则作为流水账似的记下。

系列文章说明：

> S系列·<<文章名称>>

**平台：**

- windows 10.0

- python 3.8

## 多重继承

在[`Python中的数字比较与类`](https://mp.weixin.qq.com/s/NTp90BbeouaWa9ziWMmtHg)中有简略提到类，且在[`S系列·Python中类的mro与继承关系（一）`]稍有解释继承关系，用到一个基类`Animal`如下：  

```python
class Animal:
    property_ = '能够思考'

    def __init__(self, name, age, value):
        self.name_ = name
        self.age_ = age
        self.val_ = val
```

再定义`Action`活动作为另一个基类：  

```python
class Action:

    def __init__(self, action, val):
        self.action_ = action
        self.val_ = val
```

- 现在需定义一个`Dog`类，不仅是动物，还能够跑，可以来继承上面两个类来定义：  

```python
class Dog(Animal, Action):

    def __init__(self, name, age, action, val):
        Animal.__init__(self, name, age, val+1)
        Action.__init__(self, action, val)

dog = Dog('大福', 8, '跑', 78)
print(dog.__dict__)
# {'name_': '大福', 'age_': 8, 'val_': 78, 'action_': '跑'}
```

发现打印出的实例属性，好像字典的键值更新，先初始化Animal时，val传入的值为79，而后被更新为78，这里为什么不能像继承单个类一样直接用super方法代替呢。

上一篇有提到mro解析顺序，可进行尝试，不重写\_\_init\_\_方法，发现`Dog`类只能传入三个参数，且都为`Animal`类的参数，因为继承的两个父类都有该方法，优先继承左边的父类方法，如果想都继承可以考虑这样的形式，然而会提高后续维护的困难性。  

可以将最左边的父类改成super方式：  

```python
class Dog(Animal, Action):

    def __init__(self, name, age, action, val):
        super().__init__(name, age, val+1)
        Action.__init__(self, action, val)
```

mro解析顺序，与上面所述一致：  

```python
Dog.mro()
# [__main__.Dog, __main__.Animal, __main__.Action, object]
```

- 祖孙类

如再进行继承，视`Dog`为父类，其`Animal`,`Action`都为祖父类，定义一个`Pet`类：  

```python
class Pet(Dog):

    pass

pet = Pet('大福', 8, '跑', 78)
```

传入参数，和实例化的对象跟`Dog`一样，如果需要改写某个方法，可以参照之前的方法进行改写，另外若在保留原方法的逻辑上进行补充则用super方法。  

`Pet`类的mro：  

```python
Pet.mro()
# [__main__.Pet, __main__.Dog, __main__.Animal, __main__.Action, object]
```

## 思考片刻

通过上面的继承及对应的mro解析顺序，可以思考以下通过多重继承类后，输出的x属性值为多少：  

```python
class Alpha:
    def __init__(self, val):
        self.x = val

class Beta(Alpha):
    pass

class Gamma:
    def __init__(self, val):
        self.x = val + 1

class Omega(Gamma):
    def __init__(self, val):
        super().__init__(val + 1)

class Kappa(Beta, Omega):
    pass

k = Kappa(1)
print(k.x)
```

如果脑内没有一个mro解析顺序图，这里准备了：  

```python
[__main__.Kappa, __main__.Beta, __main__.Alpha, __main__.Omega, __main__.Gamma, object]
```

这里或许会有疑问，`Beta`后面不是`Omega`吗？怎么到`Alpha`了，可以先看下`Omega`，继承`Gamma`，而`Gamma`跟`Alpha`并不是同源的，类似于`Dog`类的继承，那么优先就会使用`Alpha`的\_\_init\_\_方法，所以在传入参数值1的时候，仅运行了`Alpha`内的self.x = val，属性x被赋值成1，在最后print输出即为1，打印结果检查：  

```python
print(k.x)
# 1
```

若把`Gamma`类改成继承`Alpha`类，再次猜测print(k.x)的值为多少？  

```python
class Alpha:
    def __init__(self, val):
        self.x = val

class Beta(Alpha):
    pass

class Gamma(Alpha):
    def __init__(self, val):
        self.x = val + 1

class Omega(Gamma):
    def __init__(self, val):
        super().__init__(val + 1)

class Kappa(Beta, Omega):
    pass

k = Kappa(1)
print(k.x)
```

查看mro解析顺序：  

```python
[__main__.Kappa, __main__.Beta, __main__.Omega, __main__.Gamma, __main__.Alpha, object]
```

此时发现`Alpha`解析优先级排在最后，`Beta`跟`Omega`可以看做是`Beta`跟`Gamma`的优先级比较，因为`Omega`继承`Gamma`，且重写了\_\_init\_\_方法，所以当传入参数时会对`Gamma`类的属性进行赋值，虽然`Beta`类直接继承`Alpha`，但`Gamma`类也直接继承，所以`Alpha`解析顺序需要排在`Gamma`后面，从而当`Kappa`类传入参数时，经过`Omega`的super加1，传入到Gamma处时为：self.x = val + 1中的val为2，输出的k.x的值即为3，查看打印结果：  

```print
print(k.x)
# 3
```

## 总结

通过连续两篇对类继承及mro解析顺序的说明，理解类在多重继承中的变化，无论继承多少遍，总归要回归本心，但也不能胡乱继承，有条理的，有意义的继承，才能让自己乃至他人更好理解当下写出的类。  

-我想未来会有一段时间，来怀念现在的自己。-  

--- 

<p align="right">2022.7.27留</p>
