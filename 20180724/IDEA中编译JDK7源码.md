编译环境：
* Win7
* IDEA 2017.2.5
* JDK 1.7.0

### 下载JDK7
JDK7官方下载地址：http://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase7-521261.html
Windows平台下，下载后得到的是一个exe安装文件，安装之后，在安装目录下，你将得到一个src.zip文件，它就是JDK的源码文件。我们需要将其解压至任意目录下

### 新建Java Project
打开IDEA，新建一个普通的Java Project即可，然后将解压后的源码目录全部复制到新建Project的src目录下，操作完成之后，如下图所示：

![](https://github.com/yida-lxw/blog/blob/master/20180724/images/_1532362453_11961.png?raw=true)

接下来我们需要设置项目编译环境，通过快捷键Ctrl + Alt + Shift + S打开IDEA的项目结构(即Project Structure)设置，具体设置如下图所示：
![](https://github.com/yida-lxw/blog/blob/master/20180724/images/_1532362770_9715.png?raw=true)

上图主要设置了项目使用的SDK以及项目的语言级别，不同语言级别的语法会有所不同，这里我们是编译JDK7源码，故统一设置为JDK7

![](https://github.com/yida-lxw/blog/blob/master/20180724/images/_1532362899_329.png?raw=true)

![](https://github.com/yida-lxw/blog/blob/master/20180724/images/_1532363216_11431.png?raw=true)

上图设置的每个模块的语言级别为7以及模块使用的SDK版本。然后我们就可以开始尝试进行JDK源码编译啦！请如下图所示进行操作：
![](https://github.com/yida-lxw/blog/blob/master/20180724/images/_1532363369_12574.png?raw=true)

此时可能会提示你：sun.awt.UNIXToolkit not found，此时我们需要从openJDK里找到UNIXToolkit.java文件，复制到我们当前项目里即可，下面是UNIXToolkit.java的获取链接：
[UNIXTookit.java](http://hg.openjdk.java.net/jdk7/jdk7/jdk/file/5f452be1691e/src/solaris/classes/sun/awt/UNIXToolkit.java)
注意：你需要手动新建sun.awt包，然后新建UNIXToolkit.java文件，然后将代码复制过去即可，最后重新编译整个Project即可，直到编译成功。这样我们就可以愉快的Debug源码了，有助于更高效的阅读JDK源码。