---
title: "ROS建制機械手臂-02（定義碰撞與制動）"
layout: collection
permalink: /ROS-Mechanical-Arm-02-collision-actuator/
excerpt: "How to build a mechanical arm."
last_modified_at: 2024-03-28T17:10:05-04:00
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

# ROS-Mechanical-Arm-02-collision-actuator

## ＃ROS建制機械手臂（定義碰撞與制動）

延續ROS建制機械手臂（基座）

1. 定義碰撞base：在robot_base.urdf.xacro中<link>加入<collision>標籤
    
    ```python
        <collision>
          <origin
            xyz="0 0 0"
            rpy="1.5707963267949 0 3.14" />
          <geometry>
            <mesh filename="package://robot_description/meshes/robot_base.stl" />
          </geometry>
        </collision>
    ```
    
2. 定義碰撞wheel：在robot_essentials.xacro
    
    ```python
        <collision>
          <origin
            xyz="0 0 0"
            rpy="1.5707963267949 0 0" />
          <geometry>
            <mesh filename="package://robot_description/meshes/wheel.stl" />
          </geometry>
          <material
            name="">
            <color
              rgba="0.79216 0.81961 0.93333 1" />
          </material>
          </collision>
    ```
    
3. 定義質量與慣量
    1. 在robot_base.urdf.xacro中<link>標籤增加base質量與轉動慣量
        
        ```python
            <inertial>
              <origin
                xyz="0.0030946 4.78250032638821E-11 0.053305"
                rpy="0 0 0" />
              <mass
                value="47.873" />
              <inertia
                ixx="0.774276574699151"
                ixy="-1.03781944357671E-10"
                ixz="0.00763014265820928"
                iyy="1.64933255189991"
                iyz="1.09578155845563E-12"
                izz="2.1239326987473" />
            </inertial>
        ```
        
    2. 在robot_essentials.xacro中<link>標籤增加wheel質量與轉動慣量
        
        ```python
            <inertial>
              <origin
                xyz="-4.1867E-18 0.0068085 -1.65658661799998E-18"
                rpy="0 0 0" />
              <mass
                value="2.6578" />
              <inertia
                ixx="0.00856502765719703"
                ixy="1.5074118157338E-19"
                ixz="-4.78150098725052E-19"
                iyy="0.013670640432096"
                iyz="-2.68136447099727E-19"
                izz="0.00856502765719703" />
            </inertial>
        ```
        
4. 定義制動器
    1. 在robot_essentials.xacro檔案中加入制動器標籤
        
        ```python
        <xacro:macro name="base_transmission" params="prefix ">
        
        <transmission name="${prefix}_wheel_trans" type="SimpleTransmission">
          <type>transmission_interface/SimpleTransmission</type>
          <actuator name="${prefix}_wheel_motor">
           <hardwareInterface>hardware_interface/VelocityJointInterface</hardwareInterface>
           <mechanicalReduction>1</mechanicalReduction>
          </actuator>
          <joint name="${prefix}_wheel_joint">
           <hardwareInterface>hardware_interface/VelocityJointInterface</hardwareInterface>
          </joint>
        </transmission>
        
        </xacro:macro>
        ```
        
    2. 並且在robot_base.urdf.xacro將其作為巨集呼叫
        
        ```python
        <xacro:base_transmission prefix="front_left"/>
        <xacro:base_transmission prefix="front_right"/>
        <xacro:base_transmission prefix="rear_left"/>
        <xacro:base_transmission prefix="rear_right"/>
        ```
        
