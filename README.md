gazebo-机械臂仿真项目日志
此处记录从模型导入到进入仿真环境构建过程

-V1.0 

问题描述：机械臂模型的获取。首先需要有一个机械臂模型

解决方案：三种方案。首先，自己建模。其次，直接在github上找一个模型的urdf文件（这样甚至可以省去urdf的导出过程）最后，在网站上下载自己所需要的机械臂模型（为了得到自己需要的模型，同时了解urdf文件的导出过程，此处选用该方法）

在solidworks打开机械臂模型（此处以cs63机械臂为例，已装配夹爪，夹爪根据实际需要来安装）
<img width="564" height="701" alt="image" src="https://github.com/user-attachments/assets/3a1e710b-b753-4df2-a7c0-d0849ecf0ca1" />

-V1.1 

问题描述：机械臂模型的urdf文件导出。

想要把自己的机械臂模型导入gazebo环境，需要在solidworks导出urdf文件。

更新说明：解决了机械臂模型的urdf文件导出问题

urdf文件在solidworks的导出首先需要安装插件SW2URDF
此处借鉴古月居的教程。在此给出链接https://www.bilibili.com/video/BV1Tx411o7rH/?spm_id_from=333.337.top_right_bar_window_default_collection.content.click&vd_source=1a9747ec3d5fd6acd376b9ab43f94b53

在导出urdf文件时，对机械臂模型需要做建系和旋转轴的定义处理——软件自动识别的坐标系不可靠。建系与旋转轴的定于可参考图中所示：
<img width="569" height="655" alt="image" src="https://github.com/user-attachments/assets/9b97b5f2-c0fe-424d-b605-cad9eb4151f0" />

V2.0

问题描述：使机械臂各个关节联系起来

更新说明：解决了机械臂的关节连接问题

未加入控制器以及关节联系的代码，机械臂各个关节会耷拉下去。
<img width="739" height="681" alt="image" src="https://github.com/user-attachments/assets/98115861-385d-45a9-a288-fdff7a2223a5" />

在模型urdf文件各个关节的<joint>后加入<transmission>,为机械臂关节配置传动机构与硬件接口，此处给出一个示例：

```
<transmission name="joint1_trans">
  <type>transmission_interface/SimpleTransmission</type>
  <joint name="joint1">
    <hardwareInterface>hardware_interface/PositionJointInterface</hardwareInterface>
  </joint>
  <actuator name="joint1_motor">
    <mechanicalReduction>1</mechanicalReduction>
  </actuator>
</transmission>
```

同时需要在模型urdf文件的</robot>标签前加上：
```
<ros2_control name="cs_cs63" type="system">
  <hardware>
    <plugin>gazebo_ros2_control/GazeboSystem</plugin>
  </hardware>

  <joint name="joint1">
    <command_interface name="position"/>
    <state_interface name="position"/>
    <state_interface name="velocity"/>
  </joint>

  <joint name="joint2">
    <command_interface name="position"/>
    <state_interface name="position"/>
    <state_interface name="velocity"/>
  </joint>

  <joint name="joint3">
    <command_interface name="position"/>
    <state_interface name="position"/>
    <state_interface name="velocity"/>
  </joint>
  
  <joint name="joint4">
    <command_interface name="position"/>
    <state_interface name="position"/>
    <state_interface name="velocity"/>
  </joint>

  <joint name="joint5">
    <command_interface name="position"/>
    <state_interface name="position"/>
    <state_interface name="velocity"/>
  </joint>

  <joint name="joint6">
    <command_interface name="position"/>
    <state_interface name="position"/>
    <state_interface name="velocity"/>
  </joint>
  
  <joint name="endjoint1">
    <command_interface name="position"/>
    <state_interface name="position"/>
    <state_interface name="velocity"/>
  </joint>
  <joint name="endjoint2">
    <command_interface name="position"/>
    <state_interface name="position"/>
    <state_interface name="velocity"/>
  </joint>
</ros2_control>

<gazebo>
  <plugin name="gazebo_ros2_control" filename="libgazebo_ros2_control.so">
    <parameters>$(find cs_cs63)/config/controllers.yaml</parameters>
    <robot_description>robot_description</robot_description>
  </plugin>
</gazebo>
```

完成各个关节的<transmission>定义后，效果如图所示：
<img width="520" height="334" alt="image" src="https://github.com/user-attachments/assets/40609a82-9b53-4ec7-89aa-172e30b52683" />

V2.1 

问题描述：对机械臂各个关节进行控制

更新说明：此处使用可视化的手动滑块控制器——rqt控制器，对模型进行控制

rqt控制器的安装，安装命令
```
sudo apt install ros-humble-joint-state-publisher-gui
```

