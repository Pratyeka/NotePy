### 概述
- 运行时（区别于编译时），指的是程序被加载到内存中执行的时候
- 反射（/自省）：在运行时获取类型定义信息
    - Python中，表现为通过一个对象找出其type/class/attribute/method的能力
    - 具有反射能力的函数：type()、isinstance()、callable()、dir()、gatattr()

### 反射相关的函数和魔术方法
- `getattr(object, name [, default])`: 通过name返回object的属性值。
    - 属性值不存在时，将使用default返回
    - 如果不存在default，则抛出AttributeError
    - 要求name必须是字符串
- `__getattr__(self, item)`:
    - 实例对象的属性搜索顺序为：实例字典->类字典->父类字典s->__getattr__()
    - item 必须是字符串数据
    - 对类的属性访问不起作用
 
- `setattr(object, name, value)`: 给object设置属性
    - 若object中存在该属性，覆盖该属性
    - 不存在该属性，增添属性对
- ***注意***：可以使用该函数给(类/实例)添加函数
    - 给类添加函数，使用方法与类定义中的方法调用方法一致（绑定）
    - 给实例添加函数，没有实例隐式传参，调用方式：p.foo(p)
        - 手动传入实例对象
- `__setattr__(self, item, value)`
    - 是给`__init__()`函数中的self.c = a赋值语句调用
    - setattr(obj)会调用该魔术方法（小心出现递归）
        - 可以用来拦截对实例属性的增加、修改操作
        - 如果要设置生效，需要直接对实例的字典进行操作
    - 在函数外p.z = 100也会调用该方法

- `hasattr(object, name)`: 判断对象是否有这个属性
    - name必须是字符串

- `del p.x`: 删除对象属性
    - `__delattr__(self, item)`: 删除实例属性
        - 可以阻止使用实例删除属性的操作，但是通过类依然可以删除属性

- `__getattributer__(self, item)`: 暂时理解为（属性访问符`.`重载）
    - 属性访问运算符首先会调用该函数
    - 可以用来拦截对特定属性的访问
    - 一般建议在方法中调用基类的`__getattributer__(self, item)`方法（以便实现正确的属性搜索）
    - 在语句块中主动抛出AttributeError，会直接调用到__getattr__函数
    - 如果没有主动抛出AttributError异常，会按照正常的属性搜索顺序访问
    - 如果在函数中使用了self.name,会出现递归调用
- *** 注意 ***：
    - 如果不是十分明确`__getattribute__`函数的作用，不建议使用它

```
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
    del p.m  # Please don't del item m (调用__delattr__函数)
    setattr(p, 'x', 100)  # 给实例对象p添加属性x
    setattr(Point, 'show', lambda self:print(self.x, self.y)) # 给类添加方法（实例调用自动绑定）
    p.show()
    setattr(p, 'add', lambda self:self.x + self.y) # 给实例添加方法（调用方法：p.add(p), 手动传参）
    print(p.add(p))
    print(p.__dict__)
    # print(getattr(p, 'z'))
```
- 动态增加属性相比于装饰器、Mixin类，更加灵活，并且可以给实例对象添加函数