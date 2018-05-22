### 引入
- 程序执行流程
    - `x = A()`: 在定义B类时执行A()完成对属性 x 的定义
    - `b = B()`: 调用B类的__init__函数，构建B类的实例对象
    - `b.x.a1` : 通过链式方法，调用到A类实例的属性值
```
    class A:
        def __init__(self):
            self.a1 = 'a1'
            print('A.init() ++++')

    class B:
        x = A() # 定义时调用A类的__init__函数
        def __init__(self):
            print('B.init() ----')

    print(B.x.a1)
    b = B()
    print(b.x.a1)
```
### 描述器
- [描述器](http://youchen.me/2017/02/04/Python-property%20access/)
- [知乎帮助](https://zhuanlan.zhihu.com/p/25566017)
- [Guide](http://pyzh.readthedocs.io/en/latest/Descriptor-HOW-TO-Guide.html)

***
- 将类A修改为描述器，描述器定义：
    - 首先要求类A的定义中实现了`__get__`/`__set__`/`__delete__`中任意一个
        - 只实现`__get__`方法，称为非数据描述器
        - 实现了`__get__`和`__set__`方法， 称为数据描述器 
    - 要求B类的属性x必须为类属性，且该类属性是一个类的实例对象
        - 会重写默认的查找行为
        - 描述器一定是类属性
    - ***注意***：函数也是对象，所以x与__init__都是通过描述器来实现的
- 类B的类属性设置为描述器，则类B被称为owner属主
```
    class A:
        def __init__(self):
            self.a1 = 'a1'
            print('A.init() ++++')

        # self表示B类中初始化的A()
        # instance：‘b.x’时表示实例b；‘B.x’时表示None
        # owner：表示B（x的所属类）
        def __get__(self, instance, owner):
            print("A.__get__ {} {} {}".format(self, instance, owner))

    class B:
        x = A() # 定义时调用A类的__init__函数
        def __init__(self):
            print('B.init() ----')

    print(B.x) # 此时x已经被编译成描述器
    # 打印结果为 A(), None, B
    b = B()
    print(b.x) # 调用到的x是描述器
    # 打印结果为 A(), b, B
```
***
- 描述器在属性访问时被自动调用
### 属性访问顺序
- 非数据描述器访问顺序
    - 给类B的实例添加属性x，`b.x`调用时会现在b的属性字典中查找，再去类的属性字典中查找
        - 这种情况下 x 不是属性描述器
```
    class A:
        def __init__(self):
            self.a1 = 'a1'
            print('A.init() ++++')

        def __get__(self, instance, owner):
            print("A.__get__ {} {} {}".format(self, instance, owner))

    class B:
        x = A() # 定义时调用A类的__init__函数
        def __init__(self):
            print('B.init() ----')
            self.x = 'B instance attributer x'

    print(B.x) # 此时x已经被看作是描述器
    b = B()
    print(b.x) # 调用到的x是实例属性
```
- 数据描述器属性访问
    - 给描述器增加`__set__`方法, 定义中的x都是数据描述器
    - `self.x = '  '`：x是描述器，调用了`__set__`方法
- 数据描述器属性访问优先于字典原因：
    - 将实例字典中同名属性抹去，造成了数据描述器优先于字典的假象
```
    class A:
        def __init__(self):
            self.a1 = 'a1'
            print('A.init() ++++')

        def __get__(self, instance, owner):
            print("A.__get__ {} {} {}".format(self, instance, owner))
        def __set__(self, instance, value):
            print("A.__set__ {} {} {}".format(self, instance, value))
            self.data = value # 给当前实例添加属性

    class B:
        x = A() # 定义时调用A类的__init__函数
        def __init__(self):
            print('B.init() ----')
            self.x = 'B instance attributer x' # x是描述器，调用了`__set__`方法

    print(B.x) # 此时x已经被看作是描述器，调用__get__方法
    b = B()
    print(b.x) # 调用到的x是描述器，调用__get__方法
```
- `__set__`函数使用（动态调用）
    - 实例：`b.x=100`调用了`__set__`函数
    - 类：  `B.x=100`不调用`__set__`函数，直接覆盖
```
    class A:
        def __init__(self):
            self.a1 = 'a1'
            print('A.init() ++++')

        def __get__(self, instance, owner):
            print("A.__get__ {} {} {}".format(self, instance, owner, self.data))
        def __set__(self, instance, value):
            print("A.__set__ {} {} {}".format(self, instance, value))
            self.data = value # 给当前实例添加属性

    class B:
        x = A() # 定义时调用A类的__init__函数
        def __init__(self):
            print('B.init() ----')
            self.x = 'B instance attributer x' # x是描述器，调用__set__方法
    b = B()
    b.x   # 调用__get__函数
    b.x = 100 # 调用__set__函数
    b.x   # 调用__get__函数
    B.x = 100 # 类属性的覆盖(为什么与实例的赋值不同：帮助说不去调用__set__)
```
- 从帮助可知
    - 实例在可以调用__get__、__set__函数
    - 类在访问中可以调用__get__函数，但是不调用__set__函数