同时需要在launch文件中加上：
```
joint_state_publisher_gui = Node(
    package='joint_state_publisher_gui',
    executable='joint_state_publisher_gui'
)
```
使文件启动时识别到控制器

在controllers.yaml文件末尾加上：
```
arm_controller:
  ros__parameters:
    joints:
      - joint1
      - joint2
      - joint3
      - joint4
      - joint5
      - joint6
    command_interfaces:
      - position
    state_interfaces:
      - position
      - velocity
    allow_partial_joints_goal: false
    open_loop_control: false
    constraints:
      stopped_velocity_tolerance: 0.01
      goal_time: 0.0
    allow_integration_in_goal_trajectories: true
```
识别控制器名称和类型

将模型导入gazebo：
```
cd ~/arm_ws
colcon build --packages-select cs_cs63
source install/setup.bash
ros2 launch cs_cs63 gazebo_control.launch.py
```

安装后启动rqt控制器命令：
```
ros2 run rqt_joint_trajectory_controller rqt_joint_trajectory_controller
```

效果如图所示：
<img width="778" height="537" alt="image" src="https://github.com/user-attachments/assets/18c37a81-3dd6-4fb9-a333-602e3ab8304f" />

V2.2

问题描述：想要控制夹具对物体进行夹持，需要控制夹具的张紧

更新说明：解决了对夹具控制的问题，可通过控制器滑块对夹具张紧程度进行调节

滑块控制器的构建，首先在scripts文件中创建python脚本：
```
import rclpy
from rclpy.node import Node
from std_msgs.msg import Float64MultiArray
import tkinter as tk
from tkinter import ttk

class GripperSliderController(Node):
    def __init__(self):
        super().__init__("gripper_slider_ctrl")
        self.pub = self.create_publisher(
            Float64MultiArray,
            "/gripper_controller/commands",
            10
        )
        # 最大角度 0.785 rad
        self.max_rad = 0.785

    def send_gripper(self, val):
        # val 是滑块0~100的值，映射到 0 ~ 0.785
        scale = float(val) / 100.0
        rad = scale * self.max_rad

        msg = Float64MultiArray()
        msg.data = [-rad, rad]
        self.pub.publish(msg)
        self.get_logger().info(f"夹紧角度: {rad:.3f} rad  百分比: {val}%")

def main():
    rclpy.init()
    node = GripperSliderController()

    # 搭建GUI滑块
    root = tk.Tk()
    root.title("夹爪实时滑块控制器")
    root.geometry("450x120")

    label = ttk.Label(root, text="拖动滑块调节夹紧程度（0%全开 → 100%全闭）")
    label.pack(pady=10)

    slider = ttk.Scale(
        root,
        from_=0,
        to=100,
        orient=tk.HORIZONTAL,
        command=node.send_gripper
    )
    slider.pack(fill=tk.X, padx=20)

    # 固定刷新ros2
    def ros_spin():
        rclpy.spin_once(node, timeout_sec=0.01)
        root.after(20, ros_spin)

    ros_spin()
    root.mainloop()

    node.destroy_node()
    rclpy.shutdown()

if __name__ == "__main__":
    main()
```

然后在controllers.yaml文件末尾加上：
```
gripper_controller:
  ros__parameters:
    joints:
      - endjoint1
      - endjoint2
    command_interfaces:
      - position
    state_interfaces:
      - position
      - velocity

    pid:
      endjoint1:
        p: 700.0
        i: 0.0
        d: 30.0
      endjoint2:
        p: 700.0
        i: 0.0
        d: 30.0
```
使得控制文件能够识别并读取你的夹具控制器。

同时也需要在launch文件加上：
```
    load_gripper_controller = ExecuteProcess(
        cmd=["ros2", "control", "load_controller", "--set-state", "active", "gripper_controller"],
        output="screen"
    )
```
使得启动后识别该控制器。

启动夹具张紧控制器命令（如果报错请在脚本所处文件夹中打开终端）：
```
python3 gripper_slider_control.py
```
效果如图所示：
<img width="1138" height="553" alt="image" src="https://github.com/user-attachments/assets/cc1523ce-d614-495c-b230-64047e8585ae" />



-V2.3

问题描述：将新模型的urdf文件导入至虚拟机/gazebo后，模型错位

更新说明：
在导入新模型后，即使模型的.stl文件相同，但不同包对应不同的结果，所以在导入新的urdf文件包后，需要使用新包的.stl文件。

导入新模型而未改stl文件：
<img width="987" height="724" alt="image" src="https://github.com/user-attachments/assets/9663127c-2426-42b3-b9ae-a6d021bf6685" />

导入新模型stl文件后，恢复正常：
<img width="880" height="669" alt="image" src="https://github.com/user-attachments/assets/f02ae683-de76-4d57-99ce-d142ed37aca9" />



-V2.4

