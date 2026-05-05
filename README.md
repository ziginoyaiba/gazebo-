# gazebo-机械臂仿真项目日志
此处记录从模型导入到进入仿真环境构建过程

-V1.0 遇到问题：机械臂模型的获取。首先需要有一个机械臂模型
解决方案：三种方案。首先，自己建模。其次，直接在github上找一个模型的urdf文件（这样甚至可以省去urdf的导出过程）最后，在网站上下载自己所需要的机械臂模型（为了得到自己需要的模型，同时了解urdf文件的导出过程，此处选用该方法）
在solidworks打开机械臂模型（此处以cs63机械臂为例，已装配夹爪，夹爪根据实际需要来安装）
<img width="564" height="701" alt="image" src="https://github.com/user-attachments/assets/3a1e710b-b753-4df2-a7c0-d0849ecf0ca1" />

-V1.1 遇到问题：机械臂模型的urdf文件导出。
想要把自己的机械臂模型导入gazebo环境，需要在solidworks导出urdf文件。
解决方案：urdf文件在solidworks的导出首先需要安装插件SW2URDF
此处借鉴古月居的教程。在此给出链接https://www.bilibili.com/video/BV1Tx411o7rH/?spm_id_from=333.337.top_right_bar_window_default_collection.content.click&vd_source=1a9747ec3d5fd6acd376b9ab43f94b53
在导出urdf文件时，对机械臂模型需要做建系和旋转轴的定义处理——软件自动识别的坐标系不可靠。建系与旋转轴的定于可参考图中所示：
<img width="569" height="655" alt="image" src="https://github.com/user-attachments/assets/9b97b5f2-c0fe-424d-b605-cad9eb4151f0" />

-V2.0 遇到问题：在将机械臂导入模型并形成关节控制后，物体抓取的参数设置
解决方案：想要仿真实际的抓取，需要对物理属性进行定义和调整。
更新说明：
对夹爪和所抓取物体进行了物理属性的定义和参数调整
  <joint name="endjoint1" type="revolute">  #夹具1
    <origin xyz="0 0.035 0.183" rpy="0 1.5708 0" />
    <parent link="link6" />
    <child link="end_link1" />
    <axis xyz="0 0 -1" />
    <limit lower="-1.57" upper="1.57" effort="300" velocity="0.3" />  #effort，最大力矩/力
  </joint>
<gazebo reference="endjoint1">
  <damping>100.0</damping>  #阻尼系数，数值越大 → 关节运动越“重”，减少振荡，但运动变慢
  <friction>0.1</friction>  #摩擦系数，越大 → 抓取物体不容易滑掉
</gazebo>

  <joint name="endjoint2" type="revolute">  #夹具2
    <origin xyz="0 -0.035 0.183" rpy="0 1.5708 0" />
    <parent link="link6" />
    <child link="end_link2" />
    <axis xyz="0 0 -1" />
    <limit lower="-1.57" upper="1.57" effort="300" velocity="0.3" />
    <mimic joint="endjoint1" multiplier="-1" offset="0" />
  </joint>
<gazebo reference="endjoint2">
  <damping>100.0</damping>
  <friction>0.1</friction>
</gazebo>
