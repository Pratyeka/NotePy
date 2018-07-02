### CSV文件(Comma-Separated Values)
- CSV是一个被行分割符、列分隔符划分成行和列的文本文件，*且不指定字符编码*
- 行分隔符是\r\n，列分割符一般是“,”或者制表符
- 文件的每一行成为一条记录record
- 字段中如果没有双引号、逗号、换行符，字段的双引号可以省略不写
- 表头可选，和字段列对齐即可

- CSV模块操作
    - csv.reader(csvfileobject, dialect='excel', **fmtparams)
        - 返回DictReader对象，是一个*行迭代器*
        - 可以指定可选关键字参数：
            - `delimiter`: 列分割符，逗号
            - `lineterminator`: 行分割符\r\n
            - `quotechar`: 字段的引用符号，缺省为”
    - csv.writer(csvfileobject, dialect='excel', **fmtparms)
        - 返回DictWriter的实例
        - 主要方法有：
            - writerow(iterable)/writerows(iterables)
    - *注意*：为了防止出现写入空行，可在打开csvfile时，指定newline为‘’
- 栗子
```Python
    from pathlib import Path
    import csv

    def csv_print(filep):
        with open(str(p)) as f:
            reader = csv.reader(f)  # 返回行迭代器
            for line in reader:
                print(line)

    csv_body = """\
    id,name,age,comment
    1,zs,23,'Iom old'
    2,ls,33,'this is a test'
    3,ww,44,'你好'
    """
    csv_body1 = """\
    4,tom,23,'Iom old'
    5,jerry,33,'this is a test'
    6,lucy,44,'你好'
    """
    csv_body2 = [
        [7,'abd',34,'abd'],
        (8,'kom',45,'kom'),
        (9,'justin',54,'justin'),
        'afadsfdfiohkjfhdsajk'
    ]

    print("第一次写入")
    p  = Path("doc/csvt.csv")
    p.write_text(csv_body)  # 将内容写入，普通的字符串写入
    csv_print(p)

    print("第二次写入")      # 使用文件write的方式写入
    with open(str(p), mode='a') as f:
        f.write(csv_body1)
    csv_print(p)

    print("第三次写入")       # 使用csv的模块方法写入
    with open(str(p),mode='a',newline='') as f:
        writer = csv.writer(f, dialect='excel')
        writer.writerow(csv_body2[0])
        writer.writerows(csv_body2)
    csv_print(p)
```

### ini文件
- 在ini文件中的所有内容都是字符串格式
- ini文件作为*配置文件*，基本格式如下：
```ini
    [DEFAULT]
    a = default

    [section]
    option=value
```
- 注：
    - `DEFAULT`是缺省的section的名字，必须大写
    - `section`称为节、区、段
    - 每个section的内容都是由键值对构成，其中key称为option
- 可以把ini配置文件当做一个嵌套的字典，默认使用的是有序字典

#### configparse模块
- 模块导入：`from configparser import ConfigParse`
- 使用configparse来处理ini文件，
    - 可以将section当做key
    - 对应的value也是option组成的键值对
    - 嵌套的字典结构(默认使用的是有序字典)

#### 常用实例方法
- 以下方法都是对 `conf=ConfigParse()` conf实例的操作

- `read(inifiles, encoding=None)`:读取ini文件
    - `inifiles`: 可以是多个ini文件组成的文件列表
    - `encoding`: 可以指定文件编码
- `sections()`: 返回一个由section名组成的列表，不包括缺省的section在内
- `add_section(section_name)`: 增加一个section
- `has_section(section_name)`: 判定section是否存在
- `options(section)`: 返回指定section中所有option，会追加缺省section的option
- `has_option(section,option)`: 判定section是否存在这个option
- `get(section, option)`: 从指定的字段上取值，如果没有找到就去DEFAULT中查找
- `getint(sectin, option)`: 将返回结果从str转为int
- `getfloat(sectin, option)`: 将返回结果从str转为float
- `getboolean(sectin, option)`: 将返回结果从str转为boolean
- `items([section])`
    - section省略，返回视图对象
    - section不省略，返回列表，列表元素是有option和value组成的二元组
- `set(section, option, value)`:写入option=value键值对
    - 要求section必须存在
    - 要求option、value必须是字符串
- `remove_section(section)`: 移除section及其所有option
- `remove_option(section, option)`: 移除section下指定的option
- `write(fileobject)`: 将当前config的内容写入fileobject中
    - 要求fileobject是可写的文件流对象

- *注：* cfg可以按照字典操作：
    - `cfg[section][option]`: 读取数值
    - `cfg[section][option] = 100`: 字典中写入数值
```Python
    from pathlib import Path
    from configparser import ConfigParser

    filep = Path('doc/json.ini')
    newfile = Path('doc/new.ini')

    cfg = ConfigParser()
    cfg.read(str(filep), 'utf-8')
    cfg['mysql']['a'] = '100'
    print(cfg['mysql']['a'])
    cfg['mysection'] = {'abd':'1000'}
    print(cfg['mysection']['abd'])

    with open(str(newfile), mode='w') as f:
        cfg.write(f)
```
```Python
    import os
    from configparser import ConfigParser

    cfg = ConfigParser()
    cfg.read('./doc/logs/new.ini')
    print(cfg.sections())
    print(cfg.has_section('mysql'))
    print(cfg.options('mysql'))
    print(cfg.get('mysql', 'a'))
    print(cfg.items('mysql'))
    print(tuple(cfg.items()))
    cfg.set('mysql', 'kkk', '10000')
    print(cfg.options('mysql'))
    cfg.add_section('json')
    print(cfg.sections())
    cfg.remove_section('json')
    print(cfg.sections())
    cfg.remove_option('mysql', 'a')
    print(cfg.options('mysql'))


    with open('./doc/mi.ini', encoding='UTF-8', mode='w') as f:
        cfg.write(f)
```