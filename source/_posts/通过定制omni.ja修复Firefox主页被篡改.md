---
title: 通过定制omni.ja修复Firefox主页被篡改
date: 2022-2-28 22:31:38
tags: 
- Firefox
categories: 
- 记录
---

# 通过定制omni.ja修复Firefox主页被篡改

## 前言

几个月前，偶然机会用到火狐浏览器，感觉一些地方比Chrome做的好。在此之前，由于宝塔面板没有使用https，而Chrome会自动将宝塔的http链接定向（？不晓得怎么描述）到https，导致无法正常使用，而且关掉一段时间会自动打开。虽然很早就下载了火狐，但是使用简体中文语言包的火狐会不定期把主页修改成北京某流氓公司做的firefox.cn页面，与这里描述的问题一致：https://www.v2ex.com/t/811824，再加上收藏夹密码等都在Chrome，就一直没转用Firefox。但是今天Chrome再次自动https，我直接angry，便有了今天这篇文章。

## 发现

尝试了谷歌，换开发者版，国际版都不好用，换语言包可以解决，但是繁体中文看起来还是有点难受。

最终在一个帖子的回下找到了正解：https://www.v2ex.com/t/576632

![Screenshot 2022-12-15 at 10.58.53 PM](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-12/Screenshot%202022-12-15%20at%2010.58.53%20PM.png)

## 操作步骤

首先找到omni.ja：macos下如图

[![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-02/截屏2022-02-28-15.12.31.png)](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-02/截屏2022-02-28-15.12.31.png)

复制omni.ja到某个文件夹，这样比较好操作并且不会损坏源文件。然后需要解压缩，貌似win下改后缀成zip解压缩再用windows自带压缩工具压缩回去时可以的，因为我记得我上次就是这么操作的。但是既然使用macos，就直接用官方推荐的方法：

```
unzip omni.ja
```

打开终端，cd到放omni.ja的目录，使用上面的unzip命令：

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-12/%E6%88%AA%E5%B1%8F2022-02-28-15.45.06.png)

解压后应该如图所示（最好把这个omni.ja删了，方便后面的压缩）：

[![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-02/截屏2022-02-28-15.15.08.png)](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-02/截屏2022-02-28-15.15.08.png)

然后找到browser.propertie

[![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-02/截屏2022-02-28-15.15.51.png)](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-02/截屏2022-02-28-15.15.51.png)

用文本编辑器打开，拉到最下方，把browser.startup.homepage改成你想要的主页。最好不要使用windows自带的文本编辑器，这里使用vscode：

[![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-02/截屏2022-02-28-15.16.19.png)](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-02/截屏2022-02-28-15.16.19.png)

编辑好后保存退出，在当时存放解压文件的目录下用zip命令压缩打包（之前一直把它们放在omni目录下，打包整个目录，导致firefox启动报错）（如果之前没删omni.ja就得修改命令）：

```
zip -qr9XD omni.ja *
```

[![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-02/截屏2022-02-28-15.17.08.png)](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-02/截屏2022-02-28-15.17.08.png)

然后当前目录下就生成一个omni.ja，用它来替换最初的omni.ja就好啦，记得备份。