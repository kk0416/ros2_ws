# ROS 2 学习记录

> 记录截至 2026-11-09 的学习过程、实验结果、学习反馈和后续计划。\
> 环境：Windows + WSL2 Ubuntu 22.04 + ROS 2 Humble。

## 1. 学习目标

以 ROS 2 C++ 为主要方向，结合已有的 Qt/C++、Linux
和嵌入式开发经验，逐步掌握 ROS 2 工程开发，并最终完成 Qt + ROS 2
机器人监控 HMI 实战项目。

目标实战项目包括：

-   机器人状态、位置、电池和速度信息显示
-   机器人速度设置
-   任务下发、状态反馈和取消
-   故障信息处理
-   Qt 与 ROS 2 集成
-   ARM64 设备部署、systemd 服务管理和故障恢复

## 2. 开发环境

  项目           当前环境
  -------------- ---------------------------
  主机系统       Windows
  Linux 环境     WSL2 Ubuntu 22.04.5 LTS
  ROS 2 发行版   Humble
  CPU 架构       x86_64
  C++ 标准       C++17
  构建工具       Colcon、CMake
  IDE            Visual Studio Code（WSL）

WSL 环境检查结果：

``` text
Ubuntu 22.04.5 LTS
x86_64
ROS_DISTRO=humble
ros2: /opt/ros/humble/bin/ros2
colcon: /usr/bin/colcon
```

ROS 2 安装命令：

``` bash
sudo apt install -y ros-humble-desktop ros-dev-tools
```

已在 `~/.bashrc` 中配置：

``` bash
source /opt/ros/humble/setup.bash
```

新开终端后会自动加载 ROS 2 环境。

### 软件源问题及处理

最初使用 HTTPS 访问 ROS 2 软件源时，出现证书主机名不匹配：

``` text
Certificate verification failed
subjectAltName does not match packages.ros.org
```

排查后将软件源调整为：

``` text
http://packages.ros.org/ros2/ubuntu
```

随后安装成功。软件包仍通过仓库签名和校验机制验证。没有通过关闭 TLS
验证来绕过问题。

## 3. 已完成的学习与实验

### 3.1 ROS 2 环境验证

已验证 ROS 2 环境变量和命令可用：

``` bash
echo $ROS_DISTRO
which ros2
which colcon
ros2 --help
```

预期关键结果：

``` text
humble
/opt/ros/humble/bin/ros2
/usr/bin/colcon
```

### 3.2 Talker / Listener 通信实验

分别在两个终端运行：

``` bash
ros2 run demo_nodes_cpp talker
```

``` bash
ros2 run demo_nodes_cpp listener
```

通过 ROS 2 CLI 检查节点和 Topic：

``` bash
ros2 node list
ros2 topic list
ros2 topic echo /chatter
ros2 topic info /chatter
```

已观察到：

``` text
节点：
/listener
/talker

Topic：
/chatter
/parameter_events
/rosout

/chatter 类型：
std_msgs/msg/String

Publisher count: 1
Subscription count: 1
```

`/chatter` 持续收到 `Hello World` 消息，证明两个 C++ 节点已经通过 Topic
完成通信。

理解要点：

-   Node 是 ROS 2 中组织功能的基本单元。
-   Topic 用于发布/订阅式通信。
-   Publisher 负责发布消息。
-   Subscription 负责接收消息。
-   `ros2 node list` 和 `ros2 topic list` 可以检查运行状态。
-   对 `ros2 topic echo` 按 `Ctrl+C`，只会结束当前 echo
    命令，不会自动关闭 Talker 和 Listener。

### 3.3 创建 robot_demo 功能包

工作空间：

``` text
~/ros2_ws
```

创建命令：

``` bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src

ros2 pkg create robot_demo \
    --build-type ament_cmake \
    --license Apache-2.0 \
    --dependencies rclcpp std_msgs
```

生成的主要结构：

