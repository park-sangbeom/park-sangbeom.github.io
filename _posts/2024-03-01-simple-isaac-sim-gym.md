---
layout: post
title: "NVIDIA Isaac Sim and Isaac Gym Installation Guide for Ubuntu"
date: 2024-03-01
categories:
  - robotics
tags:
  - isaac sim
  - isaac gym
  - setup
  - ubuntu
  - robotics
description: "A simplified guide for installing NVIDIA Isaac Sim and Isaac Gym on Ubuntu"
---

## System Requirements
<br>
- **Operating System**: Ubuntu 20.04 or 22.04 LTS (64-bit)
- **GPU**: NVIDIA RTX/GeForce/Quadro with 8GB+ VRAM (RTX series recommended)
- **Driver**: NVIDIA Driver R525 or newer
- **RAM**: 32GB minimum (64GB recommended)
- **Storage**: 50GB available SSD space
- **CPU**: Intel Core i7/AMD Ryzen 7 or better
<br>

# Isaac Sim Installation
<br>

## Installation Method 1: Using Omniverse Launcher
### Step 1: Create NVIDIA Account & Download Launcher
```bash
# Download the Omniverse Launcher AppImage
wget -O omniverse-launcher-linux.AppImage https://install.launcher.omniverse.nvidia.com/installers/omniverse-launcher-linux.AppImage

# Make it executable
chmod +x omniverse-launcher-linux.AppImage
```
<br>

### Step 2: Install Isaac Sim
1. Run the launcher:
   ```bash
   ./omniverse-launcher-linux.AppImage
   ```

2. Sign in with your NVIDIA account (create one if needed)

3. In the Launcher:
   - Go to the "Exchange" tab
   - Search for "Isaac Sim"
   - Click "Install"
   - Choose your installation location
   - Wait for installation to complete
<br>

### Step 3: Launch Isaac Sim
1. In the Omniverse Launcher, go to the "Library" tab
2. Find Isaac Sim and click "Launch"
<br>

## Installation Method 2: Using Docker Container
<br>

### Step 1: Install Docker and NVIDIA Container Toolkit
```bash
# Install Docker
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io

# Add current user to docker group
sudo usermod -aG docker $USER
newgrp docker

# Install NVIDIA Container Toolkit
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
sudo systemctl restart docker
```
<br>

### Step 2: Pull and Run Isaac Sim Container
```bash
# Log in to NGC (you'll need an NVIDIA account and API key)
docker login nvcr.io

# Pull the container
docker pull nvcr.io/nvidia/isaac-sim:latest

# Create a workspace directory
mkdir -p ~/isaac-sim-workspace

# Run the container with GUI support
docker run --name isaac-sim -it --gpus all -e "ACCEPT_EULA=Y" --rm \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  -v ~/isaac-sim-workspace:/workspace \
  -e DISPLAY=$DISPLAY \
  nvcr.io/nvidia/isaac-sim:latest
```
<br>

# Isaac Gym Installation
<br>

## Prerequisites for Isaac Gym
```bash
# Install required system packages
sudo apt-get update
sudo apt-get install -y \
  git \
  python3-dev \
  python3-pip \
  libopenmpi-dev \
  libosmesa6-dev \
  libgl1-mesa-glx \
  libglfw3-dev \
  libglew-dev \
  patchelf
```
<br>

## Method 1: Install Isaac Gym Preview
<br>

### Step 1: Download the Isaac Gym package
1. Go to the NVIDIA Isaac Gym page: https://developer.nvidia.com/isaac-gym
2. Sign in with your NVIDIA account
3. Download the Isaac Gym Preview package for Linux
<br>

### Step 2: Extract and Install
```bash
# Extract the downloaded package (replace with actual filename)
tar -xvf isaacgym_preview_x_package.tar.gz

# Navigate to the extracted directory
cd isaacgym

# Create a Python virtual environment
python3 -m venv gym_env
source gym_env/bin/activate

# Install the package
pip install -e .

# Install additional dependencies
pip install -r requirements.txt
```
<br>

### Step 3: Verify Installation
```bash
# Go to the examples directory
cd examples

# Run a sample environment
python3 1080_balls_of_solitude.py
```
<br>

## Method 2: Install IsaacGymEnvs
```bash
# Clone the repository
git clone https://github.com/NVIDIA-Omniverse/IsaacGymEnvs.git
cd IsaacGymEnvs

# Create a Python virtual environment
python3 -m venv isaacgym_env
source isaacgym_env/bin/activate

# Install the requirements
pip install -r requirements.txt

# Install the package
pip install -e .
```
<br>

### Running IsaacGymEnvs Examples
```bash
# Test with the Cartpole example
python isaacgymenvs/train.py task=Cartpole

# Test with the Ant example
python isaacgymenvs/train.py task=Ant
```
<br>

## Troubleshooting Isaac Gym
<br>

### Python/CUDA Issues
```bash
# If you encounter CUDA initialization errors
pip install torch==1.13.1+cu116 torchvision==0.14.1+cu116 torchaudio==0.13.1 --extra-index-url https://download.pytorch.org/whl/cu116

# For other Python dependency issues
pip install -r requirements.txt --upgrade
```
<br>

### Display Issues
```bash
# If you get "cannot open display" errors
export DISPLAY=:0
xhost +local:

# If you have rendering issues
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/lib/x86_64-linux-gnu
```
<br>

# Troubleshooting (Common for Both)
<br>

## GPU Driver Issues
```bash
# Check installed driver version
nvidia-smi

# Install latest driver if needed
sudo apt update
sudo apt install nvidia-driver-535  # or newest available version
sudo reboot
```
<br>

## Display/Graphics Issues
```bash
# Fix common X server issues
xhost +local:docker

# If getting "Cannot open display" error
export DISPLAY=:0
```
<br>

# Quick ROS Integration (Optional)
<br>
If you want to use Isaac Sim with ROS:
<br>
```bash
# For ROS Noetic (Ubuntu 20.04)
sudo apt install ros-noetic-desktop-full -y

# OR for ROS 2 Humble (Ubuntu 22.04)
sudo apt install ros-humble-desktop -y

# Don't forget to source ROS
echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc  # for ROS Noetic
# OR
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc  # for ROS 2 Humble

source ~/.bashrc
```

# One-Line Installation Commands
<br>
## Complete Omniverse Launcher Setup
```bash
wget -O omniverse-launcher-linux.AppImage https://install.launcher.omniverse.nvidia.com/installers/omniverse-launcher-linux.AppImage && chmod +x omniverse-launcher-linux.AppImage && ./omniverse-launcher-linux.AppImage
```
<br>

## Complete Docker Setup
```bash
sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common && curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - && sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" && sudo apt-get update && sudo apt-get install -y docker-ce docker-ce-cli containerd.io && sudo usermod -aG docker $USER && distribution=$(. /etc/os-release;echo $ID$VERSION_ID) && curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add - && curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list && sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit && sudo systemctl restart docker
```
<br>

## Isaac Gym Dependencies Setup
```bash
sudo apt-get update && sudo apt-get install -y git python3-dev python3-pip libopenmpi-dev libosmesa6-dev libgl1-mesa-glx libglfw3-dev libglew-dev patchelf && python3 -m venv isaacgym_env && source isaacgym_env/bin/activate && pip install torch==1.13.1+cu116 torchvision==0.14.1+cu116 --extra-index-url https://download.pytorch.org/whl/cu116
```