5. 定義ROS_CONTROLLERS
    1. 建立一個可以讓“Gazebo與ROS溝通的外掛
    2. 建立檔案gazebo_essentials_base.xacro
    3. 建立通訊外掛
        
        ```python
        <?xml version="1.0"?>
        <robot xmlns:xacro="http://ros.org/wiki/xacro" name="gazebo_essentials" >
        <gazebo>
            <plugin name="gazebo_ros_control" filename="libgazebo_ros_control.so">
              <robotNamespace>/</robotNamespace>
              <controlPeriod>0.001</controlPeriod>
              <legacyModeNS>false</legacyModeNS>
            </plugin>
          </gazebo>
        </xacro:macro>
        </robot>
        ```
        
    4. 建立差速器外掛
        
        ```python
        <gazebo>
              <plugin name="diff_drive_controller" filename="libgazebo_ros_diff_drive.so">
                <legacyMode>false</legacyMode>
                <alwaysOn>true</alwaysOn>
                <updateRate>1000.0</updateRate>
                <leftJoint>front_left_wheel_joint, rear_left_wheel_joint</leftJoint>
                <rightJoint>front_right_wheel_joint, rear_right_wheel_joint</rightJoint>
                <wheelSeparation>0.5</wheelSeparation>
                <wheelDiameter>0.2</wheelDiameter>
                <wheelTorque>10</wheelTorque>
                <publishTf>1</publishTf>
                <odometryFrame>map</odometryFrame>
                <commandTopic>cmd_vel</commandTopic>
                <odometryTopic>odom</odometryTopic>
                <robotBaseFrame>base_link</robotBaseFrame>
                <wheelAcceleration>2.8</wheelAcceleration>
                <publishWheelJointState>true</publishWheelJointState>
                <publishWheelTF>false</publishWheelTF>
                <odometrySource>world</odometrySource>
                <rosDebugLevel>Debug</rosDebugLevel>
              </plugin>
        </gazebo>
        ```
        
    5. 建立輪子的摩擦外掛
        
        ```python
        <xacro:macro name="wheel_friction" params="prefix ">
        
        <gazebo reference="${prefix}_wheel">
         <mu1 value="1.0"/>
         <mu2 value="1.0"/>
         <kp value="10000000.0" />
         <kd value="1.0" />
         <fdir1 value="1 0 0"/>
        </gazebo>
        
        </xacro:macro>
        ```
        
    6. 並且在robot_base.urdf.xacro將其作為巨集呼叫
        
        ```python
        <xacro:wheel_friction prefix="front_left"/>
        <xacro:wheel_friction prefix="front_right"/>
        <xacro:wheel_friction prefix="rear_left"/>
        <xacro:wheel_friction prefix="rear_right"/>
        ```
        
        記得在robot_base.urdf.xacro中<robot>引入
        
        ```python
        <xacro:include filename="$(find robot_description)/urdf/gazebo_essentials_base.xacro" />
        ```
        
