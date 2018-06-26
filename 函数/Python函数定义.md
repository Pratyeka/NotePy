### 函数
- 定义
    - Python函数是由关键字def、函数名、参数列表、语句组成的语句块
    - 是组织代码的最小单元
- 作用
    - 将某一功能封装起来，方便复用，增强可读性
    - 结构化编程对代码最基本的封装
    - 为了代码美观，减少冗余代码的出现次数
- 基本形式
    ```python
    def fName(args):
        codeblock
        [return m]
    ```
    - 函数名满足标识符的要求，一般使用全小写或者连字符拼接形式
    - return语句省略时，返回None
    - 定义在调用之前，否则会抛出NameError异常
    - 函数是可调用对象

***
### 函数定义(调用)的基本格式
- 函数调用传入的参数与定义的参数个数要保持匹配（不是相等）
    - 因为存在可变参数

#### 普通参数
1. 普通参数定义法：`def fName(x1,x2)`
    - 位置参数调用：`fName(1,2)`
        - 按顺序依次匹配
    - 关键字参数调用：`fName(x1=1, x2=2)`
        - 按照key值来匹配，变量顺序无所谓
    - 混合调用：`fName(2,x2=3)`
        - *要求关键字参数对只能写在位置参数之后*
        - 不能写成`fName(1,x1=3)`，会提示多次给x1赋值
            - x1后的其余变量必须都使用关键字赋值方式，否则会报语法错误

#### 可变参数
- 无法确定参数个数时，处理方式有：
    - 形参为列表、元祖、集合、字典，手动构建实参来完成不确定数目参数输入
    - 使用下面的可变位置参数、可变关键字参数，函数机制自动打包

2. 可变位置参数定义法：`def fName(x1,x2,*xn)`
    - 在形参前使用*表示该形参是可变参数，可以接受多个参数
    - 位置参数调用：`fName(1,2,3,4,5,6,...)`
        - 参数x1、x2必须有实参
        - 按顺序依次匹配x1，x2，再使用剩下的全部参数匹配xn
        - xn在函数的语句块中会构成一个元祖
        - xn可以是0个参数，所以实参个数>=2
        - 可变位置参数后不能出现普通参数，只能是关键字参数
    - 关键字参数调用（错误栗子）
        - `fName(x1=1,x2=3,1,2,3,4,5)`：关键字参数不允许写在简单变量之前
        - `fName(1,2,3,4,5,x1=1,x2=3)`: 会提示x1,x2被多次赋值
```Python
    # 可表位置参数
    def add(*nums):
        sum = 0
        for i in nums:
            sum += i
        print(sum)

    add(2,3,3,5,7,7,8)

    # 接受单个参数（容器）
    def add(nums):
        sum = 0
        for i in nums:
            sum += i
        print(sum)

    add([1,3,3,4,45,6,7])
```

3. 可变关键字参数定义法：`def fName(**kwargs)`
    - 在形参前使用**符号，表示可以接受多个关键字参数
    - 调用：`fName('a'=a, 'b'=b, 'c'=c)`
    - 接受任意多个参数对，构成一个字典
```Python
    def showconfig(**kwargs):
        for k,v in kwargs.items():
            print('{} = {}'.format(k,v))

    showconfig(host='127.0.0.1', port='80', name='xdlb')
```
- 可变参数混用
    - `def func(name, passwd, **kwargs)`
    - `def func(name, *args, **kwargs)`
    - `def func(name, **kwargs, *args)`: 错误,*不能在**后

#### 缺省（默认）参数
- 可以在未传入足够的实参的情况下，对没有给定实参的参数赋值为默认值
- 在参数很多的情况下，减轻函数调用时的负担

4. 缺省参数定义法：`def fName(a,b=3,c=3)`
    - *形参中的缺省参数只能在普通参数后面出现*
    - 调用：`fName(1)`
        - 对a采用位置参数方式匹配，b、c使用默认值
    - 调用：`fName(1,2)`
        - 对a、b采用位置参数方式匹配，c使用默认值
    - 调用：`fName(1,2,3)`
        - 对a、b、c三个参数按顺序依次匹配，b、c覆盖默认值
    - 调用：`fName(1,c=3)`
        - 对a采用位置参数匹配，b采用缺省值参与计算，c关键字参数匹配
    - 调用：`fName(b=1,c=2,a=4)`
        - 使用关键字方式调用，顺序无所谓
```Python
    def login(host='127.0.0.1', port='80', usename='xdlb', passwd='xdliubin'):
        print(host, port, usename, passwd)

    login()
    login('192.168.1.23')
    login(port=8000)
    login(usename='wxr', port=10010)
```

#### KW-ONLY参数
- 在*后或者在可变位置参数后出现的普通参数，就是KW_ONLY参数
- 要求函数调用时必须使用关键字的形式

5. kw-only参数定义：`def fName(*,a,b)`
    - `*`: 没有实际意义，只是为了表明a和b是kw-only参数
    - 无法捕获位置参数了
    - 调用：`fName(a=1,b=3)`  参数顺序无所谓
    - 可以将a、b写成缺省参数形式（调用时不给实参）

6. kw-only参数定义：`def fName(*arg,a,b)`
    - 调用：`fName(1,2,3,3,5,5,a=3,b=4)`
- *注：fName(**kwargs, x, y) 报错*
```Python
    # args截获所有的位置参数、kwargs截获除了name和host外的关键字参数
    def showconfig(*args, name, host, **kwargs):
        print('name={} host={} args={}'.format(name,host,args))
        print('kwargs={}'.format(kwargs))

    showconfig(3,4,5, host='127.0.0.1', name='xdlb', passwd='xdlbin')
```

#### 汇总
7. 定义中参数的顺序一般是：
    - 普通参数、缺省参数、可变位置参数、kw-only参数、可变关键字参数
        - 缺省参数调用时按照位置参数方式使用
    - `fName(x,y,z=3,*args,m=4,n,**kwarga)`

***
### 参数解构
给函数提供参数时，使用参数结构可以将形参列表中的集合拆成单独的元素
- 参数解构
    - `fName(*arg)`表示对除字典外的可迭代对象进行解构
        - 要求形参个数与实参个数相匹配
    - `fName(**arg)` 解构字典对象
        - 要求形参个数与实参个数相匹配
        - 要求实参字典的关键字与形参的关键字匹配