---
layout: post
title: "Isaac Sim Installation Guide for Ubuntu"
date: 2024-03-19
categories:
  - robotics
tags:
  - isaac sim
  - setup
  - ubuntu
  - robotics
description: "A simplified guide for installing NVIDIA Isaac Sim on Ubuntu"
---

## System Requirements

- **Operating System**: Ubuntu 20.04 or 22.04 LTS (64-bit)
- **GPU**: NVIDIA RTX/GeForce/Quadro with 8GB+ VRAM (RTX series recommended)
- **Driver**: NVIDIA Driver R525 or newer
- **RAM**: 32GB minimum (64GB recommended)
- **Storage**: 50GB available SSD space
- **CPU**: Intel Core i7/AMD Ryzen 7 or better

# Isaac Sim Installation

## Method 1: Using Omniverse Launcher (Recommended)

### Step 1: Create NVIDIA Account & Download Launcher

```bash
# Download the Omniverse Launcher AppImage
wget -O omniverse-launcher-linux.AppImage https://install.launcher.omniverse.nvidia.com/installers/omniverse-launcher-linux.AppImage

# Make it executable
chmod +x omniverse-launcher-linux.AppImage
```

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

## Method 2: Using Docker Container

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

# Troubleshooting

## GPU Driver Issues

```bash
# Check installed driver version
nvidia-smi

# Install latest driver if needed
sudo apt update
sudo apt install nvidia-driver-535  # or newest available version
sudo reboot
```

## Display/Graphics Issues

```bash
# Fix common X server issues
xhost +local:docker

# If getting "Cannot open display" error
export DISPLAY=:0
```

# One-Line Installation Commands

## Complete Omniverse Launcher Setup

```bash
wget -O omniverse-launcher-linux.AppImage https://install.launcher.omniverse.nvidia.com/installers/omniverse-launcher-linux.AppImage && chmod +x omniverse-launcher-linux.AppImage && ./omniverse-launcher-linux.AppImage
```

## Complete Docker Setup

```bash
sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common && curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - && sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" && sudo apt-get update && sudo apt-get install -y docker-ce docker-ce-cli containerd.io && sudo usermod -aG docker $USER && distribution=$(. /etc/os-release;echo $ID$VERSION_ID) && curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add - && curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list && sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit && sudo systemctl restart docker
```