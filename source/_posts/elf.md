---
title: 【译】Linux系统中的 ELF 文件的理解与分析
date: 2021-02-17 23:29:35
tags: ["elf"]
categories: ["OS", "Linux"]
desc: ELF是可执行和可链接格式（Executable and Linkable Format）的缩写，定义了二进制文件、库和核心文件的结构。正式的规范允许操作系统正确解释其底层机器指令。
toc: true
---

世界上一些真正的工匠精神，我们认为是理所当然的。其中之一就是 Linux 上常用的工具，比如`ps`和`ls`。尽管这些命令可能被认为是简单的，但当看清其本质时，却有更多的东西。这就是ELF或可执行和可链接格式的作用。一个用得很多的文件格式，但真正了解的人却寥寥无几。让我们通过这个介绍教程来了解一下吧!

<!-- more -->

通过阅读本指南，你将了解：

- 为什么要使用 ELF，以及使用在什么样的文件上？

- 了解 ELF 的结构和格式的细节。

- 如何读取和分析 ELF 文件，如二进制文件。

- 哪些工具可以用于二进制分析。

## 什么是 ELF 文件？

ELF 是可执行和可链接格式（Executable and Linkable Format）的缩写，定义了二进制文件、库和核心文件的结构。正式的规范允许操作系统正确解释其底层机器指令。ELF 文件通常是编译器或链接器的输出，是一种二进制格式。通过合适的工具，可以对这类文件进行分析，更好地理解。

为什么要学习 ELF 的细节？

在深入了解更多技术细节之前，不妨先解释一下为什么了解 ELF 格式是有用的。作为一个入门者，它有助于了解我们操作系统的内部运作。当出现问题时，我们可能会更好地理解发生了什么（或为什么）。然后是能够研究 ELF 文件的价值，特别是在安全漏洞或发现可疑文件后。最后但同样重要的是，为了在开发时更好地理解。即使你使用像Golang这样的高级语言编程，你仍然可能从了解幕后发生的事情中受益。

### 为什么学习 ELF？

- 对操作系统工作原理的一般理解

- 软件的开发

- 数字取证和事件响应(DFIR)

- 恶意软件研究（二进制分析）

### 从源头到过程

所以无论我们运行的是什么操作系统，都需要将常用的函数翻译成 CPU 的语言，也就是机器代码。一个函数可能是一些基本的东西，比如在磁盘上打开一个文件或在屏幕上显示一些东西。我们不是直接与 CPU 对话，而是使用一种编程语言，使用内部函数。然后编译器将这些函数翻译成对象代码。然后，通过使用链接器工具，将这些对象代码链接成一个完整的程序。其结果是一个二进制文件，然后可以在特定的平台和 CPU 类型上执行。

### 在你开始之前

这篇博文将分享很多命令。不要在生产系统上运行它们。最好在测试机上做。如果你喜欢测试命令，可以复制一个现有的二进制文件，然后使用它。另外，我们还提供了一个小的 C 程序，你可以编译一下。毕竟，尝试是学习和比较结果的最好方法。

## ELF文件的结构

一个常见的误解是，ELF 文件只是用于二进制文件或可执行文件。我们已经看到它们可以用于部分片段（对象代码）。另一个例子是共享库甚至核心转储（那些`core`或`a.out`文件）。ELF规范在Linux上也用于内核本身和Linux内核模块。

