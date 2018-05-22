### 特殊属性
- `__name__` : 函数、类、方法等的名字（字符串）
- `__class__`: 类、实例对象的类类型
- `__doc__`: 函数、类的文档字符串
- `__module__`: 函数定义、类定义所在的模块名
- `__bases__`: 类、实例对象的基类元祖（按继承列表顺序）
- `__mro__`: 类继承列表元祖
- `__dict__`: 类、实例对象的字典
- `__file__`：模块的绝对路径

***
### 魔术方法

#### 属性查看
- `__dir__(self)`: 返回实例对象的所有成员名列表
    - 要求函数的返回值是一个可迭代对象
    - `dir([obj])`: 返回obj的属性列表
        - 如果obj实现了__dir__函数，dir()会主动调用该函数返回属性类表
        - 如果obj没有实现__dir__函数，会按照继承顺序尽可能多的从__dict__中获取信息
    - `dir(cls)`: 收集类以及父类的属性信息
    - `dir()`: 默认收集当前模块的属性信息
 - ***注意***：导入模块名（import A），就是导入对应的属性名

#### 构造函数、析构函数
- `__init__(self [,a ,b])`：在生成类对象时，自动调用
    - 主要作用：资源申请（新建属性）
- `__del__(self)`: 在类对象的引用计数为0时，自动调用该函数
    - 主要是释放资源
    
#### 哈希函数
- 哈希解决了存储地址的问题（计算可哈希对象的散列值）
- 哈希值相等不代表两个对象相同（哈希冲突），需要进一步使用eq来判定是否相同

- `__hash__(self)`: 返回一个正整数
    - 类中定义该函数，则该类的实例对象可哈希（被hash()函数调用）
- `__eq__(self)`: 返回布尔值，两个对象是够相同
    - 如果类中未定义该函数，object默认使用内存地址判定
```
    class A:
        def __init__(self, name, age=18):
            self.name = name

        # def __hash__(self):
        # 	return 1

        def __eq__(self, other):
            return self.name == other.name

        def __repr__(self):
            return self.name

    print({A('tom'), A('tom')})
```
- ***注意***：上面两种函数同时出现
- list类不可hash：list的源码中有__hash__ = None


#### 布尔函数
- `__bool__(self)`：返回布尔值
    - 内建函数bool()、或者放在逻辑表达式的位置，会自动调用该函数
    - 如果没有实现该函数，则调用__len__()函数，长度非0为真，反之
    - 如果两个函数都没有实现，则全部默认为真

####　可视化函数
- `__repr__(self)`: 返回字符串数据
    - 内建函数调用该函数(repr(obj))，返回该对象的字符串表达
        - 如果该函数未定义，object类的定义：内存地址组成的字符串
- `__str__(self)`: 返回字符串数据
    - str(obj)、format(obj)、print(obj)，这三个函数调用返回obj的字符串表达
        - 如果该函数未定义，使用__repr__
        - 如果__repr__也没有实现，调用object的定义：内存地址组成的字符串
    - ***注意***：认真区分函数作用对象，非直接作用都使用__repr__函数
        - `print([obj1, obj2])`: 使用__repr__风格显示
- `__bytes__(self)`: 返回bytes表达式
    - 被bytes(obj)函数调用


#### 运算符重载
![运算符重载](https://upload-images.jianshu.io/upload_images/9112509-8fcbc78c9f3b68c7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- Point类的运算符重载实现
```
    class Point:
        def __init__(self, x, y):
            self.x = x
            self.y = y

        def __hash__(self):
            return hash((self.x, self.y))

        def __eq__(self, other):
            return self.x == other.x and self.y == other.y

        def __repr__(self):
            return "({},{})".format(self.x, self.y)

        def __add__(self, other):
            return self.__class__(self.x + other.x, self.y + other.y)

        def __sub__(self, other):
            return self.__class__(self.x - other.x, self.y - other.y)

        def __iadd__(self, other):
            self.x, self.y = self.x + other.x, self.y + other.y
            return self
        def __radd__(self, other):
            self.x, self.y = self.x + other, self.y + other
            return self

        def __isub__(self, other):
            self.x, self.y = self.x - other.x, self.y - other.y
            return self

    a = Point(3, 4)
    b = Point(3, 5)
    c = Point(3, 4)
    print(a - b)
    print(a + b)
    print(a + c)
    print(a - c)
    a -= c
    print(a)
    a += c
    print(a)

    ra = 1 + a # 调用了radd方法
    print(ra)
    m = B(3)
    rb = m + b # 调用了radd方法
    print(rb)
    # print(a == b)
    # print(a == c)
    # print({a, b, c})
```
- ***注意***：在函数中直接返回self，可以实现链式编程
- *__radd__方法注释*：
    - `1+a`等价与`1.__add__(a)`,int类实现了__add__方法，但是无法完成两个实例的元算，返回NotImplemented关键字，解释器发现是该关键字就会调用a.__radd__(1)
    - `m + b`等价于`m.__add__(b)`, m没有实现__add__方法，则调用b.__radd__(m)

#### 容器相关方法
- `__len__(self)`: 返回对象的长度（>=0的整数）
    - 被内建函数len()调用
- `__iter__(self)`: 返回一个新的迭代器对象
    - 容器迭代时调用
- `__contains__(self)`: 返回布尔值
    - in成员函数运算符，没有实现，使用__iter__方法遍历实现
- `__getitem__(self, item)`: 返回对应的value值
    - 实现self[item]的访问
    - item接受整数为索引，或者切片（list/tuple类对象）
    - item对于set和dict对象，item为可哈希类
    - item不存在抛异常
- `__setitem__(self, item, value)`: 返回None
    - 给self设置itme.value对
- `__missing__(self)`:
    - 字典或者其子类使用__getitme__(self,item)函数时，key不存在的情况下调用该函数

```
    class Snack:
        def __init__(self, price, volume):
            self.__price = price
            self.__volume = volume

        @property
        def price(self):
            return self.__price

        @property
        def volume(self):
            return self.__volume

    # 购物车
    class ShopCar:
        def __init__(self):
            self._res = {}
            self._len = 0

        def __len__(self):
            return self._len

        def __iter__(self):
            return iter(self._res.keys())

        def __getitem__(self, item):
            return self._res[item]

        def __setitem__(self, key, value):
            self._len += value
            self._res[key] = value

        def __contains__(self, item):
            return item in self._res

    shop_car = ShopCar()
    shop_car['walch'] = 2
    shop_car['silang'] = 30
    print('walch' in shop_car)
    for i in shop_car:
        print(shop_car[i])
```