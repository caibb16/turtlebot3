# TurtleBot3 导航完全指南

本指南详细说明如何启动TurtleBot3导航系统，并在RViz中设置导航目标。

---

## 📋 前置条件

### 1. 环境设置
```bash
# 源代码编译后，设置以下环境变量
source /opt/ros/humble/setup.bash
source ~/colcon_ws/install/setup.bash

# 设置机器人型号（必须）
export TURTLEBOT3_MODEL=burger    # 或 waffle, waffle_pi
```

### 2. 必要的包
确保已安装以下ROS 2包：
- `turtlebot3_navigation2` - 导航包
- `nav2_bringup` - Navigation2核心
- `cartographer` 或其他SLAM包（用于建图）
- `rviz2` - 可视化工具

---

## 🚀 快速开始（仿真环境）

### 方案1：使用Gazebo仿真（推荐用于开发测试）

#### 步骤1：启动Gazebo仿真环境（终端1）
```bash
export TURTLEBOT3_MODEL=burger
ros2 launch turtlebot3_gazebo empty_world.launch.py
```

**或使用带地图的环境：**
```bash
# 带房屋环境
ros2 launch turtlebot3_gazebo turtlebot3_house.launch.py

# 带障碍物的测试世界
ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py
```

**预期输出：**
- ✓ Gazebo窗口打开，显示机器人和环境
- ✓ 终端输出Gazebo和ROS节点信息
- ✓ 没有错误消息

#### 步骤2：启动导航栈（终端2）

**方案A：使用默认地图启动**
```bash
export TURTLEBOT3_MODEL=burger
ros2 launch turtlebot3_navigation2 navigation2.launch.py use_sim_time:=true
```

**方案B：使用自定义地图**
```bash
export TURTLEBOT3_MODEL=burger
ros2 launch turtlebot3_navigation2 navigation2.launch.py \
    use_sim_time:=true \
    map:=<path_to_your_map.yaml>
```

**示例：**
```bash
# 使用包内置的地图
ros2 launch turtlebot3_navigation2 navigation2.launch.py \
    use_sim_time:=true \
    map:=$(ros2 pkg prefix turtlebot3_navigation2)/share/turtlebot3_navigation2/map/map.yaml
```

**预期输出：**
- ✓ RViz窗口打开，显示导航界面
- ✓ 显示成本地图（costmap）
- ✓ 显示机器人位置和激光扫描
- ✓ Navigation2节点启动成功

---

## 🏠 真实机器人导航

### 前置条件
- 机器人已通过SSH连接或本地启动
- 机器人硬件驱动正常运行
- LiDAR和IMU传感器工作正常
- 已获得环境地图（.yaml + .pgm 文件）

### 步骤1：启动机器人基础驱动（机器人端或终端1）
```bash
export TURTLEBOT3_MODEL=burger
ros2 launch turtlebot3_bringup robot.launch.py
```

### 步骤2：启动导航栈（工作站端或终端2）
```bash
export TURTLEBOT3_MODEL=burger
export ROS_DOMAIN_ID=30  # 确保与机器人使用相同的域ID

# 启动导航，指定已有地图
ros2 launch turtlebot3_navigation2 navigation2.launch.py \
    use_sim_time:=false \
    map:=/path/to/your/map.yaml
```

**注意：** `use_sim_time:=false` 用于真实机器人

---

## 📍 在RViz中设置导航目标

### 方法1：使用"2D Nav Goal"工具（推荐）

#### 1. 打开RViz
在启动导航栈时，RViz会自动打开。如果没有打开，手动启动：
```bash
rviz2 -d $(ros2 pkg prefix turtlebot3_navigation2)/share/turtlebot3_navigation2/rviz/tb3_navigation2.rviz
```

#### 2. 初始化机器人位置（首次启动时必需）

**点击工具栏中的"2D Pose Estimate"按钮**
1. 在RViz工具栏找到 `2D Pose Estimate` 按钮（箭头图标）
2. 在地图上点击机器人的当前位置
3. 拖动鼠标设置机器人朝向（绿色箭头）
4. 释放鼠标完成设置

**预期结果：**
- RViz中机器人位置更新
- 导航栈开始跟踪机器人位置

