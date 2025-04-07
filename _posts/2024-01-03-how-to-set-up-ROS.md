---
layout: post
title: "Complete Guide to ROS Installation and Setup"
date: 2024-01-03
header-includes:
    - \usepackage{textcomp}
    - \usepackage{graphicx}
categories:
  - robotics
tags:
  - ros
  - setup
  - ubuntu
  - robotics
description: "A comprehensive guide on installing and configuring ROS (Robot Operating System) on Ubuntu"
use_math: true
classes: wide
giscus_comments: false
related_posts: true
---

## Introduction

Robot Operating System (ROS) is a flexible framework for writing robot software. It is a collection of tools, libraries, and conventions that aim to simplify the task of creating complex and robust robot behavior across a wide variety of robotic platforms. In this guide, we'll walk through the installation process for both ROS 1 and ROS 2, the two major versions currently in use.

## ROS 1 Installation (Noetic - for Ubuntu 20.04)

### Setting Up the Repository

To begin with, you need to configure your system to accept software from the ROS repositories:

```bash
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
```

### Setting Up Keys

Next, you need to set up the necessary keys:

```bash
sudo apt install curl -y
curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add -
```

### Updating and Installing ROS

Now, update your package lists and install ROS:

```bash
sudo apt update
sudo apt install ros-noetic-desktop-full -y
```

If you prefer a smaller installation, you can choose from these options:
- `ros-noetic-desktop`: GUI tools without simulation
- `ros-noetic-ros-base`: Base packages only, no GUI tools

### Environment Setup

For the changes to take effect in your current terminal and all future terminal sessions, add the following to your shell configuration:

For Bash users:
```bash
echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

For Zsh users:
```bash
echo "source /opt/ros/noetic/setup.zsh" >> ~/.zshrc
source ~/.zshrc
```

### Installing Dependencies for Building Packages

Install the necessary build tools and dependencies:

```bash
sudo apt install python3-rosdep python3-rosinstall python3-rosinstall-generator python3-wstool build-essential -y
```

### Initializing rosdep

Initialize rosdep, which is used for installing system dependencies:

```bash
sudo rosdep init
rosdep update
```

## ROS 2 Installation (Humble - for Ubuntu 22.04)

### Locale Configuration

ROS 2 requires a UTF-8 locale. Set this up with:

```bash
locale  # Check current settings
sudo apt update && sudo apt install locales -y
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8
```

### Setting Up Sources

Add the Universe repository:

```bash
sudo apt install software-properties-common -y
sudo add-apt-repository universe
```

### Adding the ROS 2 GPG Key

```bash
sudo apt update && sudo apt install curl -y
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
```

### Adding the Repository to Your Sources List

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
```

### Installing ROS 2

Update your package lists and install ROS 2:

```bash
sudo apt update
sudo apt install ros-humble-desktop -y
```

For a minimal installation (CLI tools only):
```bash
sudo apt install ros-humble-ros-base -y
```

Install development tools:
```bash
sudo apt install ros-dev-tools -y
```

### Environment Setup

For Bash users:
```bash
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

For Zsh users:
```bash
echo "source /opt/ros/humble/setup.zsh" >> ~/.zshrc
source ~/.zshrc
```

## Creating a Workspace

### ROS 1 Workspace

Create a catkin workspace, which is a directory where you'll develop and build your ROS packages:

```bash
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws
catkin_make
echo "source ~/catkin_ws/devel/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### ROS 2 Workspace

Create a workspace using colcon:

