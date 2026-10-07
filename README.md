# Docker -- Complete Practical Guide

## 1. What is Docker?

Docker is a platform used to **package, distribute, and run applications
in containers**.

A Docker container contains the application and the dependencies it
needs to run, such as:

-   Application code
-   Runtime
-   Libraries
-   Configuration
-   System tools required by the application

The main goal is:

> **Build once, run consistently anywhere.**

For example, a .NET application may require a particular .NET runtime,
libraries, environment variables, and configuration. Docker packages
these requirements into an image so the application can run consistently
on a developer laptop, test server, Azure VM, or Kubernetes cluster.

------------------------------------------------------------------------

# 2. Why Do We Need Docker?

Before Docker, a common problem was:

``` text
Developer Laptop
    |
    | "It works on my machine"
    v
Application Server
    |
    | Different OS / Runtime / Library / Configuration
Application fails
```

Typical problems:

-   Different operating system versions
-   Different runtime versions
-   Missing libraries
-   Different configuration
-   Dependency conflicts
-   Difficult application deployment
-   Long server setup time
-   "Works in development but not production"
-   Applications competing for different versions of the same dependency

Docker solves many of these problems by packaging the application and
its dependencies together.

``` text
Without Docker

Application
   |
   +-- Runtime
   +-- Libraries
   +-- OS dependencies
   +-- Configuration
   |
   v
"Environment must be configured correctly"
```

``` text
With Docker

+--------------------------------+
|          Container             |
|                                |
| Application                    |
| Runtime                        |
| Libraries                      |
| Required dependencies          |
+--------------------------------+

Same image can be used across environments.
```

------------------------------------------------------------------------

# 3. What Problems Does Docker Solve?

## 3.1 Environment Consistency

The same Docker image can be used in:

``` text
Developer
    |
    v
Testing
    |
    v
UAT
    |
    v
Production
```
![Docker Architecture](images\dokcer-problem.png)


This reduces environment differences.

------------------------------------------------------------------------

## 3.2 Dependency Management

Suppose Application A needs:

``` text
Node.js 18
```

and Application B needs:

``` text
Node.js 22
```

Installing both directly on the same server can create conflicts.

With containers:

``` text
Docker Host
|
+-- Container A
|      |
|      +-- Node.js 18
|
+-- Container B
       |
       +-- Node.js 22
```

Both applications can run independently.

------------------------------------------------------------------------

## 3.3 Faster Deployment

Traditional VM deployment:

``` text
Create VM
   |
Install OS
   |
Install Runtime
   |
Install Dependencies
   |
Configure Application
   |
Configure Service
   |
Deploy Application
```

Docker:

``` text
Docker Image
     |
     v
docker run
     |
     v
Application Running
```

------------------------------------------------------------------------

# 4. Docker vs Virtual Machine

This is one of the most important Docker concepts.

## Virtual Machine Architecture

``` text
+--------------------------------------+
| Application A | Application B        |
+--------------------------------------+
| Guest OS A     | Guest OS B          |
+--------------------------------------+
| Hypervisor                            |
+--------------------------------------+
| Host Operating System                 |
+--------------------------------------+
| Physical / Cloud Infrastructure       |
+--------------------------------------+
```

Each VM generally contains its own guest operating system.

------------------------------------------------------------------------

## Docker Container Architecture

``` text
+--------------------------------------+
| Container A | Container B | C        |
| Application | Application | App      |
+--------------------------------------+
| Docker Engine                         |
+--------------------------------------+
| Host Operating System                 |
+--------------------------------------+
| Physical / Cloud Infrastructure       |
+--------------------------------------+
```

Containers share the host OS kernel rather than requiring a complete
guest OS for each application.

------------------------------------------------------------------------

# 5. Docker Architecture

The basic Docker architecture contains:

1.  Docker Client
2.  Docker Host / Docker Engine
3.  Docker Daemon
4.  Docker Images
5.  Docker Containers
6.  Docker Registry

