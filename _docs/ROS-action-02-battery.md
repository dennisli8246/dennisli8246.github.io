---
title: "ROS-action-02-battery"
layout: collection
permalink: /ROS-action-02-battery/
excerpt: "How to build a mechanical arm."
last_modified_at: 2024-04-11T14:50:05-04:00
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

## actionlib 範例

情境設定：機器人會消耗電力，當機器人接上電源後會回饋充電狀態

1. 建立Action資料夾
2. 建立goal,result and feedback檔案
3. 修改套件檔案
4. 定義用戶端：機器人電量節點
5. 定義伺服器端：機器人電池狀態節點

## 建立套件與Action資料夾

1. 在工作空間中chapter_4_ws/src使用指令建立battery_simulator套件
    
    ```tsx
    catkin_create_pkg battery_simulator actionlib std_msgs
    ```
    
    ![Untitled](/assets/images/ROS-action-02-battery%20264b85cc1b6a446b82c97f1232f53ce6/Untitled.png)
    
2. 在~/chapter4_ws/src/battery_simulator底下建立actiony資料夾
    
    ```tsx
    mkdir action
    ```
    
3. 在action資料夾底下建立battery_sim.action的動作檔
    
    ```tsx
    # This is an action definition file, which has three parts: the goal, the
    # result, and the feedback.
    #
    # Part 1: the goal, to be sent by the client
    #
    # The amount of time we want to wait
    bool charge_state
    
    # Part 2: the result, to be sent by the server upon completion
    #
    # How much time we waited
    string battery_status
    
    # Part 3: the feedback, to be sent periodically by the server during
    # execution.
    #
    # The amount of time that has elapsed from the start
    float32 battery_percentage
    ```
    

## 修改套件與編輯套件

1. 修改package.xml新增呼叫套件
    
    ```tsx
    <build_depend>actionlib_msgs</build_depend>
    <exec_depend>actionlib_msgs</exec_depend>
    ```
    
2. 修改CMakeLists.txt，增加generate_messages()呼叫加入動作檔
    
    ```tsx
    ## Generate actions in the 'action' folder
     add_action_files(
        DIRECTORY action
        FILES battery_sim.action Timer.action
     )
    ```
    
3. 修改CMakeLists.txt，增加generate_messages()呼叫套件
    
    ```tsx
    ## Generate added messages and services with any dependencies listed here
     generate_messages(
       DEPENDENCIES
       actionlib_msgs std_msgs
     )
    ```
    
4. 修改CMakeLists.txt，增加catkin_package()呼叫相依套件套件
    
    ```tsx
    catkin_package(
      CATKIN_DEPENDS actionlib_msgs std_msgs
    )
    ```
    
5. 在~/chapter4_ws/編譯工作空間
    
    ```tsx
    catkin_make
    ```
    

## 定義伺服器

在~/chapter_4_ws/src/battery_simulator/建立**battery_sim_server.py**

```python
#! /usr/bin/env python

import time
import rospy
from multiprocessing import Process
from std_msgs.msg import Int32, Bool
import actionlib
from battery_simulator.msg import battery_simAction, battery_simGoal, battery_simResult, battery_simFeedback

def batterySim():
	battery_level = 100
	result = battery_simResult()
	while not rospy.is_shutdown():
		if rospy.has_param("/MyRobot/BatteryStatus"):
			time.sleep(1/10)
			param = rospy.get_param("/MyRobot/BatteryStatus")
			if param == 1:
				if battery_level == 100:
					result.battery_status = "Full"
					server.set_succeeded(result)
					break
				else:								
					battery_level += 1
					rospy.loginfo("Charging...currently, %s", battery_level)
					time.sleep(4/10)
			elif param == 0:
				if battery_level == 0:
					result.battery_status = "exhausted"
					server.set_succeeded(result)
					break
				else:
					battery_level -= 1
					rospy.logwarn("Discharging...currently, %s", battery_level)
					time.sleep(1/10)

def goalFun(goal):
	rate = rospy.Rate(2)
	process = Process(target = batterySim)
	process.start()
	time.sleep(1/10)
	if goal.charge_state == 0:
		rospy.set_param("/MyRobot/BatteryStatus",goal.charge_state)
	elif goal.charge_state == 1:
		rospy.set_param("/MyRobot/BatteryStatus",goal.charge_state)

if __name__ == '__main__':	
	rospy.init_node('BatterySimServer')
	server = actionlib.SimpleActionServer('battery_simulator', battery_simAction, goalFun, False)
	server.start()	
	rospy.spin()

```

## 定義用戶端

在~/chapter_4_ws/src/battery_simulator/建立**battery_sim_client.py**

```python
#! /usr/bin/env python

import sys
import rospy
from std_msgs.msg import Int32, Bool
import actionlib
from battery_simulator.msg import battery_simAction, battery_simGoal, battery_simResult

def battery_state(charge_condition):
	goal = battery_simGoal()
	goal.charge_state = charge_condition
	client.send_goal(goal)

if __name__ == '__main__':
	rospy.init_node('BatterySimclient')
	client = actionlib.SimpleActionClient('battery_simulator', battery_simAction)
	client.wait_for_server()
	battery_state(int(sys.argv[]))
	client.wait_for_result()

```

記得使用chmod +x 修改權限

![Untitled](/assets/images/ROS-action-02-battery%20264b85cc1b6a446b82c97f1232f53ce6/Untitled%201.png)

## 測試

```tsx
roscore
```

```tsx
initros1
source devel/setup.bash
rosrun battery_simulator battery_sim_server.py
```

開啟client並且設定param初始直為0:可以看到原本電池為100％開始下降

```tsx
initros1
source devel/setup.bash
rosrun battery_simulator battery_sim_client.py 0
```

如果在次輸入參數為1則可以充電

```python
rosrun battery_simulator battery_sim_client.py 1
```

[Screencast from 2024年四月11日 08時58分18秒.webm](/assets/images/ROS-action-02-battery%20264b85cc1b6a446b82c97f1232f53ce6/Screencast_from_2024%25E5%25B9%25B4%25E5%259B%259B%25E6%259C%258811%25E6%2597%25A5_08%25E6%2599%258258%25E5%2588%258618%25E7%25A7%2592.webm)

📃Reference:

[https://github.com/PacktPublishing/ROS-Robotics-Projects-SecondEdition/tree/master/chapter_4_ws/src](https://github.com/PacktPublishing/ROS-Robotics-Projects-SecondEdition/tree/master/chapter_4_ws/src)
