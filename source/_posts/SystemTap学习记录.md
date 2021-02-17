---
title: SystemTap学习记录
date: 2020-06-13 19:03:48
tags: ["SystemTap"]
categories: ["OS", "Linux"]
desc: SystemTap 允许用户在不重新编译代码的情况下利用静态追踪、动态追踪工具，比如在任何地方动态插入printk，或者改变内核的关键数据结构（guru模式）。所有的操作都要以`root`用户模式下进行。
---

## SystemTap 工具

SystemTap 允许用户在不重新编译代码的情况下利用静态追踪、动态追踪工具，比如在任何地方动态插入printk，或者改变内核的关键数据结构（guru模式）。所有的操作都要以`root`用户模式下进行。

<!-- more -->

### 安装

```bash
 $ sudo apt install systemtap systemtap-runtime
```

安装 kernel debug symbol

```bash
# 16.04 或更高
$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys C8CAB6595FDFF622 
# 旧版本
$  sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys ECDCAD72428D7C01 
$ codename=$(lsb_release -c | awk  '{print $2}')
$ sudo tee /etc/apt/sources.list.d/ddebs.list << EOF
deb http://ddebs.ubuntu.com/ ${codename}      main restricted universe multiverse
deb http://ddebs.ubuntu.com/ ${codename}-security main restricted universe multiverse
deb http://ddebs.ubuntu.com/ ${codename}-updates  main restricted universe multiverse
deb http://ddebs.ubuntu.com/ ${codename}-proposed main restricted universe multiverse
EOF

$ sudo apt-get update
$ sudo apt-get install linux-image-$(uname -r)-dbgsym
```

### 基础

+ 类 Awk/C 语言

+ 可以嵌入到C语言中（guru 模式）

+ [SystemTap Language Reference](https://sourceware.org/systemtap/langref/)

### 指令

### 限制

+ SystemTap 可以在内核空间进行追踪，但在用户空间追踪事件取决于内核的支持（Utrace机制）

+ 没有内置于内核中，所以性能比eBPF稍差。

**[示例](https://sourceware.org/systemtap/examples/)**

+ [¶](https://sourceware.org/systemtap/examples/#apps/gmalloc_watch.stp)  [apps/gmalloc_watch.stp](https://sourceware.org/systemtap/examples/apps/gmalloc_watch.stp) - Tracing glib2 memory allocations
  
  The gmalloc_watch.stp script from [Colin Walters' blog ](https://blog.verbum.org/2011/03/19/analyzing-memory-use-with-systemtap/) traces the allocation of glib2 memory using the markers in glib2.
  
  ```bash
  # stap gmalloc_watch.stp -T 1
  ```

+ [¶](https://sourceware.org/systemtap/examples/#memory/glibc-malloc.stp)  [memory/glibc-malloc.stp](https://sourceware.org/systemtap/examples/memory/glibc-malloc.stp) - Overview glibc malloc internal operations 
  
  This script reports on internal statistics of the glibc malloc implementation, as used by a process restricted by `stap -x/-c`
  
  ```bash
  # stap glibc-malloc.stp -c 'stap --dump-functions'
  ```

+ [¶](https://sourceware.org/systemtap/examples/#memory/numa_faults.stp)  [memory/numa_faults.stp](https://sourceware.org/systemtap/examples/memory/numa_faults.stp) - Summarize Process Misses across NUMA Nodes
  
  The numa_faults.stp script tracks the read and write pages faults for each process. When the script exits it prints out the total read and write pages faults for each process. The script also provide a break down of page faults per node for each process. This script is useful for determining whether the program has good locality (page faults limited to a single node) on a NUMA computer.
  
  *[sample usage in memory/numa_faults.txt](https://sourceware.org/systemtap/examples/memory/numa_faults.txt)*

+ ……

### 火焰图

[http://www.brendangregg.com/FlameGraphs/memoryflamegraphs.html](http://www.brendangregg.com/FlameGraphs/memoryflamegraphs.html)

### 参考

1. [Ubuntu - Systemtap](https://wiki.ubuntu.com/Kernel/Systemtap)

2. [SystemTap man](http://manpages.ubuntu.com/manpages/cosmic/man1/stap.1.html)

3. [SystemTap Tutorial](https://sourceware.org/systemtap/tutorial/)

4. [SystemTap Beginners Guide](https://sourceware.org/systemtap/SystemTap_Beginners_Guide/)

5. [SystemTap Example](https://sourceware.org/systemtap/examples/)

6. [FlameGraph](https://github.com/brendangregg/FlameGraph)
