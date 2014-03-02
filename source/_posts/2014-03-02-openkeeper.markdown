---
layout: post
title: "重邮等高校在linux下使用openkeeper代替netkeeper连接网络"
date: 2014-03-02 15:27:27 +0800
comments: true
categories: [linux, solution]
tags: [linux]
---
首先下载大神们做好的软件包：[点此下载]()（其实这里面也有安装说明的）

然后执行以下命令：

    tar -jxv -f openkeeper-cli-1.1.tar.bz2  # 解压缩到当前目录
    cd openkeeper-cli-1.1                   # 进入目录
    ls                                      #显示当前目录文件
    32  64  build-essential_11.6ubuntu4_amd64.deb  README
如果电脑没有安装build-essential需要先安装它，如果有归档管理器，那么双击`build-essential_11.6ubuntu4_amd64.deb`即可安装，如果没有或者是你的电脑是32位系统那么就执行`sudo apt-get install build-essential`即可安装依赖
<!--more-->
然后进入进入目录安装，执行：

    cd 64               # 进入该目录，如果是32位系统请选择32这个文件夹
    sudo ./install.sh   # 执行安装命令
这样就安装好了，安装好后，可以使用如下命令进行配置和连接(此时不一定要当前目录的，因为这几个命令已经放到了你的/usr/bin/中去了)：

    ok-config       # 配置网络参数，这里会要求输入用户名密码以及网卡(网卡默认eth0)
    okok            # 这一点与README里介绍的不同，我想是因为这个东西是后人对前人改进了的，以前的版本会每十分钟断一次的
    okok-stop       # 断开网络
    
如此，便可以连接外网了，如果要连接内网，可以进行如下配置：

点击网络设置，如下图：

![cao](images/logo.jpg)

如果没有该网络管理软件，可执行`sudo apt-get install network-manager-network`安装

然后点击以太网里面的一个选择编辑，出现如下对话框：

![cao](../images/logo.jpg)

然后选择

