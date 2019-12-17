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

## Futhur Reading
- [Learn Docker Fundamental 18.x](https://learning.oreilly.com/library/view/learn-docker-/9781788997027/)
