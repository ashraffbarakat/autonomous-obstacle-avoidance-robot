# Autonomous Obstacle-Avoidance Mobile Robot

<p align="center">
  <img src="cover.jpeg" width="100%">
</p>


<p align="center">
  <strong>ROS 2 Jazzy · Gazebo Harmonic · Python · 360° GPU LiDAR</strong>
</p>

<p align="center">
  An autonomous differential-drive robot that navigates a custom maze using real-time LiDAR perception and a reactive obstacle avoidance algorithm.
</p>



<p align="center">
  <img src="https://img.shields.io/badge/ROS_2-Jazzy-22314E?logo=ros" alt="ROS 2 Jazzy">
  <img src="https://img.shields.io/badge/Simulation-Gazebo_Harmonic-orange" alt="Gazebo Harmonic">
  <img src="https://img.shields.io/badge/Language-Python_3-blue?logo=python" alt="Python 3">
  <img src="https://img.shields.io/badge/Status-Completed-success" alt="Completed">
  <img src="https://img.shields.io/badge/Platform-Ubuntu_24.04-E95420?logo=ubuntu" alt="Ubuntu 24.04">
</p>

---

## Overview

This project presents a simulated autonomous mobile robot built with **ROS 2 Jazzy** and **Gazebo Harmonic**.

The robot uses a 360° 2D GPU LiDAR sensor to perceive its surroundings and navigate through a custom 14 × 14 maze. A custom Python-based reactive obstacle avoidance controller continuously processes LiDAR measurements, identifies obstacles, evaluates available paths, and generates velocity commands.

Unlike map-based navigation systems, this implementation operates without SLAM, mapping, or Nav2. Navigation decisions are made directly from real-time sensor measurements.

The project also addresses practical simulation integration challenges, including ROS-Gazebo communication, TF frame consistency, and synchronized system startup.

## Simulation Demo



*Replace the placeholder link with the actual hosted video URL.*

## Key Features

- **Autonomous Navigation:** Reactive obstacle avoidance without a predefined map.
- **360° LiDAR Perception:** Continuous environmental sensing using a GPU-based 2D LiDAR.
- **Dynamic Direction Selection:** Evaluates left and right free-space distances to select a navigation direction.
- **Hysteresis-Based Direction Lock:** Reduces unnecessary switching between turning directions.
- **Dead-End Recovery:** Successfully tested escape behavior for U-shaped dead ends.
- **Custom Simulation Environment:** A 14 × 14 maze developed in Gazebo.
- **ROS 2 Integration:** Communication between Gazebo and ROS 2 through `ros_gz_bridge`.
- **Live Visualization:** Robot model, LiDAR scans, TF frames, and odometry displayed in RViz2.
- **Unified Launch:** Starts the complete simulation using a single ROS 2 launch command.

## System Architecture

The system is divided into three ROS 2 packages, each responsible for a specific part of the simulation.

<p align="center">
  <img src="media/architecture.png" alt="System Architecture" width="90%">
</p>

### Package Structure

| Package | Build Type | Responsibility |
|---|---|---|
| `edges_description` | `ament_cmake` | Robot description, Gazebo plugins, maze world, and RViz configuration |
| `obstacle_avoidance_controller` | `ament_python` | LiDAR processing and autonomous obstacle avoidance |
| `edges_bringup` | `ament_cmake` | Simulation orchestration and unified launch system |

### Workspace Structure

```text
modeling_ws/
├── src/
│   ├── edges_description/
│   │   ├── urdf/
│   │   │   └── edges_mobile_robot.xacro
│   │   ├── worlds/
│   │   │   └── obstacle_world.sdf
│   │   ├── rviz/
│   │   │   └── edges_config.rviz
│   │   └── ...
│   │
│   ├── obstacle_avoidance_controller/
│   │   ├── obstacle_avoidance_controller/
│   │   │   └── obstacle_avoidance_node.py
│   │   ├── config/
│   │   │   └── obstacle_avoidance.yaml
│   │   └── ...
│   │
│   └── edges_bringup/
│       ├── launch/
│       │   └── autonomous_robot.launch.py
│       └── ...
│
└── ...
```

*The tree above illustrates the main package organization. Exact paths should be aligned with the final source tree.*

## Obstacle Avoidance Algorithm

The robot implements a custom reactive navigation algorithm based on real-time LiDAR measurements.

### 1. Sensor Processing

The controller subscribes to `/scan` and analyzes the LiDAR readings to estimate obstacle distances around the robot.

### 2. Front Obstacle Detection

A forward field of view of ±30° is monitored.

- Safe distance threshold: **0.85 m**
- Forward linear velocity: **0.45 m/s**

