---
title: 解决黑苹果iMessage和FaceTime无法登陆
date: 2022-2-14 12:19:38
tags: 
- Hackintosh
categories: 
- 黑苹果
---

# 解决黑苹果iMessage和FaceTime无法登陆

## 问题出现

第一次装黑苹果一切正常，同一个三码，EFI下，重装了一次系统，登陆iMessage的时候就登不上，前几次持续时间还久一点（1分钟左右），之后登陆10s内必定自动退出并弹出登陆页面。同一个Apple ID在另一台白苹果上正常登陆，另一个Apple ID在这台黑苹果上也能正常登陆。

## 尝试解决

重制NVRAM：无效

换过一次三码（序列号查不到）：无效

新建一个管理员账户，在管理员账户内登陆（打电话给客服给的方法）：第一次登陆和之前症状一样，之后显示未知错误

修改uuid（忘记是不是这个）：无效

修改ROM：无效

换成可查询到序列号的三码：无效

## 继续搜索

[![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-02/截屏2022-02-14-下午12.12.05.png)](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-02/截屏2022-02-14-下午12.12.05.png)

在bilibili发现一个视频的评论区发现：这种情况在big sur及以上的系统中不会有提示，直接退出登陆，而Catalina中会有提示客户代码，直接打电话给客服报客户代码就行。

原视频链接：https://www.bilibili.com/video/BV1hi4y1g7PV/



还有在tonymacx86搜到的：

[![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-02/截屏2022-02-14-下午12.16.12.png)](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-02/截屏2022-02-14-下午12.16.12.png)

目前打算先不管，等个几周看看。

原链接：https://www.tonymacx86.com/threads/unable-to-login-to-imessage-on-big-sur.303322/page-2#post-2233046





问题是昨天中午出现的，今天用可查询三码重新装了一下神奇的登上了



3月1日更新：

正常用了一段时间后，发现转发短信不能及时的提醒，于是在系统便好设置里勾选从icloud同步，结果还是不行，今天把账号退出，又登不上去了。

3月9日更新：

前几天突然想起来可能是因为把efi给了别人用，虽然修改了三码，但是没有修改rom值，导致imessage失效，今天按照国光黑苹果群里有个老哥分享的办法试了一下（其实就是换了个rom值），好像又能正常登录了。



> 该老哥分享的方法
>
> 1.请先确保你的三码成功生成即System Serial Number、System UUID、MLB
> 2.生成三码后，PlatformInfo的ROM可以填写en0的MAC地址。方法：系统报告中查看“以太网卡”的“BSD名称”，和“WIFI”的“接口”，笔者的以太网卡BSD名称为en0，即以太网卡的MAC地址就为机型的ROM值。
> 3.确定号en0是wifi还是以太网卡后，在网络偏好设置中查看mac地址：打开网络，选择以太网卡-高级-硬件复制MAC地址（wifi网卡为en0同理）
> 4.填写ROM值，例如我的MAC地址为84:1f:a1:05:40:11，则ROM一栏填写841FA1054011 注意全部为大写
> 5.保存Config.plist
> 6.退出AppleID再重启电脑！！！选择rest nvram重置nvram
> 7.进入系统后登录Apple，再打开FaceTime和iMessage验证是否无法登录和闪退，如果成功的话结束，如果失败继续接下来的教程
> 8.打开终端输入 /System/Applications/Messages.app/Contents/Macos/Messages
> 9.复制终端的报错信息。例如我的错误信息为
> Messages[758:6613] Invalid absolute dimension, must be > 0.0. NOTE: This will be a hard-assert soon, please update your call site
> 顾客代码:5375-2728-3312
> 10.打开Apple支持https://getsupport.apple.com/products 选择你的苹果设备-遇到了哪方面的问题？-App与软件-信息-继续-联系支持人员-聊天
> 11.弹出聊天对话窗口后，等待客服接应，将从终端复制的错误信息以及你的AppleID、绑定的手机号一并提供给客服，再跟客服报告问题即自己的FaceTime无法在Mac上成功登录
> 12.等待客服处理，处理完成后就可以结束聊天重启电脑来验证iMessage和FaceTime是否可以登录，或者等待一个工作日再重新登录