```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws
colcon build
echo "source ~/ros2_ws/install/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

## Creating Your First Package

### ROS 1 Package Creation

```bash
cd ~/catkin_ws/src
catkin_create_pkg my_package std_msgs rospy roscpp
cd ~/catkin_ws
catkin_make
```

### ROS 2 Package Creation

For a Python package:
```bash
cd ~/ros2_ws/src
ros2 pkg create --build-type ament_python my_package
```

For a C++ package:
```bash
cd ~/ros2_ws/src
ros2 pkg create --build-type ament_cmake my_package
```

Build the package:
```bash
cd ~/ros2_ws
colcon build --packages-select my_package
```

## Testing ROS Commands

### Basic ROS 1 Commands

Start the ROS master:
```bash
roscore
```

In a new terminal, run a simple publisher node:
```bash
rosrun rospy_tutorials talker
```

In another terminal, run a subscriber node:
```bash
rosrun rospy_tutorials listener
```

Check the list of running nodes:
```bash
rosnode list
```

Check the list of active topics:
```bash
rostopic list
```

### Basic ROS 2 Commands

Run a publisher node (no separate master needed):
```bash
ros2 run demo_nodes_cpp talker
```

In a new terminal, run a subscriber node:
```bash
ros2 run demo_nodes_cpp listener
```

List active nodes:
```bash
ros2 node list
```

List active topics:
```bash
ros2 topic list
```

## Useful Tools

### RViz (Visualization Tool)

ROS 1:
```bash
rosrun rviz rviz
```

ROS 2:
```bash
ros2 run rviz2 rviz2
```

### rqt (GUI Tools Collection)

ROS 1:
```bash
rqt
```

ROS 2:
```bash
ros2 run rqt rqt
```

## Troubleshooting Common Issues

### ROS 1 Common Issues

1. **'roscore not found' error**:
   ```bash
   source /opt/ros/noetic/setup.bash
   ```

2. **Package not found errors**:
   ```bash
   source ~/catkin_ws/devel/setup.bash
   ```

3. **rosdep errors**:
   ```bash
   sudo rosdep init
   rosdep update
   ```

### ROS 2 Common Issues

1. **Environment issues**:
   ```bash
   source /opt/ros/humble/setup.bash
   ```

2. **colcon build errors**:
   ```bash
   sudo apt install python3-colcon-common-extensions
   ```

3. **Package dependency errors**:
   ```bash
   rosdep install --from-paths src --ignore-src -r -y
   ```

## Managing Multiple ROS Versions

If you have multiple ROS versions installed on the same system, you can create switching scripts:

```bash
# ros1.sh
#!/bin/bash
source /opt/ros/noetic/setup.bash
source ~/catkin_ws/devel/setup.bash
export ROS_MASTER_URI=http://localhost:11311
export ROS_VERSION=1
echo "ROS 1 Activated"
```

```bash
# ros2.sh
#!/bin/bash
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
export ROS_VERSION=2
echo "ROS 2 Activated"
```

Make the scripts executable:
```bash
chmod +x ros1.sh ros2.sh
```

Use them by sourcing:
```bash
source ~/ros1.sh  # Activate ROS 1
source ~/ros2.sh  # Activate ROS 2
```

## Additional Resources

- [ROS 1 Official Documentation](http://wiki.ros.org/)
- [ROS 2 Official Documentation](https://docs.ros.org/)
- [ROS Answers (Q&A Site)](https://answers.ros.org/)
- [ROS Discourse Forum](https://discourse.ros.org/)

## One-line Installation Commands for Quick Setup

For those who want a quick setup, here are one-line commands that combine multiple installation steps:

### ROS 1 (Noetic) Complete Setup
```bash
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list' && sudo apt install curl -y && curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add - && sudo apt update && sudo apt install ros-noetic-desktop-full -y && echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc && source ~/.bashrc && sudo apt install python3-rosdep python3-rosinstall python3-rosinstall-generator python3-wstool build-essential -y && sudo rosdep init && rosdep update && mkdir -p ~/catkin_ws/src && cd ~/catkin_ws && catkin_make && echo "source ~/catkin_ws/devel/setup.bash" >> ~/.bashrc && source ~/.bashrc
```

### ROS 2 (Humble) Complete Setup
```bash
sudo apt update && sudo apt install locales -y && sudo locale-gen en_US en_US.UTF-8 && sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8 && export LANG=en_US.UTF-8 && sudo apt install software-properties-common -y && sudo add-apt-repository universe && sudo apt update && sudo apt install curl -y && sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null && sudo apt update && sudo apt install ros-humble-desktop -y && sudo apt install ros-dev-tools -y && echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc && source ~/.bashrc && mkdir -p ~/ros2_ws/src && cd ~/ros2_ws && colcon build && echo "source ~/ros2_ws/install/setup.bash" >> ~/.bashrc && source ~/.bashrc
```

These commands are particularly useful for scripting or setting up new development environments quickly.