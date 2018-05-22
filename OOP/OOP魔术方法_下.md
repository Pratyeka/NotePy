### 可调用对象
- `__call__(self)`: 在类中定义该函数，实例就可以像函数一样被调用
- 给实例对象加上()，就是调用对象的`__call__()`函数
- 等价式：`obj(eles) = obj.__call__(eles)`
- 函数与类（实现__call__()）都是可调用对象
```
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
    3. 执行with语句块
    4. 执行A()对象的`__exit__(self,elems)`函数
        - 即使在执行步骤3时抛出异常，也会执行步骤4（安全性高）
        - elems是与异常退出有关参数, 正常退出三个值都为None
            - 异常类型：exc_type
            - 异常值: exc_value
            - 追踪信息: exc_tc
        - 函数返回值为True，表示压制异常。反之
        - 会用来释放连接等
- 应用场景：
    - 函数功能增强（类似与函数装饰器）
    - 资源管理（在__exit__文件对象关闭、网络连接断开、数据库连接断开）
    - 权限验证（在__enter__函数中验证权限）

```
    import datetime
    import time

    class TimeIt:
        def __init__(self, func):
            self._start = 0
            self._func = func

        def __enter__(self):
            self._start = datetime.datetime.now()
            return self

        def __exit__(self, exc_type, exc_val, exc_tb):
            print("class method use {}".format((datetime.datetime.now() - self._start).total_seconds()))

        def __call__(self, *args):
            time.sleep(2)
            return self._func(*args)

    def add(x, y):
        time.sleep(1)
        return x + y

    with TimeIt(add) as add:
        print(add(3, 5))
```
- 将类用作装饰器（将装饰器函数转为类对象）
    - 装饰器类比装饰器函数可以封装更多的数据以及方法
```
    类定义不变
    @TimeIt
    def add(x, y):
        time.sleep(1)
        return x + y

    with add:
        print(add(3,5))
```
- 使用装饰器后，属性信息的修改（这里是用self（实例）来装饰func函数）
```
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
    - 所以：wraps(wrapped)(wrapper)