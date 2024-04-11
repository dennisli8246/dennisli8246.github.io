---
title: "ROS-SMACH"
layout: collection
permalink: /ROS-SMACH/
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
# ROS-SMACH

安裝SMACH-ROS

```python
sudo apt-get install ros-noetic-smach ros-noetic-smach ros-noetic-executive-smach ros-noetic-smach-viewer
```

## 簡單範例

四個狀態機

![notion-template.drawio.svg](/assets/images/ROS-SMACH%20369a7bc88035427ca29d6c02acd579a4/notion-template.drawio.svg)

在~/chapter4_ws/src底下建立smach_example資料夾以及建立**simple_fsm.py**

```python
#!/usr/bin/env python
import rospy
from smach import State,StateMachine
from time import sleep
import smach_ros

class A(State):
  def __init__(self):
    State.__init__(self, outcomes=['1','0'], input_keys=['input'], output_keys=[''])
  
  def execute(self, userdata):
    sleep(1)
    if userdata.input == 1:
      return '1'
    else:
      return '0'

class B(State):
  def __init__(self):
    State.__init__(self, outcomes=['1','0'], input_keys=['input'], output_keys=[''])
  def execute(self, userdata):
    sleep(1)
    if userdata.input == 1:
      return '1'
    else:
      return '0'

class C(State):
  def __init__(self):
    State.__init__(self, outcomes=['1','0'], input_keys=['input'], output_keys=[''])
  def execute(self, userdata):
    sleep(1)
    if userdata.input == 1:
      return '1'
    else:
      return '0'

class D(State):
  def __init__(self):
    State.__init__(self, outcomes=['1','0'], input_keys=['input'], output_keys=[''])
  def execute(self, userdata):
    sleep(1)
    if userdata.input == 1:
      return '1'
    else:
      return '0'

if __name__ == '__main__':

  rospy.init_node('test_fsm', anonymous=True)
  sm = StateMachine(outcomes=['success'])
  sm.userdata.sm_input = 1
  with sm:
    StateMachine.add('A', A(), transitions={'1':'B','0':'D'}, remapping={'input':'sm_input','output':'input'})
    StateMachine.add('B', B(), transitions={'1':'C','0':'A'}, remapping={'input':'sm_input','output':'input'})
    StateMachine.add('C', C(), transitions={'1':'D','0':'B'}, remapping={'input':'sm_input','output':'input'})
    StateMachine.add('D', D(), transitions={'1':'A','0':'C'}, remapping={'input':'sm_input','output':'input'})
  sis = smach_ros.IntrospectionServer('server_name', sm, '/SM_ROOT')
  sis.start()

  sm.execute()
  rospy.spin()
  sis.stop()
```

記得要開啟**simple_fsm.py**權限

```python
chomd +x **simple_fsm.py**
```

在~/chapter4_ws/src底下建立**CMakeLists.txt：**[CMakeLists.txt](https://github.com/PacktPublishing/ROS-Robotics-Projects-SecondEdition/blob/master/chapter_4_ws/src/smach_example/CMakeLists.txt)

在~/chapter4_ws/src底下建立**package.xml：**[package.xml](https://github.com/PacktPublishing/ROS-Robotics-Projects-SecondEdition/blob/master/chapter_4_ws/src/smach_example/CMakeLists.txt)

在~/chapter4_ws/做編譯動作

```python
catkin_make
```

在一個terminal開啟

```python
initros1
roscore
```

另外開啟terminal

```python
initros1
source devel/setup.bash
rosrun smach_example **simple_fsm**.py
```

第三個terminal

```python
initros1
source devel/setup.bash
rosrun smach_viewer smach_viewer.py
```

## 結果

[Screencast from 2024年四月11日 14時25分57秒.webm](/assets/images/ROS-SMACH%20369a7bc88035427ca29d6c02acd579a4/Screencast_from_2024%25E5%25B9%25B4%25E5%259B%259B%25E6%259C%258811%25E6%2597%25A5_14%25E6%2599%258225%25E5%2588%258657%25E7%25A7%2592.webm)

📃Reference:

[https://github.com/PacktPublishing/ROS-Robotics-Projects-SecondEdition/tree/master/chapter_4_ws/src](https://github.com/PacktPublishing/ROS-Robotics-Projects-SecondEdition/tree/master/chapter_4_ws/src)
