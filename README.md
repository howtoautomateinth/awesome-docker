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

![Example Namespace](https://learning.oreilly.com/library/view/learn-docker-/9781788997027/assets/e3ecc488-ad6c-4530-83db-751451b5e3e4.png)

### Cgroups
Linux cgroups are used to limit, manage, and isolate resource usage of collections of processes running on a system. Resources are CPU time, system memory, network bandwidth, or combinations of these resources, and so on.

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

## Createing Images
### Using DockerFiles
Command List
- FROM
- RUN 
- COPY
- etc.

## Data Volumes
...

## Single-Host Networking
...

## Futhur Reading
- [Learn Docker Fundamental 18.x](https://learning.oreilly.com/library/view/learn-docker-/9781788997027/)