#### 3. 设置导航目标

**方法A：点击"2D Nav Goal"工具**
1. 在RViz工具栏找到 `2D Nav Goal` 按钮（红色箭头图标）
   - 通常位于工具栏中，工具栏显示: `2D Pose Estimate`, `2D Nav Goal`
   
2. 在地图上点击目标位置
   - 黑色箭头显示目标位置
   
3. 拖动鼠标设置目标朝向（目标方向）
   - 箭头方向指示机器人到达目标后的朝向

4. 释放鼠标完成目标设置

**预期行为：**
- ✓ 路径规划器计算路径（通常在1-3秒内）
- ✓ RViz显示全局路径（绿线）和局部路径（红线）
- ✓ 机器人开始沿路径移动
- ✓ 终端显示导航状态信息

### 方法2：通过命令行发送导航目标

如果RViz工具不可用，可以通过命令行发送目标：

```bash
# 发送单个目标点
ros2 action send_goal /navigate_to_pose nav2_msgs/action/NavigateToPose \
"{pose: {header: {frame_id: 'map'}, pose: {position: {x: 1.0, y: 1.0, z: 0.0}, orientation: {x: 0.0, y: 0.0, z: 0.0, w: 1.0}}}}"
```

### 方法3：通过Python脚本发送目标

创建文件 `send_goal.py`：

```python
#!/usr/bin/env python3
import rclpy
from geometry_msgs.msg import PoseStamped
from nav2_simple_commander.robot_navigator import BasicNavigator

def navigate_to_goal():
    rclpy.init()
    navigator = BasicNavigator()
    
    # 等待导航栈完全启动
    navigator.waitUntilNav2Active()
    
    # 创建目标位置
    goal_pose = PoseStamped()
    goal_pose.header.frame_id = 'map'
    goal_pose.header.stamp = rclpy.clock.Clock().now().to_msg()
    
    # 设置目标坐标和朝向
    goal_pose.pose.position.x = 1.0
    goal_pose.pose.position.y = 1.0
    goal_pose.pose.position.z = 0.0
    goal_pose.pose.orientation.w = 1.0
    
    print(f"导航到目标: x={goal_pose.pose.position.x}, y={goal_pose.pose.position.y}")
    
    # 发送目标
    navigator.goToPose(goal_pose)
    
    # 监控导航过程
    while not navigator.isTaskComplete():
        feedback = navigator.getFeedback()
        print(f"距离目标: {feedback.distance_remaining:.2f}m")
        rclpy.spin_once(navigator, timeout_sec=0.1)
    
    result = navigator.getResult()
    if result == BasicNavigator.TaskResult.SUCCEEDED:
        print("✓ 导航成功！")
    else:
        print("✗ 导航失败！")
    
    rclpy.shutdown()

if __name__ == '__main__':
    navigate_to_goal()
```

运行脚本：
```bash
# 安装依赖（如果还没有）
sudo apt-get install ros-humble-nav2-simple-commander

# 运行脚本
python3 send_goal.py
```

---

## 📊 RViz主要功能说明

### 1. 工具栏按钮

| 按钮 | 功能 | 说明 |
|------|------|------|
| **2D Pose Estimate** | 设置初始位置 | 点击地图设置机器人起始位置和朝向 |
| **2D Nav Goal** | 设置导航目标 | 点击地图设置目标位置和朝向 |
| **Select** | 选择工具 | 选择和查看对象 |
| **Move Camera** | 移动视图 | 调整摄像头视角 |
| **Zoom** | 缩放 | 放大/缩小地图 |

### 2. 左侧面板（Displays）

常用的显示选项：
- **Global Costmap** - 全局成本地图（绿色/黄色/红色表示可通行/未知/障碍）
- **Local Costmap** - 局部成本地图
- **Global Plan** - 全局路径规划（绿色线）
- **Local Plan** - 局部规划（红色线）
- **Robot** - 机器人模型
- **Laser Scan** - LiDAR扫描数据
- **Navigation Goal** - 当前导航目标
- **Initial Pose** - 初始位置

### 3. 快捷键

