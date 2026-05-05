# gazebo-机械臂仿真项目日志
此处记录从模型导入到进入仿真环境构建过程

-V1.0 

遇到问题：机械臂模型的获取。首先需要有一个机械臂模型

解决方案：三种方案。首先，自己建模。其次，直接在github上找一个模型的urdf文件（这样甚至可以省去urdf的导出过程）最后，在网站上下载自己所需要的机械臂模型（为了得到自己需要的模型，同时了解urdf文件的导出过程，此处选用该方法）

在solidworks打开机械臂模型（此处以cs63机械臂为例，已装配夹爪，夹爪根据实际需要来安装）
<img width="564" height="701" alt="image" src="https://github.com/user-attachments/assets/3a1e710b-b753-4df2-a7c0-d0849ecf0ca1" />

-V1.1 

遇到问题：机械臂模型的urdf文件导出。
想要把自己的机械臂模型导入gazebo环境，需要在solidworks导出urdf文件。

解决方案：urdf文件在solidworks的导出首先需要安装插件SW2URDF
此处借鉴古月居的教程。在此给出链接https://www.bilibili.com/video/BV1Tx411o7rH/?spm_id_from=333.337.top_right_bar_window_default_collection.content.click&vd_source=1a9747ec3d5fd6acd376b9ab43f94b53

在导出urdf文件时，对机械臂模型需要做建系和旋转轴的定义处理——软件自动识别的坐标系不可靠。建系与旋转轴的定于可参考图中所示：
<img width="569" height="655" alt="image" src="https://github.com/user-attachments/assets/9b97b5f2-c0fe-424d-b605-cad9eb4151f0" />

gazebo导入模型及相应控制器指令
```
cd ~/arm_ws
colcon build --packages-select cs_cs63
source install/setup.bash
ros2 launch cs_cs63 gazebo_control.launch.py
```
加载rqt控制器：
```
ros2 run rqt_joint_trajectory_controller rqt_joint_trajectory_controller
```
夹具张紧控制器：
```
python3 gripper_slider_control.py
```

若想启动gazebo和rviz，请运行以下代码：

-V2.0 

遇到问题：在将机械臂导入模型并形成关节控制后，物体抓取的参数设置

解决方案：想要仿真实际的抓取，需要对物理属性进行定义和调整。
更新说明：
对夹爪和所抓取物体进行了物理属性的定义和参数调整
解决了抓取时穿模的问题
```
#夹具1的物理属性
  <joint name="endjoint1" type="revolute"> 
    <origin xyz="0 0.035 0.183" rpy="0 1.5708 0" />
    <parent link="link6" />
    <child link="end_link1" />
    <axis xyz="0 0 -1" />
    <limit lower="-1.57" upper="1.57" effort="300" velocity="0.3" />  #effort，最大力矩/力
  </joint>
<gazebo reference="endjoint1">
  <kp>8000.0</kp>  #控制器层面，阻尼，根据关节速度变化产生反向力矩/力，抑制振荡
  <kd>800.0</kd>  #碰撞接触刚度，提高夹具以及物体刚度以防止抓取穿模
  <damping>100.0</damping>  #阻尼系数，物理层面，数值越大 → 关节运动越“重”，减少振荡，但运动变慢
  <friction>0.1</friction>  #摩擦系数，越大 → 抓取物体不容易滑掉
</gazebo>
 #夹具2的物理属性
  <joint name="endjoint2" type="revolute">
    <origin xyz="0 -0.035 0.183" rpy="0 1.5708 0" />
    <parent link="link6" />
    <child link="end_link2" />
    <axis xyz="0 0 -1" />
    <limit lower="-1.57" upper="1.57" effort="300" velocity="0.3" />
    <mimic joint="endjoint1" multiplier="-1" offset="0" />
  </joint>
<gazebo reference="endjoint2">
  <kp>8000.0</kp>
  <kd>800.0</kd>
  <damping>100.0</damping>
  <friction>0.1</friction>
</gazebo>
```

所抓取物体的物理属性：
```
      <collision name="collision">
        <geometry>
          <box>
            <size>0.05 0.05 0.05</size>
          </box>
        </geometry>
        <surface>
          <contact>
            <ode>
              <kp>1000000.0</kp>    <!-- 极高刚度 → 完全不穿模 -->
              <kd>500.0</kd>        <!-- 强阻尼 → 不弹飞、不抖动 -->
              <max_vel>0.01</max_vel>
              <min_depth>0.0001</min_depth>
            </ode>
          </contact>
          <friction>
            <ode>
              <mu>20.0</mu>         <!-- 超高摩擦 → 不打滑 -->
              <mu2>20.0</mu2>
            </ode>
          </friction>
        </surface>
      </collision>
```
V2.1 

遇到问题：gazebo仿真力控层面无法实现对物体的真实抓取——抓取物体时滑落或弹开
更新说明：
创建了.world文件定义世界环境
```
#世界环境的定义
<?xml version="1.0"?>
<sdf version="1.6">
  <world name="default">

    <!-- 光照 -->
    <include>
      <uri>model://sun</uri>
    </include>

    <!-- 地面 -->
    <include>
      <uri>model://ground_plane</uri>
    </include>

    <!-- 物理引擎（超强防穿模、防抖动） -->
    <physics name="default_physics" default="true" type="ode">
      <max_step_size>0.001</max_step_size>
      <real_time_factor>1.0</real_time_factor>
      <real_time_update_rate>1000</real_time_update_rate>

      <ode>
        <solver>
          <type>quick</type>
          <iterations>150</iterations>
          <use_dynamic_moi_resolution>true</use_dynamic_moi_resolution>
        </solver>
        <constraints>
          <cfm>0.000001</cfm>
          <erp>0.2</erp>
          <contact_max_correcting_vel>0.01</contact_max_correcting_vel>
          <contact_surface_layer>0.0001</contact_surface_layer>
        </constraints>
      </ode>
    </physics>

    <!-- 重力 -->
    <gravity>0 0 -9.81</gravity>
  <plugin name="gazebo_link_attacher" filename="libgazebo_link_attacher.so"/>
  </world>
</sdf>
```
