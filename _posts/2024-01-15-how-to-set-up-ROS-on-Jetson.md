---
layout: post
title: "ROS Installation and Setup Guide for NVIDIA Jetson Nano"
date: 2024-01-15
header-includes:
    - \usepackage{textcomp}
    - \usepackage{graphicx}
categories:
  - robotics
tags:
  - ros
  - jetson nano
  - embedded
  - setup
  - robotics
description: "A guide for installing and configuring ROS on NVIDIA Jetson Nano"
use_math: true
classes: wide
giscus_comments: false
related_posts: true
---

## Introduction

The NVIDIA Jetson Nano is a powerful embedded system that provides GPU acceleration for AI and robotics projects in a compact form factor. Installing the Robot Operating System (ROS) on this platform enables the development of advanced robotic applications such as computer vision, autonomous navigation, and object recognition. This guide will walk you through installing and optimizing ROS on the Jetson Nano step by step.

## Jetson Nano Hardware Requirements

- NVIDIA Jetson Nano Developer Kit
- MicroSD card (64GB or larger, UHS-I or better speed recommended)
- Power supply (5V/4A barrel jack recommended for maximum performance)
- Cooling fan (strongly recommended)
- Optional: Camera module (CSI or USB), sensors, motor controllers

## Jetson Nano Basic Setup

### Operating System Installation

1. Download the Jetson Nano Developer Kit SD Card Image from NVIDIA's website
   ```bash
   # Download the latest JetPack image from https://developer.nvidia.com/embedded/downloads
   ```

2. Flash the image to your microSD card using SDK Manager or balenaEtcher

3. Insert the microSD card into your Jetson Nano and power it on

4. Complete the initial setup (create user, connect to WiFi, etc.)

5. Update the system:
   ```bash
   sudo apt update
   sudo apt upgrade -y
   ```

### Jetson Nano Performance Optimization

1. Set the maximum performance mode:
   ```bash
   sudo nvpmodel -m 0  # Sets to 10W mode
   sudo jetson_clocks  # Maximizes clock frequencies
   ```

2. Install system monitoring tools:
   ```bash
   sudo apt install htop iotop -y
   htop  # Check CPU and memory usage
   ```

3. Adjust swap space:
   ```bash
   # Create a swap file
   sudo fallocate -l 8G /var/swapfile
   sudo chmod 600 /var/swapfile
   sudo mkswap /var/swapfile
   sudo swapon /var/swapfile
   # Make swap persistent
   echo "/var/swapfile swap swap defaults 0 0" | sudo tee -a /etc/fstab
   ```

4. Set maximum performance automatically at boot:
   ```bash
   echo '#!/bin/bash
   /usr/bin/nvpmodel -m 0
   /usr/bin/jetson_clocks
   ' | sudo tee /usr/local/bin/jetson_performance
   
   sudo chmod +x /usr/local/bin/jetson_performance
   
   echo '[Unit]
   Description=Jetson Performance Mode
   After=multi-user.target
   
   [Service]
   Type=oneshot
   ExecStart=/usr/local/bin/jetson_performance
   RemainAfterExit=yes
   
   [Install]
   WantedBy=multi-user.target
   ' | sudo tee /etc/systemd/system/jetson-performance.service
   
   sudo systemctl enable jetson-performance.service
   ```

## ROS Installation Based on JetPack Version

Different ROS versions can be installed depending on the JetPack version on your Jetson Nano. Here are installation guides for major versions.

### Installing ROS Melodic on JetPack 4.x

1. Set up the ROS repository:
   ```bash
   sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
   ```

2. Add the ROS key:
   ```bash
   sudo apt install curl -y
   curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add -
   ```

3. Update package lists:
   ```bash
   sudo apt update
   ```

4. Install ROS Melodic:
   ```bash
   sudo apt install ros-melodic-desktop -y
   ```

5. Set up your environment:
   ```bash
   echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc
   source ~/.bashrc
   ```

6. Install dependencies:
   ```bash
   sudo apt install python-rosdep python-rosinstall python-rosinstall-generator python-wstool build-essential -y
   sudo rosdep init
   rosdep update
   ```

### Installing ROS 2 Foxy on JetPack 5.x

1. Set the locale:
   ```bash
   sudo apt update && sudo apt install locales -y
   sudo locale-gen en_US en_US.UTF-8
   sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
   export LANG=en_US.UTF-8
   ```

2. Add the ROS 2 repository:
   ```bash
   sudo apt install software-properties-common -y
   sudo add-apt-repository universe
   sudo apt update && sudo apt install curl -y
   sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
   ```

3. Install ROS 2 Foxy:
   ```bash
   sudo apt update
   sudo apt install ros-foxy-ros-base -y
   ```

4. Set up your environment:
   ```bash
   echo "source /opt/ros/foxy/setup.bash" >> ~/.bashrc
   source ~/.bashrc
   ```

## Setting Up a ROS Workspace

