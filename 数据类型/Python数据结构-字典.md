***
### 字典dict结构特点
- 元素是key-value对形式
- 是一种可变的、无序的、key不重复、可迭代、不可索引类型数据对象

### 字典dict的初始化
- `{}`: 新建一个空字典
- `dict()`: 新建一个空字典
- `dict(**kargs)`:使用多个kv对来新建字典
    - d = dict(a=1,b=1,c=4)  *注：a不能用'a'/'b'*
- `dict(iterable, **kargs)`:使用可迭代对象和kv对新建字典
    - 要求可迭代对象的元素只能是二元组
    - 会自动去重并更新value大小
        - d = dict((['a',1],['b',2],['c',4]), d=[2,3,3])
- `dict(mapping, **kargs)`:使用字典对象和kv对新建字典
    - mapping表示一个字典
    - 会自动去重并更新value大小
- `dict.fromkeys(iterable [,value])`
    - 根据可迭代对象的元素与value值构建kv对
    - value默认是None
    - d.fromkeys(range(10), 'a')

### 字典元素的访问
- `dict[key]`: 返回key对应的value值
    - key不存在时抛出KeyError异常
- `dict.get(key [,value])`: 返回key对应的value值
    - key存在返回value值
    - key不存在返回value值
    - value缺省值为none
- `dict.setdefault(key [,value])`:
    - key存在返回value值，不修改value大小
    - key不存在时，在dict中自动添加新的kv对，并返回值value
    - value的缺省值为None

### 字典增加和修改
- `dict[key] = value`: 修改key对应的value
    - key不存在时，在dict中新建kv对
- `dict.update([other])`: 更新字典中的元素
    - 返回值为None，就地修改
    - 字典中不存在，整合添加
    - 字典中存在，修改value值大小
    - other：可以是字典，kv对，元素为二元组的可迭代对象
```Python
    dict1 = dict.update({'d':2})
    dict1 = dict.update(name=2)
    dict1 = dict.update((('name',3),))
```
### 字典删除
- `dict.pop(key [,default])`: 移除kv对，返回key对应的value
    - key存在，移除kv对，返回对应的value值
    - key不存在，返回设置的default
    - key不存在，并且没有设置default，抛出keyError异常
- `dict.popitems()`:移除任意的kv对，返回键值对
    - 移除并返回任意的kv对
    - 字典为空时，抛出keyError异常
- `dict.clear()`: 清空字典
- `del 语句`
    - 只是删除对象名称(引用计数)，不是删除对象

### 字典遍历
- 遍历key
    - `for _ in dict`: 默认遍历字典的key值
    - `for _ in dict.keys()`: 
        - dict.keys()返回的是字典view对象
        - 是一个可迭代对象
        - 惰性计算方式
- 遍历value
    - `for _ in dict.values()`:
        - dict.values 返回的时字典view对象
        - 是一个可迭代对象
        - 惰性计算方式
- 遍历kv对
    - `for _,_ in dict.items()`:
        - 可以使用封装解构直接获得对应的KV参数值
        - 也可以拿到元祖，再手动拆分处理
        - 惰性计算方式
- 遍历删除元素
    - 在对字典进行for循环时，不可以删除元素中元素
    - 一般使用列表记录满足删除条件的key，在迭代列表执行pop

***
### 特殊字典类
- defaultdict
    - 语法：`defaultdict([factory [, ...]])`
    - `facroty`: 为找不到的键创造默认值的方法(可调用对象)
    - 来自collections模块
    - 如果key不存在，新建kv对，value根据工厂函数设置
```Python
    dd = defaultdict(list)，如果键'new-key' 在dd 中还不存在的话，
    表达式dd['new-key'] 会按照以下的步骤来行事
    (1) 调用list() 来建立一个新列表
    (2) 把这个新列表作为值，'new-key' 作为它的键，放到dd中
    (3) 返回这个列表的引用
```

- OrderedDict
    - 语法：`OrderedDict([items])`
    - 来自collections模块
    - 有序字典可以记录元素的插入顺序，打印输出也是按照插入顺序打印