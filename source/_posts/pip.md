---
title: pip 安装包指定头文件和静态库目录
date: "2018-04-24 16:09:02"
categories: "Python"
tags: ["python", "pip", "pypi"]
desc: 在pip中使用build_ext标识指定编译的目录
toc: true
---

## 开发中遇到的问题

1. `pip` 指定`include` 和`lib` 目录

   在指定目录时需要添加标识 `build_ext`，例如：

   `sudo pip install --global-option=build_ext --global-option="-I/usr/local/include/" --global-option="-L/usr/local/lib"  <you package name>`

 <!-- more -->  
 
   其中`-L` 指定 `lib ` 文件，

   ```
   Options for 'build_ext' command:
     --build-lib (-b)     directory for compiled extension modules
     --build-temp (-t)    directory for temporary files (build by-products)
     --plat-name (-p)     platform name to cross-compile for, if supported
                          (default: linux-x86_64)
     --inplace (-i)       ignore build-lib and put compiled extensions into the
                          source directory alongside your pure Python modules
     --include-dirs (-I)  list of directories to search for header files
                          (separated by ':')
     --define (-D)        C preprocessor macros to define
     --undef (-U)         C preprocessor macros to undefine
     --libraries (-l)     external C libraries to link with
     --library-dirs (-L)  directories to search for external C libraries
                          (separated by ':')
     --rpath (-R)         directories to search for shared C libraries at runtime
     --link-objects (-O)  extra explicit link objects to include in the link
     --debug (-g)         compile/link with debugging information
     --force (-f)         forcibly build everything (ignore file timestamps)
     --compiler (-c)      specify the compiler type
     --swig-cpp           make SWIG create C++ files (default is C)
     --swig-opts          list of SWIG command line options
     --swig               path to the SWIG executable
     --user               add user include, library and rpath
     --help-compiler      list available compilers
   ```

   > https://stackoverflow.com/questions/18783390/python-pip-specify-a-library-directory-and-an-include-directory

2. 基于Pycharm远程调试

   > * https://blog.csdn.net/zhaihaifei/article/details/53691873
   > * https://www.xncoding.com/2016/05/26/python/pycharm-remote.html

   ​