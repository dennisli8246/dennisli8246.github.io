---
title: "ROS建立導航系統（LiDAR）"
layout: collection
permalink: /ROS-Navigation-02-LiDAR/
excerpt: "How to build a mechanical arm."
last_modified_at: 2024-03-29T13:19:00-00:00
classes: wide
categories:
  - ROS
  - ROS建制機械機械手臂
  - Navigation
tags:
  - ROS
  - 機械手臂
  - mechanical arm
  - Gazebo
  - Navigation
redirect_from:
  - /theme-setup/
sidebar:
  nav: "docs"
---
# ROS-Navigation-02-LiDAR

## ＃ROS建立導航系統（LiDAR）

由建立導向系統的專案中

1. 將之前ROS建制機械手臂的檔案mobile_manipulator.urdf.xacro增加感測器模型
    1. 其中<link name="laser_link">中為一個虛擬方塊表示為雷射
    2. <gazebo reference="laser_link">為gazebo掃描的行為狀態
    3. <min_angle>-1.570796</min_angle><max_angle>1.570796</max_angle>為掃描角度
    4. <range>設定範圍以及解析度
    
    ```tsx
    <!--########################## SENSORS ############################################-->
    
    <link name="laser_link">
      <collision>
       <origin xyz="0 0 0" rpy="0 0 0"/>
       <geometry>
        <box size="0.1 0.1 0.1"/>
       </geometry>
      </collision>
      <visual>
       <origin xyz="0 0 0" rpy="0 0 0"/>
        <geometry>
         <box size="0.05 0.05 0.05"/>
        </geometry>
      </visual>
      <inertial>
        <mass value="1e-5" />
        <origin xyz="0 0 0" rpy="0 0 0"/>
        <inertia ixx="1e-6" ixy="0" ixz="0" iyy="1e-6" iyz="0" izz="1e-6" />
      </inertial>
    </link>
    
    <joint name="laser_joint" type="fixed">
    <axis xyz="0 1 0" />
    <origin xyz="0.350 0 0.115" rpy="0 0 0"/>
    <parent link="base_link"/>
    <child link="laser_link"/>
    </joint>
    
    <gazebo reference="laser_link">
    	<sensor type="ray" name="laser">
    		<pose>0 0 0 0 0 0</pose>
    		<visualize>true</visualize>
    		<update_rate>40</update_rate>
    		<ray>
    			<scan>
    				<horizontal>
    				<samples>720</samples>
    				<resolution>1</resolution>
    				<min_angle>-1.570796</min_angle>
    				<max_angle>1.570796</max_angle>
    				</horizontal>
    			</scan>
    			<range>
    				<min>0.10</min>
    				<max>30.0</max>
    				<resolution>0.01</resolution>
    			</range>
    		</ray>
    		<plugin name="gazebo_ros_head_hokuyo_controller" filename="libgazebo_ros_laser.so">
    		  <topicName>/scan</topicName>
    		  <frameName>laser_link</frameName>
    		</plugin>
    	</sensor>
    </gazebo>
    
    <!--###############################################################################-->
    ```
    
