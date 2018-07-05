### 重载(overload)
- 概念：同一个函数名，根据参数类型、参数个数的不同来区分函数
- Python的动态语言特性，因此Python没有重载（天生自带重载）

***
### 类的继承
- 从父类继承可直接拥有父类的属性和方法，子类还可以定义自己的属性和方法
- 定义格式：
```Python
    class 子类名(基类1[,基类2...])
        属性
        方法
```
- 在定义中如果没有基类名，等同于继承自object（根基类）
- 查看继承关系的特殊属性、特殊方法：
    - `__base__`: 查看类的基类
    - `__bases__`: 查看类的基类元祖
    - `__mro__`: 显示方法查找顺序，返回基类元祖（尤其用在多继承时）
    - `mro()`: 方法调用，返回基类列表
    - `__subclasses__()`: 类的子类列表

***
### 继承中的访问控制
- 按照访问顺序来搜索：实例自己的字典-> 类字典-> 父类字典->object字典
- *私有在类定义之外都是不可访问的，可以通过方法来设置访问权限*
- 公有的在子类以及继承中都是可以访问的
```Python
    class Animal:

        __COUNT = 1        # 类变量（变量名是：_Animal__COUNT）
        HEIGHT = 10 

        def __init__(self, age, hight, weight):
            self.age = age
            self.__weight = weight
            self.__COUNT += 1     # 新添加的实例变量（变量名也是_Animal__COUNT）
            self.HEIGHT = hight   # 新添加的实例变量

        def eat(self):
            print('{} eat'.format(self.__class__.__name__))

        def getweight(self):
            return self.__weight

        @classmethod
        def showcount1(cls):
            return cls.__COUNT

        @classmethod
        def __showcount2(cls):
            return cls.__COUNT

        def showcount3(self):
            return self.__COUNT

    class Cat(Animal):
        NAME = 'CAT'
        __COUNT = 20


    cat = Cat(1,2,3)
    print(cat.__dict__)
    print(Animal.__dict__)
    print(cat.NAME)
    print(cat.HEIGHT)
    print(cat.showcount1())
    print(cat.showcount3())
```

***
### 方法的重写、覆盖override
- 覆盖中的实例函数、类函数、静态函数都一样的覆盖（赋值即重新定义）
```Python
    class Animal:
        def eat(self):
            print('{} eat'.format(self.__class__.__name__))

    class Cat(Animal):
        # 覆盖父类方法
        def eat(self):
            print('miao eat')

        # 覆盖自己的方法
        def eat(self):
            super().eat()

    animal = Animal()
    animal.eat()   # Animal eat
    cat = Cat()
    cat.eat()     # Cat eat，调用了super()隐式送入self（cat类实例）
```

***
### 继承中的初始化
- 如果父类中有构造函数，在子类继承中调用父类的构造函数，就可以拥有父类的属性了
- 可以避免在调用父类方法时出错的问题（因为继承的方法要使用父类对应的变量）
    - 要注意父类构造函数的调用顺序，会对属性覆盖，带来错误

- 如果父类中的属性是私有属性：因为私有变量的只在定义范围内有效，所以会访问出错
```Python
    class Animal:

        def __init__(self,a):
            self.a = a
            self.__age = a
        def eat(self):
            print('{} eat'.format(self.__class__.__name__))

    class Cat(Animal):

        def __init__(self, a, b, c):
            # 调用父类的方法，给实例增添父类属性
            super().__init__(a)  # Animal.__init__(self, a)
            self.b = b
            self.c = c
            self.__age = a
        
        # 覆盖父类方法
        def eat(self):
            print('miao eat')
cat = Cat(10,3,3)
print(cat.__dict__)  # {'a': 10, '_Animal__age': 10, 'b': 3, 'c': 5, '_Cat__age': 10}
```
- ***注意***：在调用父类是生成了`_Animal__age`，实例自己生成了 `_Cat__age`,两个变量在各自的定义中访问不受干扰
- *针对私有变量的设计原则：自己的私有属性，就该有自己的方法读取和修改函数，不应该借用其他类的方法，即使是父类或者派生类的方法*

### super关键字
- http://python.jobbole.com/86787/