![coredump](https://cdn.jsdelivr.net/gh/jnhu76/Image-Hosting/img/20210217161516.png)

### 架构

由于 ELF 文件的可扩展设计，每个文件的结构不同。一个 ELF 文件由以下几个部分组成

1. ELF header

2. 文件数据

通过`readelf`命令，我们可以查看一个文件的结构，它看起来是这样的。

![Details of an ELF binary](https://cdn.jsdelivr.net/gh/jnhu76/Image-Hosting/img/20210217161428.png)

#### ELF header

从这个截图中可以看到，ELF header 以一些魔术数字开始。这个 ELF header 的魔术数字提供了关于文件的信息。前4个十六进制部分定义了这是一个ELF文件(45=**E**,4c=**L**,46=**F**)，前缀为**7f**值。

这个 ELF header 是强制性的。它确保数据在链接或执行过程中被正确解释。为了更好地理解 ELF文件的内部工作，了解这个 header 信息的使用是很有用的。

##### Class

在ELF类型声明之后，定义了一个 `Class` 字段。这个值决定了文件的体系结构，可以是 32 位(`=01`)或64位(`=02`)的体系结构。它可以是**32-bit**（=01）或 **64-bit**（=02）的架构。魔法显示的是02，它被`readelf`命令翻译成ELF64文件。换句话说，是一个使用64位架构的 ELF 文件。这并不奇怪，因为这台特殊的机器包含一个现代 CPU。

##### Data

接下来的部分是数据字段。它知道两个选项。`01`代表**LSB**([Least_significant_bit](https://en.wikipedia.org/wiki/Least_significant_bit)), 也就是小字段. 然后是值`02`，代表**MSB**（Most Significant Bit，big-endian）。这个特殊的值有助于正确解释文件中的其余对象。这一点很重要，因为不同类型的处理器对输入的指令和数据结构的处理方式不同。在这种情况下，使用 LSB，这对于 AMD64 类型的处理器来说是很常见的。

当对二进制文件使用`hexdump`时，LSB 的效果就会显现出来。让我们展示一下`/bin/ps`的 ELF header 细节。

```bash
$ hexdump -n 16 /bin/ps
0000000 457f 464c 0102 0001 0000 0000 0000 0000

0000010
```

我们可以看到，value pair 是不同的，这是由于字节顺序的正确解释造成的。

##### Version

接下来在魔法中又多了一个`01`。这就是版本号。目前，版本类型只有1个：当前，也就是值`01`。所以没什么有趣的东西可记。

##### OS/ABI

每个操作系统的共同功能都有很大的重叠。此外，他们每个人都有特定的，或者至少他们之间有小的差异。正确集的定义是通过**应用二进制接口**（[ABI](https://en.wikipedia.org/wiki/Application_binary_interface)）来完成的。这样操作系统和应用程序都知道期望什么，功能也能正确转发。这两个字段描述了使用什么ABI和相关的版本。在这种情况下，值是`00`，这意味着没有使用特定的扩展。输出显示为[System V](https://en.wikipedia.org/wiki/UNIX_System_V)。

##### Machine

我们还可以在头文件中找到预期的机器类型（`AMD64`）。

##### Type

类型字段告诉我们文件的目的是什么。有几种常见的文件类型。

- CORE（值4)

- DYN(Shared object file)，用于 library (值3)

- EXEC(Executable file)，用于二进制文件(值2)

- REL(Relocatable file)，在链接到可执行文件之前(值1)

#### See full header details

虽然有些字段已经可以通过`readelf`输出的魔术数字来显示，但还有更多。例如对于文件是什么特定的处理器类型。使用`hexdump`我们可以看到完整的 ELF header 及其值。

*通过`hexdump -C -n 64 /bin/ps`创建的输出内容*

```bash
7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 |.ELF............|
02 00 3e 00 01 00 00 00 a8 2b 40 00 00 00 00 00 |..>......+@.....|
40 00 00 00 00 00 00 00 30 65 01 00 00 00 00 00 |@.......0e......|
00 00 00 00 40 00 38 00 09 00 40 00 1c 00 1b 00 |....@.8...@.....|
```

第二行第三组是定义机器类型的。值`3e`是十进制的`62`，相当于`AMD64`。要了解所有机器类型，请看一下这个 ELF 头文件。

虽然你可以用十六进制转储做很多事情，但让工具为你做这些工作是有意义的。在这方面，`dumpelf`工具可以提供帮助。它显示的格式化输出与 ELF 头文件非常相似。很好地了解到使用了哪些字段以及它们的典型值。

明确了所有这些字段后，是时候看看真正的神奇发生在哪里，并进入下一个头文件了!

#### File data

除了 ELF 头，ELF 文件还包括三个部分。

- **Program Headers** or **Segments** (9)

- **Section Headers** or **Sections** (28)

- **Data**

在我们深入研究这些头文件之前，很高兴知道ELF有两个互补的 "视图（view）"。一个view 用于链接器允许执行（segment）。另一个用于对指令和数据进行分类（section）。因此，根据目标的不同，会使用相关的头类型。让我们从程序头开始，我们可以在ELF二进制文件上找到它。

#### 程序头文件

一个 ELF 文件由0个或多个段组成，描述了如何为运行时执行创建一个进程/内存映像。当内核看到这些段时，它会使用它们来映射到虚拟地址空间，使用`mmap(2)`系统调用。换句话说，它将预定义的指令转换为内存映像。如果你的 ELF 文件是一个正常的二进制文件，它需要这些程序头。否则，它根本无法运行。它使用这些头文件和底层数据结构，形成一个进程。这个过程与共享库类似。

![An overview of program headers in an ELF binary](https://cdn.jsdelivr.net/gh/jnhu76/Image-Hosting/img/20210217161348.png)

我们在这个例子中看到，有9个程序头文件。当第一次看的时候，很难理解这里发生了什么。所以我们来了解一些细节。

##### GNU_EH_FRAME*

这是 GNU C 编译器（gcc）使用的一个排序队列。它存储了异常处理程序。所以当出现问题时，它可以使用这个区域来正确处理。

##### GNU_STACK

这个头是用来存储栈信息的。栈是一个缓冲区，或者说是一个划痕的地方，在这里存储项目，就像本地变量一样。这将与 LIFO（Last In, First Out） 一起发生，类似于把盒子放在彼此的顶部。当一个进程函数启动时，会保留一个块。当函数完成后，它将再次被标记为空闲。现在有趣的部分是，堆栈不应该是可执行的，因为这可能会引入安全漏洞。通过对内存的操作，人们可以引用这个可执行的栈并运行预期的指令。

如果`GNU_STACK`段不可用，那么通常会使用可执行栈。`scanelf`和`execstack`工具是两个显示栈细节的例子。

```bash
# scanelf -e /bin/ps
 TYPE   STK/REL/PTL FILE 
ET_EXEC RW- R-- RW- /bin/ps

# execstack -q /bin/ps
- /bin/ps
```

查看程序 header 文件的命令：

- `dumpelf(pax-utils)`

- `elfls -S /bin/ps`

- `eu-readelf –program-headers /bin/ps`

#### ELF sections

##### Section headers

Section header 定义了文件中的所有章节。如前所述，这个 "view"是用来链接和重新定位的。

段可以在一个 ELF 二进制文件中找到，在 GNU C 编译器将 C 代码转化为汇编后，再由 GNU 汇编器将其创建对象。

如上图所示，一个段可以有0个或多个段。对于可执行文件来说，主要有四个部分：`.text`、`.data`、`.rodata`和`.bss`。每一个部分都加载了不同的访问权限，可以用`readelf -S`来查看。

##### .text

包含可执行代码。它将被打包到一个具有读取和执行访问权限的段中。它只被加载一次，因为内容不会改变。这可以通过`objdump`工具看到。

##### .data

初始化的数据，具有读/写访问权。

##### .rodata

初始化的数据，只有读取访问权（=A）。

##### .bss

未初始化的数据，具有读/写访问权(=WA)

> [24] .data PROGBITS 00000000006172e0 000172e0  
> 0000000000000100 0000000000000000 **WA** 0 0 8  
> [25] .bss NOBITS 00000000006173e0 000173e0  
> 0000000000021110 0000000000000000 **WA** 0 0 32

*查看 section 和 header 的命令：*

- `dumpelf`

- `elfls -p /bin/ps`

- `eu-readelf -section-headers /bin/ps`

- `readelf -S /bin/ps`

- `objdump -h /bin/ps`

##### Section groups

有些部分可以分组，因为它们形成一个整体，或者换句话说是一个依赖关系。新的链接器支持这个功能。不过，这种情况还是不常见，不常发现。

```bash
# readelf -g /bin/ps


```

*在该文件下没有 section group。*

虽然这看起来可能不是很有趣，但它显示了研究现有的 ELF 工具包的明显好处，以供分析。为此，本文末尾列入了对工具及其主要目标的概述。

### 静态和动态二进制文件

在处理 ELF 二进制文件时，最好知道有两种类型以及它们之间的联系。类型是静态或动态的，指的是使用的库。出于优化的目的，我们经常看到二进制文件是 "动态 "的，这意味着它需要外部组件才能正确运行。通常这些外部组件是普通的库，其中包含了一些常用的功能，比如打开文件或创建网络套接字。而静态二进制文件则包含了所有的库。这使得它们更大，但更容易移植（例如在另一个系统上使用它们）。

如果你想检查一个文件是静态还是动态编译的，使用`file`命令。如果它显示类似

```bash
$ file /bin/ps
/bin/ps: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.24, BuildID[sha1]=2053194ca4ee8754c695f5a7a7cff2fb8fdd297e, stripped
```

要确定正在使用哪些外部库，只需在同一二进制文件上使用`ldd`即可。

```bash
$ ldd /bin/ps
linux-vdso.so.1 => (0x00007ffe5ef0d000)
libprocps.so.3 => /lib/x86_64-linux-gnu/libprocps.so.3 (0x00007f8959711000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f895934c000)
/lib64/ld-linux-x86-64.so.2 (0x00007f8959935000)
```

**提示**：要查看底层的依赖关系，最好使用  `lddtree` 工具。

## 二进制分析的工具

当你想分析 ELF 文件时，首先寻找可用的工具肯定是有用的。一些可用的软件包提供了逆向工程二进制文件或可执行代码的工具包。如果你是分析ELF恶意软件或固件的新手，可以考虑先学习**静态分析（static analysis)**。这意味着你检查文件而不实际执行它们。当您更好地了解它们的工作方式时，然后转向**动态分析(dynamic analysi)**。现在，您将运行文件样本，并查看它们作为实际处理器指令执行低级代码时的实际行为。无论你做什么类型的分析，都要确保在一个专用系统上进行，最好是有严格的网络规则。在处理未知样本或那些与恶意软件有关的样本时，尤其如此。

### 常用工具

#### Radare2

[Radare2](https://www.radare.org/r/) 工具包由 Sergi Alvarez 创建。版本中的`2`指的是与第一个版本相比，该工具的全面重写。现在很多逆向工程师都用它来学习二进制文件的工作原理。它可以用来剖析固件、恶意软件以及其他任何看起来是可执行格式的东西。

#### Software packages

大多数 Linux 系统已经安装了`binutils`包。其他的软件包可能会帮助显示更多的细节。拥有正确的工具包可能会简化你的工作，特别是在做分析或了解更多关于 ELF 文件的时候。因此，我们收集了一个软件包和其中的相关实用程序的列表。

##### elfutils

- /usr/bin/eu-addr2line
- /usr/bin/eu-ar – alternative to ar, to create, manipulate archive files
- /usr/bin/eu-elfcmp
- /usr/bin/eu-elflint – compliance check against gABI and psABI specifications
- /usr/bin/eu-findtextrel – find text relocations
- /usr/bin/eu-ld – combining object and archive files
- /usr/bin/eu-make-debug-archive
- /usr/bin/eu-nm – display symbols from object/executable files
- /usr/bin/eu-objdump – show information of object files
- /usr/bin/eu-ranlib – create index for archives for performance
- /usr/bin/eu-readelf – human-readable display of ELF files
- /usr/bin/eu-size – display size of each section (text, data, bss, etc)
- /usr/bin/eu-stack – show the stack of a running process, or coredump
- /usr/bin/eu-strings – display textual strings (similar to strings utility)
- /usr/bin/eu-strip – strip ELF file from symbol tables
- /usr/bin/eu-unstrip – add symbols and debug information to stripped binary

*注：`elfutils`包是一个很好的开始，因为它包含了大多数执行分析的实用程序。*

##### elfkickers

- /usr/bin/ebfc – compiler for [Brainfuck](https://en.wikipedia.org/wiki/Brainfuck) programming language
- /usr/bin/elfls – shows program headers and section headers with flags
- /usr/bin/elftoc – converts a binary into a C program
- /usr/bin/infect – tool to inject a dropper, which creates setuid file in /tmp
- /usr/bin/objres – creates an object from ordinary or binary data
- /usr/bin/rebind – changes bindings/visibility of symbols in ELF file
- /usr/bin/sstrip – strips unneeded components from ELF file

*注：ELFKickers包的作者专注于对ELF文件的操作，当你发现畸形的ELF二进制文件时，不妨多学习一下。*

##### pax-utils

- /usr/bin/dumpelf – dump internal ELF structure
- /usr/bin/lddtree – like ldd, with levels to show dependencies
- /usr/bin/pspax – list ELF/PaX information about running processes
- /usr/bin/scanelf – wide range of information, including PaX details
- /usr/bin/scanmacho – shows details for Mach-O binaries (Mac OS X)
- /usr/bin/symtree – displays a leveled output for symbols

*备注：本软件包中的几个实用程序可以在整个目录中递归扫描。非常适合对一个目录进行大规模分析。这些工具的重点是收集PaX的细节。除了支持ELF之外，还可以提取一些关于Mach-O二进制文件的细节。*

输出示例

```bash
scanelf -a /bin/ps
 TYPE    PAX   PERM ENDIAN STK/REL/PTL TEXTREL RPATH BIND FILE 
ET_EXEC PeMRxS 0755 LE RW- R-- RW-    -      -   LAZY /bin/ps
```

##### prelink

- /usr/bin/execstack – display or change if stack is executable
- /usr/bin/prelink – remaps/relocates calls in ELF files, to speed up the process

### Example

如果你想自己创建一个二进制程序，只需创建一个小的C程序，然后编译它。下面是一个例子，它打开`/tmp/test.txt`，将内容读入缓冲区并显示出来。确保创建相关的`/tmp/test.txt`文件。

```c
#include <stdio.h>

int main(int argc, char **argv)
{
   FILE *fp;
   char buff[255];

   fp = fopen("/tmp/test.txt", "r");
   fgets(buff, 255, fp);
   printf("%s\n", buff);
   fclose(fp);

   return 0;
}
```

这个程序通过`gcc -o test test.c`进行编译。

## 常见问题解答

### 什么是ABI？

ABI 是 Application Binary Interface 的缩写，指定了操作系统和一段可执行代码之间的低级接口。

### 什么是ELF？

ELF是可执行和可链接格式的简称。它是一种正式的规范，定义了如何在可执行代码中存储指令。

### 如何确定未知文件的文件类型？

使用文件命令进行第一轮分析。该命令可能会根据头信息或魔术数字来显示细节。

## 结论

ELF 文件是用于执行或链接的。根据主要的目标，它包含了所需的段或节。段被内核查看并映射到内存中(使用`mmap`)，段被链接器查看以创建可执行代码或共享对象。段被链接器查看以创建可执行代码或共享对象。

ELF 文件类型非常灵活，支持多种CPU类型、机器架构和操作系统。它还具有很强的可扩展性：根据所需的部分，每个文件的构造不同。

头文件构成了文件的重要部分，准确描述了 ELF 文件的内容。通过使用正确的工具，你可以获得对文件目的的基本理解。从那里，你可以进一步检查二进制文件。这可以通过确定它使用的相关函数或文件中存储的字符串来完成。对于那些从事恶意软件研究的人来说，或者想更好地了解进程的行为（或不行为！），这是一个很好的开始。

## 更多资源

如果你想了解更多关于 ELF 和逆向工程的知识，你可能会喜欢我们在 Linux 安全专家所做的工作。作为培训计划的一部分，我们有一个[带有实际实验室任务的逆向工程模块](https://linuxsecurity.expert/training/domains/reverse-engineering)。

对于那些喜欢阅读的人来说，一个好的深度文档。[ELF格式](http://www.skyfree.org/linux/references/ELF_Format.pdf)和由[Brian Raiter](http://www.muppetlabs.com/~breadbox/software/ELF.txt)（ELFkickers）撰写的文件。对于那些喜欢阅读实际源代码的人来说，可以看看苹果公司的一个文档化的[ELF结构头文件](http://www.opensource.apple.com/source/dtrace/dtrace-90/sys/elf.h)。

**小贴士**：如果你喜欢在分析文件和样本方面有更好的表现，那就开始使用流行的[二进制分析工具](https://linuxsecurity.expert/security-tools/binary-analysis-tools)吧。

-----

[Magic data / Magic number](https://zh.wikipedia.org/wiki/%E9%AD%94%E8%A1%93%E6%95%B8%E5%AD%97_(%E7%A8%8B%E5%BC%8F%E8%A8%AD%E8%A8%88)):

- 缺乏解释或命名的独特数值。常常在程序中出现多次，并且可以（从规范上而言也应当）被有名字的常量取代。

- 用于识别一个文件格式或协议类型的一段常量或字符串，例如UNIX的特征签名。

- 不易于其他值混淆的值，例如 UUID。

魔术数字也会在文件中使用。在特定文件格式中加入固定数值和固定字符串，然后便可以通过检查文件是否包含这些数据来快速地识别文件格式。

例如：GIF 文件开头会包含`GIF89a`（`47 49 46 38 39 61`）或`GIF87a`（`47 49 46 38 37 61`）这两种字符串。

> 原为为[The 101 of ELF files on Linux: Understanding and Analysis - Linux Audit (linux-audit.com)](https://linux-audit.com/elf-binaries-on-linux-understanding-and-analysis/#what-is-an-elf-file)
