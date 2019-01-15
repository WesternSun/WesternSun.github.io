---
title: Make
date: 2019-01-16 00:15:50
categories:
- Tools
tags:
- tool
- make
---

# Make

## configure, make 和 make install 的区别

* ./configure是用来检测你的安装平台的目标特征的。比如它会检测你是不是有CC或GCC，并不是需要CC或GCC，它是个shell脚本。
* make是用来编译的，它从Makefile中读取指令，然后编译。
* make install是用来安装的，它也从Makefile中读取指令，安装到指定的位置。

### configure
这一步一般用来生成 Makefile，为下一步的编译做准备，你可以通过在 configure 后加上参数来对安装进行控制，比如代码:./configure --prefix=/usr上面的意思是将该软件安装在 /usr 下面，执行文件就会安装在 /usr/bin.同时一些软件的配置文件你可以通过指定 --sys-config= 参数进行设定。有一些软件还可以加上 --with、--enable、--without、--disable 等等参数对编译加以控制，你可以通过允许 ./configure --help 察看详细的说明帮助。

### make
这一步就是编译，大多数的源代码包都经过这一步进行编译（当然有些perl或Python编写的软件需要调用perl或python来进行编译）。make 的作用是开始进行源代码编译，以及一些功能的提供，这些功能由 Makefile 文件体现，比如 make install 一般表示进行安装，make uninstall 是卸载，不加参数就是默认的进行源代码编译。

make 是 Linux 开发套件里面自动化编译的一个控制程序，它通过借助 Makefile 里面编写的编译规范进行自动化的调用 gcc 、ld 以及运行某些需要的程序进行编译的程序。一般情况下，他所使用的 Makefile 控制代码，由 configure 这个配置脚本根据给定的参数和系统环境生成。

### make install

这条命令来进行安装（当然有些软件需要先运行 make check 或 make test来进行一些测试），这一步一般需要你有 root 权限（因为要向系统写入文件）


## CMake

你或许听过好几种 Make 工具，例如 GNU Make ，QT 的 qmake ，微软的 MS nmake，BSD Make（pmake），Makepp，等等。这些 Make 工具遵循着不同的规范和标准，所执行的 Makefile 格式也千差万别。这样就带来了一个严峻的问题：如果软件想跨平台，必须要保证能够在不同平台编译。而如果使用上面的 Make 工具，就得为每一种标准写一次 Makefile ，这将是一件让人抓狂的工作。

CMake就是针对上面问题所设计的工具：它首先允许开发者编写一种平台无关的 CMakeList.txt 文件来定制整个编译流程，然后再根据目标用户的平台进一步生成所需的本地化 Makefile 和工程文件，如 Unix 的 Makefile 或 Windows 的 Visual Studio 工程。从而做到“Write once, run everywhere”。显然，CMake 是一个比上述几种 make 更高级的编译配置工具。

### 使用

在 linux 平台下使用 CMake 生成 Makefile 并编译的流程如下：

1. 编写 CMake 配置文件 CMakeLists.txt 。
2. 执行命令 cmake PATH 或者 ccmake PATH 生成 Makefile。其中， PATH 是 CMakeLists.txt 所在的目录。
3. 使用 make 命令进行编译。