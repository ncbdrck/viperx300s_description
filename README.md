# viperx300s_description

Trossen ViperX-300 S sim extras for the `rl_environments` stack. Mirrors
`reactorx200_description`'s role for the RX200:

- Wraps the upstream `interbotix_xsarm_descriptions/urdf/vx300s.urdf.xacro`
  so we can ship Gazebo / scene additions locally without forking the
  arm definition.
- Adds a **head-mount Kinect v2** at the same pose `reactorx200_description`
  uses, so cube tracking + extrinsic calibration carry across the two
  Interbotix robots.
- Brings up **ros_control** with PID gains tuned for the 6-DOF arm
  (`waist, shoulder, elbow, forearm_roll, wrist_angle, wrist_rotate`)
  plus the 2-finger prismatic gripper (`left_finger, right_finger`).
- Reuses the `reactorx200_description` desk model (`models/table/`)
  so both Interbotix sims sit on the same surface.

## Prerequisites

- ROS Noetic
- `interbotix_xsarm_descriptions` (Trossen — installed via the usual
  `xsarm_amd64_install.sh` script)
- `interbotix_xsarm_moveit_interface` (same)
- `interbotix_xsarm_gazebo` (same)
- `reactorx200_description` (desk model)
- `common_sensors` (Kinect v2 xacro)

## Verify the sim works

```bash
roscore                                                          # term 1
roslaunch viperx300s_description vx300s_gazebo.launch            # term 2
```

Expected:
- Gazebo opens, table at origin, vx300s on top at `z=0.78`.
- `rostopic echo /joint_states` shows 8 joints publishing at ~50 Hz.
- `rostopic list | grep arm_controller` shows `/arm_controller/command`
  + state topics.

## Layout

```
viperx300s_description/
├── package.xml
├── CMakeLists.txt
├── README.md
├── urdf/
│   ├── vx300s.urdf.xacro          # wraps upstream interbotix vx300s
│   └── vx300s_kinect.urdf.xacro   # vx300s + head-mount kinect2
├── launch/
│   ├── vx300s_gazebo.launch       # world + table + arm + control
│   └── vx300s_control.launch      # joint_state + arm + gripper spawn
└── config/
    └── vx300s_controller.yaml     # PID gains for arm + gripper
```

## Why this exists

Catkin doesn't allow nested packages. Dropping URDFs / launch files into
`rl_environments` works but bloats an RL-focused package and prevents
reusing the description for MoveIt / RViz demos. Separate description
package is the canonical ROS pattern; same shape as
`niryo_ned2_description_extras` (NED2) and `reactorx200_description`
(RX200).
