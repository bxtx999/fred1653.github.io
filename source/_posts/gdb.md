---
title: 【译】RMS's gdb Debugger Tutorial —— Example Debugging Sessions
date: 2022-05-02 22:54:30
categories: "编程"
tags: ["GDB"]
toc: true
---

> Unknown -- "Don't worry if it doesn't work right. If everything did, you'd be out of a job."

## 例子

### Infinite Loop Example

```c
#include <stdio.h>
#include <ctype.h>

int main(int argc, char **argv) {
    char c;

    c = fgetc(stdin);
    while (c != EOF) {
        if (isalnum(c))
            printf("%c", c);
        else
            c = fgetc(stdin);
    }

    return 1;
}
```

<!-- more -->

首先，使用 debug 标识符编译源程序。

```sh
gcc -g 1.c
```

运行程序，输入字符后会出现无限循环的错误。

```sh
> ./a.out
mom
mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm^C
```

使用 GDB 调试程序：

```sh
gdb git:(master) ✗ gdb a.out
GNU gdb (Debian 10.1-1.7) 10.1.90.20210103-git
Copyright (C) 2021 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from a.out...
(gdb) r
Starting program: /home/hoo/Projects/codeBase/base/cpp/gdb/a.out
mom
mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm^C
Program received signal SIGINT, Interrupt.
0x00007ffff7ee4f33 in __GI___libc_write (fd=1, buf=0x5555555596b0, nbytes=1024)
    at ../sysdeps/unix/sysv/linux/write.c:26
26      ../sysdeps/unix/sysv/linux/write.c: No such file or directory.
(gdb)
```

程序运行后出现无限循环的问题，使用 Ctrl + C 向程序发送 SIGHT。GDB 将会捕获这个信号并停止程序。

现在我们使用`backtrace`命令查看运行栈。

```sh
(gdb) bt
#0  0x00007ffff7ee4f33 in __GI___libc_write (fd=1, buf=0x5555555596b0, nbytes=1024)
    at ../sysdeps/unix/sysv/linux/write.c:26
#1  0x00007ffff7e76665 in _IO_new_file_write (f=0x7ffff7fb56a0 <_IO_2_1_stdout_>, data=0x5555555596b0,
    n=1024) at fileops.c:1181
#2  0x00007ffff7e759d6 in new_do_write (fp=fp@entry=0x7ffff7fb56a0 <_IO_2_1_stdout_>,
    data=0x5555555596b0 'm' <repeats 200 times>..., to_do=to_do@entry=1024) at libioP.h:948
#3  0x00007ffff7e77709 in _IO_new_do_write (to_do=1024, data=<optimized out>,
    fp=0x7ffff7fb56a0 <_IO_2_1_stdout_>) at fileops.c:423
#4  _IO_new_do_write (fp=fp@entry=0x7ffff7fb56a0 <_IO_2_1_stdout_>, data=<optimized out>, to_do=1024)
    at fileops.c:423
#5  0x00007ffff7e77bdf in _IO_new_file_overflow (f=0x7ffff7fb56a0 <_IO_2_1_stdout_>, ch=109)
    at fileops.c:779
#6  0x00007ffff7e6e2ae in putchar (c=109) at putchar.c:28
#7  0x00005555555551a3 in main (argc=1, argv=0x7fffffffe338) at 1.c:10
```

我们可以看到程序在`__GI___libc_write()`出现问题。查看`main`函数中哪里调用了这个函数，所以使用`frame 7`查看`main`函数。

```sh
(gdb) frame 7
#7  0x00005555555551a3 in main (argc=1, argv=0x7fffffffe338) at 1.c:10
10                  printf("%c", c);
```

从上面的输出来看，我们可以知道 `c`的值可能是`m`。

```sh
(gdb) print c
$1 = 109 'm'
```

现在我们必须找到这个loop。我们使用`next`命令的几个迭代来观察发生了什么。注意，我们必须沿着调用堆栈向上工作，回到 `main()`。下一个命令将退出任何我们不能调试的函数(如 c 库函数)。我们也可以在这里使用 `finish` 命令。

```sh
(gdb) next
mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm_IO_new_file_write (f=0x7ffff7fb56a0 <_IO_2_1_stdout_>, data=0x5555555596b0, n=1024)
    at fileops.c:1182
1182    fileops.c: No such file or directory.
(gdb) next
_IO_new_do_write (fp=fp@entry=0x7ffff7fb56a0 <_IO_2_1_stdout_>,
    data=<optimized out>, to_do=<optimized out>) at fileops.c:423
423     in fileops.c
(gdb) next
_IO_new_file_overflow (f=0x7ffff7fb56a0 <_IO_2_1_stdout_>, ch=109) at fileops.c:781
781     in fileops.c
(gdb) next
782     in fileops.c
(gdb) next
787     in fileops.c
(gdb) next
putchar (c=109) at putchar.c:27
27      putchar.c: No such file or directory.
(gdb) next
main (argc=1, argv=0x7fffffffe338) at 1.c:8
8           while (c != EOF) {
```

现在我们已经进入到 `main` 函数，使用 `next` 命令查看程序如何执行：