| 快捷键 | 功能 |
|-------|------|
| `1` | 切换到第一个工具 |
| `2` | 切换到第二个工具 |
| `R` | 重置视图 |
| `鼠标滚轮` | 缩放地图 |
| `右键拖动` | 旋转视图 |
| `中键拖动` | 平移视图 |

---

## 🎯 常见导航场景

### 场景1：单目标点导航
```bash
# 在RViz中
1. 点击"2D Pose Estimate"，设置初始位置
2. 点击"2D Nav Goal"，设置目标位置
3. 观察机器人自动导航到目标
```

### 场景2：多目标点导航（通过代码）

```python
#!/usr/bin/env python3
import rclpy
from geometry_msgs.msg import PoseStamped
from nav2_simple_commander.robot_navigator import BasicNavigator

def navigate_to_waypoints():
    rclpy.init()
    navigator = BasicNavigator()
    navigator.waitUntilNav2Active()
    
    # 定义多个目标点
    waypoints = [
        (0.0, 0.0, 0.0),      # (x, y, yaw)
        (1.0, 1.0, 0.785),    # 45度
        (2.0, 0.0, 1.57),     # 90度
        (0.0, 0.0, 0.0),      # 返回起点
    ]
    
    for i, (x, y, yaw) in enumerate(waypoints):
        goal_pose = PoseStamped()
        goal_pose.header.frame_id = 'map'
        goal_pose.header.stamp = rclpy.clock.Clock().now().to_msg()
        goal_pose.pose.position.x = x
        goal_pose.pose.position.y = y
        goal_pose.pose.position.z = 0.0
        
        # 从yaw角度创建四元数
        import math
        qz = math.sin(yaw / 2.0)
        qw = math.cos(yaw / 2.0)
        goal_pose.pose.orientation.z = qz
        goal_pose.pose.orientation.w = qw
        
        print(f"导航到路点 {i+1}/{len(waypoints)}: ({x}, {y})")
        navigator.goToPose(goal_pose)
        
        while not navigator.isTaskComplete():
            rclpy.spin_once(navigator, timeout_sec=0.1)
    
    print("✓ 所有路点导航完成！")
    rclpy.shutdown()

if __name__ == '__main__':
    navigate_to_waypoints()
```

### 场景3：避障导航

导航栈自动处理动态障碍物：
1. 启动导航后，LiDAR检测到障碍物
2. 成本地图自动更新
3. 路径规划器重新规划路径
4. 机器人绕过障碍物继续导航

**监控避障行为：**
```bash
# 查看成本地图更新
ros2 topic echo /global_costmap/costmap

# 查看路径重规划
ros2 topic echo /plan
```

### 场景4：导航失败处理

```python
#!/usr/bin/env python3
import rclpy
from geometry_msgs.msg import PoseStamped
from nav2_simple_commander.robot_navigator import BasicNavigator

def navigate_with_recovery():
    rclpy.init()
    navigator = BasicNavigator()
    navigator.waitUntilNav2Active()
    
    goal_pose = PoseStamped()
    goal_pose.header.frame_id = 'map'
    goal_pose.header.stamp = rclpy.clock.Clock().now().to_msg()
    goal_pose.pose.position.x = 5.0
    goal_pose.pose.position.y = 5.0
    goal_pose.pose.orientation.w = 1.0
    
    print("开始导航...")
    navigator.goToPose(goal_pose)
    
    while not navigator.isTaskComplete():
        feedback = navigator.getFeedback()
        rclpy.spin_once(navigator, timeout_sec=0.1)
    
    result = navigator.getResult()
    
    if result == BasicNavigator.TaskResult.SUCCEEDED:
        print("✓ 导航成功!")
    elif result == BasicNavigator.TaskResult.CANCELED:
        print("⚠ 导航被取消")
    elif result == BasicNavigator.TaskResult.FAILED:
        print("✗ 导航失败，执行恢复动作...")
        # 执行恢复：后退，转圈等
        navigator.spin(spin_dist=1.0)  # 转圈
        
        # 重新尝试导航
        print("重新尝试导航...")
        navigator.goToPose(goal_pose)
        
        while not navigator.isTaskComplete():
            rclpy.spin_once(navigator, timeout_sec=0.1)
    
    rclpy.shutdown()

if __name__ == '__main__':
    navigate_with_recovery()
```

