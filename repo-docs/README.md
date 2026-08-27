# HierarchicalRL-robot-navigation 项目导读

本项目在 ROS Gazebo 仿真中训练移动机器人导航策略。核心思路是把导航拆成两层：高层用离散决策选子目标，低层用连续控制执行运动。仓库同时保留单层 TD3 基线与多种训练入口，便于对比分层方案是否带来收益。

项目基于课程小组仓库 [zacz08/DRL-robot-navigation](https://github.com/zacz08/DRL-robot-navigation)（再 fork 改进），该小组仓库又基于 [DRL-robot-navigation](https://github.com/reiniscimurs/DRL-robot-navigation)。仿真平台为 ROS Noetic + Gazebo，算法侧使用 PyTorch 与 Stable-Baselines3（SB3）。

**证据状态：除特别标注外，本页基于当前源码已确认。**

## 当前成熟度（请如实预期）

根目录 [README.md](../README.md) 只做项目介绍与上手命令。成熟度、已知限制与论文/实现差异见本节，不放在首页。

根据仓库 README、课程论文 PDF 与源码对照，当前状态可概括为：

| 能力 | 状态 |
| --- | --- |
| 单层 TD3（自定义实现 / SB3） | 有较完整训练与评估链路，论文报告收敛较稳定 |
| DQN + TD3 分层训练 | 可运行，但训练不稳定，尚无可复现的定量对比结论 |
| 超参数搜索（Optuna） | 代码存在，未与主训练流程深度集成 |
| 论文中的 PathBench | 论文提及，源码中未见对应集成 |
| 论文中的 PyBullet | 论文方法草图提及，实际仿真为 Gazebo + ROS |

下文文档按「能跑通什么」来写，不夸大分层方案已验证的效果。

## 文档地图

| 页面 | 适合谁 | 作用 |
| --- | --- | --- |
| [端到端训练走读](walkthroughs/01-end-to-end-training.md) | 第一次上手 | 从环境编译到选训练脚本、看日志与模型产物 |
| [分层强化学习模块](modules/hierarchical-rl.md) | 想改 DQN+TD3 的人 | 两层智能体如何采样、算奖励、写 buffer |
| [仿真环境模块](modules/gazebo-environment.md) | 想改观测/奖励/场景的人 | Gazebo 交互、Gym 包装、状态与奖励构成 |
| [训练入口模块](modules/training-scripts.md) | 想选算法或对比基线的人 | 各 `train_*.py` 的差异与适用场景 |
| [观测、动作与奖励](references/observation-action-reward.md) | 查表 | 维度、范围、奖励公式与阈值 |
| [命令与产物](references/commands-and-artifacts.md) | 运维/复现 | 编译、运行、清理、日志与模型路径 |
| [术语表](glossary.md) | 通读前速查 | 中文句柄与英文标识符对照 |

## 阅读路径

| 你的目标 | 建议顺序 |
| --- | --- |
| 尽快跑通单层 TD3 基线 | 本页 → [命令与产物](references/commands-and-artifacts.md) → [训练入口](modules/training-scripts.md) → [端到端走读](walkthroughs/01-end-to-end-training.md) Step 1–4 |
| 理解并调试 DQN+TD3 分层 | 本页 → [仿真环境](modules/gazebo-environment.md) → [分层 RL](modules/hierarchical-rl.md) → [观测动作奖励](references/observation-action-reward.md) |
| 改奖励或评估指标 | [仿真环境](modules/gazebo-environment.md) → [分层 RL](modules/hierarchical-rl.md) 奖励段 → [观测动作奖励](references/observation-action-reward.md) |
| 排查 Gazebo/ROS 起不来 | [命令与产物](references/commands-and-artifacts.md) 环境变量与清理 → [端到端走读](walkthroughs/01-end-to-end-training.md) Step 2 |

## 仓库顶层结构（速览）

```
HierarchicalRL-robot-navigation/
├── catkin_ws/          # ROS 工作空间：机器人模型、Gazebo 世界、Velodyne 插件
├── TD3/                # Python 训练代码：环境包装、分层 RL、各训练脚本
├── repo-docs/          # 本中文文档
└── README.md           # 中英双语项目说明与安装步骤
```

变更记录见 [change-log.md](change-log.md)。
