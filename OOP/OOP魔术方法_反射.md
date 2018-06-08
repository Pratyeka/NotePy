### 概述
- 运行时（区别于编译时），指的是程序被加载到内存中执行的时候
- 反射（/自省）：在运行时获取类型定义信息
    - Python中，表现为通过一个对象找出其type/class/attribute/method的能力
    - 具有反射能力的函数：`type()、isinstance()、callable()、dir()、gatattr()`

### 反射相关的函数和魔术方法
***
####属性获取
- `getattr(object, name [, default])`: 通过name返回object的属性值。
    - 属性值不存在时，将使用default返回
    - 如果不存在default，则抛出AttributeError
    - 要求name必须是字符串
 - 利用getattr实现命令分发器
 ```Python
    class Dispatcher:
        def __init__(self):
            pass

        def cmd1(self):
            print('cmd1')

        def cmd2(self):
            print('cmd2')

        def run(self):
            while True:
                cmd = input("input the cmd: ").strip()
                if cmd == 'quit':
                    break
                getattr(self, cmd, lambda : print('Unkonwn func'))()

    dis = Dispatcher()
    dis.run()
 ```
- `__getattr__(self, item)`:
    - 实例对象的属性搜索顺序为：实例字典->类字典->父类字典s->`__getattr__()`
    - 如果没有查到对应的类属性并且没有`__getattr__`，抛AttributeError异常
    - item 必须是字符串数据
    - 对类的属性访问不起作用
```Python
    class Base:
        n = 0

    class Point(Base):
        z = 6
        def __init__(self, x, y):
            self.x = x
            self.y = y

        def show(self):
            print(self.x, self.y)

        def __getattr__(self, item):
            return 'Missing {}'.format(item)

    p1 = Point(5,6)
    print(p1.x)
    print(p1.z)
    print(p1.n)
    print(p1.t)       # missing t : 验证.运算符的搜索顺序
```

***
#### 属性设置
- `setattr(object, name, value)`: 给object设置属性
    - 若object中存在该属性，覆盖该属性
    - 不存在该属性，增添属性对
- ***注意***：可以使用该函数给(类/实例)添加函数
    - 给类添加函数，使用方法与类定义中的方法调用方法一致（绑定）
    - 给实例添加函数，没有实例隐式传参，调用方式：p.foo(p)
        - 手动传入实例对象
- `__setattr__(self, item, value)`：属性赋值时使用
    - `__init__()`函数中的`self.c = a`赋值语句会调用__setattr__函数
    - setattr(obj)会调用该魔术方法（小心出现递归）
        - 可以用来拦截对实例属性的增加、修改操作
        - 如果要设置生效，需要直接对实例的字典进行操作
    - 在函数外`object.z = 100`也会调用该方法
```Python
    class Base:
        n = 0

    class Point(Base):
        z = 6
        def __init__(self, x, y):
            self.x = x    # 触发__setattr__函数，导致属性没有添加到字典中
            self.y = y

        def show(self):
            print(self.x, self.y)

        def __getattr__(self, item):
            return 'Missing {}'.format(item)

        def __setattr__(self, key, value):
            print('setattr {} = {}'.format(key, value))
            # self.__dict__[key] = value # 在字典中添加实例
            """
            为什么实例的字典属性调用不会出现递归：
            
            """
    p1 = Point(5,6)   
    print(p1.x) # 因为__setattr__导致字典中没有该属性，所以到__getattr__
    print(p1.z)
    print(p1.n)
    print(p1.t)
    p1.t = 1000 # 添加属性
```


#### 属性查询
- `hasattr(object, name)`: 判断对象是否有这个属性
    - name必须是字符串

- `del p.x`: 删除对象属性
    - 会调用`__delattr__`函数
    - `__delattr__(self, item)`: 删除实例属性
        - 可以阻止使用实例删除属性的操作，但是通过类依然可以删除属性

- `__getattributer__(self, item)`: 暂时理解为（属性访问符`.`重载）
    - 属性访问运算符首先会调用该函数
    - 可以用来拦截对特定属性的访问
    - 一般建议在方法中用`object.__getattributer__(self, item)`方法（实现正确属性搜索）
    - 在语句块中主动抛出AttributeError，会直接调用到__getattr__函数
    - 如果没有主动抛出AttributError异常，会按照正常的属性搜索顺序访问
    - 如果在函数中使用了self.name,会出现递归调用
- *** 注意 ***：
    - 如果不是十分明确`__getattribute__`函数的作用，不建议使用它

```Python
class Point:
	def __init__(self, x, y):
		self.x = x  # 调用__setattr__函数
		self.y = y  # 调用__setattr__函数

	def __repr__(self):
		return "Point({}, {})".format(self.x, self.y)

	def show(self):
		print(self.x,  self.y)

	def __getattr__(self, item):
		return "missing {}".format(item)

	def __setattr__(self, key, value):
		self.__dict__[key] = value
		print("{} add the {}".format("P", key))

	def __delattr__(self, item):
		print("Please don't del item {}".format(item))

    p = Point(4, 5)
    p.m = 10   # 调用__setattr__函数
    del p.m    # Please don't del item m (调用__delattr__函数)
    setattr(p, 'x', 100)  # 给实例对象p添加属性x,调用__setattr__函数
    setattr(Point, 'show', lambda self:print(self.x, self.y)) # 给类添加方法（实例调用自动绑定）
    p.show()
    setattr(p, 'add', lambda self:self.x + self.y) # 给实例添加方法（调用方法：p.add(p), 手动传参）
    print(p.add(p))
    print(p.__dict__)
    # print(getattr(p, 'z'))
```
- 动态增加属性相比于装饰器、Mixin类，更加灵活，可以在运行期间为类添加属性
- 并且可以给实例对象添加函数