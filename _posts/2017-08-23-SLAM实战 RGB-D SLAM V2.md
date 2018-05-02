---
layout: post
title:  SLAM实战RGB-D SLAM V2
description: 使用handsfree机器人Stone进行RGBDSLAM
img:
date: 2017-08-23  +0800
---

# 写在前面

我用的是handsfree团队的Stone机器人,摄像头是Xtion;
我的实战是结合半闲居士的SLAM实战参考着来的,所以文章结构也按照这个来(就是这么臭不要脸)
原文链接:http://www.cnblogs.com/gaoxiang12/p/4462518.html
这次跑得是视觉SLAM的经典程序:RGBD SLAM V2

# 实验器材

## 1,硬件

高博用的机器人是改造的turtlebot,Viewbot,差不多啦,其实只要上手快就行.**但是!!!**,一个机器人,要一万多!对学生太特么不友好了.严重阻碍了我们学生对机器人的学习.

**前方广告**<center>**关键字!!!**</center>

<center>**便宜**</center>
<center>**简单**</center>
<center>**上手快**</center>

在此郑重介绍Handsfree团队的机器人,现在主打的Mini,和Stone,都贼便宜;基础版的两三千,自己买个摄像头按上去就可以跑SLAM;为啥这么便宜呢,可以去搜一搜他们,毕竟都是学僧;
贴个高配版:
![这里写图片描述](http://img.blog.csdn.net/20170823221356394?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzA1MTM1OTM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
我用的是Stone机器人.加Xtion摄像头

## 软件

本机为ubuntu14.04,ROS版本为indigo

# SLAM程序

和原作的不同,我用的是indigo,所以在下载程序的时候,我下载的是indigo版本,也就是在github上选择branch,换成indigo,下载链接在https://github.com/felixendres/rgbdslam_v2/tree/indigo
下载zip或者

```
git clone -b indigo https://github.com/felixendres/rgbdslam_v2.git

```

然后按照readme,装装装!主要就是opencv和pcl,还有一个g2o,这个我在另一个博客里已经写了,执行

```
rosdep install rgbdslam
```

就会直接自动安装

## 按照说明

mkdir啊catkin_make啊等等,就是把这个包编译一下,搞好之后,就可以直接roslaunch啦!

# 运行程序

直接roslaunch的话,貌似是不行滴,然后我看了一下,貌似是topic没有对应上,还有摄像头的某个服务好像搞错了,rgbdslam这个节点接受不到照片信息.

## 小改动以适应Stone

将depth_registration和sw_registered_processing参数设置为true
在rgbdslam的launch中加入

```
<remap from="/camera/rgb/image_color" to="/camera/rgb/image_rect_color" />
```

因为我的摄像头没有输出rect....偏偏rgbdslam要Sub

程序运行良好,就是我的笔记本不太给力...想换电脑,就是没钱....
