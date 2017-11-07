# Docker Operations

Docker orchestration, continuous integration and operation tools.

- Docker Compose
- Docker CI Pipeline
- Docker Swarm

## Docker Compose

Docker compose is a tool for creating and managing multi container applications. Consider below scenario:

- different container runs a particular component or service of the application and these containers are linked together. For example:
  - web application
  - authentication
  - payments
  - database
- containers are all defined in a single file called **docker-compose.yml**
- running compose will spin up all containers in a single command

As the number of containers grows, it is not very practical to start up each container separately and link them manually. That is why compose becomes very useful.

### Installing compose

Please refer to this [documentation](https://docs.docker.com/compose/install/) for the latest instruction.

### Configuring compose yaml file

- defines all services that make up the application
- each service contains instructions for building and running a container
- all services must have either a **build** or **image** instruction
- **links** instruction is not required anymore for compose version 2 or above
- more instructions are available at https://docs.docker.com/compose/overview/

**Example docker-compose.yml:**

```yaml
version: '3'
services:
    dockerapp:
        build: .
        ports:
            - "5000:5000"
        depends_on:
            - redis
    redis:
        image: redis:3.2.0
```

### Managing compose-managed application

**Start up all the containers**

```shell
> docker-compose up
> docker-compose up -d    # detached mode
```

**Check status of the containers**

```shell
> docker-compose ps
```

**Output logs for the containers**

```shell
> docker-compose logs
> docker-compose logs -f                  # appended log
> docker-compose logs [container name]    # for specific container
```

**Stop all the running containers without removing them**

```shell
> docker-compose stop
```

**Remove all the containers**

```shell
> docker-compose rm
```

**Rebuild any images created from the Dockerfile**

```shell
> docker-compose build
```

## Docker CI Pipeline

![Docker P](sources/docker-ci-pipeline.png "Docker CI Pipeline")

**Travis CI docker integration example:**

- https://docs.travis-ci.com/user/docker/
- https://docs.travis-ci.com/user/build-stages/share-docker-image/

## Docker Swarm

Docker swarm is a tool that clusters docker hosts and schedules containers. It turns a pool of host machines into a single virtual host. Docker swarm also ships with simple scheduling backend.

### Installing swarm

Swarm image is available on [Docker Hub](https://hub.docker.com/r/library/swarm/). Swarm container is a convenient packaging mechanism for the swarm binary and can be run from the image to do the following:

- create a cluster
- start the swarm manager
- join nodes to the cluster
- list nodes on a cluster

### Setup process using hosted discovery

- run a command to create the cluster on the machine that will be used as the swarm master
  - **swarm create** command will output the cluster token
  - token is an alphanumeric sequence of characters that identifies the cluster when using the hosted discovery protocol
  ```shell
  > docker run --rm swarm create
  ```
- start swarm master
  - **swarm manage** command will start swarm master
  ```shell
  > docker run -dP swarm manage token://<cluster token>
  ```
- run a command to start the swarm agent for each node with docker installed (agents can be started before or after the master)
  - **swarm join** command will connect a node to the cluster
  - specify the IP address of the node and the port the docker daemon is listening on
  ```shell
  # docker on the machine must be configured to listen on a TCP port instead of unix socket
  > docker run -d swarm join --addr=<NODE IP>:<DAEMON PORT> token://<cluster token>
  ```

### Connect docker client to swarm

There are two methods:

- configuring the *DOCKER_HOST* variable with the swarm IP and port
- run docker with *-H* and specify the swarm IP and port
```shell
> export DOCKER_HOST=127.0.0.1:<swarm port>
> docker -H tcp://127.0.0.1:<swarm port>
```

Run **docker version** to verify docker client is connected to swarm and run **docker info** to check all connected nodes.

### Run a container in the cluster

- use standard **docker run** command
- use **docker ps** command to show which node a container is on
- more swarm scheduler strategy are available at https://docs.docker.com/swarm/scheduler/strategy/

## Sources

- https://training.docker.com/docker-operations
- https://docs.docker.com/compose/compose-file/
- https://docs.docker.com/engine/swarm/secrets/
- https://docs.docker.com/datacenter/ucp/2.2/guides/
- https://docs.docker.com/datacenter/dtr/2.4/guides/
- https://docs.docker.com/engine/security/trust/content_trust/

------