``` text
robot_demo/
├── CMakeLists.txt
├── include/
│   └── robot_demo/
├── LICENSE
├── package.xml
└── src/
```

`package.xml` 已声明：

``` xml
<buildtool_depend>ament_cmake</buildtool_depend>
<depend>rclcpp</depend>
<depend>std_msgs</depend>
```

### 3.4 构建和发现功能包

已成功执行：

``` bash
cd ~/ros2_ws
colcon build --packages-select robot_demo
```

构建结果：

``` text
Starting >>> robot_demo
Finished <<< robot_demo

Summary: 1 package finished
```

加载工作空间：

``` bash
source ~/ros2_ws/install/setup.bash
```

检查功能包：

``` bash
ros2 pkg list | grep robot_demo
```

已找到 `robot_demo`，说明工作空间构建和功能包发现正常。

注意：当前 `robot_demo` 仍是刚创建的空功能包，还没有确认完成自己的 C++
节点。

## 4. VS Code 与 WSL

已通过 VS Code 的 WSL 环境打开工作空间，并验证终端使用的是 WSL
Ubuntu，而不是 Windows 本机环境。

建议项目放在 Linux 文件系统：

``` text
/home/hannah/ros2_ws
```

初期不建议放在 `/mnt/c` 下，以减少文件权限和文件监听性能方面的问题。

推荐插件：

-   WSL（Microsoft）
-   C/C++（Microsoft）
-   CMake Tools
-   clangd（可选，后续再配置）

## 5. Git 仓库进展

已在工作空间初始化 Git：

``` bash
cd ~/ros2_ws
git init
git branch -m main
```

已创建 `.gitignore`，忽略构建产物和 IDE 临时文件：

``` gitignore
build/
install/
log/
.vscode/
.idea/
*.swp
*.tmp
*~
```

已创建 `README.md`，记录开发环境、工作空间结构和学习进度。

GitHub 远程仓库：

``` text
https://github.com/kk0416/ros2_ws
```

远程地址使用 SSH：

``` text
git@github.com:kk0416/ros2_ws.git
```

已成功推送 `main` 分支：

``` text
[new branch] main -> main
Branch 'main' set up to track remote branch 'main'
```

### 当前文档目录

本学习记录计划保存为：

``` text
~/ros2_ws/docs/ROS2学习记录.md
```

此前只创建了 `docs/`
目录，文档尚未成功复制进去。本文件补齐后，还需要执行 Git 提交和推送。

### 日常 Git 流程

``` bash
git status
git add .
git commit -m "feat: add robot node"
git push
```

建议一个小而完整、经过验证的功能对应一个
commit。不要每敲一行就提交，也不要把一堆未经验证的改动堆到最后。

## 6. 学习方式与反馈

学习过程中明确采用主动编码方式，而不是直接复制完整代码。

约定如下：

1.  先理解概念和运行机制，再开始编码。
2.  先给任务、代码框架和关键提示，不急着给完整答案。
3.  用户先独立实现，再由助手检查代码并解释问题。
4.  出错时优先学习定位和分析过程，而不是直接替换成正确代码。
5.  每节课安排一个独立的小改造任务。
6.  每完成一个阶段，更新学习记录。
7.  每隔几节课安排一次综合练习。

每日建议约 1 小时：

  时间      内容
  --------- ------------------
  10 分钟   复习前一天内容
  15 分钟   学习新概念
  25 分钟   编码、构建和调试
  10 分钟   总结并记录问题

学习重点不是写出更多代码，而是逐步做到不看教程也能独立设计、实现和调试。

## 7. Qt 与 ROS 2 概念对照

  ROS 2 概念       Qt 中可参考的概念            说明
  ---------------- ---------------------------- ----------------------
  Node             QObject 功能模块             组织一个独立功能单元
  Publisher        信号发送方的部分职责         发布消息
  Subscription     信号接收方的部分职责         接收消息
  Timer            QTimer                       定时触发回调
  Executor         Qt 事件循环的部分作用        调度 ROS 回调
  Callback Group   回调并发管理机制             控制回调执行关系
  Service          请求-响应调用                适合短时请求
  Action           带反馈和取消能力的异步任务   适合长时间任务

