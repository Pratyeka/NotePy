### 基本数据类型介绍

- 一维数据
    - 由对等关系
    - 列表和集合类型构成
- 列表和数组
    - 相同：一组数据的有序结构
    - 不同：
        - 列表：数据可以不同
        - 数组：要求数据类型一致
- 二维数据
    - 由多个一维数据构成，是一维数据的组合形式
    - 表格是典型的二维数据
    - 列表形式构成
- 多维数据
    - 由二维数据在新的维度上组合形成
    - 列表形式构成
- 高维数据
    - 仅利用最基本的二元关系展示数据间的复杂结构（键值对组织起来的）
    - 字典类型构成
- 数据格式：json、xml、yaml
***

### numpy的数据对象ndarray
- ndarray: 是一种同类型的多维数组，在程序中的别名是array
    - 存储包括数据元素本身，以及数组的元信息（数据维度、数据类型等）
    - ndarray与列表结构之间的区别是：
        - 去掉元素运算时所需的循环结构，使用时更类似与一个元素，而不是数组
        ```Python
            import numpy as np

            a = np.arange(5)
            b = np.arange(9,4,-1)
            c = a**2 + b**2
        ```
        - python中提供专用的数据结构，可以更好的进行计算优化
        - ndarray结构中的元素是同类型数据，相比于列表结构，可以更好的节省存储空间
        - ndarray支持多种元素定义：元素的精确定义可以优化存储和运行速度
        - ndarray可以由非同质对象构成，会导致如法发挥numpy的优势，尽量避免

- ndarray对象的属性：
    - .ndim: 秩，轴的数量或者维度的数量
    - .shape: ndarray对象的尺寸，对于矩阵(n,m)
    - .size: ndarray对象的元素个数，n*m
    - .dtype: ndarray对象的元素类型
    - .itemsize: ndarray对象中每个元素占用内存大小，以字节为单位
- ndarray对象的数据类型：
    - bool：布尔类型
    - int8: 8位长度的整数类型
    - int64：64位长度的整数类型
    - uint8: 8位无符号整数类型
    - float16: 16位的浮点数类型
    - complex64：64位的浮点数类型（实、虚都是32位）
    - 可以使用dtype类创建自定义的数据类型

### ndarray数组的创建和变换
- 数组的创建：
    - `np.array(list/tuple, dtype=np.float32)`: 创建一个ndarray实例对象
        - array函数，接受一个可迭代对象(列表/元祖)，返回一个ndarray对象
        - dtype省略时，np会根据list元素类型来创建ndarray对象
    - `np.arange(10).reshape(2,5)`: 使用arange函数新建一维的实例对象，在形状修改
        - arange函数，接受一个整数，返回一维ndarray对象
    - `np.ones(shape)`: 返回一个尺寸为shape，元素全为1的ndarray实例对象
        - shape一般是一个tuple
    - `np.zeros(shape)`: 返回一个尺寸为shape，元素全为0的ndarray实例对象
    - `np.full(shape,val)`:返回一个尺寸为shpae，元素为val的ndarray实例对象
    - `np.eye(n)`: 返回一个尺寸为(n,n)的ndarray对象，对角线为1，其余为0
    - `np.ones_like(a)`: 根据数组a的shape生成一个全1的数组
    - `np.zeros_like(a)`: 根据数组a的shape生成一个全0的数组
    - `np.full_like(a:ndarray, val)`:返回对象尺寸与a一致，元素值全为value的数组
    - `np.linspace(start, end, nums)`:介于[start,end]之间的nums个元素的一维数组
        - 设置参数endpoint=False，数组元素介于[start,end)之间
    - `np.concatenate((a,b),axis=1)`:在指定维度拼接a和b两个数组对象

- 数组的变换
    - `a.reshape(shape)`: 按照给定shape修改数组a的尺寸
        - 返回一个数组，数组a不变化
        - `a.shape=(4,6)`: 也可以改名数组a的形状
        a = np.arange(1,21).reshape(4,5)
        ```Python
            a = np.arange(1,21).reshape(4,5)
            array([[ 1,  2,  3,  4,  5],
                [ 6,  7,  8,  9, 10],
                [11, 12, 13, 14, 15],
                [16, 17, 18, 19, 20]])
        ```
    - `a.resize(shape)`: 按照给定shape修改数组a的尺寸
        - 就地修改，数组a的尺寸变化为shape指定的尺寸
    - `a.swapaxes(ax1, ax2)`: 交换数组a的两个维度
        - 返回新数组，原数组a不变
    - `a.ravel()`: 将数组a展平为一维数组
        - 返回的是数组a的视图(view)
    - `a.flatten()`: 将数组a展平成一维的数组
        - 返回新数组，原数组a不变
    - `a.astype(newtype)`: 将数组a中元素类型修改为newtype
        - 返回新数组，原数组a不变
        - 即使两个数组类型一致，也会创建一个新数组
    - `a.transpose()`: 数组a的转置操作
    - `a.tolist()`: 将数组a转为list类型数据（列表嵌套式）
