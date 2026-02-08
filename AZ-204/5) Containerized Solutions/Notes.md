# Containerized Solutions

# Azure Container Registry 
Azure Container Registry is a managed registry service based on the open-source Docker Registry 2.0.  This is to store container image and related images.
- You can configure images to be rebuilt when on check in to keep them in sync. 
- You can pull images from Azure container registry to deploy from you CI/CD pipelines.  

| SKU          | Description                                                                                                                                                                                                                                                                                                                                           |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Basic**    | A cost-optimized entry point for developers learning about Azure Container Registry. Basic registries have the same programmatic capabilities as Standard and Premium. However, the included *storage* and image *throughput* are most appropriate for *lower usage scenarios*.                                                                       |
| **Standard** | Standard registries offer the same capabilities as Basic, with increased included storage and image throughput. Standard registries *should satisfy the needs of most production scenarios*.                                                                                                                                                          |
| **Premium**  | - Highest amount of included storage and *concurrent* operations, enabling *high-volume* to help wit  large-scale concurrent deployments.<br><br>- High availability and resiliency through *geo-replication* for managing a single registry across multiple regions,<br><br>- *Private link* with private endpoints restrict access to the registry. |
All container registry tiers:
- **Encryption-at-rest:** : Azure automatically encrypts an image before storing it and decrypts them when services pull images.
- **Regional storage:** Azure Container Registry stores data in the region where the registry is created, to help customers meet data residency and compliance requirements. Unless geo replication is enabled in the premium tier.
- **Zone redundancy:** A feature of the Premium service tier allow uses availability zones to replicate your registry to a minimum of three separate zones in each enabled region.
## ACR Task 

> [!NOTE] 
> Even though you could do a lot of task: build, deploy and test containers. I prefer the flow of doing it in my CI/CD pipeline.  I could run unit test, integration test, build container and then deploy to container registry without needing to use ACR Tasks. May be helpful in more niche deployment scenarios.

Azure Container Registry tasks support several scenarios to build and maintain container images and other artifacts. By default, Azure Container Registry tasks build images for the Linux OS and the AMD64 architecture. Specify the `--platform` tag to build Windows images or Linux images for other architectures. There are multiple types of task. 
### Quick Task
`az ac build` - command then sends the context (source code) to Azure Container Registry and (by default) pushes the built image to its registry upon completion.

Allows for quick developer loop since changes doesn't have to be in source control. Also since it's using remote builds you don't have to have docker installed on you machine.

``` bash
az acr build \
	--registry $ACR_NAME \
	--image helloacrtasks:v1 \
	--file /path/to/Dockerfile/path/to/build/context.
```
### Automatically triggered task 
You can generate build of containers if certain conditions are met:
1)  *Source code update*
   
```bash
az acr task create \ 
	--registry $ACR_NAME \ 
	--name taskhelloworld \ 
	--image helloworld:{{.Run.ID}} \ 
	--context https://github.com/$GIT_USER/acr-build-helloworld-node.git
	--file Dockerfile \ 
	--git-access-token $GIT_PAT
```

Trigger task to test

```bash 
az acr task run --registry $ACR_NAME --name taskhelloworld
```
   
2) *Rebuild when base image has been updated*.   Create an application image in the same registry to track the base image. You may need to manually copy this in your docker hub base image into the ACR. Once the base container image updates in your ACR it triggers an update on other containers that use that base image.
   
```bash 
az acr task create\
	--registry $ACR_NAME \
	--name baseexample1 \
	--image helloworld:{{.Run.ID}} \
	--arg REGISTRY_NAME=$ACR_NAME.azurecr.io \
	--context https://github.com/$GIT_USER/acr-build-helloworld-node.git#master \
	--file Dockerfile-app \
	--git-access-token $GIT_PAT
```

3) *Based on Cron schedule* 
 
``` bash
az acr task create
 --registry cloudengineerskillscliacr
 --name nodehelloworldtimertask
 --image node-hello-world
 --context https://github.com/jarrodlilkendey/acr-trigger-on-source.git#master
 --file Dockerfile
 --git-access-token <GIT-ACCESS-TOKEN>
 --schedule "* * * * *"
```
### Multi-step Tasks
Extend the single-image build-and-push capability of Azure Container Registry tasks with multi-step workflows that are based on multiple containers.
1. Build a web application image.
2. Run the web application container.
3. Build a web application test image.
4. Run the web application test container.

```YAML
version: v1.1.0
steps:
  - build: -t $Registry/hello-world:$ID .
  - push: ["$Registry/hello-world:$ID"]
```

You can use the azure cli to initiate a build to generate a new container image.

```bash
az acr build --registry myRegistry --image myimage:latest
```
