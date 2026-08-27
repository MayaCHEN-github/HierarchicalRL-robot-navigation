<div align="center">

# HierarchicalRL-robot-navigation

[English](#english) · [中文](#中文)

[![ROS](https://img.shields.io/badge/ROS-Noetic-22314E?style=flat-square)](http://wiki.ros.org/noetic)
[![Python](https://img.shields.io/badge/Python-3.8.10-3776AB?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![PyTorch](https://img.shields.io/badge/PyTorch-1.10-EE4C2C?style=flat-square&logo=pytorch&logoColor=white)](https://pytorch.org/)
[![License](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](LICENSE)

Mobile robot navigation with deep reinforcement learning in ROS Gazebo.

[Overview](#overview) · [Get started](#getting-started) · [Train](#train) · [Architecture](#architecture) · [Docs](#documentation)

<img src="training.gif" alt="Training in Gazebo: the robot navigates to a random goal while avoiding obstacles" width="100%" />

</div>

## English

### Overview

This repository trains a mobile robot to reach a random goal in Gazebo while avoiding obstacles. Laser returns (from a simulated [Velodyne](https://github.com/lmark1/velodyne_simulator) 3D lidar) form the obstacle observation; the goal is given in polar coordinates relative to the robot.

It extends [reiniscimurs/DRL-robot-navigation](https://github.com/reiniscimurs/DRL-robot-navigation) (ICRA 2022 / IEEE RA-L) with:

- a hierarchical **DQN + TD3** stack
- Stable-Baselines3 (SB3) training entries
- a Gymnasium wrapper around the ROS–Gazebo environment

Tested with **ROS Noetic**, **Ubuntu 20.04**, **Python 3.8.10**, and **PyTorch 1.10**.

Upstream installation tutorial: [Deep Reinforcement Learning in Mobile Robot Navigation](https://medium.com/@reinis_86651/deep-reinforcement-learning-in-mobile-robot-navigation-tutorial-part1-installation-d62715722303).

### Features

- **Several training tracks** — hierarchical DQN+TD3, SB3 TD3/DQN, and the original custom TD3
- **Gymnasium interface** — `VelodyneGymWrapper` exposes the Gazebo env as a standard `gym.Env`
- **Velodyne observation** — 20 lidar sectors plus goal distance/angle and current velocities (24-D)
- **Evaluation metrics** — success rate, path efficiency, trajectory smoothness, time cost, collision rate
- **Chinese project guide** — detailed walkthroughs and tables in [`repo-docs/`](repo-docs/README.md)

### Current status

> [!IMPORTANT]
> Start with single-agent TD3. Hierarchical DQN+TD3 is runnable but still unstable, and there is no trustworthy quantitative win over the TD3 baseline yet.

| Track | Script | Status |
| --- | --- | --- |
| SB3 TD3 | `TD3/train_sb3_td3.py` | Recommended baseline |
| Custom TD3 | `TD3/train_velodyne_td3.py` | Mature, closest to the original paper |
| Hierarchical DQN+TD3 | `TD3/train_hierarchical.py` | Experimental / unstable |
| SB3 / custom DQN | `TD3/train_sb3_DQN.py`, `TD3/train_dqn.py` | Discrete single-agent control |

Paper mentions of PyBullet and PathBench are **not** in this source tree. Simulation is Gazebo + ROS only.

### Getting started

#### Prerequisites

- [ROS Noetic](http://wiki.ros.org/noetic/Installation)
- [PyTorch](https://pytorch.org/get-started/locally/)
- Python packages in [`requirements.txt`](requirements.txt): `numpy`, `torch`, `tensorboard`, `tqdm`, `optuna`, `stable-baselines3`, `gymnasium`, `squaternion`

```shell
pip install -r requirements.txt
```

#### Clone and compile

```shell
cd ~
git clone https://github.com/MayaCHEN-github/HierarchicalRL-robot-navigation.git
cd HierarchicalRL-robot-navigation/catkin_ws
catkin_make_isolated
```

The policy can run on a 2D laser, but this tree uses a simulated 3D Velodyne.

#### Source the workspace (every new terminal)

```shell
export ROS_HOSTNAME=localhost
export ROS_MASTER_URI=http://localhost:11311
export ROS_PORT_SIM=11311
export GAZEBO_RESOURCE_PATH=~/HierarchicalRL-robot-navigation/catkin_ws/src/multi_robot_scenario/launch
source ~/HierarchicalRL-robot-navigation/catkin_ws/devel_isolated/setup.bash
```

### Train

```shell
cd ~/HierarchicalRL-robot-navigation/TD3

# Recommended first run: SB3 TD3 baseline
python train_sb3_td3.py

# Original custom TD3
python train_velodyne_td3.py

# Hierarchical experiment (expect instability)
python train_hierarchical.py

# Optional Optuna search (standalone module entry)
python hierarchical_rl.py --optimize
```

Useful flags for hierarchical training: `--environment_dim 20`, `--max_timesteps 2000000`, `--eval_freq 5000`, `--device cuda|cpu`, `--load_high_level`, `--load_low_level`.

Monitor:

```shell
tensorboard --logdir ~/HierarchicalRL-robot-navigation/TD3/logs
```

> [!TIP]
> If Gazebo or ROS linger after a crash, stop them with:
> `killall -9 rosout roslaunch rosmaster gzserver nodelet robot_state_publisher gzclient python python3`

<p align="center">
  <img src="env1.png" alt="Gazebo environment with Pioneer3DX and obstacles" width="80%" />
</p>
<p align="center">
  <img src="velodyne.png" alt="RViz view of the Velodyne point cloud and goal marker" width="80%" />
</p>

### Architecture

```
HierarchicalRL-robot-navigation/
├── catkin_ws/     # ROS workspace: Pioneer3DX, Gazebo world, Velodyne plugin
├── TD3/           # Training code, Gym wrapper, hierarchical RL
├── repo-docs/     # Chinese project guide
└── README.md
```

**Hierarchical loop (DQN + TD3)**

1. High-level DQN picks a sub-goal from **200** discrete actions (20 directions × 10 distances).
2. The direction index is converted to radians and concatenated with the 24-D base observation.
3. Low-level TD3 outputs continuous `(linear velocity, angular velocity)`.
4. Gazebo steps 0.1 s; high- and low-level rewards are stored separately and trained every 100 steps after the replay buffer exceeds 1000 samples.

High-level reward mixes direction match, distance match, lidar-based obstacle avoidance, path smoothness, collision, and a small time penalty. Low-level reward adds the environment reward from `GazeboEnv.get_reward` (goal +100, collision −100).

**Evaluation**

| Metric | Definition |
| --- | --- |
| Success rate | Goal reached (distance < 0.3 m) |
| Path efficiency | Straight-line distance / path length |
| Trajectory smoothness | Inverse of mean curvature change |
| Time cost | Seconds to goal or termination |
| Collision rate | Episodes ending in collision (env reward < −90) |

### Documentation

| Resource | Description |
| --- | --- |
| [repo-docs/README.md](repo-docs/README.md) | Chinese hub and reading paths |
| [End-to-end walkthrough](repo-docs/walkthroughs/01-end-to-end-training.md) | Compile → train → artifacts |
| [Hierarchical RL](repo-docs/modules/hierarchical-rl.md) | Two-level loop and rewards |
| [Gazebo environment](repo-docs/modules/gazebo-environment.md) | Sensors, wrapper, world |
| [Training scripts](repo-docs/modules/training-scripts.md) | Which `train_*.py` to use |
| [Observation / action / reward](repo-docs/references/observation-action-reward.md) | Dimension and formula tables |
| [AGENTS.md](AGENTS.md) | Short English map for coding agents |

### Citation

Upstream DRL-robot-navigation paper:

```bibtex
@ARTICLE{9645287,
  author={Cimurs, Reinis and Suh, Il Hong and Lee, Jin Han},
  journal={IEEE Robotics and Automation Letters},
  title={Goal-Driven Autonomous Exploration Through Deep Reinforcement Learning},
  year={2022},
  volume={7},
  number={2},
  pages={730-737},
  doi={10.1109/LRA.2021.3133591}}
```

Paper: [IEEE Xplore](https://ieeexplore.ieee.org/document/9645287).

---

## 中文

### 项目简介

本仓库在 ROS Gazebo 中训练移动机器人：前往随机目标并避开障碍。障碍来自仿真 [Velodyne](https://github.com/lmark1/velodyne_simulator) 三维激光；目标以相对极坐标给出。

项目基于 [reiniscimurs/DRL-robot-navigation](https://github.com/reiniscimurs/DRL-robot-navigation)（ICRA 2022 / IEEE RA-L），并扩展了：

- **DQN + TD3** 分层架构
- Stable-Baselines3（SB3）训练入口
- 面向 ROS–Gazebo 环境的 Gymnasium 包装器

已在 **Ubuntu 20.04 + ROS Noetic + Python 3.8.10 + PyTorch 1.10** 上验证。上游安装教程见 [这篇 Medium 文章](https://medium.com/@reinis_86651/deep-reinforcement-learning-in-mobile-robot-navigation-tutorial-part1-installation-d62715722303)。

更细的中文导读从 [`repo-docs/README.md`](repo-docs/README.md) 开始。

### 特性

- **多条训练路线**：分层 DQN+TD3、SB3 TD3/DQN、原版自定义 TD3
- **Gymnasium 接口**：`VelodyneGymWrapper` 把 Gazebo 环境做成标准 `gym.Env`
- **Velodyne 观测**：20 个激光扇区 + 目标距离/角度 + 当前速度（24 维）
- **评估指标**：成功率、路径效率、轨迹平滑度、耗时、碰撞率
- **中文文档**：走读、模块说明与查表见 [`repo-docs/`](repo-docs/README.md)

### 当前成熟度

> [!IMPORTANT]
> 第一次运行请先走单层 TD3。分层 DQN+TD3 可以启动，但训练仍不稳定，目前没有可靠的定量结果证明它优于 TD3 基线。

| 路线 | 脚本 | 状态 |
| --- | --- | --- |
| SB3 TD3 | `TD3/train_sb3_td3.py` | 推荐基线 |
| 自定义 TD3 | `TD3/train_velodyne_td3.py` | 较成熟，最接近原论文 |
| 分层 DQN+TD3 | `TD3/train_hierarchical.py` | 实验性 / 不稳定 |
| SB3 / 自定义 DQN | `TD3/train_sb3_DQN.py`、`TD3/train_dqn.py` | 单层离散控制 |

论文中的 PyBullet、PathBench **未**出现在当前源码中，仿真只有 Gazebo + ROS。

### 快速开始

#### 依赖

- [ROS Noetic](http://wiki.ros.org/noetic/Installation)
- [PyTorch](https://pytorch.org/get-started/locally/)
- [`requirements.txt`](requirements.txt) 中的 Python 包：`numpy`、`torch`、`tensorboard`、`tqdm`、`optuna`、`stable-baselines3`、`gymnasium`、`squaternion`

```shell
pip install -r requirements.txt
```

#### 克隆并编译

```shell
cd ~
git clone https://github.com/MayaCHEN-github/HierarchicalRL-robot-navigation.git
cd HierarchicalRL-robot-navigation/catkin_ws
catkin_make_isolated
```

策略也可以用二维激光，但本仓库使用仿真三维 Velodyne。

#### 每个新终端都需要 source

```shell
export ROS_HOSTNAME=localhost
export ROS_MASTER_URI=http://localhost:11311
export ROS_PORT_SIM=11311
export GAZEBO_RESOURCE_PATH=~/HierarchicalRL-robot-navigation/catkin_ws/src/multi_robot_scenario/launch
source ~/HierarchicalRL-robot-navigation/catkin_ws/devel_isolated/setup.bash
```

### 训练

```shell
cd ~/HierarchicalRL-robot-navigation/TD3

# 推荐首选：SB3 TD3 基线
python train_sb3_td3.py

# 原版自定义 TD3
python train_velodyne_td3.py

# 分层实验（可能不收敛）
python train_hierarchical.py

# 可选：Optuna 超参搜索（模块独立入口）
python hierarchical_rl.py --optimize
```

分层训练常用参数：`--environment_dim 20`、`--max_timesteps 2000000`、`--eval_freq 5000`、`--device cuda|cpu`、`--load_high_level`、`--load_low_level`。

监控：

```shell
tensorboard --logdir ~/HierarchicalRL-robot-navigation/TD3/logs
```

> [!TIP]
> 训练异常退出后若 Gazebo/ROS 残留，可执行：
> `killall -9 rosout roslaunch rosmaster gzserver nodelet robot_state_publisher gzclient python python3`

### 架构

```
HierarchicalRL-robot-navigation/
├── catkin_ws/     # ROS 工作空间：Pioneer3DX、Gazebo 世界、Velodyne 插件
├── TD3/           # 训练代码、Gym 包装、分层 RL
├── repo-docs/     # 中文项目导读
└── README.md
```

**分层循环（DQN + TD3）**

1. 高层 DQN 从 **200** 个离散动作（20 个方向 × 10 档距离）中选子目标。
2. 方向索引转为弧度，拼到 24 维基础观测后面。
3. 低层 TD3 输出连续 `(线速度, 角速度)`。
4. Gazebo 步进 0.1 s；高层/低层奖励分别写入 buffer，在样本超过 1000 后每 100 步更新一次。

高层奖励包含方向匹配、距离匹配、基于激光的避障、路径平滑、碰撞与时间惩罚。低层奖励叠加 `GazeboEnv.get_reward`（到达 +100，碰撞 −100）。

**评估指标**

| 指标 | 含义 |
| --- | --- |
| 成功率 | 到达目标（距离 < 0.3 m） |
| 路径效率 | 直线距离 / 实际轨迹长度 |
| 轨迹平滑度 | 平均曲率变化的倒数 |
| 时间成本 | 到达或终止所用秒数 |
| 碰撞率 | 以碰撞结束的回合比例（环境奖励 < −90） |

### 文档

| 资源 | 说明 |
| --- | --- |
| [repo-docs/README.md](repo-docs/README.md) | 中文导读入口与阅读路径 |
| [端到端训练走读](repo-docs/walkthroughs/01-end-to-end-training.md) | 从编译到训练与产物 |
| [分层强化学习](repo-docs/modules/hierarchical-rl.md) | 两层循环与奖励 |
| [仿真环境](repo-docs/modules/gazebo-environment.md) | 传感器、包装器、场景 |
| [训练入口](repo-docs/modules/training-scripts.md) | 如何选择 `train_*.py` |
| [观测 / 动作 / 奖励](repo-docs/references/observation-action-reward.md) | 维度与公式查表 |
| [AGENTS.md](AGENTS.md) | 给编码 agent 的英文地图 |

### 引用

上游 DRL-robot-navigation 论文：

```bibtex
@ARTICLE{9645287,
  author={Cimurs, Reinis and Suh, Il Hong and Lee, Jin Han},
  journal={IEEE Robotics and Automation Letters},
  title={Goal-Driven Autonomous Exploration Through Deep Reinforcement Learning},
  year={2022},
  volume={7},
  number={2},
  pages={730-737},
  doi={10.1109/LRA.2021.3133591}}
```

论文链接：[IEEE Xplore](https://ieeexplore.ieee.org/document/9645287)。
