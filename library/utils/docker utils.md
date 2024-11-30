# 1

Here's a comprehensive list of useful Docker commands:

1. **Basic Operations**:
```bash
# Build image
docker build -t myapp .
docker build -f path/to/Dockerfile -t myapp .

# Run container
docker run -d -p 8000:8000 myapp  # Detached mode with port mapping
docker run -it myapp bash         # Interactive mode with bash
docker run --name mycontainer myapp  # Named container

# Stop/Start/Restart
docker stop <container_id>
docker start <container_id>
docker restart <container_id>
```

2. **Container Management**:
```bash
# Execute command in running container
docker exec -it <container_id> bash

# Copy files to/from container
docker cp ./local/path <container_id>:/container/path
docker cp <container_id>:/container/path ./local/path

# View container logs
docker logs <container_id>
docker logs -f <container_id>  # Follow log output
docker logs --tail 100 <container_id>  # Last 100 lines

# Container resource usage
docker stats
docker top <container_id>  # Show running processes
```

3. **Image Management**:
```bash
# Pull/Push images
docker pull image:tag
docker push myimage:tag

# Tag images
docker tag source:tag target:tag

# Save/Load images
docker save myimage > myimage.tar
docker load < myimage.tar

# Image history and details
docker history myimage
docker inspect myimage
```

4. **Cleanup Commands**:
```bash
# Remove containers
docker rm <container_id>
docker rm $(docker ps -aq)  # Remove all containers

# Remove images
docker rmi <image_id>
docker rmi $(docker images -q)  # Remove all images

# Clean up system
docker system prune  # Remove unused data
docker system prune -a  # Remove all unused data
docker volume prune  # Remove unused volumes
```

5. **Network Commands**:
```bash
# List networks
docker network ls

# Create network
docker network create mynetwork

# Connect container to network
docker network connect mynetwork <container_id>

# Inspect network
docker network inspect mynetwork
```

6. **Volume Commands**:
```bash
# Create volume
docker volume create myvolume

# List volumes
docker volume ls

# Mount volume
docker run -v myvolume:/container/path myimage

# Bind mount
docker run -v $(pwd):/container/path myimage
```

7. **Compose Commands**:
```bash
# Start services
docker-compose up
docker-compose up -d  # Detached mode

# Stop services
docker-compose down

# View logs
docker-compose logs

# Scale services
docker-compose up -d --scale service=3
```

8. **Debugging & Troubleshooting**:
```bash
# View container details
docker inspect <container_id>

# Resource usage
docker stats

# View port mappings
docker port <container_id>

# View changes in container
docker diff <container_id>
```

9. **Environment & Configuration**:
```bash
# Set environment variables
docker run -e "VAR=value" myimage

# List environment variables
docker exec <container_id> env

# Override entrypoint
docker run --entrypoint="/bin/bash" myimage
```

10. **Resource Constraints**:
```bash
# Memory limits
docker run --memory="1g" myimage

# CPU limits
docker run --cpus=".5" myimage

# Storage limits
docker run --storage-opt size=10G myimage
```

Remember:
- Use `--help` with any command for more options
- Container IDs can be shortened to first few unique characters
- Use tab completion for container/image IDs
- Consider using docker-compose for complex multi-container applications
- Always clean up unused resources to save space

These commands should cover most common Docker operations!


--- 
# 2

Here are the key Docker commands for listing containers and images:

1. **Container Commands**:
```bash
# List running containers
docker ps

# List all containers (including stopped ones)
docker ps -a

# List containers with specific format
docker ps -a --format "table {{.ID}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}"

# List only container IDs
docker ps -aq
```

2. **Image Commands**:
```bash
# List all downloaded images
docker images
# or
docker image ls

# List all images including intermediary layers
docker images -a

# List images with specific format
docker images --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"

# List dangling images (untagged images)
docker images -f "dangling=true"
```

3. **Cleanup Commands**:
```bash
# Remove specific container
docker rm <container_id>

# Remove all stopped containers
docker container prune

# Remove specific image
docker rmi <image_id>

# Remove all unused images
docker image prune

# Remove everything (containers, images, networks, cache)
docker system prune -a
```

4. **Filter Commands**:
```bash
# Filter images by reference
docker images "fraud*"

# Filter containers by status
docker ps -a -f "status=exited"

# Filter images by date
docker images -f "before=fraud:latest"
```

Common options:
- `-a` or `--all`: Show all (including stopped containers or intermediate images)
- `-q` or `--quiet`: Only display numeric IDs
- `-f` or `--filter`: Filter output based on conditions
- `--format`: Pretty-print container/image output using a Go template
