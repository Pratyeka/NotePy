### 正则表达式 ###
- 用途：对字符串按照某种规则进行检索、替换

##### 基本语法 ####
- 元字符：
    - `.`: 可以用来表示除了换行符之外的任一字符
    - `[abc]`: 字符集，可以用来表示字符集中的任一字符
    - `[^abc]`: 字符集，表示除了字符集外的任一字符
    - `[a-z]`: 字符范围，表示a-z范围内的任一字符
    - `[^a-z]`: 字符范围，表示除了a-z之外的任一字符
    - `\d`: 表示0-9范围内的任一数字，==[0-9]
    - `\D`: 表示除了0-9之外的任一字符，==[^0-9]
    - `\s`: 表示所有的空白字符
    - `\S`: 表示除了空白字符外的任一字符
    - `\w`: ==[a-zA-Z0-9_]
    - `\W`: ==[^a-zA-Z0-9_]
    - `\b`: 表示匹配单词的边界
    - `\B`: 表示不匹配单词的边界
    - `^`: 表示字符串的行首
    - `$`: 表示字符串的行尾
- ***注意：如果在正则表达式中有特殊意义，在使用时需要使用转义符***

- 字符重复
    - `*`: 表示前面的字符可以重复至少0次
    - `+`: 表示前面的字符可以重复至少1次
    - `?`: 表示前面的字符可以重复0次或者1次
    - `{n}`: 表示前面的字符重复n次
    - `{n,}`: 表示前面的字符重复至少n次
    - `{m,n}`: 前面的字符重复次数区间为：[m,n]
- ***注意：字符的重复是指模式的重复***

- 分组与断言
    - `x|y`: 表示匹配x或者y
    - `(pattern)`: 表示一个分组，自动按顺序给编号
    - `\num`: 表示引用标号为num的分组（引用内容不是模式）
    - `(?:pattern)`: 取消分组,表示括号用于指定优先级
    - `(?<name>pattern)`: 命名分组，python中使用`(?P<name>pattern)`
    - `f(?=pattern)`: 表示在f后一定有pattern
    - `(?<=pattern)f`: 表示在f前一定有pattern
    - `f(?!pattern)`: 表示在f后一定没有pattern字符
    - `(?<!pattern)f`: 表示在f前一定没有pattern字符
- ***注意：断言不是分组，不占用分组号，相当于一个判定条件***

- 引擎选项
    - `re.I`: 表示忽略大小写
    - `re.M`: 表示多行模式
    - `re.S`: 表示单行模式
    - `re.X`: 表示忽略表达式中的空白符
- ***注意：单行模式时，'.'符号可以匹配所有字符（包括换行符）***

- 栗子
```
    匹配1-3位的数字：^([1-9]\d\d?|\d)\D
    匹配IP地址：^(?:\d{1,3}\.){3}(?:\d{1,3})，需要Py的进一步判定
```

### Python正则模块re
- 常量
    - 多行模式：`re.M`
    - 单行模式：`re.S`
    - 忽略大小写：`re.I`
    - 忽略表达式中的空白字符：`re.X`
- 注：使用`|`运算符开启多种选项

#### 方法
- 编译：`re.complie(pattern, flags=0)`
    - pattern就是正则表达式字符串，flags是编译模式
    - 返回一个regex
    - 正则表达式需要被编译，编译后的结果保存使用，可以提高效率
        - re的其他方法都调用了编译方法，为了提速
- 单次匹配
    - match
        - 可用方法：
            - `re.match(pattern, string, flags=0)`
            - `regex.match(string [,pos [,endpos]])`
        - match匹配从字符串的开头匹配，首字母不一样直接返回None
        - 完成匹配时返回Match对象
    - search
        - 可用方法：
            - `re.search(pattern, string, flags=0)`
            - `regex.search(string [,pos [,endpos]])`
        - 在string中搜索到第一个匹配的模式
        - 返回match对象
    - fullmatch
        - 可用方法：
            - `re.fullmatch(pattern, string, flags=0)`
            - `regex.fullmatch(string [,pos [,endpos]])`
        - 要求整个字符串和正则表达式匹配
        - 返回match对象
