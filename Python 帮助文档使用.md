### 帮助查询方式 ###
- ipython方式查询
    - `help(object)`
    - `object?`   ：查看简略信息
    - `object??`  : 查看更详细的信息
- 官方帮助文档 

### 帮助命令格式解析 ###
- `L.sort() -> None`: `->`表示返回值类型为None,就地修改方式
- `len(obj)`: 参数是一个对象 s(seqence序列类型/collection容器类型)
- `[object]`: 表示object是可以省略的
- `map(func, *iterables) -> map object`: 可以接受多个可迭代对象
    - 返回是一个map对象
- `print(*objects, sep=' ', end='\n')` : 
    - `*obj` : 接受多个对象
    - 因为可以接受多个输入，所以后面的关键字必须指定名称
- `str.format(*args, **kwargs)`:
    - 使用`*args`和`**kwargs`传递可变长参数
    - `*args`：传递多个无名参数，它是一个tuple
    - `**kwargs`：传递键值可变长参数列表，它是一个dict
    - 同时使用`*args`和`**kwargs`时，`*args`参数列要在`**kwargs`前
- `set.union(*other)` :
    - `*other` ：表示多个set形式的输入