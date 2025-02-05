# 1 docker cli

Here's a comprehensive list of useful Docker cli commands:

1. **Basic Operations**:
```bash
# Build image
# bsaic command. '.' is build context. this is what is copied when COPY . . is used. everything inside this is accesible only to the container. 
docker build .
# t is tag name.
docker build -t myapp .
docker build -f path/to/Dockerfile -t myapp .
# no cache means cached layers are not used. fresh build everytime
docker build --no-cache  -f=./Dockerfile -t vizdum .  

# Run container
docker run -d -p 8000:8000 myapp  # Detached mode with port mapping device-port:container-port
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
# 2 containers and images management and cleanup

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


# 3. dockerfile 

**Dockerfile Command Reference**

**1. Basics**

`FROM <image>  `

- Sets the **base image** for the build
- Example: FROM ubuntu:20.04

`LABEL <key>=<value>`

- Adds **metadata** to the image
- Example: LABEL maintainer="user@example.com"

`ARG <name>=<default_value>`

- Defines a **build-time variable**
- Example: ARG VERSION=1.0
- Pass with: docker build --build-arg VERSION=2.0 .

`ENV <key>=<value>`

- Sets an **environment variable** available at build and runtime
- Example: ENV APP_ENV=production

**2. Files and Directories**

`COPY <src> <dest>`

- Copies files from **host to image**
- Example: COPY ./app /app

`ADD <src> <dest>`

- Like COPY, but can **unpack tar files** or download URLs
- Example: ADD archive.tar.gz /app

`WORKDIR <path>`

- Sets the **working directory** for commands
- Example: WORKDIR /app

**3. Execution**

`RUN <command>`

- Executes **shell commands** during build
- Example: RUN apt-get update && apt-get install -y nginx

`CMD ["executable", "arg1", "arg2"]`

- **Default command** to run when the container starts
- Example: CMD ["nginx", "-g", "daemon off;"]
- Can be overridden by docker run <image> <command>.

`ENTRYPOINT ["executable", "arg1"]`

- Similar to CMD, but **always executed** (cannot be easily overridden)
- Example: ENTRYPOINT ["python", "app.py"]

`EXPOSE <port>`

- Documents the **port** the container listens on
- Example: EXPOSE 8080

- Does **not** publish the port automatically (use -p).

**4. Container Settings**

`VOLUME ["/path"]`

- Defines mount points for **persistent storage**
- Example: VOLUME ["/data"]

`USER <username>`

- Switches to a **specific user** inside the container
- Example: USER appuser

`HEALTHCHECK [OPTIONS] CMD <command>`

- Defines a **health check** for the container
- Example: HEALTHCHECK CMD curl --fail http://localhost || exit 1

**5. Optimization**

`ONBUILD <command>`

- Specifies a command to run **when the image is used as a base** for another build.

`SHELL ["/bin/bash", "-c"]`

- Changes the default shell used by RUN.

**6. Default Execution Order**

1. FROM

2. ARG

3. LABEL

4. ENV

5. COPY/ADD

6. WORKDIR

7. RUN

8. EXPOSE

9. VOLUME

10. ENTRYPOINT

11. CMD

**7. Best Practices**

- Minimize layers by chaining commands:

RUN apt-get update && apt-get install -y nginx && rm -rf /var/lib/apt/lists/*

- Use .dockerignore to exclude unnecessary files
- Keep images **small** by using lightweight base images like alpine
- Leverage multi-stage builds to reduce final image size.