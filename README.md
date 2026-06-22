# SUPER_DRONE

`SUPER_DRONE` is a complete ROS1 source workspace snapshot built around `hku-mars/SUPER`, extended with the real-flight packages required for a `Livox MID-360 + FAST_LIO + px4ctrl` stack.

This repository keeps `SUPER` as the main planner, keeps the `SUPER_DRONE` mission layer added earlier, and vendors the required real-flight packages from the local `REAL_DRONE_400` snapshot so the workspace now contains planning, mapping, control, and optional camera drivers in one repo.

## 1. Added Workspace Directories

The following package groups were added into this repository.

### 1.1 `realflight_modules/`

Added from `D:\super\REAL_DRONE_400\src\realflight_modules`:

- `realflight_modules/mid360_fastlio/FAST_LIO`
- `realflight_modules/mid360_fastlio/livox_ros_driver`
- `realflight_modules/mid360_fastlio/livox_ros_driver2`
- `realflight_modules/px4ctrl`
- `realflight_modules/realsense-ros`

### 1.2 `utils/`

Added from `D:\super\REAL_DRONE_400\src\utils`:

- `utils/uav_utils`
- `utils/cmake_utils`

`cmake_utils` was added because the vendored `uav_utils` and the existing `quadrotor_msgs` manifest both depend on it.

## 2. Why `REAL_DRONE_400/utils/quadrotor_msgs` Was Not Copied

This workspace now keeps only one `quadrotor_msgs` package:

- `mars_uav_sim/mars_quadrotor_msgs`

`REAL_DRONE_400/utils/quadrotor_msgs` was not copied because it would conflict with the existing `SUPER` message package by package name.

Instead, the `SUPER` message package was extended to satisfy both sides.

## 3. Changes Made to SUPER

The following existing `SUPER` files were modified.

### 3.1 Unified message package

- `mars_uav_sim/mars_quadrotor_msgs/CMakeLists.txt`
- `mars_uav_sim/mars_quadrotor_msgs/ros/ros1.CMakeLists.txt`
- `mars_uav_sim/mars_quadrotor_msgs/package.xml`
- `mars_uav_sim/mars_quadrotor_msgs/ros/ros1.package.xml`

These files were updated so `SUPER` and `px4ctrl` share the same `quadrotor_msgs` package.

### 3.2 Existing SUPER_DRONE integration files already kept in use

The previously added `SUPER_DRONE` planner and mission files remain the default interfaces for the planning stack:

- `super_planner/config/super_drone_ros1.yaml`
- `mission_planner/config/super_drone_waypoint.yaml`
- `mission_planner/data/super_drone_competition_template.txt`
- `mission_planner/launch/super_drone.launch`

## 4. New Files Added by This Expansion

### 4.1 Message file added into SUPER

- `mars_uav_sim/mars_quadrotor_msgs/msg/Px4ctrlDebug.msg`

This was added because `px4ctrl` depends on `quadrotor_msgs/Px4ctrlDebug` but `SUPER` did not provide it.

### 4.2 FAST_LIO compatibility layer

- `realflight_modules/mid360_fastlio/FAST_LIO/include/livox_msg_compat.hpp`

This compatibility header aliases the old `livox_ros_driver` message namespace to `livox_ros_driver2`, so the default `MID-360` chain can use `livox_ros_driver2` without adding an extra bridge node.

### 4.3 livox_ros_driver2 ROS package metadata

- `realflight_modules/mid360_fastlio/livox_ros_driver2/package.xml`

This file was added because the local snapshot did not include a ROS package manifest for `livox_ros_driver2`.

### 4.4 Integrated launch entry

- `mission_planner/launch/super_drone_realflight.launch`

This is the full real-flight launch entry for:

- `livox_ros_driver2`
- `FAST_LIO`
- `px4ctrl`
- `super_drone_mission`
- `fsm_node`

## 5. Default Runtime Chain

Default runtime data flow:

`MID-360 -> livox_ros_driver2 -> FAST_LIO -> SUPER -> px4ctrl`

Key default topics:

- Driver inputs/outputs:
  - `/livox/lidar`
  - `/livox/imu`
- FAST_LIO outputs:
  - `/cloud_registered`
  - `/Odom_high_freq`
- Mission to planner handoff:
  - `/planning/click_goal`
- Planner to controller:
  - `/position_cmd`
- Takeoff trigger:
  - `/traj_start_trigger`
- Landing command:
  - `/px4ctrl/takeoff_land`

## 6. Launch Files

### 6.1 Lightweight planner entry

- `mission_planner/launch/super_drone.launch`

This starts only the `SUPER_DRONE` mission node and the `SUPER` planner node.

### 6.2 Full real-flight entry

- `mission_planner/launch/super_drone_realflight.launch`

This starts the full chain:

- `livox_ros_driver2/launch/msg_MID360.launch`
- `fast_lio/launch/mapping_mid360.launch`
- `px4ctrl/launch/run_ctrl.launch`
- `mission_planner/launch/super_drone.launch`

## 7. Notes About Optional Packages

- `realsense-ros` is present because you asked to vendor the full `realflight_modules` block.
- The default integrated launch does not use `realsense-ros`.
- The repository now contains both `livox_ros_driver` and `livox_ros_driver2`, but the default `MID-360` chain is `driver2`.

## 8. External System Dependencies

These are still external system dependencies and are not vendored as source packages here:

- ROS1 Noetic
- `mavros`
- `apr`
- `PCL`
- `Eigen`
- other standard ROS dependencies required by the vendored packages

## 9. Build and Run

Use this repository as a ROS1 source tree inside your catkin workspace, then launch the full chain with:

```bash
roslaunch mission_planner super_drone_realflight.launch
```