``` text
                         Docker Registry
                      +-------------------+
                      | Docker Hub / ACR   |
                      +---------+---------+
                                |
                         pull / push
                                |
                                v
+--------------------------------------------------+
|                  Docker Host                     |
|                                                  |
|  +-------------+        +--------------------+   |
|  | Docker CLI  | -----> | Docker Daemon       |   |
|  | docker      | API    | dockerd             |   |
|  +-------------+        +---------+----------+   |
|                                    |              |
|                      +-------------+----------+   |
|                      |                        |   |
|                      v                        v   |
|                +-----------+            +-----------+
|                | Container |            | Container |
|                | App A     |            | App B     |
|                +-----------+            +-----------+
|                                                  |
+--------------------------------------------------+
```

------------------------------------------------------------------------

# 6. Docker Components

## 6.1 Docker Client

The Docker CLI is what you normally interact with.

Examples:

``` bash
docker ps
docker images
docker pull nginx
docker run nginx
```

The CLI communicates with the Docker daemon using Docker APIs.
------------------------------------------------------------------------

# 7. How Docker Works

Consider:

``` bash
docker run -d -p 8080:80 nginx
```

Docker performs approximately these steps:

``` text
docker run
    |
    v
Check local image
    |
    +---- Image exists ----> Use it
    |
    +---- Image missing
              |
              v
         Pull image
              |
              v
       Create container
              |
              v
       Configure network
              |
              v
       Configure filesystem
              |
              v
       Start container
```

The application then listens on the container's port.

------------------------------------------------------------------------

# 8. Dockerfile

A Dockerfile contains instructions for building an image.

Example for a simple Node.js application:

``` dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

Build:

``` bash
docker build -t my-node-app:1.0 .
```

Run:

``` bash
docker run -d -p 3000:3000 my-node-app:1.0
```

------------------------------------------------------------------------

# . Dockerfile Important Instructions

## FROM

Defines the base image.

``` dockerfile
FROM ubuntu:24.04
```

or:

``` dockerfile
FROM nginx:alpine
```

------------------------------------------------------------------------

## WORKDIR

Sets the working directory.

``` dockerfile
WORKDIR /app
```

------------------------------------------------------------------------

## COPY

Copies files from the build context into the image.

``` dockerfile
COPY . .
```

------------------------------------------------------------------------

## RUN

Executes a command while building the image.

``` dockerfile
RUN npm install
```

------------------------------------------------------------------------

## EXPOSE

Documents the port that the application listens on.

``` dockerfile
EXPOSE 8080
```

Important:

> `EXPOSE` does not publish the port to the host by itself.

You still need:

``` bash
docker run -p 8080:8080 myapp
```

------------------------------------------------------------------------

## CMD

Defines the default command when the container starts.

``` dockerfile
CMD ["npm", "start"]
```

------------------------------------------------------------------------

## ENTRYPOINT

Defines the main executable for the container.

Example:

``` dockerfile
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

------------------------------------------------------------------------

# 13. CMD vs ENTRYPOINT

A common interview question.

### CMD

Provides a default command or default arguments.

``` dockerfile
CMD ["nginx", "-g", "daemon off;"]
```

### ENTRYPOINT

Defines the main executable.

``` dockerfile
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

They can also be combined:

``` dockerfile
ENTRYPOINT ["dotnet"]
CMD ["MyApp.dll"]
```

Conceptually:

``` text
ENTRYPOINT = Main executable
CMD        = Default arguments
```

------------------------------------------------------------------------

# 14. Docker Container Lifecycle

``` text
                  docker create
                       |
                       v
                    Created
                       |
                  docker start
                       |
                       v
                    Running
                   /       \
                  /         \
        docker stop       docker kill
              |                |
              v                v
           Stopped          Stopped
              |
          docker start
              |
              v
           Running

Running
   |
docker rm
   |
   v