```sh
(gdb) next
9               if (isalnum(c))
(gdb) next
10                  printf("%c", c);
(gdb)
8           while (c != EOF) {
(gdb) next
9               if (isalnum(c))
(gdb) next
10                  printf("%c", c);
(gdb) next
8           while (c != EOF) {
(gdb) next
9               if (isalnum(c))
(gdb) next
10                  printf("%c", c);
```

可以看到这几行代码重复执行，其中 `c` 的值并没有改变一直是 `m`。所以肯能是这一段程序出现问题：

```c
if (isalnum(c))
    printf("%c", c);
else
    c = fgetc(stdin);
```

所以，去掉`else` 就可以解决问题。

### Segmentation Fault Example

源码如下：

```c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char **argv) {
    char* buf;

    buf = malloc(1 << 31);
    
    fgets(buf, 1024, stdin);
    printf("%s\n", buf);

    return 1;
}
```

使用命令编译源程序。

```sh
 gcc -g 2.c
2.c: In function ‘main’:
2.c:7:11: warning: argument 1 value ‘18446744071562067968’ exceeds maximum object size 9223372036854775807 [-Walloc-size-larger-than=]
    7 |     buf = malloc(1 << 31);
      |           ^~~~~~~~~~~~~~~
In file included from 2.c:2:
/usr/include/stdlib.h:539:14: note: in a call to allocation function ‘malloc’ declared here
  539 | extern void *malloc (size_t __size) __THROW __attribute_malloc__
      |              ^~~~~~
```

运行程序后，出现`segmentation fault` 错误。

```sh
> ./a.out
123
[2]    2531 segmentation fault  ./a.out
```

现在使用 GDB 调试程序：

```sh
 gdb a.out
GNU gdb (Debian 10.1-1.7) 10.1.90.20210103-git
Copyright (C) 2021 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from a.out...
(gdb) r
Starting program: /home/hoo/Projects/codeBase/base/cpp/gdb/a.out
st

Program received signal SIGSEGV, Segmentation fault.
0x00007ffff7e6ba71 in __GI__IO_getline_info (
    fp=fp@entry=0x7ffff7fb4980 <_IO_2_1_stdin_>, buf=buf@entry=0x0, n=n@entry=1023,
    delim=delim@entry=10, extract_delim=extract_delim@entry=1, eof=eof@entry=0x0)
    at iogetline.c:77
77      iogetline.c: No such file or directory.
```

使用命令查看运行栈。

```sh
(gdb) bt
#0  0x00007ffff7e6ba71 in __GI__IO_getline_info (
    fp=fp@entry=0x7ffff7fb4980 <_IO_2_1_stdin_>, buf=buf@entry=0x0, n=n@entry=1023,
    delim=delim@entry=10, extract_delim=extract_delim@entry=1, eof=eof@entry=0x0)
    at iogetline.c:77
#1  0x00007ffff7e6bb58 in __GI__IO_getline (
    fp=fp@entry=0x7ffff7fb4980 <_IO_2_1_stdin_>, buf=buf@entry=0x0, n=n@entry=1023,
    delim=delim@entry=10, extract_delim=extract_delim@entry=1) at iogetline.c:34
#2  0x00007ffff7e6aa56 in _IO_fgets (buf=0x0, n=1024,
    fp=0x7ffff7fb4980 <_IO_2_1_stdin_>) at iofgets.c:53
#3  0x000055555555518c in main (argc=1, argv=0x7fffffffe338) at 2.c:9
```

查看 frame 3 中的代码，到底是哪里出现问题。

```sh
(gdb) frame 3
#3  0x000055555555518c in main (argc=1, argv=0x7fffffffe338) at 2.c:9
9           fgets(buf, 1024, stdin);
```

是在调用 `fgets` 的时候让程序崩溃了。一般来说，`fgets`的实现是正确的，所以问题出在参数中。其中，`stdin`变量是 `stdio` 库的，也应该是正确的。所以问题可能处在 `buf` 上。

```sh
(gdb) print buf
$1 = 0x0
```

`buf` 的值是 `0x0`，是空指针。在源码中，我们在第 7 行 `buf = malloc(1 << 31);` 分配内存，所以我们需要在 第 7 行设置断点。现在我们需要 kill 当前运行的程序。

```sh
(gdb) kill
Kill the program being debugged? (y or n) y
[Inferior 1 (process 2567) killed]
```

设置断点：

```sh
(gdb) break 2.c:7
Breakpoint 1 at 0x555555555164: file 2.c, line 7.
```

继续运行程序，在断点处暂停：

```sh
(gdb) r
Starting program: /home/hoo/Projects/codeBase/base/cpp/gdb/a.out

Breakpoint 1, main (argc=1, argv=0x7fffffffe338) at 2.c:7
7           buf = malloc(1 << 31);
```

在调用后 `malloc` 后，`buf` 仍然是 `0x0` 空指针。是 `malloc` 参数出了问题。`1 << 31` 是18446744071562067968，超过了9223372036854775807。同时，我们只在 `fgets` 上读取 `1024` bytes 数据。所以我们要把参数修改为`1024` 或者 `1<<9`。

## 资料

1. http://www.unknownroad.com/rtfm/gdbtut/
