

# 轻松玩转Numpy

## 前言

NumPy 是一个运行速度非常快的数学库，主要用于数组计算。

## 为什么使用 numpy？

- 一个强大的N维数组对象 ndarray
- 广播功能函数
- 整合 C/C++/Fortran 代码的工具
- 线性代数、傅里叶变换、随机数生成等功能

## numpy 安装

```
# 使用 python 3+:
pip3 install numpy

# 使用 python 2+:
pip install numpy
```

## Numpy 属性

介绍几种 numpy 的属性:

- `ndim`：维度
- `shape`：行数和列数
- `size`：元素个数

```
import numpy as np #为了方便使用numpy 采用np简写
array = np.array([[1,2,3],[2,3,4]])  #列表转化为矩阵
print(array)
"""
array([[1, 2, 3],
       [2, 3, 4]])
"""
```

接着我们看看这几种属性的结果:

```
print('number of dim:',array.ndim)  # 维度
# number of dim: 2

print('shape :',array.shape)    # 行数和列数
# shape : (2, 3)

print('size:',array.size)   # 元素个数
# size: 6
```

## NumPy 创建数组

ndarray 数组除了可以使用底层 ndarray 构造器来创建外，也可以通过以下几种方式来创建。

### 关键字

- `array`：创建数组
- `dtype`：指定数据类型
- `zeros`：创建数据全为0
- `ones`：创建数据全为1
- `empty`：创建数据接近0
- `arrange`：按指定范围创建数据
- `linspace`：创建线段

### 创建数组

```
a = np.array([12,54,3])  # list 1d
print(a)
# [12 54 3]
```

### 指定数据 dtype

```
np.array([12,54,3],dtype=np.int)
```

### 创建特定数据

```
a = np.array([[2,23,4],[2,32,4]])  # 2d 矩阵 2行3列
print(a)
"""
[[ 2 23  4]
 [ 2 32  4]]
"""
np.ones((3,4),dtype = np.int)   # 数据为1，3行4列
np.empty((3,4)) # 数据为empty，3行4列
np.arange(10,20,2) # 10-19 的数据，2步长
np.arange(12).reshape((3,4))    # 3行4列，0到11
np.linspace(1,10,20)    # 开始端1，结束端10，且分割成20个数据，生成线段
np.linspace(1,10,20).reshape((5,4)) # 更改shape
```

## Numpy 基础运算

NumPy 算术函数包含简单的加减乘除: **add()**，**subtract()**，**multiply()** 和 **divide()**。

需要注意的是数组必须具有相同的形状或符合数组广播规则。

```
import numpy as np
 
a = np.arange(9, dtype = np.float_).reshape(3,3)
print ('第一个数组：')
print (a)
print ('\n')
print ('第二个数组：')
b = np.array([10,10,10])
print (b)
print ('\n')
print ('两个数组相加：')
print (np.add(a,b))
print ('\n')
print ('两个数组相减：')
print (np.subtract(a,b))
print ('\n')
print ('两个数组相乘：')
print (np.multiply(a,b))
print ('\n')
print ('两个数组相除：')
print (np.divide(a,b))
```

此外 Numpy 也包含了其他重要的算术函数。

### numpy.reciprocal()

numpy.reciprocal() 函数返回参数逐元素的倒数。如 **1/4** 倒数为 **4/1**。

### numpy.power()

numpy.power() 函数将第一个输入数组中的元素作为底数，计算它与第二个输入数组中相应元素的幂。

### numpy.mod()

numpy.mod() 计算输入数组中相应元素的相除后的余数。函数 numpy.remainder() 也产生相同的结果。

# NumPy 字符串函数

以下函数用于对 dtype 为 numpy.string_ 或 numpy.unicode_ 的数组执行向量化字符串操作。它们基于 Python 内置库中的标准字符串函数。

这些函数在字符数组类（numpy.char）中定义。

