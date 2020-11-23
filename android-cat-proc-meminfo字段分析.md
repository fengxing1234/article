---
title: android_cat_/proc/meminfo字段分析
date: 2020-09-22 10:13:50
tags:

---



`Android`手机的内存大小信息存放在手机系统的`/proc/meminfo`文件中，可以通过读取这个文件来获取内存信息。

```
:/ $ cat /proc/meminfo
MemTotal:         877368 kB  ：所有可用RAM大小（即物理内存减去一些预留位和内核的二进制代码大小）（HighTotal + LowTotal）,系统从加电开始到引导完成，BIOS等要保留一些内存，内核要保留一些内存，最后剩下可供系统支配的内存就是MemTotal。这个值在系统运行期间一般是固定不变的。
MemFree:           22516 kB  ：LowFree与HighFree的总和，被系统留着未使用的内存,MemFree是说的系统层面
MemAvailable:     470244 kB  ：应用程序可用内存数。系统中有些内存虽然已被使用但是可以回收的，比如cache/buffer、slab都有一部分可以回收，所以MemFree不能代表全部可用的内存，这部分可回收的内存加上MemFree才是系统可用的内存，即：MemAvailable≈MemFree+Buffers+Cached，它是内核使用特定的算法计算出来的，是一个估计,MemAvailable是说的应用程序层面
Buffers:            1772 kB  ：用来给文件做缓冲大小
Cached:           459224 kB  ：被高速缓冲存储器（cache memory）用的内存的大小（等于 diskcache minus SwapCache ）
SwapCached:           16 kB  ：被高速缓冲存储器（cache memory）用的交换空间的大小，已经被交换出来的内存，但仍然被存放在swapfile中。用来在需要的时候很快的被替换而不需要再次打开I/O端口
Active:           333148 kB  ：在活跃使用中的缓冲或高速缓冲存储器页面文件的大小，除非非常必要否则不会被移作他用. (Active(anon) + Active(file))
Inactive:         330384 kB  ：在不经常使用中的缓冲或高速缓冲存储器页面文件的大小，可能被用于其他途径. (Inactive(anon) + Inactive(file))
Active(anon):     104368 kB  ：活跃的与文件无关的内存（比如进程的堆栈，用malloc申请的内存）(anonymous pages),anonymous pages在发生换页时，是对交换区进行读/写操作
Inactive(anon):   104508 kB  ：非活跃的与文件无关的内存（比如进程的堆栈，用malloc申请的内存）
Active(file):     228780 kB  ：活跃的与文件关联的内存（比如程序文件、数据文件所对应的内存页）(file-backed pages) File-backed pages在发生换页(page-in或page-out)时，是从它对应的文件读入或写出
Inactive(file):   225876 kB  ：非活跃的与文件关联的内存（比如程序文件、数据文件所对应的内存页）
Unevictable:        6708 kB  ：
Mlocked:            1428 kB  ：
HighTotal:        261888 kB  ：高位内存总大小（Highmem是指所有内存高于860MB的物理内存,Highmem区域供用户程序使用，或用于页面缓存。该区域不是直接映射到内核空间。内核必须使用不同的手法使用该段内存）
HighFree:           5680 kB  ：未被使用的高位内存大小
LowTotal:         615480 kB  ：低位内存总大小,低位可以达到高位内存一样的作用，而且它还能够被内核用来记录一些自己的数据结构
LowFree:           16836 kB  ：未被使用的低位大小
SwapTotal:        614396 kB  ：交换空间的总大小
SwapFree:         611044 kB  ：未被使用交换空间的大小
Dirty:                40 kB  ：等待被写回到磁盘的内存大小
Writeback:             0 kB  ：正在被写回到磁盘的内存大小
AnonPages:        209224 kB  ：未映射页的内存大小
Mapped:           280668 kB  ：设备和文件等映射的大小
Shmem:              1084 kB  ：
Slab:              59840 kB  ：内核数据结构缓存的大小，可以减少申请和释放内存带来的消耗
SReclaimable:      34196 kB  ：可收回Slab的大小
SUnreclaim:        25644 kB  ：不可收回Slab的大小（SUnreclaim+SReclaimable＝Slab）
KernelStack:        7504 kB  ：
PageTables:        15508 kB  ：管理内存分页页面的索引表的大小
NFS_Unstable:          0 kB  ：不稳定页表的大小
Bounce:                0 kB  ：
WritebackTmp:          0 kB  ：
CommitLimit:     1053080 kB  ：根据超额分配比率（'vm.overcommit_ratio'），这是当前在系统上分配可用的内存总量，这个限制只是在模式2('vm.overcommit_memory')时启用。CommitLimit用以下公式计算：CommitLimit =（'vm.overcommit_ratio'*物理内存）+交换例如，在具有1G物理RAM和7G swap的系统上，当`vm.overcommit_ratio` = 30时 CommitLimit =7.3G
Committed_AS:   16368536 kB  ：目前在系统上分配的内存量。是所有进程申请的内存的总和，即时所有申请的内存没有被完全使用，例如一个进程申请了1G内存，仅仅使用了300M，但是这1G内存的申请已经被 "committed"给了VM虚拟机，进程可以在任何时间使用。如果限制在模式2('vm.overcommit_memory')时启用，分配超出CommitLimit内存将不被允许
VmallocTotal:     245760 kB  ：可以vmalloc虚拟内存大小
VmallocUsed:           0 kB  ：已经被使用的虚拟内存大小
VmallocChunk:          0 kB  ：最大的连续未被使用的vmalloc区域
```

