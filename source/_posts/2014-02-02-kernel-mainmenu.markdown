---
layout: post
title: "Linux内核个性化配置"
date: 2014-02-02 11:21:03 +0800
comments: true
categories: [Kernel,linux]
tags: [Kernel]
---

这次更新内核的时候忘了先用3.13.1来进行通用配置安装，所以只能和3.13.0比较，分别是5.6MB和5.8MB，只减少了200kb左右，其实作为个人桌面用户，这一些修改都还太少，主要原因是我很多功能并不知道有什么用(而且貌似有几个选项没有选，下次吧，编译内核太费CPU太费时间太费电了)，而且我对自己的电脑硬件也不甚了解，希望以后能逐步完善精简内核的选项：
###Linux内核配置选项简介
<!--more-->
目标内核：[Latest Stable Kernel: 3.13.1](https://www.kernel.org)

电脑环境：ubuntu13.04

由于内核的配置选项过于麻烦，所以这次更新到3.13.1的时候特地整理了一下这些配置菜单：

首先，在我之前的安装内核说明的文章里说道执行`make menuconfig`时会显示主菜单以及操作说明：

    Arrow keys navigate the menu. <Enter> selects submenus ---> (or empty submenus ----). Highlighted letters are hotkeys. Pressing <Y> includes, <N> excludes, <M>modularizes features. Press <Esc><Esc> to exit, <?> for Help, </> for Search. Legend: [*] built-in [ ] excluded <M> module < > module capable      即用方向键操作菜单，回车进入子菜单，高亮的字母表示该选项的快捷方式。按Y表示编译进内核额，按N表示不编译进内核，按M表示编译位模块。按两下Esc返回上一级菜单，按？表示帮助，/表示搜索。
选项前四种括号的意义：
     
    [*]：表示选取了该选项，编译好后的kernel就会有该功能
    [ ]：表示未选取该项，编译后的kernel不会有此功能
    <M>：表示选取了该选项，而且是编译成模块module的形式，它会在kernel被载入后被动态地加载，编译成module可以减少kernel image的空间，加快开机速度，方便以后修改
    < >：表示未选取该项，但是该功能被当做module，今后可以在开机后另外载入
####主菜单：
    
    |--[*]	64-bit kernel					此项决定内核是64位的
	General setup	--->				常规选项，与Linux最相关的、核心版说明、是否使用程序代码、是否使用虚拟内存等都在这里设定
    |--_*_	Provide system-wide ring of trusted keys
    |--[*]	Enable loadable module support	--->	可加载模块支持
    |--[*]	Enable the block layer	--->			块设备层
    |--	Processor type and features	--->		    处理器类型和功能
    |--	Power management and ACPI options	--->	电源管理和ACPI选项
    |--	Bus options (PCI etc.)	--->			    总线选项
    |--	Executable file formats / Emulations	--->可执行文件格式/仿真
    |--_*_	Networking support	--->			    网络支持
    |--	Device Drivers	--->				        设备驱动
    |--	Firmware Drivers	--->			        固件驱动
    |--	File systems	--->				        文件系统
    |--	Kernel hacking	--->				        内核监视
    |--	Security options	--->			        安全选项
    |--_*_	Cryptographic API	--->			    加密选项
    |--_*_	Virtualization	--->				    虚拟化
    |--	Library routines	--->			        程序库程序
### 目前发现的可修改选项
 - General setup

    	...
     	|--()	Local version - append to kernel release	自定义版本名称，即之后可用uname -r 看到的字符串，可以修改，但没多大意义
        ...
        |--	Kernel compression mode (Gzip)	--->	内核镜像要用的压缩模式，最好选更新的Bzip2,压缩效率较好
	    ...
     	|--	Timer subsystem	--->
		    ...
		    |--[*]	High Resolution Timer Support	高精度内核定时器，一般用户可关闭
	        ...   
	    |--_*_	Control Group support	--->	cgroups支持，文档资料，cgroups主要作用是给进程分组，并可以动态调控进程组的CPU占用率，一般桌面用户没有必要，所以可以完全禁止，如果不完全禁止，那也可以禁止下面几个选项
		    ...
		    |--[*]	Cpuset support	CPU分组支持，只有含有大量CPU(大于16个)的SMP系统或NUMA(非一致内存访问)系统才需要它,所以这里可以禁用它
		    |--[*]	Simple CPU accounting cgroup subsystem	cgroup子系统的简单CPU统计，会生成一个资源控制器来监控，在一个cgroup组内的独立任务的CPU使用情况,可禁止
		    |--[*]	Resource counters	资源计数器，使控制器的独立资源统计功能能够统计cgroup，可禁止
		    |--[*]  Enable perf_event per-cpu per-container group (cgroup) monitoring	允许开发者扩展每个CPU的模式，使它可以只监视运行在特定CPU上的一个特别的cgroup组的线程
	        ...
	    ...
	    |--	Choose SLAB allocator (SLUB (Unqueued Allocator))	--->使用SLAB完全取代SLOB进行内存分配，SLAB是一种优秀的内存分配管理器，推荐使用
	    |--[*]	Profiling support	对系统的活动进行分析，仅供内核开发者使用,应该可以禁用
	    ...
	    |--[*]	Kprobes 		仅供内核开发者使用
 - _*_	Provide system-wide ring of trusted keys
 - [*]	Enable loadable module support	--->
 - [*]	Enable the block layer	--->
 - Processor type and features --->	处理器类型及特性

    	...
    	|--[*]  Support for etended (non-PC) x86 platforms	支持除x86的其他平台，不用选撒
	    ...
    	|--     Processor family (Generic-x86-64)   --->    这里选择相应的CPU
	    |--[*]	Supported processor vendors	--->	选择相应的处理器供应商
	    ...
	    |--(256)Maximum number of CPUs	支持的最大CPU数，每增加一个内核将增加8k体积，我的是双核心四线程，所以选4
	    ...
	    |--[*]	Machine Check / overheating reporting   
    	|--[*]    Intel MCE features
    	|--[*]    AMD MCE features          这两项貌似只根据自己的CPU来选
    	|--<M>  Machine check injector support
    	|--<M>  Dell laptop support			Dell笔记本模块支持，我用的宏碁
    	|--<M>  CPU microcode loading support
    	|--[*]    Intel microcode loading support	这也根据自己的CPU来选
    	|--[*]    AMD microcode loading support
	    ...
	    |--[*]	Old style AMD Opteron NUMA detection	 老的AMD什么，反正我没用AMD
	    ...
	    |--     Timer frequency (250HZ)    --->    内核始终频率，默认250HZ，如果是桌面用户就改为1000HZ
	    ...
	    |--[*]	kernel crash dumps	内核崩溃时，dump运行时信息，一般桌面用户应该用不到吧，我暂时不会去研究这个的
	    ...
    	|--[*]  Build a relocatable kernel  建立一个移动的内核，并增加10%的内核尺寸，运行时会被丢弃，他认为没有实质性作用

 - Power management and ACPI otions --->(ACPI、APM)是高级电源管理。要注意ACPI与APM不能同时使用,所谓高级电源管理，如软关机，系统休眠，智能省电等功能

	    ...
	    |--[*]  Power Management Debug Support	仅供调试使用
	    ...
	    |--	CPU Frequency scaling	--->
		    ...
		    |--	x86 CPU frequency scaling drivers	--->
			...
			|--<*>	AMD Opteron/Athloan64 PowerNow!	又是AMD	
			|--<M>	AMD frequency sensitivity feedback powersave bias
			...
 - Bus options (PCI etc.)	--->
 - Executable file formats /Emulations	--->
 - _*_ Networking support --->          网络支持，包括网络协议和网络设备

	    ...
	    |--[*]  Amateur Radio support   --->	无线电，高级货
	    ...
	    |--<M>	Bluetooth subsystem support	--->	蓝牙，我电脑没这东西


 - Device Drivers --->    用来选择设备驱动程序，如声卡、显卡、网卡等驱动

	    ...
	    |--[*]  Macintosh device drivers    --->    Mac系统硬件设备驱动，我可不是土豪
	    ...
    	    |--     Input device support    --->
		            ...
		            |--<M>	Joystcick interface	游戏设备
		            ...
		            |--[*]	  Tables	--->	平板PC
		            |--[*]	  Touchscreens	--->	触摸板
		            ...
	    ...
	    |--<M>	Sony MemoryStick card support	--->	又扯到索尼了
	    ...
	    |--_*_	X86 Platform Specific Device Drivers	--->	这里面有很多电脑品牌，很明显只选自己的

 - Firmware Drivers	--->
	
        ...
	    |--<M>	Dell Systems Management Base Driver	戴尔的

 - File systems --->				文件系统的前面几项，我是都用的Ext4的文件系统

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
 - Security options	--->
 - Cryptographic API	--->
 - VIrtualization	--->
 - Library routines	--->