Removed
```

Common states:

``` text
Created
Running
Paused
Stopped
Exited
Removed
```

------------------------------------------------------------------------

# 15. Important Docker Commands

## Check Docker Version

``` bash
docker --version
```

More detailed:

``` bash
docker version
```

``` bash
docker info
```

## Give Container a Name

``` bash
docker run -d --name web-server nginx
```

------------------------------------------------------------------------

## Port Mapping

``` bash
docker run -d -it --name web-server -p 8080:80 nginx
```

------------------------------------------------------------------------

## List Running Containers

``` bash
docker ps
```

------------------------------------------------------------------------

## List All Containers

``` bash
docker ps -a
```

------------------------------------------------------------------------

## Stop Container

``` bash
docker stop web-server
```

------------------------------------------------------------------------

## Start Container

``` bash
docker start web-server
```

------------------------------------------------------------------------

## Restart Container

``` bash
docker restart web-server
```

------------------------------------------------------------------------

## Remove Container

``` bash
docker rm web-server
```

Force removal:

``` bash
docker rm -f web-server
```

------------------------------------------------------------------------

## Remove Image

``` bash
docker rmi nginx
```

------------------------------------------------------------------------

## View Container Logs

``` bash
docker logs web-server
```

Follow logs:

``` bash
docker logs -f web-server
```

Last 100 lines:

``` bash
docker logs --tail 100 web-server
```

------------------------------------------------------------------------

## Inspect Container

``` bash
docker inspect web-server
```

This is very useful for troubleshooting.

It can show:

-   IP address
-   Network
-   Mounts
-   Environment variables
-   Ports
-   Container configuration

------------------------------------------------------------------------

## Execute Command Inside Container

``` bash
docker exec -it web-server /bin/bash
```

For Alpine images:

``` bash
docker exec -it web-server /bin/sh
```

Example:

``` bash
docker exec -it web-server ls -la
```

------------------------------------------------------------------------

# 17. Push Image to Docker Registry

Login:

``` bash
docker login
```

Tag:

``` bash
docker tag myapp:1.0 username/myapp:1.0
```

Push:

``` bash
docker push username/myapp:1.0
```

Pull:

``` bash
docker pull username/myapp:1.0
```

------------------------------------------------------------------------

# 18. Azure Container Registry Example

For Azure environments, Azure Container Registry (ACR) is commonly used.

``` text
Developer
    |
    | docker build
    v
Docker Image
    |
    | docker tag
    v
Azure Container Registry
    |
    | docker pull
    v
Azure VM / AKS
```

Typical commands:

``` bash
az login
```

Login to ACR:

``` bash
az acr login --name <acr-name>
```

Build:

``` bash
docker build -t myapp:1.0 .
```

Tag:

``` bash
docker tag myapp:1.0 <acr-name>.azurecr.io/myapp:1.0
```

Push:

``` bash
docker push <acr-name>.azurecr.io/myapp:1.0
```

------------------------------------------------------------------------

# 19. Docker Networking

Docker provides networking so containers can communicate with each other
and with external systems.

Common network types:

-   bridge
-   host
-   none
-   overlay

List networks:

``` bash
docker network ls
```

Inspect a network:

``` bash
docker network inspect bridge
```

Create a network:

``` bash
docker network create app-network
```

Run container:

``` bash
docker run -d --name app --network app-network myapp
```

Another container:

``` bash
docker run -d --name database --network app-network mysql
```

Containers attached to the same user-defined network can communicate
using container/service names.

Example:

``` text
+---------------- Docker Network ----------------+

+-------------+          +----------------+
| App         | -------> | Database       |
| container   |          | container      |
+-------------+          +----------------+
       app                    database

Application can use:

database:3306

+-------------------------------------------------+
```

------------------------------------------------------------------------

# 20. Docker Volumes

Container filesystem data can be lost when a container is removed.

For persistent data, use volumes.

``` text
Container
    |
    | mount
    v
Docker Volume
    |
    v
Persistent Data
```

Create volume:

``` bash
docker volume create mydata
```

List volumes:

``` bash
docker volume ls
```

Run container with volume:

``` bash
docker run -d \
  --name mysql \
  -v mydata:/var/lib/mysql \
  mysql
```

Here:

``` text
mydata
   |
   +----> /var/lib/mysql inside container
```

The volume survives container deletion.

------------------------------------------------------------------------

# 21. Bind Mount

You can also mount a host directory.

``` bash
docker run -d \
  -v /host/path:/container/path \
  nginx
