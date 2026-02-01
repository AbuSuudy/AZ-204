# Docker

Container is light weight virtualisation technology that can hold your application and it's dependencies need to for the application to run. The container will get most of it's resources from the host kernel which makes it light weight. Since containers needs to interact with the host kernel the container need to be based on the same OS to environment it's being deployed to.

Ubuntu based container comes with these components. This mostly utilities that allow you manage the container. 

> [!NOTE] 
> There could be trimmed version of image that doesn't come with: Shell, package manager etc for security. To reduce the attack space especially when you don't need to use these tools

| **Category**             | **Specific Components**                                                                                                                                                           |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **The Shells**           | `/bin/bash`                                                                                                                                                                       |
| **Package Management**   | The `apt` database, `/etc/apt/sources.list`, and the `dpkg` binary.                                                                                                               |
| **GNU Coreutils**        | Essential binaries: `cat`, `chown`, `cp`, `date`, `dd`, `df`, `echo`, `grep`, `hostname`, `id`, `ls`, `mkdir`, `mv`, `pwd`, `rm`, `sed`, `sleep`, `tar`.                          |
| **System Configuration** | `/etc/passwd`, `/etc/group`, `/etc/os-release`, and `/etc/hostname`.                                                                                                              |
| **Shared Libraries**     | Basic C libraries (`libc6`), `libselinux1`, and `libtinfo6` to support the binaries above.                                                                                        |
| **Directory Tree**       | A full standard hierarchy: `/bin`, `/boot`, `/dev`, `/etc`, `/home`, `/lib`, `/media`, `/mnt`, `/opt`, `/proc`, `/root`, `/run`, `/sbin`, `/srv`, `/sys`, `/tmp`, `/usr`, `/var`. |
There extra dependencies are installed with .NET SDK Ubuntu based [Docker image](https://github.com/dotnet/dotnet-docker/blob/bb1c8cd2b964c36e7279ccea4c4831fef8b83295/src/sdk/10.0/noble/amd64/Dockerfile#L37C1-L43C35)

```DOCKERFILE
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        curl \
        git \
        libatomic1 \
        wget \
    && rm -rf /var/lib/apt/lists/*
```
## Docker Architecture 
Docker uses a client-server architecture. The Docker client talks to the Docker daemon which does the heavy lifting. The Docker client and daemon communicate using a REST API, over UNIX sockets or a network interface.

![](Images/Pasted%20image%2020260117133212.png)

- *Docker Daemon*-  listens for Docker API requests and manages Docker objects such as images, containers, networks, and volumes.
- *Docker Client* - CLI to interact with daemon
- *Docker Desktop* - provides a GUI and includes:  Docker daemon (`dockerd`), the Docker client (`docker`), Docker Compose, Docker Content Trust, Kubernetes, and Credential Helper.
- *Docker Registry*- Stores docker images and Docker Hub is public registry Or private registry like Azure Container Registry.
## Docker Objects

![](Images/Pasted%20image%2020260117135303.png)

- *DockerFile* - This is the source code that generates the docker image. It contains all the dependencies required of the container and the application itself. 
  
- *Image* - Image is a standardized package that includes all of the files, binaries, libraries, and configurations to run a container. Images are immutable and it's use to upload to a container registry. Each layer represents a set of file system changes that add, remove, or modify files. Each layer is saved a `.tar` in the blob in blob folder. Once extracted will contain all the binaries need to run the app. `OverlayFS` layers two directories on a single Linux host and presents them as a single directory and used. 

![500](Images/Pasted%20image%2020260125210842.png)

If you extract all the blob files it will reveal the all directory and files in the container 
![](Images/Pasted%20image%2020260126001744.png)

Is the location of the .NET runtime 
```
\usr\share\dotnet\shared
```

![](Images/Pasted%20image%2020260126002634.png)

With way I setup my DockerFile the the publish artifact of my API is in App folder. 
![](Images/Pasted%20image%2020260126002817.png)

- *Container* -  is a running instance of  docker image.  
## Common Commands 
### Run 
```bash
docker run ubuntu
```

1) If you don't have the `ubuntu` image locally, Docker pulls it from your configured registry, as though you had run `docker pull ubuntu` manually.
2) Docker creates a new container from the pulled image a `docker container create` command manually.
3) Docker allocates a read-write filesystem to the container, as its final layer. Allow container to modify files in it's local filesystem.
4) Docker creates a network interface to connect the container to the default network. This includes assigning an IP address to the container. By default, containers can connect to external networks using the host machine's network connection.

```bash
docker run -i -t ubuntu /bin/bash
```

Does the same as above but adds to two extra flags:
- Docker starts the container and executes `/bin/bash`. This will open up bash terminal and attach it your session so you could provide input to your container. So you can debug issues within the running container.
### Images
``` bash
#Pull docker image from docker hub 
docker pull redis:latest

#Or you can create Image from DockerFile
docker build -t redisapi -f RedisAPI/Dockerfile .

#List docker images
docker images

#Remove images
docker rmi redisapi
```
### Containers
``` shell
#creates a new container from the specified image
docker create --name redisapi 

#Start container
docker start  redisapi 

#list running containers
docker ps

#list all containers
docker ps -a

#Remove Container
docker rm redisapi
```
### Clean up 
Event if you  remove the container there are still lagging object: volume, network and cache
``` bash 
#Remove Image
docker rmi redisapi

#Remove Container
docker rm redisapi

#Remove orphaned: network, cache
docker network prune
docker builder prune

#Remove orphaned volumes
#Will need do additional command to remove named volumes
docker volume prune
```

