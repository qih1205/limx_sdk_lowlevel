**TRON1 SDK开发指南**

| 版本 | 修订日期 | 修订人 | 修改内容 | 备注 |
| :--- | :--- | :--- | :--- | :--- |
| V1.0 | 2024-10-15 | <font style="color:rgb(51, 112, 255);">@Aiden</font> | 整合明确信息，正式对外 | |
| V1.1 | 2024-12-11 | <font style="color:rgb(51, 112, 255);">@Eleven</font> | WebSocket上层应用开发接口 | |
| V1.2 | 2024-12-23 | <font style="color:rgb(51, 112, 255);">@Eleven</font> | 模型训练启用pointfoot_flat地形 | |
| V1.3 | 2024-12-24 | <font style="color:rgb(51, 112, 255);">@Jack</font> | 增加RL 训练环境 docker容器场景下启动步骤 | |
| V1.4 | 2025-3-13 | <font style="color:rgb(51, 112, 255);">@Eleven</font> | 1、上层应用接口新增里程计接口<br/>2、上层应用接口新增调整身高接口 | robot-r-3.0.4 及以上版本 |
| V1.5 | 2025-4-17 | <font style="color:rgb(51, 112, 255);">@Eleven</font> | 1、上层应用接口新增摔倒恢复接口<br/>2、上层应用接口notify_robot_info 新增摔倒状态 | robot-r-3.0.18 及以上版本 |
| V1.6 | 2025-7-18 | <font style="color:rgb(51, 112, 255);">@Eleven</font> | Pointfoot 改名 tron1 | |


<font style="color:#3370FF;">1. </font>**点足SDK 概述**

| **limxsdk-lowlevel**：在开发者模式下，用户可以使用此接口开发自己的运动控制算法，完成算法的仿真和实机无缝部署。<br/>**上层应用开发接口**：用户可以在机器人预先安装的运动控制算法基础上，利用上层应用开发接口开发自己的软件业务功能，例如建图、导航和机器人管理等。 |
| :--- |


<font style="color:#3370FF;">1.1 </font>**通讯架构图**

下图呈现了开发者的电脑与机器人本体的系统组成及交互关系。开发电脑部分涵盖运控算法节点和软件业务逻辑实现模块，通过上层应用开发接口和lowlevel的数据通讯控制机器人本体的运动。机器人本体由数据交换机、主控电脑及各类硬件组件构成，主控电脑负责协调各组件运行。

| ![](https://cdn.nlark.com/yuque/0/2025/jpeg/43111222/1757738046058-f6faf451-5221-4f81-8b0e-999fc30f6ea5.jpeg) | ![](https://cdn.nlark.com/yuque/0/2025/jpeg/43111222/1757738046444-3bdb621e-0688-43ce-ab63-644336b5495c.jpeg) |
| :---: | :---: |


<font style="color:#3370FF;">2. </font>**查看/设置机器人型号**

在编译和运行RL训练、控制算法及仿真器程序时，选择正确的机器人型号至关重要。您可以通过查看机器人型号并将其设置到环境变量 ROBOT_TYPE 中，确保在不同任务中准确识别并应用相应的机器人模型。以下是查看和配置机器人型号的步骤。

请选择并连接您机器人的 Wi-Fi 热点，密码为：12345678

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738046702-19044e77-2c83-413e-b608-b06ab47145f3.png)

在浏览器中输入[http://10.192.1.2:8080](http://10.192.1.2:8080)可以进入“机器人信息页”，并查看机器人信息。如下图所示，页面中显示的SN (序列号) 为PF_TRON1A_075，其中PF_TRON1A是机器人型号类型。

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738046865-23621a2b-b852-413b-8496-51b80e246996.png)

设置机器人型号：打开 Bash 终端，输入以下 Shell 命令来设置机器人型号。这样在编译和运行 RL 训练、控制算法以及仿真器程序时，您将能获取到正确的机器人型号信息。

| <font style="color:rgb(100, 106, 115);">Bash</font>echo 'export ROBOT_TYPE=PF_TRON1A' >> ~/.bashrc && source ~/.bashrc |
| :--- |


<font style="color:#3370FF;">3. </font>**仿真与真机调试**

| 这部分主要介绍利用我们的SDK中的example进行 **仿真和真机调试**，开发者可以参考example以及后面的接口介绍开发自己的运控程序替换我们的example来进行仿真和真机调试。建议开发好的运控程序先在gazebo中运行，效果符合预期后，再在真机上进行测试。 |
| :--- |


<font style="color:#3370FF;">3.1 </font>**搭建开发环境**

在算法开发者自己的电脑中，我们推荐在 **Ubuntu 20.04 **操作系统上建立基于 ROS Noetic 的算法开发环境。ROS提供了一系列工具和库，如核心库、通信库和仿真工具（如Gazebo），极大地便利了机器人算法的开发、测试和部署。这些资源为用户提供了一个丰富而完整的算法开发环境。

<u>当然，即使没有ROS，您也可以选择在其他环境中开发自己的运动控制算法。</u>我们提供的运动控制开发接口，是一个基于标准C++11和Python的无依赖SDK。它支持跨操作系统和平台调用开发，为开发者提供了更灵活的选择。