---

## 🔧 配置和调优

### 1. 调整导航参数

编辑配置文件：
```bash
# 根据机器人型号选择配置
nano ~/colcon_ws/src/turtlebot3_navigation2/param/burger.yaml
```

关键参数：

```yaml
# 全局路径规划器
planner_server:
  ros__parameters:
    expected_planner_frequency: 10.0  # 规划频率(Hz)
    GridBased:
      tolerance: 0.5  # 目标到达容差(m)
      use_astar: false  # 是否使用A*算法

# 局部路径规划器
controller_server:
  ros__parameters:
    controller_frequency: 10.0  # 控制频率(Hz)
    FollowPath:
      max_vel_x: 0.3  # 最大线速度(m/s)
      max_vel_theta: 1.0  # 最大角速度(rad/s)
      min_speed_xy: 0.0  # 最小线速度
      
# 成本地图
local_costmap:
  local_costmap:
    ros__parameters:
      width: 3  # 本地成本地图宽度(m)
      height: 3  # 本地成本地图高度(m)
      resolution: 0.05  # 分辨率(m/cell)
      robot_radius: 0.1  # 机器人半径(m)
      inflation_radius: 0.5  # 膨胀半径(m)
```

### 2. 速度限制

```yaml
# 最大速度限制
max_vel_x: 0.3        # 前进最大速度
max_vel_theta: 1.0    # 转向最大速度

# 加速度限制
acc_lim_x: 3.0        # 前进加速度
acc_lim_theta: 3.2    # 转向加速度

# 减速度限制  
decel_lim_x: -2.5     # 前进减速度
decel_lim_theta: -3.2 # 转向减速度
```

### 3. 成本地图参数优化

```yaml
# 膨胀层配置
inflation_layer:
  inflation_radius: 0.5  # 膨胀半径，越大越保守
  cost_scaling_factor: 5.0  # 成本缩放因子

# 观测源配置
obstacle_layer:
  observation_sources: scan
  scan:
    topic: /scan
    max_obstacle_height: 2.0
    clearing: true  # 是否清除被扫描过的区域
    marking: true   # 是否标记障碍物
    raytrace_max_range: 3.0  # 最大射线追踪范围
    obstacle_max_range: 2.5   # 最大障碍物检测范围
```

---

## 🐛 故障排除

### 问题1：RViz显示不出地图

**症状**：RViz中看不到地图

**解决方案**：
```bash
# 1. 检查地图话题是否发布
ros2 topic list | grep map
ros2 topic echo /map

# 2. 检查地图文件是否存在
ls -la /path/to/your/map.yaml

# 3. 在RViz中添加地图显示
# 点击 Add > By Topic > Map，选择 /map
```

### 问题2：机器人位置不更新

**症状**：机器人在RViz中不移动或位置错误

**原因**：
- 定位失败
- 里程计噪声过大
- AMCL未收敛

**解决方案**：
```bash
# 1. 重新设置初始位置
# 在RViz中点击"2D Pose Estimate"重新设置

# 2. 检查定位节点
ros2 node list | grep amcl

# 3. 查看定位话题
ros2 topic echo /amcl_pose

# 4. 调整AMCL参数
# 编辑: turtlebot3_navigation2/param/burger.yaml
# 增加 max_particles: 3000（默认2000）
```

### 问题3：路径规划失败

**症状**：设置目标后没有反应，RViz中看不到路径

**原因**：
- 目标点在障碍物中
- 目标点超出地图范围
- 规划器已禁用

**解决方案**：
```bash
# 1. 确认导航栈启动
ros2 node list | grep nav2

# 2. 查看规划器输出
ros2 topic echo /plan

# 3. 检查成本地图
# RViz中显示 Global Costmap，查看目标区域

# 4. 在可通行区域选择目标

# 5. 查看规划器错误
ros2 service call /plan_to_pose nav2_msgs/srv/ComputePathToPose \
    "{start: {header: {frame_id: 'map'}, pose: {position: {x: 0, y: 0}}}, goal: {header: {frame_id: 'map'}, pose: {position: {x: 1, y: 1}}}}"
```

