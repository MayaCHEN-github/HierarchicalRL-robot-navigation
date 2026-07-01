# 仿真环境模块

证据状态：除特别标注外，本页基于当前源码已确认。

## 白话模型

训练时的「环境」不是纯 Python 函数，而是一套 ROS + Gazebo 仿真：

1. 启动 Gazebo 世界（含静态障碍与可移动纸箱）
2. Pioneer3DX 机器人（模型名 `r1`）配备 Velodyne 点云
3. 每个 RL 步：发布速度 → 仿真 0.1 秒 → 读激光与里程计 → 算奖励

Python 侧用两层包装：底层 [GazeboEnv](../../TD3/velodyne_env.py) 直接跟 ROS 话题交互；上层 [VelodyneGymWrapper](../../TD3/gym_wrapper.py) 把接口改成 Gymnasium 标准（`reset` 返回 `(obs, info)`，`step` 返回五元组）。

## 代码模型

### GazeboEnv 生命周期

初始化时（[velodyne_env.py](../../TD3/velodyne_env.py)）：

| 步骤 | 行为 |
| --- | --- |
| 启动 roscore | 默认端口 11311 |
| `rospy.init_node("gym")` | 注册 ROS 节点 |
| `roslaunch` | 加载 [TD3/assets/multi_robot_scenario.launch](../../TD3/assets/multi_robot_scenario.launch) |
| 订阅 | `/velodyne_points`、`/r1/odom` |
| 发布 | `/r1/cmd_vel`、`gazebo/set_model_state`、RViz 标记 |

### 激光观测处理

360° 点云按 `environment_dim`（默认 20）个扇区取最近距离，无点处填 10.0 m。碰撞检测：任一扇区距离 < `COLLISION_DIST`（0.35 m）判为碰撞。

### 状态向量（24 维）

```
[laser_0 … laser_19, distance_to_goal, theta_to_goal, linear_vel, angular_vel]
```

- `distance_to_goal`：欧氏距离
- `theta_to_goal`：目标相对航向角（经归一化）
- 速度来自上一步动作

`GazeboEnv.step` 返回 `(state, reward, done, target)` 四元组；wrapper 将其转为 Gymnasium 五元组，`target` 放入 `info['target_reached']`。

### 回合重置

`reset()` 流程：

1. 调用 `/gazebo/reset_world`
2. 随机合法机器人位姿（`check_pos` 排除固定障碍区域）
3. `change_goal()` 随机新目标（随训练进行逐渐扩大采样范围）
4. `random_box()` 重新摆放 4 个 `cardboard_box_*` 障碍
5. 返回初始观测

### 环境奖励（低层直接使用）

[get_reward](../../TD3/velodyne_env.py)：

| 条件 | 奖励 |
| --- | --- |
| 到达目标（距离 < 0.3 m） | +100 |
| 碰撞 | -100 |
| 否则 | `v/2 - |ω|/2 - r(min_laser)/2`，`r(x)=1-x`（x<1） |

### VelodyneGymWrapper 额外职责

- 定义 `observation_space` / `action_space`（连续：线速度 [0,1]，角速度 [-1,1]）
- 可选离散动作映射（20 动作，供 DQN 脚本使用）
- `close()` 时发零速度并 `rospy.signal_shutdown`

## 仿真场景资产

| 资产 | 位置 |
| --- | --- |
| Gazebo 世界 | [catkin_ws/.../launch/TD3.world](../../catkin_ws/src/multi_robot_scenario/launch/TD3.world) |
| 机器人 URDF/xacro | [catkin_ws/.../xacro/p3dx/](../../catkin_ws/src/multi_robot_scenario/xacro/p3dx/) |
| Velodyne 插件 | [catkin_ws/src/velodyne_simulator/](../../catkin_ws/src/velodyne_simulator/) |
| 训练用 launch 副本 | [TD3/assets/multi_robot_scenario.launch](../../TD3/assets/multi_robot_scenario.launch) |

## 已知不足

1. **步进仿真开销大**：每步 `sleep(0.1)` + 暂停/恢复物理，I/O 重，GPU 难以加速（论文亦提及）。
2. **目标合法性硬编码**：`check_pos()` 用固定矩形区域判断，换地图需手改。
3. **里程计超时仅警告**：10 秒内无 odom 仍继续，可能导致训练初期状态异常。
4. **无动态障碍**：论文强调动态环境，但仿真中障碍仅在 reset 时随机摆放，不会中途移动。
5. **gym / gymnasium 混用痕迹**：部分脚本与注释仍反映迁移过程中的兼容问题。

## 接下去阅读

- 分层如何用 24 维观测扩展为 26 维 → [分层强化学习模块](hierarchical-rl.md)
- 各训练脚本如何选择 `action_type` → [训练入口模块](training-scripts.md)