When the front path is clear, the robot continues moving forward.

### 3. Direction Selection

When an obstacle is detected, the controller compares the average available free distance on the left and right sides.

The direction with more available space is selected for the avoidance maneuver.

### 4. Direction Lock and Hysteresis

A direction-lock mechanism with hysteresis prevents rapid switching between left and right decisions.

This improves the consistency of turning behavior in narrow passages and around nearby obstacles.

### 5. U-Shaped Dead-End Recovery

The controller includes a recovery behavior for U-shaped dead ends.

When the robot cannot continue through the current path, it performs a 180° turning maneuver to escape the dead end.

<details>
<summary><strong>Algorithm Parameters</strong></summary>

| Parameter | Value |
|---|---:|
| Front detection angle | ±30° |
| Safe obstacle distance | 0.85 m |
| Forward linear velocity | 0.45 m/s |
| Angular velocity | 0.9 rad/s |
| Dead-end recovery rotation | 180° |

</details>

## ROS 2 Communication

The simulation uses `ros_gz_bridge` to exchange data between Gazebo and ROS 2.

| Topic | Direction / Purpose | Description |
|---|---|---|
| `/scan` | Gazebo → ROS 2 | LiDAR measurements |
| `/cmd_vel` | ROS 2 → Gazebo | Robot velocity commands |
| `/odom` | Gazebo → ROS 2 | Odometry information |
| `/tf` | ROS 2 / Gazebo integration | Coordinate transforms |
| `/joint_states` | Gazebo → ROS 2 | Wheel joint states |
| `/clock` | Gazebo → ROS 2 | Simulation clock |

## Visualization

RViz2 is configured to monitor the robot's state and sensor data during navigation.

<p align="center">
  <img src="media/screenshots/rviz_visualization.png" alt="RViz2 Visualization" width="90%">
</p>

The visualization includes:

- Robot model and link structure.
- Live 360° LiDAR scan.
- TF coordinate frames.
- Odometry information.

## Engineering Implementation

### TF Tree Integration

The differential-drive plugin is configured with:

```text
frame_id: odom
child_frame_id: base_footprint
```

The joint state publisher and `/joint_states` bridge are integrated to maintain the robot's joint information and visualization in RViz2.

### Simulation Startup Synchronization

A delayed controller startup is implemented using `TimerAction`.

This ensures that Gazebo, robot spawning, and the required communication bridges have sufficient time to initialize before the autonomous controller starts.

### Gazebo and ROS 2 Integration

The `ros_gz_bridge` package connects the simulation environment with ROS 2 topics, allowing sensor data to reach the controller and velocity commands to control the simulated robot.

## Technology Stack

| Category | Technology |
|---|---|
| Robotics Middleware | ROS 2 Jazzy |
| Simulation | Gazebo Harmonic |
| Programming | Python 3 |
| Robot Description | URDF / Xacro |
| Simulation World | SDF |
| Visualization | RViz2 |
| Communication | `ros_gz_bridge` |
| Build System | Colcon / CMake |
| Operating System | Ubuntu 24.04 |

## Installation & Execution

### Prerequisites

- Ubuntu 24.04
- ROS 2 Jazzy
- Gazebo Harmonic
- ROS-Gazebo integration packages
- Colcon build tools

### 1. Clone the Repository

```bash
git clone https://github.com/YOUR_USERNAME/autonomous-obstacle-avoidance-robot.git
cd autonomous-obstacle-avoidance-robot
```

### 2. Build the Workspace

```bash
colcon build --symlink-install
```

### 3. Source the Workspace

```bash
source install/setup.bash
```

### 4. Launch the Simulation

```bash
ros2 launch edges_bringup autonomous_robot.launch.py
```

The launch file brings up the Gazebo world, spawns the robot, initializes the required bridges, starts RViz2, and launches the obstacle avoidance controller.

## Project Status

<p align="center">
  <img src="https://img.shields.io/badge/Development-Completed-success" alt="Development Completed">
  <img src="https://img.shields.io/badge/Testing-Successful-success" alt="Testing Successful">
</p>

The project has been implemented and tested in simulation, including autonomous obstacle avoidance and U-shaped dead-end recovery.

## Future Development

Potential extensions include:

- Integrating SLAM for real-time map generation.
- Implementing ROS 2 Nav2 for map-based navigation.
- Testing the navigation algorithm on physical robot hardware.
- Evaluating navigation performance across more complex environments.

## Author

**Ashraf Barakat**

Individual Engineering Project

---

<p align="center">
  <sub>Built with ROS 2 · Simulated with Gazebo · Documented for robotics development</sub>
</p>
