# ME462LittleDaisiesMKII

ROS2 Humble & MoveIt package and Docker container to draw words on a sand pool with a UR5e robot arm.

## Folder Structure

```
.
├── my_robot_cell
│   ├── LICENSE
│   ├── my_robot_cell_control
│   ├── my_robot_cell_description
│   └── my_robot_cell_moveit_config
├── README.md
├── ZenPoolContainer
│   ├── bash_aliases
│   ├── bashrc
│   ├── Dockerfile
│   ├── entrypoint.sh
│   ├── README.md
│   ├── tmux.conf
│   ├── zenpool.config.rviz
│   ├── zenpoolsim.yml
│   └── zenpool.yml
└── zenpool_draw_letter
```

## Overview

This repository contains ROS2 Humble & MoveIt packages to draw words on a sand pool with a UR5e robot arm.

* **`my_robot_cell`**: Contains the core packages (`my_robot_cell_control`, `my_robot_cell_description`, `my_robot_cell_moveit_config`). To modify or create new versions of these packages, users can refer to the UR driver documentation's custom workcell example [here](https://docs.universal-robots.com/Universal_Robots_ROS2_Documentation/doc/ur_tutorials/my_robot_cell/doc/index.html#).
* **`zenpool_draw_letter`**: A ROS2 package used to send position commands to the MoveIt planner, perform tool changes, and draw text on the sand surface.
* **`ZenPoolContainer`**: Contains the Dockerfile (which installs every necessary package needed to run the above code) and `tmuxinator` scripts to quickly set up terminals and launch ROS processes.

---

## Usage Guide

### 1. Workspace Setup & Cloning

First, create a workspace directory and a `src` folder in your home directory, then clone the repository.

```bash
# Create the workspace and navigate into it
mkdir -p ~/ws_me462/src
cd ~/ws_me462/src

# Clone this repository (replace [URL] with the actual repository URL)
git clone git@github.com:OganAltug/ME462S25_LittleDaisiesMKII.git

# NOTE: If you are using a Raspberry Pi or another ARM-based machine, 
# clone the rasppi branch instead:
# git clone -b rasppi git@github.com:OganAltug/ME462S25_LittleDaisiesMKII.git

```

*(Note: If you change the `ws_me462` folder name, you must also modify the volume mount paths in the Dockerfile, tmuxinator yml's etc.)*

### 2. Building the Docker Image

Navigate to the `ZenPoolContainer` directory and build the Docker image:

```bash
cd ~/ws_me462/src/ME462LittleDaisiesMKII/ZenPoolContainer
docker build -t zenpool_container .

```

### 3. Setting Up the Docker Alias

To easily run the container with all necessary hardware, network configurations, and volume mounts, add an alias to your `.bashrc` file.

You can do this automatically by running the following command block in your terminal:

```bash
cat << 'EOF' >> ~/.bashrc

# Alias for running the ZenPool UR5e Docker Container
alias run_zenpool_container='docker run -it \
            --user ros \
            --network=host \
            --ipc=host \
            --runtime=nvidia \
            -v ~/ws_me462:/home/ros/ws_me462 \
            -v /tmp/.X11-unix:/tmp/.X11-unix:rw \
            --device-cgroup-rule="c *:* rmw" \
            --env=DISPLAY \
            -v /dev:/dev \
            --env=NVIDIA_VISIBLE_DEVICES=all \
            --env=NVIDIA_DRIVER_CAPABILITIES=all \
            --gpus all \
            zenpool_container'
EOF

# Apply the changes to your current terminal
source ~/.bashrc

```

### 4. Running the Container & Building the Workspace

Now you can start the container and build the ROS2 packages. Note that you must build the workspace the first time you run the container.

```bash
# Launch the container using the alias created above
run_zenpool_container

# Navigate to the workspace (inside the container)
cd ~/ws_me462

# Build the ROS2 workspace
colcon build --symlink-install

# Source the newly built workspace
source install/setup.bash

```

### 5. Launching the Robot & Drawing

Once the packages are built, use `tmuxinator` to set up the robot control terminals.

```bash
# Launch the tmux workspace
tmuxinator start zenpool

```

*Note: This launches the robot controller expecting the IP `192.168.8.4`. If your robot uses a different IP, you will need to modify the `zenpool.yml` file manually.*

Finally, in an empty terminal within the container, run the drawing node:

```bash
ros2 run zenpool_draw_letter zenpool_draw_letter --ros-args -p text:="'ROMER'" -p debug:=true

```

**Parameters:**

* `text`: Modify this string to change the written text (e.g., `"'HELLO'"`).
* `debug`: Set to `false` if you want the robot to execute every action continuously. If set to `true`, the robot will stop at every planning step, requiring the user to inspect the planned path and confirm the solution before moving.

---

## Robot Setup and Running with `ur_sim`

Robot setup must be completed before following the ROS communication steps above.

1. **Physical Robot:** The user needs to create a program on the UR5e teach pendant with **External Control** loaded and provide the host computer's IP address.
2. **UR Simulator (`ur_sim`):** You can use the provided `ur_sim` emulator instead of the physical robot. If you use the "ur_sim" use:
```bash
tmuxinator start zenpoolsim
```
which would connect to robot default ur_sim robot ip.
* You must start the emulator **before** running the `tmuxinator` script.
* You must set up the `ur_sim` exactly like the real robot (loading the external control program).
* These processes are heavily documented on the [ur_driver ROS2 documentation page](https://docs.universal-robots.com/Universal_Robots_ROS2_Documentation/index.html).

## Good Luck
Also refer to MoveIt Humble Documentation [tutorial section](https://moveit.picknik.ai/humble/doc/tutorials/tutorials.html) to get quickly familiar with MoveIt.