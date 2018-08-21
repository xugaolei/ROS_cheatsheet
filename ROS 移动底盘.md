# 带ROS接口的移动底盘的各个组成部分职责

- Arduino 负责获取编码器等传感器信息,驱动电机等执行器,运行电机的PID调速算法.
- 底盘控制器 Base Controller 通过串口与Arduino通讯,向Arduino发送串口指令,Arduino根据指令执行响应的动作,并通过串口返回数据给 Base Controller.
- 底盘控制器 Base Controller 是运行在电脑上的节点,负责向 /odom 话题发布里程计数据,并且订阅 /cmd_vel 话题.
- 一般来说 Base Controller 节点还会发布 /odom 坐标系到 /base_link 或 /base_footprint 坐标系的变换矩阵.不过不总是这样,例如: TurtleBot 采用另外一个节点 robot_pose_ekf 来发布/odom 坐标系到 /base_link 或 /base_footprint 坐标系的坐标系变换, 该节点利用 IMU 来优化里程计数据.
frame—either /base_link or /base_footprint .

这样就完成了一个 ROS 移动底盘,该**ROS底盘要满足3点:** 接受速度信息 /cmd_vel ,发布里程计数据 /odom,发布里程计坐标系到基坐标系的变换.

接下来就可以利用**move_base**功能包集对小车进行路径规划和避障.

![move_base 功能包集](http://wiki.ros.org/move_base?action=AttachFile&do=get&target=overview_tf_small.png)

下一步 SLAM:
- gmapping

下一步基于语义的任务:
- 比如: 去厨房给我拿瓶雪碧.
- smach , behavior trees 等工具.

# 总结
