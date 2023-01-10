访问题目地址看到部分源码

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-30-下午8.14.30.png)

大概就是要先登陆admin，用户名和密码直接get传入参数。序列化然后base64编码一下放到cookie里。但是admin用户的密码是本地变量secret：

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-30-下午8.17.52.png)

admin的密码secret是不知道的，但是有pickle反序列化，可以变量覆盖。传入的pickle_data里又不能出现secret关键字，可以用双层exec绕过。

```
opcode=b'''(S'exec('admin.se'+'cret="123456"')'
i__builtin__
exec
.'''

print('payload_orgin:',(opcode))
print('payload:',base64.b64encode(opcode))
```

先进行一次登陆，方便修改cookie的值

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-30-下午8.23.08.png)

然后上面payload进行base64编码后，写入cookie的userdata里。发送一次请求到/balancer页面。

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-30-下午8.24.40.png)

此时admin.secret的值已经被修改成我们想要的了，访问登陆页面，然后访问/balancer，通过注释发现nginx配置文件

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-30-下午8.29.08.png)

查了一下，关键在这lua的负载均衡

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-30-下午8.30.17.png)

联想到之前看过的bilibili 7.13事件，加上题目提示的504 page，传入权重为0，提交，等待几十秒

