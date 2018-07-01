***
### 文件IO基本操作
- 文件打开： `fileObj = open(file, mode='r', buffering=None, encoding=None, errors=None, newline=None, closefd=True)`
    - 打开一个文件，返回一个文件对象（流对象）和文件描述符。打开文件失败，返回异常
    
    - 文件的打开模式（model参数）：
        - `r`: 只读模式打开（默认打开模式）
            - 使用write方法，会抛出异常
            - 如果文件不存在，会抛出FileNotFoundError异常
        - `w`: 只写模式打开
            - 使用read方法，会抛出异常
            - 如果文件不存在，会直接新建文件
            - 如果文件存在，会清空文件内容
        - `x`: 创建模式打开
            - 文件不存在，新建文件，并以只写方式打开
            - 文件存在，抛出FileExistsError异常
        - `a`: 追加模式打开
            - 文件存在，只写模式打开，追加内容
            - 文件不存在，则创建后，只写打开，追加内容
        - `+`: 为上面四种打开模式提供缺失的读写功能
            - 获取的文件依旧按照r.w.a.x的特征（使用缺失功能时不报错而已）
            - 不能单独使用，只是给r.w.a.x做功能增强

    - 文本模式（model参数）：
        - `t`: 将文件的字节按照指定的字符编码理解，按照字符操作
            - 以文本模式处理（默认），主要编码格式（gbk:两字节，utf-8:三字节）
        - `b`: 将文件按照字节理解，与字符编码无关
            - 二进制模式操作，字节使用bytes类型

    - 缓冲区（buffering参数）：
        - 二进制模式：默认的缓冲大小是4096和8192
            - `buffering=0`: 表示关闭缓冲区
            - `buffering>0`: 表示缓冲区大小
        - 文本模式：默认是行缓冲方式
            - 一般使用默认缓冲区大小
            - `buffering=0`: 不支持
            - `buffering>0`: 行缓冲，遇到换行符才flush
        - 一般使用默认缓冲区设置，在需要写入磁盘时，手动flush一次
    
    - newline参数：(只在文本模式中使用)
        - `None`: 在读文件时，表示将'/n','/r','/r/n'转换成'/n'
            - 在写文件时，表示将'/n'转换成系统设定的分隔符`os.linesep`
        - `''`: 在读文件时，不会自动装换换行符
            - 在写文件时，不会自动转换 （'/n'也不会自动换换行符）
        - `char`: 在读文件时，指定的char字符作为分隔符号，不转换行符
            - 在写文件时，会将之对应的char字符转换成'/n'分隔符

- 文件关闭： `fileObj.close()`
    - flush并关闭文件对象
    - 与open配对出现，文件已经关闭，再次关闭没有任何效果
    - 可以使用with语法来自动关闭文件
    - `closed`: 关闭文件描述符，True表示文件已经关闭

***
### 文件指针：指向当前字节位置
- 指针位置:
    - `mode=r`: 指针开始在0位置处
    - `mode=a`: 指针开始在EOF位置处
- 指针操作： `fileObj.seek(int [,when])`
    - `when`: 有三种模式0、1、2
        - `0`: 表示从文件开始，偏移int个*字节*（int只能是整数，可以>字符数）
        - `1`: 表示从当前位置开始偏移
            - 文本模式下：int只能是0，相当于原地不动
            - 二进制模式下：int可正可负（不能超出文件头指针，否则抛异常）
        - `2`: 表示从EOF位置开始偏移
            - 文本模式下：int只能是0，相当于指针移动到文件末尾
            - 二进制模式下：int可正可负（不能超出文件头指针，否则抛异常）
- 指针位置：`fileObj.tell()`：显示当前文件指针

***
### 文件读写操作
- 文件读取：`fileObj.read([size=-1])`
    - `size`: 具体数值表示读入size个*字节/字符*
        - *负数或者省略不写*表示从当前指针到文件末尾EOF
- 行读取：`fileObj.readline([size=-1])`
    - 默认读取一行内容
        - size参数表示一次能读入的字符/字节数量
- 多行读取：`fileObj.readlines([hint=-1])`
    - 读取文件的所有行, 返回一个列表
        - hint参数表示返回指定的行数
- 文件写入：`fileObj.write(str/bytes)`
    - 文本模式: 输入字符串，写完后指针在文件的EOF处，返回字符/字节数量
    - bytes模式: 输入bytes文件，写完后指针在文件的EOF处
        - 英文字符：b'abdjadj'    /     'afadfa'.encode()
        - 中文字符：'交罚款了多少'.encode()
            - encode: 默认是`utf-8`编码格式
- 字符串列表写入：`writelines(lines)`
    - 将字符串列表写入文件
    - 列表元素是字符/字节串

***
### 文件权限查询 ###
- `seekable()`: 文件指针是否可以移动
- `readable()`: 文件是否可读
- `writable()`: 文件是否可写

***
### 上下文管理
- 一种特殊的语法，交给解释器来释放文件对象
- 上下文管理：
    - 使用with...as关键字管理文件的打开关闭
    - *上下文管理的语句块不会开启新的作用域*
    - with语句块执行完的时候，会自动关闭文件对象

***
### 栗子栗子
- 实现文件内容的拷贝
```python
    def copy(source, dest, encoding = 'utf-8'):
        with open(source, encoding=encoding) as f:
            file = f.read()
        with open(dest, mode='w+', encoding=encoding) as f:
            f.write(file)

    if __name__ == '__main__':
        file_ori = 'doc/sample.txt'
        file_cpy = 'doc/sampla.txt'
        copy(file_ori, file_cpy, 'utf-8')

    with open(file_cpy,encoding='utf-8') as f:
        print(f.read())
```

- 统计一个文件中单词的数量，并输出排名前十的单词统计数据
```python
    CHARS = set(""",.[]()-+"'/\*&%#$@` \t\n\r""")

    def make_key(line):
        start = 0
        line = line.lower()
        for i, char in enumerate(line):
            if char in CHARS:
                if start != i:
                    yield line[start:i]
                start = i + 1

    def word_count(file, encoding):
        count = {}
        with open(file, mode='r', encoding=encoding) as f:
            for line in f:
                for name in make_key(line):
                    count[name] = count.get(name, 0) + 1
        return count

    def topn(count, num):
        lst = sorted(count.items(), key=lambda x:x[1], reverse=True)
        for i in range(num):
            print(lst[i])

    file_ori = 'doc/sample.txt'
    count_dict = word_count(file_ori, 'utf-8')
    topn(count_dict, 10)
```
