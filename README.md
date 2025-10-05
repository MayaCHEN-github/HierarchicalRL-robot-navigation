# HierarchicalRL-robot-navigation

This project is based on **`DRL-robot-navigation`**, a deep reinforcement learning repository for mobile robot navigation in ROS Gazebo simulator.

The implementation supports multiple reinforcement learning algorithms including a hierarchical two-tier architecture (DQN + TD3), custom TD3 implementations, and Stable-Baselines3 integrations. gymnasium interface for this ROS-Gazebo environment is also implemented.

Trained in ROS Gazebo simulator with PyTorch.  Tested with ROS Noetic on Ubuntu 20.04 with python 3.8.10 and pytorch 1.10.

**Installation and code overview tutorial of DRL-robot-navigation available** [here](https://medium.com/@reinis_86651/deep-reinforcement-learning-in-mobile-robot-navigation-tutorial-part1-installation-d62715722303)

Training example:
<p align="center">
    <img width=100% src="https://github.com/reiniscimurs/DRL-robot-navigation/blob/main/training.gif">
</p>



**ICRA 2022 and IEEE RA-L paper(DRL-robot-navigation):**


Some more information about the implementation is available [here](https://ieeexplore.ieee.org/document/9645287?source=authoralert)

Please cite as:<br/>
```
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

## Installation
Main dependencies: 

* [ROS Noetic](http://wiki.ros.org/noetic/Installation)
* [PyTorch](https://pytorch.org/get-started/locally/)
* [Tensorboard](https://github.com/tensorflow/tensorboard)

Clone the repository:
```shell
$ cd ~
### Clone this repo
$ git clone https://github.com/MayaCHEN-github/HierarchicalRL-robot-navigation.git
```
The network can be run with a standard 2D laser, but this implementation uses a simulated [3D Velodyne sensor](https://github.com/lmark1/velodyne_simulator)

Compile the workspace:
```shell
$ cd DRL-robot-navigation/catkin_ws/
### Compile
$ catkin_make_isolated
```

Open a terminal and set up sources:
```shell
$ export ROS_HOSTNAME=localhost
$ export ROS_MASTER_URI=http://localhost:11311
$ export ROS_PORT_SIM=11311
$ export GAZEBO_RESOURCE_PATH=~/DRL-robot-navigation/catkin_ws/src/multi_robot_scenario/launch
$ source ~/.bashrc
$ cd ~/DRL-robot-navigation/catkin_ws
$ source devel_isolated/setup.bash
```

Run the training:
```shell
$ cd ~/DRL-robot-navigation/TD3

### Launches hierarchical RL training
$ python train_hierarchical.py

### or Launches Stable-Baselines3 TD3
$ python train_sb3_td3.py

### or Launches custom TD3 implementation in DRL-robot-navigation
$ python train_velodyne_td3.py
```

To mannually kill the training process:
```shell
$ killall -9 rosout roslaunch rosmaster gzserver nodelet robot_state_publisher gzclient python python3
```

Gazebo environment:
<p align="center">
    <img width=80% src="https://github.com/reiniscimurs/DRL-robot-navigation/blob/main/env1.png">
</p>

Rviz:
<p align="center">
    <img width=80% src="https://github.com/reiniscimurs/DRL-robot-navigation/blob/main/velodyne.png">
</p>

## Hierarchical RL System
The hierarchical approach decomposes the navigation problem into two learning levels:

### High-Level DQN Agent:
- Class: customized DQN from Stable-Baselines3
- Action Space: 200 discrete actions representing 20 directions × 10 distance levels 
- Responsibility: Strategic planning - selects navigation sub-goals
- Training Frequency: Every 100 steps when replay buffer > 1000 samples

### Low-Level TD3 Agent:
- Class: TD3 from Stable-Baselines3 
- Action Space: Continuous 2D actions (linear velocity, angular velocity) 
- Observation: Extended 26D state = 24D base + direction + distance 
- Responsibility: Tactical execution - generates robot control commands to achieve sub-goals
- Training Frequency: Every 100 steps when replay buffer > 1000 samples

## Evaluation Metrics
- **Success Rate**: Percentage of episodes reaching goal (distance < 0.3m)
- **Path Efficiency**: Ratio of straight-line distance to actual path traveled
- **Trajectory Smoothness**: Inverse of average curvature changes 
- **Time Cost**: Seconds to reach goal or episode termination
- **Collision Rate**: Percentage of episodes ending in collision (reward < -90)

