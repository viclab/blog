title: YouCompleteMe插件再编译机环境再安装记录
date: 2016-02-16 08:22:52
tags:
---
由于最近更换项目组，编译机之前配置好的VIM的IDE环境也需要迁移，由于YouCompleteMe插件
需要重新编译，而且新的编译机所依赖的库的情况也不同，所以重新编译插件时有几个问题，在此
予以记录。

# 编译YouCompleteMe插件
之前在151的编译机上全新编译YouCompleteMe插件，由于编译机不能联网，所有的依赖都需要手动
拷贝，再源码安装，过程很费事费力，这里有全新安装的[教程地址](https://www.zybuluo.com/viclab/note/199525)
考虑到所有费时费力的安装其实都只是为了得到llvm+clang的库，而YouCompleteMe插件编译安装主要
也只是依赖这些库而已，所有借着这次迁移环境的机会尝试一下直接从上次已经编译好的clang库来安装
YouCompleteMe插件。
首先，依照[教程](http://blog.csdn.net/leaf5022/article/details/21290509)中自定义指定自定义的clang库
路径。
```bash
cd ~/.vim/YouCompleteMe
cmake -G "Unix Makefiles" -DEXTERNAL_LIBCLANG_PATH=~/ycm/lib/libclang.so ~/.vim/bundle/YouCompleteMe/cpp
```
发现出错
```bash
CMake Error at /usr/share/cmake-2.8/Modules/FindPackageHandleStandardArgs.cmake:108          
(message):
Could NOT find PythonLibs (missing: PYTHON_LIBRARIES PYTHON_INCLUDE_DIRS)
```
很显然，这里是cmake找不到系统的python库，所以需要指定，那么如何指定呢，一番波折之后，按照如下方式指定即可。
```bash
cmake -G "Unix Makefiles" -DEXTERNAL_LIBCLANG_PATH=~/ycm/lib/libclang.so -DPYTHON_LIBRARY=/usr/lib/python2.6 
-DPYTHON_INCLUDE_DIR=/usr/include/python2.6 . ~/.vim/YouCompleteMe/third_party/ycmd/cpp
```
当然DPYTHON_INCLUDE_DIR和DPYTHON_LIBRARY是根据系统本地路径来填写。解决了这个问题后发现又有新的问题，就是在编译过程
中，发现到50%的时候出现了这个错误`Compilation failed : pyconfig.h: No such file or directory`，这里其实缺少python库
的某些头文件，其实很好解决，在centos系列的系统中使用
```bash
yum install python-devel
```
在Ubuntu系列的系统中用
```bash
sudo apt-get install python-dev
```
之后成功编译完成。

# 编译高版本VIM
由于YouCompleteMe插件需要高于7.3版本的VIM，所有需要源码重装VIM，选择[7.4版本](ftp://ftp.vim.org/pub/vim/unix/vim-7.4.tar.bz2)，安装如下：
```bash
./configure --with-features=huge --enable-rubyinterp --enable-pythoninterp 
--with-python-config-dir=/usr/lib64/python2.6/config/ --enable-perlinterp 
--enable-gui=gtk2 --enable-cscope --enable-luainterp --enable-perlinterp 
--enable-multibyte --prefix=/data/home/victorschen/ycm
```
> 需要注意一点，就是一定要有--enable-pythoninterp，开启支持python的选项，否则徒劳。

# 重建链接
由于之前配置YouCompleteMe插件软链接时用的都是绝对路径，所以很多路径配置都失效，导致进入VIM中无法找到YCM的命令。
在.vim文件夹中重建autoload和plugin到YouCompleteMe中对应文件夹的软链接路径即可。其实改为相对路径，以后就不用再配置了。
> 在以后配置软链接中尽量采用相对路径来配置


