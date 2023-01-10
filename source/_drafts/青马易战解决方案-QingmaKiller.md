# 青马易战解决方案-QingmaKiller

## 前言

免责声明：本文所涉及内容仅供学习和交流 请勿用于其他用途

## 抓包分析

易班内青马易战授权之后会跳转到一个链接：http://qm.linyisong.top/yiban-web/stu/homePage.jhtml?verify_request=789e7\*\*\*\*\*1d1&yb_uid=545\*\*\*\*8，以下称为“青马易战链接”。通过浏览器修改UA后可以正常访问，免去了手机端抓包的麻烦。

### 获取题目

先回简单回答几道题就可以发现获取题目请求和响应

[![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2021-12/image.png)](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2021-12/image.png)

[![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2021-12/image-2.png)](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2021-12/image-2.png)

选项，题目，类型都有了，但是不知道uuid是什么

### 提交题目

[![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2021-12/image-3.png)](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2021-12/image-3.png)

[![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2021-12/image-4.png)](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2021-12/image-4.png)

[![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2021-12/image-5.png)](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2021-12/image-5.png)

uuid在这里使用，不同请求相同的题目uuid不同，猜测可能跟用户和题目和答题时间等绑定

### 获取Cookie

根据之前的分析，可以发现需要使用到一个cookie。尝试直接访问易班内青马易战的链接，从返回头里获取setcookie。这种方法在一段时间内是可行的，但是12月初左右，访问此链接会显示一个叫“青马易战推荐”的界面而不是真正的青马易战。虽然返回头里有setcookie，但是使用这个cookie不能直接获取题目。

经过简单的分析，发现：获得到setcookie之后，只要在请求头加上此cookie再次请求青马易战链接，就可以使其变为有效的cookie

```
def updateCookie(self):
        headers = {
            'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9',
            'Accept-Encoding': 'gzip, deflate',
            'Accept-Language': 'zh-CN,zh;q=0.9',
            'Host': 'qm.linyisong.top',
            'Proxy-Connection': 'keep-alive',
            'Upgrade-Insecure-Requests': '1',
            'User-Agent': 'Mozilla/5.0 (Linux; Android 6.0; Nexus 5 Build/MRA58N) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.55 Mobile Safari/537.36'
        }
        res = requests.get(self.url, headers=headers)
        headers['Cookie']=re.match(r'JSESSIONID=\w*',res.headers['set-cookie']).group()
        res = requests.get(self.url, headers=headers)
        self.cookie=headers['Cookie']
```

## 逻辑构建

获取Cookie->通过Cookie获取题目->数据库/第三方搜题接口搜题->提交答案->获取响应/加入数据库->回到第二步

## 代码实现

略