---
layout: post
title:  a simple person-follower
description: 一个简单的人体（物体）跟踪的例子。
img:
date: 2018-04-04  +0800
---

回头再看一遍，才有更多收获。这是一个简单的跟踪节点的脚本，展开记录一下。

# 1,导入所需模块

```
import rospy
#创建节点所需
from roslib import message
from sensor_msgs import point_cloud2
from sensor_msgs.msg import PointCloud2
from geometry_msgs.msg import Twist
#数据类型
from math import copysign
#数学函数
```

# 2,类的创建及初始化

```
class Follower():
```

## 2.1初始化类及参数

``` 

    def __init__(self):
        rospy.init_node("follower")
        
        # Set the shutdown function (stop the robot)
        rospy.on_shutdown(self.shutdown)
        
        # The dimensions (in meters) of the box in which we will search
        # for the person (blob). These are given in camera coordinates
        # where x is left/right,y is up/down and z is depth (forward/backward)
        self.min_x = rospy.get_param("~min_x", -0.2)
        self.max_x = rospy.get_param("~max_x", 0.2)
        self.min_y = rospy.get_param("~min_y", -0.3)
        self.max_y = rospy.get_param("~max_y", 0.5)
        self.max_z = rospy.get_param("~max_z", 1.2)
        
        # The goal distance (in meters) to keep between the robot and the person
        self.goal_z = rospy.get_param("~goal_z", 0.6)
        
        # How far away from the goal distance (in meters) before the robot reacts
        self.z_threshold = rospy.get_param("~z_threshold", 0.05)
        
        # How far away from being centered (x displacement) on the person
        # before the robot reacts
        self.x_threshold = rospy.get_param("~x_threshold", 0.05)
        
        # How much do we weight the goal distance (z) when making a movement
        self.z_scale = rospy.get_param("~z_scale", 1.0)

        # How much do we weight left/right displacement of the person when making a movement        
        self.x_scale = rospy.get_param("~x_scale", 2.5)
        
        # The maximum rotation speed in radians per second
        self.max_angular_speed = rospy.get_param("~max_angular_speed", 2.0)
        
        # The minimum rotation speed in radians per second
        self.min_angular_speed = rospy.get_param("~min_angular_speed", 0.0)
        
        # The max linear speed in meters per second
        self.max_linear_speed = rospy.get_param("~max_linear_speed", 0.3)
        
        # The minimum linear speed in meters per second
        self.min_linear_speed = rospy.get_param("~min_linear_speed", 0.1)
        
        # Slow down factor when stopping
        self.slow_down_factor = rospy.get_param("~slow_down_factor", 0.8)
```

上面主要是类中的一些限制参数的初始化，下面是一些主要的函数部分

```        
        # 首先初始化一个twist结构，用于发布命令
        self.move_cmd = Twist()

        # 定义pub函数，在cmd_vel发布twist的消息
        self.cmd_vel_pub = rospy.Publisher('cmd_vel', Twist, queue_size=5)

        # 定义一个sub程序，接收来自point_cloud的数据，然后调用set_cmd_vel进行处理。因为点云数据较大，所以队列数据设置为1，时间太长没处理的会被丢弃。
        self.depth_subscriber = rospy.Subscriber('point_cloud', PointCloud2, self.set_cmd_vel, queue_size=1)

        rospy.loginfo("Subscribing to point cloud...")
        #logininfo 是显示信息的函数
        # wait for message是等待来自point的cloud2数据，检测到之后进行下一步
        rospy.wait_for_message('point_cloud', PointCloud2)
	# 初始化完成
        rospy.loginfo("Ready to follow!")
```

## 2.2函数初始化

这个脚本最重要的函数就是这个函数，就是最简单的一个sub转pub的节点。

```
    def set_cmd_vel(self, msg):
        # 初始化重心节点的计数
        x = y = z = n = 0

        # msg是cloud2的数据，这个数据是检测到的深度点的坐标。
	# read_points 是读取深度点，skip_nans设置为true的话就会自动将距离是NaN的数据点返回
        for point in point_cloud2.read_points(msg, skip_nans=True):
            pt_x = point[0]
            pt_y = point[1]
            pt_z = point[2]
            
            # 总结在框选范围内的点，并总计坐标数据。不太明白为啥y会有区别
            if -pt_y > self.min_y and -pt_y < self.max_y and pt_x < self.max_x and pt_x > self.min_x and pt_z < self.max_z:
                x += pt_x
                y += pt_y
                z += pt_z
                n += 1
        
       # 如果n不是0，就找到中心点位置
        if n:
            x /= n 
            y /= n 
            z /= n
            
            # 看一下中心点是在哪，有没有超过需要移动的阈值，超过了就计算一下速度。
            if (abs(z - self.goal_z) > self.z_threshold):
                # Compute the angular component of the movement
                linear_speed = (z - self.goal_z) * self.z_scale
                #如果在0.6内，设置为负，往后退。
                # Make sure we meet our min/max specifications
                self.move_cmd.linear.x = copysign(max(self.min_linear_speed, 
                                        min(self.max_linear_speed, abs(linear_speed))), linear_speed)
            else:  #else就慢慢停下来，让现在的速度乘一个参数
                self.move_cmd.linear.x *= self.slow_down_factor
            # 设置转角    
            if (abs(x) > self.x_threshold):     
                # Compute the linear component of the movement
                angular_speed = -x * self.x_scale
                
                # Make sure we meet our min/max specifications
                self.move_cmd.angular.z = copysign(max(self.min_angular_speed, 
                                        min(self.max_angular_speed, abs(angular_speed))), angular_speed)
            else:
                # 慢慢停下来
                self.move_cmd.angular.z *= self.slow_down_factor
                
        else:
            # 没有检测到也要慢慢停下来
            self.move_cmd.linear.x *= self.slow_down_factor
            self.move_cmd.angular.z *= self.slow_down_factor
            
        # 发布命令，让机器人动起来
        self.cmd_vel_pub.publish(self.move_cmd)
```

## 2.3 停止follower节点

```
    def shutdown(self):
        rospy.loginfo("Stopping the robot...")
        
        # Unregister the subscriber to stop cmd_vel publishing
        self.depth_subscriber.unregister()
        rospy.sleep(1)
        
        # Send an emtpy Twist message to stop the robot
        self.cmd_vel_pub.publish(Twist())
        rospy.sleep(1)    
```

# 3 main函数

```
                   
if __name__ == '__main__':
    try:
        Follower()
        rospy.spin()
    except rospy.ROSInterruptException:
        rospy.loginfo("Follower node terminated.")
```




