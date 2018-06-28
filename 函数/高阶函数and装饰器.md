***
- 函数也是对象，一个可调用的对象
- 函数可以作为普通变量、参数、返回值
***
### 高阶函数
- 高阶函数需要满足下面任一条件：
    - 接受一个或者多个函数作为输入
    - 输出是函数
- 内建高阶函数
    - `sorted(iterable [,key=None] [,reversed=True])->list`
        - 对输入的可迭代对象进行排序，并返回一个列表
        - key：表示对可迭代对象中元素的处理
        - rev：表示是否按照默认的升序排列
        - `list.sort(key, reversed)`: 就地修改
        - `lst = sorted(random.sample(range(1000), 1000), key x:5-x, reversed=True)`
    - `filter(function, iterable)->filter object`
        - 对输入的可迭代对象进行过滤，返回一个*迭代器*
        - 生成器可以返回的元素是function为True的元素值
        - `list(filter(lambda x:x%2==0,range(10)))`
    - `map(function, *iterable)->map object`
        - 对输入的可迭代对象进行函数映射，返回一个生成器对象
        - 生成器可以返回的元素是function(x)的计算结果
        - 可以接受多个可迭代对象，要求可迭代对象数目与lambda的输入参数个数一致
- map函数栗子
```Python
    # 接受一个可迭代对象
    mapp = map(lambda x:str(x)+'-', random.sample(range(1000), 10))
    print(type(mapp), tuple(mapp))

    # 接受多个可迭代对象（要求lambda的参数是多个）
    mapp1 = map(lambda x,y:(x,y),
                random.sample(range(1000), 20),
                random.sample(range(1000), 20))
    print(type(mapp1), set(mapp1))
```
- 实现排序函数
```Python
    import random

    def sor(itt, key=lambda x:divmod(x,5)[1], reverse=False):
        lengt = len(itt)
        for i in range(lengt - 1):
            count = 0
            for j in range(lengt - i - 1):
                if key(itt[j]) > key(itt[j+1]):
                    itt[j], itt[j+1] = itt[j+1], itt[j]
                    count += 1
            if not count:
                print(itt[::-1]) if reverse else print(itt)
                return itt

    sor(random.sample(range(1000), 20))
```

