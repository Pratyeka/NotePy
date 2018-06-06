### 多继承
- 多继承可以更好模拟事物之间的关系
- 建议：少用多继承
- 在Python中使用C3算法来给出多继承的搜索路径
    - 主要是为了解决多继承中出现的二义性，深度优先或者广度优先

### 抽象基类
- 抽象基类提供的方法没有具体实现，因为不一定适合子类的要求
- 要求子类必须覆盖,否则调用父类的抽象基类，会抛出未实现异常
```Python
    class Document:
        def __init__(self, content):
            self.content = content

        def print(self):
            raise NotImplementedError()

    class Word(Document):
        pass

    class Pdf(Document):
        pass
```

***
### 类动态添加功能
- 1. 装饰器
- 2. Mixin

#### 装饰器方法
- 在需要的地方进行装饰，可以使用多个装饰器函数
- 装饰器方法的优点：在需要的地方动态增加，直接使用装饰器
- 栗1
```Python
    # 类装饰器，给类增添一个实例打印函数，隐式接受self参数
    def enanimal(cls):
        cls.print = lambda self:print(self.a)
        return cls

    @enanimal
    class Animal:
        def __init__(self,a):
            self.a = a
            self.__age = a

    cat = Animal(10)
    print(cat.__dict__)
    print(Animal.__dict__)
    cat.print()
```
- 栗2:
- 先继承后装饰
```Python
    def printable(cls):
        def _print(self):
            print(self.content, '装饰器')

        cls.print = _print
        return cls

    class Document:
        def __init__(self, content):
            self.content = content

        def print(self):
            raise NotImplementedError()

    @printable  # 先继承再装饰
    class Word(Document):
        pass

    @printable
    class Pdf(Document):
        pass

    word = Word('word')
    word.print()

    pdf = Pdf('PDF')
    pdf.print()

```

#### Mixin
- 可以把 Mixin 看作多重继承的一种在特定场景下的应用，混入Mixin类是为了添加（可选的）功能。自由地混入Mixin类就可以灵活地为被混入的类添加不同的功能
- Mixin是通过多继承的方式实现的，体现了一种组合的设计模式
    - 多组合，少继承
```Python
    class Animal:
        def __init__(self,a):
            self.a = a
            self.__age = a

    class PrintAbleMixin:
        def print(self):
            print(self.a)

    class Cat(PrintAbleMixin, Animal):
        pass

    cat = Cat(10)
    print(cat.__dict__)
    cat.print()
```
- **Mixin类的使用原则**：
    - Mixin类中不应该出现显示的__init__初始化函数
    - Mixin类通常不单独工作，一般用来混入别的类中实现部分功能
    - Mixin类的祖先类也应该是Mixin类
    - Mixin类通常放在继承列表的第一位(通过mro方法，解决多继承的二义性)

### 抽象基类、Mixin方法栗子
```Py
    import math
    import json

    class Shape:
        def __init__(self, shape):
            self.shape = shape

        def area(self):
            raise NotImplementedError()

        def ser(self):
            raise NotImplementedError()

    class Triangle(Shape):
        def __init__(self, shape, a, b, c):
            super().__init__(shape)
            self.a = a
            self.b = b
            self.c = c

        def area(self):
            return self.a * self.b * self.c

    class Circle(Shape):
        def __init__(self,shape, radius):
            super().__init__(shape)
            self.radius = radius

        def area(self):
            return math.pi * self.radius * self.radius

    class Rectangle(Shape):
        def __init__(self, shape, a, b):
            super().__init__(shape)
            self.a = a
            self.b = b

        def area(self):
            return self.a * self.b

    class SerMixin:
        def dumps(self):
            return json.dumps(self.__dict__)

    class SerRectangle(SerMixin, Rectangle):
        pass

    class SerTriangle(SerMixin, Triangle):
        pass

    class SerCircle(SerMixin, Circle):
        pass

    sercircle = SerCircle('circle', 10)
    print(sercircle.area())
    print(sercircle.dumps())
```