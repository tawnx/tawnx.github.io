# Python复习（三）String

## 编码方式

**GB2312** is a Chinese code formulated in China, which uses one byte to represent English and two bytes to represent Chinese; **GBK is an extension of GB2312**, and CP936 is a coding method developed by Microsoft on the basis of GBK. GB2312, GBK and CP936 all use two bytes to represent Chinese. 

**UTF-8** encodes the characters needed by all countries in the world. One byte represents English characters (compatible with ASCII), **three bytes represents common Chinese characters**, and **some language symbols use two bytes** (such as Russian and Greek symbols) or four bytes. 

There are great differences between different coding formats. Adopting different coding formats means different representation and storage forms. **We must understand the coding rules and decode it correctly**.

```
>>>x='python程序开发'.encode('utf8')
>>> x
b'python\xe7\xa8\x8b\xe5\xba\x8f\xe5\xbc\x80\xe5\x8f\x91'
>>> x.decode('utf8')
'python程序开发'
>>> x.decode('cp936')
Traceback (most recent call last):
  File "<pyshell#5>", line 1, in <module>
    z=x.decode('cp936')
UnicodeDecodeError: 'gbk' codec can't decode byte 0x80 in position 14: illegal multibyte sequence

>>> z='python程序开发'.encode('gbk')
>>> t=z.decode('gbk')
>>> t
'python程序开发'
```

Python 3. X fully supports Chinese characters and **uses UTF8 encoding format by default**. Whether it is a number, English letter or Chinese character, it is treated and processed as one character when counting the length of the string.

```
>>> s = '中国山东烟台'
>>> len(s)                
6
>>> s = '中国山东烟台ABCDE'  
>>> len(s)
11
>>> 姓名 = '张三'           
>>> print(姓名)           
张三
```

## 格式化

### %格式化

%格式化字符串是python最早的，也是能兼容所有版本的一种字符串格式化方法，在一些python早期的库中，建议使用%格式化方式，他会把字符串中的格式化符按顺序后面参数替换，用法和C语言的printf函数一样，格式：

```
"%[-][+][0][m][.n]format chars"%(value1, value2 ...)
```

| **format characters** | **explain**                                                  |
| --------------------- | ------------------------------------------------------------ |
| %s                    | string (use str() to display)  such as：'hello'              |
| %r                    | string (use repr() to display) such as：'''hello''' #'''\'hello\'''' |
| %c                    | Single character                                             |
| %d                    | Decimal integer                                              |
| %i                    | Decimal integer  #There is no obvious difference             |
| %o                    | Octal integer                                                |
| %x                    | Hexadecimal integer                                          |
| %e                    | Index number (The base is written as e)                      |
| %E                    | Index number (The base is written as E)                      |
| %f、%F                | Float number #6 decimal places                               |
| %g                    | Index number(e) or float number                              |
| %G                    | Index number(E) or float number #There is no obvious difference |
| %%                    | %                                                            |

Example:

```
>>> x = 1235

>>> "%o" % x
"2323"
>>> "%x" % x
"4d3"
>>> "%e" % x
"1.235000e+03"
>>> "%s" % 65
'65'
>>> "%s" % 65333
'65333'
>>> "%d" % "555"
TypeError: %d format: a number is required, not str
>>> '%s'% [1, 2, 3] #Convert objects directly to strings
'[1, 2, 3]'
```

### format方法

%虽然强大，但用起来难免有些麻烦，代码也不是特别美观，因此，在python 2.5 之后，提供了更加优雅的`str.format()`方法。

待补充。。。