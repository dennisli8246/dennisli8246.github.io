# ROS-Navigation-01-environment

## #ROS建立導航系統（環境建立）

[https://docs.google.com/document/d/1Thvde7KwDTyKo7bCRBlS7gnxIGwnMcVbOtedHwimGz4/edit](https://docs.google.com/document/d/1Thvde7KwDTyKo7bCRBlS7gnxIGwnMcVbOtedHwimGz4/edit)

1. 建立work space
    
    先建立initros1的環境
    
    ```tsx
    initros1
    ```
    
    建立工作空件
    
    ```tsx
    mkdir -p chapter5_ws/src
    catkin_init_workspace
    ```
    
    將之前所建立的移動機械手臂放到這個工作空間chapter5_ws下
    
    [https://www.notion.so/ROS-base-arm-e829de5fd3a24f0c96adfc290c92b2ce](https://www.notion.so/ROS-Mechanical-Arm-05-combine-base-arm-e829de5fd3a24f0c96adfc290c92b2ce?pvs=21)
    
    ```tsx
    cp -r ~/chapter3_ws/src/robot_description ~/chapter5_ws/src/
    ```
    
    ```tsx
    catkin_make
    ```
    
2. 在~/chapter5_ws/src/robot_description/底下建立worlds的資料夾
    
    ```python
    mkdir worlds
    ```
    
3. 並且在此資料夾下建立環境模型檔案[postoffice.world](https://docs.google.com/document/d/1Thvde7KwDTyKo7bCRBlS7gnxIGwnMcVbOtedHwimGz4/edit)
    
    使用gazebo測試所建立的環境(要有耐心因為要開很久)
    
    ```tsx
    gazebo postoffice.world
    ```
    
    ![Untitled](ROS-Navigation-01-environment%20f5c6372221ba4eba94895976a00c90f3/Untitled.png)
    
4. 修改/chapter5_ws/src/robot_description/launch/中的檔案mobile_manipulator_gazebo_control_xacro.launch
    
    ```python
    <?xml version="1.0"?>
    <launch>
    
    <param name="robot_description" command="$(find xacro)/xacro --inorder $(find robot_description)/urdf/mobile_manipulator.urdf.xacro" />
      <include file="$(find gazebo_ros)/launch/empty_world.launch">
    	  
      <arg name="world_name" value="/home/ros/ros/chapter5_ws/src/robot_description/world/postoffice.world"/>
      
      </include>
       <!-- Spawn the robot model robot position-->
      <node name="spawn_urdf" pkg="gazebo_ros" type="spawn_model" args="-param robot_description -urdf -x 0 -y 0 -z 1 -model mobile_manipulator" />
      
      <!-- <node name="spawn_urdf" pkg="gazebo_ros" type="spawn_model" args="-param robot_description -urdf -model robot_arm" />-->
      
      <rosparam command="load" file="$(find robot_description)/config/arm_control.yaml" />
      <node name="arm_controller_spawner" pkg="controller_manager" type="controller_manager" args="spawn arm_controller" respawn="false" output="screen"/>
      <rosparam command="load" file="$(find robot_description)/config/joint_state_controller.yaml" />
      <node name="joint_state_controller_spawner" pkg="controller_manager" type="controller_manager" args="spawn joint_state_controller" respawn="false" output="screen"/>
      <rosparam command="load" file="$(find robot_description)/config/control.yaml" />
      <node name="base_controller_spawner" pkg="controller_manager" type="spawner" args="robot_base_joint_publisher robot_base_velocity_controller"/>
      <node name="robot_state_publisher" pkg="robot_state_publisher" type="robot_state_publisher" respawn="false" output="screen"/>
     
      
    </launch>
    
    ```
    
5. 其中要將postoffice.world的路徑檔案放到world_name標籤下
    
    ```jsx
    /home/ros/ros/navigation_ws/worlds/postoffice.world
    ```
    
    ```python
    <arg name="world_name" value="/home/ros/ros/chapter5_ws/src/robot_description/world/postoffice.world"/>
    ```
    
    spawn_urdf標籤下設定機器人的（x,y,z）初始位置
    
    ```python
    <node name="spawn_urdf" pkg="gazebo_ros" type="spawn_model" args="-param robot_description -urdf -x 0 -y 0 -z 1 -model mobile_manipulator" />
    ```
    
6. 運作檔案測試
    
    ```tsx
    initros1
    source devel/setup.bash
    roslaunch robot_description mobile_manipulator_gazebo_control_xacro.launch
    ```
    
    ![Untitled](ROS-Navigation-01-environment%20f5c6372221ba4eba94895976a00c90f3/Untitled%201.png)
    

📃Reference: 

[https://automaticaddison.com/setting-up-the-ros-navigation-stack-for-a-simulated-robot/](https://automaticaddison.com/setting-up-the-ros-navigation-stack-for-a-simulated-robot/)

[https://automaticaddison.com/how-to-create-a-new-package-in-ros-noetic/](https://automaticaddison.com/how-to-create-a-new-package-in-ros-noetic/)

[https://github.com/PacktPublishing/ROS-Robotics-Projects-SecondEdition/tree/master/chapter_5_ws/src](https://github.com/PacktPublishing/ROS-Robotics-Projects-SecondEdition/tree/master/chapter_5_ws/src)