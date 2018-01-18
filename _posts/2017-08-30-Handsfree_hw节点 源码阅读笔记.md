---
layout: post
title:  handsfree_hw节点源码阅读笔记
description: Handsfree的底层节点的阅读笔记
img:
date: 2017-08-30  +0800
---
#按顺序梳理
##main
main函数看起来很简单啊...
初始化节点
创建命名空间handsfree...
##HF_HW_ros 初始化hf
构造函数先进行初始化:
*初始化hf_hw_
&emsp;**寻找port,并初始化hflink
&emsp;**处理config文件,最后设置initialize_ok_
*初始化节点处理句柄,将nh_初始化为handsfree,pub等
*设置参数
&emsp;**回调队列
&emsp;**机器人参数:arm,freq,底座模式等等
&emsp;**初始化系统状态,x,y,z等
*注册在robothw上的硬件接口(robothw部分没仔细打开看)
##进入mainloop
*创建命名空间mobile_base,并初始化
&emsp;**创建回调队列并在controller中注册
&emsp;**为两个队列开辟线程
*当ros::ok()时,进入mainloop
&emsp;**handshake,(shaking_hands为什么要把measure_globle_coordinate发送给机器人呢?)
&emsp;**读取机器人信息并发布
&emsp;**更新readbuffer
&emsp;**更新writebuffer并给机器人发送更新命令
*返回进行循环
#结构认识
感觉代码太多了...而且上面要和ros的API交流数据,下面要通过ttyUSB0和机器人交流指令,看得想吐...
##hf_hw_ros.h
*首先它包含了HF_HW一个对象hf_hw_
*定义了用于在controller中注册的xyz,cmd等,controller中的原理不是很清楚,貌似是可以将队列中的消息自动分类??
*定义读取和写入机器人消息的内联函数,机器人定义在HF_HW中
```
RobotAbstract my_robot_;
```
只有一个机器人的定义,在hflink中机器人的参数是直接由这个机器人的定义初始化过去的,在HF_HW的构造函数中就已经初始化
**HF_HW_ROS主要用来承接ros和硬件部分,在controller中接收消息,存储到my_robot_,然后,updatecommand,用不同的command设置和读取不同参数**
command就属于HF_HW啦
##hf_hw
*包含智能指针port和hflink
*updatecommand会按照设定好的频率调用sendcommand,并检查hflink的package行不行
*sendcommand函数会将命令和数据整理好,然后通过transport.h传给机器人
**感觉hf_hw是比较主要的部分,因为它把所有的资源都用到了,ros,hflink,transpot.**
##hf_link
是state_machine的子类,不是特别明白为什么要命名成这样?
*根据不同的命令,mastersendcommand写入不同的参数,调用sendstruct
*setCommandAnalysis是多机协同用的么?
*它继承了state_machine的tx,rx等,在h文件中还定义了handsfreemessage,命令和Recstate的枚举
**并没有很看懂hflink的交流过程...只是大概看懂了数据的流向**
***
#总结

*只看懂了大概的流程,还有类的作用等,实际运行过程中程序的先后和时间顺序控制(尤其是hflink和transport)都不是很清楚.
*感觉自己还是没有能领悟到精髓,只感觉跳来跳去...看起来很费劲...
*感觉自己差距还很大...我觉得需要拿起C++...再看看
**下一步计划,C++方面,再看看C++primerplus,看看qt;然后接着看slam,跑demo,跑跑不同的slam;