### ROS 1 (Melodic) Workspace

```bash
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws
catkin_make
echo "source ~/catkin_ws/devel/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### ROS 2 (Foxy) Workspace

```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws
colcon build
echo "source ~/ros2_ws/install/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

## Installing CUDA-Supported Packages for Jetson Nano

JetPack already comes with pre-installed CUDA, cuDNN, and TensorRT. Let's install additional packages to leverage these capabilities.

### 1. Verify OpenCV with CUDA Support

```bash
# Check the version of OpenCV included with JetPack
python3 -c "import cv2; print(cv2.__version__); print(cv2.getBuildInformation())"
```

### 2. Install CUDA Packages for Use with ROS

```bash
# For ROS Melodic
sudo apt install ros-melodic-perception ros-melodic-vision-opencv -y

# For ROS 2 Foxy
sudo apt install ros-foxy-perception ros-foxy-vision-opencv -y
```

## Project Examples Using Jetson Nano and ROS

### 1. GPU-Accelerated Image Processing Node

#### Creating a ROS Node Using CUDA-enabled OpenCV:

1. Create a package:
   ```bash
   cd ~/catkin_ws/src
   catkin_create_pkg jetson_vision roscpp rospy sensor_msgs cv_bridge std_msgs
   cd jetson_vision
   mkdir scripts
   ```

2. Create a GPU-accelerated image processing node:
   ```python
   #!/usr/bin/env python3
   
   import rospy
   from sensor_msgs.msg import Image
   from cv_bridge import CvBridge
   import cv2
   import numpy as np
   import time
   
   class JetsonVision:
       def __init__(self):
           self.bridge = CvBridge()
           self.image_sub = rospy.Subscriber('/camera/image_raw', Image, self.image_callback)
           self.image_pub = rospy.Publisher('/jetson_vision/processed', Image, queue_size=1)
           self.gpu_mat = cv2.cuda_GpuMat()
           
       def image_callback(self, data):
           try:
               # Convert image to OpenCV format
               cv_image = self.bridge.imgmsg_to_cv2(data, "bgr8")
               
               # Start processing time measurement
               start_time = time.time()
               
               # Upload image to GPU
               self.gpu_mat.upload(cv_image)
               
               # Convert to grayscale on GPU
               gpu_gray = cv2.cuda.cvtColor(self.gpu_mat, cv2.COLOR_BGR2GRAY)
               
               # Apply Gaussian blur
               gpu_blur = cv2.cuda.GaussianBlur(gpu_gray, (7, 7), 0)
               
               # Detect edges using Canny
               gpu_edges = cv2.cuda.Canny(gpu_blur, 50, 150)
               
               # Download results to CPU memory
               edges = gpu_edges.download()
               
               # Calculate processing time
               process_time = time.time() - start_time
               
               # Add processing time to result image
               cv2.putText(edges, f"Process time: {process_time:.3f}s", (10, 30),
                          cv2.FONT_HERSHEY_SIMPLEX, 0.6, (255, 255, 255), 2)
                          
               # Publish processed image
               self.image_pub.publish(self.bridge.cv2_to_imgmsg(edges, "mono8"))
               
           except Exception as e:
               rospy.logerr(e)
   
   if __name__ == '__main__':
       rospy.init_node('jetson_vision', anonymous=True)
       jv = JetsonVision()
       rospy.spin()
   ```

3. Set execution permissions:
   ```bash
   chmod +x ~/catkin_ws/src/jetson_vision/scripts/jetson_vision_node.py
   ```

4. Build the package:
   ```bash
   cd ~/catkin_ws
   catkin_make
   ```

### 2. Building an Object Detection System with Jetson Nano

1. Install required packages:
   ```bash
   sudo apt install ros-melodic-vision-msgs -y
   ```

2. Create an object detection node using TensorRT:
   ```python
   #!/usr/bin/env python3
   
   import rospy
   from sensor_msgs.msg import Image
   from vision_msgs.msg import Detection2DArray, Detection2D, BoundingBox2D
   from cv_bridge import CvBridge
   import cv2
   import numpy as np
   import tensorrt as trt
   import pycuda.driver as cuda
   import pycuda.autoinit
   
   # TensorRT model loading and inference code (omitted)
   # This would include code to load and run a TensorRT engine optimized for Jetson Nano
   
   class ObjectDetector:
       def __init__(self):
           # Model initialization code (omitted)
           
           self.bridge = CvBridge()
           self.image_sub = rospy.Subscriber('/camera/image_raw', Image, self.image_callback)
           self.detection_pub = rospy.Publisher('/object_detections', Detection2DArray, queue_size=1)
           self.image_pub = rospy.Publisher('/detection_visualization', Image, queue_size=1)
           
       def image_callback(self, data):
           # Image conversion and object detection code (omitted)
           # Publish results
           pass
   
   if __name__ == '__main__':
       rospy.init_node('object_detector', anonymous=True)
       detector = ObjectDetector()
       rospy.spin()
   ```

