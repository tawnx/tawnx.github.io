## EASY-PHP01

根据注释里的提示，传入hint参数，然后根据php的特性，传入post参数

![image-20221104165721573](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-11/image-20221104165721573.png)

## EASY-PHP02

根据特性传入参数

![image-20221104170401548](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-11/image-20221104170401548.png)

然后把前半部分ascii对应 后半部分base64解码即可获得flag

## FakeWeb

curl 题目地址即可

![image-20221104170745986](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-11/image-20221104170745986.png)

## simplephp

str参数url编码，num参数前面加个%0c即可绕过is_numeric

![image-20221104171103912](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-11/image-20221104171103912.png)

## 猫和老鼠

普通的反序列化，关键在于怎么绕过dog，可以设置把b设置为a的引用，这样就可以通过dog后面的那句赋值。

```php
$a=new mouse();
$a->v='php://filter/read=convert.base64-encode/resource=flag.php';
$b=new cat();
$b->b=&$b->a;
$b->c=$a;
$b->b=$b->c;

$str=serialize($b);
echo urlencode($str);
```

## curl

Flag.php必须本地访问，然后根据注释里的代码，传入urls参数 指向flag.php即可

![image-20221104171742489](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-11/image-20221104171742489.png)





www.zip获得源码

![image-20221104172439831](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-11/image-20221104172439831.png)

不能直接看到回显，反弹shell也不行，然后发现www.zip可以读取app.py。只要将flag写入app.py即可

先写入

![image-20221104173102522](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-11/image-20221104173102522.png)

在读取

![image-20221104173036315](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-11/image-20221104173036315.png)