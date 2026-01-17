# Docker

Containers are lightweight and contain everything needed to run the application, so you don't need to rely on what's installed on the host.

> [!NOTE] 
> One best practice for containers is that each container should do one thing and do it well. While there are exceptions to this rule, avoid the tendency to have one container do multiple things.
## Docker Architecture 
Docker uses a client-server architecture. The Docker client talks to the Docker daemon, which does the heavy lifting of building, running, and distributing your Docker containers. The Docker client and daemon communicate using a REST API, over UNIX sockets or a network interface.

![](Images/Pasted%20image%2020260117133212.png)

- *Docker Daemon*-  listens for Docker API requests and manages Docker objects such as images, containers, networks, and volumes.
- *Docker Client* - CLI to interact with daemon
- *Docker Desktop* - provides a GUI and includes:  Docker daemon (`dockerd`), the Docker client (`docker`), Docker Compose, Docker Content Trust, Kubernetes, and Credential Helper.
- *Docker Registry*- Stores docker images and Docker Hub is public registry.  Or private registry like Azure Container Registry.
## Docker Objects

![](Images/Pasted%20image%2020260117135303.png)

- *DockerFile* - This is the source code for a docker image. It contains all the dependencies required of the container and the application itself. For example for a .NET Web API docker file will contain: .NET Runtime, Steps to Build and Publish file and entry point of the application.
  
- *Image* -  image is a standardized package that includes all of the files, binaries, libraries, and configurations to run a container. All the publish artifacts. Images are immutable. Once an image is created, it can't be modified. You can only create new ones.
  
  Each layer represents a set of file system changes that add, remove, or modify files.. When you change the DockerFile and rebuild the image, only those layers which have changed are rebuilt. This is part of what makes images so lightweight, small, and fast.

  Images are stored in a public registries that can be pulled down. You can create images from other images. If you are building a .NET  app, you can start from the .NET image and add additional layers to specify your application code to be run.

```DockerFile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src
COPY ["RedisAPI/RedisAPI.csproj", "RedisAPI/"]
RUN dotnet restore "./RedisAPI/RedisAPI.csproj"
COPY . .
WORKDIR "/src/RedisAPI"
RUN dotnet build "./RedisAPI.csproj" -c $BUILD_CONFIGURATION -o /app/build
```

- *Container* -  is a isolate process of your application running. It's a runnable instance form an image. By default, a container is relatively well isolated from other containers and its host machine.  You can control how isolated a container's network, storage from other container or host system. 

## Docker Run Internals

```bash
docker run -i -t ubuntu /bin/bash
```

1) If you don't have the `ubuntu` image locally, Docker pulls it from your configured registry, as though you had run `docker pull ubuntu` manually.
2) Docker creates a new container from the pulled image a `docker container create` command manually.
3) Docker allocates a read-write filesystem to the container, as its final layer. Allow container to modify files in it's local filesystem.
4) Docker creates a network interface to connect the container to the default network. This includes assigning an IP address to the container. By default, containers can connect to external networks using the host machine's network connection.
5) Docker starts the container and executes `/bin/bash`. This will open up bash terminal and attach it your session so you could provide input to your container.
6) When you run `exit` to terminate the `/bin/bash` command, the container stops but isn't removed
## Docker Compose
Allows you to run multiple containers in based on a single YAML file. To save you run the command to run each container individually. 

You will have create user defined network that is shared so containers can communicate, run Redis on that network along with .NET Web API.

```bash 
docker network create mynet
docker run -d --name redis --network mynet -p 6379:6379 redis
docker run -d --name redisapi --network mynet -p 5000:8080 redisapi
```

You can use this single command to spin up both Redis and .NET Web API instance with YAML below. Since it's in the same compose file they'll be on the same network and communicate with each other by default.

``` bash
docker compose up
```

```YAML
version: "3.9"

services:
  redisapi:
    image: redisapi
    build:
      context: .
      dockerfile: RedisAPI/Dockerfile

    ports:
      - "8080:80"     # HTTP
      - "8443:443"    # HTTPS

    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ASPNETCORE_URLS=https://+:443;http://+:80
      - ASPNETCORE_Kestrel__Certificates__Default__Password=TEST12
      - ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx

    volumes:
      - /mnt/c/Users/asuudy/.aspnet/https:/https:ro

    depends_on:
      - redis

  redis:
    image: redis
    container_name: redis
    ports:
      - "6379:6379"
    command: ["redis-server", "--appendonly", "yes"]
    volumes:
      - redisdata:/data

volumes:
  redisdata:

```


## Docker File

## Running Containers Manually 

*Redis*
``` bash
docker pull redis:latest

docker run -d --name redis -p 6379:6379 redis:latest
```

*.NET API API*
```bash
#Docker build process to create a Docker image
docker build -t redisapi -f RedisAPI/Dockerfile .

#creates a new container
docker create --name redisapi redisapi

#Run Container
docker run -d --name redisapi -p 5000:8080 redisapi
```

## Networking in Containers 
User-defined networks allows you connect groups of containers to the same network.

```bash 
docker network create mynet
docker run -d --name redis --network mynet -p 6379:6379 redis
docker run -d --name redisapi --network mynet -p 5000:8080 redisapi
```
## Docker Compose 

### Creating Certificate for Kestrel to work in Docker Container
https://learn.microsoft.com/en-us/aspnet/core/security/docker-compose-https?view=aspnetcore-10.0

```powershell
mkdir "$env:USERPROFILE\.aspnet\https" -Force

dotnet dev-certs https 
-ep "$env:USERPROFILE\.aspnet\https\aspnetapp.pfx"  
-p {PASSWORD}

dotnet dev-certs https --trust
``` 