### 3. Jetson-Based Autonomous Navigation Robot

1. Install required packages:
   ```bash
   sudo apt install ros-melodic-navigation ros-melodic-gmapping ros-melodic-move-base -y
   ```

2. Configure robot navigation:
   ```xml
   <!-- jetson_robot_nav.launch -->
   <launch>
     <!-- Robot model and transformation setup -->
     <param name="robot_description" textfile="$(find jetson_robot)/urdf/robot.urdf" />
     <node name="robot_state_publisher" pkg="robot_state_publisher" type="robot_state_publisher" />
     
     <!-- LIDAR sensor setup -->
     <node name="rplidar" pkg="rplidar_ros" type="rplidarNode">
       <param name="serial_port" value="/dev/ttyUSB0"/>
       <param name="frame_id" value="laser"/>
     </node>
     
     <!-- SLAM mapping -->
     <node name="gmapping" pkg="gmapping" type="slam_gmapping">
       <param name="base_frame" value="base_link"/>
       <param name="odom_frame" value="odom"/>
       <param name="map_update_interval" value="5.0"/>
     </node>
     
     <!-- Navigation stack -->
     <node name="move_base" pkg="move_base" type="move_base" output="screen">
       <rosparam file="$(find jetson_robot)/config/costmap_common_params.yaml" command="load" ns="global_costmap" />
       <rosparam file="$(find jetson_robot)/config/costmap_common_params.yaml" command="load" ns="local_costmap" />
       <rosparam file="$(find jetson_robot)/config/local_costmap_params.yaml" command="load" />
       <rosparam file="$(find jetson_robot)/config/global_costmap_params.yaml" command="load" />
       <rosparam file="$(find jetson_robot)/config/base_local_planner_params.yaml" command="load" />
     </node>
   </launch>
   ```

## Optimizing Deep Learning Models for Jetson Nano

You can optimize deep learning models for Jetson Nano using TensorRT.

### Converting TensorFlow Models to TensorRT

1. Install required packages:
   ```bash
   sudo apt install python3-pip
   pip3 install --upgrade pip
   pip3 install numpy tensorflow
   ```

2. Convert TensorFlow models to UFF or ONNX and apply TensorRT optimization:
   ```python
   import tensorflow as tf
   import tensorrt as trt
   import uff
   
   # Load a model saved from TensorFlow
   # model = tf.saved_model.load("path_to_saved_model")
   
   # Convert to UFF
   # uff_model = uff.from_tensorflow(graphdef, output_nodes=["output"])
   
   # Create TensorRT engine
   # TRT_LOGGER = trt.Logger(trt.Logger.INFO)
   # builder = trt.Builder(TRT_LOGGER)
   # ...
   ```

## Monitoring and Analyzing GPU on Jetson Nano

1. Using Tegrastats:
   ```bash
   tegrastats
   ```

2. Installing and using Jetson-stats:
   ```bash
   sudo -H pip install -U jetson-stats
   jtop
   ```

## One-Line Installation Script for Jetson Nano + ROS

Here's a one-line script to install and optimize ROS Melodic on Jetson Nano:

```bash
#!/bin/bash
# ROS Melodic installation script for Jetson Nano

# Update system
sudo apt update && sudo apt upgrade -y && 
# Set up ROS repositories
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list' && 
sudo apt install curl -y && 
curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add - && 
sudo apt update && 
# Install ROS Melodic
sudo apt install ros-melodic-desktop -y && 
# Set up environment
echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc && 
source ~/.bashrc && 
# Install dependencies
sudo apt install python-rosdep python-rosinstall python-rosinstall-generator python-wstool build-essential -y && 
sudo rosdep init && 
rosdep update && 
# Create workspace
mkdir -p ~/catkin_ws/src && 
cd ~/catkin_ws && 
catkin_make && 
echo "source ~/catkin_ws/devel/setup.bash" >> ~/.bashrc && 
source ~/.bashrc && 
# Optimize Jetson Nano
sudo nvpmodel -m 0 && 
sudo jetson_clocks && 
# Create startup service for maximum performance
echo '#!/bin/bash
/usr/bin/nvpmodel -m 0
/usr/bin/jetson_clocks
' | sudo tee /usr/local/bin/jetson_performance && 
sudo chmod +x /usr/local/bin/jetson_performance && 
echo '[Unit]
Description=Jetson Performance Mode
After=multi-user.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/jetson_performance
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
' | sudo tee /etc/systemd/system/jetson-performance.service && 
sudo systemctl enable jetson-performance.service && 
# Add swap space
sudo fallocate -l 8G /var/swapfile && 
sudo chmod 600 /var/swapfile && 
sudo mkswap /var/swapfile && 
sudo swapon /var/swapfile && 
echo "/var/swapfile swap swap defaults 0 0" | sudo tee -a /etc/fstab && 
echo "ROS Melodic installation and optimization complete!"
```