# Python复习（六）面向对象

面向对象编程（OOP）主要针对大规模软件设计提出，这使得软件设计更加灵活，可以很好地支持代码重用和设计重用，并使代码更具可读性和可扩展性。

面向对象编程的一个关键概念是封装数据和对数据的操作，以形成一个相互依赖的对象。合理定义面向对象和面向类的关键是如何在这些面向对象类之间形成相同的抽象关系。

Python完全采用了面向对象编程的思想。它是一种真正的面向对象的高级动态编程语言。它完全支持面向对象的基本功能，例如封装，继承，多态以及覆盖或重写基类的方法。

Python中对象的概念非常广泛。Python中的所有内容都可以称为对象。除了数字、字符串、列表、元组、字典、集合、范围对象、zip 对象等之外，函数也是对象，类也是对象。

创建类时，以变量形式表示的对象属性称为数据成员，以函数形式表示的对象行为称为成员方法，成员属性和成员方法统称为类的成员。

## Define class

Python 使用 class 关键字来定义类。

```
class ClassName:
   '类的帮助信息'   #类文档字符串
   class_suite  #类体
```

定义类后，它可用于实例化对象并通过以下方式访问其数据成员或成员方法 "`object name.Member`"

在 Python 中，可以使用内置方法 `isinstance()` 来测试对象是否是类的实例。

```
>>>isinstance(car, Car)         
True
>>>isinstance(car, str)        
False
```

### self parameter

类的实例的方法与普通的函数只有一个特别的区别——它们必须有一个额外的**第一个参数名称**, 按照惯例它的名称是 self。相当于C++中的this，对象本身将作为第一个参数隐式传递，所以在通过实例调用时不必传入，但是需要在定义类的方法的时候写出来。

self代表类的实例，而非类，所以如果该类不打算实例化成对象，就无需self。（自己的理解）

```
class Tool:
    def sayHi():
        print("Hi")

Tool.sayHi()
```

### Class members and instance members

#### Properties

由于Python声明变量的特殊性，属于**实例的数据成员**一般定义在constructor `__init__()` , 只能通过对象名称访问。

属于某个**类的数据成员**是在类中的所有方法之外定义的，可以通过类名或对象名进行访问。

Python对象存储对类的数据成员存储其引用。(不知道准不准确，以后学完python一些基础概念再来改)

如果实例没有对某个属性进行修改，类就相当于一个静态的变量，通过类名和对象名访问到的是同一个地址，就像“连接”起来一样。看个例子：

```
class Car:
    price = 100000                     #Define class properties
    def __init__(self, c):
        self.color = c                 #Define instance properties

car1 = Car("Red")                      #Instantiate object
car2 = Car("Blue")
print(car1.color, Car.price)           
class Car:
    price = 100000                     #Define class properties
    def __init__(self, c):
        self.color = c                 #Define instance properties

car1 = Car("Red")                      #Instantiate object
car2 = Car("Blue")
print(car1.color, Car.price)           
#the values of instance properties and class properties
#car1.price    Car.color(wrong)
Car.price = 110000             #Modify class properties
Car.name = 'QQ'             #Dynamically add class attributes
car1.color = "Yellow"        #Modify instance properties
print(car2.color, car2.price, car2.name)
print(car1.color, Car.price, Car.name)
```

> 运行结果：
>
> Red 100000
> Blue 110000 QQ
> Yellow 110000 QQ

可以看到，执行了 `Car.price = 110000` 之后，通过已经生成的实例 `car2` 访问到的 `price` 是修改后的值 `110000`，而不是定义在类中的 `100000` ，而且也能通过实例访问新添加的 `name` 成员。

但是如果某个实例对某个类的数据成员进行赋值拷贝，那么该实例就会新存储一个修改后的数据成员，以后通过该实例访问这个数据成员访问的也都是修改后的数据，像是断掉了“连接”。而其他实例或对应的类不受影响。当然，还是可以通过内建属性 `__class__` 来访问：`car1.__class__.price`

```
class Car:
    price = 100000                     #class properties
    def __init__(self, c):
        self.color = c

car1 = Car("Red")                      #Instantiate object
car2 = Car("Blue")

car1.price=1

print(car1.price,car2.price,Car.price) #output:1 100000 100000
```

这个网站可以可视化内存布局：https://pythontutor.com/live.html#mode=edit

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-04/截屏2022-04-27-16.00.47.png)

如果类的数据成员存储的列表，并且通过对象调用一些列表的方法来修改，该对象还是对类的数据成员有“连接”

```
class T:
    value=[1,2,3]
t1=T()
t2=T()
t1.value=[4,5,6]
t2.value.append(114)
```

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-04/截屏2022-04-27-16.13.33.png)

#### Methods

和动态添加类的数据成员一样，Python的动态允许我们动态地向自定义类及其对象添加新的方法，通常称为mixin机制，这在大型项目开发中非常方便实用。

一般使用types.MethodType来实现动态添加方法到类或者实例。用MethodType将方法绑定到类，并不是将这个方法直接写到类内部，而是在内存中创建一个link指向外部的方法，在创建实例的时候这个link也会被复制。