```

Example:

``` bash
docker run -d \
  -v /home/user/html:/usr/share/nginx/html \
  -p 8080:80 \
  nginx
```

Difference:

 ``` Volume                      
  Managed by Docker Good for application data Docker manages location 
  ```
 ``` Bind Mount
   Managed by user/OSGood for sharing host files User specifies host path
   ```


------------------------------------------------------------------------

# 25. Docker vs Kubernetes

Docker and Kubernetes are not the same thing.

### Docker

Primarily used to:

-   Build images
-   Run containers
-   Manage containers
-   Manage local/containerized workloads

### Kubernetes

Used to orchestrate containers at scale.

``` text
Docker
   |
   +-- Build container image
   +-- Run container
   +-- Manage container
```

``` text
Kubernetes
   |
   +-- Deploy containers
   +-- Scale containers
   +-- Self-healing
   +-- Service discovery
   +-- Load balancing
   +-- Rolling updates
   +-- Scheduling
```

A common enterprise flow:

``` text
Developer
    |
    v
Dockerfile
    |
    v
Docker Build
    |
    v
Container Image
    |
    v
Azure Container Registry
    |
    v
Azure Kubernetes Service (AKS)
    |
    +-- Pod
    +-- Pod
    +-- Pod
```
# 31. Docker Troubleshooting Flow

When a container is not working, follow this order:

``` text
Container issue
      |
      v
docker ps -a
      |
      v
Check container status
      |
      v
docker logs <container>
      |
      v
docker inspect <container>
      |
      v
Check port mapping
      |
      v
Check application configuration
      |
      v
Check environment variables
      |
      v
Check network
      |
      v
Check volume/mount
      |
      v
Check resource usage
```

Useful commands:

``` bash
docker ps -a
docker logs <container>
docker inspect <container>
docker stats
docker network ls
docker network inspect <network>
docker volume ls
docker events
```

------------------------------------------------------------------------

# 32. Resource Monitoring

Check running containers:

``` bash
docker ps
```

Check CPU and memory:

``` bash
docker stats
```

Example:

``` text
CONTAINER       CPU %       MEM USAGE
api             5.2%        150MiB
database        12.1%       500MiB
frontend        1.5%        80MiB
```

------------------------------------------------------------------------

# 33. Clean Up Unused Docker Resources

Show disk usage:

``` bash
docker system df
```

Clean unused resources:

``` bash
docker system prune
```

More aggressive cleanup:

``` bash
docker system prune -a
```

**Be careful:** aggressive cleanup can remove unused images and other
resources that you may still want.

------------------------------------------------------------------------

# 34. Useful Command Cheat Sheet

  Task                  Command
  --------------------- ----------------------------------
  Docker version        `docker --version`
  Docker details        `docker info`
  Pull image            `docker pull nginx`
  List images           `docker images`
  Build image           `docker build -t app:1.0 .`
  Run container         `docker run nginx`
  Run detached          `docker run -d nginx`
  List containers       `docker ps`
  List all containers   `docker ps -a`
  Stop                  `docker stop <name>`
  Start                 `docker start <name>`
  Restart               `docker restart <name>`
  Remove container      `docker rm <name>`
  Remove image          `docker rmi <image>`
  Logs                  `docker logs <name>`
  Follow logs           `docker logs -f <name>`
  Execute command       `docker exec -it <name> /bin/sh`
  Inspect               `docker inspect <name>`
  Resource usage        `docker stats`
  Networks              `docker network ls`
  Volumes               `docker volume ls`
  Docker disk usage     `docker system df`
  Compose start         `docker compose up -d`
  Compose stop          `docker compose down`
  Compose logs          `docker compose logs`

------------------------------------------------------------------------

# 35. Most Important Docker Commands to Remember for Interviews

If you are preparing for an interview, remember these first:

``` bash
docker pull
docker build
docker images
docker run
docker ps
docker ps -a
docker stop
docker start
docker restart
docker rm
docker rmi
docker logs
docker exec
docker inspect
docker stats
docker network
docker volume
docker login
docker tag
docker push
docker pull
docker compose
```

------------------------------------------------------------------------

# 36. Docker End-to-End CI/CD Architecture

A typical enterprise deployment can look like this:

``` text
                 Developer
                     |
                     v
              Git Repository
                     |
                     v
                CI Pipeline
                     |
              docker build
                     |
                     v
              Security Scan
                     |
                     v
             Docker Image
                     |
                     v
        +-----------------------+
        | Container Registry    |
        | Azure Container       |
        | Registry (ACR)        |
        +-----------+-----------+
                    |
                    | docker pull
                    v
             Deployment System
                    |
          +---------+---------+
          |                   |
          |              
                             AKS
          |                   |
          v                   v
     Container             Pods
          |                   |
          +---------+---------+
                    |
                    v
              Application
