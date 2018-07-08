### 数据库
- 数据库：按照数据结构组织、管理、存储数据的仓库
- 关系型数据库：使用行和列来组织数据、和关系

### MySQL简介
- MySQL是一种关系型数据库管理软件，支持网络访问，默认服务器端口是3306
- MySQL通信使用mysql协议，基于tcp网络协议通信
- [Mysql安装参考](https://github.com/Pratyeka/NotePy/blob/master/%E9%A1%B9%E7%9B%AE%E7%AE%A1%E7%90%86/gogs%E6%90%AD%E5%BB%BAgit%E7%A7%81%E6%9C%8D.md)


### 视图
- 也称为虚表，是由查询语句生成的，可以通过视图进行CRUD操作
- 视图的作用：
    - 简化操作：将复杂查询SQL语句定义为视图，可以简化查询
    - 数据安全：视图可以只显示真实表的部分列，隐藏真实的数据表

### 关系操作
- 在关系行数据库中，关系就是二维表
    - 选择（selection）：又称为限制，是从关系中选择出满足给定条件的元祖
    - 投影（projection）：选择出若干属性列组成新的关系
    - 连接（join）：将不同的两个关系连接成一个关系


### SQL语句
- SQL是结构化查询语言，所有主流的关系型数据库都支持SQL语句
- SQL语句分类:
    - DDL数据定义语言:负责数据库定义、数据库对象定义，CREATE/ALTER/DROP 
    - DML数据操作语言:负责对数据库对象的操作，增删改查
    - DCL数据控制语言:负责数据库权限访问控制，有GRANT和REVOKE权限管理
    - TCL事务控制语言:负责处理ACID事务，支持commit、rollback指令
- *注：*
    - SQL语句对大小写不敏感
    - SQL语句末尾使用；作为结束符

***
#### DCL语句
- GRANT授权、REVOKE撤销
    - `GRANT ALL ON database.* TO 'user'@'IP' IDENTIFIED by 'passwd';`
    - `REVOKE ALL ON *.* FROM user;`
- `*`表示通配符，指代任意库或者任意表
    - `*.*`表示所有库的所有表
    - `database.*`表示database库中的所有表
- %为通配符，它是SQL语句的统配符，匹配任意长度字符串

***
#### DDL语句
- 删除 DROP
    - 删除用户：`DROP USER user`
    - 删除数据库： `DROP DATABASE IF EXISTS gogs`
- 创建 CREATE
    - 建库
        - 库是数据的集合，所有数据按照数据模型组织在数据库中
        - 需要指定字符集，以及对应的排序规则
        - 建议使用工具新建数据库
    - 建表
        - 表分为行和列，MySQL是行存储数据库，数据的列必须是固定的
        - 行：称为记录或者元祖
        - 列：称为字段
- 主键 PRIMARY KEY
    - 表中一列或者多列组成唯一的KEY，通过一列或者多列能唯一的标识一条记录
    - 主键的列不能为null，要求唯一（但是可以不连续）
    - 表可以没有主键，但是一般都会有一个
- 索引 INDEX
    - 可以看做是字典的目录，为了快速检索使用
    - 可以对一列或者多列字段设定索引
        - 主键索引：主键会自动建立主键索引 （PRIMARY KEY）
        - 唯一索引：表中的索引可以为null，要求非空的索引必须唯一 (UNIQUE)
        - 普通索引：没有唯一性要求，只是一个索引（不常用）
- 约束 Constraint
    - UNIQUE 唯一键约束
        - 定义了唯一键索引，就定义了唯一键约束
    - PRIMARY KEY 主键约束
        - 定义了主键，就定义了主键约束
    - Foreign Key 外键约束
        - 外键：在表B中的列，关联表A中的主键，表B中的列就是外键
            - 在表B中插入一条数据，B的外键列插入了一个值，这个值必须是表A中存在的主键
            - 删除表A中的主键，如果表B中使用该主键，要先删除B中的记录
            - 外键约束，是为了保证数据完整性、一致性，杜绝数据冗余、数据讹误
***
#### DML语句
- INSERT语句
    - 表中插入一行数据，自增字段，缺省字段，可以为空的字段可以省略不写
        - `INSERT INTO table (c1,...,cn) VALUES (v1,v2,..,vn);`
    - 如果主键冲突、唯一键冲突执行update后的设置（更新字段）
        - `INSERT INTO table (C1) VALUES (V1) ON DUPLICATE KEY UPDATE c1=v1,...;`
    - 如果主键冲突、唯一键冲突就忽略错误，返回一个警告
        - `INSERT IGNORE INTO table (c1) VALUES (v1);`
- UPDATA语句
    - 更新字段值
        - `UPDATE [IGNORE] table SET cn1=new,... where define;`
    - ignore 也是忽略字段
- DELETE语句
    - 删除符合条件的记录
        - `DELETE FROM table WHERE define;`
- SELECT语句
    - 返回表中的所有字段：
        - `SELECT * FROM table`
            - 建议：尽量不使用*，指定对应的字段名，即使是全部的字段名
            - 一般主键必须指定，方便以后的查找
    - 字符串合并：
        - `SELECT id,CONCAT(first_name, last_name) FROM table;`
    - as 定义别名
        - `SELECT CONCAT(first_name,last_name) as name FROM table;`
    - LIMIT子句
        - 限制只返回5条记录
            - `SELECT id FROM table LIMIT 5;`
        - 限制返回5条记录，偏移15
            - `SELECT id FROM table LIMIT 5 OFFSET 15;`
    - WHERE子句
        - 如果where表达式需要使用很多的AND、OR表达式，使用小括号来安排顺序
    - order by子句
        - 对*查询结果*进行排序，可以升序ASC、降序DESC
        - `SELECT * FROM table ORDER BY id DESC;`
        - 如果指定两个字段名：先按第一个排序，字段名相同时按第二字段排序
    - DISTINCT
        - 不返回重复记录
        - `SELECT DISTINCT names from tabel;`
        - 如果有两个字段，会将两个字段按照一个元祖来匹配去重复
    - 聚合函数（一般与分组一起出现）
        - `COUNT(EXPR)`: 返回表中记录的数目，如果指定列，返回非null的行数
        - `COUNT(DISTINCT expr)`: 返回不重复的非NULL中的行数
        - `AVG([DISTINCT] expr)`: 返回平均值，返回不同值的平均值
        - `MIN(expr)/MAX(expr)`: 返回最小值，最大值
        - `SUM([DISTINCE] expr)`: 求和
    - 分组查询
        - 使用Group by子句，可以使用Having子句过滤分组、聚合过的结果
        - 聚合所有
            - `SELECT SUM(id) FROM table`
        - 聚合被选择的记录
            - `SELECT SUM(id) FROM table WHERE sal>1000`
        - 分组后，按组聚合
            - `SELECT SUM(id) FROM table GROUP BY name`
        - 分组并过滤后，按组聚合
            - `SELECT SUM(id) FROM table GROUP BY name HAVING AVG(sal）> 2000`
        - 分组并过滤后，按组聚合并排序
            - `SELECT SUM(id) FROM table GROUP BY name HAVING AVG(sal）> 2000 ORDER BY sal`
- 子查询
    - 查询语句可以嵌套，内部查询就是子查询
    - 子查询必须在一组小括号内
    - 自查询不能使用ORDER BY 
    ```SQL
    SELECT * FORM table WHERE id IN (SELECT id FROM table1 WHERE sal>1000) order by sal;
    ```
- 连接join
    - 交叉连接 cross join： 全部连接
    - 内连接 inner join：省略为join
    - 等值连接： 只选用某些field相等的行，使用ON限定关联的结果
        - `SELECT * FROM table JOIN table1 ON table.id = table1.id`
    - 左外连接
        - 看表的数据的方向，谁是主表，谁的所有数据都显示
        - 没有的补null
    - 右外连接
        - 同左外连接原理
    - 自连接
        - 自己和自己连接

***
#### select语句的执行顺序
- **select语句中关键字的出现顺序**
1. from
2. where
3. group by
4. having
5. order by
6. limit
7. for update

***
### 事务Transation
- InnoDB引擎，支持事务
- 事务：由若干条语句组成，指的是要做的一系列操作
- 关系型数据库中支持事务：必须支持四个属性ACID
    - 原子性A：一个事务是不可分割的单位
    - 一致性C：串行与并行结果一致性
    - 隔离性I：事务的执行不受其他事务的干扰
        - 两个事务之间的数据是隔离的，不干扰
    - 持久性D：事务提交后，对数据库中的数据改变是永久性的

#### 隔离性问题
- 事务之间的隔离级别不好，事务操作之间就会互相影响，带来不同程度的后果
    - 更新丢失：事务A和B同时更新一个数据，后面事务的提交会覆盖前面事务的提交
    - 脏读：事务B读取到事务A未提交的数据
    - 不可重复读：不能保证在事务中执行相同的查询语句得到相同的结果
    - 幻读：事务A中多次查询，事务B插入数据，导致返回的结果数目不一致
- 隔离级别来避免上述问题，由低到高：
    - READ UNCOMMITTED: 读到未提交的数据
    - READ COMMITTED: 读到以提交的数据
    - REPEATABLE READ: 可重复读
    - SERIALIZABLE: 可串行化，事务完全隔离为串行执行
- *注：*
    - 隔离级别越高，串行化越高，数据库的效率越低
    - 隔离界别越低，并行程度越高，性能越高
    - 隔离级别越高，当前事务的中间结果对其他事务的不可见程度越高
