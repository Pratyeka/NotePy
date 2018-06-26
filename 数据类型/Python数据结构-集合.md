### 集合的基本特点
- 集合是一种可变的、无序的、元素无重复的结构
- 集合可迭代、但是无法索引（因为无序）

***
### 集合的初始化
- `setl = set()`：新建一个空集合
    - `dict = {}`：新建一个字典对象（注意）
- `setl = {ele1, ele2,..., elen}`：新建一个包括如上元素的集合
    - `ele`：必须是可哈希的类型（不变的类型）
    - 不可哈希的类型：列表、集合、bytesarray
- `setl = set(iterable)`：使用一个可迭代对象初始化集合

***
### 集合的常用方法
- set添加元素
    - `set.add(elem)`：给集合中添加元素
        - 如果在set中有该元素，不做任何操作
    - `set.update(*other)`：将other的元素添加到set中
        - `other`：可迭代对象，可以不止一个
        - 就地修改，直接修改set中的元素
- set删除元素
    - `set.remove(elem)`：从set中删除元素
        - 返回类型为None，就地修改
        - set中不存在elem时，会抛出KeyError异常
    - `set.discord(elem)`：从set中删除元素
        - 返回类型为None，就地修改
        - set中不存在elem时，不做任何操作
    - `set.pop()`：从set中随机弹出元素
        - 返回set中的元素，set元素减少
        - pop函数不指定索引，返回的元素是随机的
        - set是空集合时，返回KeyError异常
    - `set.clear()`：清除集合中的所有元素
- set的成员方法
    - `x in set`：元素x在set中
        - 时间复杂度是O(1)--哈希索引，查询时间与数据大小无关
    - `x not in set`：元素x不在set中
        - 时间复杂度时O(1)--哈希索引，查询时间与数据大小无关
        
- *不建议在顺序结构中使用成员方法，效率差*
    - 线性结构的查询时间复杂度是O(n)，随着数据规模增大而增加耗时
    - 用hash作为key，可以将时间复杂度降低到O(1)

***
### 集合运算
- 并集
    - `set1.union(*set2)`：求集合之间的并集
        - `set2`:表示多个集合
        - 返回新的集合，非就地修改
        - `set1 | set2`:并集，返回新的集合
        - `set1.update(*set2)`等价`set1 |= set2`: 就地修改
- 交集
    - `set1.intersection(*set2)`:求集合之间的交集
        - `set2`:表示多个集合
        - 返回新的集合，非就地修改
        - `set1 & set2`:交集，返回新的集合
        - `set1 &= set2`: 就地修改
- 差集
    - `set1.difference(*set2)`: 求集合之间的差集
        - 在set1中存在，但是不在set2中包含的元素
        - `set2`: 表示多个集合
        - `set1 - set2` : 求差集，返回新的集合
        - `set1 -= set2`: 就地修改
- 对称差集
    - `set1.symmetric_differece(set2)`
        - 表示不属于set1和set2交集的其余元素
        - `set1 ^ set2`：返回新的集合
        - `set1 ^= set2`：就地修改set1
- 子集、超集判定
    - `< > <= >=` :根据包含关系即可判定子集、超集
    - `issubset`:是否是子集
    - `issuperset`:是否是超集
    - `set1.isdisjoint(set2)`:是否有交集

- 集合应用：
    - 判定共同好友
    - 权限判定