2. 設定導航堆疊的控制器（YAML檔）,來建立move_base節點可以參考[http://wiki.ros.org/move_base](http://wiki.ros.org/move_base)
    
    ![Untitled](ROS-Navigation-02-LiDAR%20ffd4e84fec7c4245b2396db6b5f9be06/Untitled.png)
    
3. 建立move_base的服務器以下四個檔案在/chapter5_ws/src/robot_description/config資料夾中
    1. **base_local_planner_params.yaml**
        
        ```tsx
        TrajectoryPlannerROS:
         holonomic_robot: false
        ```
        
    2. **costmap_common_params.yaml**
        
        定義機器人的行走極限（外觀為矩形）
        
        定義使用的雷射掃描器並且提供資訊
        
        ```tsx
        footprint: [[0.70, 0.65], [0.70, -0.65], [-0.70, -0.65], [-0.75, 0.65]]
        observation_sources: laser_scan_sensor
        laser_scan_sensor:
         sensor_frame: laser_link
         data_type: LaserScan
         topic: scan
         marking: true
         clearing: true
        ```
        
    3. **global_costmap_params.yaml**
        
        機器人（base_linkl）相對於世界移動（world）地圖來逕行移動
        
        ```tsx
        global_costmap:
         global_frame: map
         robot_base_frame: base_link
         static_map: true
        ```
        
    4. **local_costmap_params.yaml**
        
        局部地圖的資訊
        
        ```tsx
        local_costmap:
         global_frame: odom
         robot_base_frame: base_link
         rolling_window: true
        ```
        
4. 建構環境地圖使用gmapping以及karto的地圖建構技術,我們將使用gmapping
    
    ```tsx
    sudo apt-get install ros-noetic-slam-gmapping
    ```
    
    ![Untitled](ROS-Navigation-02-LiDAR%20ffd4e84fec7c4245b2396db6b5f9be06/Untitled%201.png)
    
5. 定義機器人基座
    1. 使用ROS的自主導航套件amcl[https://wiki.ros.org/amcl](https://wiki.ros.org/amcl)
    2. amcl會根據所建構的圖形列出可能的位置清單最後收斂出機器人的可能位置
    3. 在/chapter5_ws/src/robot_description/launch建立兩個檔案
        1. **amcl_diff.launch**
            
            ```tsx
            <launch>
            <node pkg="amcl" type="amcl" name="amcl" output="screen">
              <!-- Publish scans from best pose at a max of 10 Hz -->
              <param name="odom_model_type" value="diff"/>
              <param name="odom_alpha5" value="0.1"/>
              <param name="transform_tolerance" value="0.2" />
              <param name="gui_publish_rate" value="10.0"/>
              <param name="laser_max_beams" value="30"/>
              <param name="min_particles" value="500"/>
              <param name="max_particles" value="5000"/>
              <param name="kld_err" value="0.05"/>
              <param name="kld_z" value="0.99"/>
              <param name="odom_alpha1" value="0.2"/>
              <param name="odom_alpha2" value="0.2"/>
              <!-- translation std dev, m -->
              <param name="odom_alpha3" value="0.8"/>
              <param name="odom_alpha4" value="0.2"/>
              <param name="laser_z_hit" value="0.5"/>
              <param name="laser_z_short" value="0.05"/>
              <param name="laser_z_max" value="0.05"/>
              <param name="laser_z_rand" value="0.5"/>
              <param name="laser_sigma_hit" value="0.2"/>
              <param name="laser_lambda_short" value="0.1"/>
              <param name="laser_lambda_short" value="0.1"/>
              <param name="laser_model_type" value="likelihood_field"/>
              <!-- <param name="laser_model_type" value="beam"/> -->
              <param name="laser_likelihood_max_dist" value="2.0"/>
              <param name="update_min_d" value="0.2"/>
              <param name="update_min_a" value="0.5"/>
              <param name="odom_frame_id" value="odom"/>
              <param name="resample_interval" value="1"/>
              <param name="transform_tolerance" value="0.1"/>
              <param name="recovery_alpha_slow" value="0.0"/>
              <param name="recovery_alpha_fast" value="0.0"/>
            </node>
            </launch>
            ```
            
        2. **mobile_manipulator_move_base.launch**
            
            ```tsx
            <launch>
            
            <node name="map_server" pkg="map_server" type="map_server" args="/home/ros/ros/chapter5_ws/postoffice.yaml"/>
            
            <include file="$(find robot_description)/launch/amcl_diff.launch"/>
            
            <node pkg="move_base" type="move_base" respawn="false" name="move_base" output="screen">
             <rosparam file="$(find robot_description)/config/costmap_common_params.yaml" command="load" ns="global_costmap" />
             <rosparam file="$(find robot_description)/config/costmap_common_params.yaml" command="load" ns="local_costmap" />
             <rosparam file="$(find robot_description)/config/local_costmap_params.yaml" command="load" />
             <rosparam file="$(find robot_description)/config/global_costmap_params.yaml" command="load" />
             <rosparam file="$(find robot_description)/config/base_local_planner_params.yaml" command="load" />
             <remap from="cmd_vel" to="/robot_base_velocity_controller/cmd_vel"/>
            </node>
            </launch>
            ```
            

📃Reference:

[https://www.notion.so/ROS-ffd4e84fec7c4245b2396db6b5f9be06](https://www.notion.so/ROS-Navigation-02-LiDAR-ffd4e84fec7c4245b2396db6b5f9be06?pvs=21)
