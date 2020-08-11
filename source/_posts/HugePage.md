---
title: Huge Page
date: 2020-08-11 21:55:14
tags: "Huge Page"
categories: ["OS", "Linux"]
desc: Linux系统中使用 Huge Page。
---

### Ⅰ. Check Huge Page

1. Linux 内核支持多种 page size。
   
   | 架构    | HugePage Size                                                                                |
   | ----- | -------------------------------------------------------------------------------------------- |
   | arm64 | 4K, 2M and 1G (or 64K and 512M if one builds their own kernel with CONFIG_ARM64_64K_PAGES=y) |
   | x86   | 4K and 4M (2M in PAE mode，1GB if architecturally supported)                                  |
   | amd64 | 2MB, 1GB                                                                                     |
   | ia64  | 4K, 8K, 64K, 256K, 1M, 4M, 16M, 256M                                                         |
   | ppc64 | 4K, 16M                                                                                      |

2. Huge Page 支持 `mmap` 和 `shmget`、`shmat` 调用。

3. 当前系统 Huge Page 设置信息
   
   ```bash
    $ grep Huge /proc/meminfo 
    AnonHugePages:         0 kB
    ShmemHugePages:        0 kB
    HugePages_Total:       0
    HugePages_Free:        0
    HugePages_Rsvd:        0
    HugePages_Surp:        0
    Hugepagesize:       2048 kB
    Hugetlb:               0 kB
   ```
   
   - `HugePages_Total` Huge Page 池的页面数量。
   
   - `HugePages_Free`  Huge Page 池中未分配的页面数量。
   
   - `HugePages_Rsvd`  已承诺从池中分配但尚未进行分配的 Huge Page 的数量。
   
   - `HugePages_Surp`  `/proc/sys/vm/nr_hugepages`中 表示 Huge Page 池中页面数量。 `/proc/sys/vm/nr_overcommit_hugepages` 表示最大值。
   
   - `Hugepagesize`  默认 hugepage 尺寸（Kb）。
   
   - `Hugetlb`  内存总量（kB） `HugePages_Total * Hugepagesize`

4. `/proc/filesystems` 可以查看 `hugetlbfs` 配置信息

5. `/proc/sys/vm/nr_hugepages`  表示当前内核大页面池中“持久”大页面的数量。

6. 检查 NUMA 系统中大型页面的每个节点分布情况，使用：
   
   ```bash
   cat /sys/devices/system/node/node*/meminfo | fgrep Huge
   ```

7. Shirink：
   
   > Shrinking the persistent huge page pool via `nr_hugepages` such that it becomes less than the number of huge pages in use will convert the balance of the in-use huge pages to surplus huge pages.  This will occur even if the number of surplus pages it would exceed the overcommit value.  As long as this condition holds--that is, until `nr_hugepages+nr_overcommit_hugepages` is increased sufficiently, or the surplus huge pages go out of use and are freed -- no more surplus huge pages will be allowed to be allocated.

8. 运行时 huge page 接口
   
   - `/proc/sys/vm` 
   
   - `/sys/kernel/mm/hugepages` 中 `hugepages-${size}kB` 

## Ⅱ、Huge Page 分配/释放与 NUMA 内存策略

分配或释放 huge page 可以使用：

```bash
numactl -m <node-list> echo 20 >/proc/sys/vm/nr_hugepages_mempolicy
```

这个操作将会释放或分配 `abs(20 - nr_hugepages)`  到 `<node-list>` 。



## Ⅲ、使用 Huge Page

需要使用 `mmap` ，应该先挂载 hugetlbfs：

```bash
  mount -t hugetlbfs \
	-o uid=<value>,gid=<value>,mode=<value>,pagesize=<value>,size=<value>,\
	min_size=<value>,nr_inodes=<value> none /mnt/huge
```

在内核 2.6 之后，可以使用 `MAP_HUGETLB` 的方式操作内存。

- `map_hugetlb`

- `hugepage-shm`

- `hugepage-mmap`https://man7.org/linux/man-pages/man2/mmap.2.html

- [libhugetlbfs](https://github.com/libhugetlbfs/libhugetlbfs)

**测试**

- 设置 huge page
  
  ```bash
  sysctl vm.nr_hugepages=192

  AnonHugePages:         0 kB
  ShmemHugePages:        0 kB
  HugePages_Total:     192
  HugePages_Free:      192
  HugePages_Rsvd:        0
  HugePages_Surp:        0
  Hugepagesize:       2048 kB
  Hugetlb:          393216 kB

  ```

- huge.c 测试
  
  ```c
  #include <sys/mman.h>
  #include <stdio.h>
  #include <memory.h>
  
  int main(int argc, char *argv[]) {
    char *m;
    size_t s = (8UL * 1024 * 1024);
  
    m = mmap(NULL, s, PROT_READ | PROT_WRITE,
                      MAP_PRIVATE | MAP_ANONYMOUS | 0x40000 /*MAP_HUGETLB*/, -1, 0);
    if (m == MAP_FAILED) {
      perror("map mem");
      m = NULL;
      return 1;
    }
  
    memset(m, 0, s);
  
    printf("map_hugetlb ok, press ENTER to quit!\n");
    getchar();
  
    munmap(m, s);
    return 0;
  }
  ```

- 查看 huge page 信息
  
  ```bash
  $ gcc huge.c
  $ ./a.out
   map_hugetlb ok, press ENTER to quit!

  $ cat /proc/meminfo |grep -i huge

  AnonHugePages:         0 kB
  ShmemHugePages:        0 kB
  HugePages_Total:     192
  HugePages_Free:      188
  HugePages_Rsvd:        0
  HugePages_Surp:        0
  Hugepagesize:       2048 kB
  Hugetlb:          393216 kB

  ```



## Reference

1. https://www.kernel.org/doc/Documentation/vm/hugetlbpage.txt

2. http://blog.chinaunix.net/uid-28541347-id-5783934.html

3. https://wiki.debian.org/Hugepages

4. [Linux下试验大页面映射（MAP_HUGETLB）_cncnlg的专栏-CSDN博客_map_hugetlb](https://blog.csdn.net/cncnlg/article/details/37498057)

5. https://www.kernel.org/doc/Documentation/vm/transhuge.txt

6. [mmap(2) - Linux manual page](https://man7.org/linux/man-pages/man2/mmap.2.html)

