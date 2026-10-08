# semantic-warehouse-sim-and-ros2-robot
# Warehouse Robotics: PyBullet Semantic Simulation and ROS2 Delivery Robot

MSc Artificial Intelligence, Robotics Software and Programming
Assignment: *Dynamic Robotics Simulation and Autonomous System Design*

This repository contains two linked projects built around a warehouse theme.

| Task | Tool | What it does |
|---|---|---|
| Task 1 | PyBullet (Google Colab) | A structured warehouse scene where every object has a semantic role, observed from three camera viewpoints |
| Task 2 | ROS2 Humble (TheConstruct.ai) | An autonomous delivery robot with a state machine, obstacle avoidance, keyboard control and status monitoring |

## Links

- Task 1 Colab notebook: `ADD PUBLIC COLAB LINK HERE`
- Task 2 Rosject name: `7Kd2mQ9xTn`

## Task 1: PyBullet warehouse with object semantics

The notebook builds the whole scene itself. Every URDF is defined as a Python string and written to disk, so no files need to be uploaded. Run it with **Runtime > Run all**.

**Objects (each with its own URDF and role)**

| Object | Role |
|---|---|
| Long shelf | `storage` |
| Restricted zone around each shelf | `no_entry` |
| Inbound zone | `inbound` |
| Conveyor belt | `conveyor` |
| Outbound packages | `outbound` |
| Charging station | `landmark` |
| Two-wheel robot | moves between camera frames |

**Features**

- PyBullet in DIRECT mode, with a `scene_objects` dictionary that stores each object's role and category
- Functions to list objects by role and describe any object by id
- A no-entry check that stops the robot being placed inside a restricted zone
- Three camera viewpoints (side view, top-down, angled) with the robot in a different position each time
- A nearest-object query run at each position
- Results displayed with Matplotlib

## Task 2: ROS2 delivery robot

A ROS2 Python package, `delivery_robot`, running in the Fastbot simulation on TheConstruct.ai (ROS2 Humble). Fastbot is used as the TurtleBot equivalent.

**Structure**

```
delivery_robot/
├── package.xml
├── setup.py
├── resource/delivery_robot
├── launch/delivery_mission.launch.py
└── delivery_robot/
    ├── delivery_robot_node.py   # state machine, navigation, monitoring
    └── keyboard_hri_node.py     # start/stop from the keyboard
```

**State machine (5 states)**

`IDLE` -> `NAVIGATE_TO_SHELF` -> `AVOID_OBSTACLE` -> `NAVIGATE_TO_OUTBOUND` -> `MISSION_COMPLETE`

**Topics**

| Topic | Direction | Purpose |
|---|---|---|
| `/fastbot/odom` | subscribe | Odometry (position and heading) |
| `/fastbot/scan` | subscribe | LIDAR, used for obstacle detection |
| `/fastbot/cmd_vel` | publish | Velocity commands |
| `/mission_trigger` | subscribe | `start` / `stop` from the keyboard node |
| `/robot_status` | publish | JSON status: state, elapsed time, distance, obstacle count |

Every state transition is also written to `delivery_mission_log.txt`.

### How to run (TheConstruct.ai)

1. Open the Rosject `7Kd2mQ9xTn` and make sure the Fastbot simulation is running (`ros2 topic list` should show `/fastbot/...` topics).
2. Build the package:

```bash
cd ~/ros2_ws
colcon build --packages-select delivery_robot
source install/setup.bash
```

3. Link the executables where `ros2 run` looks for them. On this platform `colcon` puts them in `bin/`:

```bash
mkdir -p ~/ros2_ws/install/delivery_robot/lib/delivery_robot
ln -sf ~/ros2_ws/install/delivery_robot/bin/delivery_robot_node ~/ros2_ws/install/delivery_robot/lib/delivery_robot/delivery_robot_node
ln -sf ~/ros2_ws/install/delivery_robot/bin/keyboard_hri_node ~/ros2_ws/install/delivery_robot/lib/delivery_robot/keyboard_hri_node
```

4. In terminal 1, start the robot node:

```bash
ros2 run delivery_robot delivery_robot_node
```

5. In terminal 2, start the keyboard node:

```bash
ros2 run delivery_robot keyboard_hri_node
```

Press `s` to start the mission, `x` to stop and `q` to quit the keyboard node.

To watch the status topic in a third terminal:

```bash
ros2 topic echo /robot_status
```

### Known issues

- `colcon build --symlink-install` fails on this platform (`option --editable not recognized`), so use a plain build and the symlink step above after every rebuild.
- The waypoints are fixed values near the top of `delivery_robot_node.py` and depend on the simulation world. Adjust them if the robot cannot reach them.
- Obstacle avoidance is a simple turn-and-resume behaviour, so it can still get slow in tight spaces.

## Planned improvements

- Task 1: drive the robot with real physics instead of placing it at set positions, and add segmentation-based object queries
- Task 2: replace the hand-written navigation with Nav2, use smarter avoidance than turning on the spot, and set waypoints dynamically

## AI use

Claude (Anthropic) was used as a coding and learning aid, and to help draft documentation. All code was run, tested and edited by the author.

## References

- Open Robotics (2022) *ROS 2 Humble documentation*. https://docs.ros.org/en/humble/
- Coumans, E. and Bai, Y. *PyBullet*. https://pybullet.org
- Articulated Robotics. *Articulated Robotics* [YouTube channel]. https://www.youtube.com/@articulatedrobotics
