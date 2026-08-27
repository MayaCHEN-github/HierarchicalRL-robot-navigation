# 观测、动作与奖励

本页为查表用；机制说明见各模块页。

证据状态：除特别标注外，本页基于当前源码已确认。

## 观测空间

### 基础观测（GazeboEnv / VelodyneGymWrapper）

| 索引 | 字段 | 范围 | 说明 |
| --- | --- | --- | --- |
| 0 – 19 | `laser_i` | [0, 10] | 第 i 扇区最近障碍距离（米），无检测为 10 |
| 20 | `distance_to_goal` | [0, 7] | 到目标欧氏距离 |
| 21 | `theta_to_goal` | [-π, π] | 目标相对航向 |
| 22 | `linear_vel` | [0, 1] | 当前线速度 |
| 23 | `angular_vel` | [-1, 1] | 当前角速度 |

维度：**24**（`environment_dim=20` 时）

### 分层低层扩展观测

在基础 24 维后追加：

| 索引 | 字段 | 范围 | 说明 |
| --- | --- | --- | --- |
| 24 | `subgoal_direction` | [-π, π] | 高层方向扇区转为弧度：`(direction / environment_dim) * 2π - π` |
| 25 | `subgoal_distance` | [0.5, 5.0] | `(action // environment_dim) * 0.5 + 0.5` |

维度：**26**

## 动作空间

### 连续（TD3 / 分层低层 / SB3 TD3）

| 维度 | 含义 | 范围 |
| --- | --- | --- |
| 0 | 线速度 `v` | [0, 1] m/s |
| 1 | 角速度 `ω` | [-1, 1] rad/s |

### 离散 DQN（高层）

| 项目 | 值 |
| --- | --- |
| 动作数 | 200 = 20 方向 × 10 距离档 |
| 方向解码 | `direction = action % environment_dim`（默认 20） |
| 距离解码 | `distance = (action // environment_dim) * 0.5 + 0.5` |
| 低层方向输入 | `direction_rad = (direction / environment_dim) * 2π - π` |

### 离散 DQN（单层，VelodyneGymWrapper）

| 项目 | 值 |
| --- | --- |
| 动作数 | 20 = 4 线速度 × 5 角速度 |
| 线速度集合 | `{0.0, 0.3, 0.6, 0.9}` |
| 角速度集合 | `{-1.0, -0.5, 0.0, 0.5, 1.0}` |

### 自定义 TD3 Actor 输出映射

Actor `tanh` 输出 [-1,1]²，环境步前映射为：

```
v_env = (action[0] + 1) / 2
ω_env = action[1]
```

## 奖励与终止

### 环境奖励（GazeboEnv.get_reward）

| 事件 | 奖励 | 终止 |
| --- | --- | --- |
| 到达目标 | +100 | 是 |
| 碰撞（min_laser < 0.35） | -100 | 是 |
| 普通步 | 见模块页公式 | 否 |

阈值常量：

| 常量 | 值 | 文件 |
| --- | --- | --- |
| `GOAL_REACHED_DIST` | 0.3 m | velodyne_env.py |
| `COLLISION_DIST` | 0.35 m | velodyne_env.py |
| `TIME_DELTA` | 0.1 s | velodyne_env.py |

### 高层自定义奖励权重

| 符号 | 权重/值 |
| --- | --- |
| 方向 `direction_reward` | × 0.4 |
| 距离 `distance_reward` | × 0.4 |
| 避障 `obstacle_avoidance_reward` | × 0.1（激光最小值 ≥ 0.5 m 时该项为 0.2；环境仍不填充 `info['obstacle_ahead']`） |
| 平滑 `smoothness_reward` | × 0.1 |
| 碰撞惩罚 | 非成功结束记 1.0，再从总分减去 |
| 时间惩罚 | 每步 0.01，从总分减去 |
| 下限裁剪 | max(reward, -2.0) |

### 低层自定义奖励

```
R_low = R_env + 0.5 * (R_dir + R_dist) - 0.5 * P_collision
```

## 评估指标

| 指标 | 定义（源码） |
| --- | --- |
| 成功率 | 终点距目标 < 0.3 m 的回合比例 |
| 碰撞率 | 任一步 `reward < -90` 的回合比例 |
| 路径效率 | 成功回合：`直线距离 / 实际路径长度`，失败为 0 |
| 轨迹平滑度 | `1 / (1 + 平均转角变化)` |
| 时间成本 | 回合墙钟时间（秒） |

## 训练超参默认值（分层）

| 参数 | 默认值 |
| --- | --- |
| `max_timesteps` | 2,000,000 |
| `max_ep`（每回合最大步） | 500 |
| `learn_starts` | 1000 |
| `train_freq` | 100 |
| `eval_freq` | 5000 |
| DQN `batch_size` | 64 |
| TD3 `batch_size` | 40 |
| DQN `gamma` | 0.99 |
| TD3 `gamma` | 0.99999 |
