---
title: "ROS建制機械手臂-3（手臂模型）"
layout: collection
permalink: /ROS-Mechanical-Arm-03-model/
excerpt: "How to build a mechanical arm."
last_modified_at: 2024-03-27T08:48:05-04:00
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
# ROS-Mechanical-Arm-03-model

#ROS建制機械手臂（手臂模型）

延續ROS建制機械手臂（定義碰撞與制動）[https://www.notion.so/ROS-3c4f28844b474c4eaa4e85a51c915b63](https://www.notion.so/ROS-Mechanical-Arm-02-collision-actuator-937e8636d4f44ac7937ff956ab533f76?pvs=21)

[https://automaticaddison.com/how-to-build-a-simulated-robot-arm-using-ros/](https://automaticaddison.com/how-to-build-a-simulated-robot-arm-using-ros/)

1. 初始化工作空間,在~/chapter3_ws/src/robot_description/urdf建立檔案**robot_arm.urdf.xacro**
輸入架構如下：
    
    ```python
    <?xml version="1.0"?>
    
    <robot xmlns:xacro="http://ros.org/wiki/xacro" name="robot_arm" >
    
    </robot>
    ```
    
2. 定義連結,在<robot>標籤下建立機器人關節
    
    ```python
      <link name="arm_base">
        <visual>
          <origin
            xyz="0 0 0"
            rpy="0 0 0" />
          <geometry>
            <mesh filename="package://robot_description/meshes/arm_base.stl" />
          </geometry>
          <material
            name="">
            <color
              rgba="0.79216 0.81961 0.93333 1" />
          </material>
        </visual>
    </link>
    ```
    
3. 將其餘連結複製到<robot>標籤下,參考如下：
    
    ```python
    https://github.com/PacktPublishing/ROS-Robotics-Projects-SecondEdition/blob/master/chapter_3_ws/src/robot_description/urdf/robot_arm.urdf.xacro
    ```
    
4. 定義關節：建立檔案**robot_arm_essentials.xacro**參考如下
    
    ```python
    https://github.com/PacktPublishing/ROS-Robotics-Projects-SecondEdition/blob/master/chapter_3_ws/src/robot_description/urdf/robot_arm_essentials.xacro
    ```
    
5. 另外開啟一個terminal在rviz中測試所定義的機械手臂
    
    ```python
    initros1
    ```
    
    在~/chapter3_ws/資料夾下
    
    ```python
    source devel/setup.bash
    roscd robot_description/urdf/
    roslaunch urdf_tutorial display.launch model:=robot_arm.urdf.xacro
    ```
    
6. 在global options選項將fixed frame選為arm_base

![Untitled](/assets/images/ROS-Mechanical-Arm-03-model/Untitled.png)

1. 目前本節測試所使用的兩個檔案（有增加軸向變數）
    1. 在robot_arm_essentials.xacro中加入參數axis,並且讓 軸可以導入參數<axis xyz="＄{axis}"/>
    2. 在robot_arm.urdf.xacro增加axis="x y z”軸向參數（x y z 依照不同軸有不同）
        
        ![Untitled](/assets/images/ROS-Mechanical-Arm-03-model/Untitled01.png)
        
    3. robot_arm_essentials.xacro
    
    ```python
    <?xml version="1.0"?>
    <robot xmlns:xacro="http://ros.org/wiki/xacro" name="robot_essentials" >
    <xacro:macro name="arm_joint" params="prefix parent child originxyz originrpy">
    
    <joint name="${prefix}_joint" type="continuous">
      <axis xyz="${axis}"/>
      <parent link="${parent}"/>
      <child link="${child}"/>
      <origin rpy="${originrpy}" xyz="${originxyz}"/>
    </joint>
    
    </xacro:macro>
    </robot>
    
    ```
    
    b. robot_arm.urdf.xacro
    
    ```python
    <?xml version="1.0"?>
    
    <robot xmlns:xacro="http://ros.org/wiki/xacro" name="robot_arm" >
    <xacro:include filename="$(find robot_description)/urdf/robot_arm_essentials.xacro" />
    <xacro:include filename="$(find robot_description)/urdf/gazebo_essentials_base.xacro" />
    
    <link name="world"/>
    
      <link name="arm_base">
        <inertial>
          <origin
            xyz="7.7128E-09 -0.063005 -3.01969999961422E-08"
            rpy="0 0 0" />
          <mass
            value="1.6004" />
          <inertia
            ixx="0.00552196561445819"
            ixy="7.9550614501301E-10"
            ixz="-1.34378458924839E-09"
            iyy="0.00352397447953875"
            iyz="-1.10071809773382E-08"
            izz="0.00553739792746489" />
        </inertial>
        <visual>
          <origin
            xyz="0 0 0"
            rpy="0 0 0" />
          <geometry>
            <mesh filename="package://robot_description/meshes/arm_base.stl" />
          </geometry>
          <material
            name="">
            <color
              rgba="0.79216 0.81961 0.93333 1" />
          </material>
        </visual>
        <collision>
          <origin
            xyz="0 0 0"
            rpy="0 0 0" />
          <geometry>
            <mesh filename="package://robot_description/meshes/arm_base.stl" />
          </geometry>
        </collision>
    </link>
    
    <link
        name="bicep">
        <inertial>
          <origin
            xyz="0.12821 3.5589E-06 0.052492"
            rpy="0 0 0" />
          <mass
            value="1.1198" />
          <inertia
            ixx="0.0012474"
            ixy="-5.4004E-07"
            ixz="-0.0013148"
            iyy="0.0072923"
            iyz="-1.8586E-07"
            izz="0.0068178" />
        </inertial>
        <visual>
          <origin
            xyz="0 0 0"
            rpy="0 -1.5708 0" />
          <geometry>
            <mesh filename="package://robot_description/meshes/bicep.stl" />
          </geometry>
          <material
            name="">
            <color
              rgba="0.79216 0.81961 0.93333 1" />
          </material>
        </visual>
        <collision>
          <origin
            xyz="0 0 0"
            rpy="0 -1.5708 0" />
          <geometry>
            <mesh filename="package://robot_description/meshes/bicep.stl" />
          </geometry>
        </collision>
      </link>
    
      <link
        name="bottom_wrist">
        <inertial>
          <origin
            xyz="-9.1053E-08 -0.069257 -1.86629999995759E-07"
            rpy="0 0 0" />
          <mass
            value="0.27721" />
          <inertia
            ixx="0.00104290750143942"
            ixy="4.37155302268076E-09"
            ixz="-2.45049603914627E-09"
            iyy="0.000380518373895034"
            iyz="-7.56009835172156E-09"
            izz="0.00106006525067445" />
        </inertial>
        <visual>
          <origin
            xyz="0 0 0.13522"
            rpy="3.14 0 1.5708" />
          <geometry>
            <mesh filename="package://robot_description/meshes/wrist.stl" />
          </geometry>
          <material
            name="">
            <color
              rgba="0.79216 0.81961 0.93333 1" />
          </material>
        </visual>
        <collision>
          <origin
            xyz="0 0 0.13522"
            rpy="3.14 0 1.5708" />
          <geometry>
            <mesh filename="package://robot_description/meshes/wrist.stl" />
          </geometry>
        </collision>
      </link>
    
     <link
        name="elbow">
        <inertial>
          <origin
            xyz="-0.11109 1.1476E-08 0.046469"
            rpy="0 0 0" />
          <mass
            value="0.84845" />
          <inertia
            ixx="0.00079656"
            ixy="-7.8011E-10"
            ixz="0.00053616"
            iyy="0.003576"
            iyz="4.6326E-10"
            izz="0.0033698" />
        </inertial>
        <visual>
          <origin
            xyz="0.0 0.05163 0.20994"
            rpy="0 -1.5708 0" />
          <geometry>
            <mesh filename="package://robot_description/meshes/elbow.stl" />
          </geometry>
          <material
            name="">
            <color
              rgba="0.79216 0.81961 0.93333 1" />
          </material>
        </visual>
        <collision>
          <origin
            xyz="0.0 0.05163 0.20994"
            rpy="0 -1.5708 0" />
          <geometry>
            <mesh filename="package://robot_description/meshes/elbow.stl" />
          </geometry>
        </collision>
      </link>
    
    <link
        name="top_wrist">
        <inertial>
          <origin
            xyz="-9.1053E-08 -0.069257 -1.86629999995759E-07"
            rpy="0 0 0" />
          <mass
            value="0.27721" />
          <inertia
            ixx="0.00104290750143942"
            ixy="4.37155302268076E-09"
            ixz="-2.45049603914627E-09"
            iyy="0.000380518373895034"
            iyz="-7.56009835172156E-09"
            izz="0.00106006525067445" />
        </inertial>
        <visual>
          <origin
            xyz="0 0 0.13522"
            rpy="3.14 0 1.5708" />
          <geometry>
            <mesh filename="package://robot_description/meshes/wrist.stl" />
          </geometry>
          <material
            name="">
            <color
              rgba="0.79216 0.81961 0.93333 1" />
          </material>
        </visual>
        <collision>
          <origin
            xyz="0 0 0.13522"
            rpy="3.14 0 1.5708" />
          <geometry>
            <mesh filename="package://robot_description/meshes/wrist.stl" />
          </geometry>
        </collision>
      </link>
    
    <xacro:arm_joint prefix="arm_base" parent="world" child="arm_base" originxyz="0.0 0.0 0.1" originrpy="0 0 0" axis="0 0 1"/>
    <xacro:arm_joint prefix="shoulder" parent="arm_base" child="bicep" originxyz="-0.05166 0.0 0.20271" originrpy="0 0 1.5708" axis="0 1 0"/>
    <xacro:arm_joint prefix="bottom_wrist" parent="bicep" child="bottom_wrist" originxyz="0.0 -0.05194 0.269" originrpy="0 0 0" axis="0 1 0"/>
    <xacro:arm_joint prefix="elbow" parent="bottom_wrist" child="elbow" originxyz="0.0 0 0.13522" originrpy="0 0 0" axis="0 0 1" />
    <xacro:arm_joint prefix="top_wrist" parent="elbow" child="top_wrist" originxyz="0.0 0 0.20994" originrpy="0 0 0" axis="0 1 0" />
    
    <xacro:arm_transmission prefix="arm_base"/>
    <xacro:arm_transmission prefix="shoulder"/>
    <xacro:arm_transmission prefix="bottom_wrist"/>
    <xacro:arm_transmission prefix="elbow"/>
    <xacro:arm_transmission prefix="top_wrist"/>
    
    </robot>
    
    ```
    
2. 呈現效果
    
    [Screencast from 2024年三月06日 17時28分06秒.webm](/assets/images/ROS-Mechanical-Arm-03-model/Screencast_from_2024.webm)
    
3. 確認主題
    
    ```python
    rostopic list
    ```
    
    ![Untitled](/assets/images/ROS-Mechanical-Arm-03-model/Untitled02.png)