- *注意*：两个函数对象的ID是不一致的，但是两个常量对象的ID是一致的
- 在列表中`==`号是比较两个列表的内容是否一致，因为重构了该符号
- 这里的`==`号比较的是两个对象的ID号，因为函数没有对该符号重构
```python
    def counter(base):
        def inc(step=1):
            nonlocal base
            base += step
            return base
        return inc

    f1 = counter(3)
    f2 = counter(3)
    print(id(f1), id(f2))    # 两个局部变量肯定不一致，要是一致的话不就乱套了吗
    print(f1 == f2)     # False
    print(f1 is f2)     # False
    m = f1()
    n = f2()
    print(f1() == f2())  # True
    print(f1() is f2())  # True
    print(id(m), id(n))
```
is与==区别 (https://zhuanlan.zhihu.com/p/35219174)

***
### 函数柯里化
- 柯里化的定义：
    - 将接受两个参数的函数编程接受一个参数的函数，并且返回接受余下参数的函数
    - z = f(x,y) ---->  z = f(x)(y)
        - 通过嵌套函数就可以把函数转换成柯里化函数
- 柯里化的作用：
    - 提高代码的复用性与灵活性，同时降低代码的耦合性
    - 构建出单值输入单值输出的函数形式
```Python
    def add(x,y):
        print(x+y)

    def add1(x):
        def add2(y):
            print(x+y)
        return add2

    add(2,3)
    add1(2)(3)
```

***
### 函数装饰器
- 无参装饰器
    1. 它是一个函数
    2. 函数作为它的形参
    3. 返回值也是一个函数
    4. 可以使用@functionname方式，简化调用
        - func = functionname(func): 在调用func的时候已经是装饰后的内层函数
    5. 装饰器是高阶函数，是对传入函数的装饰（功能增强）
- 带参装饰器
    1. 它是一个函数
    2. 函数作为他的形参
    3. 返回值是一个装饰器函数
    4. 可以使用@functionname(param list)方式调用
    5. 可以看做是在无参装饰器外增加了一层函数

- 函数装饰器的特性：
    - *能把被装饰的函数替换成其他函数*（柯里化让表达式的修饰意义更明显）
    - 装饰前后的函数接受相同的参数
    - 并且可以返回被装饰函数本该返回的值，以及额外操作
    - 装饰器在*加载模块时*立即执行（函数定义之后立即执行）
        - 导入时 和 运行时的区别
    - 由于装饰函数函数的返回值与原函数一致，嵌套装饰时（按未装饰的函数写装饰函数即可）
```python
    from datetime import datetime
    import time

    def logger(fn):
        print("logger")
        def _add(*args, **kwargs):
            print("Before the {} call".format(fn.__name__))
            ret = fn(*args, **kwargs)
            print("After the {} call".format(fn.__name__))
            return ret
        return _add

    def time_logger(threshold, key = lambda x:print(x)): # 功能抽象
        print("time_logger")
        def _time_logger(fn):
            print("_time_logger")
            def _sadd(*args, **kwargs):
                print("Before the {} call".format(fn.__name__))
                start = datetime.now()
                ret = fn(*args, **kwargs)
                time_use = (datetime.now() - start).total_seconds()
                if time_use > threshold:
                    key(time_use)
                print("After the {} call".format(fn.__name__))
                return ret
            return _sadd
        return _time_logger

    if __name__ == "__main__":
        @time_logger(3)
        @logger
        def add(x:int, y:int)->int:
            """ This is the function add """
            time.sleep(3)
            return x + y
    # 输出结果
    # time_logger
    # logger
    # _time_logger
```
- *注：双层嵌套栗子*
    - 看到装饰器函数在模块加载（函数对象生成时执行）时就执行
    - 可以分析双层嵌套时函数的调用顺序

- 函数装饰器的作用：
    - 可以保证函数的代码不做任何改动的前提下，为函数增添功能
    - 可以用来保护业务代码不受影响
- 函数装饰器的用途：
    - 给已经存在的对象添加额外的功能

- 函数装饰器的问题：
    - 被装饰后，函数的元数据就被装饰函数替换会带来问题
- 问题解决：
    - 1. 使用`wraps(wrapped, assigned=WRAPPER_ASSIGNMENTS,updated=WRAPPER_UPDATES)`装饰器，完成对函数元数据的更新替换
        - wrapped：被替换的函数
        - assigned：被覆盖的属性
        - updated：被更新的属性
        - 增加了一个__wrapped__属性，记录wrapped函数
    - 2. 使用`update_wrapper(wrapper, wrapped，assigned=WRAPPER_ASSIGNMENTS,updated=WRAPPER_UPDATES)`函数，更新元数据
        - wrapper：替换后的函数
        - wrapped：被替换的函数
        - assigned：被覆盖的属性
        - updated：被更新的属性
        - 增加了一个__wrapped__属性，记录wrapped函数
        - *返回值是warpper函数*
    - 3. *[方法1与方法2比较](https://github.com/Pratyeka/NotePy/blob/master/%E5%B8%B8%E7%94%A8%E6%A8%A1%E5%9D%97.md)*

```python
    import datetime
    import functools

    def time_use(func):
        """
        This is TIME_USE
        :param func:
        :return:
        """
        def inc(*args, **kwargs):
            """
            THIS  IS TIME_USE INC
            :param args:
            :param kwargs:
            :return:
            """
            start = datetime.datetime.now()
            print('start time:{}'.format(start))
            ret = func(*args, **kwargs)
            delta = (datetime.datetime.now() - start).total_seconds()
            print('Time Use:{}'.format(delta))
            return ret
        return functools.update_wrapper(inc, func)

    def logger(func):
        """
        THIS IS LOGGER
        :param func:
        :return:
        """
        def inc(*args):
            """
            THIS IS LOGGER INC
            :param args:
            :return:
            """
            print('learn the wrapper')
            return func(*args)
        return functools.update_wrapper(inc,func)

    @time_use
    @logger
    def add(x,y):
        """
        This is ADD
        :param x: int x
        :param y: int y
        :return: int x+y
        """
        return x+y

    print(add(3,5))
    print(add.__doc__)
    print(add.__name__)
```