### 问题4：机器人来回抖动

**症状**：导航时机器人不稳定地前进

**原因**：
- 局部规划器参数不合适
- 传感器噪声过大
- 控制器频率过低

**解决方案**：
```yaml
# 调整局部规划器参数
FollowPath:
  sim_time: 1.5  # 降低到1.0试试
  vx_samples: 15  # 降低采样数
  vtheta_samples: 30
  
  # 增加转弯半径
  BaseObstacle.scale: 0.05  # 提高障碍物权重
  PathDist.scale: 24.0  # 降低
```

### 问题5：导航速度过慢

**症状**：机器人以很低的速度导航

**原因**：
- 规划器检测到"危险"
- 控制器设置太保守

**解决方案**：
```yaml
# 增大速度限制
max_vel_x: 0.5  # 提高线速度
max_vel_theta: 2.0  # 提高角速度

# 增大加速度限制
acc_lim_x: 5.0
acc_lim_theta: 5.0

# 降低保险距离
PolygonStop:
  radius: 0.05  # 从0.1降低到0.05
```

### 问题6：导航频繁失败并重试

**症状**：导航任务反复失败，出现重试行为

**原因**：
- 目标点不可达（被永久障碍物封闭）
- 自由空间过小
- 恢复行为无法脱困

**解决方案**：
```bash
# 1. 可视化全局成本地图，检查目标区域
ros2 launch rviz2 rviz2 -d /path/to/nav2.rviz

# 2. 选择新的目标点，确保在可通行区域

# 3. 调整膨胀参数让障碍物周围有更多空间
inflation_radius: 0.3  # 从0.5降低到0.3

# 4. 增加恢复行为的超时时间
failure_tolerance: 0.5  # 从0.3提高到0.5
```

---

## 📚 常用命令参考

### 导航相关话题和服务

```bash
# 查看可用的导航目标服务
ros2 service list | grep navigate

# 查看当前导航目标
ros2 topic echo /goal_pose

# 查看全局规划路径
ros2 topic echo /plan

# 查看机器人里程计
ros2 topic echo /odom

# 查看激光扫描
ros2 topic echo /scan

# 查看导航状态
ros2 action list

# 查看成本地图
ros2 topic echo /global_costmap/costmap

# 取消当前导航目标
ros2 action send_goal /navigate_to_pose nav2_msgs/action/NavigateToPose \
  "{}" --feedback
```

### 调试命令

```bash
# 启动导航栈时启用调试输出
RCL_LOGGING_CONFIG_DIR=/etc/ros2/logging_config \
ros2 launch turtlebot3_navigation2 navigation2.launch.py use_sim_time:=true

# 查看tf变换树
ros2 run tf2_tools view_frames

# 记录bag数据用于离线分析
ros2 bag record /scan /odom /amcl_pose /plan

# 回放bag数据
ros2 bag play <bagfile>
```

---

## 📖 进阶话题

### 1. 自定义行为树

在 `humble/burger.yaml` 中更改：
```yaml
bt_navigator:
  ros__parameters:
    default_nav_to_pose_bt_xml: "/path/to/custom_bt.xml"
```

### 2. 使用其他路径规划器

可用的规划器：
- `nav2_navfn_planner::NavfnPlanner` - Dijkstra（默认）
- `nav2_theta_star_planner::ThetaStarPlanner` - Theta*
- `nav2_smac_planner::SmacPlannerLattice` - Lattice

### 3. 集成自定义传感器

添加新的观测源到costmap配置中。

---

## ✅ 检查列表

启动导航前的准备清单：

- [ ] 设置了 `TURTLEBOT3_MODEL` 环境变量
- [ ] 机器人硬件/仿真已启动
- [ ] LiDAR数据正在发布
- [ ] 里程计数据正常
- [ ] 地图文件存在且路径正确
- [ ] 机器人能接收 `/cmd_vel` 指令
- [ ] RViz已打开并显示地图
- [ ] 初始位置已在RViz中设置

---

**更新时间**: 2024年11月  
**对应版本**: TurtleBot3 2.3.4, ROS 2 Humble
