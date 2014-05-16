---
layout: post
title: "Linux装机必备"
date: 2014-05-16 21:29:00 +0800
comments: true
categories: [linux, config]
tags: 装机
---
本来想写个afterinstall的shell的，但是貌似只有到暑假才有时间了，所以这里就只是记录一下平时经常用的linux软件吧。以下软件如没有特殊说明，均可以通过apt-get或者软件中心下载。
##系统工具
 - build_essential：安装openkeeper之前必须安装的  
 - gcc：GNU的C语言编译器，一般发行版都内置了的  
 - vim：非Emacs党必备  
 - gdb：调试程序用，一般也是内置了的  
 - trash-cli：防止mv命令对文件误删，先`apt-get install trash-cli`，然后执行`alias rm=trash`以后再用rm删除的文件都能在回收站中找到  
 - 新立得软件包：无论是Deepin自带的还是Ubuntu自带的软件中心在管理已安装软件的方面都没它强大，包名叫synaptic  
<!--more-->
##网络工具
 - Chrome：其实我也挺喜欢火狐的，但先接触的是Chrome，并且书签账号什么的都是使用的gmail，所以也就一直使用这个了，虽然内存占的有点大，但是运行流畅，并且操作都是我喜欢的类型。等几天再写一篇文章介绍一下我所使用的一些插件  
 - network-manager-gnome：由于Deepin Linux的网络设置和Ubuntu不大一样，所以需要使用这个工具来连接802.1x，即连接内网  
 - axel：linux下的多线程下载工具，一般wget不行就用这个  
金山快盘：金山还是有几款良心作品。我使用金山主要是为了在linux平台下实现自动备份，对于一个程序员，自动备份太重要了。不过它貌似目前只能自动备份桌面，无法备份整个home目录，要实现这个功能，还需要在桌面添加对应的快捷方式，[官网下载](http://www.ubuntukylin.com/applications/showimg.php?lang=cn&id=21)  
##开发工具
 - Code::Blocks：一个开放源代码的全功能的跨平台的C/C++ IDE，windows下也比较习惯使用这个，比VS小很多  
 - xMind：国产的制作思维导图的工具，[官网下载](http://www.xmind.net/download/linux/)
##其他工具
 - 搜狗输入法：算是搜狗对开源世界一个巨大的贡献了，安装方法见[搜狗输入法 for linux](http://pinyin.sogou.com/linux/)  
 - 为之笔记：少有的能在linux平台下使用的国产云端笔记应用，Deepin Linux可以直接在软件中心下载，其他发行版，可以直接在[官网](http://www.wiz.cn/download.html)下载linux版本  
 - Virtualbox：虚拟机，不解释，virtualbox是开源免费的，如果想在linux使用windows系统，就使用它了  
 - smplayer：SM播放器，别邪恶了，这可是linux上功能很强大的播放器，我使用它主要是代替深度影音，目前深度影音还未开发完备，又很多bug和不足的地方  
 - wally：可以实现自动切换壁纸，不过要记得设置为开机启动  
 - rar：解压rar文件  
 - bafeFTP：linux下的FTP客户端  
 - 茄子：linux下的“镜子”，功能还算强大吧，apt安装的时候包名叫cheese  
