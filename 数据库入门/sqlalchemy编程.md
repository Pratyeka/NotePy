### SQLAlchemy框架
- SQLAlchemy框架是一个ORM框架，大量使用了元编程
- 在SQLAlchemy框架内部使用了连接池

### SQLAlchemy框架使用
1. 创建连接
```Python
    import sqlalchemy

    host = '192.168.40.129'
    username = 'xdlb'
    password = 'xdliubin'
    port = 3306
    database = 'test'

    conn_str = "mysql+pymysql://{}:{}@{}:{}/{}".format(
        username, password, host, port, database
    )

    # echo可以返回sql语句，用于调试检查语法
    engine = sqlalchemy.create_engine(conn_str, echo=True)
```

2. 创建基类
    - 基类中已经修改了元类，创建实体类继承自基类
    - 在基类中添加了关于表的通用处理函数
```Python
    from sqlalchemy.ext.declarative import declarative_base

    Base = declarative_base()
```

3. 创建实体类
    - 要求从Base中继承（修改了元类）
    - 设置__tablename__类属性，指定表名
    - 添加字段名，并设置对应的字段信息
    - **实体类对应一张表**
```Python
    from sqlalchemy import Column,Integer,String

    class Student(Base):
        __tablename__ = 'Student'

        id = Column(Integer, primary_key=True)
        name = Column('name', String(16), nullable=False)
        # 第一个参数是字段名，如果和属性名不一致，一定要指定
        age = Column(Integer, nullable=False)
        
        def __repr__(self):
            return "{}, id={} name={} age={}".format(
                self.__tablename__, self.id, self.name, self.age
            )
```
4. 实例化
    - 实例化表示表中的一条记录
    - 给出如下两种实例化方法：
```Python
    tom = Student(name='Tom', age=12)
    print(tom)

    jerry = Student()
    jerry.name = 'jerry'
    jerry.age = 33
    print(jerry)
```
5. 创建表
    - drop_all会删除数据库中的所有表
    - create_all可以用临时库来检查语法正确性
    - *生产环境中一般很少这样生成表*
    - *生产环境中一般不会删除表信息*
```Python
    Base.metadata.drop_all(engine)
    Base.metadata.create_all(engine)
```
6. 建立会话
    - 首先创建一个会话类，设置对应的engine
    - 创建会话实例，用来提交数据，查询数据
```Python
    from sqlalchemy.orm import sessionmaker

    Session = sessionmaker(bind=engine)
    session = Session()
```
***
7. DML操作
- 数据增加
    - add方法添加一个实例，commit提交到数据表中持久化
    - add_all方法接受一个实例列表，commit提交到数据表中持久化
        - *注意：下面代码中的add_all操作并没有提交数据，原因如下*
            - tom在提交后有状态以及ID，tom没有修改的情况下是不提交的
```Python
    session.add(tom)
    session.commit()

    session.add_all([jerry, tom])
    session.commit()
```
- 数据修改
    - tom如果是为提交过的实例，设置主键，添加到表中
    - tom如果是已提交过的实例，修改表中信息
        - commit后对实例属性的修改
        - 从表中拿回来实例再修改
```Python
    session.add(tom)
    session.commit()

    tom.age = 120
    session.add_all([jerry, tom])
    session.commit()

    stu = session.query(Student).get(2)
    stu.name = 'aliy'
    stu.age = 44
    session.add(stu)
    session.commit()
```
- 数据删除
    - 删除数据要求是已经commit过的
    - 删除为提交过的数据会抛出 `为持久化异常`
```Python
    try:
        tom = session.query(Student).get(2)
        session.delete(tom)

        # 抛出未持久化异常
        stu = Student(name='kaka', age=20)
        session.delete(stu)
        session.commit()
    except Exception as e:
        session.rollback()
```
#### 数据查询
- 简单查询
    - 使用query得到的返回值是一个Query对象（可迭代对象）
    - query方法的参数表示返回的可迭代对象中的元素的具体类型
    - Query对象的get方法返回Student实例
