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
- Ships local table + red cube models under `models/` so the VX300S env can
  spawn the same workspace objects without depending on the RX200 package.

## Prerequisites

- ROS Noetic
- `interbotix_xsarm_descriptions` (Trossen — installed via the usual
  `xsarm_amd64_install.sh` script)
- `interbotix_xsarm_moveit_interface` (same)
- `interbotix_xsarm_gazebo` (same)
- `common_sensors` (Kinect v2 xacro)

## Verify the sim works

```bash
roscore                                                          # term 1
roslaunch viperx300s_description vx300s_gazebo.launch            # term 2
```

Expected:
- Gazebo opens with the table at `x=0.2`, red cube on top, and `vx300s`
  mounted at `z=0.78`.
- `rostopic echo /vx300s/joint_states` publishes the arm, gripper prop,
  and finger joints at ~50 Hz.
- `rostopic list | grep /vx300s/arm_controller` shows `/command`
  + state topics.

To bring up MoveIt against the same Gazebo controllers:

```bash
roslaunch viperx300s_description vx300s_gazebo.launch start_moveit:=true
```

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
├── config/
│   └── vx300s_controller.yaml     # PID gains for arm + gripper
└── models/
    ├── table/                     # cafe table model
    └── block/                     # 4 cm red cube
```

## Why this exists

Catkin doesn't allow nested packages. Dropping URDFs / launch files into
`rl_environments` works but bloats an RL-focused package and prevents
reusing the description for MoveIt / RViz demos. Separate description
package is the canonical ROS pattern; same shape as
`niryo_ned2_description_extras` (NED2) and `reactorx200_description`
(RX200).
