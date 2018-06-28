### 匿名函数
- 匿名：函数没有定义函数名（没有标识符）
- 使用lambda关键字来定义匿名函数
    - 匿名函数形式：`lambda 参数列表：表达式`
    - 参数列表不需要小括号（因为函数名没有标识符，可以分辨是参数）
    - 使用冒号来区分参数列表和表达式
    - 函数调用形式 `(lambda 参数列表：表达式)(实际参数)`
- 不需要使用return来返回值，*表达式的值就是匿名表达式的返回值*
- lambda函数只能写在一行中，不能换行（单行函数）
- 用在高阶的函数传参表达式中
```Python
    print((lambda :0)())            # 形参为0
    print((lambda x,y=4:x+y)(3))    # 形参包含缺省值
    print((lambda x,y:x*y)(3,4))    # 多个形参
    print((lambda x,*,y:x+y)(3,y=3))   # kw-only形参
    print((lambda *args:(x for x in args))(3,4,5,6))   # 可变位置参数

    lst = [x for x in (lambda *args:map(lambda x:x+1, args))(*range(5))] # 高阶函数
    print(lst)
    lst = [x for x in (lambda *args:map(lambda x:(x+1,args), args))(*range(5))]
    print(lst)

```

### 递归函数
- 直接或间接调用自身的函数称为递归函数
- 递归函数一定要有边界条件,并要求一定要执行到递归条件
- 递归函数分为前进块和返回块
    - 不满足边界条件时，递归前进
    - 满足边界条件时，递归返回
- 递归不宜过深，python递归过深抛出RecursionError异常
```Python
    lst = lst1 = 0   # 计数

    def fab(n):
        lst += 1
        if n < 2:       # 边界条件
            return 1
        num = fab(n-1) + fab(n-2)
        return num
    
    def fab1(n):
        lst1 += 1
        return 1 if n < 2 else fab1(n-1)+fab1(n-2)

    print(fab(10), lst)
    print(fab1(10))
```

- 递归VS普通函数 性能对比
    - 普通函数的执行效率高于递归函数
    - 递归太复杂，反复的压栈会导致栈溢出
```Python
    import datetime

    def fab(n):
        return 1 if n < 2 else fab(n-1)+fab(n-2)

    start = datetime.datetime.now()
    print(fab(35))      # 3.381005
    print((datetime.datetime.now()-start).total_seconds())


    lst = [1,1]

    start = datetime.datetime.now()
    for i in range(2, 36):
        lst.append(lst[i-1] + lst[i-2])
    print(lst[-1])     # 0
    print((datetime.datetime.now()-start).total_seconds())
```
- 改进的递归函数（将n作为次数递归）
```Python
    import datatime

    def fab1(n, cur=1, pre=1):
        cur, pre = cur+pre, cur
        if n == 2:
            return cur
        return fab1(n-1, cur, pre)

    start = datetime.datetime.now()
    print(fab1(35))
    print((datetime.datetime.now()-start).total_seconds())
```

- 间接递归
    - 应该要尽力避免间接调用的出现
```Python
    def foo1():
        foo2()

    def foo2():
        foo1()

    foo1()
```

### 递归函数练习
- 求n的阶乘
```Python
    def factorial(n):
        if n == 1:
            return 1
        return n * factorial(n-1)

    print(factorial(5))
```

- 将一个数逆序放入列表中
```Python
    def revlst(num, lst=[]):
        num, b = divmod(num, 10)
        lst.append(b)
        if not num:
            return lst
        return revlst(num, lst)

    print(revlst(1234567))
```

- 猴子吃桃问题
```python
    def monkey(n = 1):
        if n == 10:
            return 1
        return 2 * monkey(n + 1) + 1

    print(monkey())
```

- 将一个字典扁平化
    - 一个无法统一确定循环层级的问题，一般使用递归处理
```Python
    def dict_flat(inputs, name = None, dit={}):
        if not isinstance(inputs, (dict, list, tuple)):
            dit.update({name:inputs})
        else:
            for k,v in inputs.items():
                names = name + '.' + k if name else k
                dict_flat(v, names, dit)

    ori_dict = {'a':{'b':1,'c':2}, 'd':{'e':3,'f':{'g':4}}}
    flat_dict = {}
    dict_flat(ori_dict, None, flat_dict)
    print(flat_dict)    # {'a.b': 1, 'a.c': 2, 'd.e': 3, 'd.f.g': 4}
 
    def flat(dic, name, pdd=[]):
        if isinstance(dic,dict):
            for k,v in dic.items():
                names = name + k + '.'
                flat(v, names, pdd)
        else:
            names = name + str(dic)
            pdd.append(names)

    lst = []
    orid = {'a':{'b':1, 'c':2}, 'd':{'e':3, 'f':{'g':4}}}
    flat(orid, '', lst)
    print(lst)   # ['a.b.1', 'a.c.2', 'd.e.3', 'd.f.g.4']
```