以上是帮助理解的类比，不是严格的一一对应关系。尤其 Executor 与 Qt
事件循环的实现机制并不相同。

## 8. 12 周学习路线

  ----------------------------------------------------------------------------------------------
  阶段                                主要内容
  ----------------------------------- ----------------------------------------------------------
  第 1-2 周                           ROS 2 环境、Workspace、Package、Node、CLI、Launch

  第 3-4 周                           Topic、Service、Action、标准消息

  第 5-6 周                           rclcpp、Timer、Parameter、自定义接口、Executor、Callback
                                      Group、多线程、Lifecycle、Component、rosbag2

  第 7 周                             DDS、RTPS、Discovery、Domain ID、QoS、Fast DDS、Cyclone
                                      DDS

  第 8-9 周                           TF2、URDF、RViz2、Gazebo、传感器、Odometry、Nav2

  第 10-12 周                         综合工程、Qt + ROS 2、ARM64 部署、systemd、故障恢复
  ----------------------------------------------------------------------------------------------

路线会根据实际理解情况调整，不以赶进度为目标。

## 9. 当前进度与下一步

### 已完成

-   [x] WSL2 Ubuntu 22.04 环境准备
-   [x] 安装 ROS 2 Humble
-   [x] 配置 ROS 2 环境变量
-   [x] Talker / Listener 通信实验
-   [x] 使用 ROS 2 CLI 查看节点和 Topic
-   [x] 创建 `ros2_ws` 工作空间
-   [x] 创建 `robot_demo` 功能包
-   [x] 成功构建并发现 `robot_demo`
-   [x] VS Code 连接 WSL
-   [x] 初始化 Git 并成功推送 GitHub

### 进行中

-   [ ] 创建第一个自己的 ROS 2 C++ 节点 `robot_node`
-   [ ] 配置 CMakeLists.txt 构建并安装节点
-   [ ] 使用 `ros2 run` 启动自己的节点
-   [ ] 完成节点基础知识思考题

### 下一步任务：实现 robot_node

源文件计划位置：

``` text
~/ros2_ws/src/robot_demo/src/robot_node.cpp
```

先完成以下任务，不急着复制完整实现：

``` cpp
#include "rclcpp/rclcpp.hpp"

// TODO 1: Define a class named RobotNode.
// It should inherit from rclcpp::Node.

// TODO 2: Add a constructor.
// Set the node name to "robot_node".

// TODO 3: Print "Robot node started!" using RCLCPP_INFO.

int main(int argc, char * argv[])
{
    // TODO 4: Initialize ROS 2.

    // TODO 5: Create a RobotNode instance.

    // TODO 6: Keep the node running.

    // TODO 7: Shutdown ROS 2.

    return 0;
}
```

思考题：

1.  为什么 `RobotNode` 要继承 `rclcpp::Node`？
2.  `rclcpp::init()` 应该放在创建节点之前还是之后？
3.  为什么需要 `rclcpp::spin()`？

完成源码后，再修改
`CMakeLists.txt`，添加可执行程序、依赖和安装规则。随后构建、运行并验证。

预期运行命令：

``` bash
cd ~/ros2_ws
colcon build --packages-select robot_demo
source install/setup.bash
ros2 run robot_demo robot_node
```

预期日志：

``` text
[INFO] Robot node started!
```

节点基础功能完成后，再加入 1 秒定时器，学习 ROS 2 Timer 和回调执行机制。

------------------------------------------------------------------------

**记录原则：**
只把已经实际完成并验证的内容标记为完成。计划、推测和待验证事项单独列出，避免学习记录变成一份看起来很努力、实际上不太诚实的文件。
