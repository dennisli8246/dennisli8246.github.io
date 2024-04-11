---
title: "ROS-action-01-robot-arm"
layout: collection
permalink: /ROS-Mechanical-Arm-01-base/
excerpt: "How to build a mechanical arm."
last_modified_at: 2024-04-11T14:41:05-04:00
classes: wide
categories:
  - ROS
tags:
  - ROS
  - mechanical arm
  - Gazebo
redirect_from:
  - /theme-setup/
sidebar:
  nav: "docs"
---

# ROS-action-01-robot-arm

## 在章節ROS-Mechanical-Arm-05-combine-base-arm中啟動模型

觀察ROS的topic

1. 視覺化啟動測試~/chapter_3_ws
    
    ```python
    initros1
    source devel/setup.bash
    roslaunch robot_description mobile_manipulator_gazebo_control_xacro.launch
    ```
    
2. 在另外一個terminal~/chapter_3_ws下開啟topic
    
    ```tsx
    initros1
    source devel/setup.bash
    rostopic list
    ```
    
3. 觀察topic
    
    ![Untitled](/assets/images/ROS-action-01-robot-arm%20d011de546cf34117b896e469452331e4/Untitled.png)
    

可以看到/cancel,/goal,/result,/status,/feedback,這接就是ROS的action實作

其關係圖如下：

![notion-template.drawio.svg](/assets/imagesROS-action-01-robot-arm%20d011de546cf34117b896e469452331e4/notion-template.drawio.svg)

## 範例模擬

製作一個工作空間來觀察

```tsx
initros1
mkdir -p chapter4_ws/src
```

cd ~/chapter4_ws/

```tsx
catkin_init_workspace
catkin_create_pkg arm_client
```

cd ~/chapter4_ws/src/arm_client

在此資料夾下放入以下程式，需要改變權限chmod +x arm_action_client.py

**arm_action_client.py**

```tsx
#!/usr/bin/env python

import rospy
import actionlib
from std_msgs.msg import Float64
from trajectory_msgs.msg import JointTrajectoryPoint
from control_msgs.msg import FollowJointTrajectoryAction, FollowJointTrajectoryGoal

def move_joint(angles):
    goal = FollowJointTrajectoryGoal()
    goal.trajectory.joint_names = ['arm_base_joint', 'shoulder_joint','bottom_wrist_joint' ,'elbow_joint', 'top_wrist_joint']
    point = JointTrajectoryPoint()
    point.positions = angles
    point.time_from_start = rospy.Duration(3)
    goal.trajectory.points.append(point)
    client.send_goal_and_wait(goal)

if __name__ == '__main__':
    rospy.init_node('joint_position_tester')
    client = actionlib.SimpleActionClient('arm_controller/follow_joint_trajectory', FollowJointTrajectoryAction)
    client.wait_for_server()
    move_joint([-0.1, 0.210116830848170721, 0.022747275919015486, 0.0024182584123728645, 0.00012406874824844039])
```

1. 注意原本mobile_manipulator_gazebo_control_xacro.launch的節點
    
    ![Untitled](/assets/imagesROS-action-01-robot-arm%20d011de546cf34117b896e469452331e4/Untitled%201.png)
    
2. 宣告節點：rospy.init_node('joint_position_tester')
3. 定義Action的topic：client = actionlib.SimpleActionClient('arm_controller/follow_joint_trajectory', FollowJointTrajectoryAction)
4. 等待Action Client回覆：client.wait_for_server()
5. 執行子程式：move_joint(angles)
6. 設定Action中goal的message：goal = FollowJointTrajectoryGoal()
7. 設定Action的goal中的關節名稱：goal.trajectory.joint_names = ['arm_base_joint', 'shoulder_joint','bottom_wrist_joint' ,'elbow_joint', 'top_wrist_joint']
8. 設定Action中point的message：point = JointTrajectoryPoint()
9. 設定Action中point時間：point.time_from_start = rospy.Duration(3)
10. 將目標的位置塞入goal的訊號中：goal.trajectory.points.append(point)
11. 請Client端傳送goal的訊號後等待client.send_goal_and_wait(goal)

💡先把chapter4_ws底下的CMakeLists.txt刪除

![Untitled](/assets/imagesROS-action-01-robot-arm%20d011de546cf34117b896e469452331e4/Untitled%202.png)

在使用catkin_make編譯工作空間

```tsx
catkin_make
```

一個terminal開啟chapter3_ws的機些手臂

```tsx
initros1
source devel/setup.bash
roslaunch robot_description mobile_manipulator_gazebo_control_xacro.launch
```

另外一個開啟chapter4_ws

```tsx
initros1
source devel/setup.bash
rosrun arm_client arm_action_client.py
```

💡note:如果沒有給予檔案權限可能造成問題

## 顯示效果

[Screencast from 2024年四月09日 18時30分41秒.webm](/assets/imagesROS-action-01-robot-arm%20d011de546cf34117b896e469452331e4/Screencast_from_2024%25E5%25B9%25B4%25E5%259B%259B%25E6%259C%258809%25E6%2597%25A5_18%25E6%2599%258230%25E5%2588%258641%25E7%25A7%2592.webm)

📃Reference:

[https://github.com/PacktPublishing/ROS-Robotics-Projects-SecondEdition/tree/master/chapter_4_ws/src](https://github.com/PacktPublishing/ROS-Robotics-Projects-SecondEdition/tree/master/chapter_4_ws/src)
