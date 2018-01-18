---
layout: post
title:  SLAM的傻瓜说明手册
description: 读SLAM for Dummies有感
img:
date: 2017-08-15  +0800
---
# 初识SLAM   

&#8195;&#8195;之前只是大概的知道SLAM概念，但是看哪部分都是糊里糊涂，偶尔在知乎上看见了有人推荐入门教程，*SLAM for Dummies*,英文版很好找，词汇也都还挺简单的，就看了一下，作为本人有生以来第一个博客发一下，哈哈（师兄教的，发博客还有利于自己理解和记忆，哈哈）   
&#8195;&#8195;那个傻瓜教程介绍了很多东西，我按照自己的整理了一些   
## WHY SLAM
&#8195;&#8195;里程计Odom不准确并不能如实反映自己所在位置，误差大，就算利用雷达测出周围有东西，也不能很好mapping，所以利用EKF（扩展卡尔曼滤波），结合里程计和路标干活。   
## 大致处理过程
&#8195;&#8195;SLAM包含很多部分，主要有：路标提取，数据关联，状态估计，状态更新，地标更新。处理过程大致如图：   
![这里写图片描述](http://img.blog.csdn.net/20170815204923661?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzA1MTM1OTM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
## 小零件
### handware硬件
 - **机器人**
 这个不用说，没个机器人难道拿着雷达到处跑？   
 - **range measurement device**
 就是环境探测装置啦，没有环境探测也不是事啊，总不能自己站在机器人上面看吧，这个还要分一下   
 1,laser scan,激光雷达，比较适用，但是贵（估计是写这篇文章的时候贵，现在不是特别贵了已经），比较好，但是激光测量距离不长，遇到玻璃一些折射反射等，水里会出问题，这个只能是2D啊！   
 2，sonar,声纳，这个比较适用在水里，声波嘛   
 3，vision，视觉，这个太TM火了，可以建3D地图啊，应用广泛，室内飞行器等等，很有潜力，但是问题也贼多，处理图片速度啊，光的强度对version的影响啊等等，我以后应该会看一些这个吧   
### laser scan
 &#8195;&#8195;文章主要介绍了一下激光雷达，因为写这个的人是用的激光，激光雷达可以测量从min的一个角度到max的每隔一个wide的角度上，障碍到雷达的距离，数据就是从min角度开始，[2.88 2.99 ....8.1,2.99..]的一个矩阵，当激光数据err或者不可测量时，会输出一个特别大的数据，如8.1，单位m   
### Odometer
&#8195;&#8195;里程计，会提供一个由轮子算出来的position。   
&#8195;&#8195;问题主要是需要雷达数据和里程计数据同步，所以需要一个控制部分可以同时获取两者消息。   
## 处理部分
### landmark
 - **路标的提取**
路标的提取有几个关键   
1，容易被re-observable   
2，应易于区分   
3，should be plentiful（多了应该不是不容易区分了么？？）   
4，应该是固定的   
 - **提取方法**
方法主要由路标和senser决定，书中用了laser和室内，所以介绍了两种方法：   
1，spike landmarks，峰值提取，找到diff from other的点进行记录   
2，RANSAC（Random Sampling Consensus），在laser scan中提取直线   
3，other，还有multiple，可能是多种一起上？？？   
## Data Association，数据关联   
### problem
 - 可能不能每步都能re-observe   
 - 可能认不出已经标出的之前的landmark   
 - 可能认错landmark   
### solution
rule：不能相信一个路标，除非我们已经看见过它很多次，下面是关联过程中的概念   
 - **database**，首先应该有一个database to store landmarks
 - **validation gate**,区分landmark是否以前见过
**过程**：
1，extract landmarks   
2，associate landmarks with 在database中最近的landmark，把landmark都结成对   
3，将这些pairs送到validation gate验证，   
&#8195;&#8195;1)matched，将在database中landmark的被提取次数，+1   
&#8195;&#8195;2)没有匹配上，将路标加入database，并将time设置为1   
* 说明：步骤二中，用的是the nearest-neighbor approach，计算欧式距离？？（不懂）。步骤三中，validation gate 和EKF有关，大概就是划定一个容错圈，进了就说明差不多   
***
啊呀呀，其实最主要的那部分还是EKF，我还没看（哭脸），等我先大概看看EKF是干嘛的再去看，下次把最主要核心的部分EKF写写。路漫漫兮！   
2017.8.15 周二 晴   


