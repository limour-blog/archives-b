---
title: 复旦高性能计算 (一) Linux无root权限安装软件：以R为示例
tags: []
id: '631'
categories:
  - - 生物信息学
  - - 超算基础
date: 2021-09-19 19:08:22
---

## 第一步 ssh登录到终端

推荐使用 `[FinalShell](https://www.hostbuf.com/)` 或 [`MobaXterm`](https://mobaxterm.mobatek.net/)

或者 `PowerShell` 也可以用，下面是 `PowerShell` 优化SSH连接体验的方法

*   打开 C:\\Users 找到自己的 home，比如我是C:\\Users\\11248
*   新建 C:\\Users\\11248\\.ssh 目录，或打开已有的这个目录
*   新建 config 配置文件，并且没有后缀名
*   记事本打开 C:\\Users\\11248\\.ssh\\ config
*   添加内容 `ServerAliveInterval 60` 并保存

[![](https://img.limour.top/archives_2023/blog_wp/2021/09/image.webp)](https://img.limour.top/archives_2023/blog_wp/2021/09/image.webp)

*   按下 `win+R` 快捷键
*   输入 `powershell` 确定
*   输入 `ssh userName@server` 回车，如 `pi@192.168.1.2` 或 `pi@raspberrypi`

## 第二步 建立相关文件夹

*   bin: `mkdir ~/zlliu/bin`
*   lib: `mkdir ~/zlliu/lib`
*   include: `mkdir ~/zlliu/include`
*   源码存放位置`libsource`：`mkdir ~/zlliu/libsource`
*   软件存放位置`dev`：`mkdir ~/zlliu/dev`

## 第三步 安装编译器

*   下载 [https://mirrors.ustc.edu.cn/gnu/gcc/gcc-11.1.0/gcc-11.1.0.tar.gz](https://mirrors.ustc.edu.cn/gnu/gcc/gcc-11.1.0/gcc-11.1.0.tar.gz) 并通过ssh上传到 `libsource` 目录
*   打开 `libsource` 目录：`cd ~/zlliu/libsource`
*   解压 `[gcc-11.1.0.tar.gz](https://mirrors.ustc.edu.cn/gnu/gcc/gcc-11.1.0/gcc-11.1.0.tar.gz)`：`tar -zvxf gcc-11.1.0.tar.gz`
*   进入 `gcc-11.1.0` 目录，创建 `gcc-build` 目录：`cd gcc-11.1.0 && mkdir gcc-build`
*   安装依赖库：`./contrib/download_prerequisites`
*   编译前配置：`cd gcc-build && ../configure --prefix=$HOME/zlliu/dev/gcc11 --enable-checking=release --enable-languages=c,c++,fortran,d,go,lto,objc,obj-c++ --disable-multilib`
*   编译并安装：`make -j8 && make install`
*   创建动态链接1：`ln -s /home/rqzhang/zlliu/dev/gcc11/lib64/lib* ~/zlliu/lib/`
*   创建动态链接2：`ln -s /home/rqzhang/zlliu/dev/gcc11/bin/* ~/zlliu/bin/`
*   创建动态链接3：`ln -s /home/rqzhang/zlliu/dev/gcc11/include/* ~/zlliu/include/`
*   修改 `~/.bash_profile` 文件，添加如下内容：  
    [Makefile编译选项CC与CXX/CPPFLAGS、CFLAGS与CXXFLAGS/LDFLAGS](https://www.cnblogs.com/lidabo/p/6068448.html)

```
export PATH=$HOME/zlliu/bin:$HOME/zlliu/lib:$HOME/zlliu/include:$PATH:$HOME/.local/bin:$HOME/bin

export LD_LIBRARY_PATH=$HOME/zlliu/lib/
export LIBRARY_PATH=$HOME/zlliu/lib/ 
export CPLUS_INCLUDE_PATH=$HOME/zlliu/include/
export C_INCLUDE_PATH=$HOME/zlliu/include/
export BOOST_ROOT=$HOME/zlliu/dev/boost

export CC=$HOME/zlliu/bin/gcc
export CXX=$HOME/zlliu/bin/g++
export FC=$HOME/zlliu/bin/gfortran

export CXXFLAGS="-ggdb -pipe -Wall -pedantic -I/home/rqzhang/zlliu/include" 
export CPPFLAGS='-I/home/rqzhang/zlliu/include'
export LDFLAGS='-L/home/rqzhang/zlliu/lib -Wl,-R/home/rqzhang/zlliu/lib'
```

*   激活配置：`source .bash_profile`
*   测试是否成功：`gcc --version`

## 第四步 编译安装R

*   下载R语言安装文件：[https://mirrors.tuna.tsinghua.edu.cn/CRAN/](https://mirrors.tuna.tsinghua.edu.cn/CRAN/)
*   上传解压进入并创建r-build目录
*   编译配置：``../`configure` `` `--prefix=$HOME/zlliu/dev/r4 --enable-R-shlib --with-x=no` `--with-cairo`
*   编译并安装：`make -j8 && make install`
*   创建动态链接1：`ln -s /home/rqzhang/zlliu/dev/r4/bin/* ~/zlliu/bin/`
*   创建动态链接2：``ln -s /home/rqzhang/zlliu/dev/`r4`/lib64/R/lib/lib* ~/zlliu/lib/``
*   测试是否成功：`R --version`

## 第五步 依次解决编译过程中的依赖缺失报错

*   修改 **`config.site`** 文件  
    `nano -K ../**config.site**`  
    `CPPFLAGS='-I/es01/paratera/sce2622/include'  
    LDFLAGS='-L/es01/paratera/sce2622/lib'`
*   configure: error: --with-readline=yes (default) and headers/libs are not available  
    下载源码：[https://ftp.gnu.org/gnu/readline/?C=M;O=D](https://ftp.gnu.org/gnu/readline/?C=M;O=D)  
    `tar -zxvf readline-master.tar.gz  
    ./configure --prefix=$HOME/dev/readline`  
    `make -j8 && make install`  
    `ln -s /es01/paratera/sce2622/dev/readline/lib/lib* ~/lib/  
    ln -s /es01/paratera/sce2622/dev/readline/include/* ~/include/`  
    下载源码：[https://ftp.gnu.org/gnu/termcap/](https://ftp.gnu.org/gnu/termcap/)  
    安装同上
*   configure: error: bzip2 library and headers are required  
    下载源码：[https://sourceforge.net/projects/bzip2/files/](https://sourceforge.net/projects/bzip2/files/)  
    `make && make install PREFIX=$HOME/dev/bzip2`  
    `ln -s /es01/paratera/sce2622/dev/bzip2/lib/lib* ~/lib/  
    ln -s /es01/paratera/sce2622/dev/bzip2/bin/* ~/bin/  
    ln -s /es01/paratera/sce2622/dev/bzip2/include/* ~/include/`
*   configure: error: "liblzma library and headers are required"  
    下载源码：[https://tukaani.org/xz/](https://tukaani.org/xz/)  
    安装同上
*   configure: error: PCRE2 library and headers are required, or use --with-pcre1 and PCRE >= 8.32 with UTF-8 support  
    下载源码：[http://www.pcre.org/](http://www.pcre.org/)  
    安装同上
*   configure: error: libcurl >= 7.28.0 library and headers are required with support for https  
    确保有 openssl  
    下载源码：[https://curl.haxx.se/download.html](https://curl.haxx.se/download.html)   
    `whereis openssl`  
    `./configure --with-ssl --prefix=$HOME/dev/curl`  
    安装同上
*   编译安装新版make  
    下载源码：[https://ftp.gnu.org/gnu/make/](https://ftp.gnu.org/gnu/make/)  
    安装同上
*   编译安装binutils  
    下载源码：[https://ftp.gnu.org/gnu/binutils/](https://ftp.gnu.org/gnu/binutils/)  
    安装同上
*   libbz2.a(blocksort.o): warning: relocation against `memset@@GLIBC_2.2.5' in read-only section`.text'  
    修改bzip2-1.0.6的Makefile文件   
    CC=gcc ---> CC=gcc -fPIC  
    LDFLAGS= -L/es01/paratera/sce2622/lib -Wl,-R/es01/paratera/sce2622/lib  
    重编译 bzip2
*   libtermcap.a(tparam.o): relocation R\_X86\_64\_32 against \`.rodata' can not be used when making a shared object; recompile with -fPIC  
    处理同上  
    

## 第七步 删除无效软连接

```
#! /bin/bash

read path

if [ -z $path ] 
then
    echo "please enter scan path"
    exit 
fi 
for file in $(find $path -type l) 
do
    if [ ! -e $file ]
    then
        echo "rm $file"
        rm -f $file
    fi 
done 
```