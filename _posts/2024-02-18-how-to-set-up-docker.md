---
layout: post
title: "Docker Ubuntu Setup Guide"
date: 2024-02-18
header-includes:
    - \usepackage{textcomp}
    - \usepackage{graphicx}
categories:
  - docker
  - ubuntu
tags:
  - docker
  - ubuntu
  - setup
  - '2024'
description: "A comprehensive guide on setting up and using Ubuntu with Docker"
classes: wide
giscus_comments: false
related_posts: true
---

## Introduction

Docker has revolutionized how we develop, package, and deploy applications by enabling consistent environments across different systems. This guide will walk you through setting up an Ubuntu-based Docker environment, from installation to advanced usage patterns. Whether you're new to Docker or looking to optimize your containerized Ubuntu workflow, this guide has you covered.

## Installing Docker

Before you can start working with Ubuntu containers, you need to install Docker on your system.

### Installing Docker on Ubuntu Host

```bash
# Update your package index
sudo apt update

# Install prerequisites
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

# Add Docker's official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Set up the stable repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update the package index again
sudo apt update

# Install Docker Engine
sudo apt install -y docker-ce docker-ce-cli containerd.io

# Add your user to the docker group to run Docker without sudo
sudo usermod -aG docker $USER

# Apply the new group membership
newgrp docker
```

### Installing Docker on macOS

1. Download and install Docker Desktop from the [official website](https://www.docker.com/products/docker-desktop)
2. Open Docker Desktop and follow the setup wizard

## Basic Ubuntu Container Usage

Once Docker is installed, you can start working with Ubuntu containers.

### Pulling an Ubuntu Image

```bash
# Pull the latest Ubuntu image
docker pull ubuntu:latest

# Pull a specific version (e.g., Ubuntu 22.04 LTS)
docker pull ubuntu:22.04
```

### Running an Ubuntu Container

#### Interactive Mode

```bash
# Run a container interactively with a bash shell
docker run -it ubuntu:latest bash
```

This command:
- `-i`: Keeps STDIN open
- `-t`: Allocates a pseudo-TTY
- `bash`: Runs the bash shell inside the container

#### Accessing a Running Container

```bash
# Access a running container with bash
docker exec -it my-ubuntu bash
```

#### Stopping and Removing Containers

```bash
# Stop a running container
docker stop my-ubuntu

# Remove a container
docker rm my-ubuntu

# Force remove a running container
docker rm -f my-ubuntu
```

## Creating Custom Ubuntu Images

Creating custom Ubuntu images allows you to package your applications and dependencies for easier deployment.

### Basic Dockerfile Example

```dockerfile
# Use Ubuntu 22.04 as the base image
FROM ubuntu:22.04

# Set non-interactive mode for apt
ENV DEBIAN_FRONTEND=noninteractive

# Update and install packages
RUN apt-get update && apt-get install -y \
    python3 \
    python3-pip \
    nginx \
    vim \
    curl \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Set the working directory
WORKDIR /app

# Copy application files
COPY . /app/

# Expose a port
EXPOSE 80

# Define the command to run when the container starts
CMD ["nginx", "-g", "daemon off;"]
```

### Building the Image

```bash
# Build an image from the Dockerfile in the current directory
docker build -t my-ubuntu-app .

# Build with a specific tag
docker build -t my-ubuntu-app:1.0 .
```

### Running the Custom Image

```bash
# Run the custom image
docker run -d -p 8080:80 --name my-webapp my-ubuntu-app
```

This command:
- `-p 8080:80`: Maps port 8080 on the host to port 80 in the container
- `--name my-webapp`: Names the container

## Persistent Data with Volumes

Docker containers are ephemeral, meaning any data inside them is lost when they're removed. Volumes solve this problem.

### Creating and Using Volumes

```bash
# Create a named volume
docker volume create my-data

# Run a container with the volume mounted
docker run -it -v my-data:/data ubuntu:latest bash
```

Inside the container, any data written to `/data` will persist in the `my-data` volume.

### Bind Mounts

You can also mount host directories into containers:

```bash
# Mount a host directory to a container directory
docker run -it -v $(pwd)/host-data:/container-data ubuntu:latest bash
```


## Docker Compose for Multi-Container Applications

Docker Compose simplifies managing multi-container applications.

### Example Docker Compose File

```yaml
# docker-compose.yml
version: '3'

services:
  web:
    build: ./web
    ports:
      - "8080:80"
    volumes:
      - web-data:/var/www/html
    depends_on:
      - db
    networks:
      - app-network

  db:
    image: mysql:8.0
    volumes:
      - db-data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: mydb
      MYSQL_USER: user
      MYSQL_PASSWORD: password
    networks:
      - app-network

networks:
  app-network:

volumes:
  web-data:
  db-data:
```

### Using Docker Compose

```bash
# Start all services defined in docker-compose.yml
docker-compose up

# Run in detached mode
docker-compose up -d

# Stop all services
docker-compose down

# Stop services and remove volumes
docker-compose down -v
```

## One-Line Setup Commands

For quick setup, here are one-line commands combining multiple steps:

### Basic Ubuntu Container Setup
```bash
docker pull ubuntu:22.04 && docker run -d --name my-ubuntu -v my-data:/data ubuntu:22.04 tail -f /dev/null && docker exec -it my-ubuntu bash
```

### Web Server Setup
```bash
echo "<html><body><h1>Hello from Docker!</h1></body></html>" > index.html && docker run -d --name nginx-ubuntu -p 8080:80 -v $(pwd)/index.html:/var/www/html/index.html ubuntu:22.04 bash -c "apt-get update && apt-get install -y nginx && nginx -g 'daemon off;'"
```
