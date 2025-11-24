# TurtleBot3 - ROS 2 机器人平台

<img src="https://raw.githubusercontent.com/ROBOTIS-GIT/emanual/master/assets/images/platform/turtlebot3/logo_turtlebot3.png" width="300">

## 📋 项目概述

**TurtleBot3** 是ROBOTIS推出的开源移动机器人平台，基于ROS 2框架。它是一个小型、低成本、可完全自定义的教育和研究机器人。本项目提供了完整的ROS 2软件栈，包括控制、导航、SLAM、遥控等功能。

- **活跃分支**: noetic, humble, jazzy, main(rolling)
- **遗留分支**: *-devel
- **版本**: 2.3.4
- **许可证**: Apache 2.0
- **网站**: http://turtlebot3.robotis.com

---

## 🎯 核心特性

| 特性 | 说明 |
|-----|------|
| **模块化设计** | 支持多种机器人型号（Burger、Waffle、Waffle Pi） |
| **完整软件栈** | 包含控制、导航、SLAM、遥控等完整功能 |
| **仿真支持** | 集成Gazebo和RViz可视化 |
| **开源硬件** | 所有硬件设计开源可获取 |
| **教育导向** | 适合学习ROS 2和机器人开发 |
| **社区活跃** | 拥有大量第三方扩展和应用 |

---

## 📦 项目组成

本项目是一个**元包（Metapackage）**，包含以下子包：

### 1. **turtlebot3_bringup** - 启动包
启动和配置机器人硬件的核心包。

**主要功能**:
- 加载机器人参数配置
- 启动必要的硬件驱动
- 初始化传感器
- 管理机器人状态

**关键文件**:
- `launch/` - 启动脚本（robot.launch.py等）
- `param/` - 机器人配置文件（YAML格式）
- `script/` - 辅助脚本

**依赖**:
- rclcpp
- geometry_msgs, nav_msgs, sensor_msgs
- turtlebot3_msgs

---

### 2. **turtlebot3_description** - 机器人描述包
定义机器人的物理结构和外观。

**主要功能**:
- 提供URDF模型文件
- 定义机器人的连接和关节
- 提供3D网格模型
- 配置RViz可视化

**主要目录**:
- `urdf/` - 机器人URDF模型
  - `turtlebot3_burger.urdf.xacro` - Burger型号
  - `turtlebot3_waffle.urdf.xacro` - Waffle型号
  - `turtlebot3_waffle_pi.urdf.xacro` - Waffle Pi型号（含摄像头）
- `meshes/` - 3D模型文件（STL格式）
- `rviz/` - RViz配置文件

**关键参数**:
```yaml
# 机器人物理参数
质量: 0.82-2.6 kg（型号相关）
尺寸: 150-220 mm × 140-200 mm
最大速度: 0.22-0.26 m/s
转向速度: 1.82-2.84 rad/s
```

---

### 3. **turtlebot3_teleop** - 遥控包
提供键盘和手柄遥控机器人的功能。

**主要功能**:
- 键盘遥控脚本
- 摇杆遥控支持
- 速度控制和限制
- 实时速度显示

**脚本说明**:

#### `teleop_keyboard.py` - 键盘遥控
**用途**: 通过键盘实时控制机器人

**键位映射**:
```
      w        (前进)
   a    d      (左转/右转)
      x        (后退)
   空格/s      (停止)

w/x: 增加/减少线速度
a/d: 增加/减少角速度
```

**工作原理**:
- 读取键盘输入
- 将键盘命令转换为速度指令
- 发布到 `/cmd_vel` 话题
- 实时显示当前速度

**特性**:
- 不同型号的速度限制
- 平滑加速/减速
- 紧急停止功能

**运行方法**:
```bash
# 方法1：直接运行（推荐）
ros2 run turtlebot3_teleop teleop_keyboard

# 方法2：在另一个终端中运行
# 在终端1启动机器人硬件驱动后，在终端2执行上述命令
```

---

### 4. **turtlebot3_node** - 硬件驱动节点
与实际机器人硬件通信的底层驱动节点。

**主要功能**:
- 与OpenCR微控制器通信
- 发送电机控制信号
- 接收传感器数据
- 管理硬件状态

**发布话题**:
- `/odom` - 里程计信息
- `/scan` - LiDAR扫描数据
- `/imu` - IMU数据
- `/battery_state` - 电池状态

