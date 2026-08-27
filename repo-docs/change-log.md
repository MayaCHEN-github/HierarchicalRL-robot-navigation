# 变更记录

## 2026-08-27 — 合并上游与双语 README

- 合并 `zacz08/DRL-robot-navigation:main` 的分层奖励重构：避障项改为激光最小值回退，碰撞/时间惩罚按减法计入。
- 保留本仓库后续修复：里程计位移、子目标方向用弧度、CUDA `device`、逐步写入高层奖励、更克制的 Gazebo 清理。
- 合并 `cursor/repo-docs-zh-a9d6` 中文文档树。
- 根目录 `README.md` 改为中英双语落地页（create-readme 技能）。

**验证**：`python3 -m py_compile TD3/hierarchical_rl.py`。本环境未启动 Gazebo/ROS，未跑完整训练。

## 2026-07-01 — 初始中文文档

- 新增 `repo-docs/` 中文导读体系：README、端到端走读、三个模块页、两份参考表、术语表。
- 新增根目录 `AGENTS.md`（英文），供自动化 agent 定位文档与训练入口。
- 文档如实标注项目不足：分层 DQN+TD3 不稳定、单层 TD3 为可靠基线、论文与实现差异（PyBullet/PathBench）、若干源码缺口（环境仍不填充 `obstacle_ahead` 等）。
- 更新根目录 `README.md`，把文档入口、训练路线、当前成熟度与实际仓库路径放到首页，避免继续引用旧的 `DRL-robot-navigation` 本地路径。

**验证**：仓库内无 `repo-docs` 校验脚本，本次未运行自动 validator。

**未覆盖区域**：

- `catkin_ws` 内 Velodyne 插件与 xacro 的逐文件说明
- 自定义 TD3 网络结构与 `replay_buffer.py` 细节
- Docker 部署（论文提及，仓库无 Dockerfile）
- 多机协同与真实机器人部署
