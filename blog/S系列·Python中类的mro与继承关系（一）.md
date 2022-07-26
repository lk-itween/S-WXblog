# S系列·Python中类的mro与继承关系（一）

S又称水，亦可读作Small，在日常工作学习过程中，偶尔会发现之前没有看见的、小的、有趣的操作，或许这些操作对于当下的问题解决并无意义，仍然想记录下来，或许能以单独写成一篇完整的文章，则作为流水账似的记下。

系列文章说明：

> S系列·<<文章名称>>

**平台：**

- windows 10.0

- python 3.8

## 类

在[`Python中的数字比较与类`](https://mp.weixin.qq.com/s/NTp90BbeouaWa9ziWMmtHg)中有简略提到类，那么什么是类呢？  

在python中定义一个类很简单，使用关键字`class`就能实现。  

```python
class Animal:
    pass
```

如何使用它，在类结构中称作实例化。  

```python
animal = Animal()
```

这样，就有了一个Animal的实例。  

## 继承

类的其中一个特性就是能够继承，把`Animal`类丰富下，将其作为基类：  

```python
class Animal:
    property_ = '能够思考'

    def __init__(self, name, age, value):
        self.name_ = name
        self.age_ = age
        self.val_ = val
```

在这里面，property_作为类属性，无需实例化就能使用，而_\_init\_\_下的self.name_, self.age_和self.val_需要在实例化后才能使用，且这里__init__需要传入参数，其中self用来指代类本身，不作为传参值。  

```python
print(Animal.property_)  # 输出：能够思考
print(Animal.name_)   # 引发AttributeError错误

a = Animal('阿黑', 12, 70)
print(a.property_)  # 能够思考
print(a.name_)  # 阿黑
print(a.age_)  # 12
print(a.val_)  # 70
```

`Animal`类可以正常使用，再写一个`Monkey`类，继承`Animal`类。  

```python
class Monkey(Animal):
    pass
```

在`Monkey`后面调用`Animal`类，继承了其属性及方法，也可通过实例化，查看`Monkey`实例的属性。  

```python
print(Monkey.property_)  # 类属性： 能够思考

m = Monkey('阿黄', 15, 40)
print(m.name_)  # 阿黄
print(m.age_)  # 15
print(m.val_)  # 40
```

当然也能继承类，对其已有的方法进行改写，这里再定义一个`Cat`类。  

```python
class Cat(Animal):

    def __init__(self, name, age):
        self.name_ = '我是' + name
        self.age_ = age
```

`Cat`类继承后对\_\_init\_\_进行了改写，修改了name_，并且删除了val_。  

```python
print(Cat.property_)  # 类属性：能够思考

c = Cat('小花', 6)
print(c.name_)  # 我是小花
print(c.age_)  # 6
print(c.val_)  # 引发AttributeError报错
```

对于类属性还是能使用，不见的val_再调用就会引发报错。  

除此之外，如果想在保留基类的属性基础上增加属性，可以用`super()`进行处理：  

```python
class Fish(Animal):

    def __init__(self, name, age, val, env):
        super().__init__(name, age, val)
        self.env_ = env
```

在实例化`Fish`类时，需要多传入一个生活环境env参数：  

```python
f = Fish('小鲤', 2, 57, '水里')
print(f.env_)  # 水里
```

## mro

`mro`的含义为`方法解析顺序`，在类的继承中，明白解析顺序是尤为重要的，对于上述几个类可以简单看下mro的顺序情况。  

```python
Animal.mro()
# [__main__.Animal, object]

Monkey.mro()
# [__main__.Monkey, __main__.Animal, object]

Cat.mro()
# [__main__.Cat, __main__.Animal, object]

Fish.mro()
# [__main__.Fish, __main__.Animal, object]
```

mro的解析顺序是从左至右，越在左边优先级越高，可以看到最先解析的是当前类本身，再是继承的上一个类，最后是原生`object`类。上述罗列的所有类，继承关系都很简单，mro的顺序也简单明了。  

## 总结

本篇仅简单介绍了类的创建及单继承类，初步认识mro的解析顺序，方便后续了解并使用类，类作为面向对象编程的重要结构，虽然不能完全理解，但需知悉一二。有关多继承将在下一篇介绍。  

-形形色色，千变万化。-  

---

<p align="right">2022.7.26留</p>