```

------------------------------------------------------------------------

# 38. Important Interview Questions

## Q1. What is Docker?

Docker is a containerization platform that packages an application and
its dependencies into a portable image and runs it as an isolated
container.

------------------------------------------------------------------------

## Q2. Why use Docker?

Main reasons:

-   Consistent environments
-   Dependency isolation
-   Portability
-   Faster deployments
-   Efficient resource utilization
-   Easier CI/CD
-   Application isolation
-   Simplified scaling

------------------------------------------------------------------------

## Q3. What is the difference between image and container?

``` text
Image     = Template
Container = Running instance of the image
```

------------------------------------------------------------------------

## Q4. What is Dockerfile?

A Dockerfile is a text file containing instructions used to build a
Docker image.

------------------------------------------------------------------------

## Q5. What is Docker Hub?

Docker Hub is a public container image registry.

------------------------------------------------------------------------

## Q6. What is Docker Compose?

Docker Compose is used to define and manage multi-container applications
using a YAML configuration file.

------------------------------------------------------------------------

## Q7. How does Docker provide isolation?

Docker uses OS-level isolation mechanisms such as namespaces and control
groups (cgroups), along with filesystem and network isolation.

------------------------------------------------------------------------

## Q8. Why are containers faster than VMs?

Containers share the host OS kernel and do not require a separate full
guest OS for every application.

------------------------------------------------------------------------

## Q9. What happens when a container is deleted?

The writable container layer is deleted. Data stored only inside that
writable layer can be lost.

Data stored in Docker volumes can persist.

------------------------------------------------------------------------

## Q10. How do you troubleshoot a container?

Start with:

``` bash
docker ps -a
docker logs <container>
docker inspect <container>
docker exec -it <container> /bin/sh
docker stats
docker network inspect <network>
```

------------------------------------------------------------------------

# 39. Docker Mental Model

The easiest way to remember Docker is:

``` text
Dockerfile
    |
    | docker build
    v
  IMAGE
    |
    | docker run
    v
CONTAINER
    |
    | docker push
    v
REGISTRY
    |
    | docker pull
    v
Another Environment
```

And:

``` text
Image = Blueprint
Container = Running Application
Volume = Persistent Storage
Network = Communication
Registry = Image Storage
Dockerfile = Image Build Instructions
Compose = Multi-container Definition
```

------------------------------------------------------------------------

# 40. Final Summary

Docker solves the problem of **environment inconsistency and difficult
application deployment** by packaging an application and its
dependencies into a portable container image.

The core workflow is:

``` text
              Dockerfile
                   |
                   v
             docker build
                   |
                   v
                IMAGE
                   |
                   v
              docker run
                   |
                   v
              CONTAINER
                   |
                   +------> Network
                   |
                   +------> Volume
                   |
                   +------> Environment
                   |
                   v
             Application
```

For enterprise deployments:

``` text
Source Code
    |
    v
CI/CD
    |
    v
Docker Build
    |
    v
Security Scan
    |
    v
Container Registry
    |
    +----------+
    |          |
    v          v
   VM          AKS
Container     Pods
    |          |
    +----------+
         |
         v
     Application
```

## The five concepts you should know first

1.  **Dockerfile** -- instructions to build an image
2.  **Image** -- immutable package/template
3.  **Container** -- running instance of an image
4.  **Volume** -- persistent data
5.  **Network** -- communication between containers

Once these five concepts are clear, Docker becomes much easier to
understand.
