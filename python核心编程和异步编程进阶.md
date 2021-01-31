# 1. python高级编程

# 2. python一切皆对象

## 函数和类也是对象

1. 赋值给一个变量
2. 可以添加到集合对象中

3. 可以作为参数传递给函数
4. 可以当做函数的返回值

## type，object和class之间的关系

type继承了object，object无继承，是最顶层的基类

object是type的一个实例，任何类都是type的实例对象

## 2.3 常见的内置类型

1. 对象的三个特征

* 内存
* 类型
* 值

2. None,全局只有一个None

# 3. 魔法函数

* \__getitem__：将对象变成迭代类型，对实例进行索引时或者切片触发,定义了本方法的可直接对对象进行迭代
  	for 会判断对象是否能处理成迭代器(iter)，如果不行则会调用本方法
* \__len__: 如果未定义使用\__getitem__

# 4. 深入类和对象

## 4-1.鸭子类型和多态

只要有一个鸟走起来像鸭子，叫起来像鸭子，我们就称之为鸭子类型，意思就是只要一个对象拥有某个方法，我们就统认为他们为鸭子类型，比如函数要求传入一个可迭代对象，那么实现了迭代协议的对象就OK

## 4-2.抽象基类abc

* 有点类似GO中的interface类型
* 抽象基类无法实例化
* collections.abc实现了一些抽象基类，可以方便我们判断对象类型
* 我们并不是常用abc抽象基类，因为自己也能实现，这些抽象基类只是给我们提供了一些文档，方便我们理解类型

## 4-3.属性的查找顺序

* 属性搜索算法：C3算法，用传统的广度和深度算法，对V和菱形继承都会出现问题，打印\__mro__可以看到查找顺序

## 4-4.私有属性

双下划线开头

## 4-5.super

super是按照mro算法进行调用的

## 4-6.mixin多继承模式

参考django_rest_framework的混合多继承模式

```python
class Displayer():
    def display(self, message):
        print(message)


class LoggerMixin():
    def log(self, message, filename='logfile.txt'):
        with open(filename, 'a') as fh:
            fh.write(message)

    def display(self, message):
        super().display(message)
        self.log(message)


class MySubClass(LoggerMixin, Displayer):
    def log(self, message):
        super().log(message, filename='subclasslog.txt')


subclass = MySubClass()
subclass.display("This string will be shown and logged in subclasslog.txt")
print(MySubClass.__mro__)
```

## 4-7.使用contextmanager上下文管理器

# 5.自定义序列类

## 5-1.查看序列定义

```python
from collections.abc import *

# 查看如何定义可迭代对象
class Iterable(metaclass=ABCMeta):

    @classmethod
    def __subclasshook__(cls, C):
        if cls is Iterable:
            return _check_methods(C, "__iter__")
        return NotImplemented
    
# 查看如何定义迭代器
class Iterator(Iterable):

    def __iter__(self):
        return self

    @classmethod
    def __subclasshook__(cls, C):
        if cls is Iterator:
            return _check_methods(C, '__iter__', '__next__')
        return NotImplemented
```

## 5-2.实现可切片的对象

```python
class My:

    def __init__(self, name_list):
        self.name_list = name_list

    def __getitem__(self, item):
        """
        使用for，使用in，使用切片，使用索引都会触发
        :param item: 
        :return: 
        """
        t = type(self)
        if isinstance(item, slice):
            return t(name_list=self.name_list[item])
        elif isinstance(item, int):
            return t(name_list=[self.name_list[item]])

    def __iter__(self):
        """
        使用for循环时触发
        :return: 
        """
        return iter(self.name_list)

    def __contains__(self, item):
        """
        使用in语法时触发
        :param item: 
        :return: 
        """
        if item in self.name_list:
            return True
        return False

    def __str__(self):
        return str(self.name_list)

m = My([1,2,3,4,5])
if 5 in m:
    print(1)
```

## 5-3.维护已排序序列

如果需要在项目中维护一个排序序列，并且数据量比较大，使用bisect库，它使用二分查找，效率比较高，我们可以无需再插入完成之后再进行耗费时间排序

```python
import bisect

l = []
bisect.insort(l, 3)
bisect.insort(l, 2)
bisect.insort(l, 5)
bisect.insort(l, 1)
bisect.insort(l, 6)
print(l)
```

## 5-4.什么时候可以不使用list

array，只能存放同样的类型，效率比list高

```python
import array

l = array.ArrayType('i')
l.append(3)
l.append(2)
l.append('1')
print(l)
```

## 5-5.列表、生成器、字典推导式

# 6.dict和set

##  6-1.dict的子类

如果需要定义自己的数据类型，不要继承原生的,原生是C写的，会有些问题，使用User的，是python重写过的

```python
from collections import UserDict
class MyDict1(dict):

    def __setitem__(self, key, value):
        super().__setitem__(key, value*2)

d = MyDict1(a=1, b=2)
print(d)
d['c'] = 3
print(d)

class MyDict2(UserDict):

    def __setitem__(self, key, value):
        super().__setitem__(key, value*2)

d = MyDict2(a=1, b=2)
print(d)
```

## 6-2.set

* frozenset：不可变集合，所以可以作为dict的key
* 学习集合的各种符号运算

## 6-3.dict和set的实现原理

不可变的才能哈希

set和dict都是使用哈希生成连续空间，所以查找特别快

# 7.变量、引用、垃圾回收

## 7-1.变量是什么

动态语言的特性，变量只是一个便利贴，指向某个值在的内存地址

小整数做了内部优化，是固定的内存地址，而其他数据类型不会，是重新生成的

## 7-2.del和垃圾回收的关系

```python
a = object
b = a
del a
print(b)
print(a)
```

## 7-3.一个经典的错误

```python

class My:

    def __init__(self, names=[]):
        self.names = names

    def add(self, name):
        self.names.append(name)

m1 = My()
m1.add('a')
m2 = My()
print(m1.names)
print(m2.names)
# 所以这里注意，不要将list设置为默认值，如果非要设置，也是判断如果未传入则重新生成一个新的空列表
```

# 8.元类编程

