### 日志级别
- 日志级别表示产生日志时间的严重程度
- *设置日志级别后，只有在事件严重程度高于日志级别*，才会被打印输出
- 日志级别名称、数值以及对应的事件发生方法
    - CRITICAL: 50 ------------- critical()
    - ERROR: 40 ---------------  error()
    - WARNING: 30(默认级别) ----  warning()
    - INFO: 20 ----------------  info()
    - DEBUT: 10 ---------------  debug()
    - NOTSET: 0 ---------------  notset()

### logging模块的基本使用
- 日志字符串使用
    ```Python
    import logging

    FORMAT = '%(asctime)s %(thread)s %(levelname)s %(lineno)s %(message)s'
    logging.basicConfig(format=FORMAT, level=logging.INFO) # 其实是在设置跟logger的信息

    logging.info('testing {}'.format('abd'))
    ```
- 字符串字段
    - `%(message)s`: *打印的日志消息内容*
    - `%(asctime)s`: 创建日志记录时的可读时间
        - 默认格式为: `2018-06-13 11:21:20,857`
        - 日期格式修改：`logging.basicConfig(datefmt='%Y%m%d-%H%M%S')`
    - `%(funcName)s`: 日志记录所在的函数名
    - `%(levelname)s`: 日志级别的名称（字符形式）
    - `%(levelno)s`: 日志级别对应的数值（数值形式）
    - `%(lineno)d`: 日志调用在源代码中的行号
    - `%(module)s`: 日志所在的模块名
    - `%(process)d`: 进程ID号
    - `%(thread)d`: 线程ID号
    - `%(threadName)s`: 线程名
    - `%(processName)s`: 进程名
    - `%(school)s`: 日志消息扩展

- 日志消息扩展
```Python
    import logging

    FORMAT = '%(asctime)s %(thread)s %(levelname)s %(lineno)s ' \
            '%(message)s %(school)s'
    logging.basicConfig(format=FORMAT, level=logging.INFO)

    d = {'school': 'a u ok'}        # 要求扩展的字段是一个字典
    logging.info('testing', extra = d)
    # 2018-06-13 11:59:57,046 11096 INFO 66 testing a u ok
```

- 日志输出到文件
```Python
    import logging

    FORMAT = '%(asctime)s %(thread)s %(levelname)s %(lineno)s ' \
            '%(message)s %(school)s'
    logging.basicConfig(format=FORMAT, filename='doc/test.log')

    d = {'school': 'a u ok'}
    for _ in range(5):
        logging.warning('testing', extra = d)
```

### logging类
- root logger实例
    - 在模块加载时, 会自动创建root logger, 默认日志级别是warning
    - 使用basicConfig是对root logger做设置

- loggger实例构造
    - `getlogger([name])`: 获取logger实例对象
        - 如果不设置name，返回的是root logger对象
        - 返回的logger实例是按照name区分的，name相同为同一个logger对象
```Python
    import logging

    mlog = logging.getLogger('mlog')
    print(mlog, id(mlog))   # <Logger mlog (WARNING)> 42370888

    xlog = logging.getLogger('mlog')
    print(xlog, id(xlog))  # <Logger mlog (WARNING)> 42370888

    x = logging.getLogger()
    print(x)               # <RootLogger root (WARNING)>
```

- logger层次结构
    - logger之间的层级结构按照`.`来分割
    - root logger的parent为None

```Python
    import logging

    x = logging.getLogger()
    print(x, x.parent, id(x), x.name)

    mlog = logging.getLogger('mlog')
    print(mlog, mlog.parent, id(mlog), mlog.name)

    xlog = logging.getLogger('mlog.child')
    print(xlog, xlog.parent, id(xlog), xlog.name)
```

- logger级别设置
    - *控制是否有日志输出是按照当前logger的有效级别*
    - logger在创建时，都有等效的level（默认为0），没什么用的一个参数
        - 如果设置了level，就用该日志级别比较
        - 如果没有使用setlevel来设置level，按照父类的level控制日志打印（对应的是有效级别）
    - logger可以使用`setlevel(num)`来设置自身的日志级别
    - *注：level属性的继承是一条线*
    - *注：geteffEctiveLevel的level是一条继承线*
```Python
    import logging

    FORMAT = '%(asctime)s %(funcName)s %(lineno)s %(message)s'
    logging.basicConfig(format=FORMAT, level=logging.ERROR)

    x = logging.getLogger()
    print(x, x.parent, id(x), x.name, x.level)

    # 新建logger的level是0，有效level是父类的level
    mlog = logging.getLogger('mlog')
    print(mlog, mlog.parent, id(mlog), mlog.name, mlog.level)
    print('truth level: {}'.format(mlog.getEffectiveLevel()))
    # 设置level后，level以及有效level都是设置的level
    mlog.setLevel(39)
    print(mlog, mlog.parent, id(mlog), mlog.name, mlog.level)

    xlog = logging.getLogger('mlog.child')
    print(xlog, xlog.parent, id(xlog), xlog.name, xlog.level)
    print('truth level: {}'.format(xlog.getEffectiveLevel()))
```

- Handler
    - 根rooter至少有一个handler：sys.stderr
    - Handler的初始level是0
    - 控制日志输出去向的真正部件
        - 可以单独设置级别
        - 可以单独设置过滤
        - 可以单独设置格式
    - 包括两种处理类
        - `FileHandler`: 文件处理函数
        - `StreamHandler`：字符流处理函数
    - 在basicConfig根据是否提供文件名(Filename)，采取两种操作模式
        - 有文件名时，使用FileHandler处理
        - 没有文件名是，使用StreamHandler处理，默认到标准输出
    - 使用如下语句给logger实例添加Handler
        - handler没有设置格式，输出信息中只留最后的消息，没有时间戳等信息
        - `handler = logging.FileHandler('doc/mlog.log', 'w')`
        - `mlog.addHandler(handler)`

```Python
    import logging

    FORMAT = '%(asctime)s %(funcName)s %(lineno)s %(message)s'
    logging.basicConfig(format=FORMAT, level=logging.INFO)

    x = logging.getLogger()
    print(x, x.parent, id(x), x.name, x.level)

    handler = logging.FileHandler('doc/mlog.log', 'w')
    mlog = logging.getLogger('mlog')
    mlog.addHandler(handler)
    mlog.info('test the handler')   # 为什么文件、终端都有输出?
    # 因为日志传递给文件，还有流转给父类来处理（无视父类的级别）
    # mlog.propagate = False # 关闭传递
```

- 消息传递流程
    - 参考官方日志传递图

- Formatter类
    - 设置日志字符串格式，未设置使用%(message)s
    - *注：是在handler对象上设置添加*
```Python
    import logging

    mlog = logging.getLogger('mlog')
    handler = logging.FileHandler('doc/flog.log', 'w')
    FORMAT = logging.Formatter('%(asctime)s %(funcName)s %(lineno)s %(message)s')
    handler.setLevel(23) # 给handler单独设置level
    handler.setFormatter(FORMAT) # 给handler单独设置字符串格式，不设置默认为message
    mlog.addHandler(handler)  # 给当前logger添加对应的handler

    mlog.error('test the handler')   # 为什么文件中、终端都有输出?
```
- Filter类
    - 在Handler上添加过滤器，只会影响某一个handler
    - 在logger上添加，会影响处理流程
    - *根据logger的名字来过滤*：`record.name.find(self.name, 0, self.nlen) != 0`
    