- 数组的组合（反人类的理解方式）只使用concatenate即可
    - `np.hstack(a,b)`: 实现数组a和b在水平方向的拼接
    - `np.vstack(a,b)`: 实现数组a和b在垂直方向上的拼接
    - `np.dstack(a,b)`: 实现数组a和b在深度方向的拼接
    - `column_stack(a,b)`: 一维数组按照列方向进行组合，二维==hstack
    - `row_stack(a,b)`: 一维数组按照行方向进行组合，二维==vstack
    - `concatenate((a,b), axis=0)`: 将数组a和b在指定维度进行组合
- 数组的分割 只是用split即可
    - `np.split(a, 3, axis=1)`:将数组a在指定维度等分
    - `np.vsplit(a, 3)`: 将数组a在垂直方向等分
    - `np.hsplit(a, 3)`: 将数组a在水平方向等分
    - `np.dsplit(a, 3)`: 将数组a在深度方向等分

- 数组的索引和切片
    - 一维数组的索引和切片：与python的列表类似 
        - a[2] : 返回一个元素
        - a[1:14:2] : 返回一个array数组
    - 多维数组的索引和切片
        - a[1,2,3]/a[-1,-2,-3]: 返回一个元素
        - a[:,1,-3]/a[:,1:3,:]/a[:,:,::2]：返回一个array数组
- 数组的运算：
    - 数组与标量之间的运算：标量作用在数组中的每一个元素
        - a = a/a.mean()
    - 对数组执行元素级元算的函数(返回新的数组)：
        - `np.abs(x)/np.fabs(x)`: 计算各元素的绝对值
        - `np.sqrt(x)`: 计算各元素的平方根
        - `np.square(x)`: 计算个元素的平方
        - `np.log(x)/np.log10(x)/np.log2(x)`: 计算数组各元素的自然对数、10为底的数和2为底的数
        - `np.ceil(x)/np.floor(x)`: 计算各元素向上取整和向下取整的数
        - `np.rint(x)`: 计算数组各元素的四舍五入值
        - `np.modf(x)`: 将数组各元素的小数和整数部分以两个独立数组的形式返回
        - `np.cos(x)/np.sin(x)/np.tan(x)`: 计算数组各元素的三角函数值
        - `np.exp(x)`: 计算数组个元素的指数值
        - `np.sign(x)`: 计算数组各元素的符号值
    - numpy的二元函数
        - `+-*/`: 两个数组各元素进行对应运算
        - `np.maximum(x,y)/np.fmax(x,y)`: 元素级的最大值计算，并且数据类型按最准的转换
        - `np.minimun(x,y)/np.fmin(x,y)`: 元素级的最小值计算
        - `np.mod(x,y)`: 元素级的模运算
        - `np.copysign(x,y)`: 将数组y中各元素的符号赋值给数组x对应的元素
        - `> < == !=`: 算数比较，产生布尔型数组


- 数组的存取
    - 一二维数据可以存储为csv文件：
        - `np.savetxt(path, array, fmt, delimiter)`: 存储
        - `np.loadtxt(path, delimiter)`: 取出
    - 多维数据存储为dat文件：
        - `a.tofile(path, sep='', fmt)`: 数据存储(元数据丢失)
        - `np.loadfile(path, dtype, count, sep='')`: 取出
        - 可以另外存储元数据信息，可以更好的取出数据
    - python中的数据存数格式npy：
        - `np.save(path, a)`: 存为npy格式
        - `np.savez(path, a)`: 存为npz格式
        - `np.load(path)`: 加载数据
        - 在save时已经默认存储了数组的元数据信息

- random类
    - `npm.rand(d1, d2, d3, d4)`: 新建一个4维的数组
        - 元素值都是0-1之间的float数据,服从均匀分布
    - `npm.randn(d1, d2, d3, d4)`: 新建一个4为的数组
        - 元素值服从标准正态分布
    - `npm.randint(start, end, shape)`
    - `npm.shuffle(a)`: 对a元素打乱排序，a数据就地修改
    - `npm.uniform(start,end,shape)`
    - `npm.normal(std,val,shape)`