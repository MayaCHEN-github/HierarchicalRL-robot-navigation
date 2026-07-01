# 端到端训练走读

证据状态：除特别标注外，本页基于当前源码已确认。

本走读覆盖从编译 ROS 工作空间到启动训练、查看产物的完整路径。默认你在 Ubuntu 20.04 + ROS Noetic 环境操作（与项目 README 一致）。

## Step 1: 准备依赖

需要事先安装：

- ROS Noetic
- PyTorch、TensorBoard
- Python 包：`stable-baselines3`、`gymnasium`、`rospy` 相关 ROS Python 绑定、`squaternion`、`optuna`（仅分层超参搜索时用）

克隆仓库后，catkin 工作空间位于仓库内的 [catkin_ws](../..) 目录。

## Step 2: 编译并 source ROS 工作空间

在终端执行（路径按你的克隆位置调整）：

```shell
cd ~/HierarchicalRL-robot-navigation/catkin_ws
catkin_make_isolated
```

每次新开终端需要设置环境变量并 source：

```shell
export ROS_HOSTNAME=localhost
export ROS_MASTER_URI=http://localhost:11311
export ROS_PORT_SIM=11311
export GAZEBO_RESOURCE_PATH=~/HierarchicalRL-robot-navigation/catkin_ws/src/multi_robot_scenario/launch
source ~/HierarchicalRL-robot-navigation/catkin_ws/devel_isolated/setup.bash
```

`GAZEBO_RESOURCE_PATH` 让 Gazebo 能找到本地世界与模型文件。

## Step 3: 选择训练入口

进入 [TD3](../../TD3) 目录，按目标选脚本：

| 目标 | 命令 | 说明 |
| --- | --- | --- |
| 分层 DQN+TD3（实验性） | `python train_hierarchical.py` | 调用 [hierarchical_rl.py](../../TD3/hierarchical_rl.py) |
| SB3 版 TD3 基线 | `python train_sb3_td3.py` | 推荐作为稳定对比基线 |
| 自定义 TD3 实现 | `python train_velodyne_td3.py` | 原 DRL-robot-navigation 风格，手写 Actor-Critic |
| 单层 DQN | `python train_dqn.py` 或 `train_sb3_DQN.py` | 离散动作导航 |

**建议**：若你第一次运行，优先 `train_sb3_td3.py` 或 `train_velodyne_td3.py`，确认 Gazebo 与传感器链路正常后，再尝试分层训练。

## Step 4: 仿真如何被拉起

无论选哪个脚本，环境侧都会经过类似链路：

1. [VelodyneGymWrapper](../../TD3/gym_wrapper.py) 创建 Gymnasium 兼容接口
2. 内部实例化 [GazeboEnv](../../TD3/velodyne_env.py)
3. `GazeboEnv` 启动 `roscore`，再通过 [multi_robot_scenario.launch](../../TD3/assets/multi_robot_scenario.launch) 拉起 Gazebo 世界与 Pioneer3DX 机器人（`r1`）
4. 订阅 `/velodyne_points` 点云、发布 `/r1/cmd_vel` 速度指令

Launch 默认 `gazebo_gui=false`（无 Gazebo 窗口）、`rviz=true`（开 RViz 可视化）。

## Step 5: 训练过程中发生什么

以分层训练为例（[HierarchicalRL.train](../../TD3/hierarchical_rl.py)）：

1. **重置回合**：随机机器人位姿、目标点、四个纸箱障碍物
2. **每步**：
   - 高层 DQN 从 200 个离散动作中选方向×距离子目标
   - 低层 TD3 根据「原观测 + 子目标」输出连续 `(线速度, 角速度)`
   - Gazebo 执行 0.1s 仿真步，返回新观测与环境奖励
   - 分别计算高层/低层自定义奖励，写入各自 replay buffer
3. **每 100 步**（buffer > 1000 后）：对两层各做一次梯度更新
4. **每 5000 步**：评估、保存检查点到 `pytorch_models/`

单层 SB3 TD3 则由 SB3 的 `learn()` 驱动，逻辑更简单。

## Step 6: 查看训练产物

| 产物 | 典型路径 | 用途 |
| --- | --- | --- |
| 模型检查点 | `TD3/pytorch_models/` | 分层：`high_level/`、`low_level/`；SB3：`sb3_td3/` 等 |
| TensorBoard 日志 | `TD3/logs/` | `tensorboard --logdir TD3/logs` |
| 评估指标 | `TD3/logs/evaluation_metrics.npy` | 分层评估回调写出 |
| Optuna 数据库 | `TD3/optuna.db` | 仅 `--optimize` 时生成 |

## Step 7: 结束训练与清理

训练异常退出时，Gazebo/ROS 进程可能残留。可执行：

```shell
killall -9 rosout roslaunch rosmaster gzserver nodelet robot_state_publisher gzclient python python3
```

[hierarchical_rl.py](../../TD3/hierarchical_rl.py) 还在进程退出时注册 `_cleanup_gazebo_ros()` 做兜底清理。

## 验证清单

按顺序自检：

- [ ] `catkin_make_isolated` 无报错
- [ ] `source devel_isolated/setup.bash` 后 `rospack find multi_robot_scenario` 有输出
- [ ] 运行 `train_sb3_td3.py` 后终端出现「Roscore launched!」「里程计数据初始化成功」
- [ ] RViz 中能看到绿色目标柱与机器人
- [ ] `TD3/logs/` 或 `TD3/pytorch_models/` 在训练数分钟后有新文件

若分层训练出现「原地打转、回合过早结束」，这与论文自述的不稳定现象一致，不一定是环境配置错误；可先用单层 TD3 排除仿真链路问题。

## 接下去阅读

- 改两层协作逻辑 → [分层强化学习模块](../modules/hierarchical-rl.md)
- 改传感器或奖励 → [仿真环境模块](../modules/gazebo-environment.md)
- 查状态维度与奖励公式 → [观测、动作与奖励](../references/observation-action-reward.md)
