# Python复习（二）List

## 方法

| **Methods**                       | **I****llustrate**                                           |
| --------------------------------- | ------------------------------------------------------------ |
| lst.append(x)                     | Add element x to the end of list lst.                        |
| lst.extend(L)                     | Add all elements in list L to the end of list lst.           |
| lst.insert(index, x)              | Add the element x at the position index by lst in the list, and move all elements after the position one position later. |
| lst.remove(x)                     | Delete the first occurrence of the specified element in the list lst, and move all elements after the element forward one position |
| lst.pop([index])                  | Delete and return the elements whose subscript is index  in the list lst |
| lst.clear()                       | Delete all elements in the list lst, but keep the list object |
| lst.index(x)                      | Returns the subscript of the first element x in the list lst. |
| lst.count(x)                      | Returns the number of occurrences of the specified element x in the list lst |
| lst.reverse()                     | Reverse the order of all elements in the list lst            |
| lst.sort(key=None, reverse=False) | Sort the elements in the list lst. The key is used to specify the sorting basis. Reverse determines whether the order is ascending (false) or descending (true) |
| lst.copy()                        | Returns a shallow copy of the list lst                       |

## 切片

切片是Python序列的重要操作之一，适用于列表、元组、字符串、range对象等类型。

### 基本用法

切片使用2个冒号分隔的3个数字来完成：`[a:b:c]`

- `a` 表示切片的开始位置，默认为0
- `b` 表示切片的截止（但不包含）位置（默认为列表长度）
- `c` 表示切片的步长(默认为1)，当步长省略时，顺便可以省略最后一个冒号。为负就是倒着读。

```
>>> list=[1,2,3,4,5,6]

>>> list[:] #Returns a new list containing all elements
[1, 2, 3, 4, 5, 6]

>>> list[:2] #All elements with subscript between [0, 2)
[1, 2]

>>> list[::2] #Even position, one after another
[1, 3, 5]

>>> list[5:2:-1]
[6, 5, 4]

>>> list[100] #Using subscript access directly will cross the boundary
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: list index out of range
>>> list[100:] #All elements after subscript 100 are automatically truncated
[]
```

### 其他操作：增删改

也可以利用切片实现其他操作：增加、修改、删除

```
>>> aList = [3, 5, 7]

>>> aList[len(aList):] = [9] #Append element at the end
>>> aList
[3, 5, 7, 9]

>>> aList[:3] = [1, 2, 3] #Replace the first 3 elements
>>> aList
[1, 2, 3, 9]

>>> aList[:3] = [] #Delete the first 3 elements
>>> aList
[9]

>>> aList = list(range(10))
>>> aList
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

>>> aList[::2] = [0]*5 #Replace elements at even positions
>>> aList
[0, 1, 0, 3, 0, 5, 0, 7, 0, 9]
>>> aList[::2] = [0]*3    
#The slices are discontinuous, the number of elements on both sides must be the same
ValueError: attempt to assign sequence of size 3 to extended slice of size 5
```

### 复制和浅复制

切片返回的是列表元素的浅复制，与列表对象的直接赋值并不一样。

**复制**：复制前后的列表指向同一内存块，拥有相同的内存地址，其中一个列表改变，另一个列表也会随之改变。二者可以视为同一个对象。

**浅复制** ：二者相等，但是二者不是同一个对象，指向的内存地址也不一样，对一个列表做修改，也不改变另一个列表。

```
>>> alist=[1,2,3]
>>> blist=alist #复制
>>> clist=alist[:] #Slice,light copy. 效果和copy库的deepcopy函数一样

>>> blist is alist
True
>>> clist is alist
False

>>> alist[1]=88 #clist不会被改变
>>> blist
[1, 88, 3]
>>> clist
[1, 2, 3]
```

### 多维数组的访问

多位数组本质上也是一样，每个嵌套的列表也还是一个元素

```
>>> a_list=[[1,2,3,4],
...       [4,5,6,7],
...       [7,8,9,10]]
>>> a_list[1:3]
[[4, 5, 6, 7], [7, 8, 9, 10]]
```

## 排序

### 正向排序

可以使用list自带的方法 `sort` ，也可以使用内置的 `sorted` 函数，它返回一个新的排序过的list。

```
>>> alist=[5,3,1,2,6]

>>> sorted(alist)
[1, 2, 3, 5, 6]
>>> alist #sorted不改变传入的list
[5, 3, 1, 2, 6]

>>> alist.sort()
>>> alist
[1, 2, 3, 5, 6]
```

### 逆向排序

可以使用list自带的方法 `reverse` ，也可以使用内置的 `reversed` 函数，它返回一个新的排序过的list。与上面正向排序相同，不再演示。还可以在 `sort` 或者 `sorted` 函数添加 `reverse=True` 来实现：

```
>>> alist=[5,3,1,2,6]

>>> sorted(alist,reverse=True)
[6, 5, 3, 2, 1]

>>> alist.sort(reverse=True)
>>> alist
[6, 5, 3, 2, 1]
```

## 常见的内置函数

### `all()` and `any()`

`all() `function is used to test whether all elements in sequence objects such as lists and tuples, map and zip objects are equivalent to true. The `any()` function is used to test whether there are elements equivalent to true. For example:

```
>>> all([1,2,3])
True
>>> all([0,1,2,3])
False
>>> any([0,1,2,3])
True
>>> any([0])
False
```

### `len(list)`

`len(list)`：Returns the number of elements in the list, which is also applicable to tuples, dictionaries, sets, strings, etc. 

```
>>> len(2,3,[1,4])
3
```

### `max(list)` and `min(list)`

`max(list)`、 `min(list)`：Returns the largest or smallest element in the list. The same applies to tuples, dictionaries, sets, range objects,  etc. 

```
>>> alist=[2,3,[1,2]]

>>> min(alist) #必须是可比较类型???
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: '<' not supported between instances of 'list' and 'int'
>>> alist.pop()
[1, 2]
>>> min(alist)
2
>>> max(alist)
3
```

### `sum(list)`

`sum(list)`：Sum the elements of the list. For non-numeric list operations, need to specify the start parameter, which is also applicable to tuples and ranges.

```
>>> sum(range(1, 11)) #The start parameter of the sum() defaults to 0
55
>>> sum(range(1, 11), 5) #the start parameter=5, which is equivalent to 5+sum(range(1,11))
60
>>> sum([[1, 2], [3], [4]], [])   
[1, 2, 3, 4]
>>> sum([[1,2],[3],[[4,5],[6,7]]],[])#最后一个参数是说明+的类型
[1, 2, 3, [4, 5], [6, 7]]
```

### `enumerate(list)`

`enumerate(list)`:Enumerate list elements and return enumeration objects, where each element is a tuple containing subscripts and values. This function is also valid for tuples and strings.

```
>>> for item in enumerate('abcdef'):
...     print(item)
... 
(0, 'a')
(1, 'b')
(2, 'c')
(3, 'd')
(4, 'e')
(5, 'f')
```

## List derivation

**List derivation** uses a very simple way to quickly generate a list that meets specific needs, and the code is very readable.

The syntax form of list derivation is: 

```
[expression for expr1 in sequence1 if condition1
    for expr2 in sequence2 if condition2
    for expr3 in sequence3 if condition3
    …
    for exprN in sequenceN if conditionN]
```