6. 設定控制器,切換到config檔下
    
    ```python
    geany control.yaml
    ```
    
    將控制腳本貼上，可以參考作者的github
    
    [https://github.com/PacktPublishing/ROS-Robotics-Projects-SecondEdition/blob/master/chapter_3_ws/src/robot_description/config/control.yaml](https://github.com/PacktPublishing/ROS-Robotics-Projects-SecondEdition/blob/master/chapter_3_ws/src/robot_description/config/control.yaml)
    
    ```python
    robot_base_joint_publisher:
      type: "joint_state_controller/JointStateController"
      publish_rate: 50
    
    robot_base_velocity_controller:
      type: "diff_drive_controller/DiffDriveController"
      left_wheel: ['front_left_wheel_joint', 'rear_left_wheel_joint']
      right_wheel: ['front_right_wheel_joint', 'rear_right_wheel_joint']
      publish_rate: 50
      pose_covariance_diagonal: [0.001, 0.001, 0.001, 0.001, 0.001, 0.03]
      twist_covariance_diagonal: [0.001, 0.001, 0.001, 0.001, 0.001, 0.03]
      cmd_vel_timeout: 0.5
      wheel_separation : 0.445208
      wheel_radius : 0.0625
      base_frame_id: base_link
      odom_frame_id: odom
      enable_odom_tf: true
      estimate_velocity_from_position: false
      wheel_separation_multiplier: 1.0 
      wheel_radius_multiplier    : 1.0 
      linear:
        x:
          has_velocity_limits    : true
          max_velocity           : 2.0   
          has_acceleration_limits: true
          max_acceleration       : 3.0   
      angular:
        z:
          has_velocity_limits    : true
          max_velocity           : 3.0   
          has_acceleration_limits: true
          max_acceleration       : 6.0   
    
    /gazebo_ros_control:
        pid_gains:
          front_left_wheel_joint: {p: 20.0, i: 0.01, d: 0.0, i_clamp: 0.0}
          front_right_wheel_joint: {p: 20.0, i: 0.01, d: 0.0, i_clamp: 0.0}
          rear_left_wheel_joint: {p: 20.0, i: 0.01, d: 0.0, i_clamp: 0.0}
          rear_right_wheel_joint: {p: 20.0, i: 0.01, d: 0.0, i_clamp: 0.0}
    ```
    
7. 測試基座
    1. ~/chapter3_ws/src/robot_description/launch
        
        ```python
        geany base_gazebo_control_xacro.lunch
        ```
        
        將腳本貼上
        
        [https://github.com/PacktPublishing/ROS-Robotics-Projects-SecondEdition/blob/master/chapter_3_ws/src/robot_description/launch/base_gazebo_control.launch](https://github.com/PacktPublishing/ROS-Robotics-Projects-SecondEdition/blob/master/chapter_3_ws/src/robot_description/launch/base_gazebo_control_xacro.launch)
        
        ```python
        <?xml version="1.0"?>
        
        <launch>
          <param name="robot_description" command="$(find xacro)/xacro --inorder $(find robot_description)/urdf/robot_base.urdf.xacro" />
          <include file="$(find gazebo_ros)/launch/empty_world.launch"/>
          <node name="spawn_urdf" pkg="gazebo_ros" type="spawn_model" args="-param robot_description -urdf -model robot_base" />
          <rosparam command="load" file="$(find robot_description)/config/control.yaml" />
          <node name="base_controller_spawner" pkg="controller_manager" type="spawner" args="robot_base_joint_publisher robot_base_velocity_controller"/>
        
        </launch>
        ```
        
    2. 在~/chapter3_ws底下
        
        ```python
        source devel/setup.bash
        ```
        
        ```python
        roslaunch robot_description base_gazebo_control_xacro.lunch
        ```
        
        ![Untitled](/assets/images/ROS-Mechanical-Arm-02-collision-actuator%20937e8636d4f44ac7937ff956ab533f76/Untitled.png)
        
    3. 在另外一個terminal查看topic
        
        ```python
        initros1
        rostopic list
        ```
        
        ![Untitled](/assets/images/ROS-Mechanical-Arm-02-collision-actuator%20937e8636d4f44ac7937ff956ab533f76/Untitled%201.png)
        

💡注意：使用rviz視覺化查看視覺話錯誤問題

1. 新開的terminal使用rviz視覺化
    
    ```python
    initros1
    ```
    
2. cd ~/chapter3_ws
    
    ```python
    source devel/setup.bash
    roscd robot_description/urdf/
    roslaunch urdf_tutorial display.launch model:=robot_base.urdf.xacro
    ```
    
3. 錯誤原因為visual所以查看<virual>標籤下是否有問題,例如<virual>中是否有其他標籤造成衝突
    
    ![Untitled](/assets/images/ROS-Mechanical-Arm-02-collision-actuator%20937e8636d4f44ac7937ff956ab533f76/Untitled%202.png)
    
    修正後結果如下
    
    ![Untitled](/assets/images/ROS-Mechanical-Arm-02-collision-actuator%20937e8636d4f44ac7937ff956ab533f76/Untitled%203.png)
    
4. 啟動rqt_robot_steering讓機器人移動測試
    
    ```python
    rosrun rqt_robot_steering rqt_robot_steering
    ```
    

💡 注意：

rqt_robot_steering無法控制,要確認一下rqt_robot_steering中所控制的topic是否正確,應該要是/robot_base_controller/cmd_vel

![Untitled](/assets/images/ROS-Mechanical-Arm-02-collision-actuator%20937e8636d4f44ac7937ff956ab533f76/Untitled%204.png)

控制結果如下：

[Screencast from 2024年三月01日 14時20分23秒.webm](ROS-Mechanical-Arm-02-collision-actuator%20937e8636d4f44ac7937ff956ab533f76/Screencast_from_2024%25E5%25B9%25B4%25E4%25B8%2589%25E6%259C%258801%25E6%2597%25A5_14%25E6%2599%258220%25E5%2588%258623%25E7%25A7%2592.webm)
