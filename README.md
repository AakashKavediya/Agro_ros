# Agro_ros

ROS 2 (Humble) workspace for **AgroBot**, a tracked agricultural robot. It holds the robot's URDF model with CAD meshes, LiDAR-based odometry (rf2o) and 2D SLAM (slam_toolbox) configuration, and helper commands for visualising the TF tree.

![ROS 2](https://img.shields.io/badge/ROS_2-Humble-22314E?logo=ros)
![Build](https://img.shields.io/badge/build-ament__python-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Status](https://img.shields.io/badge/status-work_in_progress-orange)

## Features

- **Robot model**: URDF (`Agrobot`) with five STL meshes: body, tracks, track outer cover, water pump, water-level sensor.
- **LiDAR odometry**: [rf2o_laser_odometry](https://github.com/MAPIRlab/rf2o_laser_odometry) configured to turn `/scan` into `/odom` and publish `odom -> base_link`.
- **2D SLAM**: `slam_toolbox` in mapping mode (Ceres solver, loop closure, 5 cm resolution, 0.2–12 m laser range).
- **TF tree**: `map -> odom -> base_link -> {tracks, track_outer_cover, water_level_sensor, water_pump}`, with a `view_frames` snapshot included.

## Repository Structure

```
Agro_ros/
├── src/Agro_ros/               # ament_python package
│   ├── Agro_ros/               # Python module (placeholder)
│   ├── urdf/robot.urdf         # Robot description
│   ├── meshes/                 # STL parts (scaled 0.05 in URDF)
│   │   ├── agro_ros_body.stl
│   │   ├── agro_ros_tracks.stl
│   │   ├── agro_ros_tracks_outer_cover.stl
│   │   ├── agro_ros_water_pump.stl
│   │   └── agro_ros_water_level_sensor.stl
│   ├── slam_params.yaml        # slam_toolbox parameters
│   ├── package.xml · setup.py · setup.cfg · resource/ · test/
├── rf2o_params.yaml            # rf2o laser odometry parameters
├── command.txt                 # Handy ROS 2 commands
├── frames_2026-09-18_20.48.39.{gv,pdf}   # TF tree snapshot (view_frames)
└── LICENSE                     # MIT
```

> `build/`, `install/` and `log/` are colcon outputs and are regenerated on build.

## TF Tree

```
map ──► odom ──► base_link ─┬─► tracks
                            ├─► track_outer_cover
                            ├─► water_level_sensor
                            └─► water_pump
```

`map -> odom` comes from SLAM, `odom -> base_link` from rf2o (~7 Hz), and the `base_link` children are fixed joints from the URDF.

## Prerequisites

- Ubuntu 22.04 + [ROS 2 Humble](https://docs.ros.org/en/humble/Installation.html)
- `colcon`, `rviz2`
- A 2D LiDAR publishing `sensor_msgs/LaserScan` on `/scan`

```bash
sudo apt update
sudo apt install ros-humble-robot-state-publisher \
                 ros-humble-joint-state-publisher-gui \
                 ros-humble-slam-toolbox \
                 ros-humble-tf2-tools \
                 ros-humble-rviz2
```

rf2o is not bundled in this repo; clone it into the workspace `src/`:

```bash
git clone -b ros2 https://github.com/MAPIRlab/rf2o_laser_odometry.git src/rf2o_laser_odometry
```

## Build

```bash
git clone https://github.com/AakashKavediya/Agro_ros.git
cd Agro_ros
git clone -b ros2 https://github.com/MAPIRlab/rf2o_laser_odometry.git src/rf2o_laser_odometry

source /opt/ros/humble/setup.bash
colcon build
source install/setup.bash
```

## Usage

**1. Publish the robot model** (re-run after editing `robot.urdf`, then rebuild):

```bash
ros2 run robot_state_publisher robot_state_publisher src/Agro_ros/urdf/robot.urdf
```

**2. Inspect joints (optional):**

```bash
ros2 run joint_state_publisher_gui joint_state_publisher_gui
```

**3. Start LiDAR odometry:**

```bash
ros2 run rf2o_laser_odometry rf2o_laser_odometry_node --ros-args --params-file rf2o_params.yaml
```

**4. Start SLAM mapping:**

```bash
ros2 launch slam_toolbox online_async_launch.py slam_params_file:=src/Agro_ros/slam_params.yaml
```

**5. Visualise:** open `rviz2`, set Fixed Frame to `map`, and add **RobotModel**, **TF**, **LaserScan** (`/scan`) and **Map** (`/map`) displays.

**6. Dump the TF tree:**

```bash
ros2 run tf2_tools view_frames   # writes frames_<timestamp>.gv / .pdf
```

## Configuration

| File | Purpose | Key values |
|---|---|---|
| `rf2o_params.yaml` | rf2o odometry | `/scan` → `/odom`, `base_link`/`odom` frames, `publish_tf: true`, `freq: 7.0` |
| `src/Agro_ros/slam_params.yaml` | slam_toolbox | `mode: mapping`, Ceres solver, `resolution: 0.05`, laser range 0.20–12.0 m, loop closing on, min travel 0.05 m / 0.05 rad |
| `src/Agro_ros/urdf/robot.urdf` | Robot description | 5 links, all fixed joints to `base_link`, meshes scaled ×0.05 |

## Current Status & Known Issues

- Robot model loads in RViz; the robot moves with LiDAR odometry.
- **The `/map` is not yet being generated** by SLAM (latest commit).
- Joints are all `fixed` with zero origins; wheel/track motion is not modelled yet.
- `setup.py` installs `launch/*.py`, but no `launch/` folder exists yet.
- `package.xml` still has TODO description/license and `entry_points` is empty.

## Roadmap

- [ ] Fix map generation (check `/scan` frame, `odom -> base_link` TF timing, and slam_toolbox `scan_topic`)
- [ ] Add a launch file bringing up robot_state_publisher + rf2o + slam_toolbox + RViz
- [ ] Add a saved RViz config
- [ ] Model track/wheel joints and add differential-drive control
- [ ] Add sensors (IMU, camera) and water pump / level-sensor interfaces
- [ ] Add `.gitignore` for `build/ install/ log/`

## License

MIT, see [LICENSE](LICENSE). © 2026 Aakash Kavediya.
