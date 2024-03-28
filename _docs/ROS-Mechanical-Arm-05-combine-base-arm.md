---
title: "ROS建制機械手臂-5（組合base＆arm）"
layout: collection
permalink: /ROS-Mechanical-Arm-05-combine-base-arm/
excerpt: "How to build a mechanical arm."
last_modified_at: 2024-03-28T18:14:05-04:00
classes: wide
categories:
  - ROS
  - ROS建制機械機械手臂
tags:
  - ROS
  - 機械手臂
  - mechanical arm
  - Gazebo
redirect_from:
  - /theme-setup/
sidebar:
  nav: "docs"
---
# ROS-Mechanical-Arm-05-combine-base-arm

#ROS建制機械手臂（組合base＆arm）

如上兩節建立base與arm

1. 將兩個模做連接,在~/chapter3_ws/src/robot_description/urdf 建立**mobile_manipulator.urdf.xacro**

```python
<?xml version="1.0"?>

<robot xmlns:xacro="http://ros.org/wiki/xacro" name="robot_base" >
<xacro:include filename="$(find robot_description)/urdf/robot_base.urdf.xacro" />
<xacro:include filename="$(find robot_description)/urdf/robot_arm.urdf.xacro" />
<xacro:arm_joint prefix="arm_base_link" parent="base_link" child="arm_base" originxyz="0.20 0.0 0.145" originrpy="0 0 0" axis="0 0 1"/>
</robot>
```

<aside>
💡 注意行會呼叫arm_joint,會需要有axis的參數
<xacro:arm_joint prefix="arm_base_link" parent="base_link" child="arm_base" originxyz="0.20 0.0 0.145" originrpy="0 0 0" axis="0 0 1"/>

</aside>

1. 使用以下指令查看目前的組裝後模型,在~/chapter3_ws/底下

```python
initros1
source devel/setup.bash
roscd robot_description/urdf/
roslaunch urdf_tutorial display.launch model:=**mobile_manipulator.urdf.xacro**
```

<aside>
💡 產生問題：

Failed to find root link: Two root links found: [base_link] and [world]

![Untitled](/assets/images/ROS-Mechanical-Arm-05-combine-base-arm/Untitled.png)

將arm_base修改為world測試

```python
<xacro:arm_joint prefix="arm_base_link" parent="base_link" child="arm_base" originxyz="0.20 0.0 0.145" originrpy="0 0 0"/>
```

可以看到的確可以組成但是在world與arm_base中有一個沒有銜接的空間,原因是在arm的模組中我們之前有加入一個world為link

![Untitled](/assets/images/ROS-Mechanical-Arm-05-combine-base-arm/Untitled01.png)

1. 兩個模型結構如下：
    
    arm_module:
    
    world(root_link)→arm_base(joint)→arm_base(link)→shoulder(link)→bicep(joint)→bottom_wrist(link)→bottom_wrist(joint)→elbow(link)→top_wrist(joint)
    
    base_module:
    
    base_link(root_link)
    
    →front_left(joint)→front_left(link)
    
    →front_right(joint)→front_right(link)
    
    →rear_left(joint)→rear_left(link)
    
    →rear_right(joint)→rear_right(link)
    
2. 結合模型結構如下(會造成Two root links found)：
    
    world(root_link)→arm_base_link(joint)→…….
    
    base_link(root_link)
    
    →arm_base_link(joint)→arm_base(joint)→arm_base(link)→………→top_wrist(joint)
    
    →front_left(joint)→front_left(link)
    
    →front_right(joint)→front_right(link)
    
    →rear_left(joint)→rear_left(link)
    
    →rear_right(joint)→rear_right(link)
    

c. 用world連結合模型結構如下：

base_link(root_link)

→arm_base_link(joint)→arm_base(joint)→arm_base(link)→..→top_wrist(joint)

→front_left(joint)→front_left(link)

→front_right(joint)→front_right(link)

→rear_left(joint)→rear_left(link)

→rear_right(joint)→rear_right(link)

</aside>

將結構調整如下：

base_link(root_link)

→**arm_base_link**(joint)→arm_base(joint)→arm_base(link)→..→top_wrist(joint)

→front_left(joint)→front_left(link)

→front_right(joint)→front_right(link)

→rear_left(joint)→rear_left(link)

→rear_right(joint)→rear_right(link)

只需要將robot_arm.urdf.xacro中的world的link跟相關的joint註解掉

```python
<!--#####<link name="world"/>###########-->
<!--##<xacro:arm_joint prefix="arm_base" parent="world" child="arm_base" originxyz="0.0 0.0 0.1" originrpy="0 0 0"/>###-->
```

並且要在**mobile_manipulator.urdf.xacro的arm_base_link**(joint)改為**arm_base**(joint)**因為我們原本是用arm_base(joint)接world,但先在更改為接上base_link**

結果如下：

![Untitled](/assets/images/ROS-Mechanical-Arm-05-combine-base-arm/Untitled02.png)

[Screencast from 2024年三月07日 20時45分39秒.webm](/assets/images/ROS-Mechanical-Arm-05-combine-base-arm/Screencast_from_2024.webm)

