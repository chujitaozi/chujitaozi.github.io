---
layout: post
title:  systemback的使用
description: systemback可以用来备份系统，还可以用来制作iso文件，是个好工具。
img:
date: 2018-01-31  +0800
---

# 安装

```
sudo add-apt-repository ppa:nemh/systemback
sudo apt-get update
sudo apt-get install systemback
```

# 制作iso

* 在dash菜单中搜索systemback，然后点击运行。输入密码   
* Live system create
* inclue user data要选上
* create new

* 生成结束后，转为iso文件

# 安装镜像

* 先在虚拟机中创建一个Ubuntu 64位的系统，分配好内存，硬盘，如果不知道具体选什么可以按照默认选项。
* 在启动时选择创建好的iso文件
* 在开机界面选择Boot System installer
* 填写用户名和密码 //若保留用户资料，填写以前的，不是以前的话，还在试
* 分区直接分到一个根目录下，且选择保留之前用户数据。（如果自己知道如何分区，也可以自己分区，注意要有一个/boot/efi的分区）
* 安装