```Python
    stus = session.query(Student)

    for stu in stus:
        print(stu)

    stu = session.query(Student).get(2)
    print(stu)
```
- 复杂查询
    ##### **filter语句 -> where子句**
    
    ```Python
        stus = session.query(Student).filter(Student.id < 20)
        for stu in stus:
            print(stu)
        # 两个filte语句表示与操作
        stus = session.query(Student).filter(Student.id < 20).filter(Student > 3)
        for stu in stus:
            print(stu)
    ```
    ##### **与或非操作**
    - 使用sqlalchemy库中方法（与或可以使用，非手动取反使用）
    ```Python
        stus = session.query(Student).filter(and_(Student.id<10, Student.id>3))
        stus = session.query(Student).filter(or_(Student.id>10, Student.id<3))
        stus = session.query(Student).filter(not_(Student.id<20))
    ```
    - 使用操作符
    ```Python
        stus = session.query(Student).filter((Student.id<10) & (Student.id>3))
        stus = session.query(Student).filter((Student.id>10) | (Student.id<3))
        stus = session.query(Student).filter(~(Student.id<20))
    ```
    ##### **in/not in/like**
    ```Python
    id_lst = [1,2,3]
    stus = session.query(Student).filter(Student.id.in_(id_lst))
    stus = session.query(Student).filter(~Student.id.in_(id_lst))
    stus = session.query(Student).filter(Student.name.like('%P'))
    # ilike可以忽略大小写
    stus = session.query(Student).filter(Student.name.ilike('%P'))
    ```
    ##### 排序
    ```Python
        # 默认是升序
        stus = session.query(Student).order_by(Student.id)
        stus = session.query(Student).order_by(Student.id.desc())
        # 多列排序（先按age排，age一致再按id排）
        stus = session.query(Student).order_by(Student.age.desc()).order_by(Student.id)
    ```
    ##### 分页
    ```Python
        stus = session.query(Student).limit(5)
        stus = session.query(Student).limit(5).offset(3)
    ```
    ##### 消费者方法
    - 消费这方法调用后，Query对象就变成了一个容器
    ```Python
        stus = session.query(Student).all()
        stu = session.query(Student).first()
        stu = session.query(Student).one()
        stu = session.query(Student).delete()
    ```
    ##### 聚合函数
    ```Python
        from sqlalchemy import func

        query = session.query(func.count(Student.age))
        print(query.scalar())
        age = session.query(func.max(Student.age)).scalar()
        age = session.query(func.min(Student.age)).scalar()
        age = session.query(func.avg(Student.age)).scalar()
        # 分组的执行顺序，先分组，在select，返回列表
        lst = session.query(Student.name, func.count(Student.age)).group_by(Student.name).all()
    ```
    ##### 关联查询
    - 新建实体类(添加外键)
    ```Python
        from sqlalchemy import Column,Integer,String,Date,SmallInteger,ForeignKey

        class Tee(Base):
            __tablename__ = 'Tee'

            emp_no = Column(Integer, primary_key=True)
            birth_date = Column(Date, nullable=False)
            first_name = Column(String(13), nullable=False)
            last_name = Column(String(14), nullable=False)
            gender = Column(SmallInteger, nullable=False)
            hire_date = Column(Date, nullable=False)

            def __repr__(self):
                return "{} name={} emp_no={}".format(
                    self.__tablename__, self.first_name + self.last_name, self.emp_no)

        class Department(Base):
            __tablename__ = 'Department'

            dept_no = Column(String(5), primary_key=True)
            dept_name = Column(String(50), nullable=False, unique=True)

            def __repr__(self):
                return "{} no={} name={}".format(
                    self.__tablename__, self.dept_no, self.dept_name
                )

        class Dept_emp(Base):
            __tablename__ = 'dept_emp'

            # 使用Foreignkey定义外键约束
            emp_no = Column(Integer, ForeignKey('Tee.emp_no',ondelete='CASCADE'),primary_key=True)
            dept_no = Column(String(4), ForeignKey('Department.dept_no', ondelete='CASCADE'),primary_key=True)
            from_data = Column(Date, nullable=False)
            to_date = Column(Date, nullable=False)

            def __repr__(self):
                return "{} empno={} deptno={}".format(
                    self.__tablename__, self.emp_no, self.dept_no
                )
    ```
    - join查询
    ```Python
        # a join b where con
        emps = session.query(Tee).join(Dept_emp).filter(Tee.emp_no == 10010).all()
        # a join b on con
        emps = session.query(Tee).join(Dept_emp, Tee.emp_no == 10010).all()
        # a join b on con1&con2
        emps = session.query(Tee).join(Dept_emp, (Tee.emp_no == 10010) & (Dept_emp.emp_no == 10000).all()
    ```
    - 修改属性
        - 添加relationship属性值，可以用来查看其它表对应的对象的内容
    ```Python
        from sqlalchemy import Column,Integer,String,Date,SmallInteger,ForeignKey
        from sqlalchemy.orm import relationship

        class Tee(Base):
            __tablename__ = 'Tee'

            emp_no = Column(Integer, primary_key=True)
            birth_date = Column(Date, nullable=False)
            first_name = Column(String(13), nullable=False)
            last_name = Column(String(14), nullable=False)
            gender = Column(SmallInteger, nullable=False)
            hire_date = Column(Date, nullable=False)

            dept_emps = relationship('Dept_emp')

            def __repr__(self):
                return "{} name={} emp_no={} rel={}".format(
                    self.__tablename__, self.first_name + self.last_name, 
                    self.emp_no, self.dept_emps)
    ```