ROS Noetic 安装请参考文档：[https://wiki.ros.org/noetic/Installation/Ubuntu](https://wiki.ros.org/noetic/Installation/Ubuntu)，选择“ros-noetic-desktop-full”进行安装。

ROS Noetic 安装完成后，Bash终端输入以下Shell命令，安装开发环境所依赖的库：

| <font style="color:rgb(100, 106, 115);">Bash</font>sudo apt-get update   sudo apt install ros-noetic-urdf \                    ros-noetic-kdl-parser \                    ros-noetic-urdf-parser-plugin \                    ros-noetic-hardware-interface \                    ros-noetic-controller-manager \                    ros-noetic-controller-interface \                    ros-noetic-controller-manager-msgs \                    ros-noetic-control-msgs \                    ros-noetic-ros-control \                    ros-noetic-robot-state-* \                    ros-noetic-joint-state-* \                    ros-noetic-gazebo-* \                    ros-noetic-rqt-gui \                    ros-noetic-rqt-controller-manager \                    ros-noetic-plotjuggler* \                    ros-noetic-joy-teleop ros-noetic-joy \                    cmake build-essential libpcl-dev libeigen3-dev libopencv-dev libmatio-dev \                    python3-pip libboost-all-dev libtbb-dev liburdfdom-dev liborocos-kdl-dev -y |
| :--- |


<font style="color:#3370FF;">3.2 </font>**创建工作空间**

可以按照以下步骤，创建一个算法开发工作空间：

打开一个Bash终端。

创建一个新目录来存放工作空间。例如，可以在用户的主目录下创建一个名为“limx_ws”的目录：

| <font style="color:rgb(100, 106, 115);">Bash</font>mkdir -p ~/limx_ws/src |
| :--- |


下载运动控制开发接口：

| <font style="color:rgb(100, 106, 115);">Bash</font>cd ~/limx_ws/src   git clone [https://github.com/limxdynamics/limxsdk-lowlevel.git](https://github.com/limxdynamics/limxsdk-lowlevel.git) |
| :--- |


下载Gazebo仿真器：

| <font style="color:rgb(100, 106, 115);">Bash</font>cd ~/limx_ws/src   git clone [https://github.com/limxdynamics/tron1-gazebo-ros.git](https://github.com/limxdynamics/tron1-gazebo-ros.git) |
| :--- |


下载机器人模型描述文件

| <font style="color:rgb(100, 106, 115);">Bash</font>cd ~/limx_ws/src   git clone [https://github.com/limxdynamics/robot-description.git](https://github.com/limxdynamics/robot-description.git) |
| :--- |


下载可视化调试工具

| <font style="color:rgb(100, 106, 115);">Bash</font>cd ~/limx_ws/src   git clone [https://github.com/limxdynamics/robot-visualization.git](https://github.com/limxdynamics/robot-visualization.git) |
| :--- |


编译工程：

| <font style="color:rgb(100, 106, 115);">Bash</font># 如您安装了Conda，请临时禁用 Conda 环境   # 因为 Conda 会干扰 ROS 的运行环境设置   conda deactivate      cd ~/limx_ws   catkin_make install |
| :--- |


<font style="color:#3370FF;">3.3 </font>**仿真调试**

设置机器人型号：请参考“查看/设置机器人型号”章节，查看您的机器人型号。如果尚未设置，请按照以下步骤进行设置。

通过 Shell 命令 tree -L 1 src/robot-description/pointfoot 列出可用的机器人类型：

| <font style="color:rgb(100, 106, 115);">Plain Text</font>limx@limx:~$ tree -L 1 src/robot-description/pointfoot   src/robot-description/pointfoot   ├── PF_P441A   ├── PF_P441B   ├── PF_P441C   ├── PF_P441C2   ├── PF_TRON1A   ├── SF_TRON1A   └── WF_TRON1A |
| :--- |


以 PF_TRON1A（请根据实际机器人类型进行替换）为例，设置机器人型号类型：

| <font style="color:rgb(100, 106, 115);">Plain Text</font>echo 'export ROBOT_TYPE=PF_TRON1A' >> ~/.bashrc && source ~/.bashrc |
| :--- |


运行仿真器：

| <font style="color:rgb(100, 106, 115);">Bash</font># 如您安装了Conda，请临时禁用 Conda 环境   # 因为 Conda 会干扰 ROS 的运行环境设置   conda deactivate      cd ~/limx_ws   source install/setup.bash   roslaunch pointfoot_gazebo empty_world.launch |
| :--- |


运行控制例程，确保仿真器中机器人有运动，说明仿真环境搭建完成：

| <font style="color:rgb(100, 106, 115);">Bash</font># 如您安装了Conda，请临时禁用 Conda 环境   # 因为 Conda 会干扰 ROS 的运行环境设置   conda deactivate      cd ~/limx_ws   source install/setup.bash   rosrun limxsdk_lowlevel pf_groupJoints_move |
| :--- |


![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738047112-b8bae3ea-9dac-46e2-980d-3375c0ee5932.png)

注：为nanogedit

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738047351-9b87762a-0263-4093-88e6-d7fea756a9e1.png)

<font style="color:#3370FF;">3.4 </font>**真机调试**

打开开发者模式：机器人开机后，同时按下遥控器按键 R1 + Left，这时本体主机将会自动重启并切换机器人到开发者模式。在此模式下，用户可以开发自己的运动控制算法。模式设置掉电后不会失效，重新开机后仍然是开发者模式。遥控器切换工作模式的按键列表如下：

| **按键** | **模式** | **说明** |
| :--- | :--- | :--- |
| R1+Left | 开发者模式（需授权） | 用户使用运动控制开发接口开发自己的运动控制算法。 |
| R1+Right | 遥控模式 | 运行预安装的控制算法，实现复杂地形的平稳行走，如：上下台阶、过坎等。 |


修改开发者电脑IP：确保您的开发电脑与机器人本体通过外置网口连接。

设置您的电脑IP地址为：10.192.1.200，并通过Shell命令ping 10.192.1.2 能够正常ping通。如下图所示对您的开发电脑进行IP设置：

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738047556-37b49273-5281-44a1-9fbf-849579b8d7d2.png)

进行校零动作：机器人开机启动后，执行运控程序之前，请进行校零，使机器人各个关节回到初始位置。校零对应的遥控器按键为L1+R1。

设置机器人型号：请参考“查看/设置机器人型号”章节，查看您的机器人型号。如果尚未设置，请按照以下步骤进行设置。

通过 Shell 命令 tree -L 1 src/robot-description/pointfoot 列出可用的机器人类型：

| <font style="color:rgb(100, 106, 115);">Plain Text</font>limx@limx:~$ tree -L 1 src/robot-description/pointfoot   src/robot-description/pointfoot   ├── PF_P441A   ├── PF_P441B   ├── PF_P441C   ├── PF_P441C2   ├── PF_TRON1A   ├── SF_TRON1A   └── WF_TRON1A |
| :--- |


以 PF_TRON1A（请根据实际机器人类型进行替换）为例，设置机器人型号类型：

| <font style="color:rgb(100, 106, 115);">Plain Text</font>echo 'export ROBOT_TYPE=PF_TRON1A' >> ~/.bashrc && source ~/.bashrc |
| :--- |


实机部署运行。按以下方式指定机器人IP地址运行例程，实现实机部署（在进行实机部署运行时，**确保机器人吊装非常重要**）：

| <font style="color:rgb(100, 106, 115);">Bash</font># 如您安装了Conda，请临时禁用 Conda 环境   # 因为 Conda 会干扰 ROS 的运行环境设置   conda deactivate      source install/setup.bash   rosrun limxsdk_lowlevel pf_groupJoints_move 10.192.1.2 |
| :--- |


<font style="color:#3370FF;">3.5 </font>**可视化工具**

我们提供了一套可视化调试工具，可用于仿真和实际机器部署。这些工具包括使用RViz和Plotjuggler来直观展示数据。通过我们的GitHub链接：[https://github.com/limxdynamics/robot-visualization](https://github.com/limxdynamics/robot-visualization)，您可以轻松获取并使用这些工具。这些工具的使用简单直观，有助于提高开发效率和代码调试的便利性。按照以下步骤，创建一个可视化工具工作空间：

打开一个Bash终端。

创建一个新目录来存放工作空间。例如，可以在用户的主目录下创建一个名为“limx_ws”的目录：

| <font style="color:rgb(100, 106, 115);">Bash</font>mkdir -p ~/limx_ws/src |
| :--- |


下载机器人模型描述文件

| <font style="color:rgb(100, 106, 115);">Bash</font>cd ~/limx_ws/src   git clone [https://github.com/limxdynamics/robot-description.git](https://github.com/limxdynamics/robot-description.git) |
| :--- |


下载可视化工具

| <font style="color:rgb(100, 106, 115);">Bash</font>cd ~/limx_ws/src   git clone [https://github.com/limxdynamics/robot-visualization.git](https://github.com/limxdynamics/robot-visualization.git) |
| :--- |


编译可视化工具：

| <font style="color:rgb(100, 106, 115);">Bash</font># 如您安装了Conda，请临时禁用 Conda 环境   # 因为 Conda 会干扰 ROS 的运行环境设置   conda deactivate      cd ~/limx_ws   catkin_make install |
| :--- |


<font style="color:#3370FF;">3.5.1 </font>**Plotjuggler的使用**

PlotJuggler 是一个功能强大的数据可视化工具，为用户提供直观的界面，方便加载、显示和分析各种类型的数据。用户可以通过图表、曲线和图形来展示数据，从而更好地理解数据之间的关系和趋势。PlotJuggler 不仅支持实时数据的可视化，还可以加载和处理大量的历史数据。官网地址：[https://plotjuggler.io](https://plotjuggler.io)

**仿真运行：**运行Plotjugler的方法如下

| <font style="color:rgb(100, 106, 115);">Bash</font># 如您安装了Conda，请临时禁用 Conda 环境   # 因为 Conda 会干扰 ROS 的运行环境设置   conda deactivate      source install/setup.bash   roslaunch robot_visualization pointfoot_plot_sim.launch |
| :--- |


**真机运行：**运行Plotjugler的方法如下

打开pointfoot_plot_hw.launch文件，修改机器人的 IP 地址为实际机器人地址：

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738047764-03e3d52d-b2e2-4387-853d-ec89b0a41af7.png)

然后重新编译工具，执行以下命令：

| <font style="color:rgb(100, 106, 115);">Bash</font># 如您安装了Conda，请临时禁用 Conda 环境   # 因为 Conda 会干扰 ROS 的运行环境设置   conda deactivate      cd ~/limx_ws   catkin_make install |
| :--- |


编译完成后，可以通过以下 Shell 命令运行 PlotJuggler 工具：

| <font style="color:rgb(100, 106, 115);">Bash</font># 如您安装了Conda，请临时禁用 Conda 环境   # 因为 Conda 会干扰 ROS 的运行环境设置   conda deactivate      cd ~/limx_ws   source install/setup.bash   roslaunch robot_visualization pointfoot_plot_hw.launch |
| :--- |


如下所示为Plotjugler的运行效果

| **数据主题** | **说明** |
| :--- | :--- |
| /ImuData | IMU 实时数据 |
| /RobotCmdPointFoot | 机器人控制指令实时数据 |
| /RobotStatePointFoot | 机器人状态实时数据 |


![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738048233-6432bddc-a11b-4f74-909b-79666c95a827.png)

<font style="color:#3370FF;">3.5.2 </font>**RViz的使用**

在实际机器部署中，除了使用 PlotJuggler 进行数据可视化外，还可以利用 RViz 工具来实时查看机器人的运行情况。RViz 是一款强大的3D可视化工具，可用于呈现机器人的传感器数据、运动状态和环境感知。通过 RViz，用户可以实时观察 **机器人的运动轨迹、传感器数据的变化情况**，帮助调试和监控机器人的运行状态。在部署过程中，可以将 RViz 配置为订阅机器人的传感器数据话题，并显示在3D场景中，以便实时监测机器人的行为并进行必要的调整和优化。按以下步骤运行RViz：

打开pointfoot_rviz_hw.launch文件，修改机器人的 IP 地址为实际机器人地址：

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738048548-de6d6c61-3fcf-4dca-93a7-d5305aa80d7f.png)

然后重新编译工具，执行以下命令：

| <font style="color:rgb(100, 106, 115);">Bash</font># 如您安装了Conda，请临时禁用 Conda 环境   # 因为 Conda 会干扰 ROS 的运行环境设置   conda deactivate      catkin_make install |
| :--- |


编译完成后，可以通过以下 Shell 命令运行 RViz：

| <font style="color:rgb(100, 106, 115);">Bash</font># 如您安装了Conda，请临时禁用 Conda 环境   # 因为 Conda 会干扰 ROS 的运行环境设置   conda deactivate      source install/setup.bash   roslaunch robot_visualization pointfoot_rviz_hw.launch |
| :--- |


![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738048746-0b012f6d-ff30-4fda-99c6-d6084af5e567.png)

<font style="color:#3370FF;">3.6 </font>**包/日志/埋点数据**

机器人系统会自动录制：机器人的IMU数据(ImuData)、机器人状态数据（RobotStatePointFoot）以及机器人控制数据（RobotCmdPointFoot）等重要数据。这些数据对于机器人的运动控制分析至关重要。此外，机器人还会记录运行时的日志数据以及诊断埋点的结构化数据，以便在需要时进行故障排查和性能优化。

当电脑与机器人WiFi热点连接时，可以通过浏览器输入 [http://10.192.1.2:8090](http://10.192.1.2:8090)进行访问，并从中下载这些数据。这一过程为机器人的监控、维护和调试提供了便利。

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738049022-86bc9095-29bc-4ce9-a8f6-3c8dfb92139f.png)

<font style="color:#3370FF;">3.6.1 </font>**数据包可视化分析方法**

**数据包下载：**当您把.bag文件下载后，您可以使用PlotJuggler可视化工具加载这些包数据进行分析。特别需要注意的是，如果您下载的是.bag.active文件，您需要使用如下Shell命令把它重新索引.bag.active文件，生成一个新的.bag文件，以便PlotJuggler加载。

| <font style="color:rgb(100, 106, 115);">Bash</font>rosbag reindex your_file.bag.active   mv your_file.bag.active your_file.bag |
| :--- |


**可视化查看：**通过Shell命令rosrun plotjuggler plotjuggler -n启动PlotJuggler可视化工具。如下图所示加载数据包并分析数据。

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738049240-9294d879-63c6-4ec9-8918-44b6ced9bbc9.png)

<font style="color:#3370FF;">3.6.2 </font>**日志及诊断埋点数据**

如下图所示分别为日志和埋点结构化数据。在需要时可以用于故障排查和性能优化。

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738049520-1a99ba97-37a5-42e0-b12f-68a8c5458460.png)

<font style="color:#3370FF;">3.7 </font>**MuJoCo 仿真**

MuJoCo是一款轻量级且高性能的物理仿真器，专为多关节机器人和机械系统设计。它具备高效的物理引擎，能够精确处理接触和摩擦，且无需依赖 ROS，可独立运行。凭借其高速计算能力，MuJoCo 被广泛应用于机器人仿真和强化学习，尤其适合对仿真效率要求较高的场景。以下是使用 MuJoCo 进行仿真的步骤：

<font style="color:#3370FF;">3.7.1 </font>**运行MuJoCo 仿真**

打开一个 Bash 终端。

下载 MuJoCo 仿真器代码：

| <font style="color:rgb(100, 106, 115);">Plain Text</font>git clone --recurse [https://github.com/limxdynamics/tron1-mujoco-sim.git](https://github.com/limxdynamics/tron1-mujoco-sim.git) |
| :--- |


安装运动控制开发库：

Linux x86_64 环境

| <font style="color:rgb(100, 106, 115);">Plain Text</font>pip install tron1-mujoco-sim/limxsdk-lowlevel/python3/amd64/limxsdk-*-py3-none-any.whl |
| :--- |


Linux aarch64 环境

| <font style="color:rgb(100, 106, 115);">Plain Text</font>pip install tron1-mujoco-sim/limxsdk-lowlevel/python3/aarch64/limxsdk-*-py3-none-any.whl |
| :--- |


设置机器人型号：请参考“查看/设置机器人型号”章节，查看您的机器人型号。如果尚未设置，请按照以下步骤进行设置。

通过 Shell 命令 tree -L 1 tron1-mujoco-sim/robot-description/pointfoot 列出可用的机器人类型：

| <font style="color:rgb(100, 106, 115);">Plain Text</font>limx@limx:~$ tree -L 1 tron1-mujoco-sim/robot-description/pointfoot   tron1-mujoco-sim/robot-description/pointfoot   ├── PF_P441A   ├── PF_P441B   ├── PF_P441C   ├── PF_P441C2   ├── PF_TRON1A   ├── SF_TRON1A   └── WF_TRON1A |
| :--- |


以 PF_TRON1A（请根据实际机器人类型进行替换）为例，设置机器人型号类型：

| <font style="color:rgb(100, 106, 115);">Plain Text</font>echo 'export ROBOT_TYPE=PF_TRON1A' >> ~/.bashrc && source ~/.bashrc |
| :--- |


运行 MuJoCo 仿真器：

| <font style="color:rgb(100, 106, 115);">Plain Text</font>python tron1-mujoco-sim/simulator.py |
| :--- |


<font style="color:#3370FF;">3.7.2 </font>**运行控制器程序**

打开一个 Bash 终端。

安装编译所需环境

| <font style="color:rgb(100, 106, 115);">Plain Text</font>sudo apt update   sudo apt install -y cmake build-essential |
| :--- |


编译控制器SDK示例：

| <font style="color:rgb(100, 106, 115);">Plain Text</font>cd tron1-mujoco-sim/limxsdk-lowlevel   mkdir -p build   cd build   cmake ..   make |
| :--- |


设置机器人型号：请参考“查看/设置机器人型号”章节，查看您的机器人型号。如果尚未设置，请按照以下步骤进行设置。

通过 Shell 命令 tree -L 1 tron1-mujoco-sim/robot-description/pointfoot 列出可用的机器人类型：

| <font style="color:rgb(100, 106, 115);">Plain Text</font>limx@limx:~$ tree -L 1 tron1-mujoco-sim/robot-description/pointfoot   tron1-mujoco-sim/robot-description/pointfoot   ├── PF_P441A   ├── PF_P441B   ├── PF_P441C   ├── PF_P441C2   ├── PF_TRON1A   ├── SF_TRON1A   └── WF_TRON1A |
| :--- |


以 PF_TRON1A（请根据实际机器人类型进行替换）为例，设置机器人型号类型：

| <font style="color:rgb(100, 106, 115);">Plain Text</font>echo 'export ROBOT_TYPE=PF_TRON1A' >> ~/.bashrc && source ~/.bashrc |
| :--- |


运行控制器SDK示例：

| <font style="color:rgb(100, 106, 115);">Plain Text</font>./examples/pf_groupJoints_move |
| :--- |


<font style="color:#3370FF;">3.7.3 </font>**运行效果展示**

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738049827-06d33543-bf1e-4b94-b9b6-752695136e61.png)

<font style="color:#3370FF;">3.8 </font>**仿真遥控模式功能**

请从“下载中心”下载软件包并安装，下载地址：[https://support.limxdynamics.com/down-center](https://support.limxdynamics.com/down-center)

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738050161-e2537244-2e40-4c6b-b733-0d59ecb2e580.png)

以您下载的文件 robot-tron1-r-3.4.9.20250708095546.tar.gz（请替换为实际的软件包）为例，按下面方法解压并安装到指定目录：

| <font style="color:rgb(100, 106, 115);">Shell</font>sudo mkdir -p /opt/limx \     && tar -xzvf robot-tron1-r-3.4.9.20250708095546.tar.gz \     && cd robot-tron1-r-3.4.9.20250708095546 \     && sudo tar -xzvf robot-tron1-r-3.4.9.20250708095546.tar.gz -C /opt/limx \     && cd .. && rm -rf robot-tron1-r-3.4.9.20250708095546 \     && echo 'source /opt/limx/install/setup.bash' >> ~/.bashrc \     && source ~/.bashrc |
| :--- |


以 WF_TRON1A（请根据实际机器人类型进行替换）为例，设置机器人型号类型：

| <font style="color:rgb(100, 106, 115);">Plain Text</font>echo 'export ROBOT_TYPE=WF_TRON1A' >> ~/.bashrc && source ~/.bashrc |
| :--- |


打开一个终端，运行仿真器：您可以根据需要选择MuJoco或Gazebo仿真器，这里我们以Gazebo仿真器为例启动仿真器。启动Gazebo仿真器之前，请按前面章节正确搭建开发环境并编译好Gazebo仿真器。

| <font style="color:rgb(100, 106, 115);">Shell</font># 如您安装了Conda，请临时禁用 Conda 环境   # 因为 Conda 会干扰 ROS 的运行环境设置   conda deactivate      cd ~/limx_ws   source install/setup.bash   roslaunch pointfoot_gazebo empty_world.launch |
| :--- |


打开一个终端，运行机器人算法软件。如机器人摔倒状态，通过虚拟遥控器组合按键L2 + Y恢复。请根据机器人类型选择对应的启动命令：

WF_TRON1A 启动命令：

| <font style="color:rgb(100, 106, 115);">Shell</font># 为仿真需要，设置一个假SN   setRobotSN WF_TRON1A_001      # 启动算法软件   mroslaunch ${MROS_ETC_PATH}/tron1_controllers/tron1a_wheelfoot_controllers_sim.launch  |
| :--- |


PF_TRON1A 启动命令：

| <font style="color:rgb(100, 106, 115);">Shell</font># 为仿真需要，设置一个假SN   setRobotSN PF_TRON1A_001      # 启动算法软件   mroslaunch ${MROS_ETC_PATH}/tron1_controllers/tron1a_pointfoot_controllers_sim.launch  |
| :--- |


SF_TRON1A 启动命令：

| <font style="color:rgb(100, 106, 115);">Shell</font># 为仿真需要，设置一个假SN   setRobotSN SF_TRON1A_001      # 启动算法软件   mroslaunch ${MROS_ETC_PATH}/tron1_controllers/tron1a_solefoot_controllers_sim.launch  |
| :--- |


打开一个终端，下载并运行虚拟遥控器

| <font style="color:rgb(100, 106, 115);">Bash</font># 下载虚拟遥控器   git clone [https://github.com/limxdynamics/robot-joystick.git](https://github.com/limxdynamics/robot-joystick.git)      # 运行虚拟遥控器   ./robot-joystick/robot-joystick |
| :--- |


此时，您可以按下表虚拟遥控器按键功能操作机器人：

| 按键 | 功能 |
| :--- | :--- |
| L1 + B  | 进入蹲起状态，对应遥控器“L1 + O” 按键 |
| L1 + Y | 进入行走状态，对应遥控器“L1 + △” 按键 |
| L1 + A | 进入蹲下状态，对应遥控器“L1 + X” 按键 |
| L2 + Y | 摔倒爬起，对应遥控器“L2 + △” 按键 |
| 左右摇杆 | 控制机器人行走 |


<font style="color:#3370FF;">4. </font>**底层运动控制开发接口**

**跨平台底层运动控制开发接口库**提供统一的C++/Python API，兼容ROS1、ROS2及非ROS系统，实现运动控制算法的快速移植与部署。通过硬件抽象层和标准化通信协议，开发者可无缝切换仿真与真实硬件环境，显著降低多平台适配成本。

<font style="color:#3370FF;">4.1 </font>**C++ 运动控制开发接口**

<font style="color:#3370FF;">4.1.1 </font>**getInstance 接口介绍**

| 函数名 | **getInstance** |
| :--- | --- |
| 函数原型 | static PointFoot* getInstance(); |
| 功能概述 | 获取 PointFoot 机器人类单例实例的指针 |
| 参数 | 无 |
| 返回值 | PointFoot*，指向 PointFoot 实例的指针 |
| 备注 | 使用了单例模式，确保 PointFoot 类只有一个实例存在于程序中 |
| 代码示例 |  |


<font style="color:#3370FF;">4.1.2 </font>**init 接口介绍**

| 函数名 | **init** |
| :--- | --- |
| 函数原型 | bool init(const std::string& robot_ip_address = "127.0.0.1"); |
| 功能概述 | **初始化**运动控制算法程序的通信运行环境，通常在主函数中调用其它接口之前调用，完成初始化工作。 |
| 参数 | robot_ip_address：机器人的 IP 地址。对于仿真，通常设置为 "127.0.0.1"，而对于真实机器人，可能设置为 "10.192.1.2"。 |
| 返回值 | 如果初始化成功，则返回 true；否则返回 false。 |
| 备注 | 无 |
| 代码示例 |  |


<font style="color:#3370FF;">4.1.3 </font>**getMotorNumber 接口介绍**

| 函数名 | **getMotorNumber** |
| :--- | --- |
| 函数原型 | uint32_t getMotorNumber(); |
| 功能概述 | 获取机器人中的**电机数量**。 |
| 参数 | 无 |
| 返回值 | 返回一个无符号整数，表示机器人中的总电机数量。 |
| 备注 | 通常情况下，点足机器人的电机数量为6个 |
| 代码示例 |  |


<font style="color:#3370FF;">4.1.4 </font>**subscribeImuData 接口介绍**

| **函数名** | **subscribeImuData** |
| :--- | --- |
| 函数原型 | void subscribeImuData(std::function<void(const ImuDataConstPtr&)> cb); |
| 功能概述 | 订阅机器人的 **IMU数据**，并在接收到新的 IMU 数据时调用指定的回调函数。 |
| 参数 | cb: 用于处理新 IMU 数据的回调函数。 |
| 返回值 | 无 |
| 备注 | ImuData 数据结构原型如下：<br/> |
| 代码示例 |  |


<font style="color:#3370FF;">4.1.5 </font>**subscribeRobotState 接口介绍**

| **函数名** | **subscribeRobotState** |
| :--- | --- |
| 函数原型 | void subscribeRobotState(std::function<void(const RobotStateConstPtr&)> cb); |
| 功能概述 | 订阅接收关于**机器人状态**的更新。 |
| 参数 | cb：回调函数，当接收到机器人状态更新时将被调用。回调函数参数指向 RobotState 对象的常量指针。 |
| 返回值 | 无 |
| 备注 | RobotState 数据结构原型如下：<br/>机器人状态数据tau、q、dq数组对应的电机顺序如下：<br/>点足<br/>0: abad_L_Joint, 1: hip_L_Joint, 2: knee_L_Joint<br/>3: abad_R_Joint, 4: hip_R_Joint, 5: knee_R_Joint<br/>点轮足<br/>0: abad_L_Joint, 1: hip_L_Joint, 2: knee_L_Joint, 3: wheel_L_Joint<br/>4: abad_R_Joint, 5: hip_R_Joint, 6: knee_R_Joint, 6: wheel_R_Joint<br/>点双足<br/>0: abad_L_Joint, 1: hip_L_Joint, 2: knee_L_Joint, 3: ankle_L_Joint<br/>4: abad_R_Joint, 5: hip_R_Joint, 6: knee_R_Joint, 6: ankle_R_Joint |
| 代码示例 |  |


<font style="color:#3370FF;">4.1.6 </font>**publishRobotCmd 接口介绍**

| **函数名** | **publishRobotCmd** |
| :--- | --- |
| 函数原型 | bool publishRobotCmd(const RobotCmd& cmd); |
| 功能概述 | 发布一个命令来 **控制机器人的动作**。 |
| 参数 | cmd：表示所需机器人命令的 RobotCmd 对象。<br/>命令数据数组对应的电机顺序如下：<br/>点足<br/>0: abad_L_Joint, 1: hip_L_Joint, 2: knee_L_Joint<br/>3: abad_R_Joint, 4: hip_R_Joint, 5: knee_R_Joint<br/>点轮足<br/>0: abad_L_Joint, 1: hip_L_Joint, 2: knee_L_Joint, 3: wheel_L_Joint<br/>4: abad_R_Joint, 5: hip_R_Joint, 6: knee_R_Joint, 6: wheel_R_Joint<br/>点双足<br/>0: abad_L_Joint, 1: hip_L_Joint, 2: knee_L_Joint, 3: ankle_L_Joint<br/>4: abad_R_Joint, 5: hip_R_Joint, 6: knee_R_Joint, 6: ankle_R_Joint |
| 返回值 | 无 |
| 备注 | RobotCmd 数据结构原型如下：<br/> |
| 代码示例 |  |


<font style="color:#3370FF;">4.1.7 </font>**subscribeSensorJoy 接口介绍**

| **函数名** | **subscribeSensorJoy** |
| :--- | :--- |
| 函数原型 | void subscribeSensorJoy(std::function<void(const SensorJoyConstPtr&)> cb); |
| 功能概述 | 在真机部署中，该方法用于订阅来自机器人**遥控器的数据**。当机器人接收到遥控器的数据时，将会调用指定的回调函数，并传递包含遥控器数据的 SensorJoy 结构体常量指针给回调函数进行处理。 |
| 参数 | cb: 表示回调函数，用于接收机器人遥控器的数据。回调函数的参数类型为 SensorJoyConstPtr，即指向 SensorJoy 结构体常量的共享指针。 |
| 返回值 | 无 |
| 备注 | SensorJoy 数据结构原型如下：<br/><br/>遥控器摇杆映射<br/><br/>遥控器按键映射<br/><br/>适配您自己的按键逻辑时，建议不要和系统预留按键功能冲突了。开发者模式下系统预留按键功能： |
| 代码示例 | |


<font style="color:#3370FF;">4.1.8 </font>**subscribeDiagnosticValue 接口介绍**

| 函数名 | **subscribeDiagnosticValue** |
| :--- | :--- |
| 函数原型 | void subscribeDiagnosticValue(std::function<void(const DiagnosticValueConstPtr&)> cb); |
| 功能概述 | 在真机部署中，该方法用于**订阅机器人的诊断值和状态信息**。当机器人发出诊断值时，系统会调用指定的回调函数，并传递包含诊断值的 DiagnosticValue 结构体常量指针给回调函数进行处理。这可以帮助实时监控机器人的健康状态，并及时做出反应以处理可能的问题。 |
| 参数 | cb: 用于接收机器人诊断值的回调函数，其参数类型为 DiagnosticValueConstPtr，即指向 DiagnosticValue 结构体常量的共享指针。DiagnosticValue 结构体包含了机器人诊断值的信息，包括时间戳、级别、名称、代码和消息字段。 |
| 返回值 | 无 |
| 备注 | DiagnosticValue 数据结构原型如下：<br/><br/>常见诊断数据<br/> |
| 代码示例 | |


<font style="color:#3370FF;">4.1.9 </font>**setRobotLightEffect 接口介绍**

| 函数名 | **setRobotLightEffect** |
| :--- | :--- |
| 函数原型 | bool setRobotLightEffect(int effect); |
| 功能概述 | 在真机部署中，该方法用于**设置机器人的灯光效果**。 |
| 参数 | effect: 一个整数，表示所需的机器人灯光效果，具体定义见 `PointFoot::LightEffect` 枚举。 |
| 返回值 | bool: 指示机器人灯光效果是否成功设置。 |
| 备注 | PointFoot::LightEffect 枚举定义：<br/> |
| 代码示例 | |


<font style="color:#3370FF;">4.1.10 </font>**参考例程**

单关节控制例程：

[https://github.com/limxdynamics/limxsdk-lowlevel/blob/master/examples/pf_joint_move.cpp](https://github.com/limxdynamics/limxsdk-lowlevel/blob/master/examples/pf_joint_move.cpp)

多关节控制例程：

[https://github.com/limxdynamics/limxsdk-lowlevel/blob/master/examples/pf_groupJoints_move.cpp](https://github.com/limxdynamics/limxsdk-lowlevel/blob/master/examples/pf_groupJoints_move.cpp)

<font style="color:#3370FF;">4.2 </font>**Python 运动控制开发接口**

<font style="color:#3370FF;">4.2.1 </font>**概述**

提供与C++相同功能的[<font style="color:#3370FF;">Python运动控制算法开发接口</font>](https://github.com/limxdynamics/pointfoot-sdk-lowlevel/tree/master/python3)，**使得不熟悉C++编程语言的开发者能够使用Python进行运动控制算法的开发**。

Python语言易于学习，具有简洁清晰的语法和丰富的第三方库，使开发者能够更快速地上手并迅速实现算法。通过Python接口，开发者可以利用Python的动态特性进行快速原型设计和实验验证，加速算法的迭代和优化过程。同时，Python的跨平台性和强大的生态系统支持，使得运动算法能够更广泛地应用于不同平台和环境。

此外，RL（强化学习）模型的快速部署到仿真和真机环境中也得益于Python的灵活性，开发者可以使用Python轻松地将RL模型集成到各种仿真平台和真实硬件中，实现快速迭代和验证算法的性能。

<font style="color:#3370FF;">4.2.2 </font>**安装运动控制开发库**

Linux x86_64 环境

| <font style="color:rgb(100, 106, 115);">Bash</font>git clone [https://github.com/limxdynamics/limxsdk-lowlevel.git](https://github.com/limxdynamics/limxsdk-lowlevel.git)   pip install limxsdk-lowlevel/python3/amd64/limxsdk-*-py3-none-any.whl |
| :--- |


Linux aarch64 环境

| <font style="color:rgb(100, 106, 115);">Bash</font>git clone [https://github.com/limxdynamics/limxsdk-lowlevel.git](https://github.com/limxdynamics/limxsdk-lowlevel.git)   pip install limxsdk-lowlevel/python3/aarch64/limxsdk-*-py3-none-any.whl |
| :--- |


Windows 环境

| <font style="color:rgb(100, 106, 115);">Bash</font>git clone [https://github.com/limxdynamics/limxsdk-lowlevel.git](https://github.com/limxdynamics/limxsdk-lowlevel.git)   pip install limxsdk-lowlevel/python3/win/limxsdk-*-py3-none-any.whl |
| :--- |


<font style="color:#3370FF;">4.2.3 </font>**__init__ 接口介绍**

| **函数名** | **__init__** |
| :--- | :--- |
| 函数原型 | def __init__(self, robot_type: robot.RobotType) |
| 功能概述 | 在初始化时，指定机器人的类型，并创建一个相应类型的本地机器人实例。 |
| 参数 | robot_type：表示机器人类型的枚举值，点足为RobotType.PointFoot。 |
| 返回值 | 无 |
| 备注 | 无 |
| 代码示例 | |


<font style="color:#3370FF;">4.2.4 </font>**init 接口介绍**

| **函数名** | **init** |
| :--- | --- |
| 函数原型 | def init(self, robot_ip: str = "127.0.0.1") |
| 功能概述 | 初始化运动控制算法程序的通信运行环境，通常在主函数中调用其它接口之前调用，完成初始化工作。 |
| 参数 | robot_ip：机器人的 IP 地址。对于仿真，通常设置为 "127.0.0.1"，而对于真实机器人，可能设置为 "10.192.1.2"。 |
| 返回值 | 成功：返回True<br/>失败：返回False |
| 备注 | 无 |
| 代码示例 |  |


<font style="color:#3370FF;">4.2.5 </font>**getMotorNumber 接口介绍**

| 函数名 | **getMotorNumber** |
| :--- | --- |
| 函数原型 | def getJointLimit(self, timeout: float = -1.0) |
| 功能概述 | 获取机器人中的电机数量。 |
| 参数 | 无 |
| 返回值 | 返回一个无符号整数，表示机器人中的总电机数量。 |
| 备注 | 通常情况下，点足机器人的电机数量为6个 |
| 代码示例 |  |


<font style="color:#3370FF;">4.2.6 </font>** subscribeImuData 接口介绍**

| 函数名 | **subscribeImuData** |
| :--- | --- |
| 函数原型 | def subscribeImuData(self, callback: Callable[[datatypes.ImuData], Any]) |
| 功能概述 | 订阅机器人的 IMU数据，并在接收到新的 IMU 数据时调用指定的回调函数。 |
| 参数 | callback：用于处理新 IMU 数据的回调函数。 |
| 返回值 | 成功：返回True<br/>失败：返回False |
| 备注 | datatypes.ImuData 数据结构原型如下：<br/> |
| 代码示例 |  |


<font style="color:#3370FF;">4.2.7 </font>**subscribeRobotState 接口介绍**

| 函数名 | **subscribeRobotState** |
| :--- | --- |
| 函数原型 | def subscribeRobotState(self, callback: Callable[[datatypes.RobotState], Any]) |
| 功能概述 | 订阅接收关于机器人状态的更新。 |
| 参数 | callback：回调函数，当接收到机器人状态更新时将被调用。回调函数参数指向datatypes.RobotState 对象。<br/>机器人状态数据数组对应的电机顺序如下：<br/>点足<br/>0: abad_L_Joint, 1: hip_L_Joint, 2: knee_L_Joint<br/>3: abad_R_Joint, 4: hip_R_Joint, 5: knee_R_Joint<br/>点轮足<br/>0: abad_L_Joint, 1: hip_L_Joint, 2: knee_L_Joint, 3: wheel_L_Joint<br/>4: abad_R_Joint, 5: hip_R_Joint, 6: knee_R_Joint, 6: wheel_R_Joint<br/>点双足<br/>0: abad_L_Joint, 1: hip_L_Joint, 2: knee_L_Joint, 3: ankle_L_Joint<br/>4: abad_R_Joint, 5: hip_R_Joint, 6: knee_R_Joint, 6: ankle_R_Joint<br/>datatypes.RobotState数据结构字段：<br/>stamp：时间戳，通常表示记录或生成这些数据的时间。<br/>tau：用于存储当前估计的输出扭矩（以牛顿米为单位）的向量。<br/>q：用于存储当前角度（以弧度为单位）的向量。<br/>dq：用于存储当前速度（以弧度每秒为单位）的向量。 |
| 返回值 | 成功：返回True<br/>失败：返回False |
| 备注 | datatypes.RobotState 数据结构原型如下：<br/> |
| 代码示例 |  |


<font style="color:#3370FF;">4.2.8 </font>**publishRobotCmd 接口介绍**

| 函数名 | **publishRobotCmd** |
| :--- | --- |
| 函数原型 | def publishRobotCmd(self, cmd: datatypes.RobotCmd) |
| 功能概述 | 发布一个命令来控制机器人的动作。 |
| 参数 | cmd：表示所需机器人命令的 datatypes.RobotCmd 对象，包含以下字段：<br/>stamp：记录或生成数据时的时间戳，以纳秒为单位。<br/>mode：机器人控制模式：0: 力矩模式控制；1：速度模式控制；2：位置模式控制，默认设置为：0。<br/>q：存储所需的关节角度（以弧度为单位）的向量。<br/>dq：存储所需的关节速度（以弧度每秒为单位）的向量。<br/>tau：存储所需的输出扭矩（以牛顿米为单位）的向量。<br/>Kp：存储所需的位置刚度（以牛顿米每弧度为单位）的向量。<br/>Kd：存储所需的速度刚度（以牛顿米每弧度每秒为单位）的向量。<br/>命令数据数组对应的电机顺序如下：<br/>点足<br/>0: abad_L_Joint, 1: hip_L_Joint, 2: knee_L_Joint<br/>3: abad_R_Joint, 4: hip_R_Joint, 5: knee_R_Joint<br/>点轮足<br/>0: abad_L_Joint, 1: hip_L_Joint, 2: knee_L_Joint, 3: wheel_L_Joint<br/>4: abad_R_Joint, 5: hip_R_Joint, 6: knee_R_Joint, 6: wheel_R_Joint<br/>点双足<br/>0: abad_L_Joint, 1: hip_L_Joint, 2: knee_L_Joint, 3: ankle_L_Joint<br/>4: abad_R_Joint, 5: hip_R_Joint, 6: knee_R_Joint, 6: ankle_R_Joint |
| 返回值 | 成功：返回True<br/>失败：返回False |
| 备注 | datatypes.RobotCmd 数据结构原型如下：<br/> |
| 代码示例 |  |


<font style="color:#3370FF;">4.2.9 </font>**subscribeSensorJoy 接口介绍**

| 函数名 | **subscribeSensorJoy** |
| :--- | :--- |
| 函数原型 | def subscribeSensorJoy(self, callback: Callable[[datatypes.SensorJoy], Any]) |
| 功能概述 | 在真机部署中，该方法用于订阅来自机器人遥控器的数据。当机器人接收到遥控器的数据时，将会调用指定的回调函数，并传递包含遥控器数据的 datatypes.SensorJoy 结构体常量指针给回调函数进行处理。 |
| 参数 | callback: 表示回调函数，用于接收机器人遥控器的数据。回调函数的参数类型为 datatypes.SensorJoy |
| 返回值 | 成功：返回True<br/>失败：返回False |
| 备注 | datatypes.SensorJoy 数据结构原型如下：<br/><br/>遥控器摇杆映射<br/><br/>遥控器按键映射<br/><br/>适配您自己的按键逻辑时，建议不要和系统预留按键功能冲突了。开发者模式下系统预留按键功能：<br/> |
| 代码示例 | |


<font style="color:#3370FF;">4.2.10 </font>**subscribeDiagnosticValue 接口介绍**

| 函数名 | **subscribeDiagnosticValue** |
| :--- | :--- |
| 函数原型 | def subscribeDiagnosticValue(self, callback: Callable[[datatypes.DiagnosticValue], Any]) |
| 功能概述 | 在真机部署中，该方法用于订阅机器人的诊断值和状态信息。当机器人发出诊断值时，系统会调用指定的回调函数，并传递包含诊断值的 datatypes.DiagnosticValue 结构对象给回调函数进行处理。这可以帮助实时监控机器人的健康状态，并及时做出反应以处理可能的问题。 |
| 参数 | callback: 用于接收机器人诊断值的回调函数，其参数类型为 datatypes.DiagnosticValue。datatypes.DiagnosticValue 结构体包含了机器人诊断值的信息，包括时间戳、级别、名称、代码和消息字段。 |
| 返回值 | 成功：返回True<br/>失败：返回False |
| 备注 | datatypes.DiagnosticValue 数据结构原型如下：<br/><br/>常见诊断数据<br/> |
| 代码示例 | |


<font style="color:#3370FF;">4.2.11 </font>**setRobotLightEffect 接口介绍**

| 函数名 | **setRobotLightEffect** |
| :--- | :--- |
| 函数原型 | def setRobotLightEffect(self, effect: datatypes.LightEffect) |
| 功能概述 | 在真机部署中，该方法用于设置机器人的灯光效果。 |
| 参数 | effect: 一个整数，表示所需的机器人灯光效果，具体定义见 `PointFoot::LightEffect` 枚举。 |
| 返回值 | 成功：返回True<br/>失败：返回False |
| 备注 | datatypes.LightEffect 枚举定义：<br/> |
| 代码示例 | |


<font style="color:#3370FF;">4.2.12 </font>**参考例程**

Python 接口参考例程

[https://github.com/limxdynamics/limxsdk-lowlevel/blob/master/python3/amd64/example.py](https://github.com/limxdynamics/limxsdk-lowlevel/blob/master/python3/amd64/example.py)

<font style="color:#3370FF;">5. </font>**上层应用开发接口**

<font style="color:#3370FF;">5.1 </font>**概述**

在遥控模式下机器人通过WebSocket通信端口5000来接收用户端请求指令，例如让机器人站起、蹲下、行走等。WebSocket是一种实时通信协议，在机器人和用户端之间建立长连接，以便快速有效地传输控制信息和数据。如下图所示：

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738050433-4f39586f-1adf-4547-ad55-a14356f1f0ca.png)

<font style="color:#3370FF;">5.2 </font>**通信协议格式**

当机器人通过 WebSocket 接收客户端指令时，采用 JSON 数据协议进行信息传递。这种方式具有显著优势：WebSocket 是一种全双工通信协议，能够在客户端与服务器之间建立实时、低延迟的连接，特别适合频繁交互的应用场景。JSON 数据协议则以其简洁、可读性强的结构，确保数据传输直观明了，且具有跨平台、跨语言的兼容性。WebSocket 与 JSON 的结合不仅与编程语言无关，适用于各种设备和系统，还能提升开发的灵活性和维护的便利性。

请求数据格式包含以下字段：

accid：机器人序列号，注意修为您机器人的序列号；

title：指令名称，以“request_”为前缀；

timestamp：指令发出时间戳，单位为毫秒；

guid：指令的唯一标识符，用于区分不同的请求指令。如果是同步接口，则需要在“response_xxx”响应消息中通过guid字段将值带回给客户端。客户端接收到响应消息后，可以通过比较guid字段的值是否与请求指令中的值相同来判断指令是否执行完成；

data：存放请求指令的数据内容。可以根据具体需求包含多个子字段，以存放请求指令所需的数据内容，例如执行动作的参数、发送消息的文本内容等等；

示例如下：

| <font style="color:rgb(100, 106, 115);">Plain Text</font>{     "accid": "PF_TRON1A_075", # 机器人序列号，注意修为您机器人的序列号     "title": "request_xxx",   # 指令名称，以“request_”为前缀     "timestamp": 1672373633989, # 指令发出时间戳，单位为毫秒     "guid": "746d937cd8094f6a98c9577aaf213d98", # 指令的唯一标识符，用于区分不同的请求指令     "data": {}  # 存放请求指令的数据内容   } |
| :--- |


响应数据格式包含以下字段：

accid：机器人序列号，注意修为您机器人的序列号；

title：指令名称，以“response_”为前缀；

timestamp：指令发出时间戳，单位为毫秒；

guid：与对应请求指令的guid值相同；

data：至少应该包含一个“result”子字段，用于存放请求指令的执行结果数据。如果有需要，还可以包含其他子字段，例如错误码、错误信息等用于描述操作结果的信息；

示例如下：

| <font style="color:rgb(100, 106, 115);">JSON   </font><font style="color:rgb(100, 106, 115);">代码块</font>{     "accid": "PF_TRON1A_075",   # 机器人序列号，注意修为您机器人的序列号     "title": "response_xxx",  # 指令名称，以“response_”为前缀     "timestamp": 1672373633989, # 指令发出时间戳，单位为毫秒     "guid": "746d937cd8094f6a98c9577aaf213d98", # 与对应请求指令的guid值相同     "data": { # 存放响应指令的具体数据内容       "result": "success"  # “result” 用于存放请求指令处理是否成功，它的值为：“success 或 fail_xxx”     }   } |
| :--- |


消息推送：它是机器人主动向客户端发送信息的过程。这些信息可以包括机器人的序列号、当前运行状态、执行的操作等数据。通过及时地向客户端发送这些信息，机器人可以帮助客户端更好地理解它的工作状态，从而更好地使用它提供的服务。它的数据格式包含以下字段：

accid：机器人序列号，注意修为您机器人的序列号；

title：指令名称，以“notify_”为前缀；

timestamp：消息发出时间戳，单位为毫秒；

guid：消息的guid值，唯一标识这条消息；

data：存放消息数据内容。可以根据具体需求包含多个子字段，以存放请求指令所需的数据内容；

示例如下：

| <font style="color:rgb(100, 106, 115);">JSON   </font><font style="color:rgb(100, 106, 115);">代码块</font>{     "accid": "PF_TRON1A_075",   # 机器人序列号，注意修为您机器人的序列号     "title": "notify_xxx",  # 消息名称，以“notify_”为前缀     "timestamp": 1672373633989, # 消息发出时间戳，单位为毫秒     "guid": "746d937cd8094f6a98c9577aaf213d98", # 消息的guid值，唯一标识这条消息     "data": { } # 存放消息数据内容   } |
| :--- |


<font style="color:#3370FF;">5.3 </font>**查看软件序列号(ACCID)**

连接机器人无线网络

机器人开机完成后，使用个人电脑连接机器人Wi-Fi，名称格式通常为「PF_TRON1A_xxx」

输入Wi-Fi密码：12345678

在浏览器中输入[http://10.192.1.2:8080](http://10.192.1.2:8080)可以进入“机器人信息页”，并查看机器人信息。如下图所示，页面中显示的SN (序列号) 为PF_TRON1A_075，其中PF_TRON1A_075便是此机器人的软件序列号。

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738050660-bab49c6c-1315-4a5b-8506-b7a65e6c0212.png)

<font style="color:#3370FF;">5.4 </font>**通信测试方法**

Postman是一个流行的API开发环境，可以用于测试WebSocket接口。使用Postman测试WebSocket接口，请按照以下步骤操作：

安装postman，下载地址：[https://www.postman.com/downloads/?utm_source=postman-home](https://www.postman.com/downloads/?utm_source=postman-home)；

打开Postman，并创建一个WebSocket的请求；

连接机器人无线网络

机器人开机完成后，使用个人电脑连接机器人Wi-Fi，名称格式通常为「WF_TRON1A_xxx」

输入Wi-Fi密码：12345678

在请求的URL中输入WebSocket接口的地址，例如，“ws://10.192.1.2:5000”;

在“Message”中，输入要发送的指令请求；

单击“Send”按钮，发送请求指令；

发送指令后，可以从服务器接收响应消息。使用Postman的响应窗口查看服务器返回的数据，并检查是否符合预期结果。

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738050867-e71a39f0-8ddf-427a-a491-259dead879e3.png)

<font style="color:#3370FF;">5.5 </font>**协议接口定义**

该机器人接口设计遵循与遥控器操控一致的流程和状态流转，确保调用顺序、响应时序及状态过渡与遥控器控制逻辑严格对齐。用户通过接口调用可获得如同使用遥控器的直观体验，同时支持遥控器与接口间的无缝切换，实现统一、稳定的机器人操控效果。

![](https://cdn.nlark.com/yuque/0/2025/jpeg/43111222/1757738051315-5eb5d271-7401-430f-a67e-1453beb2f553.jpeg)

<font style="color:#3370FF;">5.5.1 </font>**蹲起状态**

<font style="color:#3370FF;">5.5.1.1 </font>**请求：request_stand_mode**

| <font style="color:rgb(100, 106, 115);">JSON</font>{     "accid": "PF_TRON1A_075",  // 机器人序列号，注意修为您机器人的序列号；     "title": "request_stand_mode",     "timestamp": 1672373633989,     "guid": "746d937cd8094f6a98c9577aaf213d98",     "data": {     }   } |
| :--- |


<font style="color:#3370FF;">5.5.1.2 </font>**响应：response_stand_mode**

| <font style="color:rgb(100, 106, 115);">JSON</font>{     "accid": "PF_TRON1A_075",     "title": "response_stand_mode",     "timestamp": 1672373633989,     "guid": "746d937cd8094f6a98c9577aaf213d98",     "data": {       "result": "success"  // success: 成功, fail_imu: IMU 错误, fail_motor: 电机错误   } |
| :--- |


<font style="color:#3370FF;">5.5.1.3 </font>**消息推送：notify_stand_mode**

机器人站起过程失败或完成后，主动推送此消息。

| <font style="color:rgb(100, 106, 115);">JSON</font>{     "accid": "PF_TRON1A_075",     "title": "notify_stand_mode",     "timestamp": 1672373633989,     "guid": "746d937cd8094f6a98c9577aaf213d98",     "data": {       "result": "success"  // success: 成功, fail_imu: IMU 错误, fail_motor: 电机错误     }   } |
| :--- |


<font style="color:#3370FF;">5.5.2 </font>**行走状态**

<font style="color:#3370FF;">5.5.2.1 </font>**请求：request_walk_mode**

| <font style="color:rgb(100, 106, 115);">JSON</font>{     "accid": "PF_TRON1A_075", // 机器人序列号，注意修为您机器人的序列号；     "title": "request_walk_mode",     "timestamp": 1672373633989,     "guid": "746d937cd8094f6a98c9577aaf213d98",     "data": {     }   } |
| :--- |


<font style="color:#3370FF;">5.5.2.2 </font>**响应：response_walk_mode**

| <font style="color:rgb(100, 106, 115);">JSON</font>{     "accid": "PF_TRON1A_075",     "title": "response_walk_mode",     "timestamp": 1672373633989,     "guid": "746d937cd8094f6a98c9577aaf213d98",     "data": {       "result": "success"  // success: 成功, fail_imu: IMU 错误, fail_motor: 电机错误   } |
| :--- |


<font style="color:#3370FF;">5.5.2.3 </font>**消息推送：notify_walk_mode**

机器人站起过程失败或完成后，主动推送此消息。

| <font style="color:rgb(100, 106, 115);">JSON</font>{     "accid": "PF_TRON1A_075",     "title": "notify_walk_mode",     "timestamp": 1672373633989,     "guid": "746d937cd8094f6a98c9577aaf213d98",     "data": {       "result": "success"  // success: 成功, fail_imu: IMU 错误, fail_motor: 电机错误     }   } |
| :--- |


<font style="color:#3370FF;">5.5.3 </font>**控制行走**

<font style="color:#3370FF;">5.5.3.1 </font>**请求：request_twist**

请按30Hz及以上发送指令。

| <font style="color:rgb(100, 106, 115);">JSON</font>{     "accid": "PF_TRON1A_075", // 机器人序列号，注意修为您机器人的序列号；     "title": "request_twist",     "timestamp": 1672373633989,     "guid": "746d937cd8094f6a98c9577aaf213d98",     "data": {       "x": 0.0,   //  前进后退速度比值，取值范围[-1, 1]       "y": 0.0,   //  横向行走速度比值，取值范围[-1, 1]       "z": 0.0    //  旋转角速度比值，取值范围[-1, 1]     }   } |
| :--- |


<font style="color:#3370FF;">5.5.3.2 </font>**响应：无**

<font style="color:#3370FF;">5.5.3.3 </font>**消息推送：notify_twist**

机器人行走失败时，主动推送此消息。

| <font style="color:rgb(100, 106, 115);">JSON</font>{     "accid": "PF_TRON1A_075",     "title": "notify_twist",     "timestamp": 1672373633989,     "guid": "746d937cd8094f6a98c9577aaf213d98",     "data": {       "result": "fail_motor"  // fail_imu: IMU 错误, fail_motor: 电机错误     }   } |
| :--- |


<font style="color:#3370FF;">5.5.4 </font>**调整机器身高**

<font style="color:#3370FF;">5.5.4.1 </font>**请求：request_base_height**

| <font style="color:rgb(100, 106, 115);">JSON</font>{     "accid": "PF_TRON1A_075", // 机器人序列号，注意修为您机器人的序列号；     "title": "request_base_height",     "timestamp": 1672373633989,     "guid": "746d937cd8094f6a98c9577aaf213d98",     "data": {       "direction": -1  // 1：表示升高，-1：表示降低                        // 每次调用此请求会使机器人身高相应升高或降低 5cm     }   } |
| :--- |


<font style="color:#3370FF;">5.5.4.2 </font>**响应：response_base_height**

| <font style="color:rgb(100, 106, 115);">JSON</font>{     "accid": "PF_TRON1A_075",     "title": "response_base_height",     "timestamp": 1672373633989,     "guid": "746d937cd8094f6a98c9577aaf213d98",     "data": {       "result": "success"  // success: 成功, fail_status: 则表示机器人当前状态不允许调整身高     }   } |
| :--- |


<font style="color:#3370FF;">5.5.4.3 </font>**消息推送：无**

<font style="color:#3370FF;">5.5.5 </font>**蹲下**

<font style="color:#3370FF;">5.5.5.1 </font>**请求：request_sitdown**

| <font style="color:rgb(100, 106, 115);">JSON</font>{     "accid": "PF_TRON1A_075", // 机器人序列号，注意修为您机器人的序列号；     "title": "request_sitdown",     "timestamp": 1672373633989,     "guid": "746d937cd8094f6a98c9577aaf213d98",     "data": {}   } |
| :--- |


<font style="color:#3370FF;">5.5.5.2 </font>**响应：response_sitdown**

| <font style="color:rgb(100, 106, 115);">JSON</font>{     "accid": "PF_TRON1A_075",     "title": "response_sitdown",     "timestamp": 1672373633989,     "guid": "746d937cd8094f6a98c9577aaf213d98",     "data": {       "result": "success"  // success: 成功, fail_imu: IMU 错误, fail_motor: 电机错误     }   } |
| :--- |


<font style="color:#3370FF;">5.5.5.3 </font>**消息推送：notify_sitdown**

机器人蹲下过程失败或完成后，主动推送此消息。

| <font style="color:rgb(100, 106, 115);">JSON</font>{     "accid": "PF_TRON1A_075",     "title": "notify_sitdown",     "timestamp": 1672373633989,     "guid": "746d937cd8094f6a98c9577aaf213d98",     "data": {       "result": "success"  // success: 成功, fail_imu: IMU 错误, fail_motor: 电机错误     }   } |
| :--- |


<font style="color:#3370FF;">5.5.6 </font>**开启楼梯模式**

<font style="color:#3370FF;">5.5.6.1 </font>**请求：request_stair_mode**

本功能仅适用于 TRON1 型号的双轮足机器人。

| <font style="color:rgb(100, 106, 115);">JSON</font>{     "accid": "PF_TRON1A_075", // 机器人序列号，注意修为您机器人的序列号；     "title": "request_stair_mode",     "timestamp": 1672373633989,     "guid": "746d937cd8094f6a98c9577aaf213d98",     "data": {         "enable": true  // true: 开启楼梯模式, false: 关闭楼梯模式     }   } |
| :--- |


<font style="color:#3370FF;">5.5.6.2 </font>**响应：response_stair_mode**

| <font style="color:rgb(100, 106, 115);">JSON</font>{     "accid": "PF_TRON1A_075",     "title": "response_stair_mode",     "timestamp": 1672373633989,     "guid": "746d937cd8094f6a98c9577aaf213d98",     "data": {       "result": "success"  // success: 成功, fail_imu: IMU 错误, fail_motor: 电机错误     }   } |
| :--- |


<font style="color:#3370FF;">5.5.6.3 </font>**消息推送：无**

<font style="color:#3370FF;">5.5.7 </font>**开启踏步模式**

<font style="color:#3370FF;">5.5.7.1 </font>**请求：request_marktime_mode**

本功能仅适用于 TRON1 型号的双足机器人。

| <font style="color:rgb(100, 106, 115);">JSON</font>{     "accid": "PF_TRON1A_075", // 机器人序列号，注意修为您机器人的序列号；     "title": "request_marktime_mode",     "timestamp": 1672373633989,     "guid": "746d937cd8094f6a98c9577aaf213d98",     "data": {         "enable": true  // true: 开启踏步模式, false: 关闭踏步模式     }   } |
| :--- |


<font style="color:#3370FF;">5.5.7.2 </font>**响应：response_marktime_mode**

| <font style="color:rgb(100, 106, 115);">JSON</font>{     "accid": "PF_TRON1A_075",     "title": "response_marktime_mode",     "timestamp": 1672373633989,     "guid": "746d937cd8094f6a98c9577aaf213d98",     "data": {       "result": "success"  // success: 成功, fail_imu: IMU 错误, fail_motor: 电机错误     }   } |
| :--- |


<font style="color:#3370FF;">5.5.7.3 </font>**消息推送：无**

<font style="color:#3370FF;">5.5.8 </font>**紧急停止**

<font style="color:#3370FF;">5.5.8.1 </font>**请求：request_emgy_stop**

| <font style="color:rgb(100, 106, 115);">JSON</font>{     "accid": "PF_TRON1A_075", // 机器人序列号，注意修为您机器人的序列号；     "title": "request_emgy_stop",     "timestamp": 1672373633989,     "guid": "746d937cd8094f6a98c9577aaf213d98",     "data": {}   } |
| :--- |


<font style="color:#3370FF;">5.5.8.2 </font>**响应：response_emgy_stop**

| <font style="color:rgb(100, 106, 115);">JSON</font>{     "accid": "PF_TRON1A_075",     "title": "response_emgy_stop",     "timestamp": 1672373633989,     "guid": "746d937cd8094f6a98c9577aaf213d98",     "data": {       "result": "success"  // success: 成功, fail_imu: IMU 错误, fail_motor: 电机错误     }   } |
| :--- |


<font style="color:#3370FF;">5.5.8.3 </font>**消息推送：无**

<font style="color:#3370FF;">5.5.9 </font>**开启里程计**

<font style="color:#3370FF;">5.5.9.1 </font>**请求：request_enable_odom**

该功能用于开启里程计推送，开启后系统将主动推送里程计数据（注意：轮足机器人才有里程计）。

| <font style="color:rgb(100, 106, 115);">JSON</font>{     "accid": "WF_TRON1A_075", // 机器人序列号，注意修为您机器人的序列号；     "title": "request_enable_odom",     "timestamp": 1672373633989,     "guid": "746d937cd8094f6a98c9577aaf213d98",     "data": {         "enable": true  // true: 开启里程计, false: 禁用里程计     }   } |
| :--- |


<font style="color:#3370FF;">5.5.9.2 </font>**响应：response_enable_imu**

| <font style="color:rgb(100, 106, 115);">JSON</font>{     "accid": "WF_TRON1A_075",     "title": "response_enable_odom",     "timestamp": 1672373633989,     "guid": "746d937cd8094f6a98c9577aaf213d98",     "data": {       "result": "success"  // success: 成功, fail_odom: 不支持     }   } |
| :--- |


<font style="color:#3370FF;">5.5.9.3 </font>**消息推送：notify_odom**

开启里程计后，系统将主动推送包含里程计数据的消息。

| <font style="color:rgb(100, 106, 115);">JSON</font>{     "accid": "WF_TRON1A_075",      "title": "notify_odom",      "timestamp": 1672373633989,     "guid": "746d937cd8094f6a98c9577aaf213d98",     "data": {         "pose_orientation": [0.0, 0.0, 0.0, 0.0], // 姿态 [x, y, z, w]         "pose_position": [0.0, 0.0, 0.0],         // 位置 [x, y, z] in m         "twist_linear": [0.0, 0.0, 0.0],          // 线速度 [x, y, z] in m/s         "twist_angular": [0.0, 0.0, 0.0]          // 角速度 [x, y, z] in rad/s     }   } |
| :--- |


<font style="color:#3370FF;">5.5.10 </font>**开启IMU数据**

<font style="color:#3370FF;">5.5.10.1 </font>**请求：request_enable_imu**

该功能用于开启IMU数据推送，开启后系统将主动推送IMU数据。

| <font style="color:rgb(100, 106, 115);">JSON</font>{     "accid": "PF_TRON1A_075", // 机器人序列号，注意修为您机器人的序列号；     "title": "request_enable_imu",     "timestamp": 1672373633989,     "guid": "746d937cd8094f6a98c9577aaf213d98",     "data": {         "enable": true  // true: 开启IMU, false: 禁用IMU     }   } |
| :--- |


<font style="color:#3370FF;">5.5.10.2 </font>**响应：response_enable_imu**

| <font style="color:rgb(100, 106, 115);">JSON</font>{     "accid": "PF_TRON1A_075",     "title": "response_enable_imu",     "timestamp": 1672373633989,     "guid": "746d937cd8094f6a98c9577aaf213d98",     "data": {       "result": "success"  // success: 成功, fail_imu: IMU     }   } |
| :--- |


<font style="color:#3370FF;">5.5.10.3 </font>**消息推送：notify_imu**

开启IMU数据后，系统将主动推送包含IMU状态的消息。

| <font style="color:rgb(100, 106, 115);">JSON</font>{     "accid": "PF_TRON1A_075",      "title": "notify_imu",      "timestamp": 1672373633989,     "guid": "746d937cd8094f6a98c9577aaf213d98",     "data": {         "euler": [0.0, 0.0, 0.0],     // 欧拉角 [roll, pitch, yaw] in degrees         "acc": [0.0, 0.0, 0.0],       // 加速度 [x, y, z] in m/s²         "gyro": [0.0, 0.0, 0.0],      // 陀螺仪角速度 [x, y, z] in rad/s         "quat": [0.0, 0.0, 0.0, 0.0]  // 四元数 [w, x, y, z]     }   } |
| :--- |


<font style="color:#3370FF;">5.5.11 </font>**摔倒恢复**

<font style="color:#3370FF;">5.5.11.1 </font>**请求：request_recover**

当机器人摔倒后，可以调用此接口让机器人自动爬起来并恢复到行走模式。

| <font style="color:rgb(100, 106, 115);">JSON</font>{     "accid": "PF_TRON1A_075", // 机器人序列号，注意修为您机器人的序列号；     "title": "request_recover",     "timestamp": 1672373633989,     "guid": "746d937cd8094f6a98c9577aaf213d98",     "data": {}   } |
| :--- |


<font style="color:#3370FF;">5.5.11.2 </font>**响应：response_recover**

| <font style="color:rgb(100, 106, 115);">JSON</font>{     "accid": "PF_TRON1A_075",     "title": "response_recover",     "timestamp": 1672373633989,     "guid": "746d937cd8094f6a98c9577aaf213d98",     "data": {       "result": "success"  // success: 成功收到指令，开始恢复爬起, fail_no_fallover: 当前没摔倒     }   } |
| :--- |


<font style="color:#3370FF;">5.5.11.3 </font>**消息推送：notify_recover**

完成恢复爬起后，推送此消息。

| <font style="color:rgb(100, 106, 115);">json</font>{     "accid": "PF_TRON1A_075",      "title": "notify_recover",      "timestamp": 1672373633989,     "guid": "746d937cd8094f6a98c9577aaf213d98",     "data": {         "result": "success"  // success: 摔倒恢复成功, fail_recover: 摔倒恢复失败     }   } |
| :--- |


<font style="color:#3370FF;">5.5.12 </font>**设置灯效**

<font style="color:#3370FF;">5.5.12.1 </font>**请求：request_light_effect**

该接口用于设置机器人的灯光效果。用户可以通过此接口向机器人发送特定的灯光效果指令，机器人会根据指令调整其灯光显示。

| <font style="color:rgb(100, 106, 115);">JSON</font>{     "accid": "PF_TRON1A_075", // 机器人序列号，注意修为您机器人的序列号；     "title": "request_light_effect",     "timestamp": 1672373633989,     "guid": "746d937cd8094f6a98c9577aaf213d98",     "data": {         "effect": 1     }   } |
| :--- |


请求参数说明：

| **参数名** | **类型** | **描述** |
| :--- | :--- | :--- |
| data.effect | 数字 | 灯光效果的编号，对应不同的灯光显示模式，具体映射关系如下：   1: STATIC_RED（静态红色）   2: STATIC_GREEN（静态绿色）   3: STATIC_BLUE（静态蓝色）   4: STATIC_CYAN（静态青色）   5: STATIC_PURPLE（静态紫色）   6: STATIC_YELLOW（静态黄色）   7: STATIC_WHITE（静态白色）   8: LOW_FLASH_RED（低频闪烁红色）   9: LOW_FLASH_GREEN（低频闪烁绿色）   10: LOW_FLASH_BLUE（低频闪烁蓝色）   11: LOW_FLASH_CYAN（低频闪烁青色）   12: LOW_FLASH_PURPLE（低频闪烁紫色）   13: LOW_FLASH_YELLOW（低频闪烁黄色）   14: LOW_FLASH_WHITE（低频闪烁白色）   15: FAST_FLASH_RED（高频闪烁红色）   16: FAST_FLASH_GREEN（高频闪烁绿色）   17: FAST_FLASH_BLUE（高频闪烁蓝色）   18: FAST_FLASH_CYAN（高频闪烁青色）   19: FAST_FLASH_PURPLE（高频闪烁紫色）   20: FAST_FLASH_YELLOW（高频闪烁黄色）   21: FAST_FLASH_WHITE（高频闪烁白色） |


<font style="color:#3370FF;">5.5.12.2 </font>**响应：response_light_effect**

| <font style="color:rgb(100, 106, 115);">JSON</font>{     "accid": "PF_TRON1A_075",     "title": "response_light_effect",     "timestamp": 1672373633989,     "guid": "746d937cd8094f6a98c9577aaf213d98",     "data": {       "result": "success"  // success: 成功, fail_light_effect: 失败     }   } |
| :--- |


<font style="color:#3370FF;">5.5.12.3 </font>**消息推送：无**

<font style="color:#3370FF;">5.5.13 </font>**全局消息**

<font style="color:#3370FF;">5.5.13.1 </font>**机器人基本信息**

机器人基本信息每秒上报一次，包含以下内容：

accid：机器人序列号

title：notify_robot_info

timestamp：消息发出时间戳，单位为毫秒

guid：消息的guid值，唯一标识这条消息

data：存放消息内容，示例如下：

| <font style="color:rgb(100, 106, 115);">JSON</font>{     "accid": "PF_TRON1A_075",      "title": "notify_robot_info",      "timestamp": 1672373633989,     "guid": "746d937cd8094f6a98c9577aaf213d98",     "data": {       "accid": "PF_TRON1A_075",       "sw_version": "robot-tron1-2.0.10.20241111103012",       "imu": "OK",    # 机器人IMU诊断信息       "camera": "OK", # 机器人相机诊断信息       "motor": "OK",  # 机器人电机诊断信息       "battery": 95,  # 机器人电量       "status": "WALK" # 机器人运行模式     }   } |
| :--- |




| **字段** | **说明** |
| :---: | :--- |
| accid | 机器人序列号 |
| sw_version | 机器人本体软件版本信息 |
| imu | 机器人IMU诊断信息 |
| camera | 机器人相机诊断信息 |
| motor | 机器人电机诊断信息 |
| battery | 机器人电池电量 |
| status | 机器人运行模式：STAND、WALK、SIT、DAMPING、ROTATE、STAIR、ERROR_FALLOVER（摔倒）、RECOVER(摔倒恢复中), ERROR_RECOVER（摔倒恢复失败） |


<font style="color:#3370FF;">5.5.13.2 </font>**非法指令消息**

当机器人收到非法格式的请求指令时，发送此消息，包含以下内容：

accid：机器人序列号

title：notify_invalid_request

timestamp：消息发出时间戳，单位为毫秒

guid：消息的guid值，唯一标识这条消息

data：存放消息内容

示例如下：

| <font style="color:rgb(100, 106, 115);">JSON</font>{     "accid": "PF_TRON1A_075",      "title": "notify_invalid_request",      "timestamp": 1672373633989,     "guid": "746d937cd8094f6a98c9577aaf213d98",     "data": "返回原请求指令内容，便于客户端排查问题"   } |
| :--- |


<font style="color:#3370FF;">5.6 </font>**协议接口调用示例**

<font style="color:#3370FF;">5.6.1 </font>**Linux C++ 示例实现**

安装依赖：以Ubuntu 20.04系统为例，安装websocketpp、nlohmann/json和boost依赖：

| <font style="color:rgb(100, 106, 115);">Bash</font>sudo apt-get install libboost-all-dev libwebsocketpp-dev nlohmann-json3-dev |
| :--- |


编译代码

| <font style="color:rgb(100, 106, 115);">Bash</font>g++ -std=c++11 -o websocket_client websocket_client.cpp -lssl -lcrypto -lboost_system -lpthread |
| :--- |


运行程序

| <font style="color:rgb(100, 106, 115);">Bash</font>./websocket_client |
| :--- |


websocket_client.cpp 实现

| <font style="color:rgb(100, 106, 115);">C++</font>#include <iostream>   #include <atomic>   #include <string>   #include <thread>   #include <chrono>   #include <websocketpp/client.hpp>   #include <websocketpp/config/asio.hpp>    #include <nlohmann/json.hpp>   #include <boost/uuid/uuid.hpp>   #include <boost/uuid/uuid_generators.hpp>   #include <boost/uuid/uuid_io.hpp>      using json = nlohmann::json;   using websocketpp::client;   using websocketpp::connection_hdl;      // Replace this ACCID value with your robot's actual serial number (SN)   static std::string ACCID = "";      // WebSocket client instance   static client<websocketpp::config::asio> ws_client;      // Atomic flag for graceful exit   static std::atomic<bool> should_exit(false);      // Connection handle for sending messages   static connection_hdl current_hdl;      // Generate dynamic GUID   static std::string generate_guid() {       boost::uuids::random_generator gen;       boost::uuids::uuid u = gen();       return boost::uuids::to_string(u);   }      // Send WebSocket request with title and data   static void send_request(const std::string& title, const json& data = json::object()) {       json message;              // Adding necessary fields to the message       message["accid"] = ACCID;       message["title"] = title;       message["timestamp"] = std::chrono::duration_cast<std::chrono::milliseconds>(                                   std::chrono::system_clock::now().time_since_epoch()).count();       message["guid"] = generate_guid();       message["data"] = data;          std::string message_str = message.dump();              // Send the message through WebSocket       ws_client.send(current_hdl, message_str, websocketpp::frame::opcode::text);   }      // Handle user commands   static void handle_commands() {       while (!should_exit) {           std::string command;           std::cout << "Enter command ('stand', 'walk', 'twist', 'sit', 'stair', 'stop', 'imu') or 'exit' to quit:" << std::endl;           std::getline(std::cin, command);  // Read user input              if (command == "exit") {               should_exit = true;  // Exit flag to stop the loop               break;           } else if (command == "stand") {               send_request("request_stand_mode");  // Send stand mode request           } else if (command == "walk") {               send_request("request_walk_mode");  // Send walk mode request           } else if (command == "twist") {               float x, y, z;               std::cout << "Enter x, y, z values:" << std::endl;               std::cin >> x >> y >> z;  // Get twist values from user               send_request("request_twist", {{"x", x}, {"y", y}, {"z", z}});           } else if (command == "sit") {               send_request("request_sitdown");  // Send sit down request           } else if (command == "stair") {               std::string enable;               std::cout << "Enable stair mode (true/false):" << std::endl;               std::cin >> enable;  // Get stair mode enable flag from user               send_request("request_stair_mode", {{"enable", enable == "true" ? true : false}});           } else if (command == "stop") {               send_request("request_emgy_stop");  // Send emergency stop request           } else if (command == "imu") {               std::string enable;               std::cout << "Enable IMU (true/false):" << std::endl;               std::cin >> enable;  // Get IMU enable flag from user               send_request("request_enable_imu", {{"enable", enable == "true" ? true : false}});           }       }   }      // WebSocket open callback   static void on_open(connection_hdl hdl) {       std::cout << "Connected!" << std::endl;              // Save connection handle for sending messages later       current_hdl = hdl;          // Start handling commands in a separate thread       std::thread(handle_commands).detach();   }      // WebSocket message callback   static void on_message(connection_hdl hdl, client<websocketpp::config::asio>::message_ptr msg) {       // Parse JSON data from message payload       json data = json::parse(msg->get_payload());                    // Extract 'accid' field if present       if (data.contains("accid") && data["accid"].is_string() && ACCID.empty()) {           ACCID = data["accid"].get<std::string>();       }             }      // WebSocket close callback   static void on_close(connection_hdl hdl) {       std::cout << "Connection closed." << std::endl;   }      // Close WebSocket connection   static void close_connection(connection_hdl hdl) {       ws_client.close(hdl, websocketpp::close::status::normal, "Normal closure");  // Close connection normally   }      int main() {       ws_client.init_asio();  // Initialize ASIO for WebSocket client              // Set WebSocket event handlers       ws_client.set_open_handler(&on_open);  // Set open handler       ws_client.set_message_handler(&on_message);  // Set message handler       ws_client.set_close_handler(&on_close);  // Set close handler          std::string server_uri = "ws://10.192.1.2:5000";  // WebSocket server URI          websocketpp::lib::error_code ec;       client<websocketpp::config::asio>::connection_ptr con = ws_client.get_connection(server_uri, ec);  // Get connection pointer          if (ec) {              return 1;  // Exit if connection error occurs       }          connection_hdl hdl = con->get_handle();  // Get connection handle       ws_client.connect(con);  // Connect to server       std::cout << "Press Ctrl+C to exit." << std::endl;              // Run the WebSocket client loop       ws_client.run();          return 0;   } |
| :--- |


<font style="color:#3370FF;">5.6.2 </font>**Python 示例实现**

环境准备：以Ubuntu 20.04系统为例，安装下面依赖

| <font style="color:rgb(100, 106, 115);">Python</font>sudo apt install python3-dev python3-pip   sudo pip3 install websocket-client |
| :--- |


运行脚本

| <font style="color:rgb(100, 106, 115);">Python</font>python3 websocket_client.py |
| :--- |


websocket_client.py 实现

| <font style="color:rgb(100, 106, 115);">Python</font>import json   import uuid   import threading   import time   import websocket   from datetime import datetime      # Replace this ACCID value with your robot's actual serial number (SN)   ACCID = None      # Atomic flag for graceful exit   should_exit = False      # WebSocket client instance   ws_client = None      # Generate dynamic GUID   def generate_guid():       return str(uuid.uuid4())      # Send WebSocket request with title and data   def send_request(title, data=None):       if data is None:           data = {}              # Create message structure with necessary fields       message = {           "accid": ACCID,           "title": title,           "timestamp": int(time.time() * 1000),  # Current timestamp in milliseconds           "guid": generate_guid(),           "data": data       }          message_str = json.dumps(message)              # Send the message through WebSocket if client is connected       if ws_client:           ws_client.send(message_str)      # Handle user commands   def handle_commands():       global should_exit       while not should_exit:           command = input("Enter command ('stand', 'walk', 'twist', 'sit', 'stair', 'stop', 'imu') or 'exit' to quit:\n")                      if command == "exit":               should_exit = True  # Set exit flag to stop the loop               break           elif command == "stand":               send_request("request_stand_mode")  # Send stand mode request           elif command == "walk":               send_request("request_walk_mode")  # Send walk mode request           elif command == "twist":               # Get twist values from user               x = float(input("Enter x value:"))               y = float(input("Enter y value:"))               z = float(input("Enter z value:"))               for _ in range(30):                   send_request("request_twist", {"x": x, "y": y, "z": z})                   time.sleep(1/30)           elif command == "sit":               send_request("request_sitdown")  # Send sit down request           elif command == "stair":               # Get stair mode enable flag from user               enable = input("Enable stair mode (true/false):").strip().lower() == 'true'               send_request("request_stair_mode", {"enable": enable})           elif command == "stop":               send_request("request_emgy_stop")  # Send emergency stop request           elif command == "imu":               # Get IMU enable flag from user               enable = input("Enable IMU (true/false):").strip().lower() == 'true'               send_request("request_enable_imu", {"enable": enable})      # WebSocket on_open callback   def on_open(ws):       print("Connected!")       # Start handling commands in a separate thread       threading.Thread(target=handle_commands, daemon=True).start()      # WebSocket on_message callback   def on_message(ws, message):       global ACCID       root = json.loads(message)       ACCID = root.get("accid", None)       print(f"Received message: {message}")  # Print the received message      # WebSocket on_close callback   def on_close(ws, close_status_code, close_msg):       print("Connection closed.")      # Close WebSocket connection   def close_connection(ws):       ws.close()      def main():       global ws_client              # Create WebSocket client instance       ws_client = websocket.WebSocketApp(           "ws://10.192.1.2:5000",  # WebSocket server URI           on_open=on_open,           on_message=on_message,           on_close=on_close       )              # Run WebSocket client loop       print("Press Ctrl+C to exit.")       ws_client.run_forever()      if __name__ == "__main__":       main() |
| :--- |


<font style="color:#3370FF;">5.6.3 </font>**JavaScript 示例实现**

HTML 页面：为了便于与用户交互，您可以在 HTML 页面中添加一个输入框，用户可以在其中输入命令。以下是 index.html 实现：

| <font style="color:rgb(100, 106, 115);">HTML</font><!DOCTYPE html>   <html lang="en">   <head>       <meta charset="UTF-8">       <meta name="viewport" content="width=device-width, initial-scale=1.0">       <title>WebSocket Robot Control</title>       <style>           #commandInput {               width: 400px; /* Adjust the width to make it wider */               padding: 10px;               font-size: 14px;           }       </style>   </head>   <body>       <h2>Robot Control Commands</h2>       <input type="text" id="commandInput" placeholder="Enter command ('stand', 'walk', 'twist', 'sit', 'stair', 'stop', 'imu')">       <p>Type a command and press Enter.</p>          <script src="robotControl.js"></script>   </body>   </html> |
| :--- |


robotControl.js 实现：

| <font style="color:rgb(100, 106, 115);">JavaScript</font>// Replace this ACCID value with your robot's actual serial number (SN)   let ACCID = "";      // WebSocket client instance   let wsClient = null;      // Generate dynamic GUID   function generateGuid() {       return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function(c) {           const r = Math.random() * 16 | 0,                 v = c === 'x' ? r : (r & 0x3 | 0x8);           return v.toString(16);       });   }      // Send WebSocket request with title and data   function sendRequest(title, data = {}) {       const message = {           accid: ACCID,           title: title,           timestamp: Date.now(),  // Current timestamp in milliseconds           guid: generateGuid(),           data: data       };          // Send the message through WebSocket if client is connected       if (wsClient && wsClient.readyState === WebSocket.OPEN) {           wsClient.send(JSON.stringify(message));       }   }      // Handle user commands   function handleCommands() {       const commandInput = document.getElementById('commandInput');       commandInput.addEventListener('keydown', function(event) {           if (event.key === 'Enter') {               const command = commandInput.value.trim();               commandInput.value = '';  // Clear input field                  switch (command) {                   case 'stand':                       sendRequest('request_stand_mode');                       break;                   case 'walk':                       sendRequest('request_walk_mode');                       break;                   case 'twist':                       const x = parseFloat(prompt("Enter x value:"));                       const y = parseFloat(prompt("Enter y value:"));                       const z = parseFloat(prompt("Enter z value:"));                       sendRequest('request_twist', {x, y, z});                       break;                   case 'sit':                       sendRequest('request_sitdown');                       break;                   case 'stair':                       const enableStair = prompt("Enable stair mode (true/false):").toLowerCase() === 'true';                       sendRequest('request_stair_mode', {enable: enableStair});                       break;                   case 'stop':                       sendRequest('request_emgy_stop');                       break;                   case 'imu':                       const enableImu = prompt("Enable IMU (true/false):").toLowerCase() === 'true';                       sendRequest('request_enable_imu', {enable: enableImu});                       break;                   case 'exit':                       wsClient.close();                       break;                   default:                       alert("Invalid command. Try again.");               }           }       });   }      // WebSocket onOpen callback   function onOpen() {       console.log("Connected!");       handleCommands();   }      // WebSocket onMessage callback   function onMessage(event) {       console.log("Received message:", event.data);       const message = JSON.parse(event.data);       if (!ACCID && message.accid) {           ACCID = message.accid;           console.log(`ACCID set to: ${ACCID}`);       }   }      // WebSocket onClose callback   function onClose(event) {       console.log("Connection closed.");   }      // Initialize WebSocket client   function initWebSocket() {       // Replace this URL with your WebSocket server URI       wsClient = new WebSocket('ws://10.192.1.2:5000');          wsClient.onopen = onOpen;       wsClient.onmessage = onMessage;       wsClient.onclose = onClose;          console.log("Press Ctrl+C to exit.");   }      // Start WebSocket connection when the page loads   window.onload = initWebSocket; |
| :--- |


运行程序

将 index.html 和 robotControl.js 文件保存到同一目录下，然后在浏览器中打开 index.html 运行。你可以通过浏览器的开发者工具查看接收到的详细信息。

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738051614-4856e9e7-72f7-4b54-a4b9-ae3ac8b1ca81.png)

<font style="color:#3370FF;">5.6.4 </font>**Go 示例实现**

首先需要安装gorilla/websocket包：

| <font style="color:rgb(100, 106, 115);">JavaScript</font>go get github.com/gorilla/websocket |
| :--- |


Go实现的代码如下：

| <font style="color:rgb(100, 106, 115);">JavaScript</font>package main      import (           "encoding/json"           "fmt"           "github.com/gorilla/websocket"           "time"           "strings"           "github.com/google/uuid"   )      // Global variables   var wsClient *websocket.Conn   var shouldExit bool      // Replace with your robot's serial number   var ACCID = ""       // Generate dynamic GUID   func generateGUID() string {           return uuid.New().String()   }      // Send WebSocket request with title and data   func sendRequest(title string, data map[string]interface{}) {           if data == nil {                   data = make(map[string]interface{})           }              message := map[string]interface{}{                   "accid":    ACCID,                   "title":    title,                   "timestamp": time.Now().UnixMilli(),                   "guid":     generateGUID(),                   "data":     data,           }              messageBytes, err := json.Marshal(message)           if err != nil {                   fmt.Println("Error marshaling message:", err)                   return           }              // Send the message through WebSocket if client is connected           if wsClient != nil {                   err = wsClient.WriteMessage(websocket.TextMessage, messageBytes)                   if err != nil {                           fmt.Println("Error sending message:", err)                   }           }   }      // Handle user commands   func handleCommands() {           var command string           for !shouldExit {                   fmt.Println("Enter command ('stand', 'walk', 'twist', 'sit', 'stair', 'stop', 'imu') or 'exit' to quit:")                   fmt.Scanln(&command)                      command = strings.TrimSpace(command)                   switch command {                   case "exit":                           shouldExit = true                           return                   case "stand":                           sendRequest("request_stand_mode", nil)                   case "walk":                           sendRequest("request_walk_mode", nil)                   case "twist":                           var x, y, z float64                           fmt.Println("Enter x value:")                           fmt.Scanln(&x)                           fmt.Println("Enter y value:")                           fmt.Scanln(&y)                           fmt.Println("Enter z value:")                           fmt.Scanln(&z)                           sendRequest("request_twist", map[string]interface{}{"x": x, "y": y, "z": z})                   case "sit":                           sendRequest("request_sitdown", nil)                   case "stair":                           var enable bool                           fmt.Println("Enable stair mode (true/false):")                           var input string                           fmt.Scanln(&input)                           enable = strings.ToLower(input) == "true"                           sendRequest("request_stair_mode", map[string]interface{}{"enable": enable})                   case "stop":                           sendRequest("request_emgy_stop", nil)                   case "imu":                           var enable bool                           fmt.Println("Enable IMU (true/false):")                           var input string                           fmt.Scanln(&input)                           enable = strings.ToLower(input) == "true"                           sendRequest("request_enable_imu", map[string]interface{}{"enable": enable})                   }           }   }      // WebSocket onOpen callback   func onOpen(ws *websocket.Conn) {           fmt.Println("Connected!")           wsClient = ws           go handleCommands()   }      // WebSocket onMessage callback   func onMessage(ws *websocket.Conn, message []byte) {           fmt.Println("Received message:", string(message))              if ACCID == "" {                   var msgMap map[string]interface{}                   if err := json.Unmarshal(message, &msgMap); err != nil {                           fmt.Println("Error parsing message:", err)                           return                   }                   ACCID = msgMap["accid"].(string)                   fmt.Printf("ACCID initialized from server: %s\n", ACCID)           }   }      // WebSocket onClose callback   func onClose(ws *websocket.Conn, code int, text string) {           fmt.Println("Connection closed. Code:", code, "Message:", text)   }      // Connect to the WebSocket server   func connectWebSocket() {           url := "ws://10.192.1.2:5000" // WebSocket server URI              conn, _, err := websocket.DefaultDialer.Dial(url, nil)           if err != nil {                   fmt.Println("Error connecting to WebSocket server:", err)                   return           }              onOpen(conn)              // Start receiving messages from the server           go func() {                   for {                           _, message, err := conn.ReadMessage()                           if err != nil {                                   fmt.Println("Error reading message:", err)                                   break                           }                           onMessage(conn, message)                   }           }()              // Wait until WebSocket connection is closed           select {}   }      // Main function   func main() {           defer func() {                   if wsClient != nil {                           wsClient.Close()                   }           }()              // Connect to WebSocket server           go connectWebSocket()              // Block main goroutine to allow handling commands           select {}   } |
| :--- |


<font style="color:#3370FF;">5.6.5 </font>**Java 示例实现**

| <font style="color:rgb(100, 106, 115);">Java</font>import org.java-websocket.client.WebSocketClient;   import org.java-websocket.handshake.ServerHandshake;   import org.json.JSONObject;      import java.net.URI;   import java.util.Scanner;   import java.util.UUID;      public class WebSocketExample {          // Replace this ACCID value with your robot's actual serial number (SN)       private static String ACCID = "";          // Atomic flag for graceful exit       private static volatile boolean shouldExit = false;          // WebSocket client instance       private static WebSocketClient wsClient = null;          // Generate dynamic GUID       private static String generateGuid() {           return UUID.randomUUID().toString();       }          // Send WebSocket request with title and data       private static void sendRequest(String title, JSONObject data) {           if (data == null) {               data = new JSONObject();           }              // Create message structure with necessary fields           JSONObject message = new JSONObject();           message.put("accid", ACCID);           message.put("title", title);           message.put("timestamp", System.currentTimeMillis());  // Current timestamp in milliseconds           message.put("guid", generateGuid());           message.put("data", data);              String messageStr = message.toString();              // Send the message through WebSocket if client is connected           if (wsClient != null && wsClient.isOpen()) {               wsClient.send(messageStr);           }       }          // Handle user commands       private static void handleCommands() {           Scanner scanner = new Scanner(System.in);           while (!shouldExit) {               System.out.println("Enter command ('stand', 'walk', 'twist', 'sit', 'stair', 'stop', 'imu') or 'exit' to quit:");               String command = scanner.nextLine().trim();                  switch (command) {                   case "exit":                       shouldExit = true;                       break;                   case "stand":                       sendRequest("request_stand_mode", null);                       break;                   case "walk":                       sendRequest("request_walk_mode", null);                       break;                   case "twist":                       System.out.print("Enter x value: ");                       double x = scanner.nextDouble();                       System.out.print("Enter y value: ");                       double y = scanner.nextDouble();                       System.out.print("Enter z value: ");                       double z = scanner.nextDouble();                       scanner.nextLine();  // Consume the newline                       JSONObject twistData = new JSONObject();                       twistData.put("x", x);                       twistData.put("y", y);                       twistData.put("z", z);                       sendRequest("request_twist", twistData);                       break;                   case "sit":                       sendRequest("request_sitdown", null);                       break;                   case "stair":                       System.out.print("Enable stair mode (true/false): ");                       boolean enableStair = scanner.nextLine().trim().equalsIgnoreCase("true");                       JSONObject stairData = new JSONObject();                       stairData.put("enable", enableStair);                       sendRequest("request_stair_mode", stairData);                       break;                   case "stop":                       sendRequest("request_emgy_stop", null);                       break;                   case "imu":                       System.out.print("Enable IMU (true/false): ");                       boolean enableImu = scanner.nextLine().trim().equalsIgnoreCase("true");                       JSONObject imuData = new JSONObject();                       imuData.put("enable", enableImu);                       sendRequest("request_enable_imu", imuData);                       break;                   default:                       System.out.println("Invalid command. Try again.");                       break;               }           }       }          // WebSocket onOpen callback       private static void onOpen() {           System.out.println("Connected!");           // Start handling commands in a separate thread           new Thread(WebSocketExample::handleCommands).start();       }          // WebSocket onMessage callback       private static void onMessage(String message) {           System.out.println("Received message: " + message);                      if (ACCID.isEmpty()) {               try {                   JSONObject jsonMessage = new JSONObject(message);                   ACCID = jsonMessage.getString("accid");                   System.out.printf("ACCID initialized from server: %s%n", ACCID);               } catch (Exception e) {                   System.out.println("Error parsing ACCID from message: " + e.getMessage());               }           }       }          // WebSocket onClose callback       private static void onClose(int code, String reason, boolean remote) {           System.out.println("Connection closed. Reason: " + reason);       }          public static void main(String[] args) {           // WebSocket server URI           URI serverUri = URI.create("ws://10.192.1.2:5000");              // Create WebSocket client instance           wsClient = new WebSocketClient(serverUri) {                  @Override               public void onOpen(ServerHandshake handshakedata) {                   onOpen();               }                  @Override               public void onMessage(String message) {                   onMessage(message);               }                  @Override               public void onClose(int code, String reason, boolean remote) {                   onClose(code, reason, remote);               }                  @Override               public void onError(Exception ex) {                   System.out.println("Error: " + ex.getMessage());               }           };              // Connect to the WebSocket server           wsClient.connect();           System.out.println("Press Ctrl+C to exit.");       }   } |
| :--- |


<font style="color:#3370FF;">5.6.6 </font>**Windows C++ 示例实现**

在 Windows 上，确保您已经安装并配置了 websocketpp、nlohmann/json 和 boost 库。如果使用 vcpkg，您可以运行以下命令安装依赖：

| <font style="color:rgb(100, 106, 115);">PowerShell</font>vcpkg install websocketpp boost nlohmann-json |
| :--- |


代码实现

| <font style="color:rgb(100, 106, 115);">C++</font>#include <iostream>   #include <atomic>   #include <string>   #include <thread>   #include <chrono>   #include <websocketpp/client.hpp>   #include <websocketpp/config/asio.hpp>    #include <nlohmann/json.hpp>   #include <boost/uuid/uuid.hpp>   #include <boost/uuid/uuid_generators.hpp>   #include <boost/uuid/uuid_io.hpp>      using json = nlohmann::json;   using websocketpp::client;   using websocketpp::connection_hdl;      // Replace this ACCID value with your robot's actual serial number (SN)   static std::string ACCID = "";      // WebSocket client instance   static client<websocketpp::config::asio> ws_client;      // Atomic flag for graceful exit   static std::atomic<bool> should_exit(false);      // Connection handle for sending messages   static connection_hdl current_hdl;      // Generate dynamic GUID   static std::string generate_guid() {       boost::uuids::random_generator gen;       boost::uuids::uuid u = gen();       return boost::uuids::to_string(u);   }      // Send WebSocket request with title and data   static void send_request(const std::string& title, const json& data = json::object()) {       json message;              // Adding necessary fields to the message       message["accid"] = ACCID;       message["title"] = title;       message["timestamp"] = std::chrono::duration_cast<std::chrono::milliseconds>(                                   std::chrono::system_clock::now().time_since_epoch()).count();       message["guid"] = generate_guid();       message["data"] = data;          std::string message_str = message.dump();              // Send the message through WebSocket       ws_client.send(current_hdl, message_str, websocketpp::frame::opcode::text);   }      // Handle user commands   static void handle_commands() {       while (!should_exit) {           std::string command;           std::cout << "Enter command ('stand', 'walk', 'twist', 'sit', 'stair', 'stop', 'imu') or 'exit' to quit:" << std::endl;           std::getline(std::cin, command);  // Read user input              if (command == "exit") {               should_exit = true;  // Exit flag to stop the loop               break;           } else if (command == "stand") {               send_request("request_stand_mode");  // Send stand mode request           } else if (command == "walk") {               send_request("request_walk_mode");  // Send walk mode request           } else if (command == "twist") {               float x, y, z;               std::cout << "Enter x, y, z values:" << std::endl;               std::cin >> x >> y >> z;  // Get twist values from user               send_request("request_twist", {{"x", x}, {"y", y}, {"z", z}});           } else if (command == "sit") {               send_request("request_sitdown");  // Send sit down request           } else if (command == "stair") {               bool enable;               std::cout << "Enable stair mode (true/false):" << std::endl;               std::cin >> enable;  // Get stair mode enable flag from user               send_request("request_stair_mode", {{"enable", enable}});           } else if (command == "stop") {               send_request("request_emgy_stop");  // Send emergency stop request           } else if (command == "imu") {               std::string enable;               std::cout << "Enable IMU (true/false):" << std::endl;               std::cin >> enable;  // Get IMU enable flag from user               send_request("request_enable_imu", {{"enable", enable == "true" ? true : false}});           }       }   }      // WebSocket open callback   static void on_open(connection_hdl hdl) {       std::cout << "Connected!" << std::endl;              // Save connection handle for sending messages later       current_hdl = hdl;          // Start handling commands in a separate thread       std::thread(handle_commands).detach();   }      // WebSocket message callback   static void on_message(connection_hdl hdl, client<websocketpp::config::asio>::message_ptr msg) {       // Parse JSON data from message payload       json data = json::parse(msg->get_payload());                    // Extract 'accid' field if present       if (data.contains("accid") && data["accid"].is_string() && ACCID.empty()) {           ACCID = data["accid"].get<std::string>();       }             }      // WebSocket close callback   static void on_close(connection_hdl hdl) {       std::cout << "Connection closed." << std::endl;   }      // Close WebSocket connection   static void close_connection(connection_hdl hdl) {       ws_client.close(hdl, websocketpp::close::status::normal, "Normal closure");  // Close connection normally   }      int main() {       ws_client.init_asio();  // Initialize ASIO for WebSocket client              // Set WebSocket event handlers       ws_client.set_open_handler(&on_open);  // Set open handler       ws_client.set_message_handler(&on_message);  // Set message handler       ws_client.set_close_handler(&on_close);  // Set close handler          std::string server_uri = "ws://10.192.1.2:5000";  // WebSocket server URI          websocketpp::lib::error_code ec;       client<websocketpp::config::asio>::connection_ptr con = ws_client.get_connection(server_uri, ec);  // Get connection pointer          if (ec) {              return 1;  // Exit if connection error occurs       }          connection_hdl hdl = con->get_handle();  // Get connection handle       ws_client.connect(con);  // Connect to server       std::cout << "Press Ctrl+C to exit." << std::endl;              // Run the WebSocket client loop       ws_client.run();          return 0;   } |
| :--- |


<font style="color:#3370FF;">5.6.7 </font>**Windows C# 示例实现**

| <font style="color:rgb(100, 106, 115);">C#</font>using System;   using System.Net.WebSockets;   using System.Text;   using System.Text.Json;   using System.Threading;   using System.Threading.Tasks;      class WebSocketClient   {       // Replace this ACCID value with your robot's actual serial number (SN)       private static string ACCID = "";       private static bool isAccidInitialized = false;              private static ClientWebSocket ws;          public static async Task Main(string[] args)       {           ws = new ClientWebSocket();           Uri serverUri = new Uri("ws://10.192.1.2:5000");  // WebSocket server URI              // Handle program exit to close WebSocket connection           Console.CancelKeyPress += async (sender, e) =>           {               e.Cancel = true;               await CloseConnection();               Environment.Exit(0);           };              try           {               await ws.ConnectAsync(serverUri, CancellationToken.None);               Console.WriteLine("Connected to WebSocket server!");                  _ = Task.Run(async () => await ReceiveMessages());  // Start listening for messages                  while (true)               {                   Console.WriteLine("Enter a command ('stand', 'walk', 'twist', 'sit', 'stair', 'stop', 'imu') or 'exit' to quit:");                   string command = Console.ReadLine();                      if (command == "exit")                   {                       await CloseConnection();                       break;                   }                   else if (command == "stand")                   {                       await RequestStandMode();                   }                   else if (command == "walk")                   {                       await RequestWalkMode();                   }                   else if (command == "twist")                   {                       Console.WriteLine("Enter x, y, z values:");                       float x = float.Parse(Console.ReadLine());                       float y = float.Parse(Console.ReadLine());                       float z = float.Parse(Console.ReadLine());                       await RequestTwist(x, y, z);                   }                   else if (command == "sit")                   {                       await RequestSitDown();                   }                   else if (command == "stair")                   {                       Console.WriteLine("Enter stair mode enable (true/false):");                       bool enable = bool.Parse(Console.ReadLine());                       await RequestStairMode(enable);                   }                   else if (command == "stop")                   {                       await RequestEmgyStop();                   }                   else if (command == "imu")                   {                       Console.WriteLine("Enable IMU (true/false):");                       bool enable = bool.Parse(Console.ReadLine());                       await RequestEnableImu(enable);                   }               }           }           catch (Exception ex)           {               Console.WriteLine($"Exception: {ex.Message}");           }           finally           {               ws?.Dispose();           }       }          private static async Task RequestStandMode()       {           // Sends a stand mode request           var request = new           {               accid = ACCID,               title = "request_stand_mode",               timestamp = DateTimeOffset.UtcNow.ToUnixTimeMilliseconds(),               guid = Guid.NewGuid().ToString("N"),               data = new { }           };           await SendMessage(JsonSerializer.Serialize(request));       }          private static async Task RequestWalkMode()       {           // Sends a walk mode request           var request = new           {               accid = ACCID,               title = "request_walk_mode",               timestamp = DateTimeOffset.UtcNow.ToUnixTimeMilliseconds(),               guid = Guid.NewGuid().ToString("N"),               data = new { }           };           await SendMessage(JsonSerializer.Serialize(request));       }          private static async Task RequestTwist(float x, float y, float z)       {           // Sends a twist control request with x, y, z values           var request = new           {               accid = ACCID,               title = "request_twist",               timestamp = DateTimeOffset.UtcNow.ToUnixTimeMilliseconds(),               guid = Guid.NewGuid().ToString("N"),               data = new               {                   x = x,                   y = y,                   z = z               }           };           await SendMessage(JsonSerializer.Serialize(request));       }          private static async Task RequestSitDown()       {           // Sends a sit down request           var request = new           {               accid = ACCID,               title = "request_sitdown",               timestamp = DateTimeOffset.UtcNow.ToUnixTimeMilliseconds(),               guid = Guid.NewGuid().ToString("N"),               data = new { }           };           await SendMessage(JsonSerializer.Serialize(request));       }          private static async Task RequestStairMode(bool enable)       {           // Sends a stair mode request, enabling or disabling           var request = new           {               accid = ACCID,               title = "request_stair_mode",               timestamp = DateTimeOffset.UtcNow.ToUnixTimeMilliseconds(),               guid = Guid.NewGuid().ToString("N"),               data = new               {                   enable = enable               }           };           await SendMessage(JsonSerializer.Serialize(request));       }          private static async Task RequestEmgyStop()       {           // Sends an emergency stop request           var request = new           {               accid = ACCID,               title = "request_emgy_stop",               timestamp = DateTimeOffset.UtcNow.ToUnixTimeMilliseconds(),               guid = Guid.NewGuid().ToString("N"),               data = new { }           };           await SendMessage(JsonSerializer.Serialize(request));       }          private static async Task RequestEnableImu(bool enable)       {           // Sends a request to enable/disable IMU data streaming           var request = new           {               accid = ACCID,               title = "request_enable_imu",               timestamp = DateTimeOffset.UtcNow.ToUnixTimeMilliseconds(),               guid = Guid.NewGuid().ToString("N"),               data = new               {                   enable = enable               }           };           await SendMessage(JsonSerializer.Serialize(request));       }          private static async Task SendMessage(string message)       {           // Sends a message over WebSocket           byte[] buffer = Encoding.UTF8.GetBytes(message);           await ws.SendAsync(new ArraySegment<byte>(buffer), WebSocketMessageType.Text, true, CancellationToken.None);           Console.WriteLine($"Sent: {message}");       }          private static async Task ReceiveMessages()       {           // Receives messages from WebSocket server           byte[] buffer = new byte[1024];              while (ws.State == WebSocketState.Open)           {               WebSocketReceiveResult result = await ws.ReceiveAsync(new ArraySegment<byte>(buffer), CancellationToken.None);                  if (result.MessageType == WebSocketMessageType.Close)               {                   await ws.CloseAsync(WebSocketCloseStatus.NormalClosure, "Server closed", CancellationToken.None);                   Console.WriteLine("WebSocket server closed connection.");                   break;               }               else               {                   string message = Encoding.UTF8.GetString(buffer, 0, result.Count);                   Console.WriteLine($"Received: {message}");                                      if (!isAccidInitialized)                   {                       try                       {                           var msgJson = JsonDocument.Parse(message).RootElement;                           ACCID = msgJson.GetProperty("accid").GetString();                           isAccidInitialized = true;                           Console.WriteLine($"ACCID initialized from server: {ACCID}");                       }                       catch (Exception ex)                       {                           Console.WriteLine($"Error parsing ACCID: {ex.Message}");                       }                   }               }           }       }          private static async Task CloseConnection()       {           // Closes the WebSocket connection gracefully           if (ws.State == WebSocketState.Open)           {               await ws.CloseAsync(WebSocketCloseStatus.NormalClosure, "Client closing", CancellationToken.None);               Console.WriteLine("WebSocket connection closed.");           }       }   } |
| :--- |


<font style="color:#3370FF;">6. </font>**RL 环境搭建**

<font style="color:#3370FF;">6.1 </font>**推荐硬件配置**

| **硬件组件 ** | **推荐规格 ** | **  说明** |
| :--- | --- | :--- |
| **GPU** | GeForce RTX 3080 (12 GB) 或更高 | 12  GB GDDR6 显存或更高，适合深度学习和强化学习任务。<br/>[<font style="color:rgb(51, 112, 255);">GeForce RTX 30系列</font>](https://www.nvidia.com/en-us/geforce/graphics-cards/30-series/)：<br/>![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738051888-988ec4ad-1a0f-4d3a-93a6-cc42de3b9cd3.png)<br/>[<font style="color:rgb(51, 112, 255);">GeForce RTX 40系列：</font>](https://www.nvidia.com/en-us/geforce/graphics-cards/40-series/)<br/>![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738052087-369ec9d5-65da-45fc-bf5c-36dddd683738.png) |
| **CPU** | 至少六核处理器 | 推荐型号：<br/>Intel: Core i7-10700K 或更高版本<br/>AMD: Ryzen 5 5600X 或更高版本 |
| **内存** | 16 GB 至 32 GB | 系统内存：16 GB 至 32 GB。较大的内存可以支持更复杂的仿真和训练任务，同时提高系统的响应能力。 |
| **存储** | 512 GB 至 1 TB NVMe SSD   | SSD 提高读写速度，缩短加载时间和提升系统性能。 |
| **操作系统** | Ubuntu 20.04 LTS | 确保与GPU驱动程序兼容。 |
| **GPU 驱动程序  ** | / | 选择推荐的驱动版本进行安装。 |
| **Python**                 | Python 3.8 或更高版本 | 安装必需的 Python 包和库（如 PyTorch）。 |
| **网络环境** | 能正常访问Github | 环境搭建过程需要通过 GitHub 下载模型训练代码。 |


<font style="color:#3370FF;">6.2 </font>**一键安装RL环境**

一键安装RL环境

| <font style="color:rgb(100, 106, 115);">Bash</font>mkdir -p ~/limx_rl && cd ~/limx_rl && \   sudo apt update && sudo apt install -y git && \   if [ ! -d "pointfoot-legged-gym" ]; then \     git clone [https://github.com/limxdynamics/pointfoot-legged-gym.git](https://github.com/limxdynamics/pointfoot-legged-gym.git) pointfoot-legged-gym; \   fi && \   cd pointfoot-legged-gym && bash install.sh && source ~/.bashrc |
| :--- |


验证 NVIDIA 驱动是否安装成功

| <font style="color:rgb(100, 106, 115);">YAML</font>nvidia-smi |
| :--- |


如果显示了 NVIDIA 驱动版本和 GPU 信息，说明驱动已成功安装。

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738052263-da3b0325-ed80-4fd4-8b64-d5a70f119214.png)

验证Isaac Gym是否安装成功

运行Isaac Gym 的示例，它可以帮助确认 Isaac Gym 是否正确安装并配置。

| <font style="color:rgb(100, 106, 115);">Bash</font>conda activate pointfoot_legged_gym   cd ~/limx_rl/isaacgym/python/examples   python 1080_balls_of_solitude.py |
| :--- |


![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738052538-6dfe77b5-a81d-496a-9bd3-c64d6392c413.png)

验证pointfoot-legged-gym是否安装成功

当您安装完RL环境后，在 ~/limx_rl目录下包含如下所示的内容：

| <font style="color:rgb(100, 106, 115);">Bash</font>cd ~/limx_rl   .   ├── isaacgym   ├── pointfoot-legged-gym   └── rsl_rl |
| :--- |


<font style="color:#3370FF;">6.3 </font>**分步安装RL环境**

<font style="color:#3370FF;">6.3.1 </font>**安装显卡驱动**

检查当前安装的 NVIDIA 驱动

如果系统中已经安装了 NVIDIA 驱动，可以通过以下命令检查：

| <font style="color:rgb(100, 106, 115);">Bash</font>nvidia-smi |
| :--- |


如果显示了 NVIDIA 驱动版本和 GPU 信息，说明驱动已经安装。如果没有安装，请继续以下步骤。

添加官方的 NVIDIA 驱动 PPA（Personal Package Archive）：

| <font style="color:rgb(100, 106, 115);">Bash</font>sudo add-apt-repository ppa:graphics-drivers/ppa   sudo apt update |
| :--- |


使用以下命令列出可用的 NVIDIA 驱动版本：

| <font style="color:rgb(100, 106, 115);">Bash</font>ubuntu-drivers devices |
| :--- |


安装 NVIDIA 驱动

选择推荐的驱动版本进行安装，例如：

| <font style="color:rgb(100, 106, 115);">Bash</font>sudo apt install nvidia-driver-XXX |
| :--- |


将 XXX 替换为推荐的驱动版本号，例如 nvidia-driver-535。

禁用 Nouveau 驱动（如果必要）：某些情况下，您可能需要禁用默认的 Nouveau 驱动：

| <font style="color:rgb(100, 106, 115);">Bash</font>sudo bash -c "echo blacklist nouveau > /etc/modprobe.d/blacklist-nouveau.conf"   sudo bash -c "echo options nouveau modeset=0 >> /etc/modprobe.d/blacklist-nouveau.conf"   sudo update-initramfs -u |
| :--- |


重启系统：完成驱动安装后，重启系统使驱动生效：

| <font style="color:rgb(100, 106, 115);">Bash</font>sudo reboot |
| :--- |


验证驱动安装：重启后，使用以下命令验证 NVIDIA 驱动是否安装成功：

| <font style="color:rgb(100, 106, 115);">Bash</font>nvidia-smi |
| :--- |


如果显示了 NVIDIA 驱动版本和 GPU 信息，说明驱动已成功安装。

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738052840-24f10dc4-1e89-4042-9b1e-71ca47349234.png)

<font style="color:#3370FF;">6.3.2 </font>**Conda 环境配置**

Conda 是一个跨平台的开源包和环境管理系统，它能够快速创建虚拟环境，并在其中进行包的安装、运行和更新操作，从而有效解决不同软件包对 Python 和依赖库版本的要求问题。请按照下列步骤操作下载并安装Conda：

下载 Anaconda 安装脚本：

使用 wget 下载 Anaconda 的安装脚本（请根据需要选择适合的版本）：

| <font style="color:rgb(100, 106, 115);">Bash</font>cd && wget  [https://repo.anaconda.com/archive/Anaconda3-2024.06-1-Linux-x86_64.sh](https://repo.anaconda.com/archive/Anaconda3-2024.06-1-Linux-x86_64.sh) |
| :--- |


运行安装脚本：

执行下载的安装脚本：

| <font style="color:rgb(100, 106, 115);">Bash</font>bash Anaconda3-2024.06-1-Linux-x86_64.sh |
| :--- |


安装过程中：

按 Enter 键查看许可协议。

输入 yes 接受许可协议。

选择默认安装路径（通常为 ~/anaconda3），或根据需要自定义路径。

当脚本询问是否初始化 Conda 时，选择 yes 以自动配置环境。

初始化 Conda（如果安装过程中未自动配置）：

| <font style="color:rgb(100, 106, 115);">Bash</font>~/anaconda3/bin/conda init |
| :--- |


使 .bashrc 配置立即生效：

| <font style="color:rgb(100, 106, 115);">Bash</font>source ~/.bashrc |
| :--- |


配置 Conda 环境用于 RL 训练

创建一个专门用于 RL 训练的 Conda 环境（例如，pointfoot_legged_gym），并指定 Python 版本：

| <font style="color:rgb(100, 106, 115);">Bash</font>conda create --name pointfoot_legged_gym python=3.8 |
| :--- |


激活刚刚创建的环境：

| <font style="color:rgb(100, 106, 115);">Bash</font>conda activate pointfoot_legged_gym |
| :--- |


请按照以下步骤安装 PyTorch 库：

访问 y[<font style="color:#3370FF;">PyTorch安装页面</font>](https://pytorch.org/get-started/locally/)。

选择以下选项：

操作系统：Linux

包管理器：Conda

语言：Python

CUDA 版本：12.1

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738053005-c6ef6a45-9886-486c-8319-d9caeddc29e5.png)

生成安装命令并运行以下命令进行安装：

| <font style="color:rgb(100, 106, 115);">Bash</font>conda install pytorch torchvision torchaudio pytorch-cuda=12.1 -c pytorch -c nvidia |
| :--- |


<font style="color:#3370FF;">6.3.3 </font>**安装Isaac Gym**

<font style="color:#3370FF;">6.3.3.1 </font>**下载Isaac Gym**

**下载页面**：前往 [<font style="color:#3370FF;">Isaac Gym 下载页面</font>](https://developer.nvidia.com/isaac-gym/download)。

**接受条款和条件**：您可能需要接受一些条款和条件。

**下载**：选择合适的版本下载 Isaac Gym 的安装包。

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738053185-0bc23170-82af-4f5b-a78b-66ff87cb5b72.png)

<font style="color:#3370FF;">6.3.3.2 </font>**安装 Isaac Gym**

安装setuptools：

setuptools 是一个用于 Python 包管理和分发的库，它帮助打包 Python 项目，使其易于安装和分发，并处理项目的依赖项。通过指定版本 59.5.0，可以确保与其他依赖项的兼容性和稳定性。

| <font style="color:rgb(100, 106, 115);">Bash</font>pip install setuptools==59.5.0 |
| :--- |


安装TensorBoard

TensorBoard 是一个用于可视化 TensorFlow 图表、指标和其他相关数据的工具。为了确保兼容性和稳定性，我们指定安装版本为 2.12.0。

| <font style="color:rgb(100, 106, 115);">Bash</font>pip install tensorboard==2.12.0 |
| :--- |


激活pointfoot_legged_gym的conda环境

| <font style="color:rgb(100, 106, 115);">Bash</font>conda activate pointfoot_legged_gym |
| :--- |


创建一个用于存放训练代码的路径（如：limx_rl）

| <font style="color:rgb(100, 106, 115);">Bash</font>mkdir -p ~/limx_rl |
| :--- |


解压文件：将上一步下载的安装压缩文件解压到 limx_rl 目录。注意：请将 /path/to/IsaacGym_Preview_4_Package.tar.gz 替换为安装包的实际路径。

| <font style="color:rgb(100, 106, 115);">Bash</font>tar -xzvf /path/to/IsaacGym_Preview_4_Package.tar.gz -C ~/limx_rl/ |
| :--- |


安装 Isaac Gym并配置：

| <font style="color:rgb(100, 106, 115);">Bash</font>cd ~/limx_rl/isaacgym/python   pip install -e . |
| :--- |


解决NumPy兼容性问题：

在较新的版本的 NumPy 中，np.float 被弃用，使用时会出现警告或错误。直接使用 Python 内置的 float 类型可以避免这些问题。

| <font style="color:rgb(100, 106, 115);">Bash</font>cd ~/limx_rl/isaacgym/python   sed -i 's/np.float/float/' isaacgym/torch_utils.py |
| :--- |


验证安装

运行Isaac Gym 的示例，它可以帮助确认 Isaac Gym 是否正确安装并配置。

| <font style="color:rgb(100, 106, 115);">Bash</font>cd ~/limx_rl/isaacgym/python/examples   python 1080_balls_of_solitude.py |
| :--- |


![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738053541-c806b54f-c390-4347-8ed4-a2a7677e9065.png)

<font style="color:#3370FF;">6.3.4 </font>**安装pointfoot-legged-gym**

激活pointfoot_legged_gym的conda环境

| <font style="color:rgb(100, 106, 115);">Bash</font>conda activate pointfoot_legged_gym |
| :--- |


安装pointfoot-legged-gym

| <font style="color:rgb(100, 106, 115);">Bash</font>cd ~/limx_rl   git clone [https://github.com/limxdynamics/pointfoot-legged-gym.git](https://github.com/limxdynamics/pointfoot-legged-gym.git) pointfoot-legged-gym   cd ~/limx_rl/pointfoot-legged-gym && pip install -e .  |
| :--- |


验证安装

当您完成前面所有步骤的操作后，在 ~/limx_rl目录下包含如下所示的内容：

| <font style="color:rgb(100, 106, 115);">Bash</font>cd ~/limx_rl   .   ├── isaacgym   └── pointfoot-legged-gym |
| :--- |


<font style="color:#3370FF;">6.4 </font>** 部署RL Docker 容器环境**

<font style="color:#3370FF;">6.4.1 </font>**安装Docker服务**

| <font style="color:rgb(100, 106, 115);">Bash</font>**#### 更新包索引并安装依赖**   # Add Docker's official GPG key:   sudo apt-get update   sudo apt-get install ca-certificates curl   sudo mkdir -p /etc/apt/keyrings   curl -fsSL [https://download.docker.com/linux/ubuntu/gpg](https://download.docker.com/linux/ubuntu/gpg) | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg   sudo chmod a+r /etc/apt/keyrings/docker.gpg      # Add the repository to Apt sources:   echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] [https://download.docker.com/linux/ubuntu](https://download.docker.com/linux/ubuntu) $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null   sudo apt-get update      **#### 安装 Docker 引擎**   sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin |
| :--- |


<font style="color:#3370FF;">6.4.2 </font>** 安装 NVIDIA Docker 工具**

| <font style="color:rgb(100, 106, 115);">Bash</font>**#### 设置 NVIDIA Docker 仓库###**   distribution=$(. /etc/os-release;echo $ID$VERSION_ID)   curl -s -L [https://nvidia.github.io/nvidia-docker/gpgkey](https://nvidia.github.io/nvidia-docker/gpgkey) | sudo apt-key add -   curl -s -L [https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list](https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list) | sudo tee /etc/apt/sources.list.d/nvidia-docker.list      **#### 更新包索引并安装 NVIDIA Docker 工具**   sudo apt-get update   sudo apt-get install -y nvidia-docker2      **#### 重启 Docker 服务**   sudo systemctl restart docker |
| :--- |


<font style="color:#3370FF;">6.4.3 </font>** 验证Docker环境安装是否正常**

| <font style="color:rgb(100, 106, 115);">Bash</font>#### 运行带有 NVIDIA 支持的测试容器 查看是否能够在容器中成功执行 nvidia-smi命令   sudo docker run -it --rm --gpus all nvidia/cuda:12.1.1-base-ubuntu20.04 nvidia-smi |
| :--- |


输出为如下信息 表明Docker环境安装正常：

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738053825-3f099836-282e-4416-b9c0-dd837e6475eb.png)

**拉取RL Docker环境基础镜像**

如下RL 环境基础镜像预安装了conda  issacgym pointfoot-legged-gym 等环境，拉取镜像后即可直接使用无需再配置相关环境

| <font style="color:rgb(100, 106, 115);">Bash</font>sudo docker pull crpi-yznuflhnzo12tr44.cn-shanghai.personal.cr.aliyuncs.com/limx-rl/base:v10 |
| :--- |


<font style="color:#3370FF;">7. </font>**RL 模型训练**

<font style="color:#3370FF;">7.1 </font>**启动训练**

打开终端Ctrl+Alt+T

设置机器人型号

请参考“查看/设置机器人型号”章节，查看您的机器人型号。如果尚未设置，请按照以下步骤进行设置。

通过 Shell 命令 tree -L 1 resources/robots/pointfoot 列出可用的机器人类型：

| <font style="color:rgb(100, 106, 115);">Plain Text</font>limx@limx:~$ cd ~/limx_rl/pointfoot-legged-gym   limx@limx:~$ tree -L 1 resources/robots/pointfoot   resources/robots/pointfoot   ├── PF_P441A   ├── PF_P441B   ├── PF_P441C   ├── PF_P441C2   ├── PF_TRON1A   ├── SF_TRON1A   └── WF_TRON1A |
| :--- |


以 PF_TRON1A（请根据实际机器人类型进行替换）为例，设置机器人型号类型：

| <font style="color:rgb(100, 106, 115);">Plain Text</font>echo 'export ROBOT_TYPE=PF_TRON1A' >> ~/.bashrc && source ~/.bashrc |
| :--- |


激活pointfoot_legged_gym的conda环境

| <font style="color:rgb(100, 106, 115);">Bash</font>conda activate pointfoot_legged_gym |
| :--- |


进入训练代码所在路径，开始训练

     参数选项：

--task=pointfoot_flat：指定任务或环境类型为 pointfoot_flat。

--headless：以无头模式运行，即不显示任何图形界面。通常用于在没有显示器的服务器上运行，或者在需要高效计算而不需要图形渲染的情况下使用。

--num_envs 8192：指定要创建的环境数量为 8192。

**命令 1**: 无头模式运行

| <font style="color:rgb(100, 106, 115);">Bash</font>cd ~/limx_rl/pointfoot-legged-gym   python legged_gym/scripts/train.py --task=pointfoot_flat --headless |
| :--- |


**命令 2**: 图形模式下运行

| <font style="color:rgb(100, 106, 115);">Bash</font>cd ~/limx_rl/pointfoot-legged-gym   python legged_gym/scripts/train.py --task=pointfoot_flat |
| :--- |


![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738054095-ca6549af-1c0e-4e3d-b1b9-d0bf0df0b602.png)

<font style="color:#3370FF;">7.2 </font>**继续训练**

如果您的训练中途终止，可以指定一个检查点文件的位置，接着继续训练。

首先，您需要激活pointfoot_legged_gym的conda环境

| <font style="color:rgb(100, 106, 115);">Bash</font>conda activate pointfoot_legged_gym |
| :--- |


**命令 1**: 无头模式下继续训练

| <font style="color:rgb(100, 106, 115);">Bash</font>cd ~/limx_rl/pointfoot-legged-gym   python legged_gym/scripts/train.py --task=pointfoot_flat --resume --headless --load_run Dec23_17-38-22_ --checkpoint 200 |
| :--- |


**命令 2**: 图形模式下继续训练

| <font style="color:rgb(100, 106, 115);">Bash</font>cd ~/limx_rl/pointfoot-legged-gym   python legged_gym/scripts/train.py --task=pointfoot_flat --resume --load_run Dec23_17-38-22_ --checkpoint 200 |
| :--- |


**参数选项说明：**

--load_run:

说明：指定要加载的训练运行的标识符（例如，训练运行的名字或ID）。这个标识符通常与训练过程相关联，用于从 logs 目录中找到相应的运行记录或配置。

获取方式：

查看 logs 目录：进入 logs 目录查看子目录或文件，这些子目录或文件通常会以训练运行的标识符命名。

示例路径：

| <font style="color:rgb(100, 106, 115);">Bash</font>ls -l ~/limx_rl/pointfoot-legged-gym/logs/pointfoot_flat |
| :--- |


您会看到以机器人类型命名的目录，训练结果会按照机器人类型分别存储于对应目录下，以PF_TRON1A为例，进入后可以看到类似于 Apr16_16-51-06_ 这样的目录，Apr16_16-51-06_ 就是 --load_run 参数的值。

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738054352-323c5d8d-3f23-417d-b5a6-821dfb66a8ba.png)

--checkpoint:

说明：指定要加载的检查点文件。检查点文件保存了模型的中间状态，可以用于恢复训练或进行推断。

获取方式：

查看 logs 目录：在 logs 目录下的相应 --load_run 目录中，通常会有保存检查点的文件。这些文件通常以 .pt 或类似扩展名存在，文件名可能包含训练的轮次或时间戳。

示例路径：

| <font style="color:rgb(100, 106, 115);">Bash</font>ls -l ~/limx_rl/pointfoot-legged-gym/logs/pointfoot_flat/PF_TRON1A/Apr16_16-51-06_ |
| :--- |


会看到类似于model_200.pt 的文件，则200就是 --checkpoint 参数的值。

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738054528-a708e43d-e49c-43eb-8516-a2445810e4fb.png)

<font style="color:#3370FF;">7.3 </font>**查看训练情况**

打开终端Ctrl+Alt+T

激活pointfoot_legged_gym的conda环境

| <font style="color:rgb(100, 106, 115);">Bash</font>conda activate pointfoot_legged_gym |
| :--- |


启动 tensorboard（以PF_TRON1A为例）

| <font style="color:rgb(100, 106, 115);">Bash</font>cd ~/limx_rl/pointfoot-legged-gym   tensorboard --logdir=logs/pointfoot_flat/PF_TRON1A |
| :--- |


查看训练情况

在浏览器地址栏输入 [http://127.0.0.1:6006](http://127.0.0.1:6006)，查看训练情况。

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738054748-375f175f-4ac9-46b6-889a-cf6078357c09.png)

<font style="color:#3370FF;">7.4 </font>**导出训练结果**

打开终端Ctrl+Alt+T

激活pointfoot_legged_gym的conda环境

| <font style="color:rgb(100, 106, 115);">Bash</font>conda activate pointfoot_legged_gym |
| :--- |


安装ONNX

若您尚未安装 ONNX 库，请安装它。ONNX 允许在不同的深度学习框架（例如 PyTorch、TensorFlow、Caffe2、MXNet 等）之间轻松转换模型。

| <font style="color:rgb(100, 106, 115);">Bash</font>pip install onnx |
| :--- |


完成训练后查看训练结果

默认读取最新的run和checkpoint，如需选择特定的run和checkpoint，请输入--load_run和--checkpoint参数。

--load_run:

说明：指定要加载的训练运行的标识符（例如，训练运行的名字或ID）。这个标识符通常与训练过程相关联，用于从 logs 目录中找到相应的运行记录或配置。

获取方式：

查看 logs 目录：进入 logs 目录查看子目录或文件，这些子目录或文件通常会以训练运行的标识符命名。

示例路径：

| <font style="color:rgb(100, 106, 115);">Bash</font>ls -l ~/limx_rl/pointfoot-legged-gym/logs/pointfoot_flat |
| :--- |


您会看到以机器人类型命名的目录，训练结果会按照机器人类型分别存储于对应目录下，以PF_TRON1A为例，进入后可以看到类似于 Apr16_16-51-06_ 这样的目录，Apr16_16-51-06_ 就是 --load_run 参数的值。

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738054984-c7920703-21c7-4955-95bb-aaa068fa5549.png)

--checkpoint:

说明：指定要加载的检查点文件。检查点文件保存了模型的中间状态，可以用于恢复训练或进行推断。

获取方式：

查看 logs 目录：在 logs 目录下的相应 --load_run 目录中，通常会有保存检查点的文件。这些文件通常以 .pt 或类似扩展名存在，文件名可能包含训练的轮次或时间戳。

示例路径：

| <font style="color:rgb(100, 106, 115);">Bash</font>ls -l ~/limx_rl/pointfoot-legged-gym/logs/pointfoot_flat/PF_TRON1A/Apr16_16-51-06_ |
| :--- |


会看到类似于model_200.pt 的文件，则200就是 --checkpoint 参数的值。

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738055148-d15f9181-fb79-4c8c-87de-e2ac2f59173f.png)

使用示例

| <font style="color:rgb(100, 106, 115);">Bash</font>cd ~/limx_rl/pointfoot-legged-gym   python legged_gym/scripts/play.py --task=pointfoot_flat --load_run Apr16_16-51-06_ --checkpoint 200 |
| :--- |


![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738055440-1c18afb1-3365-4fa7-b599-17f87fa20779.png)

导出模型的ONNX格式文件

默认读取最新的run和checkpoint，如需选择特定的run和checkpoint，请输入--load_run和--checkpoint参数

**参数选项说明：**

--load_run:

说明：指定要加载的训练运行的标识符（例如，训练运行的名字或ID）。这个标识符通常与训练过程相关联，用于从 logs 目录中找到相应的运行记录或配置。

获取方式：

查看 logs 目录：进入 logs 目录查看子目录或文件，这些子目录或文件通常会以训练运行的标识符命名。

示例路径：

| <font style="color:rgb(100, 106, 115);">Bash</font>ls -l ~/limx_rl/pointfoot-legged-gym/logs/pointfoot_flat |
| :--- |


您会看到以机器人类型命名的目录，训练结果会按照机器人类型分别存储于对应目录下，以PF_TRON1A为例，进入后可以看到类似于 Apr16_16-51-06_ 这样的目录，Apr16_16-51-06_ 就是 --load_run 参数的值。

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738055688-9167455f-533c-46f2-b00f-2633dcce27cb.png)

--checkpoint:

说明：指定要加载的检查点文件。检查点文件保存了模型的中间状态，可以用于恢复训练或进行推断。

获取方式：

查看 logs 目录：在 logs 目录下的相应 --load_run 目录中，通常会有保存检查点的文件。这些文件通常以 .pt 或类似扩展名存在，文件名可能包含训练的轮次或时间戳。

示例路径：

| <font style="color:rgb(100, 106, 115);">Bash</font>ls -l ~/limx_rl/pointfoot-legged-gym/logs/pointfoot_flat/PF_TRON1A/Apr16_16-51-06_ |
| :--- |


会看到类似于model_200.pt 的文件，则200就是 --checkpoint 参数的值。

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738055867-b8bddedc-2ffe-4443-aaad-349aadfa72d7.png)

使用示例

| <font style="color:rgb(100, 106, 115);">Bash</font>cd ~/limx_rl/pointfoot-legged-gym   python legged_gym/scripts/export_policy_as_onnx.py --task=pointfoot_flat --load_run Apr16_16-51-06_ --checkpoint 200 |
| :--- |


以PF_TRON1A为例，转换后的ONNX文件保存在目录：

~/limx_rl/pointfoot-legged-gym/logs/pointfoot_flat/PF_TRON1A/exported/policies

| <font style="color:rgb(100, 106, 115);">Go</font>ls -l ~/limx_rl/pointfoot-legged-gym/logs/pointfoot_flat/PF_TRON1A/exported/policies |
| :--- |


![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738056071-6e12bb77-4a7e-415e-bfe2-39ba642c6233.png)

<font style="color:#3370FF;">7.5 </font>**Docker环境下启动训练**

打开终端Ctrl+Alt+T

拉取RL Docker容器环境基础镜像

| <font style="color:rgb(100, 106, 115);">Bash</font>sudo docker pull crpi-yznuflhnzo12tr44.cn-shanghai.personal.cr.aliyuncs.com/limx-rl/base:v10 |
| :--- |


运行RL 容器环境

| <font style="color:rgb(100, 106, 115);">Bash</font>sudo docker run -it --gpus all --rm crpi-yznuflhnzo12tr44.cn-shanghai.personal.cr.aliyuncs.com/limx-rl/base:v10 |
| :--- |


设置机器人型号：请参考“查看/设置机器人型号”章节，查看您的机器人型号。如果尚未设置，请按照以下步骤进行设置。

| <font style="color:rgb(100, 106, 115);">Plain Text</font>$ cd limx_rl/pointfoot-legged-gym   $ tree -L 1 resources/robots/pointfoot/   resources/robots/pointfoot/   ├── PF_P441A   ├── PF_P441B   ├── PF_P441C   ├── PF_P441C2   ├── PF_TRON1A   ├── SF_TRON1A   └── WF_TRON1A |
| :--- |


以 PF_TRON1A（请根据实际机器人类型进行替换）为例，设置机器人型号类型：

| <font style="color:rgb(100, 106, 115);">Plain Text</font>export ROBOT_TYPE=PF_TRON1A |
| :--- |


**        备注：通过在容器中执行export命令设置环境变量 只是在当前容器环境中生效 重新启动容器后不再生效**

**       可在通过 -e 参数 带上要执行的环境变量信息 使得每次运行RL 容器环境时 环境变量自动生效**

| <font style="color:rgb(100, 106, 115);">Bash</font>sudo docker run -it --gpus all --rm -e ROBOT_TYPE=PF_TRON1A crpi-yznuflhnzo12tr44.cn-shanghai.personal.cr.aliyuncs.com/limx-rl/base:v10 |
| :--- |


进入训练代码所在路径，开始训练

     参数选项：

--task=pointfoot_flat：指定任务或环境类型为 pointfoot_flat。

--headless：以无头模式运行，即不显示任何图形界面。通常用于在没有显示器的服务器上运行，或者在需要高效计算而不需要图形渲染的情况下使用。

--num_envs 1024：指定要创建的环境数量为 1024。

**命令 1**: 无头模式运行

| <font style="color:rgb(100, 106, 115);">Bash</font>cd limx_rl/pointfoot-legged-gym   python legged_gym/scripts/train.py --task=pointfoot_flat --headless |
| :--- |


<font style="color:#3370FF;">8. </font>**RL 训练结果部署**

<font style="color:#3370FF;">8.1 </font>**基于Python 部署**

Python运动控制算法开发接口让不熟悉C++和ROS的开发者也能轻松使用Python进行算法开发。Python语言易学、语法简洁，并拥有丰富的第三方库，使开发者能够快速上手并高效实现算法。借助Python接口，开发者可利用其动态特性进行快速原型设计和实验验证，加速算法迭代与优化。同时，Python的跨平台性和强大生态系统，使运动算法能广泛应用于不同平台和环境。

此外，Python的灵活性大大简化了强化学习（RL）模型的快速部署。开发者可以轻松将RL模型集成到仿真和真实硬件中，快速验证和优化算法性能。

<font style="color:#3370FF;">8.1.1 </font>**部署环境配置**

配置 Conda 环境用于Python部署

创建一个专门用于Python部署运行的 Conda 环境（例如，pointfoot_deploy），并指定 Python 版本：

| <font style="color:rgb(100, 106, 115);">Bash</font>conda create --name pointfoot_deploy python=3.8 |
| :--- |


激活刚刚创建的环境：

| <font style="color:rgb(100, 106, 115);">Bash</font>conda activate pointfoot_deploy |
| :--- |


<font style="color:#3370FF;">8.1.2 </font>**创建工作空间**

可以按照以下步骤，创建一个RL部署开发工作空间：

打开一个Bash终端。

创建一个新目录来存放工作空间。例如，可以在用户的主目录下创建一个名为“limx_ws”的目录：

| <font style="color:rgb(100, 106, 115);">Bash</font>mkdir -p ~/limx_ws |
| :--- |


下载 MuJoCo 仿真器

| <font style="color:rgb(100, 106, 115);">Bash</font>cd ~/limx_ws   git clone --recurse [https://github.com/limxdynamics/tron1-mujoco-sim.git](https://github.com/limxdynamics/tron1-mujoco-sim.git) |
| :--- |


下载部署实现

| <font style="color:rgb(100, 106, 115);">Bash</font>cd ~/limx_ws   git clone --recurse [https://github.com/limxdynamics/tron1-rl-deploy-python.git](https://github.com/limxdynamics/tron1-rl-deploy-python.git) |
| :--- |


安装运动控制开发库（如果尚未安装）：

Linux x86_64 环境

| <font style="color:rgb(100, 106, 115);">Bash</font># 激活pointfoot_deploy的conda环境   conda activate pointfoot_deploy      # 安装limxsdk   pip install tron1-rl-deploy-python/limxsdk-lowlevel/python3/amd64/limxsdk-*-py3-none-any.whl     |
| :--- |


Linux aarch64 环境

| <font style="color:rgb(100, 106, 115);">Bash</font># 激活pointfoot_deploy的conda环境   conda activate pointfoot_deploy      # 安装limxsdk   pip install tron1-rl-deploy-python/limxsdk-lowlevel/python3/aarch64/limxsdk-*-py3-none-any.whl     |
| :--- |


设置机器人型号：请参考“查看/设置机器人型号”章节，查看您的机器人型号。如果尚未设置，请按照以下步骤进行设置。

通过 Shell 命令 tree -L 1 tron1-mujoco-sim/robot-description/pointfoot 列出可用的机器人类型：

| <font style="color:rgb(100, 106, 115);">Plain Text</font>limx@limx:~$ tree -L 1 tron1-mujoco-sim/robot-description/pointfoot   tron1-mujoco-sim/robot-description/pointfoot   ├── PF_P441A   ├── PF_P441B   ├── PF_P441C   ├── PF_P441C2   ├── PF_TRON1A   ├── SF_TRON1A   └── WF_TRON1A |
| :--- |


以 PF_TRON1A（请根据实际机器人类型进行替换）为例，设置机器人型号类型：

| <font style="color:rgb(100, 106, 115);">Plain Text</font>echo 'export ROBOT_TYPE=PF_TRON1A' >> ~/.bashrc && source ~/.bashrc |
| :--- |


<font style="color:#3370FF;">8.1.3 </font>**更新RL训练模型**

在您的工作空间中，以 PF_TRON1A机器人类型为例，RL模型和配置文件所在路径为：~/limx_ws/tron1-rl-deploy-python/controllers/model/PF_TRON1A，如下所示。请根据您的训练结果更新替换。

| <font style="color:rgb(100, 106, 115);">Bash</font>tree ~/limx_ws/tron1-rl-deploy-python/controllers/model/PF_TRON1A   .   ├── params.yaml   └── policy       ├── policy.onnx       └── encoder.onnx |
| :--- |


<font style="color:#3370FF;">8.1.4 </font>**仿真调试**

打开一个Bash终端，运行 MuJoCo 仿真器。

| <font style="color:rgb(100, 106, 115);">Python</font># 激活pointfoot_deploy的conda环境   conda activate pointfoot_deploy      # 运行仿真器   cd ~/limx_ws   python tron1-mujoco-sim/simulator.py |
| :--- |


打开一个Bash终端，运行控制算法。

如果您的机器人处于摔倒状态，请在启动运动控制后，单击 MuJoCo 仿真器左侧菜单栏的“Reset”按钮以重置机器人。此时，您将看到机器人恢复并开始行走。

| <font style="color:rgb(100, 106, 115);">Bash</font># 激活pointfoot_deploy的conda环境   conda activate pointfoot_deploy      # 运行控制算法   python tron1-rl-deploy-python/main.py |
| :--- |


打开一个Bash终端，运行虚拟遥控器：仿真的过程中，您可以使用虚拟遥控器来控制机器人运动。左摇杆：控制前进/后退/左转/右转运动；右遥控：可控制机器人左右横向运动。

| <font style="color:rgb(100, 106, 115);">Bash</font>./tron1-mujoco-sim/robot-joystick/robot-joystick |
| :--- |


![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738056382-c810ee08-871b-4343-a83c-c1ae759c7e29.png)

<font style="color:#3370FF;">8.1.5 </font>**真机调试**

修改开发者电脑IP：确保您的开发电脑与机器人本体通过外置网口连接。设置您的电脑IP地址为：10.192.1.200，并通过Shell命令ping 10.192.1.2 能够正常ping通。如下图所示对您的开发电脑进行IP设置：

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738056692-99c2ea8c-1191-45bb-b479-08bc43b9eec7.png)

打开开发者模式：机器人开机后，同时按下遥控器按键 R1 + Left，这时本体主机将会自动重启并切换机器人到开发者模式。在此模式下，用户可以开发自己的运动控制算法。模式设置掉电后不会失效，重新开机后仍然是开发者模式。遥控器切换工作模式的按键列表如下：

| **按键** | **模式** | **说明** |
| :--- | :--- | :--- |
| R1+Left | 开发者模式（需授权） | 用户使用运动控制开发接口开发自己的运动控制算法。 |
| R1+Right | 遥控模式 | 运行预安装的控制算法，实现复杂地形的平稳行走，如：上楼、下楼、过坎等。 |


进行校零动作：机器人开机启动后，执行运控程序之前，请进行校零，使机器人各个关节回到初始位置。校零对应的遥控器按键为L1+R1。

实机部署运行。在Bash终端只需下面Shell命令启动控制算法（在进行实机部署运行时，确保机器人吊装非常重要）：

| <font style="color:rgb(100, 106, 115);">Bash</font># 激活pointfoot_deploy的conda环境   conda activate pointfoot_deploy      # 指定机器人IP地址，运行控制算法   python tron1-rl-deploy-python/main.py 10.192.1.2 |
| :--- |


这时您可以通过遥控器按键L1 + △开启机器人行走功能。左摇杆：控制前进/后退/左转/右转运动；右遥控：可控制机器人左右横向运动。

遥控器按L1 + □关闭机器人行走功能。

<font style="color:#3370FF;">8.1.6 </font>**真机部署**

当您完成仿真和真机调试后，可以将您的算法程序部署到机器人上（在部署测试完成之前，确保机器人始终保持吊装状态非常重要，以保障安全）。详细步骤如下：

准备工作

保持机器人继续处于开发者模式：确保机器人仍处于开发者模式，方便您进行程序部署和调试。

网络连接：确保开发电脑与机器人通过外置网口（Ethernet）连接，网络稳定且通信正常。部署完成后，网络连接将不再需要。

拷贝算法程序到机器人

在开发电脑上打开终端，进入到存放算法程序的工作目录，例如 ~/limx_ws。

使用 scp 命令将包含算法的目录拷贝到机器人中，默认机器人用户为 guest，密码为 123456。

| <font style="color:rgb(100, 106, 115);">Bash</font>cd ~/limx_ws   scp -r tron1-rl-deploy-python guest@10.192.1.2:/home/guest |
| :--- |


安装运动控制开发库（如果尚未安装）：

| <font style="color:rgb(100, 106, 115);">Bash</font>ssh guest@10.192.1.2 "pip install /home/guest/tron1-rl-deploy-python/limxsdk-lowlevel/python3/amd64/limxsdk-*-py3-none-any.whl"    |
| :--- |


配置算法自动启动

SSH 登录机器人：使用 ssh 命令远程登录到机器人系统，密码为 123456。

| <font style="color:rgb(100, 106, 115);">Bash</font>ssh guest@10.192.1.2 |
| :--- |


修改自启动脚本：

打开自启动脚本 /home/guest/autolaunch/autolaunch.sh 进行编辑：

| <font style="color:rgb(100, 106, 115);">Bash</font>busybox vi /home/guest/autolaunch/autolaunch.sh |
| :--- |


在脚本中找到启动 main.py 的命令：确保 python3 /home/guest/tron1-rl-deploy-python/main.py 10.192.1.2 这一行未被注释（没有 # 注释符号）。编辑完成后，保存并退出编辑器。

| <font style="color:rgb(100, 106, 115);">Bash</font>#!/bin/bash      while true; do     # 启动用于控制 Pointfoot 机器人的 Python 控制器脚本     # 参数 10.192.1.2 表示机器人所在的 IP 地址     # 该行被注释掉，若要启用自动控制器启动，请取消注释     # 请根据您控制器脚本的实际路径修改此路径     export ROBOT_TYPE=PF_TRON1A      export RL_TYPE=isaacgym #需要加上这两行环境变量，否则重启后脚本不会自启动     python3 /home/guest/tron1-rl-deploy-python/main.py 10.192.1.2        # 使用 roslaunch 启动机器人控制算法     # 若要启动机器人控制算法，请取消下面注释     # 请根据您安装的实际路径修改路径     # source /home/guest/install/setup.bash     # roslaunch robot_hw pointfoot_hw.launch        # 等待 3 秒后重新启动     sleep 3   done |
| :--- |


重启机器人电脑。修改完成后，执行以下命令重启机器人：

| <font style="color:rgb(100, 106, 115);">Bash</font>reboot |
| :--- |


控制机器人运动

等待系统启动后，您可以使用遥控器控制机器人：

L1 + △：启动机器人行走功能；

L1 + □： 关闭机器人行走功能；

左摇杆：控制机器人前进、后退、左转、右转；

右摇杆：控制机器人左右横向运动。

<font style="color:#3370FF;">8.2 </font>**基于ROS C++ 部署**

<font style="color:#3370FF;">8.2.1 </font>**部署环境配置**

安装ROS Noetic：我们推荐在Ubuntu 20.04操作系统上建立基于ROS Noetic的算法开发环境。ROS提供了一系列工具和库，如核心库、通信库和仿真工具（如Gazebo），极大地便利了机器人算法的开发、测试和部署。这些资源为用户提供了一个丰富而完整的算法开发环境。

ROS Noetic 安装请参考文档：[https://wiki.ros.org/noetic/Installation/Ubuntu](https://wiki.ros.org/noetic/Installation/Ubuntu)，选择“ros-noetic-desktop-full”进行安装。

ROS Noetic 安装完成后，Bash终端输入以下Shell命令，安装开发环境所依赖的库：

| <font style="color:rgb(100, 106, 115);">Bash</font>sudo apt-get update   sudo apt install ros-noetic-urdf \                    ros-noetic-kdl-parser \                    ros-noetic-urdf-parser-plugin \                    ros-noetic-hardware-interface \                    ros-noetic-controller-manager \                    ros-noetic-controller-interface \                    ros-noetic-robot-state-* \                    ros-noetic-joint-state-* \                    ros-noetic-controller-manager-msgs \                    ros-noetic-control-msgs \                    ros-noetic-ros-control \                    ros-noetic-gazebo-* \                    ros-noetic-rqt-gui \                    ros-noetic-rqt-controller-manager \                    ros-noetic-plotjuggler* \                    ros-noetic-joy-teleop ros-noetic-joy \                    cmake build-essential libpcl-dev libeigen3-dev libopencv-dev libmatio-dev \                    python3-pip libboost-all-dev libtbb-dev liburdfdom-dev liborocos-kdl-dev -y |
| :--- |


安装onnxruntime依赖，下载链接：[https://github.com/microsoft/onnxruntime/releases/tag/v1.10.0](https://github.com/microsoft/onnxruntime/releases/tag/v1.10.0) 。请您根据自己的操作系统和平台选择合适版本下载。如在Ubuntu 20.04 x86_64，请按下面步骤进行安装：

| <font style="color:rgb(100, 106, 115);">Bash</font>wget [https://github.com/microsoft/onnxruntime/releases/download/v1.10.0/onnxruntime-linux-x64-1.10.0.tgz](https://github.com/microsoft/onnxruntime/releases/download/v1.10.0/onnxruntime-linux-x64-1.10.0.tgz)      tar xvf onnxruntime-linux-x64-1.10.0.tgz      sudo cp -a onnxruntime-linux-x64-1.10.0/include/* /usr/include   sudo cp -a onnxruntime-linux-x64-1.10.0/lib/* /usr/lib |
| :--- |


<font style="color:#3370FF;">8.2.2 </font>**创建工作空间**

可以按照以下步骤，创建一个RL部署开发工作空间：

打开一个Bash终端。

创建一个新目录来存放工作空间。例如，可以在用户的主目录下创建一个名为“limx_ws”的目录：

| <font style="color:rgb(100, 106, 115);">Bash</font>mkdir -p ~/limx_ws/src |
| :--- |


下载运动控制开发接口：

| <font style="color:rgb(100, 106, 115);">Bash</font>cd ~/limx_ws/src   git clone [https://github.com/limxdynamics/limxsdk-lowlevel.git](https://github.com/limxdynamics/limxsdk-lowlevel.git) |
| :--- |


下载Gazebo仿真器：

| <font style="color:rgb(100, 106, 115);">Bash</font>cd ~/limx_ws/src   git clone [https://github.com/limxdynamics/tron1-gazebo-ros.git](https://github.com/limxdynamics/tron1-gazebo-ros.git) |
| :--- |


下载机器人模型描述文件

| <font style="color:rgb(100, 106, 115);">Bash</font>cd ~/limx_ws/src   git clone [https://github.com/limxdynamics/robot-description.git](https://github.com/limxdynamics/robot-description.git) |
| :--- |


下载可视化工具

| <font style="color:rgb(100, 106, 115);">Bash</font>cd ~/limx_ws/src   git clone [https://github.com/limxdynamics/robot-visualization.git](https://github.com/limxdynamics/robot-visualization.git) |
| :--- |


下载RL部署源码：

| <font style="color:rgb(100, 106, 115);">Bash</font>cd ~/limx_ws/src   git clone [https://github.com/limxdynamics/tron1-rl-deploy-ros.git](https://github.com/limxdynamics/tron1-rl-deploy-ros.git) |
| :--- |


设置机器人型号：请参考“查看/设置机器人型号”章节，查看您的机器人型号。如果尚未设置，请按照以下步骤进行设置。

通过 Shell 命令 tree -L 1 src/robot-description/pointfoot 列出可用的机器人类型：

| <font style="color:rgb(100, 106, 115);">Plain Text</font>limx@limx:~$ tree -L 1 src/robot-description/pointfoot   src/robot-description/pointfoot   ├── PF_P441A   ├── PF_P441B   ├── PF_P441C   ├── PF_P441C2   ├── PF_TRON1A   ├── SF_TRON1A   └── WF_TRON1A |
| :--- |


以 PF_TRON1A（请根据实际机器人类型进行替换）为例，设置机器人型号类型：

| <font style="color:rgb(100, 106, 115);">Plain Text</font>echo 'export ROBOT_TYPE=PF_TRON1A' >> ~/.bashrc && source ~/.bashrc |
| :--- |


编译工程：

| <font style="color:rgb(100, 106, 115);">Bash</font># 如您安装了Conda，请临时禁用 Conda 环境   # 因为 Conda 会干扰 ROS 的运行环境设置   conda deactivate      cd ~/limx_ws   catkin_make install |
| :--- |


<font style="color:#3370FF;">8.2.3 </font>**更新RL训练模型**

在您的工作空间中，RL模型和配置文件所在路径为：~/limx_ws/src/tron1-rl-deploy-ros/robot_controllers/config，如下所示。请根据你的训练结果更新替换。

在您的工作空间中，以 PF_TRON1A机器人类型为例，RL模型和配置文件所在路径为：~/limx_ws/src/tron1-rl-deploy-ros/robot_controllers/config/pointfoot/PF_TRON1A，如下所示。请根据您的训练结果更新替换。

| <font style="color:rgb(100, 106, 115);">Bash</font>tree ~/limx_ws/src/tron1-rl-deploy-ros/robot_controllers/config/pointfoot/PF_TRON1A   .   ├── params.yaml   └── policy       ├── policy.onnx       └── encoder.onnx |
| :--- |


<font style="color:#3370FF;">8.2.4 </font>**仿真调试**

请进到您的工作空间，完成编译：

| <font style="color:rgb(100, 106, 115);">Bash</font># 如您安装了Conda，请临时禁用 Conda 环境   # 因为 Conda 会干扰 ROS 的运行环境设置   conda deactivate      cd ~/limx_ws   catkin_make install |
| :--- |


运行部署仿真：启动后，您将看到如下所示的界面，包括Gazebo仿真器和Robot Steering交互窗口。

在Gazebo窗口中，您可以使用快捷键Ctrl + Shift + R复位机器人；

您还可以通过Robot Steering交互窗口设置，发布主题为/cmd_vel，控制机器人的移动。

| <font style="color:rgb(100, 106, 115);">Bash</font># 如您安装了Conda，请临时禁用 Conda 环境   # 因为 Conda 会干扰 ROS 的运行环境设置   conda deactivate      # 运行 robot_hw   source install/setup.bash   roslaunch robot_hw pointfoot_hw_sim.launch |
| :--- |


![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738056919-be2ab343-83de-4d19-817e-23cc132fbe19.png)

虚拟遥控器：如果您觉得使用 Robot Steering 控制机器人不方便，可以使用虚拟遥控器来简化操作。以下是使用虚拟遥控器的具体步骤。

下载并运行虚拟遥控器

| <font style="color:rgb(100, 106, 115);">Bash</font># 下载虚拟遥控器   git clone [https://github.com/limxdynamics/robot-joystick.git](https://github.com/limxdynamics/robot-joystick.git)      # 运行虚拟遥控器   ./robot-joystick/robot-joystick |
| :--- |


![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738057144-9a2f486b-5be4-47fd-8ce0-9834a2768af4.png)

这时，您可以使用虚拟遥控器来控制机器人运动。左摇杆：控制前进/后退/左转/右转运动；右遥控：可控制机器人左右横向运动。

<font style="color:#3370FF;">8.2.5 </font>**真机调试**

修改开发者电脑IP：确保您的开发电脑与机器人本体通过外置网口连接。设置您的电脑IP地址为：10.192.1.200，并通过Shell命令ping 10.192.1.2 能够正常ping通。如下图所示对您的开发电脑进行IP设置：

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738057329-ac30a9aa-ead6-4929-81d3-d3d76088d5ae.png)

请进到您的工作空间，找到pointfoot_hw.launch启动文件，修改机器人的IP地址为10.192.1.2，如下图所示：

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738057547-6be67cc6-785d-451b-b3cd-7706e072fd67.png)

完成修改后进行编译：

| <font style="color:rgb(100, 106, 115);">Bash</font># 如您安装了Conda，请临时禁用 Conda 环境   # 因为 Conda 会干扰 ROS 的运行环境设置   conda deactivate      cd ~/limx_ws   catkin_make install |
| :--- |


打开开发者模式：机器人开机后，同时按下遥控器按键 R1 + Left，这时本体主机将会自动重启并切换机器人到开发者模式。在此模式下，用户可以开发自己的运动控制算法。模式设置掉电后不会失效，重新开机后仍然是开发者模式。遥控器切换工作模式的按键列表如下：

| **按键** | **模式** | **说明** |
| :--- | :--- | :--- |
| R1+Left | 开发者模式（需授权） | 用户使用运动控制开发接口开发自己的运动控制算法。 |
| R1+Right | 遥控模式 | 运行预安装的控制算法，实现复杂地形的平稳行走，如：上楼、下楼、过坎等。 |


进行校零动作：机器人开机启动后，执行运控程序之前，请进行校零，使机器人各个关节回到初始位置。校零对应的遥控器按键为L1+R1。

实机部署运行。在Bash终端只需下面Shell命令启动控制算法（在进行实机部署运行时，确保机器人吊装非常重要）：

| <font style="color:rgb(100, 106, 115);">Bash</font># 如您安装了Conda，请临时禁用 Conda 环境   # 因为 Conda 会干扰 ROS 的运行环境设置   conda deactivate      # 运行 robot_hw   source install/setup.bash   roslaunch robot_hw pointfoot_hw.launch |
| :--- |


这时您可以通过遥控器按键L1 + △开启机器人行走功能。左摇杆：控制前进/后退/左转/右转运动；右遥控：可控制机器人左右横向运动。

遥控器按L1 + □关闭机器人行走功能。

<font style="color:#3370FF;">8.2.6 </font>**真机部署**

当您完成仿真和真机调试后，可以将算法程序部署到机器人上（在部署测试完成之前，确保机器人始终保持吊装状态非常重要，以保障安全）。详细步骤如下：

准备工作

保持机器人继续处于开发者模式：确保机器人仍处于开发者模式，方便您进行程序部署和调试。

网络连接：确保开发电脑通过外置网口（Ethernet）与机器人稳定连接，通信正常。部署完成后，网络连接将不再需要。

拷贝算法程序到机器人

在开发电脑上打开终端，进入到存放算法程序的工作目录，例如 ~/limx_ws。

使用 scp 命令将包含算法的目录拷贝到机器人中，默认机器人用户为 guest，密码为 123456。

| <font style="color:rgb(100, 106, 115);">Bash</font>cd ~/limx_ws   scp -r install guest@10.192.1.2:/home/guest |
| :--- |


配置算法自动启动

SSH 登录机器人：使用 ssh 命令远程登录到机器人系统，密码为 123456。

| <font style="color:rgb(100, 106, 115);">Bash</font>ssh guest@10.192.1.2 |
| :--- |


修改自启动脚本：

打开自启动脚本 /home/guest/autolaunch/autolaunch.sh 进行编辑：

| <font style="color:rgb(100, 106, 115);">Bash</font>busybox vi /home/guest/autolaunch/autolaunch.sh |
| :--- |


在脚本中找到启动 roslaunch 的命令：确保 source /home/guest/install/setup.bash 和roslaunch robot_hw pointfoot_hw.launch 这两行未被注释（没有 # 注释符号）。编辑完成后，保存并退出编辑器。

| <font style="color:rgb(100, 106, 115);">Bash</font>#!/bin/bash      while true; do     # 启动用于控制 Pointfoot 机器人的 Python 控制器脚本     # 参数 10.192.1.2 表示机器人所在的 IP 地址     # 该行被注释掉，若要启用自动控制器启动，请取消注释     # 请根据您控制器脚本的实际路径修改此路径     # 注意：如limxsdk与机器人本体版本不匹配，请pip uninstall limxsdk后，再pip install安装更新！     # python3 /home/guest/tron1-rl-deploy-python/main.py 10.192.1.2        # 使用 roslaunch 启动机器人控制算法     # 若要启动机器人控制算法，请取消下面注释     # 请根据您安装的实际路径修改路径     source /opt/ros/noetic/setup.bash     source /home/guest/install/setup.bash     roslaunch robot_hw pointfoot_hw.launch        # 等待 3 秒后重新启动     sleep 3   done |
| :--- |


重启机器人：修改完成后，关机重启机器人。

控制机器人运动

| 在首次真机部署时，机器可能出现预期外的运动表现，请确保TRON1和操作者都处于安全的状态。 |
| :--- |


等待系统启动后，您可以使用遥控器控制机器人：

L1 + △：启动机器人行走功能；

L1 + □： 关闭机器人行走功能；

左摇杆：控制机器人前进、后退、左转、右转；

右摇杆：控制机器人左右横向运动。

<font style="color:#3370FF;">9. </font>**RealSense 相机**

我们在机器人上安装了RealSense D435i相机，它是一款强大的深度摄像头，可提供深度和图像数据，主要用于地形感知。摄像头视角范围如下：

| ![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738057780-eb774077-78ec-47bc-9d4d-3438214a4bb2.png) | ![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738057959-cbdd85f5-5559-480b-bb39-27cb45112833.png) |
| :---: | :---: |


通过本章介绍，您可以学习如何使用ROS的Noetic版本获取D435i相机数据。具体步骤如下：

确认您的电脑安装了ROS的Noetic版本系统（请参考文档：[https://wiki.ros.org/noetic/Installation/Ubuntu](https://wiki.ros.org/noetic/Installation/Ubuntu)，选择“ros-noetic-desktop-full”进行安装）。

配置网络连接：确保您的开发电脑与机器人本体通过外置网口连接。设置您的电脑IP地址为：10.192.1.200，并通过Shell命令ping 10.192.1.2 能够正常ping通。如下图所示对您的开发电脑进行IP设置：

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738058136-47245644-d606-4495-a115-e98618f7e987.png)

通过机器人管理页面打开摄像头。

如果摄像头未启动，可以通过以下步骤启动：

在浏览器地址栏输入：[http://10.192.1.2:8080](http://10.192.1.2:8080) 进入机器人管理页面。

导航到“机器人信息”页面。

点击“打开摄像头”以启动D435i摄像头。

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738058357-32b7bc7c-f291-444f-a402-3b1a6bddb365.png)

配置ROS环境以获取相机数据。

为了使您的电脑通过ROS接收到机器人的相机数据，您需要设置ROS的环境变量：

设置 **ROS_MASTER_URI** 和 **ROS_IP**，以确保您的电脑能够正确连接到机器人的ROS主节点：您可以将下述命令添加到您电脑的 .bashrc 文件中，这样每次启动终端时都会自动应用设置。

| <font style="color:rgb(100, 106, 115);">Bash</font>export ROS_MASTER_URI=http://10.192.1.2:11311   export ROS_IP=10.192.1.200 |
| :--- |


通过ROS查看D435i相机发布的数据主题：此命令将列出所有相机发布的主题，您可以订阅感兴趣的主题来查看数据。

| <font style="color:rgb(100, 106, 115);">Bash</font>rostopic list |
| :--- |


使用rqt或rviz可视化相机数据

可以使用 rqt_image_view 或 rviz 来可视化D435i相机的图像和深度数据：

安装 rqt_image_view 工具：

| <font style="color:rgb(100, 106, 115);">Bash</font>sudo apt install ros-<ros版本>-rqt-image-view |
| :--- |


然后启动该工具并选择相机主题查看数据：

| <font style="color:rgb(100, 106, 115);">Bash</font>rqt_image_view |
| :--- |


使用 rviz 可视化工具查看深度数据：

| <font style="color:rgb(100, 106, 115);">Bash</font>rviz |
| :--- |


在rviz中添加相应的 Image 和 PointCloud2 类型显示器，选择相机相关主题以进行可视化。

<font style="color:#3370FF;">10. </font>**机器人软件升级**

我们通过浏览器进入机器人管理页面，选择本地提前下载好的机器人软件版本进行升级。具体步骤如下：

请选择并连接您机器人的 Wi-Fi 热点，密码为：12345678

![](https://cdn.nlark.com/yuque/0/2025/png/43111222/1757738058643-1dc76877-6a28-4c1e-8a6e-a6966a7386a1.png)

访问管理页面：

在浏览器地址栏输入：[<font style="color:#3370FF;">http://10.192.1.2:8080</font>](http://10.192.1.2:8080/) 进入机器人管理页面。

选择和升级软件：

依次选择“版本管理 -> 选择文件 -> 升级”。

升级完成后，机器人主控电脑将自动重启。

![](https://cdn.nlark.com/yuque/0/2025/jpeg/43111222/1757738058884-d5fe7b31-6fad-4f43-9813-d961bc94a5db.jpeg)

<font style="color:#3370FF;">11. </font>**常见问题**

