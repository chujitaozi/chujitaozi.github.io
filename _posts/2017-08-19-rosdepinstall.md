---
layout: post
title:  rosdep install
description: 命令rosdep install的重要性
img:
date: 2017-08-19  +0800
---
今天安装rgbdslam的indigo ros包的时候，按说明安装了g2o和pcl，自以为不用按照说明的步骤来，看了两眼readme给出的命令就觉得这些命令已经都执行过了，直接catkin_make吧！

结果出现了CMAKE_CXX_FLAGS变量设置的问题，在网上百度之后，按说明改之。catkin_make通过，但是节点启动之后马上died

于是乎，重新认真阅读readme，发现

```
rosdep update

rosdep install rgbdslam
```

没有运行也不懂作用是什么。于是重新运行，发现虽然安装了g2o，但是没有安装ros-indigo-libg2o，其实这个命令的作用就是安装rgbdslam的依赖项

于是乎，删除build和devel，重新catkin_make。

节点没问题啦～，改改topic对应起来就好啦