**订阅话题**:
- `/cmd_vel` - 速度指令

---

### 5. **turtlebot3_navigation2** - 导航包
基于ROS 2 Navigation2框架的自主导航功能。

**主要功能**:
- 路径规划（A*、Dijkstra等算法）
- 地图导航
- 目标点导航
- 自适应路由

**关键组件**:
- Costmap配置
- 本地/全局路径规划器
- 行为树（Behavior Tree）

**运行方法**:
```bash
# 启动Navigation2栈
ros2 launch turtlebot3_navigation2 navigation2.launch.py map:=<map_file>
```

---

### 6. **turtlebot3_cartographer** - SLAM包
使用Google Cartographer进行SLAM（同步定位与建图）。

**主要功能**:
- 实时地图构建
- 机器人定位
- 回环检测
- 地图优化

**特点**:
- 高精度地图生成
- 实时性能优异
- 支持多机器人

**运行方法**:
```bash
# 启动Cartographer SLAM
ros2 launch turtlebot3_cartographer cartographer.launch.py
```

---

### 7. **turtlebot3_example** - 示例包
提供各种应用示例代码。

**示例包括**:
- 基础运动示例
- 传感器读取示例
- 导航示例
- 自定义行为示例

---

## 🤖 机器人型号对比

### Burger（入门版）
- **特点**: 小型、低成本、基础功能
- **尺寸**: 150 × 140 × 140 mm
- **质量**: 0.82 kg
- **最大速度**: 0.22 m/s
- **适用**: 教学、入门、轻量级应用

### Waffle（标准版）
- **特点**: 中等规模、增强功能
- **尺寸**: 220 × 200 × 140 mm
- **质量**: 1.5 kg
- **最大速度**: 0.26 m/s
- **适用**: 研究、中等复杂度应用

### Waffle Pi（进阶版）
- **特点**: 集成树莓派、配备摄像头
- **尺寸**: 220 × 200 × 140 mm
- **质量**: 2.6 kg
- **最大速度**: 0.26 m/s
- **额外功能**: RGB摄像头、树莓派计算
- **适用**: 视觉导航、高级应用

---

## 🚀 快速开始

### 环境要求
- **操作系统**: Ubuntu 20.04 LTS (Noetic) 或 Ubuntu 22.04 LTS (Humble)
- **ROS版本**: ROS 2 Noetic 或 Humble
- **Python**: 3.8+
- **依赖包**: ros2-dev、colcon、rosdep

### 安装步骤

#### 1. 创建工作空间
```bash
mkdir -p ~/colcon_ws/src
cd ~/colcon_ws
```

#### 2. 克隆项目
```bash
cd src
git clone https://github.com/ROBOTIS-GIT/turtlebot3.git -b humble
```

#### 3. 安装依赖
```bash
cd ~/colcon_ws
sudo apt-get update
rosdep install --from-paths src --ignore-src -r -y
```

#### 4. 构建项目
```bash
colcon build --symlink-install
```

#### 5. 配置环境
```bash
source /usr/share/gazebo/setup.sh
source install/setup.bash
export TURTLEBOT3_MODEL=burger    # 或 waffle, waffle_pi
```

---

## 📝 常见使用场景

### 1. 键盘遥控机器人
```bash
# 终端1：启动机器人基础驱动
ros2 launch turtlebot3_bringup robot.launch.py

# 终端2：启动键盘遥控
ros2 run turtlebot3_teleop teleop_keyboard
```

### 2. SLAM建图
```bash
# 启动SLAM节点
ros2 launch turtlebot3_cartographer cartographer.launch.py

# 另一个终端：用遥控器移动机器人进行建图
ros2 run turtlebot3_teleop teleop_keyboard

# 第三个终端：保存地图
ros2 run nav2_map_server map_saver_cli -f ~/map
```

### 3. 自主导航
```bash
# 启动导航栈
ros2 launch turtlebot3_navigation2 navigation2.launch.py \
    map:=/path/to/your/map.yaml

# 在RViz中设置导航目标
# 或通过代码发送目标点
```

### 4. 仿真测试
```bash
# 终端1：启动Gazebo仿真
ros2 launch turtlebot3_gazebo empty_world.launch.py

# 终端2：运行遥控
ros2 run turtlebot3_teleop teleop_keyboard
```

---

## 🔄 Gazebo中的机器人重置

在Gazebo仿真中，经常需要重置机器人状态以进行新的实验。以下是几种重置方法：

