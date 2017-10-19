# Introduction to Docker

Docker setup for developers and some basic guidelines.

![Docker L](sources/docker-logo-compressed.png "Docker")

## Install Docker

Please refer to this [documentation](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/) for the latest instruction.

## What is Docker?

Docker is a platform for developing, shipping and running applications using container virtualization technology.

### Problems with single application server

- slow deployment times
- huge cost
- wasted resources
- difficult to scale
- difficult to migrate
- vendor lock in

### Introducing containers

Container based virtualization uses the kernel on the host's operating system to run multiple guest instances.

- each guest instance is called a **container**
- each container has its own:
  - root filesystem
  - processes
  - memory
  - devices
  - network ports

### Container vs Virtual Machines

- containers are more lightweight
- no need to install guest OS
- less CPU, RAM, storage space required
- more containers per machine than VMs
- greater portability

## Docker Concepts and Terms

### Docker engine

Docker engine is the program that enables containers to be built, shipped and run. It uses Linux Kernel namespaces and control groups that give us the isolated workspace.

example namespaces: pid, net, ipc, mnt, uts

Verify docker engine installation by running:

```shell
> docker run hello-world
> docker version            # checking client and daemon version
```

### Docker images

- read only template used to create containers
- built by docker users
- stored in the docker hub or local registry

### Docker containers

- isolated application platform
- contains everything needed to run application
- based on one or more images

### Registry and repository

