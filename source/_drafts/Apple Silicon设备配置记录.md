# Apple Silicon设备配置记录

## 编程开发

### 工具

### 语言

#### Java环境

目标是做到多种版本的java环境共存, 并且可以随意切换。

原先尝试使用Homebrew来安装java, 但是官方不能安装java8，而第三方AdoptOpenJDK只有x86的版本，需要安装rosetta。所以这里手动下载安装，然后进行配置。这里下载的是azul的版本，其他Oracle，Amazon版本同理。

安装完成后，到 `/Library/Java/JavaVirtualMachines` 目录下，就能看到已经安装的jvm了。![Screen Shot 2022-10-08 at 19.09.29](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-10/Screen%20Shot%202022-10-08%20at%2019.09.29.png)

接下来要配置一下路径，以便随意切换java版本。

将 `/Library/Java/JavaVirtualMachines/<Your JVM folder name>/Contents/Home` 中的 `<Your JVM folder name>` 换成你的JVM文件夹名称，替换完的路径就是对应版本的 `JAVA_HOME` 环境变量的值。

下面是一个示范，使用alias简化替换 `JAVA_HOME` 的命令。

```
# Java config
export JAVA_8_HOME="/Library/Java/JavaVirtualMachines/zulu-8.jdk/Contents/Home"
export JAVA_11_HOME="/Library/Java/JavaVirtualMachines/zulu-11.jdk/Contents/Home"

# config alias
alias jdk8="export JAVA_HOME=$JAVA_8_HOME"
alias jdk11="export JAVA_HOME=$JAVA_11_HOME"
```

将上述配置好的命令写入 `~/.zshrc` ，使用 `jdk<X>` 就能切换版本。

![Screen Shot 2022-10-08 at 19.13.44](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-10/Screen%20Shot%202022-10-08%20at%2019.13.44.png)

#### Markdown写作

**Typora+uPic+github图床**

Typora原先是一个免费软件，但是在

uPic是一个图片上传软件，截止到本文写作

官网：https://blog.svend.cc/

安装

```
brew install bigwig-club/brew/upic --cask
```



