# ROS-Mechanical-Arm-04-arm-actuator

#ROS建制機械手臂（手臂模擬與制動）

1. 在robot_arm.urdf.xacro各個<link>標籤中建立質量與碰撞,以下程式碼為單一link舉例

```python
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

---------------------
    <collision>
      <origin
        xyz="0 0 0"
        rpy="1.5707963267949 0 3.14" />
      <geometry>
        <mesh filename="package://robot_description/meshes/robot_base.stl" />
      </geometry>
    </collision>
```

1. 在robot_arm_essentials.xacro定義制動器
    
    ```python
    <!--######################################################################-->
    
    <xacro:macro name="arm_transmission" params="prefix ">
    
    <transmission name="${prefix}_trans" type="SimpleTransmission">
      <type>transmission_interface/SimpleTransmission</type>
      <actuator name="${prefix}_motor">
       <hardwareInterface>hardware_interface/PositionJointInterface</hardwareInterface>
       <mechanicalReduction>1</mechanicalReduction>
      </actuator>
      <joint name="${prefix}_joint">
       <hardwareInterface>hardware_interface/PositionJointInterface</hardwareInterface>
      </joint>
    </transmission>
    
    </xacro:macro>
    <!--######################################################################-->
    ```
    
2. 在robot_arm.urdf.xacro呼叫制動器

```python
<xacro:arm_transmission prefix="arm_base"/>
<xacro:arm_transmission prefix="shoulder"/>
<xacro:arm_transmission prefix="bottom_wrist"/>
<xacro:arm_transmission prefix="elbow"/>
<xacro:arm_transmission prefix="top_wrist"/>
```

1. 定義ROS_CONTROLLER: 在gazebo_essentials_base.xacro加入一個joint_state_publisher控制器檔

```python
  <gazebo>
    <plugin name="joint_state_publisher" filename="libgazebo_ros_joint_state_publisher.so">
      <jointName>arm_base_joint, shoulder_joint, bottom_wrist_joint, elbow_joint, bottom_wrist_joint</jointName>
    </plugin>
  </gazebo>
```

1. robot_arm.urdf.xacro中引入巨集標籤

```python
<xacro:include filename="$(find robot_description)/urdf/robot_arm_essentials.xacro" />
<xacro:include filename="$(find robot_description)/urdf/gazebo_essentials_base.xacro" />
```

1. 建立機械手臂控制器: 在~/chapter3_ws/src/robot_description/config下建立arm_control.yaml

```python
arm_controller:
    type: position_controllers/JointTrajectoryController
    joints:
       - arm_base_joint
       - shoulder_joint
       - bottom_wrist_joint
       - elbow_joint
       - top_wrist_joint
    constraints:
      goal_time: 0.6
      stopped_velocity_tolerance: 0.05
      hip: {trajectory: 0.1, goal: 0.1}
      shoulder: {trajectory: 0.1, goal: 0.1}
      elbow: {trajectory: 0.1, goal: 0.1}
      wrist: {trajectory: 0.1, goal: 0.1}
    stop_trajectory_duration: 0.5
    state_publish_rate:  25
    action_monitor_rate: 10
# Publish all joint states -----------------------------------
joint_state_controller:
    type: joint_state_controller/JointStateController
    publish_rate: 50

/gazebo_ros_control:
    pid_gains:
      arm_base_joint: {p: 1000.0, i: 100.0, d: 10.0}
      shoulder_joint: {p: 1000.0, i: 100.0, d: 10.0}
      bottom_wrist_joint: {p: 1000.0, i: 100.0, d: 10.0}
      elbow_joint: {p: 1000.0, i: 100.0, d: 10.0}
      top_wrist_joint: {p: 100.0, i: 0.0, d: 0.0}

```

1. 在/chapter3_ws/src/robot_description/config建立joint_state_controller.yaml
    
    ```python
      # Publish all joint states -----------------------------------
      joint_state_controller:
        type: joint_state_controller/JointStateController
        publish_rate: 50
    
    ```
    
2. 測試機械手臂： 在~/chapter3_ws/src/robot_description/launch建立啟動檔**arm_gazebo_control_xacro.launch**

```python
<?xml version="1.0"?>
<launch>

<param name="robot_description" command="$(find xacro)/xacro --inorder $(find robot_description)/urdf/robot_arm.urdf.xacro" />
  <include file="$(find gazebo_ros)/launch/empty_world.launch"/>
  <node name="spawn_urdf" pkg="gazebo_ros" type="spawn_model" args="-param robot_description -urdf -model robot_arm" />
  <rosparam command="load" file="$(find robot_description)/config/arm_control.yaml" />
  <node name="arm_controller_spawner" pkg="controller_manager" type="controller_manager" args="spawn arm_controller" respawn="false" output="screen"/>
  <rosparam command="load" file="$(find robot_description)/config/joint_state_controller.yaml" />
  <node name="joint_state_controller_spawner" pkg="controller_manager" type="controller_manager" args="spawn joint_state_controller" respawn="false" output="screen"/>
  <node name="robot_state_publisher" pkg="robot_state_publisher" type="robot_state_publisher" respawn="false" output="screen"/>
  
</launch>
```

