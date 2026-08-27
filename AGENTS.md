# Repo docs

Chinese reader guide lives in `repo-docs/`. Start at `repo-docs/README.md`, then follow `repo-docs/walkthroughs/01-end-to-end-training.md` for the first training run.

## Project snapshot

- **Stack**: ROS Noetic + Gazebo simulation, PyTorch, Stable-Baselines3, Gymnasium wrapper around a Velodyne-equipped Pioneer3DX (`r1`).
- **Primary training code**: `TD3/` (environment: `velodyne_env.py`, wrapper: `gym_wrapper.py`, hierarchical stack: `hierarchical_rl.py`).
- **ROS workspace**: `catkin_ws/` — compile with `catkin_make_isolated`, source `devel_isolated/setup.bash` before training.

## Maturity (do not overclaim)

| Track | Status |
| --- | --- |
| Single-agent TD3 (`train_sb3_td3.py`, `train_velodyne_td3.py`) | Stable baseline per project README and course paper |
| Hierarchical DQN+TD3 (`train_hierarchical.py`) | Runnable but unstable; no trustworthy quantitative win over TD3 yet |
| Optuna in `hierarchical_rl.py` | Present; not wired through `train_hierarchical.py` |
| Device selection | `HierarchicalRL` honors `device`; falls back to CUDA/CPU auto-detect |
| Paper mentions PyBullet / PathBench | Not reflected in current source (Gazebo + ROS only) |

## Recommended commands

```shell
cd catkin_ws && catkin_make_isolated
export GAZEBO_RESOURCE_PATH=~/HierarchicalRL-robot-navigation/catkin_ws/src/multi_robot_scenario/launch
source devel_isolated/setup.bash
cd ../TD3 && python train_sb3_td3.py   # stable baseline first
```

Hierarchical experiment: `python train_hierarchical.py` (expect instability).

## Doc map

| Path | Role |
| --- | --- |
| `repo-docs/README.md` | Chinese hub + honest scope |
| `repo-docs/walkthroughs/01-end-to-end-training.md` | Install → train → artifacts |
| `repo-docs/modules/hierarchical-rl.md` | DQN+TD3 loop, rewards, known gaps |
| `repo-docs/modules/gazebo-environment.md` | GazeboEnv + VelodyneGymWrapper |
| `repo-docs/modules/training-scripts.md` | Which `train_*.py` to use |
| `repo-docs/references/` | Observation/action/reward tables, commands |
| `repo-docs/glossary.md` | Chinese handles ↔ identifiers |

## Agent editing rules

- Keep `AGENTS.md` in English; keep reader-facing guides in Chinese under `repo-docs/`.
- Preserve exact identifiers (paths, commands, API names, metric keys) when editing docs.
- When describing behavior, cite source files; flag unimplemented fields (e.g. env still does not set `info['obstacle_ahead']`; hierarchical reward now falls back to lidar range) and README/paper mismatches.
- Minimize code changes unless the task is implementation, not documentation.

## Cleanup

If Gazebo/ROS linger after training:

```shell
killall -9 rosout roslaunch rosmaster gzserver nodelet robot_state_publisher gzclient python python3
```

`hierarchical_rl.py` also registers `_cleanup_gazebo_ros()` on exit.
