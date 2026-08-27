# 命令与产物

查表页；安装叙事见 [端到端训练走读](../walkthroughs/01-end-to-end-training.md)。

## 编译

```shell
cd ~/HierarchicalRL-robot-navigation/catkin_ws
catkin_make_isolated
```

## 环境变量（每终端）

```shell
export ROS_HOSTNAME=localhost
export ROS_MASTER_URI=http://localhost:11311
export ROS_PORT_SIM=11311
export GAZEBO_RESOURCE_PATH=~/HierarchicalRL-robot-navigation/catkin_ws/src/multi_robot_scenario/launch
source ~/HierarchicalRL-robot-navigation/catkin_ws/devel_isolated/setup.bash
```

## 训练命令

```shell
cd ~/HierarchicalRL-robot-navigation/TD3

# 分层 DQN+TD3（实验性）
python train_hierarchical.py

# SB3 TD3 基线（推荐首选）
python train_sb3_td3.py

# 自定义 TD3
python train_velodyne_td3.py

# 分层超参搜索（仅 hierarchical_rl 模块入口）
python hierarchical_rl.py --optimize
```

### train_hierarchical.py 常用参数

| 参数 | 默认 | 说明 |
| --- | --- | --- |
| `--environment_dim` | 20 | 激光扇区数 |
| `--max_timesteps` | 2000000 | 总训练步数 |
| `--eval_freq` | 5000 | 评估与保存间隔 |
| `--device` | auto | CUDA 可用时用 GPU，否则 CPU |
| `--load_high_level` | None | 预训练 DQN 路径 |
| `--load_low_level` | None | 预训练 TD3 路径 |

## 监控

```shell
tensorboard --logdir ~/HierarchicalRL-robot-navigation/TD3/logs
```

## 强制清理残留进程

```shell
killall -9 rosout roslaunch rosmaster gzserver nodelet robot_state_publisher gzclient python python3
```

分层训练退出时会关闭本次持有的 Gym/env；若进程仍残留，请用上面的 `killall` 兜底。

## 产物路径

| 类型 | 路径 | 说明 |
| --- | --- | --- |
| 分层高层模型 | `TD3/pytorch_models/high_level/` | `ckpt_<step>`、`final_model` |
| 分层低层模型 | `TD3/pytorch_models/low_level/` | 同上 |
| SB3 TD3 模型 | `TD3/pytorch_models/sb3_td3/` | CheckpointCallback |
| 自定义 TD3 模型 | `TD3/pytorch_models/` | velodyne 脚本输出 |
| TensorBoard | `TD3/logs/high_level_dqn/` | 分层高层 |
| TensorBoard | `TD3/logs/low_level_td3/` | 分层低层 |
| 评估结果 | `TD3/logs/evaluation_metrics.npy` | dict：`summary` + `episode_metrics` |
| Optuna | `TD3/optuna.db` | SQLite |
| 最优超参文本 | `TD3/configs/best_hyperparams.txt` | Optuna 结束后写入 |

## 关键 ROS 话题

| 话题 | 类型 | 方向 |
| --- | --- | --- |
| `/velodyne_points` | PointCloud2 | 订阅 |
| `/r1/odom` | Odometry | 订阅 |
| `/r1/cmd_vel` | Twist | 发布 |
| `gazebo/set_model_state` | ModelState | 发布 |
| `goal_point` | MarkerArray | 发布（RViz） |

## 依赖版本（README 声明）

| 组件 | 版本 |
| --- | --- |
| OS | Ubuntu 20.04 |
| ROS | Noetic |
| Python | 3.8.10 |
| PyTorch | 1.10 |

更高版本环境未在仓库中验证。
