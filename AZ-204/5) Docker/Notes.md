# Docker
## Container 

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