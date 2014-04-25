---
layout: post
title: "Ubuntu 无法调节屏幕亮度解决方法"
date: 2014-04-26 00:11:19 +0800
comments: true
categories: solutions
tags: solutions
---
有几个Ubuntu版本总是会出现无法调节屏幕亮度或者无法直接用fn键调节亮度，包括最新的Ubuntu 14.04 也有这个Bug，当然，只是少数的电脑会出现这种情况。每次遇到这种问题都会到网上去寻找答案，但是很多次用相同的方法无法解决，所以这里归纳了一些解决方法
<!--more-->
###方法一
最常用的方法，网上到处都能找到的
首先编辑grub文件，输入命令`sudo gedit  /etc/default/grub`
将`GRUB_CMDLINE_LINUX=""`这一行改为：  
`GRUB_CMDLINE_LINUX="acpi_backlight=vendor"`  
然后更新grub：
`sudo update-grub`  
然后设置初始亮度：`sudo gedit /etc/rc.local`  
在`exit 0`这行代码之前加上下面这一句：
`echo 100 > /sys/class/backlight/intel_backlight/brightness`  
最后重启
    
###方法二
此方法来自[IT之家](http://www.ithome.com/html/soft/71265.htm)刚好可用于我目前这个版本  
执行：`sudo gedit /usr/share/X11/xorg.conf.d/20-intel.conf`
然后添加如下代码：  

    Section "Device"
    Identifier "card0"
    Driver "intel"
    Option "Backlight" "intel_backlight"
    BusID "PCI:0:2:0"
    EndSection
最后重启