1. 測試機械手臂,~chapter3_ws/src/robot_description/launch建立起動檔案mobile_manipulator_gazebo_control_xacro.launch
    1. [https://drive.google.com/drive/folders/1Acv58Up41u5pYDM5yE1jru_YoVbn_rCl](https://drive.google.com/drive/folders/1Acv58Up41u5pYDM5yE1jru_YoVbn_rCl)
    
    ```python
    <?xml version="1.0"?>
    <launch>
    
    <param name="robot_description" command="$(find xacro)/xacro --inorder $(find robot_description)/urdf/mobile_manipulator.urdf.xacro" />
      <include file="$(find gazebo_ros)/launch/empty_world.launch">
    	<!--arg name="world_name" value="/home/robot/custom_worlds/postoffice.world"/-->
      </include>
      <node name="spawn_urdf" pkg="gazebo_ros" type="spawn_model" args="-param robot_description -urdf -model mobile_manipulator" />
      <rosparam command="load" file="$(find robot_description)/config/arm_control.yaml" />
      <node name="arm_controller_spawner" pkg="controller_manager" type="controller_manager" args="spawn arm_controller" respawn="false" output="screen"/>
      <rosparam command="load" file="$(find robot_description)/config/joint_state_controller.yaml" />
      <node name="joint_state_controller_spawner" pkg="controller_manager" type="controller_manager" args="spawn joint_state_controller" respawn="false" output="screen"/>
      <rosparam command="load" file="$(find robot_description)/config/control.yaml" />
      <node name="base_controller_spawner" pkg="controller_manager" type="spawner" args="robot_base_joint_publisher robot_base_velocity_controller"/>
      <node name="robot_state_publisher" pkg="robot_state_publisher" type="robot_state_publisher" respawn="false" output="screen"/>
    
    </launch>
    ```
    
2. 視覺化啟動測試~/chapter_3_ws
    
    ```python
    initros1
    source devel/setup.bash
    roslaunch robot_description mobile_manipulator_gazebo_control_xacro.launch
    ```
    
    <aside>
    💡 Gazebo無法組裝
    
    ![Untitled](/assets/images/ROS-Mechanical-Arm-05-combine-base-arm/Untitled03.png)
    
    </aside>
    
    <aside>
    💡 ERROR
    
    ![Untitled](/assets/images/ROS-Mechanical-Arm-05-combine-base-arm/Untitled04.png)
    
    因為node有重複啟動的可能性
    
    ![Untitled](/assets/images/ROS-Mechanical-Arm-05-combine-base-arm/Untitled05.png)
    
    衝突的可能性為
    
    robot_arm.urdf.xacro中下這行被重複引入可以註解掉
    
    ```python
    <!--#<xacro:include filename="$(find robot_description)/urdf/gazebo_essentials_base.xacro" />#-->
    ```
    
    ![Untitled](/assets/images/ROS-Mechanical-Arm-05-combine-base-arm/Untitled06.png)
    
    </aside>
    
3. 運行結果
    
    ![Untitled](/assets/images/ROS-Mechanical-Arm-05-combine-base-arm/Untitled07.png)
    
4. 列出所有topic
    
    ```python
    rostopic list
    ```
    
    ![Untitled](/assets/images/ROS-Mechanical-Arm-05-combine-base-arm/Untitled08.png)
    
5. 指令移動測試
    
    ```python
    rostopic pub /arm_controller/command trajectory_msgs/JointTrajectory '{joint_names: ["arm_base_joint","shoulder_joint", "bottom_wrist_joint", "elbow_joint","top_wrist_joint"], points: [{positions: [-0.1, 0.5, 0.02, 0, 0], time_from_start: [1,0]}]}' -1
    ```
    
    ```python
    rostopic pub /arm_controller/command trajectory_msgs/JointTrajectory '{joint_names: ["arm_base_joint","shoulder_joint", "bottom_wrist_joint", "elbow_joint","top_wrist_joint"], points: [{positions: [0, 0, 0, 0, 0], time_from_start: [1,0]}]}' -1
    ```
    
    [Screencast from 2024年三月11日 17時46分49秒.webm](/assets/images/ROS-Mechanical-Arm-05-combine-base-arm/Screencast_from_2024_1.webm)
    
6. 在另外一個視窗開啟控制程式
    
    記得先做source
    
    ```python
    initros1
    source devel/setup.bash
    ```
    
    ```python
    rosrun rqt_robot_steering rqt_robot_steering
    ```
    
    ![Untitled](/assets/images/ROS-Mechanical-Arm-05-combine-base-arm/Untitled%09.png)
    
    記得要改變控制的topic在GUI界面
    `/robot_base_velocity_controller/cmd_vel`
    
7. 可以使用另外一個是視窗觀察robot目前座標
    
    ```python
    rostopic echo /robot_base_velocity_controller/cmd_vel
    ```
    
8. 全部效果如下
    
    [Screencast from 2024年三月11日 17時57分34秒.webm](/assets/images/ROS-Mechanical-Arm-05-combine-base-arm/Screencast_from_2024_2.webm)
    

reference:

參考：[https://automaticaddison.com/how-to-build-a-simulated-mobile-manipulator-using-ros/](https://automaticaddison.com/how-to-build-a-simulated-mobile-manipulator-using-ros/)
