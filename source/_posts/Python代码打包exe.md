---
title: Python代码打包exe
date: 2022-12-20 19:50:38
tags: 
- Python
categories: 
- 编程语言
---

## 一、前言

一年前写了一个脚本，用来刷题，后来需要打包给同学用。当时没写后端来处理题目的增查，而是直接连接的数据库，用户名密码等敏感信息都在代码里，为了信息安全，需要对源码进行打包并且不那么容易反编译。

如果直接使用pyinstaller来打包，很容易就能获得源代码，所以要借助Cython，先把python代码转成c语言代码，再编译发布，这样获取源代码的难度就大得多。

但是网上一般的打包教程会出现一些依赖问题，会导致打包出来的可执行文件闪退，所以摸索并总结了如下内容。

此文根据回忆撰写，可能在细节上有些许错误！有空了再完善补充。

## 二、环境准备

建议升级pip后再进行安装

安装Visual Studio：[https://visualstudio.microsoft.com/vs/older-downloads/](https://visualstudio.microsoft.com/vs/older-downloads/)
安装PyInstaller：`pip install pyinstaller`
安装Cython：`pip install Cyphton`

## 三、打包

1. 已经写好想要打包的文件为app.py，把app.py中的依赖和代码分离，把代码定义为一个函数，删去 `if __name__ == “__main__”` ，如图所示。
![Screenshot 2023-01-03 at 10.38.57 PM.png](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2023-01/Screenshot%202023-01-03%20at%2010.38.57%20PM.png)

2. 在当前目录创建一个setup.py文件，内容如下。
```
from distutils.core import setup
from Cython.Build import cythonize
setup(
        ext_modules = cythonize("app.py") #打包的源码的文件名，这里是app.py
)
```

3. 执行下面命令生成pyd文件
```
python setup.py build_ext --inplace
```

4. 在当前目录创建一个main.py文件，写上原先代码导入的模块，然后再添加如下内容
```
from app import * #app是要打包的文件名
start() #打包代码中定义的函数
```

下面是个例子
![image.png](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2023-01/20230103224535.png)

5. 用pyinstaller打包，命令如下
```
pyinstaller -F .\main.py -i .\s.ico
```
-F指定只生成一个exe文件，-i指定exe的图标，其他参数可以自行查询。
最后，exe文件会生成在当前目录下的dist文件夹中。
