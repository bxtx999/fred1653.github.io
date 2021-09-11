---
title: pip源
date: "2017-01-08 17:55:20"
categories: "Python"
tags: ["python", "pip", "pypi"]
desc:
toc: true
---

`Pypi` 即[Python Package Index](https://pypi.python.org/pypi)，是Python官方管理第三方软件库，目前共有96202个包。Python的软件管理工具包括pip等都是用PyPI作为默认软件源和以来。

Pypi的默认镜像地址是`pypi.python.org`

<!-- more -->

### 镜像地址

| Mirror                                   | Location                           | # of packages | last update         |
| ---------------------------------------- | ---------------------------------- | ------------- | ------------------- |
| [pypi.python.org](https://pypi.python.org/) | San Francisco, California US       | 96201         | 2017-01-08 09:48:24 |
| [pypi.douban.com](https://pypi.douban.com/) | Beijing, Beijing CN                | 97126         | 2017-01-08 07:00:14 |
| [pypi.fcio.net](https://pypi.fcio.net/)  | Oberhausen, Nordrhein-Westfalen DE | 97735         | 2017-01-08 09:45:01 |
| [pypi.tuna.tsinghua.edu.cn](https://pypi.tuna.tsinghua.edu.cn/) | Beijing, Beijing CN                |               | Unavailable         |
| [mirror.picosecond.org/pypi](https://mirror.picosecond.org/pypi) | Fremont, California US             |               | Unavailable         |
| [mirrors.aliyun.com/pypi](https://mirrors.aliyun.com/pypi) | Hangzhou, Zhejiang CN              | 97513         | 2017-01-07 19:01:37 |
| [pypi.pubyun.com](https://pypi.pubyun.com/) | Changzhou, Jiangsu CN              | 95514         | 2017-01-08 09:30:03 |
| [mirrors-uk.go-parts.com/python](https://mirrors-uk.go-parts.com/python) | Reston, Virginia US                |               | Unavailable         |
| [mirrors-ru.go-parts.com/python](https://mirrors-ru.go-parts.com/python) | Reston, Virginia US                |               | Unavailable         |
| [mirrors-au.go-parts.com/python](https://mirrors-au.go-parts.com/python) | Reston, Virginia US                |               | Unavailable         |
| [pypi.mirrors.ustc.edu.cn](https://pypi.mirrors.ustc.edu.cn/) | Hefei, Anhui CN                    | 76240         | 2017-01-08 04:48:06 |

> 2017-01-08 18:09:34 更新
>
> 数据来源：https://www.pypi-mirrors.org/

### 使用方法

1. 简单使用

   `pip install -i https://<mirror>/simple <package>`  or `pip install -i http://<mirror>/simple <package>`

2. 全局设置

   * Linux

     添加` ~/.pip/pip.conf`文件，内容如下：

     ```
     [global]
     index-url = https://<mirror>/simple

     [install]
     trusted-host=<mirror>
     ```

   * Windows

     在`C:\Users\<YourPCName>`下建立`.pip`文件夹，并添加`pip.conf`文件。

     文件内容：

     ```
     [global]
     index-url = https://<mirror>/simple

     [install]
     trusted-host=<mirror>
     ```
   > http https任选

3. `easy_install`和`pip`使用方法类似。


### 参考

1. https://www.pypi-mirrors.org/
2. http://mirrors.aliyun.com/help/pypi
3. https://en.wikipedia.org/wiki/Python_Package_Index
4. https://pypi.python.org/pypi