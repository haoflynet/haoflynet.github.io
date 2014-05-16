---
layout: post
title: "新电脑上部署octopress博客"
date: 2014-05-16 19:43:54 +0800
comments: true
categories: [octopress, config]
tags: [config]
---
由于不想给云端笔记(wiznote)增加压力，所以还是直接写成博客文章为好。不过需要注意的是本文章只是针对曾经搭建过的仓库。

1.安装基础的依赖及所需的工具：

    sudo apt-get install git curl
    curl -L get.rvm.io | bash -s stable #安装rvm
    source ~/.bashrc
    source ~/.bash_profile
    sed -i -e 's/ftp\.ruby-lang\.org\/pub\/ruby/ruby\.taobao\.org\/mirrors\/ruby/g' ~/.rvm/config/db # 修改源
    sudo apt-get install ruby1.9.3 # 安装ruby1.9.3
    gem sources -a http://ruby.taobao.org/  # 替换gem源
    gem sources -r http://rubygems.org/ 
    gem sources -l  # 是否只有淘宝的了
<!--more-->
2.建立SSH

    ssh-keygen -t rsa -C haoflynet@gmail.com
之后会确认密玥保存的位置，一般直接回车保存在默认的位置即可。之后在刚才确认的目录下可以看到了.ssh目录，里面又两种文件is_rsa(私有密钥)和id_rsa.pub(共有密玥)，此时直接把共有密玥拷贝到github上的SSH key即可(在账户设置里面的SSH KEY选项)

3.配置git

    git config --global user.name haoflynet
    git config --global user.email haoflynet@gmail.com
4.克隆到本地

    git clone -b source git@github.com:haoflynet/haoflynet.github.com.git blog
    cd blog
    git clone git@github.com:haoflynet/haoflynet.github.com.git _deploy 

5.进入博客安装其他工具

    sudo gem install bundler
    bundle install 
6.更新本地仓库

    git pull origin source 
    cd ./_deploy  
    git pull origin master
7.之后就可以新建一篇文章进行测试了
    cd ..   # 回到blog目录
    rake new_post["test"]
    rake generate
    rake deploy
    git add .
    git commit -m "test"
    git push origin source