### 重置整个仿真（完全重置）
```bash
# 重置所有状态：位置、速度、时间等
ros2 service call /reset_simulation std_srvs/Empty
```

### 只重置世界状态（保留时间）
```bash
# 重置物体状态，但保留仿真时间
ros2 service call /reset_world std_srvs/Empty
```

### 停止机器人运动
```bash
# 发送零速度指令
ros2 topic pub -1 /cmd_vel geometry_msgs/Twist '{linear: {x: 0.0}, angular: {z: 0.0}}'
```

### Python自动重置脚本
```python
import rclpy
from std_srvs.srv import Empty

rclpy.init()
node = rclpy.create_node('reset_node')
client = node.create_client(Empty, '/reset_simulation')

while not client.wait_for_service(timeout_sec=1.0):
    print('等待服务...')

request = Empty.Request()
future = client.call_async(request)
rclpy.spin_until_future_complete(node, future)
print('✓ 仿真已重置')
rclpy.shutdown()
```

---

### 发布话题（Publish Topics）

| 话题 | 类型 | 说明 |
|-----|------|------|
| `/odom` | nav_msgs/Odometry | 里程计信息 |
| `/scan` | sensor_msgs/LaserScan | LiDAR扫描数据 |
| `/imu` | sensor_msgs/Imu | IMU传感器数据 |
| `/battery_state` | sensor_msgs/BatteryState | 电池状态 |
| `/tf` | tf2_msgs/TFMessage | 变换框架 |
| `/cmd_vel_out` | geometry_msgs/Twist | 实际执行的速度 |

### 订阅话题（Subscribe Topics）

| 话题 | 类型 | 说明 |
|-----|------|------|
| `/cmd_vel` | geometry_msgs/Twist | 速度指令输入 |
| `/map` | nav_msgs/OccupancyGrid | 地图数据 |

### 服务（Services）

| 服务 | 说明 |
|-----|------|
| `/sound` | 播放声音 |
| `/motor_power` | 控制电机电源 |
| `/led` | 控制LED |

---

## 🔧 项目架构

```
TurtleBot3 ROS 2栈
│
├─ turtlebot3_bringup (启动/配置)
│  ├─ 机器人参数加载
│  ├─ 硬件驱动初始化
│  └─ 节点启动管理
│
├─ turtlebot3_node (硬件驱动)
│  ├─ OpenCR通信
│  ├─ 电机控制
│  └─ 传感器读取
│
├─ turtlebot3_teleop (远程控制)
│  ├─ 键盘控制
│  ├─ 摇杆控制
│  └─ 速度管理
│
├─ turtlebot3_navigation2 (自主导航)
│  ├─ 路径规划
│  ├─ 目标导航
│  └─ 避障控制
│
├─ turtlebot3_cartographer (SLAM)
│  ├─ 地图构建
│  ├─ 机器人定位
│  └─ 回环检测
│
└─ turtlebot3_description (机器人描述)
   ├─ URDF模型
   ├─ 3D网格
   └─ RViz配置
```

---

## 🐛 调试与故障排除

### 问题1：无法找到机器人
**症状**: `Could not find robot`
**解决方案**:
```bash
# 确保TURTLEBOT3_MODEL环境变量已设置
export TURTLEBOT3_MODEL=burger

# 检查USB连接
ls /dev/ttyUSB*

# 重新上传固件到OpenCR
```

### 问题2：命令响应缓慢
**症状**: 机器人响应键盘命令延迟
**解决方案**:
```bash
# 检查ROS 2中间件
export RMW_IMPLEMENTATION=rmw_fastrtps_cpp

# 降低消息发布周期
# 编辑turtlebot3_teleop配置
```

### 问题3：传感器数据异常
**症状**: LiDAR/IMU数据不合理
**解决方案**:
```bash
# 检查传感器连接
ros2 node list | grep sensor

# 查看传感器话题
ros2 topic list | grep scan

# 调试传感器输出
ros2 topic echo /scan
```

---

## 📚 文档和学习资源

