# ROS-Navigation-04-built-map

## #ROS建立導航系統（地圖建構）

💡Note: Resource not found: navigation

![Untitled](ROS-Navigation-04-built-map%20f547c1d36f17456d90c4fc3296271dcf/Untitled.png)

安裝navigation

```tsx
sudo apt-get install ros-noetic-navigation
```

💡注意：

另外注意mobile_manipulator_gazebo_control_xacro.launch檔案中find xxxxxxx為找資料夾如果沒有找到需要修改資料夾（robot_description）

![Untitled](ROS-Navigation-04-built-map%20f547c1d36f17456d90c4fc3296271dcf/Untitled%201.png)

1. 啟動檔案與模型:~/chapter5_ws/
    
    ```tsx
    initros1
    source devel/setup.bash
    roslaunch robot_description mobile_manipulator_gazebo_control_xacro.launch
    ```
    
2. 啟動gmapping節點:~/chapter5_ws/
    
    ```tsx
    initros1
    source devel/setup.bash
    rosrun gmapping slam_gmapping
    ```
    
3. 啟動儲存庫中的RViz檔來檢視機器人正在建立的地圖:~/chapter5_ws/
    
    ```tsx
    initros1
    source devel/setup.bash
    rosrun rviz rviz -d navigation.rviz
    ```
    
    1. 要在左下角Add 加入 map
        
        ![Untitled](ROS-Navigation-04-built-map%20f547c1d36f17456d90c4fc3296271dcf/Untitled%202.png)
        
    2. 要在Topic選項中選擇/map
        
        ![Untitled](ROS-Navigation-04-built-map%20f547c1d36f17456d90c4fc3296271dcf/Untitled%203.png)
        
4. 使用teleop節點指令使機器人在環境中四處走動
    
    先安裝ros-noetic-teleop-twist-keyboard
    
    ```tsx
    sudo apt-get install ros-noetic-teleop-twist-keyboard
    ```
    
    ```tsx
    initros1
    source devel/setup.bash
    rosrun teleop_twist_keyboard teleop_twist_keyboard.py cmd_vel:=/robot_base_velocity_controller/cmd_vel
    ```
    
    可以看到控制說明
    
    ![Untitled](ROS-Navigation-04-built-map%20f547c1d36f17456d90c4fc3296271dcf/Untitled%204.png)
    
5. 測試效果
    
    ![Untitled](ROS-Navigation-04-built-map%20f547c1d36f17456d90c4fc3296271dcf/Untitled%205.png)
    
6. 在新的Terminal中開啟以下來儲存
    
    ```tsx
    *sudo apt-get install ros-noetic-map-server*
    ```
    
    ```tsx
    initros1
    source devel/setup.bash
    rosrun map_server map_saver -f postoffice
    ```
    
    會存在所在路徑下
    
    ![Untitled](ROS-Navigation-04-built-map%20f547c1d36f17456d90c4fc3296271dcf/Untitled%206.png)
    
7. 打開之前所存掃描ru的檔案postoffice.yaml
    1. 打開rivz開之前建立的地圖:r
    
    ```tsx
    initros1
    source devel/setup.bash
    rosrun map_server map_server postoffice.yaml
    ```
    
    b. 打開ros核心
    
    ```tsx
    initros1
    source devel/setup.bash
    roscore
    ```
    
    c. 打開rviz
    
    ```tsx
    rviz
    ```
    
    d. 記得要Add → map 並且將Topic中的/map打開
    
    e.  建立效果
    
    ![Untitled](ROS-Navigation-04-built-map%20f547c1d36f17456d90c4fc3296271dcf/Untitled%207.png)
    
8. 選擇環境中的點
    1. 在/chapter5_ws/src/robot_description/launch建立兩個檔案
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
            
    2. 啟動檔案
        1. 模型檔案
        
        ```tsx
        initros1
        source devel/setup.bash
        roslaunch robot_description mobile_manipulator_gazebo_control_xacro.launch 
        ```
        
        ii. 所建立的地圖
        
        ```tsx
        initros1
        source devel/setup.bash
        roslaunch robot_description mobile_manipulator_move_base.launch
        ```
        
    3. 啟動rivz
        
        ```tsx
        initros1
        source devel/setup.bash
        rosrun rviz rviz -d navigation.rviz
        ```
        
        可以 看到所建立的圖形
        
        ![Untitled](ROS-Navigation-04-built-map%20f547c1d36f17456d90c4fc3296271dcf/Untitled%208.png)
        
    4. 可以使用（Publish Point）從rivz取得座標（左下角）
        
        ![Untitled](ROS-Navigation-04-built-map%20f547c1d36f17456d90c4fc3296271dcf/Untitled%209.png)
        
        ![Untitled](ROS-Navigation-04-built-map%20f547c1d36f17456d90c4fc3296271dcf/Untitled%2010.png)
        
        ![Untitled](ROS-Navigation-04-built-map%20f547c1d36f17456d90c4fc3296271dcf/Untitled%2011.png)
        
        ![Untitled](ROS-Navigation-04-built-map%20f547c1d36f17456d90c4fc3296271dcf/Untitled%2012.png)
        

📃Reference:

[https://automaticaddison.com/setting-up-the-ros-navigation-stack-for-a-simulated-robot/](https://automaticaddison.com/setting-up-the-ros-navigation-stack-for-a-simulated-robot/)

[https://automaticaddison.com/how-to-send-a-simulated-robot-to-goal-locations-using-ros/](https://automaticaddison.com/how-to-send-a-simulated-robot-to-goal-locations-using-ros/)

[https://www.notion.so/ROS-ffd4e84fec7c4245b2396db6b5f9be06](https://www.notion.so/ROS-Navigation-02-LiDAR-ffd4e84fec7c4245b2396db6b5f9be06?pvs=21)