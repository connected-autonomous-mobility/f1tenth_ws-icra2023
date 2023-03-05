# F1TENTH Autonomous Racing Research
This a repository for doing autonomous racing research running on [F1TENTH](https://f1tenth.org/). Forked from [f1tenth_gym_ros](https://github.com/f1tenth/f1tenth_gym_ros). 

The original repository is purely a simulation environment for F1TENTH autonomous racing, without any other ROS2 packages to run code on. This forked repository contains multiple implementations to do autonomous racing (mainly solutions to [F1TENTH Labs](https://github.com/f1tenth/f1tenth_labs)):

Currently Used Algorithms:
- [Waypoint Generator](./nodes/waypoint_generator/) for generating waypoints $\rightarrow$ `nodes/waypoint_generator`
	- Soon to be replaced with automatic raceline generation
- [Pure Pursuit](./nodes/pure_pursuit/) for waypoint following $\rightarrow$ `nodes/pure_pursuit`
- [Particle Filter](./nodes/particle_filter/) for localization $\rightarrow$ `nodes/particle_filter`
- [RRT](./nodes/rrt) for obstacle avoidance $\rightarrow$ `nodes/rrt` 
- [slam_toolbox](https://github.com/SteveMacenski/slam_toolbox) for mapping

Other algorithms that are not used
- A [PID controller](./nodes/wall_follow/) to follow walls $\rightarrow$ `nodes/wall_follow`
- [Scan matching](./nodes/scan_matching) $\rightarrow$ `nodes/scan_matching` (To be completed)

# Running Simulation
Simulation sometimes seems to crash using another launch file. Do `ros2 run` instead of `ros2 launch`.

### With an NVIDIA gpu
You need **Docker**, **nvidia-docker2** and **rocker** and installed.

1. Clone this repo
2. Build the docker image by running:
```bash
docker build -t f1tenth_gym_ros -f Dockerfile .
```
3. To run the containerized environment, start a docker container by running the following. (example showned here with nvidia-docker support). By running this, the current directory that you're in (should be `f1tenth_gym_ros`) is mounted in the container at `/sim_ws/src/f1tenth_gym_ros`. Which means that the changes you make in the repo on the host system will also reflect in the container.
```bash
sudo rocker --nvidia --x11 --volume .:/sim_ws/src/f1tenth_gym_ros ./nodes:/sim_ws/src/ -- f1tenth_gym_ros
```

### Without an NVIDIA gpu
You need **Docker** installed.

1. Clone this repo 
2. Build the docker image by:
```bash
docker build -t f1tenth_gym_ros -f Dockerfile .
```
3. Bringup the novnc container and the sim container with docker-compose:
```bash
docker-compose up
``` 
4. In a separate terminal, run the following, and you'll have the a bash session in the simulation container. `tmux` is available for convenience.
```bash
docker exec -it f1tenth-autonomous-racing-research-sim-1 /bin/bash
```
5. In your browser, navigate to [http://localhost:8081/vnc.html](http://localhost:8081/vnc.html), you should see the noVNC logo with the connect button. Click the connect button to connect to the session.



# Launching the Simulation
1. `tmux` is included in the contianer, so you can create multiple bash sessions in the same terminal.
2. To launch the simulation, make sure you source both the ROS2 setup script and the local workspace setup script. Run the following in the bash session from the container:
```bash
source /opt/ros/foxy/setup.bash
source install/local_setup.bash
ros2 launch f1tenth_gym_ros gym_bridge_launch.py
```
A rviz window should pop up showing the simulation either on your host system or in the browser window depending on the display forwarding you chose.

You can then run another node by creating another bash session in `tmux`.

# Configuring the simulation
- The configuration file for the simulation is at `f1tenth_gym_ros/config/sim.yaml`.
- Topic names and namespaces can be configured but is recommended to leave uncahnged.
- The map can be changed via the `map_path` parameter. You'll have to use the full path to the map file in the container. The map follows the ROS convention. It is assumed that the image file and the `yaml` file for the map are in the same directory with the same name. See the note below about mounting a volume to see where to put your map file.
- The `num_agent` parameter can be changed to either 1 or 2 for single or two agent racing.
- The ego and opponent starting pose can also be changed via parameters, these are in the global map coordinate frame.

The entire directory of the repo is mounted to a workspace `/sim_ws/src` as a package. All changes made in the repo on the host system will also reflect in the container. After changing the configuration, run `colcon build` again in the container workspace to make sure the changes are reflected.

# Topics published by the simulation

In **single** agent:

- `/scan`: The ego agent's laser scan
- `/ego_racecar/odom`: The ego agent's odometry
- `/map`: The map of the environment

A `tf` tree is also maintained.

In **two** agents:

In addition to the topics available in the single agent scenario, these topics are also available:

- `/opp_scan`: The opponent agent's laser scan
- `/ego_racecar/opp_odom`: The opponent agent's odometry for the ego agent's planner
- `/opp_racecar/odom`: The opponent agents' odometry
- `/opp_racecar/opp_odom`: The ego agent's odometry for the opponent agent's planner

# Topics subscribed by the simulation

In **single** agent:

- `/drive`: The ego agent's drive command via `AckermannDriveStamped` messages
- `/initalpose`: This is the topic for resetting the ego's pose via RViz's 2D Pose Estimate tool. Do **NOT** publish directly to this topic unless you know what you're doing.

TODO: kb teleop topics

In **two** agents:

In addition to all topics in the single agent scenario, these topics are also available:

- `/opp_drive`: The opponent agent's drive command via `AckermannDriveStamped` messages
- `/goal_pose`: This is the topic for resetting the opponent agent's pose via RViz's 2D Goal Pose tool. Do **NOT** publish directly to this topic unless you know what you're doing.

# Keyboard Teleop
The keyboard teleop node from `teleop_twist_keyboard` is also installed as part of the simulation's dependency. To enable keyboard teleop, set `kb_teleop` to `True` in `sim.yaml`. After launching the simulation, in another terminal, run:
```bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```
Then, press `i` to move forward, `u` and `o` to move forward and turn, `,` to move backwards, `m` and `.` to move backwards and turn, and `k` to stop in the terminal window running the teleop node.