- 全文搜索
    - findall
        - 可用方法：
            - `re.findall(pattern, string, flags=0)`
            - `regex.findall(string [,pos [,endpos]])`
        - 对整个字符串从左到右匹配
        - 返回所有匹配项的列表（列表元素是字符串）
    - finditer
        - 可用方法
            - `re.finditer(pattern, string, flags=0)`
            - `regex.finditer(sting [,pos [,endpos]])`
        - 对整个字符串从左到右匹配
        - 返回迭代器，每次迭代返回的是match对象
- 匹配替换
    - sub
        - 可用方法：
            - `re.sub(pattern, replacement, string, count=0, flags=0)`
            - `regex.sub(replacement, string, count=0)`
        - 使用pattern对字符串string进行匹配，对匹配项使用repl替换
        - 返回替换后的字符串
        - count指定提换的次数
        - replacement可以是string，bytes，functions
    - subn
        - 可用方法：
            - `re.subn(pattern, replacement, string, count=0, flags=0)`
            - `regex.subn(replacement, string, count=0)`
        - 同sub返回一个元祖：（new_string, number_of_subs_made）
        - count指定替换的次数
- 分割字符串
    - `注：字符串的分割函数，不能指定多个字符进行分割`
    - split
        - 可用方法：
            - `re.split(pattern, string, maxsplit=0, flags=0)`
    - 可以一次指定多个分割字符
    - 返回值为列表
- 分组
    - 使用小括号的pattern捕获的数据被放到了组group中
    - 如果pattern中使用了分组，如果有匹配的结果，会在match对象中
        1. 使用group(N)方式返回对应的分组   
            - 1-N是对应的分组，0返回匹配到的整个分组
        2. 如果使用了命名分组，可以使用group('name')的方式取分组
        3. 也可以使用groups()返回所有分组
        4. 使用groupdict()返回所有命名分组
```python
    import re

    s = '''bottle\nbag\nbig\napple'''
    regex = re.compile('(b\w+)')
    result = regex.match(s)
    print(1, 'match', result.groups())

    result = regex.search(s, 1)
    print(2, 'search', result.groups())

    regex = re.compile('(b\w+)\n(?P<name2>b\w+)\n(?P<name3>b\w+)')
    result = regex.match(s)
    print(3, 'match', result, type(result.groupdict()))
    print(4, result.group(3), result.group(2), result.group(1))
    print(5, result.group(0).encode())
    print(6, result.group('name2'))
    print(7, result.groups())
    print(8, result.groupdict())
```
- 字符串信息提取，并做格式化处理
```Python
    from datetime import datetime
    import re

    message = '''183.60.212.153 - - [19/Feb/2013:10:23:29 +0800] \
    "GET /o2o/media.html?menu=3 HTTP/1.1" 200 16691 "-" \
    "Mozilla/5.0 (compatible; EasouSpider; +http://www.easou.com/search/spider.html)"'''

    pattern = '(?P<remote>[\d.]{7,}) - - \[(?P<datetime>[/\w:+ ]+)\]\s+' \
            '"(?P<method>\w+) (?P<url>\S+) (?P<protocol>[\w/\.]+)"\s+' \
            '(?P<status>\d+) (?P<length>\d+) "-"\s+"(?P<useragent>.+)"'
    regex = re.compile(pattern)
    time_conv = lambda str_time: datetime.strptime(str_time, '%d/%b/%Y:%H:%M:%S %z')
    ops = {'datetime':time_conv, 'status':int, 'length':int}

    matcher = regex.match(message)
    if matcher:
        resdic = {k:ops.get(k,lambda x:x)(v) for k,v in matcher.groupdict().items()}
        print(resdic)
    else:
        raise Exception("NO MATCH")
```