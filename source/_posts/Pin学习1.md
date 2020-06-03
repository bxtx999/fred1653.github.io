---
title: Pin学习1
date: 2016-11-16 09:28:56
categories: ["OS", "计算机性能"]
tags: "Pin"
---


### 什么是插桩（Instrumentation）?
向程序注入额外的代码来收集程序运行时的状态。

插桩（Instrumentation）的方法：
* 源代码插桩（source instrumentation）
  ——对源码进行操作
* 二进制插桩（Binary instrumentation）
  ——运行时直接注入

  <!-- more -->

### 为何使用动态指令注入？
* 不需要重新编译或重新链接
* 在运行时检测代码
* 处理动态生成的代码
* 附加到运行的进程中

### Pin工具的有点
1. 简单易用
  * 使用动态指令插入方式
     —— 不需要修改源代码、重新编译、重新连接
1. 多平台支持
  * 支持X86、X86_64、Itanium、Xscale
  * 支持Linux、Windows、MacOS
1. 鲁棒性
  * 支持现实常见应用： 数据库、浏览器
  * 支持多线程程序
  * 支持信号(signals)
1. 高效
  * 适用编辑器优化(Applies compiler optimizations on instrumentation code)

### Pin使用方式

```
// 加载和注入一个程序
$ pin -t  pintool -- application
// Pin —— 注入引擎，在kit中已经提供
//Pintool —— 注入工具，自己编写或者适用kit中提供的

// 注入到一个程序中（进程号）
$ pin -t pintool -pid 1234
```

### Pin注入的API
+ 架构无关的基础API：
  * 提供确定的基础功能的API
     - Control-flow changes
     - Memory accesses
+ 基于特定架构的API：
  * 如基于IA32的段寄存器信息
+ 基于调用（call-based）的API：
  * **桩**程序（Instrumentation routines）
  * 分析程序（Analysis routines）

### **桩**程序和分析程序的区别
**桩**和**分析**都是从ATOM工具发展来的概念。（搜索关键字 ATOM analysis  Instrumentation ）
> ATOM，即Analysis Tools with OM， http://atominstrument.com/products-services-overview/  http://www.hpl.hp.com/techreports/Compaq-DEC/WRL-TN-44.pdf

桩程序(instrumentation routines）定义了在哪里插桩；
 * 如在指令前 ： 
     在指令第一次执行时插桩
 分析程序（analysis routines）定义当桩启动时执行哪些操作。
* 如 增量计数器：
    在每一次指令执行时发生

### Pintool举例1——指令计数
```
sub $0xff, %edx
    counter++;
cmp %esi, %edx
     counter++;
jle <L1>
      counter++;
mov $0x1, %edi
       counter++;
add $0x10, %eax
       counter++;
```
    
#### 输出指令数
```
$ /bin/ls
Makefile imageload.out itrace proccount
imageload inscount0 atrace itrace.out
$ pin -t inscount0 -- /bin/ls
Makefile imageload.out itrace proccount
imageload inscount0 atrace itrace.out
Count 422838 
```
![输出指令数](http://ww3.sinaimg.cn/mw690/49735734gw1f9tqy8lixzj20sz0450vg.jpg)


### Pintool举例2——指令跟踪
```
// 传递ip参数到分析程序中
Print(ip); 
sub $0xff, %edx 
Print(ip); 
sub $0xff, %edx 
Print(ip); 
jle <L1>
Print(ip); 
mov $0x1, %edi
Print(ip); 
add $0x10, %eax
Print(ip); 
```

#### 输出轨迹
```
$ pin -t itrace -- /bin/ls
Makefile imageload.out itrace proccount
imageload inscount0 atrace itrace.out
$ head -6 itrace.out
0x7f20e459b2d0
0x7f20e459b2d3
0x7f20e459ea40
0x7f20e459ea41
0x7f20e459ea44
0x7f20e459ea46
```

### ManualExamples/itrace.cpp
```
#include <stdio.h>
#include "pin.H"
FILE * trace;
void printip(void *ip) { fprintf(trace, "%p\n", ip); }  //  分析程序
void Instruction(INS ins, void *v) {
INS_InsertCall(ins, IPOINT_BEFORE, (AFUNPTR)printip, IARG_INST_PTR, IARG_END);
}                                                               // 插桩程序
void Fini(INT32 code, void *v) { fclose(trace); }
int main(int argc, char * argv[]) {
trace = fopen("itrace.out", "w");
PIN_Init(argc, argv);
INS_AddInstrumentFunction(Instruction, 0);
PIN_AddFiniFunction(Fini, 0);
PIN_StartProgram();
return 0;
}
```

### 分析程序的参数举例：
+ IARG_INST_PTR
  * 指令指针值（program counter）
+ IARG_UINT32 <value>
  * 整型值
+ IARG_REG_VALUE <register name>
  * 指定寄存器的值
+ IARG_BRANCH_TARGET_ADDR
  * 分支桩的目标地址
+ IARG_MEMORY_READ_EA
  * 内存读取的有效地址

### 一个指令的桩的位置

* 前置（IPOINT_BEFORE）

* 后置
  - Fall-through edge（IPOINT_AFTER）
  - Taken edge （IPOINT_TAKEN_BRANCH）

![桩](http://ww4.sinaimg.cn/mw690/49735734gw1f9tqyeuv6bj20nq05aaak.jpg)
注： 红色代表 IPOINT_BEFORE， 绿色代表 IPOINT_AFTER，黑色代表  IPOINT_TAKEN_BRANCH。


### 来源

http://www.cs.du.edu/~dconnors/courses/comp3361/notes/PinTutorial

### 参考
http://terenceli.github.io/技术/2014/01/02/intro-to-pin
http://huirong.github.io/2015/12/30/Intel-Pin-introduction/#参考文献