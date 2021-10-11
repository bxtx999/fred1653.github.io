---
title: "[译]All About Thread-Local Storage"
date: 2021-10-11 11:31:04
tags: ["线程", "Thread-Local Storage", "线程局部存储"]
categories: ["OS", "Linux"]
desc: 线程本地存储（TLS）提供了一种为不同线程分配不同对象的机制。
toc: true
---

线程本地存储（TLS）提供了一种为不同线程分配不同对象的机制。它是GCC扩展`__thread`、C11 `_Thread_local`和C++11 `thread_local`的通常实现，它们允许使用声明的名称来指代与当前线程相关的实体。本文将详细描述ELF平台上的线程本地存储，并触及其他相关话题，如：线程特定数据键和Windows/MacOS TLS。

<!-- more -->

线程本地存储的一个例子是POSIX `errno`。

> Each thread has its own thread ID, scheduling priority and policy, errno value, floating point environment, thread-specific key/value bindings, and the required system resources to support a flow of control.

> 每个线程都有自己的线程ID、调度优先级和策略、`errno`值、浮点环境、线程特定的键/值绑定，以及支持控制流的所需系统资源。

不同的线程有不同的`errno`副本。`errno`通常被定义为一个函数，它返回一个线程本地的变量。

对于每个架构，权威的ELF ABI文件是System V ABI（通用ABI）的处理器补充（psABI）。这些文件通常参考Ulrich Drepper的[The ELF Handling for Thread-Local Storage](https://www.uclibc.org/docs/tls.pdf)。然而，该文件混合了一般的规范和glibc内部的内容。

## Representation

### 汇编程序的行为


编译器通常在`.tdata`和`.tbss`部分定义线程本地变量（这些部分的标志是`SHF_TLS`）。代表线程本地变量的符号具有`STT_TLS`类型（代表线程本地存储实体）。在GNU as语法中，你可以用`.type a`、`@tls_object`给一个类型的`STT_TLS`。一个TLS符号的`st_value`值是相对于定义部分的偏移。


```x86asm
.section .tbss,"awT",@nobits
.globl a, b
.type a, @tls_object
.type b, @tls_object
a:
  .zero 4
  .size a, .-a
b:
  .zero 4
  .size b, .-b
```

在这个例子中，`st_value(a)=0`同时`st_value(b)=4`。

在Clang和GCC产生的汇编中，线程局部变量被注释为`.type a`、`@object（STT_OBJECT）`。当汇编器看到这种符号被定义在`SHF_TLS`部分或被TLS重定位所引用时，`STT_NOTYPE/STT_OBJECT`将被升级为`STT_TLS`。

GNU支持一个指令`.tls_common`，它定义了`STT_TLS`的`SHN_COMMON`符号。这是一个不明显的特征。目前还不清楚GCC是否仍有一个发出`.tls_common`指令的代码路径。LLVM集成汇编器不支持`.tls_common`。

### 链接器行为

链接器将`.tdata`输入部分合并为`.tdata`输出部分。`.tbss`的输入部分被组合成一个`.tbss`的输出部分。两个`SHF_TLS`的输出部分被放置在`PT_TLS`的程序头中。

- `p_offset`: TLS初始化镜像的文件偏移量

- `p_vaddr`: TLS初始化映像的虚拟地址

- `p_filesz`: TLS初始化映像的大小

- `p_memsz`: 线程本地存储的总大小。最后的`p_memsz-p_filesz`字节将被动态加载器清零。

- `p_align`：对齐方式

`PT_TLS`程序头包含在`PT_LOAD`程序头中。如果使用`PT_GNU_RELRO`、`PT_TLS`被包含在一个`PT_GNU_RELRO`中，而`PT_GNU_RELRO`被包含在一个`PT_LOAD`中。概念上`PT_TLS`和`STT_TLS`符号就像在一个独立的地址空间。动态加载器应该把TLS初始化 image 的`[p_vaddr,p_vaddr+p_filesz)`复制到相应的静态TLS块中。

在可执行文件和共享对象文件中，`st_value`通常持有一个虚拟地址。对于一个`STT_TLS`符号，`st_value`持有相对于`PT_TLS`程序头的虚拟地址的偏移。`PT_TLS`的第一个字节是由`st_value==0`的TLS符号引用的。

GNU ld 将 `STT_TLS SHN_COMMON` 符号视为定义在 `.tcommon` 部分。它的内部链接器脚本将这些部分放到输出部分`.tdata`中。LLD 不支持 `STT_TLS SHN_COMMON` 符号。

### 动态加载器行为

动态加载器从主可执行文件和立即加载的共享对象中收集`PT_TLS`程序头文件（通过过渡的`DT_NEEDED`），并分配静态TLS块，每个`PT_TLS`一个块。对于每个`PT_TLS`，动态加载器从TLS初始化镜像中复制`p_filesz`字节到TLS块中，并将尾部的`p_memsz-p_filesz`字节设置为零。

对于主可执行文件的静态TLS块，模块ID是1，TLS符号的TP偏移量是一个链接时间常数。链接器和动态加载器共享相同的公式。

对于在程序开始时加载的共享对象，从线程指针到其静态TLS块的偏移量在程序开始时是一个固定值，尽管不是一个链接时常数。该偏移量可以被初始执行的TLS模型所使用的GOT动态重定位所引用。

ELF对线程本地存储的处理描述了两种TLS的变体，并指定了它们的数据结构。然而，只有主可执行文件的静态TLS块的TP偏移是一个硬性要求。尽管如此，libc的实现通常将静态TLS块放在一起，并为线程控制块和静态TLS块分配了一个空间。

对于一个由`pthread_create`创建的新线程，静态TLS块通常被分配为线程堆栈的一部分。如果在堆栈的最大地址和线程控制块之间没有一个保护页，这可能被认为是脆弱的，因为堆栈溢出可以覆盖线程控制块。

## 模型

### 本地执行 TLS 模型（执行与非抢占）

这是最有效的TLS模型。它适用于在可执行文件中定义TLS符号的情况。

编译器在`-fno-pic/fpie`模式下选择这种模式，如果变量是

- 一个定义

- 或具有非默认可见性的声明。

第一个条件是显而易见的。第二个条件是由于非默认可见性意味着该变量必须由可执行文件中的另一个翻译单元定义。

```c++
_Thread_local int def;
__attribute__((visibility("hidden"))) extern thread_local int ref;
int foo() { return def + ref; }
```

```x86asm
# x86-64
movl %fs:def@TPOFF, %eax
```

## 原文

原文为：[All about thread-local storage](https://maskray.me/blog/2021-02-14-all-about-thread-local-storage)
