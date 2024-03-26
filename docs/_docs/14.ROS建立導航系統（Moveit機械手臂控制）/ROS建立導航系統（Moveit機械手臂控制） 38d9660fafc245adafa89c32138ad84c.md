# ROS建立導航系統（Moveit機械手臂控制）

reference: 

[https://automaticaddison.com/how-to-set-up-and-control-a-robotic-arm-using-moveit-and-ros/](https://automaticaddison.com/how-to-set-up-and-control-a-robotic-arm-using-moveit-and-ros/)

[https://moveit.ros.org/robots/](https://moveit.ros.org/robots/)

1. 安裝moveit
    
    ```tsx
    sudo apt install ros-noetic-moveit
    sudo apt-get install ros-noetic-moveit-setup-assistant
    sudo apt-get install ros-noetic-moveit-simple-controller-manager
    sudo apt-get install ros-noetic-moveit-fake-controller-manager
    ```
    
2. 使用moveit設定精靈設定
    1. 建立ros環境
    2. 開啟一個新視窗並且在
    3. 開啟moveit的設定精靈
    
    ```tsx
    initros1
    source devel/setup.bash
    roslaunch moveit_setup_assistant setup_assistant.launch
    ```
    
    ![Untitled](ROS%E5%BB%BA%E7%AB%8B%E5%B0%8E%E8%88%AA%E7%B3%BB%E7%B5%B1%EF%BC%88Moveit%E6%A9%9F%E6%A2%B0%E6%89%8B%E8%87%82%E6%8E%A7%E5%88%B6%EF%BC%89%2038d9660fafc245adafa89c32138ad84c/Untitled.png)
    
3. 點選**Create New MoveIt Configuration Package**並且導入機器人模型
    
    ![Untitled](ROS%E5%BB%BA%E7%AB%8B%E5%B0%8E%E8%88%AA%E7%B3%BB%E7%B5%B1%EF%BC%88Moveit%E6%A9%9F%E6%A2%B0%E6%89%8B%E8%87%82%E6%8E%A7%E5%88%B6%EF%BC%89%2038d9660fafc245adafa89c32138ad84c/Untitled%201.png)
    
    按下Load Files
    
    ![Untitled](ROS%E5%BB%BA%E7%AB%8B%E5%B0%8E%E8%88%AA%E7%B3%BB%E7%B5%B1%EF%BC%88Moveit%E6%A9%9F%E6%A2%B0%E6%89%8B%E8%87%82%E6%8E%A7%E5%88%B6%EF%BC%89%2038d9660fafc245adafa89c32138ad84c/Untitled%202.png)
    
4. 設定自碰撞
    1. Self-Collisions選項
    2. 點選Generate Collision Matrix
        
        ![Untitled](ROS%E5%BB%BA%E7%AB%8B%E5%B0%8E%E8%88%AA%E7%B3%BB%E7%B5%B1%EF%BC%88Moveit%E6%A9%9F%E6%A2%B0%E6%89%8B%E8%87%82%E6%8E%A7%E5%88%B6%EF%BC%89%2038d9660fafc245adafa89c32138ad84c/Untitled%203.png)
        
    3. 可以調整sampling density設定這樣可以讓機器人在越狹窄空間移動,但可能造成軌跡規劃時間變長
    4. 在Planning Groups使用Add Group來加入機器人手臂群組
    5. 將Group Name取名為arm
    6. 將Kinematic Solver選擇kdl_kinematics_plugin/KDLKinematicsPlugin.
    7. Planner選擇RRTstar
        
        ![Untitled](ROS%E5%BB%BA%E7%AB%8B%E5%B0%8E%E8%88%AA%E7%B3%BB%E7%B5%B1%EF%BC%88Moveit%E6%A9%9F%E6%A2%B0%E6%89%8B%E8%87%82%E6%8E%A7%E5%88%B6%EF%BC%89%2038d9660fafc245adafa89c32138ad84c/Untitled%204.png)
        
    8. 點選Add Joints
        
        ![Untitled](ROS%E5%BB%BA%E7%AB%8B%E5%B0%8E%E8%88%AA%E7%B3%BB%E7%B5%B1%EF%BC%88Moveit%E6%A9%9F%E6%A2%B0%E6%89%8B%E8%87%82%E6%8E%A7%E5%88%B6%EF%BC%89%2038d9660fafc245adafa89c32138ad84c/Untitled%205.png)
        
        1. 將機器人的關節加入到Joint Names中
            
            ![Untitled](ROS%E5%BB%BA%E7%AB%8B%E5%B0%8E%E8%88%AA%E7%B3%BB%E7%B5%B1%EF%BC%88Moveit%E6%A9%9F%E6%A2%B0%E6%89%8B%E8%87%82%E6%8E%A7%E5%88%B6%EF%BC%89%2038d9660fafc245adafa89c32138ad84c/Untitled%206.png)
            
        2. 按下Save
            
            ![Untitled](ROS%E5%BB%BA%E7%AB%8B%E5%B0%8E%E8%88%AA%E7%B3%BB%E7%B5%B1%EF%BC%88Moveit%E6%A9%9F%E6%A2%B0%E6%89%8B%E8%87%82%E6%8E%A7%E5%88%B6%EF%BC%89%2038d9660fafc245adafa89c32138ad84c/Untitled%207.png)
            
5. 設定手臂姿勢點選Robot Poses > Add Pose
    
    ![Untitled](ROS%E5%BB%BA%E7%AB%8B%E5%B0%8E%E8%88%AA%E7%B3%BB%E7%B5%B1%EF%BC%88Moveit%E6%A9%9F%E6%A2%B0%E6%89%8B%E8%87%82%E6%8E%A7%E5%88%B6%EF%BC%89%2038d9660fafc245adafa89c32138ad84c/Untitled%208.png)
    
    1. 加入Posename: Straight並且將個關節設定如下並且按下save
        - arm_base_joint: 0.0
        - shoulder_joint: 0.0
        - bottom_wrist_joint: 0.0
        - elbow_joint: 0.0
        - top_wrist_joint: 0.0
    2. 在加入一個按下Add Pose加入Posename: Home並且將個關節設定如下並且按下save
        - arm_base_joint: 1.5794
        - shoulder_joint: 0.7116
        - bottom_wrist_joint: 1.9960
        - elbow_joint: 0.0
        - top_wrist_joint: 1.9660
        
        ![Untitled](ROS%E5%BB%BA%E7%AB%8B%E5%B0%8E%E8%88%AA%E7%B3%BB%E7%B5%B1%EF%BC%88Moveit%E6%A9%9F%E6%A2%B0%E6%89%8B%E8%87%82%E6%8E%A7%E5%88%B6%EF%BC%89%2038d9660fafc245adafa89c32138ad84c/Untitled%209.png)
        
6. 設置被動關節
    
    ![Untitled](ROS%E5%BB%BA%E7%AB%8B%E5%B0%8E%E8%88%AA%E7%B3%BB%E7%B5%B1%EF%BC%88Moveit%E6%A9%9F%E6%A2%B0%E6%89%8B%E8%87%82%E6%8E%A7%E5%88%B6%EF%BC%89%2038d9660fafc245adafa89c32138ad84c/Untitled%2010.png)
    
    1. 設置ROS控制器：將連接機器人與Moveit
        
        點選**Auto Add FollowJointsTrajectory Controllers For Each Planning Group**
        
        可以看到控制自動匯入
        
        並且將Controller Type改為**FollowJointsTrajectory**
        
        ![Untitled](ROS%E5%BB%BA%E7%AB%8B%E5%B0%8E%E8%88%AA%E7%B3%BB%E7%B5%B1%EF%BC%88Moveit%E6%A9%9F%E6%A2%B0%E6%89%8B%E8%87%82%E6%8E%A7%E5%88%B6%EF%BC%89%2038d9660fafc245adafa89c32138ad84c/Untitled%2011.png)
        
    2. 確認**Author Information**
        1. 可以輸入名字與mail
            
            ![Untitled](ROS%E5%BB%BA%E7%AB%8B%E5%B0%8E%E8%88%AA%E7%B3%BB%E7%B5%B1%EF%BC%88Moveit%E6%A9%9F%E6%A2%B0%E6%89%8B%E8%87%82%E6%8E%A7%E5%88%B6%EF%BC%89%2038d9660fafc245adafa89c32138ad84c/Untitled%2012.png)
            
        2. 注意一定要填寫不然下個步驟不能走
            
            ![Untitled](ROS%E5%BB%BA%E7%AB%8B%E5%B0%8E%E8%88%AA%E7%B3%BB%E7%B5%B1%EF%BC%88Moveit%E6%A9%9F%E6%A2%B0%E6%89%8B%E8%87%82%E6%8E%A7%E5%88%B6%EF%BC%89%2038d9660fafc245adafa89c32138ad84c/Untitled%2013.png)
            
    3. 設定Moveitconfig
        1. 點選 Configuration Files
        2. 輸入產生想放置控制器的位置以及檔名
        
        ```tsx
        /home/ros/ros/chapter5_ws/src/robot_description_moveit_config
        ```
        
        c. 按下Generate Package
        
        ![Untitled](ROS%E5%BB%BA%E7%AB%8B%E5%B0%8E%E8%88%AA%E7%B3%BB%E7%B5%B1%EF%BC%88Moveit%E6%A9%9F%E6%A2%B0%E6%89%8B%E8%87%82%E6%8E%A7%E5%88%B6%EF%BC%89%2038d9660fafc245adafa89c32138ad84c/Untitled%2014.png)
        
        d. 離開設定精靈
        
    4. 使用Moveit控制機器人手臂
        1. 在一個新terminal中打開模型檔案,在/chapter5_ws底下
        
        ```tsx
        initros1
        source devel/setup.bash
        roslaunch robot_description mobile_manipulator_gazebo_control_xacro.launch 
        ```
        
        b. 在另外一個terminal中打開moveit控制器,在/chapter5_ws底下
        
        ```tsx
        initros1
        source devel/setup.bash
        roslaunch robot_description_moveit_config move_group.launch
        ```
        
        c. 在另外一個terminal中用RViz來控制,在/chapter5_ws底下
        
        ```tsx
        initros1
        source devel/setup.bash
        roslaunch robot_description_moveit_config moveit_rviz.launch config:=True
        ```
        
        d. 左下角按下Add並且選擇MotionPlanning
        
        ![Untitled](ROS%E5%BB%BA%E7%AB%8B%E5%B0%8E%E8%88%AA%E7%B3%BB%E7%B5%B1%EF%BC%88Moveit%E6%A9%9F%E6%A2%B0%E6%89%8B%E8%87%82%E6%8E%A7%E5%88%B6%EF%BC%89%2038d9660fafc245adafa89c32138ad84c/Untitled%2015.png)
        
        e. 在Planning標籤下的Goal State選Home並且按下Plan就可以看到寫手臂在RViz中作動
        
        ![Untitled](ROS%E5%BB%BA%E7%AB%8B%E5%B0%8E%E8%88%AA%E7%B3%BB%E7%B5%B1%EF%BC%88Moveit%E6%A9%9F%E6%A2%B0%E6%89%8B%E8%87%82%E6%8E%A7%E5%88%B6%EF%BC%89%2038d9660fafc245adafa89c32138ad84c/Untitled%2016.png)
        
        f. 按下Execute就可以看到機寫手臂在Gazebo中作動
        
        ![Untitled](ROS%E5%BB%BA%E7%AB%8B%E5%B0%8E%E8%88%AA%E7%B3%BB%E7%B5%B1%EF%BC%88Moveit%E6%A9%9F%E6%A2%B0%E6%89%8B%E8%87%82%E6%8E%A7%E5%88%B6%EF%BC%89%2038d9660fafc245adafa89c32138ad84c/Untitled%2017.png)
        
        [Screencast from 2024年三月14日 19時31分56秒.webm](ROS%E5%BB%BA%E7%AB%8B%E5%B0%8E%E8%88%AA%E7%B3%BB%E7%B5%B1%EF%BC%88Moveit%E6%A9%9F%E6%A2%B0%E6%89%8B%E8%87%82%E6%8E%A7%E5%88%B6%EF%BC%89%2038d9660fafc245adafa89c32138ad84c/Screencast_from_2024%25E5%25B9%25B4%25E4%25B8%2589%25E6%259C%258814%25E6%2597%25A5_19%25E6%2599%258231%25E5%2588%258656%25E7%25A7%2592.webm)