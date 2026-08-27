# 变更记录

## 2026-07-01 — 初始中文文档

- 新增 `repo-docs/` 中文导读体系：README、端到端走读、三个模块页、两份参考表、术语表。
- 新增根目录 `AGENTS.md`（英文），供自动化 agent 定位文档与训练入口。
- 文档如实标注项目不足：分层 DQN+TD3 不稳定、单层 TD3 为可靠基线、论文与实现差异（PyBullet/PathBench）、若干源码缺口（`obstacle_ahead`、强制 CPU 等）。
- 更新根目录 `README.md`，把文档入口、训练路线、当前成熟度与实际仓库路径放到首页，避免继续引用旧的 `DRL-robot-navigation` 本地路径。

**验证**：仓库内无 `repo-docs` 校验脚本，本次未运行自动 validator。

**未覆盖区域**：

- `catkin_ws` 内 Velodyne 插件与 xacro 的逐文件说明
- 自定义 TD3 网络结构与 `replay_buffer.py` 细节
- Docker 部署（论文提及，仓库无 Dockerfile）
- 多机协同与真实机器人部署