## DockerFile

### Multi Layer Docker File

## Networking in Docker
You may have solution that consist of different container. You will have to think about how they connect together. For example, a frontend container may have external access, and a `--internal`network to communicate with containers running backend services that do not need external network access.
### Types of Network
You can create networks yourself and each driver is type of network with its own use case.

| Driver                                                             | Description                                                         |
| ------------------------------------------------------------------ | ------------------------------------------------------------------- |
| [bridge](https://docs.docker.com/engine/network/drivers/bridge/)   | The default network driver.                                         |
| [host](https://docs.docker.com/engine/network/drivers/host/)       | Remove network isolation between the container and the Docker host. |
| [none](https://docs.docker.com/engine/network/drivers/none/)       | Completely isolate a container from the host and other containers.  |
| [overlay](https://docs.docker.com/engine/network/drivers/overlay/) | Swarm Overlay networks connect multiple Docker daemons together.    |
| [ipvlan](https://docs.docker.com/engine/network/drivers/ipvlan/)   | Connect containers to external VLANs.                               |
| [macvlan](https://docs.docker.com/engine/network/drivers/macvlan/) | Containers appear as devices on the host's network.                 |
### Bridge
When you run a container without the `--network` option, it is connected to the default bridge network.

![500](Images/Pasted%20image%2020260126220745.png)

The network will have subnet mask and each connected device will get IP address from that range. All resources connected to the bridge have access to each other, but only if you have the IP address of the resource. The bridge doesn't allow for name resolution. For outbound internet access it will egress the host network card. When sending packets, if the destination is an address in a directly connected network, packets are sent to that network. Otherwise, packets are sent to a default gateway for routing to their destination.

![400](Images/Pasted%20image%2020260130180115.png)
### User Defined Network
Containers that attach to a custom network use Docker's embedded DNS server. The embedded DNS server forwards external DNS lookups. Which allows for name resolution. The hostname defaults to be the container's ID in Docker..  Containers use the same DNS servers as the host by default. The config for the DNS server could be found  `\etc\resolv.conf`
  ![](Images/Pasted%20image%2020260130173941.png)
Resources have to explicitly be ask to be put on it which allows you plan and organise your network. Provides good network isolation by default. 

![400](Images/Pasted%20image%2020260130181037.png)

## DockerFile

#ToDo

```Dockerfile
# See https://aka.ms/customizecontainer to learn how to customize your debug container and how Visual Studio uses this Dockerfile to build your images for faster debugging.

# This stage is used when running from VS in fast mode (Default for Debug configuration)
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
USER $APP_UID
WORKDIR /app
EXPOSE 8080
EXPOSE 8081

# This stage is used to build the service project
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src
COPY ["RedisAPI/RedisAPI.csproj", "RedisAPI/"]
RUN dotnet restore "./RedisAPI/RedisAPI.csproj"
COPY . .
WORKDIR "/src/RedisAPI"
RUN dotnet build "./RedisAPI.csproj" -c $BUILD_CONFIGURATION -o /app/build

# This stage is used to publish the service project to be copied to the final stage
FROM build AS publish
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "./RedisAPI.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false

# This stage is used in production or when running from VS in regular mode (Default when not using the Debug configuration)
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "RedisAPI.dll"]
```

## Docker Compose
Allows you to run multiple containers in based on a single YAML file. To save you run the command to run each container individually. You will have create user defined network that is shared so containers can communicate, run Redis on that network along with .NET Web API.

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
      - ASPNETCORE_Kestrel__Certificates__Default__Password={{password}}
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

To use https in docker compose you'll have to create a certificate and use the that certificate and password in compose YAML.

https://learn.microsoft.com/en-us/aspnet/core/security/docker-compose-https?view=aspnetcore-10.0

```powershell
mkdir "$env:USERPROFILE\.aspnet\https" -Force

dotnet dev-certs https 
-ep "$env:USERPROFILE\.aspnet\https\aspnetapp.pfx"  
-p {PASSWORD}

dotnet dev-certs https --trust
``` 
## Looking into .NET Docker File
.NET default image is based on Ubuntu from `amd64/buildpack-deps:noble-curl`. This image is ubuntu based image with some dependencies via the package manager. 
- *.NET Docker File* : https://github.com/dotnet/dotnet-docker/blob/main/src/sdk/10.0/noble/amd64/Dockerfile
- *Base Image* : https://github.com/docker-library/buildpack-deps/blob/master/ubuntu/noble/curl/Dockerfile
## Dot Net
https://github.com/dotnet/dotnet-docker/blob/main/documentation/supported-tags.md#multi-platform-tags
- tags
- defaults to ubuntu 
- MCR it's not stored on docker hub but Microsoft container registry 
- `docker image inspect mcr.microsoft.com/dotnet/aspnet:8.0` to inspect package 
- Docket file https://github.com/dotnet/dotnet-docker/blob/main/src/sdk/10.0/noble/amd64/Dockerfile
- Base image used `amd64/buildpack-deps:noble-curl` https://github.com/docker-library/buildpack-deps/blob/master/ubuntu/noble/curl/Dockerfile
- https://docs.docker.com/reference/cli/docker/buildx/build/#platform
- https://docs.docker.com/build/building/multi-platform/