```
class Car:
    price = 100000                     #Define class properties
    def __init__(self, c):
        self.color = c                 #Define instance properties

car1 = Car("Red")                      #Instantiate object
car2 = Car("Blue")

import types
def setSpeed(self, s): 
    self.speed = s

#Dynamic member adding method
Car.setSpeed = types.MethodType(setSpeed, Car) #绑定方法到类，实例可调用
car2.setSpeed = types.MethodType(setSpeed, car2) #绑定方法到实例，只有当前被绑定的实例可调用


car1.setSpeed(50)       #call class method
car2.setSpeed(50)       #call member method
print(car1.speed,car2.speed) #output:50 50
```

和上面的数据成员一样，实例的方法如果被修改，也会断开“连接”。

```
class T:
    def show(self):
        print('T')
def show1(self):
        print('show1')
def show2(self):
        print('show2')
t1=T()
t2=T()

import types
t1.show=types.MethodType(show1,t1)

t1.show() #output:show1 修改t1的show(),断开“连接”,其他show()不受影响
t2.show() #output:T

T.show=types.MethodType(show2,T)

t1.show() #output:show1 此时修改T类的show()，t1不受影响
t2.show() #output:show2
```

### **Private and public members**

Python 不为私有成员提供严格的访问保护，但可用下划线得到伪私有。

默认情况下，Python中的成员函数和成员变量都是公开的(public)，在python中没有类似public,private等关键词来修饰成员函数和成员变量。

在python中定义私有变量只需要在变量名或函数名前加上两个下划线，那么这个函数或变量就是私有的了。 

```
>>> class A():
...     def __init__(self):
...         self.__name='python'
... 
>>> instance=A()

>>> instance.__name #类外部不能直接访问私有成员
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'A' object has no attribute '__name'

>>> instance._A__name #但是变量名前加上"_类名"就可以
'python'
```

究其原因：在内部，python使用一种 name mangling 技术，将私有变量名 `__membername` 替换成`_classname__membername`，也就是说，类的内部定义中, 所有以双下划线开始的名字都被"翻译"成前面加上单下划线和类名的形式。 例如：为了保证不能在class之外访问私有变量，Python会在类的内部自动的把我们定义的__spam私有变量的名字替换成为 _classname__spam(注意，classname前面是一个下划线，spam前是两个下划线)，因此，用户在外部访问__spam的时候就会 提示找不到相应的变量。 

- `_xxx` :单下划线开始的成员变量叫做保护变量，只有类实例和子类实例能访问到这些变量，需通过类提供的接口进行访问。不能用 `from module import *` 导入
- `__xxx` :双下划线开始的是私有成员变量，只有类对象自己能访问，连子类对象也不能访问到这个数据。
- `__xxx__` :系统定义名字， 前后均有一个“双下划线” 代表python里特殊方法专用的标识，如 `__init__()` 代表类的构造函数。

## Method

类中定义的方法大致可分为四类：实例方法、类方法和静态方法。每种方法也有都可以定义为私有的形式。

### Instance methods

通常情况下，在类中定义的方法默认都是实例方法。前文，我们已经定义了不只一个实例方法。不仅如此，类的构造方法理论上也属于实例方法，只不过它比较特殊。

```
class Car():
    def __init__(self,name):
        self.name=name
    def show(self):
        print(self.name)
```

可以通过类对象直接调用，例如：

```
car1=Car("AE86")
car1.show()
```

Python 也支持使用类名调用实例方法，但此方式需要手动给 self 参数传值。例如：

```
car1=Car("AE86")
Car.show(car1)
```

### Class methods

Python 类方法和实例方法相似，它最少也要包含一个参数，只不过类方法中通常将其命名为 cls，Python 会自动将类本身绑定给 cls 参数（注意，绑定的不是类对象）。也就是说，我们在调用类方法时，无需显式为 cls 参数传参，与self一样。

和实例方法最大的不同在于，类方法需要使用 `＠classmethod` 修饰符进行修饰，例如：

```
class Car():
    
    def __init__(self,name):
        self.name=name
    
    @classmethod
    def showNum(cls):
        print('114514')
```

类方法推荐使用类名直接调用，当然也可以使用实例对象来调用（不推荐）。例如，在上面 Car 类的基础上，在该类外部添加如下代码：

```
#使用类名直接调用类方法
Car.showNum()

#使用类对象调用类方法
car1=Car('AE86')
car1.showNum()
```

### Static methods

静态方法，其实就是我们学过的函数，和函数唯一的区别是，静态方法定义在类这个空间（类命名空间）中，而函数则定义在程序所在的空间（全局命名空间）中。

静态方法没有类似 self、cls 这样的特殊参数，因此 Python 解释器不会对它包含的参数做任何类或对象的绑定。也正因为如此，类的静态方法中无法调用任何类属性和类方法。

静态方法需要使用`＠staticmethod`修饰，例如：

```
class Car():
    
    def __init__(self,name):
        self.name=name
    
    @staticmethod
    def Info():
        print('Design by tawnx in tawnx.com')
```

静态方法的调用，既可以使用类名，也可以使用类对象，例如：

```
#使用类名直接调用类方法
Car.Info()

#使用类对象调用类方法
car1=Car('AE86')
car1.Info()
```

还有些东西没写完，先发布