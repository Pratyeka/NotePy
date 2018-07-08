### 可调用对象
- `__call__(self)`: 在类中定义该函数，实例就可以像函数一样被调用
- 给实例对象加上()，就是调用对象的`__call__()`函数
- 等价式：`obj(eles) = obj.__call__(eles)`
- 函数与类（实现__call__()）都是可调用对象
```python
    import functools

    class Fib:
        def __init__(self):
            self._items = [0,1,1]

        @functools.lru_cache(3, True)
        def __call__(self, nth:int):
            if nth < 3:
                return 1
            else:
                ret = self.__call__(nth-1) + self.__call__(nth-2)
                self._items.append(ret)
                return ret
        
        def show(self):
            print(self._items)

    fib = Fib()
    print(fib(19))
    fib.show()
```

### 上下文管理
- 在类定义中实现了`__enter__(self)`函数、`__exit__(self,elems)`函数，该类的实例都是上下文管理的对象
- `with A() as obja:`语句的执行顺序：
    1. 执行调用`__init__`函数，构建新的A类实例
    2. 执行A()对象的`__enter__`函数，`__enter__`的返回值赋值给obja
        - 用来执行一些准备工作
    3. 执行with语句块
    4. 执行A()对象的`__exit__(self,elems)`函数
        - 即使在执行步骤3时抛出异常，也会执行步骤4（安全性高）
            - 甚至是解释器退出依旧会执行__exit__语句来释放资源
        - elems是与异常退出有关参数, 正常退出三个值都为None
            - 异常类型：exc_type
            - 异常值: exc_value
            - 追踪信息: exc_tc
        - 函数返回值是等效为True的语句，表示压制异常。反之
        - 会用来释放连接等
- 应用场景：
    - 函数功能增强（类似与函数装饰器）
    - 资源管理（在__exit__文件对象关闭、网络连接断开、数据库连接断开）
    - 权限验证（在__enter__函数中验证权限）
- with可以开启一个上下文运行环境，在__enter__中执行一些准备工作，在__exit__中执行一些收尾工作
- 函数计时器类栗子
```python
    import time
    import datetime

    # 装饰器计时函数
    # def countime(fn):
    #     def _fn(*args, **kwargs):
    #         start = datetime.datetime.now()
    #         ret = fn(*args, **kwargs)
    #         print('use time {}'.format((datetime.datetime.now() - start).total_seconds()))
    #         return ret
    #     return _fn

    def add(x,y):
        time.sleep(2)
        return x+y

    class TimeIt:
        def __init__(self, fn):
            self._func = fn

        def __enter__(self):
            self.start = datetime.datetime.now()
            return self

        def __exit__(self, exc_type, exc_val, exc_tb):
            time_use = (datetime.datetime.now() - self.start).total_seconds()
            print(time_use)

        def __call__(self, *args, **kwargs):
            return self._func(*args, **kwargs)

    with TimeIt(add) as func:
        print(func(3,5))
```
- 将类用作装饰器（将装饰器函数转为类对象）
    - 装饰器类比装饰器函数可以封装更多的数据以及方法
```python
    # 类定义不变
    @TimeIt    # 被类装饰后成为一个实例对象
    def add(x, y):
        time.sleep(1)
        return x + y

    print(add(3,5))
```
- 使用装饰器后，属性信息的修改（这里是用self（实例）来装饰func函数）
```python
    from functools import wraps,update_wrapper

    class TimeIt:
        def __init__(self, func):
            self._start = 0
            self._func = func
            wraps(func)(self) # 调用偏函数的方式实现
            # update_wrapper(self, func) # 调用函数方式实现
```
- ***注意***：
    - wraps(wrapped)->返回一个偏函数
        - 偏函数接受的参数为(wrapper)
    - 所以正确的调用方式为：wraps(wrapped)(wrapper)

### 补充 __slots__
- 解决的问题：
    - 当存在大量的实例对象时，实例属性的字典就占用了大量的内存
- 使用__slots__可以比字典节省内存
    - __slots__告诉解释器，实例的属性都有哪些
    - 一旦类属性中有__slots__,类实例就不会生成字典，且无法再添加实例属性
    - __slots__可以是列表，字典, 元祖（建议使用）
- 不会影响子类实例，不会继承下去, 子类需要自己定义__slots__
- 应用场景：
    - 存在大量的实例对象，实例属性简单/固定且不用动态增加
