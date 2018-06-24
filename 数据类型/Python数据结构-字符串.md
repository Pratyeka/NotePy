### 字符串的基本特点 ###
- 不可变的可迭代可索引对象
- 是由一个个字符组成的有序的序列，是字符的集合

### 字符串的初始化 ###
- `sName = 'string'`
- `sName = "string"`
- `sName = ''' string '''`   （多行字符串，可以用来作为多行注释）
- `sName = str(object)`：使用str()转换为字符串类型
  - 将object直接转换成字符串类型而不是采用可迭代的方式
- `sName = ''.join(iterable)`：使用可迭代对象新建字符串（可迭代对象的元素都是字符串类型）
- 在字符串内可以使用转义字符

### 字符串的访问 ###
- `str[index]`：支持索引访问，注意空白符的占位
- 有序的字符集合，可以顺序访问，返回的每一个元素也是字符串类型
- 可迭代对象 ： `lst = list(string)`
  **可迭代不一定可索引**

### 字符串的常用方法 ###
- 字符串join连接
  - `"string".join(iterable)`：将一个可迭代对象中的所有元素使用string连接起来
    -  返回值是一个新字符串
    - 要求可迭代对象的元素都是字符串类型
- 字符串拼接
  - `str1 + str2` ：拼接两个字符串
    - 返回一个新的字符串
- 字符串分割 (split 系列)
  - `str.split(sep='' , maxsplit=-1)`：将字符串按指定分隔符分割
    - 返回为列表
    - sep 指定分隔符，默认是使用 **空白字符串(可以是多个空白符一起)** 作为分隔符
    - 可以指定最大分割次数（默认是全部分割）
    - 方向是从左到右
    - 在分隔符前没有内容：会返回一个空字符串
  - `str.rsplit(sep='' , maxsplit=-1)`：将字符串按指定分隔符分割
    - 返回为列表
    - sep 指定分隔符，默认是使用 **空白字符串（可以是多个空白符一起）** 作为分隔符
    - 可以指定最大分割次数（默认是全部分割）
    - 从右到左的顺序
  - `str.splitlines([keepends])`：按照行来分割字符串
    - 返回为列表
    - keepends 参数（bool类型）表示是否保留行分隔符
    - 行分隔符包括：`\r、\n、\r\n`
- 字符串分割 （partition系列）
  - `str.partition(sep)`：按照sep指定的分隔符将str分割成两部分
    - 返回的是一个元祖类型
    - 必须指定分隔符，并且不能为空字符串
    - 遇到第一个分隔符，就将字符串分割为 头、分隔符、尾构成的三元组，后面的忽略
    - 没有遇到分隔符，返回由字符串、空白、空白构成的三元组
    - 从左到右的顺序
  - `str.rpartition(sep)`：同上，反向而已
```Python
  slst = 'judst do it'.split('d', maxsplit=1)
  print(slst)
  slst = 'just do it'.partition('do')
  print(slst)
  slist = 'just do it\n just\n do it'.splitlines(keepends=True)
  print(slist)
```

- 字符串大小写
  - `str.upper()` ：小写字符转大写
  - `str.lower()` ：大写字符转小写
  - `str.swapcase()`  ：大小写互相转换
- 字符串排版
  - `str.title()`  ： 单词的首字母大写
    - 返回一个新的字符串，不改变str的字符
  - `str.capitalize()` ：只有行首单词的首字母大写
    - 返回一个新的字符串，不改变str的字符
  - `str.center(width [,fillchar])`：字符串填充
    - 返回一个新的字符串，不改变str的字符
    - 返回字符串的宽度为width，不够的使用fillchar来填充
  - `zfill(width)`：右对齐，不够使用0填充
    - 返回新字符串
  - `ljust(width [,fillchar])`：左对齐，使用fillchar字符串来填充
    - 返回新字符串
  - `rjust(width [,fillchar])`：右对齐，使用fillchar字符串来填充
    - 返回新字符串

```Python
  ustr = 'just do it'.upper()
  print(ustr)
  lstr = 'Just Do IT'.lower()
  print(lstr)
  sstr = 'Just Do IT'.swapcase()
  print(sstr)
  wstr = 'just do it'.center(33,"*")
  print(wstr)
  zstr = 'just do it'.zfill(20)
  print(zstr)
  rstr = 'just do it'.replace('j', 'r')
  print(rstr)
  sstrip = 'just do it'.strip('ti')
  print(sstrip)
```

***
### 字符串修改 ###
  - `str.replace(old, new [, count])` ： 使用新的字符串替换旧字符串
    - 返回新的字符串，不改变str中的字符
    - 可以指定替换的次数，默认全部替换
  - `str.strip([chars])`：根据指定字符集删除字符串两端的字符
    - 不指定字符集时，默认为*空白字符集合*
    - 从两端删除时，尽可能多的删除两端出现的指定字符
    - 返回新的字符串，不影响str本身的字符
  - `str.lstrip([chars])`：从左侧删除指定字符集中的字符
  - `str.rstrip([chars])`：从右侧删除指定字符集中的字符
### 字符串查找 ###
- `str.find(sub [,start [, end]])`：在str中查找指定的字符串
  - 找到返回第一次匹配的索引，没找到返回 -1
  - 可以指定查找的起止位置 [start,end)
  - 从左到右的顺序
- `str.rfind(sub [,start [, end]])`：同上反向

- `str.index(sub [,start [, end]])`：在str中查找指定的字符串
  - 找到返回第一次匹配的索引，没找到抛出异常ValueError
  - 可以指定查找的起止位置 [start,end)
  - 从左到右的顺序
- `str.rindex(sub [,start [, end]])`：同上反向

- `str.count(sub [,start [, end]])` ：统计str中字串出现的次数
  - 默认从左到右
  - 可以指定起止位置 [start, end)
  - 返回整数，表示字符出现的次数

### 字符串判断 ###
- `str.startswith(prefix [, start[, end]])`：判断字符【子】串是否以prefix开始
  - 返回布尔类型值
  - 可以指定起止位置 [start, end)
- `str.endswith(suffix [, start[, end]])`：判断字符【子】串是否以suffix结尾
  - 返回布尔类型值
  - 可以指定起止位置 [start, end)

### 身份判定 ###
- `str.isalnum()` ：判定str是否由数字和字母混合组成（单独组成也满足）
- `str.isalpha()`  ：判定str是否只由字母组成
- `str.isdecimal()` ：判定str是否只包含十进制数
- `str.isdigit()`：判定str是否全部数字
- `str.isidentifier()`：判定str是否是正确的标示符名
- `str.islower()`：判定str中是否全是小写字母
- `str.isupper()` ：判定str中是否全部大写字母
- `str.isspace()` ：判定str中是否只有空白符