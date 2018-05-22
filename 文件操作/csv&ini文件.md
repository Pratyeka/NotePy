### CSV文件(Comma-Separated Values)
- CSV是一个被行分割符、列分隔符划分成行和列的文本文件，且不指定字符编码
- 行分隔符是\r\n，列分割符一般是“,”或者制表符
- 文件的每一行成为一条记录record
- 字段中如果没有双引号、逗号、换行符，字段的双引号可以省略不写
- 表头可选，和字段列对齐即可

- CSV模块操作
    - csv.reader(csvfile, dialect='excel', **fmtparams)
        - 返回DictReader对象，是一个行迭代器
        - 可以指定可选关键字参数：
            - `delimiter`: 列分割符，逗号
            - `lineterminator`: 行分割符\r\n
            - `quotechar`: 字段的引用符号，缺省为”
    - csv.writer(csvfile, dialect='excel', **fmtparms)
        - 返回DictWriter的实例
        - 主要方法有：
            - writerow(iterable)/writerows(iterables)

### ini文件
- 在ini文件中的所有内容都是字符串格式
- ini文件作为配置文件，基本格式如下：
```
[DEFAULT]
a = default

[section]
option=value
```
- `DEFAULT`是缺省的section的名字，必须大写
- `section`称为节、区、段
- 每个section的内容都是一个键值对，其中key称为option

#### configparse模块
- 模块导入：`from configparser import ConfigParse`
- 使用configparse来处理ini文件，可以将section当做key，对应的value也是option组成的键值对，嵌套的字典结构(默认使用的是有序字典)
- 以下方法都是对 `conf=ConfigParse()` conf对象的操作
- `read(inifiles, encoding=None)`:读取ini文件
    - `inifiles`: 可以是多个ini文件组成的文件列表
    - `encoding`: 可以指定文件编码
- `section()`: 返回一个section列表，不包括缺省的section在内
- `add_section(section_name)`: 增加一个section
- `has_section(section_name)`: 判定section是否存在
- `options(section)`: 返回指定section中所有option，包括缺省的option在内
- `has_option(section,option)`: 判定section是否存在这个option
- `get(section, option)`: 从指定的字段上取值，如果没有找到就去DEFAULT中查找
- `getint(sectin, option)`: 将返回结果从str转为int
- `getfloat(sectin, option)`: 将返回结果从str转为float
- `getboolean(sectin, option)`: 将返回结果从str转为boolean
- `items([section])`
    - section省略，返回所有section名字及其对象
    - section不省略，返回这个指定section的键值对组成的二元组
- `set(section, option, value)`:写入option=value键值对
    - 要求section必须存在
    - 要求option、value必须是字符串
- `remove_section(section)`: 移除section及其所有option
- `remove_option(section, option)`: 移除section下的所有option
- `write(fileobject)`: 将当前config的内容写入fileobject中