问题描述：在将机械臂导入模型并形成关节控制后，物体抓取的参数设置。想要仿真实际的抓取，需要对物理属性进行定义和调整。

在此处给出各个参数的名称以及改动范围和作用
```
| 参数位置                                     | 名称              | 作用          | 建议范围        | 对夹取影响                        |
| ---------------------------------------- | --------------- | ----------- | ----------- | ---------------------------- |
| `<inertial>`                             | `mass`          | 物体质量，单位 kg  | 0.01 ~ 5    | 质量越大，夹爪需要更大 effort 才能抬起      |
| `<collision>/<surface>/<friction>/<ode>` | `mu`            | 主摩擦系数       | 0.5 ~ 50    | 摩擦越大，夹爪不容易滑动                 |
| `<collision>/<surface>/<friction>/<ode>` | `mu2`           | 辅摩擦系数（横向摩擦） | 0.5 ~ 50    | 同上                           |
| `<collision>/<surface>/<contact>/<ode>`  | `kp`            | 碰撞刚度        | 1e3 ~ 1e7   | 越大物体不穿透夹爪，但过大可能导致抖动          |
| `<collision>/<surface>/<contact>/<ode>`  | `kd`            | 碰撞阻尼        | 0 ~ 1e4     | 阻止物体被弹飞或震动，太低容易跳动，太高可能吸收能量过多 |
| `<collision>/<surface>/<contact>/<ode>`  | `max_vel`       | 最大碰撞修正速度    | 0.001 ~ 0.1 | 太低可能夹爪无法快速抓住物体               |
| `<collision>/<surface>/<contact>/<ode>`  | `min_depth`     | 最小碰撞深度      | 1e-5 ~ 1e-3 | 防止穿模，太大可能物体浮空                |
| `<collision>/<surface>/<contact>/<ode>`  | `soft_cfm` (可加) | 软约束参数       | 0 ~ 0.01    | 调节弹性，影响夹紧稳定性                 |
| `<collision>/<surface>/<contact>/<ode>`  | `soft_erp` (可加) | 软约束弹回系数     | 0 ~ 0.8     | 配合 soft_cfm 控制夹爪稳定性          |

```
更新说明：对夹具以及所抓取物体的物理属性进行了参数的调节
解决了抓取时穿模的问题：提高夹爪和物体刚度。

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

V2.5

问题描述：gazebo仿真力控层面无法实现对物体的真实抓取——抓取物体时滑落或弹开

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

V2.6

问题描述：在经过上述调整后仍未成功抓取物体，应该是缺少力控。

更新说明：此处做力控（effort controllers）的尝试

检查已有的ros2_control 控制器
```
ros2 pkg list | grep controller
```
看看有没有类似：
```
effort_controllers/JointTrajectoryController
effort_controllers/JointGroupEffortController
effort_controllers/JointEffortController
```

如果没有，则需要安装 effort 控制器包相应包:
```
sudo apt update
sudo apt install ros-humble-ros2-controllers ros-humble-effort-controllers
```

首先在模型urdf文件后添加力控接口
```
  <!-- 夹爪：力控接口 -->
  <joint name="endjoint1">
    <command_interface name="effort"/>
    <state_interface name="effort"/>
    <state_interface name="position"/>
    <state_interface name="velocity"/>
  </joint>
  <joint name="endjoint2">
    <command_interface name="effort"/>
    <state_interface name="effort"/>
    <state_interface name="position"/>
    <state_interface name="velocity"/>
  </joint>
</ros2_control>
```

package包添加依赖：
```
  <exec_depend>effort_controllers</exec_depend>
```

controllers.yaml文件改动：
```
gripper_controller:
  type: effort_controllers/JointGroupEffortController  # 这里添加改动
  ros__parameters:
    joints:
      - endjoint1
      - endjoint2
    command_interfaces:
      - effort            # 这里改为effort
    state_interfaces:
      - position
      - velocity

    pid:
      endjoint1:
        p: 700.0
        i: 0.0
        d: 30.0
      endjoint2:
        p: 700.0
        i: 0.0
        d: 30.0
```

重新编译启动文件：
```
cd ~/arm_ws
colcon build --packages-select cs_cs63
source install/setup.bash
ros2 launch cs_cs63 gazebo_control.launch.py
```

此次改动启动失败：libgazebo_ros2_control.so 插件加载失败，直接导致 Gazebo 崩溃

<img width="1648" height="501" alt="image" src="https://github.com/user-attachments/assets/3415af06-5d22-4c99-8ac4-6a8a1e5e2eea" />

两个夹具原来的代码定义：
```
  <joint name="endjoint1">
    <command_interface name="position"/>
    <state_interface name="position"/>
    <state_interface name="velocity"/>
  </joint>
  <joint name="endjoint2">
    <command_interface name="position"/>
    <state_interface name="position"/>
    <state_interface name="velocity"/>
  </joint>
</ros2_control>
```

