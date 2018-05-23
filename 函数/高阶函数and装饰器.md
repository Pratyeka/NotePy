- 函数也是对象，一个可调用的对象
- 函数可以作为普通变量、参数、返回值
***
### 高阶函数 ###
- 高阶函数需要满足下面任一条件：
    - 接受一个或者多个函数作为输入
    - 输出是函数
- 内建高阶函数
    - `sorted(iterable [,key=None] [,reversed=True])->list`
        - 对输入的可迭代对象进行排序，并返回一个列表
    - `filter(function, iterable)->filter object`
        - 对输入的可迭代对象进行过滤，返回一个生成器对象
        - 生成器可以返回的元素是function为True的元素值
        - `list(filter(lambda x:x%2==0,range(10)))`
    - `map(function, iterable)->map object`
        - 对输入的可迭代对象进行函数映射，返回一个生成器对象
        - 生成器可以返回的元素是function(x)的计算结果

- *注意*：两个函数对象的ID是不一致的，但是两个常量对象的ID是一致的
- 在列表中`==`号是比较两个列表的内容是否一致，因为重构了该符号
- 这里的`==`号比较的是两个对象的ID号，因为函数没有对该符号重构
```
    def counter(base):
        def inc(step=1):
            nonlocal base
            base += step
            return base
        return inc

    f1 = counter(3)
    f2 = counter(3)
    print(f1 == f2)
    print(f1 is f2)
    m = f1()
    n = f2()
    print(f1() == f2())
    print(f1() is f2())
    print(id(m), id(n))
```
is与==区别 (https://zhuanlan.zhihu.com/p/35219174)

***
### 函数柯里化 ###
- 柯里化的定义：
    - 接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数，并且返回接受余下的参数同时返回结果的新函数的技术
- 柯里化的作用：
    - 提高代码的复用性与灵活性，同时降低代码的耦合性
    - 构建出单值输入单值输出的函数形式

***
### 函数装饰器 ###
- 无参装饰器
    1. 它是一个函数
    2. 函数作为它的形参
    3. 返回值也是一个函数
    4. 可以使用@functionname方式，简化调用
- 带参装饰器
    1. 可以看做是在无参装饰器外增加了一层函数

- 函数装饰器的特性：
    - *能把被装饰的函数替换成其他函数*（柯里化让表达式的修饰意义更明显）
    - 装饰前后的函数接受相同的参数
    - 并且可以返回被装饰函数本该返回的值，以及额外操作
    - 装饰器在*加载模块时*立即执行（函数定义之后立即执行）
        - 导入时 和 运行时的区别
    - 由于装饰函数函数的返回值与原函数一致，嵌套装饰时（按未装饰的函数写装饰函数即可）
```
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
    - 看到装饰器在模块记载时就执行
    - 可以分析双层嵌套时函数的调用顺序

- 函数装饰器的作用：
    - 可以保证函数的代码不做任何改动的前提下，为函数增添功能
    - 可以用来保护业务代码不受影响
- 函数装饰器的用途：
    - 给已经存在的对象添加额外的功能

- 函数装饰器的问题：
    - 被装饰后，函数的元数据就被装饰函数替换会带来问题
- 问题解决：
    - 1. 使用`wraps(wrapped)`装饰器，完成对函数元数据的更新替换
    - 2. 使用`update_wrapper(wrapper, wrapped)`函数，更新元数据
    - 3. *[方法1与方法2比较](https://github.com/Pratyeka/NotePy/blob/master/%E5%B8%B8%E7%94%A8%E6%A8%A1%E5%9D%97.md)*

```
    from functools import wraps, update_wrapper
    import time

    def logger(fn):
        print("logger")
        # @wraps(fn)
        def _add(*args, **kwargs):
            print("Before the {} call".format(fn.__name__))
            ret = fn(*args, **kwargs)
            print("After the {} call".format(fn.__name__))
            return ret
        update_wrapper(_add, fn)
        return _add

    if __name__ == "__main__":
        @logger
        def add(x:int, y:int)->int:
            """ This is the function add """
            time.sleep(3)
            return x + y

        print(add.__doc__)  # 发现装饰器函数修改了函数的字符串文档
```
    