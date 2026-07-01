# 术语表

列名格式：术语 | 项目里的意思 | 延伸阅读

| 术语 | 项目里的意思 | 延伸阅读 |
| --- | --- | --- |
| 分层强化学习（Hierarchical RL） | 高层选子目标、低层执行连续控制的二层结构；本仓库实现为 DQN + TD3 | [分层强化学习模块](../modules/hierarchical-rl.md) |
| 子目标（subgoal） | 高层每步输出的方向扇区与目标步长，拼入低层观测 | [观测、动作与奖励](../references/observation-action-reward.md) |
| Gazebo 环境（GazeboEnv） | 直接通过 ROS 话题驱动仿真的底层环境类 | [仿真环境模块](../modules/gazebo-environment.md) |
| Gym 包装器（VelodyneGymWrapper） | 把 GazeboEnv 适配为 Gymnasium API 的中间层 | [仿真环境模块](../modules/gazebo-environment.md) |
| 激光扇区（laser sector） | 将 360° Velodyne 点云按 `environment_dim` 分段后的最近距离 | [观测、动作与奖励](../references/observation-action-reward.md) |
| 经验回放（replay buffer） | 存储转移样本供离策略训练；分层时高层、低层各一份 | [分层强化学习模块](../modules/hierarchical-rl.md) |
| Stable-Baselines3（SB3） | 第三方 RL 库；本项目的 DQN/TD3 实现来源之一 | [训练入口模块](../modules/training-scripts.md) |
| TD3 | Twin Delayed DDPG，连续控制算法；作为低层执行器与单层基线 | [训练入口模块](../modules/training-scripts.md) |
| DQN | 离散动作价值学习；本仓库用于高层子目标选择或单层离散导航 | [分层强化学习模块](../modules/hierarchical-rl.md) |
| 自定义 TD3 | `train_velodyne_td3.py` 中手写 Actor-Critic，非 SB3 封装 | [训练入口模块](../modules/training-scripts.md) |
| 路径效率（path efficiency） | 成功回合中直线距离与实际轨迹长度之比 | [观测、动作与奖励](../references/observation-action-reward.md) |
| 轨迹平滑度（trajectory smoothness） | 基于轨迹转角变化的平滑指标，越大越平滑 | [分层强化学习模块](../modules/hierarchical-rl.md) |
| catkin 工作空间（catkin_ws） | ROS 编译与仿真资源所在目录 | [命令与产物](../references/commands-and-artifacts.md) |
| Pioneer3DX（r1） | Gazebo 中机器人模型名 | [仿真环境模块](../modules/gazebo-environment.md) |
| Optuna | 超参数自动搜索框架；分层代码中有试验性集成 | [分层强化学习模块](../modules/hierarchical-rl.md) |
| DRL-robot-navigation | 上游开源导航 RL 项目，本仓库在此基础上扩展 | 根目录 [README.md](../../README.md) |