| 函数           | 描述                                       |
| :------------- | :----------------------------------------- |
| `add()`        | 对两个数组的逐个字符串元素进行连接         |
| multiply()     | 返回按元素多重连接后的字符串               |
| `center()`     | 居中字符串                                 |
| `capitalize()` | 将字符串第一个字母转换为大写               |
| `title()`      | 将字符串的每个单词的第一个字母转换为大写   |
| `lower()`      | 数组元素转换为小写                         |
| `upper()`      | 数组元素转换为大写                         |
| `split()`      | 指定分隔符对字符串进行分割，并返回数组列表 |
| `splitlines()` | 返回元素中的行列表，以换行符分割           |
| `strip()`      | 移除元素开头或者结尾处的特定字符           |
| `join()`       | 通过指定分隔符来连接数组中的元素           |
| `replace()`    | 使用新字符串替换字符串中的所有子字符串     |
| `decode()`     | 数组元素依次调用`str.decode`               |
| `encode()`     | 数组元素依次调用`str.encode`               |

## NumPy 排序、条件刷选函数

NumPy 提供了多种排序的方法。这些排序函数实现不同的排序算法，每个排序算法的特征在于执行速度，最坏情况性能，所需的工作空间和算法的稳定性。下表显示了三种排序算法的比较。

| 种类                      | 速度 | 最坏情况      | 工作空间 | 稳定性 |
| :------------------------ | :--- | :------------ | :------- | :----- |
| `'quicksort'`（快速排序） | 1    | `O(n^2)`      | 0        | 否     |
| `'mergesort'`（归并排序） | 2    | `O(n*log(n))` | ~n/2     | 是     |
| `'heapsort'`（堆排序）    | 3    | `O(n*log(n))` | 0        | 否     |

### numpy.sort()

numpy.sort() 函数返回输入数组的排序副本。函数格式如下：

```
numpy.sort(a, axis, kind, order)
```

参数说明：

- a: 要排序的数组
- axis: 沿着它排序数组的轴，如果没有数组会被展开，沿着最后的轴排序， axis=0 按列排序，axis=1 按行排序
- kind: 默认为'quicksort'（快速排序）
- order: 如果数组包含字段，则是要排序的字段

```
import numpy as np
 
a = np.array([[3,7],[9,1]])
print ('我们的数组是：')
print (a)
print ('\n')
print ('调用 sort() 函数：')
print (np.sort(a))
print ('\n')
print ('按列排序：')
print (np.sort(a, axis =  0))
print ('\n')
# 在 sort 函数中排序字段
dt = np.dtype([('name',  'S10'),('age',  int)])
a = np.array([("raju",21),("anil",25),("ravi",  17),  ("amar",27)], dtype = dt)
print ('我们的数组是：')
print (a)
print ('\n')
print ('按 name 排序：')
print (np.sort(a, order =  'name'))
```

### msort、sort_complex、partition、argpartition

| 函数                                      | 描述                                                         |
| :---------------------------------------- | :----------------------------------------------------------- |
| msort(a)                                  | 数组按第一个轴排序，返回排序后的数组副本。np.msort(a) 相等于 np.sort(a, axis=0)。 |
| sort_complex(a)                           | 对复数按照先实部后虚部的顺序进行排序。                       |
| partition(a, kth[, axis, kind, order])    | 指定一个数，对数组进行分区                                   |
| argpartition(a, kth[, axis, kind, order]) | 可以通过关键字 kind 指定算法沿着指定轴对数组进行分区         |

# Numpy浅拷贝 & 深拷贝

## = 的赋值方式会带有关联性

首先 `import numpy` 并建立变量, 给变量赋值。

```
import numpy as np

a = np.arange(4)
# array([0, 1, 2, 3])

b = a
c = a
d = b
```

改变`a`的第一个值，`b`、`c`、`d`的第一个值也会同时改变。

```
a[0] = 11
print(a)
# array([11,  1,  2,  3])
```

确认`b`、`c`、`d`是否与`a`相同。

```
b is a  # True
c is a  # True
d is a  # True
```

同样更改`d`的值，`a`、`b`、`c`也会改变。

```
d[1:3] = [22, 33]   # array([11, 22, 33,  3])
print(a)            # array([11, 22, 33,  3])
print(b)            # array([11, 22, 33,  3])
print(c)            # array([11, 22, 33,  3])
```

## copy() 的赋值方式没有关联性

```
b = a.copy()    # deep copy
print(b)        # array([11, 22, 33,  3])
a[3] = 44
print(a)        # array([11, 22, 33, 44])
print(b)        # array([11, 22, 33,  3])
```

此时`a`与`b`已经没有关联。