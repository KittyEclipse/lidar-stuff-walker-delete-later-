# ROS

ROS 2 Humble on Ubuntu 22.04: an RPLIDAR A1 mapped with slam_toolbox, on real hardware or in Gazebo Harmonic.

- `lidar_sim/` drives a Sourccey Mk.V with a simulated lidar around a room and builds a map.
- `lidar_slam/` maps with a real, handheld RPLIDAR A1. It also holds the SLAM settings and the Cyclone DDS config, which the sim shares, so keep the two folders side by side.

## Install

    sudo apt install ros-humble-slam-toolbox ros-humble-navigation2 ros-humble-nav2-bringup \
      ros-humble-rmw-cyclonedds-cpp ros-humble-ros-gzharmonic ros-humble-teleop-twist-keyboard

Gazebo Harmonic comes from the OSRF package repository.

For the real lidar, also build the Slamtec driver in a colcon workspace:

    git clone https://github.com/Slamtec/sllidar_ros2.git src/sllidar_ros2
    colcon build --symlink-install --packages-select sllidar_ros2

## Run the sim

From this folder:

    source /opt/ros/humble/setup.bash
    export ROS_LOCALHOST_ONLY=1 RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
    ros2 launch lidar_sim/lidar_sim.launch.py

Close the Gazebo window to stop it. Only one sim can run at a time, so the launch refuses to start if Gazebo is already running.

Options: `drive:=false` to steer it yourself with `teleop_twist_keyboard`, `gui:=false`, `rviz:=false`, `gpu:=false`.

## Run the real lidar

    source /opt/ros/humble/setup.bash
    source install/setup.bash   # the workspace holding sllidar_ros2
    export ROS_LOCALHOST_ONLY=1 RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
    ros2 launch lidar_slam/lidar_slam.launch.py serial_port:=/dev/ttyUSB0

## Notes

- `lidar_sim/sourccey/` is the unchanged `URDF/FullBody` folder from [vulcan-forge/sourccey-hardware](https://github.com/vulcan-forge/sourccey-hardware) at commit `dbb4bfd`. That repo has no licence file, so check with Vulcan Forge before making this repo public.
- `lidar_sim/sourccey_sim.urdf` and `.sdf` are the robot the sim uses, with the lidar added and the arms and lift column removed. `make_sourccey.py` generates them. Their mesh paths start with a `SOURCCEY_DIR` placeholder that the launch file fills in, so open them through the launch rather than directly.
- `lidar_slam/cyclonedds.xml` works around a WSL2 loopback problem that makes large messages disappear, and any other terminal on WSL2 needs it too: `export CYCLONEDDS_URI=file://$PWD/lidar_slam/cyclonedds.xml`
- Under WSL, Gazebo is told to use the GPU through a Mesa OpenGL version override. Elsewhere it uses the normal driver.
