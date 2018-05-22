### 多继承
- 多继承可以更好模拟事物之间的关系
- 建议：少用多继承
- 在Python中使用C3算法来给出多继承的搜索路径

***
### 类动态添加功能
- 1. 装饰器
- 2. Mixin

#### 装饰器方法
- 在需要的地方进行装饰，可以使用多个装饰器函数
```
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

#### Mixin
- 可以把 Mixin 看作多重继承的一种在特定场景下的应用，混入Mixin类是为了添加（可选的）功能。自由地混入Mixin类就可以灵活地为被混入的类添加不同的功能
- Mixin是通过多继承的方式实现的，体现了一种组合的设计模式
    - 多组合，少继承
```
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