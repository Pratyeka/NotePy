### 概述
- oop: 是一种认识、分析问题的方法，是软件功能大规模扩张、软件工程细致化分工的结果
- class: 一类事物的抽象，包含一类事物的共同属性及方法
    - 属性: 事物**所处的状态的描述，用数据结构来描述**
    - 方法：事物**行为的描述，抽象为一个函数**
- object: class的具象，是class的一个实例
    - 是数据和操作的封装
    - 对象之间是相互独立的，对象之间可以相互作用

***
### 面向对象的三要素
- 封装
    - 将数据和方法组装起来
    - 可以隐藏数据，对外只暴露一些接口，可以通过接口操作修改属性（数据）
- 继承
    - 多复用，继承得到的属性、方法就不用再重复实现
- 多态
    - 对继承来的类中属性或者方法个性化实现的过程就是多态

***
### Python的类
- 类定义：
```Python
    class ClassName:
        属性定义
        方法定义
```
- 注意：
    - 使用class关键字
    - 类名使用大驼峰命名规则
    - **类定义完成后，就定义了一个类对象，绑定在标识符classname上**
        - 相当于执行了类定义，对其中的属性赋值定义
```Python
    class MyClass:
        '''An Example Class'''
        x = 'abd'

        def foo(self):
            return 'My Class'

    print(MyClass.x)
    print(MyClass.foo)
    print(MyClass.__doc__)
```

- 类对象：类的定义就会产生一个类对象(将该类对象命名)
- 类的属性：类定义中的变量以及方法都是类的属性
- 类变量：类的变量，是类的所有实例共享的属性和方法（一般全大写）
    - 类变量在定义阶段就已经创建好
- 实例变量：仅属于实例自己的变量（在init函数中、或通过self.name的方式添加）
- 特殊属性：
    - `__name__`：对象名
        - 是从*type类*构建的类对象才有__name__属性
            - type类是元类，新类是以type类为模板创建的
        - 实例不是从type类构造，是从父类中构建的，所以没有__name__属性
    - `__class__`: 对象的类名
        - 新建类的返回值是type
    - `__dict__`: 对象的属性字典
        - 类属性保存在类的__dict__中
        - 实例属性保存在实例的__dict__中
- 实例属性的查找顺序：（属性访问符：`.`）（不准确版本）
    - 首先在实例自己的__dict__中查找
    - 没找到的情况下，通过__class__找到自己的类，再去类的__dict__中找
    - 如果使用__dict__[name]的方式查找就不是上述顺序，因退化为字典查找
- 属性访问符的查找顺序是：实例字典-> 类字典-> 父类字典 ->...->object
    - 在找到第一个时就停止查找

- 类定义中的方法：
    - 实例方法：在方法定义的第一个关键字为self
        - 也称为方法对象method
        - self：指代当前实例本身，***实例调用方法时隐式传参***

- 实例化
    - 真正的创建一个该类的实例对象（实例变量的赋值）
    - Python类实例化----> `类名()`，会自动调用类的__init__方法
        - __init__方法的第一个参数必须是self（完成隐式传参）
            - 在实例调用方法时，Python解释器会将方法调用对象作为参数传递给self
        - __init__方法的返回值必须是None
```Python
    class MyClass:

        def __init__(self, name, age):
            self.name = name
            self.age = age

        def showperson(self):
            print('{} is {}'.format(self.name, self.age))

    tom = MyClass('tom', 11)
    jerry = MyClass('jerry', 12)
    tom.showperson()
    jerry.showperson()
    jerry.age += 1
    jerry.showperson()
```
- *注：关于实例化的容易混淆知识点*
    - 即使初始化参数一样，创建的实例对象也是不同的实例对象
    - 实例对象会绑定方法，调用方法时，**使用的是类定义中的方法**
        - 方法属性都在类的字典中，不在实例字典中
```Python
    class a:
        def add(self):
            pass

    aa = a()
    bb = a()
    # 参数相同，但不是相同的实例对象
    print(id(aa))      # 38700368
    print(id(bb))      # 38700536
    # 实例方法相同
    print(id(aa.add))  # 6043016
    print(id(bb.add))  # 6043016
```

***
### 类中的装饰函数
- 类方法
    - 类定义中，使用@classmethod装饰器装饰的方法
    - 必须至少有一个参数，且第一个参数留给了cls
        - 使用类调用时，cls指代类自身
        - 使用实例调用时，cls表示实例对应的类
    - 通过cls可以直接操作类的属性
    - cls可以使用其他的合法标示符替换，但是不建议替换
- 基本语法
```python
    class a:
        def add(self):
            pass
        
        @classmethod
        def method(cls):
            pass
```

- 静态方法
    - 类定义中，使用@staticmethod装饰器修饰的方法
    - 静态方法只是表明这个方法属于这个名词空间。*函数归在一起，方便组织管理*
    - 调用时，***不会隐式的传入参数***
- 基本语法
```Python
    class a:
        def add(self):
            pass

        @staticmethod
        def smethod():
            pass
```

- 属性装饰器（好的设计：保护实例的属性，设定了函数外对属性的访问权限）
    - property装饰器（把对方法的操作转为对属性的访问）
        - 装饰的函数名就是以后的属性名，提供只读属性
        - 函数与变量同名会无限递归（描述器）
        ```Python
            class Data:
                def __init__(self, batchsize):
                    self.batchsize = batchsize

                @property
                def batchsize(self):
                    return self.batchsize
        ```
    - setter装饰器
        - 与属性同名，接受两个参数，第一个是self，第二个是参数值，提供可写属性
    - deleter装饰器
        - 可以控制是否删除属性
- **注意：要求property必须在前，setter、deleter装饰器在后**   
```python
	@property
	def batchsize(self):
		return self._batchsize

	@batchsize.setter
	def batchsize(self,batchsize):
		self._batchsize = batchsize
```

***
### 访问控制
- 私有属性: 限制该变量只能在该类的定义中被访问
    - 使用双下划线开头的属性名（变量以及方法），就是私有属性
        - 本质是解释器将其改名为(`_类名__属性名`)，可在__dict__中查看        
- 保护属性：为了保护变量，认为约定不对这些属性进行操作
    - 使用单下划线开头的属性（变量以及方法），就是保护属性
        - 解释器不做操作，程序员之间的默认规范

### 方法调用
- 类方法调用：
    - `cls.classname(cls)`: 类方法调用
    - `self.classname(self.__class__)`: 自动将self转为对应的class
- 实例方法的调用
    - `cls.selfmethod(self)`: 只能手动赋值self，实现类调用实例方法
    - `self.selfmethod()`: self有隐式的自动传参
- 静态方法的调用
    - `cls.staticmethod()`: 默认不接受参数
    - `self.staticmethod()`: 默认不接受参数，隐式传入的参数也会被处理
- 总结：
    - 类可以调用除了实例方法外的其他方法（手动赋值self，类也可以调用实例方法）
    - 实例可以调用定义中的全部方法

### 对象的销毁
- 在类中定义`__del__`方法，称为析构方法
- 作用：在销毁类的实例时调用，来释放资源（释放连接等）
- **注意**：这个方法不能真正引起对象的销毁，只是在对象销毁时候会主动调用该函数