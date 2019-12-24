# Awesome Docker and K8S
Collection of containers knowledge

## Prepare working environment with Docker
- Window
  - Install from [official website](https://hub.docker.com/editions/community/docker-ce-desktop-windows)
  - Older versions of Windows or Windows 10 Home edition cannot run Docker for Windows.
    - Hyper-V is not available on older versions of Windows nor is it available in the Home edition.
    - They have to install [Docker Toolbox](https://docs.docker.com/toolbox/toolbox_install_windows/) for handle older versions
    - Docker Toolbox installs VirtualBox on our laptop, which is used to run the Linux VMs we need.
- Mac
  - Install from [official website](https://docs.docker.com/docker-for-mac/install/)
    - This will already include Hypervisor for VM like [hyperkit](https://github.com/moby/hyperkit) in it 
    
### Using a package manager
- Window
  - [Cocolatey](https://chocolatey.org/)
- Mac
  - [Homebrew](https://brew.sh/)
    
## Docker Architecture 
> Containers must run on a Linux host. Neither Windows or Mac can run containers natively

![Docker Architecture](https://docs.docker.com/engine/images/architecture.svg)
 
### 3 main players in docker 
Docker uses a client-server architecture
  - The Docker client (docker)
    - the primary way that many Docker users interact with Docker
    - sends these commands to dockerd, which carries them out
  - The Docker daemon (dockerd) 
    - listens for Docker API requests and manages Docker objects
  - The Docker registries
    - stores Docker images. Docker Hub is a public registry that anyone can use, and Docker is configured to look for images on Docker Hub by default
  
## Container
> Containers leverage a lot of features and primitives available in the Linux OS. The most important ones are namespaces and cgroups

![Architecuture](https://learning.oreilly.com/library/view/learn-docker-/9781788997027/assets/496ad42f-1bb7-44ef-8a79-a75317bc0e8c.png)

### Namespace
A namespace is an abstraction of global resources such as filesystems, network access, process tree (also named PID namespace) or the system group IDs, and user IDs

A Linux system is initialized with a single instance of each namespace type. After initialization, additional namespaces can be created or joined.The PID namespace is what keeps processes in one container from seeing or interacting with processes in another container

![Example Namespace](https://learning.oreilly.com/library/view/learn-docker-/9781788997027/assets/e3ecc488-ad6c-4530-83db-751451b5e3e4.png)

### Cgroups
Linux cgroups are used to limit, manage, and isolate resource usage of collections of processes running on a system. Resources are CPU time, system memory, network bandwidth, or combinations of these resources, and so on.

Using cgroups, administrators can limit the resources that containers can consume, to avoid the classical noisy neighbor problem.

### Container plumbing 
#### Runc
Runc is a lightweight, portable container runtime for spawning and running containers.

#### Containerd
Containerd builds on top of Runc, and adds higher-level features, such as image transfer and storage, container execution, and supervision, as well as network and storage attachments

### Union filesystem (UnionFS)
The UnionFS forms the backbone of what is known as container images.
#### The layered filesystem

```
FROM ubuntu:18.04
COPY . /app
RUN make /app
CMD python /app/app.py
```
This Dockerfile contains four commands, each of which creates a layer.

![Container Layers](https://docs.docker.com/storage/storagedriver/images/container-layers.jpg)

Each layer is only a set of differences from the layer before it. The layers are stacked on top of each other. When you create a new container, you add a new writable layer on top of the underlying layers. This layer is often called the ... container layer

![Sharing Layers](https://docs.docker.com/storage/storagedriver/images/sharing-layers.jpg)

The major difference between a container and an image is the top writable layer. All writes to the container that add new or modify existing data are stored in this writable layer. When the container is deleted, the writable layer is also deleted. The underlying image remains unchanged.

#### The writable container layer
##### Copy-on-write
If a layer uses a file or folder that is available in one of the low-lying layers, then it just uses it. If, on the other hand, a layer wants to modify, say, a file from a low-lying layer, then it first copies this file up to the target layer and then modifies it

### Anatomy of the docker container run expression
> docker container run alphine echo "Hello World"

- docker 
  - docker cli
- container
  - context
- run
  - command
- alphine
  - container image
- echo "Hello World"
  - process to run inside contaienr
  
### Useful command for interacting with container
#### Inspect
> docker inspect -f '{{.Json-Key}}' <<Name/ID>>

Basic command get information data

> docker inspect -f '{{json .Json-Key}}' <<Name/ID>> | jq

We can combined with grep and filter
- Inspecting container
  - docker using [`Go Template`](https://golang.org/pkg/text/template/) syntax 
  - and use [`jq`](https://stedolan.github.io/jq/) for process with JSON
 
[Example](https://gist.github.com/howtoautomateinth/c2e74527e0c09d2c2a7f1b67e8ff316c) full low-level information 

#### Exec
> docker container exec -i -t <<Name/ID>> /bin/sh 

- The flag -i signifies that we want to run the additional process interactively, and -t tells Docker that we want it to provide us with a TTY (a terminal emulator) for the command. Finally, the process we run is /bin/sh.

#### Attach
> docker container attach <<Name/ID>> 

we can attach our Terminal's standard input, output, and error (or any combination of the three) to a running container using the ID or name of the container

#### Diff
> docker container diff demo

Use the diff command to find out what has changed in the container's filesystem in relation to the filesystem of the image

## Createing Images
There are 2 ways to create a new container image on your system
- Interactive image to creation new one
  - we can create a custom image is by interactively building a container
  - e.g. with first creation of alpine image it won't have ping tool installed so we can install it and commit and create custom image
  - helpful when doing exploration, creating prototypes, or making feasibility studies
- DockerFiles
  - It contains instructions on how to build a custom container image. It is a declarative way of building images.
  - Declarative versus Imperative
    - Declarative Apporach = defining a task
    - Imperative Apporach = expected outcome and lets the system figure out how to achieve this goal
    
### Using DockerFiles

```
FROM python:2.7
RUN mkdir -p /app
WORKDIR /app
COPY ./requirements.txt /app/
RUN pip install -r requirements.txt
CMD ["python", "main.py"]
```
![Image Layer](https://learning.oreilly.com/library/view/learn-docker-/9781788997027/assets/39b09cae-4f1c-4084-ba9c-4b5484cab215.jpg)

Command List
- FROM
  - Define which base image we want to start building our custom image from
- RUN 
  - The argument for RUN is any valid Linux command
- COPY and ADD
  - Add some content to an existing base image to make it a custom image
  - From a security perspective, it is important to know that by default, all files and folders inside the image will have a user ID (UID) and a group ID (GID) of 0.
    - so, we can change the ownership that the files will have inside the image using the optional --chown flag
- WORKDIR
  - Defines the working directory or context that is used when a container is run from our custom image
  - if we didn't set it will produce file in the root of the image filesystem but if we it like in above DockerFile "WORKDIR /app" it will produce in that directory
- CMD and ENTRYPOINT
  - ENTRYPOINT is used to define the command of the expression (exec form)
  - CMD is used to define the parameters for the command 
  - Difference [1](https://stackoverflow.com/questions/47904974/what-are-shell-form-and-exec-form)
    - if we use only CMD like CMD ["python", "main.py"] it called shell form 
    - if we use ENTRYPOINT and CMD it called exec form 

#### Dockerfile best practices
- containers are meant to be ephemeral. 
  - can be stopped and destroyed and a new one built and put in place with an absolute minimum of setup and configuration
  - minimum initialize the application time
- we should order the individual commands in the Dockerfile so that we leverage caching as much as possible
```
FROM node:9.4
RUN mkdir -p /app
WORKIR /app
COPY . /app
RUN npm install
CMD ["npm", "start"]
```
to this
```
FROM node:9.4
RUN mkdir -p /app
WORKIR /app
COPY package.json /app/
RUN npm install
COPY . /app
CMD ["npm", "start"]
```
we only copy the single file that the npm install command needs as a source, which is the package.json file so it's mean if anythings change just reinstall it otherwise just copy it from previous build (layer cache)

- try to keep the number of layers that make up your image relatively small
  - The more layers an image has, the more the graph driver needs to work to consolidate the layers into a single root filesystem for the corresponding container and that takes time
```
RUN apt-get update
RUN apt-get install -y ca-certificates
RUN rm -rf /var/lib/apt/lists/*
```
to
```
RUN apt-get update \
    && apt-get install -y ca-certificates \
    && rm -rf /var/lib/apt/lists/*
```
- reduce the image size is by using a .dockerignore file to avoid copying unnecessary files and folders into an image to keep it as lean as possible
- avoid installing unnecessary packages into the filesystem of the image
- use multistage builds so that the resulting image is as small as possible

#### Multi Stage Build
technique that have some stages that are used to build the final artifacts and then a final stage where we use the minimal necessary base image and copy the artifacts into the real custom image instead to make "small images"

```
FROM alpine:3.7
RUN apk update &&
apk add --update alpine-sdk
RUN mkdir /app
WORKDIR /app
COPY . /app
RUN mkdir bin
RUN gcc -Wall hello.c -o bin/hello
CMD /app/bin/hello
```
to
```
FROM alpine:3.7 AS build
RUN apk update && \
    apk add --update alpine-sdk
RUN mkdir /app
WORKDIR /app
COPY . /app
RUN mkdir bin
RUN gcc hello.c -o bin/hello

FROM alpine:3.7
COPY --from=build /app/bin/hello /app/hello
CMD /app/hello
```

#### Image namespaces
we can setup your own namespace like this
```
<registry URL>/<User or Org>/<name>:<tag>
https://registry.acme.com/engineering/web-app:1.0
```
or use other public registery
 - Google https://cloud.google.com/container-registry
 - Microsoft https://azure.microsoft.com/en-us/services/container-registry/
 
## Data Volumes
2 ways to interact with data volumes
- named volume
  - create a named volume which can then be mounted into a container and be used for persistent data access or storage 
  - > docker volume create named-my-data 
``` $ docker volume inspect named-my-data 
[
    {
        "CreatedAt": "2019-12-24T04:35:44Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/named-my-data/_data",
        "Name": "named-my-data",
        "Options": {},
        "Scope": "local"
    }
]
```
  -- The host folder can be found in the output under Mountpoint.
  -- and then we can mount it with then all the data that are going to write in that container will write in named-my-data instead
```
docker container run --name test -it \
     -v named-my-data:/data alpine /bin/sh
```
  - removing volume
```
docker volume rm named-my-data 
```
  - Sharing data between containers
    - an application running in container A produces some data that will be consumed by another application running in container B
    - to provide race conditions on concurrency problems we have to set it to read-only mode for reader and writer
    - using keyword ":ro" read-only on the reader node
```
docker container run -it --name reader \
    -v named-shared-data:/app/data:ro \
    ubuntu:17.04 /bin/bash
```
- host volume
  - it is very useful to use volumes that mount a specific host folder
    - developing new containerized application
    - a legacy application that needs to consume data from a certain folder produced
    - Edit-and-continue experience
    - If you make changes on the host they will be propagated to the container and vice versa
```
docker container run --rm -it \
    -v $(pwd)/src:/app/src \
    alpine:latest /bin/sh
```
  - $(pwd) means current directory in [linux command](https://www.howtoforge.com/linux-pwd-command/)
    - $() mean [command substitution](http://www.tldp.org/LDP/abs/html/commandsub.html) in linux. other word is execute subshell and the output is then placed in the original command
    - there also have ${} mean [parameter substitution](https://superuser.com/questions/935374/difference-between-and-in-shell-script)
    
```
$ animal=cat
$ echo $animals
                                # No such variable as “animals”.
$ echo ${animal}s
cats
$ echo $animal_food
                                # No such variable as “animal_food”.
$ echo ${animal}_food
cat_food
```
### Volumes in images
Some applications, such as databases running in containers, need to persist their data beyond the lifetime of the container. In this case they can use volumes
```
VOLUME /app/data 
VOLUME /app/data, /app/profiles, /app/config 
VOLUME ["/app/data", "/app/profiles", "/app/config"] 
```
Docker automatically creates a volume and mounts it to the corresponding target folder of the container for each path defined in the Dockerfile
  - the folders defined as volumes in the Dockerfile are excluded from the union filesystem and thus any changes in those folders do not change the container layer but are persisted to the respective volume

## Single-Host Networking
### Network drivers
- bridge
  - default network driver
  - apply to containers running on the same Docker daemon host
![bridge](https://learning.oreilly.com/library/view/learn-docker-/9781788997027/assets/12001484-deda-4e11-b3f1-d0c7b0f1bf4d.png)
- host
  - host network
  - container’s network stack is not isolated from the Docker host (the container shares the host’s networking namespace), and the container does not get its own IP-address allocated (use host IP)
- overlay
  - creates a distributed network among multiple Docker daemon hosts
  - container different Docker daemon hosts, you can either manage routing at the OS level, or you can use an overlay network
- macvlan
  - Some applications, especially legacy applications or applications which monitor network traffic, expect to be directly connected to the physical network
  - network driver to assign a MAC address to each container’s virtual network interface
- none
  - completely disable the networking stack on a container
  
### Port management
![port mgmt.](https://learning.oreilly.com/library/view/learn-docker-/9781788997027/assets/c52dccf3-b0ef-42ac-a484-e9352f0b9fd8.png)
> docker container run --name web2 -p 8080:80 -d nginx:alpine

-p parameter for <host port>:<container port>

## Futhur Reading
- Special thanks to [Learn Docker Fundamental 18.x](https://learning.oreilly.com/library/view/learn-docker-/9781788997027/)
- [Docker Documentation](https://docs.docker.com/engine/docker-overview/)