### 官方文档
- 📖 **[TurtleBot3官方e-Manual](http://turtlebot3.robotis.com/)** - 完整的硬件和软件文档
- ⚙️ **[ROBOTIS DYNAMIXEL](https://dynamixel.com/)** - 舵机和电机文档
- 📚 **[Dynamixel SDK](http://emanual.robotis.com/docs/en/software/dynamixel/dynamixel_sdk/overview/)** - SDK使用指南

### 视频教程
- 🎥 **[ROBOTIS YouTube频道](https://www.youtube.com/@ROBOTISCHANNEL)** - 官方视频资源
- 🎥 **[TurtleBot3视频播放列表](https://www.youtube.com/playlist?list=PLRG6WP3c31_XI3wlvHlx2Mp8BYqgqDURU)** - TurtleBot3专题视频
- 🎥 **[ROBOTIS开源团队频道](https://www.youtube.com/@ROBOTISOpenSourceTeam)** - 社区贡献视频

### 社区支持
- 💬 **[ROBOTIS官方论坛](https://forum.robotis.com/)** - 技术支持和讨论
- 💬 **[ROS Discourse TurtleBot分类](https://discourse.ros.org/c/turtlebot/)** - ROS社区讨论

---

## 🔗 相关开源项目

**TurtleBot3生态系统**:
- [turtlebot3_msgs](https://github.com/ROBOTIS-GIT/turtlebot3_msgs) - 消息定义
- [turtlebot3_simulations](https://github.com/ROBOTIS-GIT/turtlebot3_simulations) - 仿真环境
- [turtlebot3_manipulation](https://github.com/ROBOTIS-GIT/turtlebot3_manipulation) - 机械臂集成
- [turtlebot3_applications](https://github.com/ROBOTIS-GIT/turtlebot3_applications) - 应用示例
- [turtlebot3_machine_learning](https://github.com/ROBOTIS-GIT/turtlebot3_machine_learning) - 机器学习示例
- [turtlebot3_autorace](https://github.com/ROBOTIS-GIT/turtlebot3_autorace) - 自动竞赛
- [turtlebot3_home_service_challenge](https://github.com/ROBOTIS-GIT/turtlebot3_home_service_challenge) - 家务挑战

**支持库**:
- [hls_lfcd_lds_driver](https://github.com/ROBOTIS-GIT/hls_lfcd_lds_driver) - LiDAR驱动
- [ld08_driver](https://github.com/ROBOTIS-GIT/ld08_driver) - 二线激光驱动
- [open_manipulator](https://github.com/ROBOTIS-GIT/open_manipulator) - OpenMANIPULATOR机械臂
- [dynamixel_sdk](https://github.com/ROBOTIS-GIT/DynamixelSDK) - Dynamixel舵机SDK

---

## 📋 项目统计

| 指标 | 信息 |
|-----|------|
| **版本** | 2.3.4 |
| **维护者** | Pyo (pyo@robotis.com) |
| **主要贡献者** | Darby Lim, Ryan Shim, Will Son, Hyungyu Kim |
| **支持的ROS版本** | Noetic, Humble, Jazzy, Rolling |
| **开发语言** | C++, Python |
| **许可证** | Apache 2.0 |

---

## 💡 最佳实践

### 1. 安全操作
- 总是在安全区域测试新代码
- 设置合理的速度限制
- 实现紧急停止机制
- 定期检查电池状态

### 2. 开发建议
- 使用仿真进行初期开发
- 遵循ROS命名规范
- 编写清晰的注释
- 提交前测试功能

### 3. 性能优化
- 使用QoS配置优化通信
- 选择合适的消息频率
- 定期监控CPU和内存使用
- 使用Rviz进行可视化调试

### 4. 代码质量
```bash
# 代码风格检查
ament_flake8 src/

# 静态分析
ament_cpplint src/

# 单元测试
colcon test
```

---

## 🤝 贡献指南

欢迎贡献代码、报告bug或提出改进建议！

### 提交PR步骤：
1. Fork本仓库
2. 创建功能分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送分支 (`git push origin feature/AmazingFeature`)
5. 开启Pull Request

### DCO签名
所有提交必须包含Signed-off-by行：
```bash
git commit -s -m "Commit message"
```

---

## 📄 许可证

本项目采用 **Apache License 2.0** 许可证。详见 [LICENSE](LICENSE) 文件。

---

## 📞 获取帮助

- 📖 查阅 [官方文档](http://turtlebot3.robotis.com/)
- 💬 在 [论坛](https://forum.robotis.com/) 提问
- 🐛 在 [GitHub Issues](https://github.com/ROBOTIS-GIT/turtlebot3/issues) 报告问题
- 💌 联系维护团队

---

**最后更新**: 2024年
**文档版本**: 2.3.4