- where images are stored
- public registry is also available (https://hub.docker.com/)

### Benefits of docker

- separation of concerns
  - developers focus on building the apps
  - devOps focus on deployment
- fast development cycle
- application portability
  - build in one environment, ship to another
- scalability
  - easily spin up new containers if needed
- run more apps on one host machine

## Getting Started with Images

Images are available in docker hub. When creating new container, docker would first find image from local machine and if it can't find the image then it will download the image from registry (private or public).

Images are specified by **repository:tag**. Some images may have multiple tags and the default tag is **latest**. Example:

```shell
# to display local image
> docker images
# REPOSITORY        TAG        IMAGE ID        CREATED        VIRTUAL SIZE
# hello-world       latest     e45a5af57b00    1 month ago    910 B
```

## Getting Started with Containers

Use **docker run** command to create a container. Example:

```shell
> docker run ubuntu:16.04 echo "hello world"
> docker run ubuntu ps ax
```

Use *-it* flags with **docker run** to create container with terminal access.

- -i flag tells docker to connect to STDIN on the container
- -t flag specifies to get a pseudo-terminal
- after exiting terminal,  it kills the process and running **docker run** again will spin off new container

```shell
> docker run -it ubuntu:16.04 /bin/bash
```

### Container process

- a container only runs as long as the process from specified **docker run** command is running
- command's process is always PID 1 inside the container

### Container ID

- containers can be specified using their ID or name
- short ID and name can be obtained using docker ps command to list running containers
- *-a* flag to list all containers (including containers that are stopped)

```shell
> docker ps
> docker ps -a
```

### Running containers in detached mode

Use *-d* flag to run container in the background or as a daemon and use **docker logs [container id]** to observe output.

```shell
> docker run -d centos:7 ping 127.0.0.1 -c 50
> docker logs -f 62ba075bee18
```

### Running a web application inside a container

Accessing web server in docker container through host web browser. Use *-P* flag to map container ports to host ports. Example:

```shell
> docker run -d -P tomcat:9
> docker ps                    # run docker ps to display mapped port
# PORTS
# 0.0.0.0:49155->8080/tcp
```

## Building Images

- images are comprised of multiple layers
- a layer is also just another image
- every image contains a base layer
- docker uses a copy on write system
- layers are read only

### Method 1: committing image

Use docker commit command to save changes in a container as a new image. Repository name should be based on username/application and container can be referenced using eith name or ID. Example:

```shell
# save the container with ID abcd1234 as a new image in the repository user/myapp with tag 1.0
> docker commit abcd1234 user/myapplication:1.0
```

### Method 2: Dockerfile

A Dockerfile is a configuration file that contains instructions for building a docker image.

- provides a more effective way to build images compared to using **docker commit**
- easily fits into continuous integration and deployment process

Dockerfile is made up of instructions. Instructions specify what to do when building the image.

**Instructions example:**

- **FROM** specifies what the base image should be 
- **RUN** specifies a command to execute
  - each **RUN** instructions will execute the command on the top writable layer and perform a commit of the image
  - can aggregate multiple **RUN** by using "&&"
- **CMD** defines a default command to execute when a container is created
  - performs no action during the image build
  - can only be specified once in a Dockerfile
  - can be overridden at run time
  - use shell and EXEC format
  ```dockerfile
  # shell format
  CMD ping 127.0.0.1 -c 30
  # EXEC format
  CMD ["ping", "127.0.0.1", "-c", "30"]
  ```
- **ENTRYPOINT** defines the command that will run when a container is executed
  - cannot be overridden at run time
  - run time arguments and **CMD** instruction are passed as parameters to the **ENTRYPOINT** instruction
  - EXEC format is preferred as shell format cannot accept arguments at run time
- more instructions are available at https://docs.docker.com/engine/reference/builder/

**Example Dockerfile:**

```dockerfile
FROM ubuntu:16.04
RUN apt-get update && apt-get install -y \
     curl \
     vim
CMD ["hello world"]
ENTRYPOINT ["echo"]
```

**Build Dockerfile:**

```shell
> docker build [options] [build context]    # build context is the path of the Dockerfile

# common option to tag the build
> docker build -t [repository:tag] [build context]

# example
> docker build -t user/myapp:1.0 .
> docker build -t user/myapp:1.0 myproject
```

## Managing Images and Containers

### Start and stop containers

```shell
> docker ps -a                   # list all containers
> docker start [container id]    # start a container
> docker stop [container id]     # stop a container
```

### Getting terminal access

Use **docker exec** command to start another process within a container and exiting from the terminal will not terminate the container.

```shell
> docker ps -a                                # list all containers
> docker exec -it [container id] /bin/bash    # execute /bin/bash to get a bash shell
```

### Deleting containers

Can only delete containers that have been stopped.

```shell
> docker ps -a                # list all containers
> docker rm [container id]
```

### Deleting local images

Remove each tag if an image is tagged multiple times.

```shell
> docker images                  # list all images
> docker rmi [image id]
> docker rmi [repository:tag]
```

### Push local images to a registry

Local repository must have same name and tag as the registry repository.

```shell
> docker push [repository:tag]
```

If repository name and tag are different, then use docker tag to rename a local image repository before pushing to registry.

```shell
> docker tag [image id] [registry repository:tag]
> docker tag [local repository:tag] [registry repository:tag]
```

## Docker Volumes

A volume is a designated directory in a container, which is designed to persist data, independent of the container's life cycle. Volume changes are excluded when updating an image but it will still persist even when a container is deleted. Volume also can be mapped to a host folder and shared between containers.

### Mount a volume

- volumes are mounted when creating and executing a container
- volume paths specified must be absolute

```shell
# execute a new container and mount the folder /myvolume into its file system
> docker run -d -P -v /myvolume nginx:1.13

# execute a new container and map the /data/src from the host into the /test/src in the container
> docker run -d -P -v /data/src:/test/src nginx:1.13
```

### Volumes in Dockerfile

- **VOLUME** instruction creates a mount point
- can specify arguments JSON array or string
- cannot map volumes to host directories
- volumes are initialized when the container is executed

```dockerfile
# string example
VOLUME /myvolume

# string example with multiple volumes
VOLUME /www/web1.com /www/web2.com

# JSON example
VOLUME ["myvolume", "myvolume2"]
```

### Uses of volumes

- de-couple the data that is stored from the container which created the data, example: app log file
- good for sharing data between containers, example: setup a data container which has a volume mounted in other containers

## Container Networking Basics

### Mapping ports

- containers have their own network and IP address
- map exposed container ports to ports on the host machine
- ports can be manually mapped (*-p* flag) or auto mapped (*-P* flag)

```shell
> docker run -d -p 8080:80 nginx:1.13
> docker ps
```

For automapping ports, host port numbers used go from 49153 to 65535 and only works for ports defined in the **EXPOSE** instruction in Dockerfile.

```dockerfile
FROM ubuntu:16.04
RUN apt-get update && apt-get install -y nginx

EXPOSE 80 443

CMD ["nginx", "-g", "daemon off;"]
```

### Linking containers

Linking is a communication method between containers which allows them to securely transfer data from one to another. Links are established based on container names. *Note: give your containers meaningful names!*

**Creating a link:**

- create the source container first
- create the recipient container and use the *--link* option

```shell
# create the source container using postgres
> docker run -d --name database postgres

# create the recipient container and link it
> docker run -d -P --name website --link database:db nginx    # db is an alias

# verify link in /etc/hosts
# 172.17.0.72    db
# or
# docker inspect database | grep IPAddress
# "IPAddress": "172.17.0.72",
```

**Uses of linking:**

- containers can talk to each other without having to expose ports to the host
- essential for micro service architecture
  - container with nginx running
  - container with mysql running
  - application on nginx needs to connect to mysql

## Sources

- https://training.docker.com/introduction-to-docker
- https://training.docker.com/docker-fundamentals
- https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/

------

*Next: Docker Operations.. stay tuned!*