在两个夹爪后增加了力控插件
```
  <!-- 夹爪：力控接口 -->
  <joint name="endjoint1">
    <command_interface name="effort"/>
    <state_interface name="effort"/>
    <state_interface name="position"/>
    <state_interface name="velocity"/>
  </joint>
  <joint name="endjoint2">
    <command_interface name="effort"/>
    <state_interface name="effort"/>
    <state_interface name="position"/>
    <state_interface name="velocity"/>
  </joint>
</ros2_control>
```

controllers.yaml文件夹爪部分加上effort部分
```
gripper_controller:
  ros__parameters:
    joints:
      - endjoint1
      - endjoint2
    command_interfaces:
      - effort
    state_interfaces:
      - position
      - velocity
      - effort  # 加了 effort state
    pid:
      endjoint1:
        p: 700.0
        i: 0.0
        d: 30.0
      endjoint2:
        p: 700.0
        i: 0.0
        d: 30.0
```

同时为了增大与物体的摩擦力，这里将夹爪的抓取面积设计更大了
<img width="863" height="742" alt="image" src="https://github.com/user-attachments/assets/6c2fb38c-5ce7-4f79-9308-1162de556245" />

运行启动文件，打开gazebo，从位置控制转到力/力矩控制，这里可以看到夹具耷拉下去了，这是因为夹具缺少初始力，因此会在重力作用下倒去
<img width="664" height="643" alt="image" src="https://github.com/user-attachments/assets/9c5ac2dd-7e1e-4edb-97c7-f79ca5a3148f" />

继续对物体进行抓取尝试后，未果。
这里通过指令
```
ros2 topic pub /gripper_controller/commands std_msgs/msg/Float64MultiArray "data: [-0.5, 0.5]"
```
向机械臂夹具发布力控信息
<img width="1118" height="748" alt="image" src="https://github.com/user-attachments/assets/05e99a7c-f05d-42a0-aade-430e4ede6136" />


现在通过给夹具配置一个力传感器，查看是否存在作用力和反作用力
```
    <gazebo reference="end_link2">
  <sensor type="force_torque" name="ft_sensor">
      <always_on>1</always_on>
      <update_rate>500</update_rate>
      <topicName>gripper/ft_sensor2</topicName>
      <frameName>end_link2</frameName>
      <gaussianNoise>0.0</gaussianNoise>
    </sensor>
  </gazebo>   #在urdf文件的<link>标签前配置传感器
```

还是没抓起来，目前夹取给我的感觉像是摩擦不存在：

现在对ATTACHLINK插件进行尝试,此处给出大佬提供插件的位置：https://github.com/IFRA-Cranfield/IFRA_LinkAttacher

如果克隆不了，请自行下载

首先安装插件包：
```bash
cd ~/arm_ws/src
git clone https://github.com/IFRA-Cranfield/IFRA_LinkAttacher.git
cd ~/arm_ws
colcon build
```

然后需要在world文件的</world>标签前加上：
```bash
<plugin name="gazebo_link_attacher" filename="libgazebo_link_attacher.so"/>
```
按理来说这个时候已经安装好插件了，但是在此处记录几个遇到的问题：

如果说在安装后突然运行不了了，请重新安装编译一次：（此处代码自用）
```bash
cd ~/arm_ws/src
git clone https://github.com/IFRA-Cranfield/IFRA_LinkAttacher.git
cd ~/arm_ws
colcon build
```
用以下命令查看自己的插件是否安装了
```bash
ros2 service list
```
成功的输出如图所示，需包含有/ATTACHLINK和/DETACHLINK
<img width="1122" height="407" alt="image" src="https://github.com/user-attachments/assets/8c7db2a1-62a5-4ece-8bcf-03122a9af437" />
每次新开终端都要执行：
```bash
cd ~/arm_ws
source install/setup.bash
```
现在启动gazebo，实行命令（夹紧or张开）：
```bash
# To ATTACH an OBJECT to a ROBOT's END-EFFECTOR:
ros2 service call /ATTACHLINK linkattacher_msgs/srv/AttachLink "{model1_name: 'model1', link1_name: 'link1', model2_name: 'model2', link2_name: 'link2'}"
# To DETACH the OBJECT from the END-EFFECTOR:
ros2 service call /DETACHLINK linkattacher_msgs/srv/DetachLink "{model1_name: 'model1', link1_name: 'link1', model2_name: 'model2', link2_name: 'link2'}"
```
插件成功运行展示：
<img width="1646" height="837" alt="image" src="https://github.com/user-attachments/assets/188c9504-3004-4b48-97ad-27bdcc23c396" />




