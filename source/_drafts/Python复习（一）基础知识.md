# Python复习（一）基础知识

## 注释

### 单行注释

Python 中单行注释以 # 开头，例如： 

```
# 这是一个注释 
print("Hello, World!")
```

### 多行注释

多行注释用三个单引号 `'''` 或者三个双引号 `"""` 将注释括起来，例如:

```
''' 
这是多行注释，用三个单引号 
这是多行注释，用三个单引号 
这是多行注释，用三个单引号 
''' 
"""
这是多行注释，用三个双引号
这是多行注释，用三个双引号
这是多行注释，用三个双引号 
"""
print("Hello, World!")
```

## print()函数

### 语法

以下是 print() 方法的语法:

```
print(*objects, sep=' ', end='\n', file=sys.stdout, flush=False)
```

### 参数

- objects -- 复数，表示可以一次输出多个对象。输出多个对象时，需要用 , 分隔。
- sep -- 用来间隔多个对象，默认值是一个空格。
- end -- 用来设定以什么结尾。默认值是换行符 \n，我们可以换成其他字符串。
- file -- 要写入的文件对象。
- flush -- 输出是否被缓存通常决定于 file，但如果 flush 关键字参数为 True，流会被强制刷新。

### 返回值

无。

### example

```
>>> print(1)  
1  
>>> print("Hello World")  
Hello World  
 
>>> a = 1
>>> b = 'runoob'
>>> print(a,b)
1 runoob
 
>>> print("aaa""bbb")
aaabbb
>>> print("aaa","bbb")
aaa bbb
>>> 
 
>>> print("www","runoob","com",sep=".")  # 设置间隔符
www.runoob.com
```

## input() 函数

### 函数语法

```
input([prompt])
```

### 参数说明

- prompt: 提示信息

### example

```
>>> a = input("input:")
input:123                  # 输入整数
>>> type(a)
<class 'str'>              # 字符串

>>> a = input("input:")    
input:runoob              # 正确，字符串表达式
>>> type(a)
<class 'str'>             # 字符串

#input() 接收多个值
>>> a,b,c = input("请输入三角形三边的长：").split()
请输入三角形三边的长：1 2 3
>>> type(a)
<class 'str'>
```

## import 语句

想使用 Python 源文件，只需在另一个源文件里执行 import 语句，语法如下：

```
import module1[, module2[,... moduleN]
```

当解释器遇到 import 语句，如果模块在当前的搜索路径就会被导入。简单的import语句导入并没有把直接定义在模块中的函数名称写入到当前符号表里，只是把模块的名字写到了那里。使用时需要通过模块名称来访问函数：

```
>>> import math
>>> sin(60)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'sin' is not defined
>>> math.sin(60)
-0.3048106211022167
```

如果直接访问函数，你可以把它赋给一个本地的名称：

```
>>> sin=math.sin
>>> sin(60)
-0.3048106211022167
```

或者使用 `from … import *` 语句（后面介绍）

### from … import 语句

Python 的 from 语句让你从模块中导入一个指定的部分到当前命名空间中，语法如下：

```
from modname import name1[, name2[, ... nameN]]
```

例如，要导入模块 math 的 sin 函数，使用如下语句：

```
>>> from math import sin
```

这个声明不会把整个math模块导入到当前的命名空间中，它只会将math里的sin函数引入进来。

```
>>> from math import sin
>>> sin(60)
-0.3048106211022167
>>> cos(60)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'cos' is not defined
```

### from … import * 语句

把一个模块的所有内容全都导入到当前的命名空间也是可行的，只需使用如下声明：

```
from modname import *
```

这提供了一个简单的方法来导入一个模块中的所有项目。然而这种声明不该被过多地使用。

```
>>> from math import *
>>> sin(60)
-0.3048106211022167
```

### 待补充。。。

## 变量

在Python中，对象变量的类型可以通过直接赋值来创建，而无需事先声明变量名称及其类型。这适用于任何类型的 Python 对象。Python是一种强类型编程语言。Python解释器将自动推断变量类型。Python也是一种动态类型语言，变量的类型可以随时更改。

```
>>> x=3
>>> type(x)
<class 'int'>
>>> x='str'
>>> type(x)
<class 'str'>
```

在Python中，允许多个变量指向相同的值，例如：

```
>>> x = 3
>>> id(x)
1786684560
>>> y = x
>>> id(y)
1786684560
```

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-04/图片.png)

```
>>> x += 6
>>> id(x)
1786684752
>>> y
3
>>> id(y)
1786684560
```

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-04/图片-1.png)

Python采用基于价值的内存管理。如果为不同的变量分配相同的值，则此值的一个副本仅保存在内存中，这可以减少内存空间的占用并提高内存利用率。 当Python启动时，它将以[-5，256]间隔缓存整数。也就是说，如果多个变量的值在 [- 5， 256] 范围内，则这些变量共享相同值的内存空间。 对于区间 [-5， 256] 之外的整数，同一程序中同一语句中具有相同值和不同名称的变量将共享相同的内存空间。

```
>>> x = -6
>>> y = -6
>>> id(x)==id(y)
False
>>> x = -5
>>> y = -5
>>> id(x) == id(y)
True

>>> x = 256
>>> y = 256
>>> id(x) == id(y)
True
>>> x = 257
>>> y = 257
>>> id(x) == id(y)
False

>>> x = 3.0
>>> y = 3.0
>>> id(x) == id(y)
False

>>> x, y = 300000, 300000
>>> id(x) == id(y)
True  

>>> x = [666666, 666666]
>>> y = (666666, 666666)
>>> id(x[0]) == id(x[1])
True
>>> id(y[0]) == id(y[1])
True
>>> id(x[0]) == id(y[0])
False
```

赋值语句的执行过程为： 首先，计算等号右侧表达式的值，然后在内存中找到一个位置来存储该值，最后创建一个变量并指向内存地址。 Python具有自动管理内存的功能。它将跟踪所有值，并自动删除不再使用或引用 0 次的值。

定义变量名称时，应注意以下问题： 

- 变量名称必须以字母，中文字符或下划线开头，但以下划线开头的变量在Python中具有特殊含义; 
- 变量名称不能包含空格和标点符号（方括号（）、引号“、、。 : ? \ /等）; 
- 关键字不能用作变量名称。使用print（keyword.kwlist）查看所有Python关键字; 
- 不建议使用内置模块名称、类型名称或函数名称。使用dir（*builtins*）查看所有内置模块，类型和函数; 
- 变量名称对英文字母的大小写很敏感。

### 数字

十六进制整数以 0x 开头，例如 0x10、0xfa。 

八进制整数以 0o 开头，例如 0o35 和 0o11 

二进制整数以 0b 开头，例如 0b101 和 0b100

浮点数也称为小数 15.0、0.37、-11.2、1.2e2、314.15e-2

Python 3.6.X开始支持在数字中间使用单个下划线作为分隔符，以提高数字的可读性，类似于逗号的数学用法，如千位分隔符

```
>>> 1_000_000
1000000
>>> 1_2_3_4
1234
>>> 1_2 + 3_4j
(12+34j)
>>> 1_2.3_45
12.345
```