<aside>
💡 **arm_control.yaml**檔案中的controller會被**arm_gazebo_control_xacro.launch的標籤尋找例如**<node name="arm_controller_spawner" pkg="controller_manager" type="controller_manager" args="spawn arm_controller" respawn="false" output="screen"/> 中的arm_controller就會去尋找**arm_control.yaml中的**arm_controller

</aside>

1. 執行視覺化: 在~/chapter3_ws/

```python
initros1
source devel/setup.bash
roslaunch robot_description arm_gazebo_control_xacro.launch
```

![Untitled](/assets/images/ROS-Mechanical-Arm-04-arm-actuator/Untitled.png)

1. 查看topic
    
    ```python
    initros1
    rostopic list
    ```
    
2. 輸入指令移動機械手臂
    
    ```python
    rostopic pub /arm_controller/command trajectory_msgs/JointTrajectory '{joint_names: ["arm_base_joint","shoulder_joint", "bottom_wrist_joint", "elbow_joint","top_wrist_joint"], points: [{positions: [-0.1, 0.5, 0.02, 0, 0], time_from_start: [1,0]}]}' -1
    ```
    
    ```python
    rostopic pub /arm_controller/command trajectory_msgs/JointTrajectory '{joint_names: ["arm_base_joint","shoulder_joint", "bottom_wrist_joint", "elbow_joint","top_wrist_joint"], points: [{positions: [0, 0, 0, 0, 0], time_from_start: [1,0]}]}' -1
    ```
    
    [Screencast from 2024年三月07日 15時06分52秒.webm](/assets/images/ROS-Mechanical-Arm-04-arm-actuator/Screencast_from_2024.webm)
    

reference:

[https://answers.ros.org/question/360732/can-you-provide-an-example-of-publishing-commands-using-the-arm_controllercommand-topic-in-the-terminal/](https://answers.ros.org/question/360732/can-you-provide-an-example-of-publishing-commands-using-the-arm_controllercommand-topic-in-the-terminal/)

[https://answers.ros.org/question/54962/understanding-trajectory_msgsjointtrajectory/](https://answers.ros.org/question/54962/understanding-trajectory_msgsjointtrajectory/)

<aside>
💡 Could not load controller 'arm_controller' because controller type 'position_controllers/JointTrajectoryController' does not exist.

![Untitled](/assets/images/ROS-Mechanical-Arm-04-arm-actuator/Untitled01.png)

[https://answers.ros.org/question/254084/gazebo-could-not-load-controller-jointtrajectorycontroller-does-not-exist-mastering-ros-chapter-10/](https://answers.ros.org/question/254084/gazebo-could-not-load-controller-jointtrajectorycontroller-does-not-exist-mastering-ros-chapter-10/)

確認版本

```python
rosversion -d
```

安裝ros controller相關套件

```python
sudo apt-get install ros-noetic-joint-trajectory-controller
```

```python
sudo apt-get install ros*controller*
```

[ERROR] [1709563170.207053491, 0.310000000]: Could not load controller 'joint_state_controller' because the type was not specified. Did you load the controller configuration on the parameter server (namespace: '/joint_state_controller')?

![Untitled](/assets/images/ROS-Mechanical-Arm-04-arm-actuator/Untitled02.png)

```python
sudo apt-get install ros-noetic-joint-state-controller ＃This will install joint_state_controller package
sudo apt-get install ros-noetic-effort-controllers ＃This will install Effort controller
sudo apt-get install ros-noetic-position-controllers ＃ This will install position controllers
```

在~/ros/chapter3_ws/src/robot_description/config/arm_control.yaml中加入以下程式碼

```python
  # Publish all joint states -----------------------------------
  joint_state_controller:
    type: joint_state_controller/JointStateController
    publish_rate: 50
```

查詢目前控制器載入狀態

1. 先lunch所要的程式
    1. 
    
    ![Untitled](/assets/images/ROS-Mechanical-Arm-04-arm-actuator/Untitled03.png)
    
2. 在另外一個terminal找尋
    
    ```python
    rosservice list | grep controller_manager
    ```
    
    ![Untitled](/assets/images/ROS-Mechanical-Arm-04-arm-actuator/Untitled04.png)
    

</aside>

reference:

模型建立參考[https://www.notion.so/ROS-09d84546634447509510d80fec247ba5](https://www.notion.so/ROS-Mechanical-Arm-03-model-09d84546634447509510d80fec247ba5?pvs=21)

[https://automaticaddison.com/how-to-build-a-simulated-robot-arm-using-ros/](https://automaticaddison.com/how-to-build-a-simulated-robot-arm-using-ros/)

[https://drive.google.com/drive/folders/1oX9Eyd1fKWX1HOQfMhK5aJoHKeJjznzM](https://drive.google.com/drive/folders/1UlkrsKflhNXCpGrL4QA26yL3ezgKfteg)

[https://drive.google.com/drive/folders/1Acv58Up41u5pYDM5yE1jru_YoVbn_rCl](https://drive.google.com/drive/folders/1Acv58Up41u5pYDM5yE1jru_YoVbn_rCl)
