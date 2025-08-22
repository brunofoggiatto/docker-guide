# Docker - Quick Reference Guide

Simple guide for beginners and quick lookup for developers.

## Table of Contents

1. [What is Docker](#what-is-docker)
2. [Installation](#installation)
3. [Images](#images)
4. [Containers](#containers)
5. [Volumes](#volumes)
6. [Networks](#networks)
7. [Dockerfile](#dockerfile)
8. [Docker Compose](#docker-compose)
9. [Monitoring](#monitoring)
10. [System Cleanup](#system-cleanup)

---

## What is Docker?

Docker packages applications with their dependencies into lightweight, portable containers. This solves the "Just works on my computer" problem.

**Key concepts:**
- **Image**: Template to create containers 
- **Container**: Running instance of an image 
- **Dockerfile**: Instructions to build an image
- **Registry**: Storage for images (Docker Hub)

---

## Installation

```bash
# Linux (Ubuntu/Debian)
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
# Please, after installation, reboot your machine

# Add user to docker group (avoid sudo)
sudo usermod -aG docker $USER

# Verify installation
docker --version
docker run hello-world
```

---

## Images

```bash
# List local images
docker images

# Download image from Docker Hub
docker pull nginx:latest

# Search Docker Hub for images
docker search nginx

# # Build an image from a Dockerfile (we will learn what a Dockerfile is later) 
docker build -t myapp:1.0 .
# -t: tag with name:version
# ".": build context (current directory)

# Tag an image (create alias)
docker tag myapp:1.0 myapp:latest

# Remove image
docker rmi nginx:latest

# View image details
docker inspect nginx:latest

# View image layers
docker history nginx:latest
```
NOTE: Pay attention when downloading any image from Docker Hub, as many of them may contain vulnerabilities. We will learn how to create our own images.

---

## Containers

### Creating and Running

```bash
# Create container without starting
docker create --name myapp nginx:latest
# Useful for preparing containers with specific config

# Run container (creates and starts)
docker run nginx:latest

# Run in background
docker run -d --name web nginx
# -d: detached mode
# --name: custom name

# Run with port mapping
docker run -d -p 8080:80 --name web nginx
# -p: host_port:container_port

# Run interactively
docker run -it ubuntu:20.04 /bin/bash
# -i: interactive
# -t: terminal

# Run with environment variables
docker run -d -e NODE_ENV=production --name app node:18

# Run with volume mount
docker run -d -v /host/data:/container/data --name app nginx
```

### Managing Containers

```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Start stopped container
docker start myapp

# Stop running container
docker stop myapp

# Restart container
docker restart myapp

# Remove container
docker rm myapp

# Force remove running container
docker rm -f myapp

# Remove all stopped containers
docker container prune
```

### Interacting with Containers

```bash
# Execute command in running container
docker exec -it myapp /bin/bash
# Most common: open shell

# Execute single command
docker exec myapp ls -la

# View container logs
docker logs myapp

# Follow logs in real-time
docker logs -f myapp

# Copy files between host and container
docker cp file.txt myapp:/path/
docker cp myapp:/path/file.txt ./
```

---

## Volumes

Volumes provide persistent storage that survives container restarts.

```bash
# Create named volume
docker volume create myvolume

# List volumes
docker volume ls

# Use named volume
docker run -d -v myvolume:/data --name app nginx

# Use host directory (bind mount)
docker run -d -v $(pwd):/app --name app nginx
# $(pwd): current directory

# Mount as read-only
docker run -d -v /host/path:/container/path:ro --name app nginx

# Remove volume
docker volume rm myvolume

# Remove unused volumes
docker volume prune
```

---

## Networks

Networks allow containers to communicate with each other.

```bash
# List networks
docker network ls

# Create custom network
docker network create mynetwork

# Run container on specific network
docker run -d --network mynetwork --name web nginx

# Connect existing container to network
docker network connect mynetwork mycontainer

# Remove network
docker network rm mynetwork

# Remove unused networks
docker network prune
```

---

## Dockerfile

Instructions to build custom images.
Frist, create a Dockerfile:
```bash
nano/vim Dockerfile
```

### Common Instructions
```dockerfile
FROM node:18-alpine          # Base image
WORKDIR /app                 # Set working directory
COPY source dest             # Copy files
ADD source dest              # Copy + extract archives
RUN command                  # Execute command during build
ENV NODE_ENV=production      # Set environment variable
EXPOSE 3000                  # Document port (doesn't publish)
CMD ["npm", "start"]         # Default command
ENTRYPOINT ["node"]          # Always executed command
USER appuser                 # Change user
VOLUME ["/data"]             # Create mount point
```

### Example 01
```Dockerfile
# Base image
# "Alpine" images are smaller and more lightweight
FROM node:18-alpine 

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy source code
COPY . .

# Expose port
EXPOSE 80

# Start command
CMD ["npm", "start"]
```

### Example 02
```Dockerfile
# Base Image
FROM debian

# Install Apache2
RUN apt-get update && apt-get install -y apache2 && apt-get clean

# Labels
LABEL description="Webserver"
LABEL version="1.0.0"

# Expose HTTP port
EXPOSE 80
VOLUME /var/www/html/
```

Inside the folder you created Dockerfile, build your image 
### Build Commands
```bash
# Build image
docker build -t myapp:1.0 .

# Build with custom Dockerfile
docker build -f custom.dockerfile -t myapp:1.0 .

# Build without cache
docker build --no-cache -t myapp:1.0 .
```
NOTE: List images ```docker image ls``` to verify the image you created works
If it works, you can run a container now ```docker container run -ti myapp:1.0```
And verify if apache is working on your container ```dpkg -l | grep apache"```

---

## Docker Compose

Manage multi-container applications with YAML files.

### Basic docker-compose.yml
```yaml
version: '3.8'

services:
  web:
    build: .                    # Build from Dockerfile
    ports:
      - "8080:80"              # Port mapping
    volumes:
      - .:/app                 # Bind mount
    environment:
      - NODE_ENV=production    # Environment variable
    depends_on:
      - database               # Start order

  database:
    image: mysql:8.0           # Use existing image
    environment:
      MYSQL_ROOT_PASSWORD: password123
      MYSQL_DATABASE: myapp
    volumes:
      - db_data:/var/lib/mysql # Named volume

volumes:
  db_data:                     # Define named volume
```

### Compose Commands
```bash
# Start all services
docker-compose up -d

# Stop and remove
docker-compose down

# View logs
docker-compose logs -f

# Build services
docker-compose build

# Start and build
docker-compose up -d --build

# Scale service
docker-compose up -d --scale web=3

# Execute command in service
docker-compose exec web bash

# List services
docker-compose ps
```

---

## Monitoring

```bash
# View resource usage
docker stats
# Shows CPU, memory, network, disk usage

# View system info
docker system info

# Check disk usage
docker system df

# View container details
docker inspect myapp

# View running processes in container
docker top myapp
```

---

## System Cleanup

```bash
# Remove stopped containers
docker container prune

# Remove unused images
docker image prune

# Remove unused volumes
docker volume prune

# Remove unused networks
docker network prune

# Remove build cache
docker builder prune

# Clean everything unused
docker system prune

# Aggressive cleanup (removes all unused images)
docker system prune -a

# Nuclear option (includes volumes)
docker system prune -a --volumes

# Stop all containers
docker stop $(docker ps -q)

# Remove all containers
docker rm $(docker ps -aq)
```

---

## Quick Tips

- **Always use specific image versions** in production: `nginx:1.21` instead of `nginx:latest`
- **Run containers as non-root user** for security
- **Use health checks** for production containers
- **Prefer Alpine images** for smaller size: `node:18-alpine`
- **Use multi-stage builds** to reduce final image size
- **Clean up regularly** to save disk space

**Author:** Bruno H. Foggiatto
