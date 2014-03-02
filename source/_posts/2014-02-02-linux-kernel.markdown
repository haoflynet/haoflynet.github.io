---
layout: post
title: "linux kernel配置详细说明"
date: 2014-02-02 11:10:30 +0800
comments: true
categories: [Kernel]
tags: [linux, kernel]
---
从去年写到今年，解释这些菜单真的太费时了，而且没有研究过markdown的排版技术，所以太混乱了。这个说明我在以后还会逐渐完善的！

 - General setup    ---> 常规选项
<!--more-->
        与Linux最相关的、核心版说明、是否使用程序代码、是否使用虚拟内存等都在这里设定。这里的项目主要是针对核心不同程序需之间的相关性来设计
         |--()	Cross-compiler tool prefix	                交叉编译工具前缀
         |--[ ]  Compile also drivers which will not load	编译不需要加载的驱动程序
         |--()   Local version - append to kernel release    	自定义版本，即用uname -r可以看到的版本名
         |--[ ]	Automatically append version information to the version string	自动在版本字符串后面添加版本信息
         |--     Kernel compression mode (Gzip)  --->    	内核镜像要用的压缩模式
         |--((none)) Default hostname				可以设置默认的主机名
         |--[*]  Support for paging of anonymous memory (swap)   支持交换分区(即虚拟内存)
         |--[*]  System V IPC  	System V进程间通信(IPC)支持，处理器在程序之间同步和交换信息，如果不选这项，很多程序运行不起来
         |--[*]  POSIX Message Queues    	POSIX消息队列，这时POSIX IPC中的一部分
         |--[*]  open by fhandle syscalls	之后调用程序就可以使用句柄而不必使用程序名
         |--_*_  Auditing support  		审计支持，用于和内核的某些子模块同时工作，(例如SELinux)需要它，只有同时选择其子项才能对系统调用进行审计，
         |--[*]    Enable system-call auditing support-	支持对系统调用的审计
         |--     IRQ subsystem   --->	硬件中断
         |--     Timers subsystem    --->	时钟子系统
         |	     |--	Timer tick handling (Full dynticks system (tickless))	--->	时钟滴答处理程序
         |	     |	    |--( )	Periodic timer ticks (constant rate, no dynticks)	无论CPU是否需要，都强制按照固定频率不断出发时钟中断，这是最耗电的方式
         |	     |	    |--( )	Idle dynticks system (tickless idle)	空闲状态时不产生不必要的时钟中断，以使处理器能够在较低能耗状态下运行以节约电力，适合于大多数场合
         |	     |	    |--(X)	Full dynticks system (tickless)	完全无滴答，即使CPU在忙碌状态也尽可能关闭所有时钟中断，适用于CPU在同一时间仅运行一个任务，或者用户控件程序极少与内核交互的场合
         |	     |--[ ]	Full dynticks system on all CPUs by default	即使没有设置"nohz_full"引导参数,也默认对所有CPU(boot CPU 除外)开启完全无滴答特性.
         |	     |--[*]	Detect full-system on all CPUs by default
         |       |--(8)	  Number of CPUs above which large-system approach is used
         |	     |--[*]	Old Idle dynticks config	等价于CONFIG_NO_HZ_IDLE,临时用来兼容老版本内核选项,未来会被删除.
         |	     |--[*]	High Resolution Timer Support	高精度定时器(hrtimer)是从2.6.16开始引入,用于取代传统timer wheel(基于jiffies定时器)的时钟子系统.可以降低与内核其他模块的耦合性,还可以提供比1毫秒更高的精度(因为它可以读取HPET/TSC等新型硬件时钟源),可以更好的支持音视频等对时间精度要求较高的应用.建议选"Y".[提示]这里说的"定时器"是指"软件定时器",而不是集成在主板上的硬件时钟发生器(ACPI PM Timer/HPET Timer/TSC Timer).
         |--     CPU/Task time and stats accounting  --->
         |       |--     Cputime accounting (Full dynticks CPU time accounting)  --->
         |       |--[*]  BSD Process Accounting	    将进程的统计信息写入文件的用户级系统调用，主要包括进程的创建时间/创建者/内存占用等信息
         |       |--[*]    BSD Process Accounting version 3 file format	使用新的第三版文件格式，可以包含每个进程的PID和其父进程的PID，但是不兼容老版本的文件格式
         |       |--_*_  Export task/process statistics through netlink	通过netlink接口向用户空间导出任务/进程的统计信息，与BSD Process Accounting 的不同之处在于这些统计信息在整个任务/进程生存期都是可用的
         |       |--_*_    Enable per-task delay accounting 	在统计信息中包含进程等候系统资源(cpu,IO同步，内存交换等)所花费的时间
         |       |--[*]    Enable extended accounting over taskstats	收集额外的进程统计信息并通过taskstats接口发送到用户空间
         |       |--[*]      Enable per-task storage I/O accounting
         |--     RCU Subsystem   ---> 非对称读写锁系统，是一种高性能的kernel锁机制，适用于读多写少环境
         |       |--     RCU Implementation (Tree-based hierarchical RCU)    --->
         |       |--_*_  Consider userspace as in RCU extended quiescent state	除非你是想hack一下，或者帮助the full dynticks mode（一种RCU机制）的发展，否则你不应该选这一项，它也增加了不必要的开销。但我这电脑上居然是默认的
         |       |--[*]  Force context tracking
         |       |--(64) Tree-based hierarchical RCU fanout value
         |       |--(16) Tree-based hierarchical RCU leaf-level fanout value
         |       |--[ ]  Disable tree-based hierarchical RCU auto-balancing
         |       |--[*]  Accelerate last non-dyntick-idle CPU's grace periods
         |       |--_*_  Offload RCU callback processing from boot-selected CPUs
         |       |--     Build-forced no-CBs CPUs (All CPUs are build_forced no-CBs CPUs) --->
                  |--< >  Kernel .config support	把内核配置信息编译进内核中，以后可以通过scripts/extract-ikconfig脚本来提取这些信息
         |--(18) Kernel log buffer size (16 => 64KB, 17 => 128 KB)	内核日志缓冲区大小
         |--[*]  Automatically enable NUMA aware memory/task placement	自动均衡NUMA(非一致性内存访问)的内存/任务
         |--[*]  Memory placement aware NUMA scheduler	由NUMA调度器进行内存分配
         |--_*_  Control Group support   --->    cgroups支持，文档资料，cgroups主要，文档资料，cgroups主要作用是给进程分组，并可以动态调控进程组的CPU占用率
         |       |--[ ]  Example debug cgroup subsystem	导出cgroup子系统的调试信息
         |       |--[*]  Freezer cgroup subsystem	冻结cgroup子系统
         |       |--[*]  Device controller for cgroups	cgroup的设备控制器
         |       |--[*]  Cpuset support   	CPU分组支持，只有含有大量CPU(大于16个)的SMP系统或NUMA(非一致内存访问)系统才需要它
         |       |--[*]    Include legacy /proc/<pid>/cpuset file
         |       |--[*]  Simple CPU accounting cgroup subsystem	cgroup子系统的简单CPU统计，会生成一个资源控制器来监控在一个cgroup组内的独立任务的CPU使用情况
         |       |--[*]  Resouce counters	资源计数器，使控制器的独立资源统计功能能够统计cgroup
         |       |--[*]    Memory Resource Controller for Control Groups
         |       |--[*]      Memory Resource Controller Swap Extension
         |       |--[ ]        Memory Resource Controller Swap Extension enabled by default
         |       |--[ ]      Memory Resource Controller Kernel Memory accounting
         |       |--[*]    HugeTLB Resource Controller for Control Groups
         |       |--[*]  Enable perf_event per-cpu per-container group (cgroup) monitoring	允许开发者扩展每个CPU的模式，使它可以只监视运行在特定CPU上的一个特别的cgroup组的线程
                         |--_*_  Group CPU scheduler --->	CPU分组调度器
                         |--_*_  Group scheduling for SCHED_OTHER
                         |--[*]    CPU bandwidth provisioning for FAIR_GROUP_SCHED
                         |--[*]  Group scheduling for SCHED_RR/FIFO
                         |--[*]  Block IO controller	块IO控制器
                         |--[ ]    Enable Block IO controller debugging	启用阻塞IO控制器的调试
          |--[*]  Checkpoint/restore support	支持检查点及还原
          |--[*]  Namespaces support  --->	命名空间支持
          |--_*_  Require conversions between uid/gids and their internal representation
          |--[*]  Automatic process group scheduling
          |--[ ]  Enable deprecated sysfs features to support old userspace tools	启用不推荐的sysfs功能来支持旧式的用户控件工具，这是调试内核时用的虚拟文件新系统
          |--_*_  Kernel->user space relay support (formerly relayfs)	在某些文件系统上(比如debugfs)提供从内核空间向用户空间传递大量数据的接口
          |--{*}  Initial RAM filesystem and RAM disk (initramfs/initrd) support  初始化内存文件系统和内存盘用于在真正内核装载前，做一些操作(俗称两阶段启动)，比如加载module，mount一些非root分区，提供灾难恢复shell环境等，资料
          |--()     Initramfs source file(s)  initrd已经被initramfs取代
          |--[*]    Support initial ramdisks compressed using gzip	这几项是初始化虚拟磁盘所支持的压缩格式
          |--[*]    Support initial ramdisks compressed using bzip2
          |--[*]    Support initial ramdisks compressed using LZMA
          |--[*]    Support initial ramdisks compressed using XZ
          |--[*]    Support initial ramdisks compressed using LZ0
          |--[*]    Support initial ramdisks compressed using LZ4
          |--[ ]  Optimize for size       这个选项将在GCC命令后用“-Os”代替“-O2”参数，这样可以得到更小的内核，没必要选，选上了有时会产生错误的二进制代码；编译时优化内核尺寸(使用"-Os"而不是"-O2"参数编译)，有时会产生错误的二进制代码
          |--[*]  Confitgure standard kernel features (expert users)  --->       配置标准的内核特性(为小型系统)专家级用户使用
      	          |--[*]  Enable 16-bit UID system calls 允许对UID系统调用进行过时的16-bit包装
                  |--[*]  Sysctl syscall support     不需要重启就能修改内核的某些参数和变量，如果你页选择了支持/proc，将能从/proc/sys存取可影响内核行为的参数或变量
                  |--_*_  Load all symbols for debugging/ksymoops 装在所有的调试符号表信息，仅供调试时选择
                  |--_*_    Include all symbols in kallsyms    在kallsyms中包含内核知道的所有符号，内核将会增大300k
                  |--[*]  Enable support for printk   允许内核向终端打印字符信息，在需要诊断内核为什么不能运行时选择      
                  |--[*]  BUG() support  显示故障和失败条件(BUG和WARN)，禁用它将可能导致隐含的错误被忽略
                  |--[*]  Enable ELF core dumps   内存转储支持，可以帮助调试ELF格式的程序
                  |--[*]  Enabel PC-Speaker support
                  |--[*]  Enable full-sized data structures for core  在内核中使用全尺寸的数据结构。禁用它将使得某些内核的数据结构减小以节约内存，但是会降低性能
                  |--[*]  Enabel futex support    快速在用户空间互斥体可以使线程串行化以避免竞态条件，页提高了响应速度，禁用它将导致内核不能正确的运行基于glibc的程序
                  |--[*]  Enable eventpoll support   支持事件轮循的系统调用
                  |--[*]  Enable singnalfd() system call
                  |--[*]  Enable timerfd() system call
                  |--_*_  Enable eventfd() system call
                  |--_*_  Use full shmem filesystem   完全使用shmem来代替ramfs.shmem是基于共享内存的文件系统(可能用到swap),在启用TMPFS后可以挂载为tmpfs供用户控件使用，它比简单的ramfs先进许多
                  |--[*]  Enable AIO support
                  |--[*]  Enable PCI quirk workarounds
                  |--[ ]  Embedded system
                  |--     Kernel Performance Events And Counters  --->
                          |--[ ]  Debug: use vmalloc to back perf mmap() buffers
          |--[*]  Enable VM event counters for /proc/vmstat   允許在/proc/vmstat中包含虚拟内存事件记数器
          |--[*]  Enable SLUB debugging support
          |--[ ]  Disable heap randomization
          |--     Choose SLAB allocator (SLUB (Unqueued Allocator))   --->	使用SLAB完全取代SLOB进行内存分配，SLAB是一种优秀的内存分配管理器，推荐使用
          |--[*]  SLUB per cpu partial cache
          |--[*]  Profiling support	对系统的活动进行分析，仅供内核开发者使用
          |--<M>  Oprofile system profiling
          |--[ ]    Oprofile multiplexing support (EXPREIMENTAL)
          |--[*]  Kprobes		仅供内核开发者使用
          |--[*]  Optimize very unlikely/likely branches
          |--_*_  Transparent user-space probes (EXPERIMENTAL)
          |--     GCOV-based kernel profiling --->
    
 - [*] Enable loadable module support  --->  可加载模块的支持可让内核支持模块

        |--[ ]  Forced module loading
        |--[*]  Module unloading		可以让用户卸载不再使用的模块，如果不选中的话用户将不能卸载任何模块(注意，有些模块一旦加载就不能卸载，与是否选择了此项无关)
        |--[ ]    Forced module unloading 	允许强制卸载正在使用中的模块(比较危险)
        |--[*]  Module versioning support   允许用户可以使用其他版本内核中编译的模块，不过并不可靠，可能会出问题，所以一般不选择它
        |--[*]  Source checksum for all modules	为所有的模块校验源码，如果你不是自己编写的内核模块就不需要它
        |--[*]  Module signature verification
        |--[ ]    Require modules to be validly signed
        |--[*]    Automatically sign all modules   
 - [*] Enable the block layer --->	块设备支持，比如硬盘/USB/SCSI设备

            |--_*_  Block layer SG support v4   支持通用scsi块设备第4版
            |--_*_  Block layer SG support v4 helper lib
            |--[*]  Block layer data integrity support	支持块设备数据完整性
            |--[*]  Block layer bio throttling support
            |--_*_  Block device command line partition parser
                    Partition Types --->
                    |--[*]  Advanced partition selection
                    |--[*]    Acorn partition support
                    |--[ ]      Cumana partition support
                    |--[ ]      EESOX partition support
                    |--[*]      ICS partition support
                    |--[ ]      Native filecore partition support
                    |--[ ]      PowerTecpartition support
                    |--[*]      RISCiX partition support
                    |--[*]    AIX basic partition table support
                    |--[*]    Alpha OSF partition support
                    |--[*]    Amiga partition table support
                    |--[*]    Atari partition table support
                    |--[*]    Macintosh partition map support
                    |--[*]    PC BIOS (MSDOS partition tables) support
                    |--[*]      BSD disklabel (FreeBSD partition tables) support
                    |--[*]      Minix subpartition support
                    |--[*]      Solaris (x86) partition table support
                    |--[*]      Unixware slices support
                    |--[*]    Windows Logical Disk Manager (Dynamic Disk) support
                    |--[ ]      Windows LDM extra logging
                    |--[*]    SGI partition support
                    |--[*]    Ultrix partition table support
                    |--[*]    Sun partition tables support
                    |--[*]    Karma Partition support
                    |--[*]    EFI GUID Partition support
                    |--[*]    SYSV68 partition table support
                    |--[*]    Command line partition support
                    IO Schedulers   --->     IO调度器支持，不同程序可以会选用不同的调度策略
                       |--<*>  Dedline I/O scheduler	使用轮询的调度器，简洁小巧，提供了最小的读取延迟和较好的吞吐量，特别适合于读取较多的环境(比如数据库)
                       |--<*>  CFQ I/O scheduler		使用Qos策略为所有任务分配等量的带宽，避免进程被饿死并实现了较低的延迟
                       |--[*]    CFQ Group Scheduling support
                       |--     Default I/O scheduler (Deadline)    --->	默认IO调度器
 
 - Processor type and features  --->	处理器类型及特性

        |--[*]  DMA memory allocation support
        |--[*]  Symmetric multi-processing support	对称多处理器支持，如果有多个CPU或者使用的是多核CPU就选上
        |--[*]  Support x2apic
        |--[*]  Enable MPS table    mps多处理器规范，居然叫我关闭
        |--[*]  Support for etended (non-PC) x86 platforms	支持除x86的其他平台
        |--[*]  Numascale NumaChip
        |--[ ]  ScaleMP vSMP
        |--[ ]  SGI Ultraviolet
        |--[*]  Intel Low Power Subsystem Support
        |--[*]  Single-depth WCNAN output
        |--[*]  Linux guest support --->
                |--[*]  Enable paravirtualization code
                |--[ ]    paravirt-ops debugging
                |--[*]    Paravirtualization layer for spinlocks
                |--[*]    Xen guest support
                |--[ ]  Enable Xen debug and tuning parameters in debugfs
                |--[*]  KVM Guest support (including kvmclock)
                |--[*]    Enable debug information for KVM Guests in debugfs
                |--[ ]  Paravirtual steal time accounting
        |--[*]  Memtest
        |--     Processor family (Generic-x86-64)   --->    这里选择相应的CPU
        |--[*]  Supported processor vendors --->
        |--[*]  Enable DMI scanning
        |--[*]  Old AMD GART IOMMU support
        |--[*]  IBM Calgary IOMMU support
        |--[*]    Should Calgary be enabled by default?
        |--[ ]  Enable Maximum number of SMP Processors and NUMA Nodes
        |--(256)Maximum number of CPUs	支持的最大CPU数，没增加一个内核将增加8k体积
        |--[*]  SMT (Hyperthreading) scheduler support  支持Intel的超线程(HT)技术超线程调度器在某些情况下将会对Intel Pentium 4 HT系列有较好的支持。如果不清楚，选N
        |--[*]  Multi-core scheduler support     针对多核CPU进行调度策略优化，多核调度机制支持，双核的CPU要选
        |--     Preemption Model (Voluntary Kernel Preemption (Desktop))    --->   内核抢占模式一些优先级很高的程序可以先让一些低优先级的程序执行，即使这些程序是在核心态下执行。从而减少内核潜伏期，提高系统的响应能力。当然在一些特殊的点的内核是不可抢先的，比如内核中的调度程序自身在执行时就是不可被抢先的，这个特性可以提高桌面系统、实时系统的性能
        |--[*]  Reroute for broken boot IRQs       防止同时收到多个boot IRQ(中断)时，系统混乱
        |--[*]  Machine Check / overheating reporting   
        |--[*]    Intel MCE features
        |--[*]    AMD MCE features          这两项貌似只根据自己的CPU来选
        |--<M>  Machine check injector support
        |--<M>  Dell laptop support			Dell笔记本模块支持
        |--<M>  CPU microcode loading support
        |--[*]    Intel microcode loading support
        |--[*]    AMD microcode loading support
        |--<M>  /dev/cpu/*/msr - Model-specific register support	是否打开CPU特殊功能寄存器的功能。这个选项桌面用户一般用不到，它主要用在intel的嵌入式CPU中的，这个寄存器的作用也依赖于不同的CPU类型，一般可以用来改变一些CPU原有物理结构的用途，但不同的CPU用途差别也很大，在多CPU系统中让特权CPU访问x86的MSR寄存器
        |--<M>  /dev/cpu/*/cpuid - CPU information support	是否打开记录CPU相关信息功能，这会在/deg/cpu中建立一系列的设备文件，用以让过程去访问指定的CPU，能从/dev/cpu/x/cpuid获得CPU的唯一标识(CPUID)
        |--[*]  Enable 1GB pages for kernel pagetables
        |--[*]  Numa Memory Allocation and Scheduler Support
        |--[*]  Old style AMD Opteron NUMA detection
        |--[*]  ACPI NUMA detection
        |--[ ]  NUMA emulation
        |--(6)  Maximum NUMA Nodes (as a power of 2)
        |--[*]  Enable sysfs memory/probe interface
        |--     Memory model (Sparse Memory)    --->	
        |--[*]  Sparse Memory virtual memmap
        |--[*]  Enable to assin a node which has only movable memory
        |--[*]  Allow for memory hot-add
        |--[*]    Allow for memory hot remove
        |--[*]  Allow for ballon memory compaction/migration
        |--_*_  Allow for memory compaction
        |--_*_    Page migration
        |--[*]  Enable bounce buffers
        |--[*]  Enable KSM for page merging
        |--(65536) Low address space to protect from user allocation
        |--[*]  Enable recovery from hardware memory errors
        |--<M>    HWPoison pages injector
        |--[*]  Transparent Hugepage Support
        |--       Transparent Hugepage Support sysfs defaults (madvise) --->
        |--[*]  Croos Memory Support
        |--[*]  Enable cleancache driver to cache clean pages if tmem is present
        |--[*]  Enable frontswap to cache swap pages if tmem is present
        |--[*]  Contiguous Memory Allocator
        |--[ ]    CMA debug message (DEVELOPMENT)
        |--[*]  Compressed cache for swap pages (EXPERRIMENTAL)
        |--[*]  Track memory changes
        |--[*]  Check for low memory corruption // 低位内存脏数据检查，默认是每60秒检查一次。一般这种脏数据是因某些Bios处理不当引起的
        |--[*]    Set the default setting of memory_corruption_check
        |--(64) Amount of low memory, in kilobytes, to reserve for the BIOS
        |--[*]  MTRR (Memory Type Range Register) support	内存类型区域寄存器。在Intel P6系列处理器(Pentium Pro, Pentium 2和更新的)上，MTRR将会用来规定和控制处理器访问某段内存区域的策略。如果你在PCI或者AGP总线上有VGA卡，这将非常有用。可以提升图像的传送速度2.5倍以上。选Y，会生成文件/proc/mtrr，它可以用来操纵你的处理器的MTRR。典型地，X server会用到。这段代码有着通用的接口，其他CPU的寄存器同样能够使用该功能。Cyrix 6x86MX和M 2处理器有ARR，它和MTRR有着类似的功能。AMD k6-2/k6-3有两格MTRR，Centaur C6有8个MCR允许复合写入。所有这些处理器都支持这段代码，你可以选Y如果你有以上处理器。选Y同样可以修正SMP BIOS的问题，它仅为第一个CPU提供MTRR，而不为其他的提供。这回导致各种各样的问题，所以应该选Y。打开它可提升PCI/AGP总线上的显卡2倍以上的速度，并且可以修正某些BIOS错误
        |--[*]    MTRR cleanup support
        |--(1)      MTRR cleanup enable value (0-1)
        |--(1)      MTRR cleanup spare rg num (0-7)
        |--[*]    x86 PAT support
        |--[*]  x86 architectural random number generator
        |--[*]  Supervisor Mode Access Prevention
        |--[*]  EFI runtime service support
        |--[*]    EFI stub support
        |--[*]  Enable seccomp to safely compute untrusted bytecode //只有嵌入式系统可以不选
        |--[*]  Enable -fstack-protector buffer overflow detection
        |--     Timer frequency (250 HZ)    --->    内核时钟频率，桌面用户1000Hz),服务器100或250
        |--[*]  kexec system call   kexec系统调用。kexec是一个用来关闭你当前内核，然后开启另一个内核的系统调用。它和重启很像，但是它不访问系统固件。由于和重启很像，你可以启动任何内核，不仅仅是LINUX。kexec这个名字是从exec系统调用来的。它只是一个进程，可以确定硬件是否正确关闭。
        |--[*]  kernel crash dumps	内核崩溃时，dump运行时信息
        |--[*]  kexec jump
        |--(0x10000000) Phusical address where the kernel is loaded
        |--[*]  Build a relocatable kernel  建立一个移动的内核，并增加10%的内核尺寸，运行时会被丢弃，他认为没有实质性作用
        |--(0x10000000) Alignment value to which kernel should be aligned
        |--_*_  Support for hot-pluggable CPUs      对SMP休眠和热插拔CPU提供支持，支持热插拔设备，如usb与pc卡等，Udev页需要它
         |--[ ]    Set default setting of cpu0_hotpluggable
        |--[ ]    Debug CPU0 hotplug
        |--[ ]  Compat VDS0 support
        |--[ ]  Built-in kernel command line 
 - Power management and ACPI otions --->(ACPI、APM)是高级电源管理。要注意ACPI与APM不能同时使用,所谓高级电源管理，如软关机，系统休眠等

        |--[*]  Suspend to RAM and standby           待机
        |--[*]  Hibernation (aka 'suspend to disk') 休眠
        |--()   Default resume partition
        |--[ ]  Opportunistic sleep
        |--[ ]  User space wakeup sources interface
        |--[*]  Run-time PM core functionality
        |--[*]  Power Management Debug Support	仅供调试使用
        |--[*]    Extra PM attributes in sysfs for low-level debugging/testing
        |--[ ]    Test suspend/resume and wakealarm during bootup
        |--[ ]    Device suspend/resume watchdog (NEW)
        |--[*]  Suspend/resume event tracing
        |--[*]  Enable workqueue power-efficient mode by default
        |--[*]  ACPI (Advanced Configuration and Power Interface) Support   --->
            |--[ ]  Deprecated /proc/acpi files
            |--<M>  Ec read/write access through /sys/kernel/debug/ec
            |--<*>  AC Adapter	如果你的系统可以在AC和电池之间转换就可以选
            |--<*>  Battery		通过/proc/acpi/battery向用户提供电池状态信息，用电池的笔记本可以选
            |--{*}  Button	守护程序不活Power,Sleep,Lid按钮事件，并根据/proc/acpi/event做相应的动作，软件控制的poweroff需要它
            |---M-  Video	仅对集成在主板上的显卡提供ACPI2.0支持，且不是所有的显卡都支持
            |--<*>  Fan	允许通过用户层的程序来对系统风扇进行控制(开，关，查询状态)，支持它的硬件并不多
            |--[*]  Dock	支持由ACPI控制的集线器(docking stations)
            |--<*>  Processor	让ACPI处理空闲状态，并使用ACPI C2和C3处理器状态在空闲时节省电能
            |--<M>  IPMI
            |--<M>  Processor Aggregator
            |--<*>  Thermal Zone        可以在系统温度过高时，及时调整系统的工作状态，以保护CPU
            |--_*_  NUMA support
            |--()   Custom DSDT Table file to include
            |--[ ]  ACPI tables override via initrd
            |--[ ]  Debug Statements	详细的ACPI调试信息，不搞开发就别选
            |--[*]  PCI slot detection driver
            |--[*]  Power Management Timer Support	这个Timer在所有ACPI兼容的平台上都可用，且不会受PM功能的影响，建议总是启用它。
            |--_*_  Container and Module Devices
            |--[*]  Memory Hotplug
            |--<M>  Smart Battery System	支持依赖与I2C的“智能电池”，这种电池非常老旧且罕见，还与当前的ACPI标准兼容性差
            |--_*_  Hardware Error Device
            |--< >  Allow ACPI methods to be inserted/replaced at run time
            |--[*]  Boottime Graphics Resource Table support
            |--[*]  ACPI Platform Error Interface (APEI)
            |--[*]    APEI Generic Hardware Error Source
            |--[*]    APEI PCIe AER logging/recovering support
            |--[*]    APEI memory error recovering support
            |--<M>    APEI Error INJection (EINJ)
            |--< >    APEI Error Record Serialization Table (ERST) Debug Support
            |--< >  Extended Error Log support (NEW)
        |--[*]  SFI (Simple Firmware Interface) Support ----
        |--     CPU Frequency scaling   --->	允许动态改变CPU主频，达到省电和降温的目的
                |--<*>  CPU frequency translation statistics	通过sysfs文件系统输出CPU频率变换的统计信息
                |--[*]    CPU frequency translation statistics details	输出详细的CPU频率变换统计信息
                |--     Default CPUFreq governor (performance)  --->	默认的CPU频率调节器
                |--_*_  'performance' governor	'性能'优先，静态的将频率设置为CPU支持的最高频率
                |--<*>  'powersave' governor	'节能'优先，静态的将频率设置位cpu支持的最低频率
                |--<*>  'userspace' governor for userspace frequency scaling	允许手动调整cpu频率，也允许用户控件的程序动态的调整cpu频率(需要额外的调频软件，比如cpufreqd)
                |--<*>  'ondemand' cpufreq policy governor	'立即响应'，周期性的考察CPU负载并自动的动态调整cpu频率，适合台式机
                |--<*>  'conservative' cpureq governor	'保守'，和ondemand相似，但是频率的升降是渐变式的(幅度不会很大)，更使用与笔记本/PDA/AMD64环境
                |--     x86 CPU frequency scaling drivers   --->
                        |--[*]  Intel P state control
                        |--<*>  Processor Clocking Control interface driver
                        |--<*>  ACPI Processor P-States driver	将ACPI2.0的处理器性能状态报告给CPUFreq processor drivers以决定如何调整频率
                        |--[*]    Legacy cpb sysfs knob support for AMD CPUs	我没用ADM
                        |--<*>  AMD Opteron/Athlon64 PowerNow!		我不是AMD
                        |--<M>  AMD frequency sensitivity feedback powersave bias	我不是AMD
                        |--<*>  Intel Enhanced SpeedStep (deprecated)
                        |--<M>  Intel Pentium 4 clock modulation
                                *** shared options ***
        |--     CPU Idle    --->
                |--_*_  CPU idle PM support
                |--[*]    Support multiple cpuidle drivers
                |--[*]    Ladder governor (for periodic timer tick)
                |--_*_    Menu governor (for tickless system)
        |--[*]  Cpuidle Driver for Intel Processors
        |--     Memory power savings    --->
 
 - Bus options (PCI etc.) --->用来设置系统总线，根据主板参数自己选择

        |--[*]  PCI support		PCI支持
        |--[*]    Support mmconfig PCI config space access
        |--[ ]    Read CNB20LE Host Bridge Windows
        |--[*]    PCI Exxpress Port Bus support
        |--[*]      PCI Express Hotplug driver	如果主板支持热插拔就悬赏
        |--[*]      Root Port Advanced Error Reporting support  硬件驱动会负责发送错误信息
        |--[ ]        PCI Express ECRC settings control
        |--< >        PCIe AER error injector support
        |--[*]      PCI Express ASPM control
        |--[ ]        Debug PCI Express ASPM
        |--           Default ASPM policy (BIOS default)    --->
        |--_*_    Message Signaled Interrupts (MSI and MSI-X)	PCI Express支持两类中断：INTx使用传统的IRQ中断，可以与现行的PCI总线的驱动程序和操作系统兼容；MSI则是通过inbound Memory Write触发和发送中断，更适合多CPU系统，可以使用“pci=nomsi”内核引导参数关闭MSI
        |--[ ]    PCI Debugging	将PCI调试信息输出到系统日志里
        |--[*]    Enable PCI resource re-allocation detection
        |--<M>    PCI Stub driver
        |--<M>    Xen PCI Frontend
        |--[*]    Interrupts on hypertransport devices	允许本地的hypertransport设备使用中断
        |--[*]  PCI IOV support
        |--_*_  PCI PRI support
        |--_*_  PCI PASID support
        |--<*>  PCI IO_APIC hotplug support
        |--     PCI host controller drivers     ----
        |--[*]  ISA-style DMA support
        |--<M>  PCCard (PCMCIA/CardBus) support --->	PCMCI卡(主要用于笔记本)支持
                |--<M>  16-bit PCMCIA support	一些老的PCMCIA卡使用16位的CardBus
                |--[*]    Load CIS updates from userspace
                |--[*]  32-bit CardBus support	当前的PCMCIA卡基本上都是32位的CardBus
                |--     *** PC-card bridges ***
                |--<M>  CardBus yenta-compatible bridge support	使用PCMCIA卡的基本上都需要选择这一项，子项请按照自己实际使用的PCMCIA卡选择
                |--[*]    Special initialization for O2Micro bridges
                |--[*]    Special initialization for Ricoh bridges
                |--[*]    Special initialization for TI and EnE bridges
                |--[*]      Auto-tune EnE bridges for CB cards
                |--[*]    Special initialization for Toshiba ToPIC bridges
                |--<M>  Cirrus PD6729 compatible bridge support
                |--<M>  i82092 compatible bridge support
        |--[*]  Support for PCI Hotplug --->	PCI热插拔支持
                |--[*]  ACPI PCI Hotplug driver
                |--<M>    ACPI PCI Hotplug driver IBM extensions
                |--[*]  CompactPCI Hotplug driver
                |--<M>    Ziatech ZT5550 CompactPCI Hotplug driver
                |--<M>    Generic port I/O CompactPCI Hotplug driver
                |--<M>  SHPC PCI Hotplug driver
        |--<*>  RapidIO support
        |--<*>    IDT Tsi721 PCI Express SRIO Controller support
        |--(30)   Discovery timeout duration (seconds)
        |--[ ]    Enable RapidIO Input/Output Ports
        |--[*]    DMA Engine support for RapidIO
        |--[ ]    RapidIO subsystem debug message
        |--<M>    Enumeration method
        |--<M>      Basic
        |--       RapidIO Switch drivers    --->
                  |--<*>    IDT Tsi57x SRIO switches support
                  |--<*>    IDT CPS-xx SRIO switches support
                  |--<*>    Tsi568 SRIO switch support
                  |--<*>    IDT CPS Gen.2 SRIO switch support
         |--[*]  Mark VGA/VBE/EFI FB as generic system framebuffer
 - Executable file formats / Emulations --->表示可执行文件格式，一般全要选上
 - _*_ Networking support --->          支持网络，包括网络协议和网络设备，协议中肯定要TCP/IP项，根据自己的网卡选择相应的设备

        |--     Networking options  --->
            |--<*>  Packet socket	这种Socket可以让应用程序直接与网络设备通讯，而不通过内核中的其他中介协议
            |--<M>    Packet: sockets monitoring interface
            |--<*>  Unix domain sockets		一种仅运行于本机上的效率高于TCP/IP的Socket，简称Unix socket.许多程序都使用它在操作系统内部进行进程间通信(IPC)，比如X Window 和 syslog
            |--<M>    Unix: socket monitoring interface
            |--<M>  Transformation user configuration interface	为IPsec(可在ip层加密)之类的工具提供XFRM用户配置接口支持
            |--[ ]  Transformation sub policy support	XFRM自测率支持，仅供开发者使用
            |--[ ]  Transformation migrate database
            |--[ ]  Transformation statistics
            |--<M>  PF_KEY sockets	用于可信任的密钥管理程序和操作系统内核内部的密钥管理进行通信，
            |--[ ]    PF_KEY MIGRATE
            |--[*]  TCP/IP networking	TCP/IP协议
            |--[*]    IP: multicasting	群组广播
            |--[*]    IP: advancde router	高级路由
            |--[*]      FIB TRIE statistics
            |--[*]      IP: policy routing	策略路由
            |--[*]      IP: equal cost multipath	用于路由基于目的地址的负载均衡
            |--[*]      IP: verbose route monitoring	显示冗余的路由监控信息
            |--[*]    IP: kernel level autoconfiguration在内核启动时自动配置ip地址/路由表等，需要从网络启动的无盘工作站才需要这个东西
            |--[*]      IP: DHCP support
            |--[ ]      IP: BOOTP support
            |--[ ]      IP: RARP support
            |--<M>    IP: tunneling	IP隧道，将一个IP报文封装在另一个IP报文内的技术
            |--<M>    IP: GRE demutiplexer
            |--<M>    IP: GRE tunnels over IP
            |--[*]      IP: broadcast GRE over IP
            |--[*]    IP: multicast routing
            |--[ ]      IP: multicast policy routing
            |--[*]      IP: PIM-SM version 1 support
            |--[*]      IP: PIM-SM version 2 support
            |--_*_    IP: TCP syncookie support
            |--<M>    Virtual (secure) IP: tunneling
            |--<M>    IP: AH transformation
            |--<M>    IP: ESP transformation
            |--<M>    IP: IPComp transformation
            |--<M>    IP: IPsec transport mode
            |--<M>    IP: IPsec tunnel mode
            |--<M>    IP: IPsec BEET mode
            |--{*}    Large Receive Offload (ipv4/tcp)
            |--<M>    INET: socket monitoring interface
            |--<M>      UDP: socket monitoring interface
            |--[*]    TCP: advancde congestion control  --->
            ...
        |--[*]  Amateur Radio support   --->	无线电
        |--<M>  CAN bus subsystem support   --->
        |--<M>  IrDA (infrared) subsystem support   --->
        |--<M>  Bluetooth subsystem support --->    蓝牙
        |--{M}  RxRPC session sockets
        |--[ ]    RxRPC dynamic debugging
        |--<M>    RxRPC Kerberos security
        |--_*_  Wireless    --->
        |--<M>  WiMAX Wireless Broadband support    --->
        |--<*>  RF switch subsystem support --->
        |--<M>  Plan 9 Resource Sharing Support (9P2000)    --->
        |--<M>  CAIF support    --->
        |--{M}  Ceph core librajry
        |--[ ]    Include file:line in ceph debug output
        |--[*]    Use in-kernel support for DNS lookup
        |--<M>  NFC subsystem support   --->
 - Device Drivers --->    用来选择设备驱动程序，如声卡、显卡、网卡等驱动

        |--     Generic Driver Options  --->
        |--     Bus devices ----
        |--{*}  Connector - unified userspace <-> kernelspace linker    --->    // 内核空间与用户空间的信道
        |--<M>  Memory Technology Device (MTD) support  --->
        |--<M>  Paralledl port support  ----
        |--<M>  PC-style hardware
        |--<M>    Multi-IO cards (paralledl and serial)
        |--[*]    Use FIFO/DMA if available
        |--[ ]    SuperIO chipset support
        |--<M>    Support for PCMCIA management for PC-style ports
        |--<M>  AX88796 Parallel Port
        |--[*]  IEEE 1284 transfer modes
        |--_*_  Plug and Play support   --->        支持即插即用
        |--[*]  Block devices   --->            支持块设备
        |--     Misc devices    --->
        |--< >  ATA/ATAPI/MFM/RLL support (DEPRECATED)  ----
        |--     SCSI device support --->            支持SCSI设备
        |--<*>  Serial ATA and Parallel ATA drivers --->
        |--[*]  Multiple devices driver support (RAID and LVM)  --->    支持RAID和逻辑卷
        |--<M>  Generic Target Core Mod (TCM) and ConfigFS Infrastructure   --->
        |--[*]  Fusion MPT device support   --->
        |--     IEEE 1394 (FireWire) support    --->
        |--<M>  I20 device support  --->
        |--[*]  Macintosh device drivers    --->    Mac系统硬件设备驱动
        |--_*_  Network device support  --->
        |--     Input device support    --->
		        |--_*_	Generic input layer (needed for ke yboard, mouse, ...)
		        |--{M}	 Support for memoryless force-feedback devices
		        |--{M}	  Polled input device skeleton
		        |--{M}	  Sparse keymap support library
		        |--{M}	  Matrix keymap support library
		        |--	  *** Userland interfaces ***
		        |--<M>	  Mouse interface
		        |--[*]	    Provide legacy /dev/psaux device
		        |--(1024)   Horizontal screen resolution
		        |--(768)    Vertical screen resolutino
		        |--<M>	  Joystcick interface	游戏设备
		        |--<*>	  Event interface	
		        |--<M>	  Event debugging
			          *** Input Device Drivers ***
		        |--[*]	  Keyboards	--->
		        |--[*]	  Mice	--->
		        |--[*]	  Joysticks/Gamepads	--->
		        |--[*]	  Tables	--->	平板PC
		        |--[*]	  Touchscreens	--->	触摸板
		        |--[*]	  Miscellaneous devices	--->
		        |--	Hardware I/O ports	--->
        |--     Character devices   --->
        |--{*}  I2C support --->    // 感知硬件装填，比如温度，风扇转速
        |--[*]  SPI support --->
        |--<M>  HSI support --->
        |--     PPS support --->
        |--     PTP clock support   --->
        |--     Pin controllers --->
        |--_*_  GPIO Support    --->
        |--{M}  Dallas's 1-wire support --->
        |--_*_  Power supply class support  --->
        |--[*]  Adaptive Voltage Scaling class support  ----
        |--{*}  Hardware Monitoring support --->
        |--_*_  Generic Thermal sysfs driver    --->
        |--[*]  Watchdog Timer Support  --->    系统监视程序
        |--     Sonics Silicon Backplane    --->
        |--     Broadcom specific AMBA  --->
        |--     Multifunction device drivers    --->
        |--_*_  Voltage and Current Regulator Support   --->
        |--<M>  Multimedia support  --->
        |--     Graphics support    --->
        |--<M>  Sound card support  --->        声卡驱动，有两种选择，一种是ALSA驱动，一种是Open Sound System驱动，选择支持自己的声卡的那种；
        |--     HID support --->        人力工程学设备
        |--[*]  USB support --->
        |--<M>  Ultra Wideband devices  --->
        |--<*>  MMC/SD/SDIO card support    --->
        |--<M>  Sony MemoryStick card support   --->
        |--_*_  LED Support --->
        |--[ ]  Accessibility support   ----
        |--<M>  InfiniBand support  --->
        |--[*]  EDAC (Error Detection And Correction) reporting --->
        |--[*]  Real Time CLock --->
        |--_*_  DMA Engine support  --->
        |--[*]  Auxiliary Display support   --->
        |--{M}  Userspace I/O drivers   --->
        |--<M>  VFIO Non-Privileged userspace driver framework  --->
        |--[*]  Virtualization drivers  ----
        |--     Virtio drivers  --->
        |--     Microsoft Hyper-V guest support --->
        |--     Xen driver support  --->
        |--[*]  Staging drivers  --->
        |--_*_  X86 Platform Specific Device Drivers    --->
        |--[ ]  Platform support for Chrome hardware (NEW)  ----
        |--     Common Clock Framework  --->
        |--     Hardware Spinlock drivers   ----
        |--[*]  Mailbox Hardware Support    ----
        |--[*]  IOMMU Hardware Support  --->
        |--     Remoteproc drivers  --->
        |--     Rpmsg drivers   ----
        |--[*]  Generic Dynamic Voltage and Frequency Scaling (DVFS) support    --->
        |--_*_  External Connector Class (extcon) support   --->
        |--[*]  Memory Controlloer drivers  ----
        |--{M}  Industrial I/O support  --->
        |--<M>  Intel Non-Transparent Bridge support
        |--<M>  VME bridge support  --->
        |--[*]  Pulse-Width Modulation (PWM) Support    ----
        |--<M>  IndustryPack bus support    --->
        |--[*]  Reset Controller Suuport    ----
        |--<M>  FMC support --->
        |--     PHY Subsystem   --->
        |--[ ]  Generic powercap sysfs driver (NEW) ----
 
 - Firmware Drivers --->		固件驱动程序

        |--<*>  BIOS Enhanced Disk Drive calls determine boot disk	有些BIOS支持从某块特定的硬盘启动(如果BIOS不支持则可能无法启动)
        |--[*]    Sets default behavior for EDD detection to off
        |--[*]  Add firmware-provided memory map to sysfs
        |--<M>  BIOS update support for DELL systems via sysfs	DELL机器特殊选项
        |--<M>  Dell Systems Management Base Driver			又是DELL
        |--[*]  Export DMI identification via sysfs to userspace   将BIOS里的DMI区信息导出到用户空间，部分系统管理工具可能会用到
        |--<M>  DMI table support in sysfs
        |--[*]  iSCSI Boot Firmware Table Attributes
        |--<M>    iSCSI Boot Firmware Table Attributes module
        |--[ ]  Google Firmware Drivers
        |--     EFI (Extensible Firmware Interface) Support --->
 
 - File systems --->

        |--<M>  Second extended fs support
        |--[*]    Ext2 extended attributes
        |--[*]      Ext2 POSIX Access Control Lists
        |--[*]      Ext2 Security Lables
        |--[ ]    Ext2 execute in place support
        |--<*>  Ext3 journalling file system support
        |--[*]    Default to 'data=ordered' in ext3
        |--[*]    Ext3 extended attributes
        |--[*]      Ext3 POSIX Access Control Lists
        |--[*]      Ext3 Security Lables
        |--<*>  The Extended 4 (ext4) filesystem
        |--[*]    Ext4 POSIX Access Control Lists
        |--[*]    Ext4 Security Labels
        |--[ ]    Ext4 debugging support
        |--[ ]  JBD (ext3) debugging support
        |--[ ]  JGBD2 (ext4) debugging support
        |--<M>  Reiserfs support
        |--[ ]    Enable reiserfs debug mode
        |--[ ]    Stats in /proc/fs/reiserfs
        |--[*]    ReiserFS extended attributes
        |--[*]      ReiserFS POSIX Access Control Lists
        |--[*]      ReiserFS Security Labels
        |--<M>  JFS filesystem support
        |--[*]    JFS POSIX Access Control Lists
        |--[*]    JFS Security Labels
        |--[ ]    JFS debugging
        |--[*]    JFS statistics
        |--<M>  XFS filesystem support
        |--[*]    XFS Quota support
        |--[*]    XFS POSIX ACL support
        |--[*]    XFS Realtime subvolume support
        |--[ ]    XFS Ferbose Warnings
        |--[ ]    XFS Debugging support
        |--<M>  GFS2 file system support
        |--[*]    GFS2 DLM locking
        |--<M>  OCFS2 file system support
        |--<M>    O2CB Kernelspace Clustering
        |--<M>    OCFS2 Userspace Clustering
        |--[*]    OCFS2 statistics
        |--[*]    OCFS2 logging support
        |--[ ]    oCFS2 expensive checks
        |--<M>  Btrfs filesystem support
        |--[*]    Btrfs POSIX Access Control Lists
        |--[ ]    Btrfs with integrity check tool compiled in (DANGEROUS)
        |--[ ]    Btrfs will run sanity testts upon loading
        |--[ ]    Btrfs debugging support
        |--[ ]    Btrfs assert support
        |--<M>  NILFS2 file system support
        |--[*]  Enable POSIX file locking API
        |--[*]  Dnotify support
        |--[*]  Inotify support for userspace
        |--[*]  Filesystem wide access notification
        |--[*]    fanotify permissions checking
        |--_*_  Quota support           磁盘配额支持，限制某个用户或者某组用户的磁盘占用空间
        |--[*]  Report quota messages through netlink interface
        |--[ ]  Print quota warnings to console (OBSOLETE)
        |--[ ]  Additional quota sanity checks
        |--<M>  Old quota format support
        |--<M>  Quota format vfsv0 and vfsv1 support
        |--<M>  Kernel automounter version 4 support (also supports v3)
        |--<*>  FUSE (Filesystem in Userspace) support
        |--<M>    Character device in Userspace support
        |--     Caches  --->
        |--     CD-ROM/DVD Filesystems  --->
        |--     DOS/FAT/NT Filesystems  --->
        |--     Pseudo filesystems  --->
        |--_*_  Miscellaneous filesystems   --->
        |--[*]  Network File Systems    --->
        |--_*_  Native language support --->            选Chinese
        |--<M>  Distributed Lock Manager (DLM)  --->
 - Kernel hacking --->

        |--     printk and dmesg options    --->
                Compile-time checks and compiler options    --->
        |--_*_  Magic SysRq key
        |--(0x1)Enable magic SysRq key functions by default (NEW)
        |--_*_  Kernel deugging
        |--     Memory Debugging    --->
        |--[ ]  Debug shared IRQ handlers
        |--     Debug Lockups and Hangs --->
        |--[ ]  Panic on Oops
        |--_*_  Collect scheduler debugging info
        |--_*_  Collect scheduler statistics
        |--[*]  Collect kernel timers statistics
        |--     Lock Debugging (spinlocks, mutexex, etc...) --->
        |--[ ]  Kobject debugging
        |--[ ]  Verbose BUG() reporting (adds 70k)
        |--[ ]  Debug filesystem writes count
        |--[ ]  Debug linked list manipulation
        |--[ ]  Debug SG table operations
        |--[ ]  Debug notifier call chains
        |--[ ]  Debug credential management
        |--     RCU Debugging   --->
        |--[ ]  Force extended block device numbers and spread them
        |--<M>  Notifier error injection
        |--<M>    CPU notifier error injection module
        |--<M>    PM notifier error injection module
        |--[ ]  Fault-injection framework
        |--[*]  Latency measuring infrastructure
        |--[ ]  Strict user copy size checks
        |--[*]  Tracers --->
        |--     Runtime Testing --->
        |--[ ]  Remote debugging over FireWire early on boot     启动过程中，允许远程调试内核
        |--[ ]  Remote degugging over FireWire with firewire-ohci
        |--[ ]  Enable debugging of DMA-API usage
        |--[ ]  Sample kernel code  ----
        |--[*]  KGDB: kernel debugger   --->
        |--[*]  Filter access to /dev/mem
        |--[ ]  Enable verbose x86 bootup info messages // 在内核镜像解压缩阶段输出启动信息，关闭后相当于无声启动(Slient Bootup)
        |--[*]  Early printk
        |--[*]    Early printk via EHCI debug port
        |--[ ]    Early printk via the EFI framebuffer (NEW)
        |--[ ]  Export kernel pagetable layout to userspac via debugfs
        |--[*]  Write protect kernel read-only data structures
        |--[ ]    Testcase for the DEBUG_RODATA feature
        |--[*]  Set loadable kernel module data as NX and text as R0
        |--< >  Testcase for the NX non-executable stack feature
        |--[*]  Enable doublefault exception handler
        |--[ ]  Set upper limit of TLB entries to flush one-by-one
        |--[ ]  Enable IOMMU debugging
        |--[ ]  Enable IOMMU stress-test mode
        |--[ ]  x86 instruction decoder selftest
        |--     IO delay type (port 0xed based port-IO delay)   --->
        |--[ ]  Debug boot parameters
        |--[ ]  CPA self-test code
        |--[*]  Allow gcc to uninline functions marked 'inline'
        |--[ ]  NMI Selftest
        |--[ ]  Debug alternatives
 - Security options --->

        |--_*_  Enable access key retention support
        |--[ ]    Enable register of persistent per-UID keyrings (NEW)
        |--[ ]    Large payload keys (NEW)
        |--<*>    TRUSTED KEYS
        |--_*_    ENCRYPTED KEYS
        |--[ ]    Enable the /proc/keys file by which keys may be viewed
        |--[ ]  Restrict unprivileged access to the kernel syslog
        |--[*]  Enable different security models
        |--_*_  Enable the securityfs filesystem
        |--_*_  Socket and Networking Security Hooks
        |--[ ]    XFRM (IPSec) Networking Security Hooks
        |--_*_  Security hooks for pathname based access control
        |--[*]  Enable Intel(R) Trusted Execution Technology (Intel(R) TXT)
        |--(0)  Low address space for LSM to protect from user allocation
        |--[*]  NSA SELinux Support
        |--[*]    NSA SELinux boot parameter
        |--(0)      NSA SELinux boot parameter default value
        |--[*]    NSA SELinux runtime disable
        |--[*]    NSA SELinux Development Support
        |--[*]    NSA SELinux AVC Statistics
        |--(1)    NSA SELinux checkreqprot default value
        |--[ ]    NSA SELinux maximum supported policy format version
        |--[*]  Simplified Mandatory Access Control Kernel Support
        |--[*]  TOMOYO Linux Support
        |--(2048) Default maximal count for learning mode
        |--(1024) Default maximal count for audit log
        |--[ ]    Activate without calling userspace policy loader.
        |--(/sbin/tomoyo-init) Location of userspace policy loader
        |--(/sbin/init) Trigger for calling userspace policy loader
        |--[*]  AppArmor support
        |--(1)    AppArmor boot parameter default value
        |--[*]    SHA1 hash of loaded profiles
        |--[*]  Yama support
        |--[*]    Yama stacked with other LSMs
        |--[*]  Digital signature verification using multiple keyrings
        |--[*]  Enables integrity auditing support
        |--[*]  Enable asymmetric keys support
        |--[ ]  Integrity Measurement Architecture(IMA)
        |--[*]  EVM support
        |--(2)    EVM HMAC version
        |--     Default security module (AppArmor)  --->
 - _*_ Cryptographic API --->  加密API，这部分选项会根据此前的优化自动调整，默认即可
 - _*_Virtualization --->    虚拟化
 - Library routines --->     库子程序，这部分选项会根据此前的优化自动调整，默认即可
 
参考：

http://www.cnblogs.com/ruiy/p/kernel.html
http://lamp.linux.gov.cn/Linux/kernel_options.html(金步国)
