# Use case for containers
- to install an application, we need a (virtual) machine, install an OS, install the application dependencies (runtime engines like apache/tomcat/java/php, libs, static assets like figures and sound files, etc.), get and compile code from developer or get binary package, set up networking, security, etc.
  - the steps to install and configure an application on the OS may be long, technically demanding, complicated, your may encounter errors/conflicts with installed apps, etc.
- by using containers, the burden of installation is handled by the containerization technology
	- end users just need to get the **application's image** and *run* it.
	- developer builds those images for their applications and makes them available to users via some channel (like hub.docker.com, a public marketplace provided by Docker company.)
	- the application's image is an archive file containing the application code, dependencies, and configuration instructions. the application image contains every thing needed for the app to run: the run time engines, libraries, the app code, system and user files, configuration, etc.
	- users get the image then runs the container the image (thus creating the **container**) on a containers driver
- there are multiple technologies for containers: docker, lxc, podman, proxmox, etc.
  - they all share the same base linux kernel technologies: namespaces, cgroups, union file systems
  - containers originated in Linux, later ported to Windows by Docker.
- advantages of running applications in containers: portability, ease of deployment, etc. while enabling isolation which is the main features of virtualization and containerization.
- how do they compare to virtual machines? shares hw and kernel -> efficient, *must be compatible with the ABI/ISA* (since processes in a containers uses the host kernel and hardware) 

# Linux Namespaces
- **Linux Namespaces** is a linux kernel feature to make processes run in isolated "spaces", meaning that they only see and use a subset of system ressources (ipc, networking, users, mounts, etc.) that an administrator places in one "space". There are different type of resources, so different types of namespaces; a two processes can share a networking space but have a distinct user namespace for each one.
- a process is set to view only one namespace of each type, it cannot access resources in other namespaces or get its own view of the system resources
- when a child process is created with `int clone(...)` (the same as `int fork()`), it can be assigned dedicated *namespaces* of any type.
- For example, the **network namespace** provides an isolated view of the networking stack (network device interfaces, IPv4 and  IPv6 protocol stacks, IP routing tables, firewall rules, the `/proc/net` and `/sys/class/net` directory trees, sockets, netfilter rules, etc.).  A physical network device can live in exactly one network  namespace. If communication between namespaces is desired, a virtual network device pair provides a pipe-like abstraction that can be used to create tunnels between network namespaces, and can be used to create a bridge to a physical network device in another namespace.
- Linux provides the following namespaces:
    - IPC: System V IPC, POSIX message queues
    - Network: Network devices, stacks, ports, etc.
    - Mount: Mount points
    - PID: Process IDs
    - User: User and group IDs
    - UTS: Hostname and NIS domain name
    - Cgroup: Cgroup root directory
- The main goal of namespaces development was to implement containers
- relates commands:
  - `lsns` to list name spaces in linux
  - `unshare` to start a process with a separate namespace (just like clone would do within a program)
  - `nsenter`

```c
pid = clone(childFunc, stackTop, CLONE_NEWUTS | SIGCHLD, argv[1]);
if (pid == -1)
    errExit("clone");
    printf("clone() returned %ld\n", (long) pid);
```

```bash
me@mypc:~$ sudo lsns --type pid
        NS TYPE NPROCS   PID USER   COMMAND
4026531836 pid     306     1 root   /sbin/init splash
4026532629 pid       2  3152 brahim /usr/share/skypeforlinux/skypeforlinux --type=zygote --electron-shared-settings=eyJjci5jb...NyL
```

```bash
me@mypc:~$ sudo unshare --uts=~/somefile bash
me@mypc:~# hostname guestname
```

# Linux control groups (cgroups)
- defines limits for the system ressources (mem, cpus, i/o, network) for given processes


# Union file systems (storages drivers)
- a type of file system where a file system is obtained by unioning multiple file systems (also called layers).
- technique used with **copy-on-write** to allow snapshotting, branching/sharing. also used with live CDs.
  - when snapshot is needed, a new layer is created where write actions are recorded in it.
- it works as follows:
  - when a file from an underlying layer is modified, it is copied to an upper layer where it is modified,
  - newer ones are kept private to the upper layer,
- idea currently implemented in multiple linux/windows file systems (Btrfs, ZFS, ReFS) or database systems (SQL Server), and Docker is using OverlayFS. The unioning idea was also applied to other hierarchical structures where storage is block or object oriented (not just files)

# Docker architecture
- **container** is a set of processes (hierarchy) running on a host operating system. runs *isolated* from othe processes, meaning that it has its own network config, user sets, file systems (besides other aspects). starting, stopping, controlling the container is done by the **containers engine** (the most common one is docker engine, by Docker).
- **container's image** refers to the content of the container's filesystem (think of it as a filesystem of a stopped OS). It also contain a manifest file describing configuration that applies to the container once started. the filesystem may itself be defined by a set of layers.
- **docker engine**: a software which manages containers, images, networks, and other configuration aspects of containers, etc.
	- reachable through a RESTful API (transport is either Unix socket `/var/run/docker.sock` or IP sockets)
 - **docker client** (in CLI mode) allows you to interact with a local/remote docker engine
 - **containers registries** or docker hub are the exchange hub for images (some are freely distributed, other requires authentication). development companies may hold their own containers registry (commercial products are JFrog and Sonatype Nexus).

# Running a container
- running a container is acheived through docker client as in ```docker run -d -p 80:80 docker/getting-started:latest```. the command 
	- fetches (i.e. pulls) an image named `getting-started` (a web application) from the public Docker Corporation registry (https://hub.docker.com) and having the tag `latest`.    
	- starts the processes making up the container (entrypoint, cmd, etc.), putting them into isolated namespaces as defined by the image configuration
	- and applies configurations from the image (check docker image inspect ...) and from the command line (-p 80:80), networking and mount configuration are typically applied 
- options -p 80:80 maps port 80 on the container host to the port 80 in the container, option `-d` makes the processes of the container run in the background (do not use the standard output)
- when container is running, any data created by the processes remains files in a distinct layer
- when the container is destroyed (stopped and removed), the modifications layer is removed; newer conainers can be created from the image
- typically, data that must be persisted across reboots/migrations/containers in cluster are stored outside of the docker (e.g. NFS, local mount). Check the example of a database container.
- containers can be run interactively, have their stdin/stdout/stderr connected to the console (run in foreground), or run in background (with the -d), defaut is the foreground.
- docker engine stores containers log (stdout and stderr)
- docker engine tracks history of the local images
- some containers expose some ports and needs you to set environment variables on which processes rely.
- it is possible to inspect containers and images to check their configuration. Use `--format` option with adequate template in order to filter the inspect command output.

# Building docker image
- the typical docker workflow: developers creates images containing their application for a specific os/architecture, publishes it on docker hub. users downloads images, start containers and uses them.
- developer *tells* how to create the image in a text file called `Dockerfile` (can also be done interactively)
```dockerfile
FROM ubuntu
RUN apt-get update
RUN apt-get install figlet
```
- then asks docker engine to build image by the following those steps:
.small[
```bash
$ docker build -t figlet .
Sending build context to Docker daemon  2.048kB
Step 1/3 : FROM ubuntu
 ---> f975c5035748
Step 2/3 : RUN apt-get update
 ---> Running in e01b294dbffd
(...output of the RUN command...)
Removing intermediate container e01b294dbffd
 ---> eb8d9b561b37
Step 3/3 : RUN apt-get install figlet
 ---> Running in c29230d70f9b
(...output of the RUN command...)
Removing intermediate container c29230d70f9b
 ---> 0dfd7a253f21
Successfully built 0dfd7a253f21
Successfully tagged figlet:latest
```
]
- preliminary step: sending the *build context* (as an archive) to the docker engine.
```bash
Sending build context to Docker daemon 2.048 kB
```
- build step 2/3:
	- a container (`e01b294dbffd`) is created from the base image.
	- the `RUN` command is executed in this container.
	- the changes to the container filesystem are committed into an image (`eb8d9b561b37`).
	- the build container (`e01b294dbffd`) is stopped then removed.
	- The output of this step will be the base image for the next one.

```bash
Step 2/3 : RUN apt-get update
 ---> Running in e01b294dbffd
(...output of the RUN command...)
Removing intermediate container e01b294dbffd
 ---> eb8d9b561b37
```
- here is a commented and modified example of dockerfile from the official getting started of docker.com.
  ```dockerfile
  FROM node:12-alpine
  RUN apk add --no-cache python g++ make
  WORKDIR /app
  COPY . .
  RUN yarn install --production
  ENV DIRPATH=/app
  EXPOSE 8080/tcp
  CMD ["node", "src/index.js"]
  ```

# Container networking model
- each container has its own network namespace, i.e. its *own* networking stack (tcp/ip stack, firewall rules, hostname, etc.) including one or more virtual devices, and packets are being processed in different ways that in other containers from the same docker host.
- the linux kernel implements virtual bridges (segments) to interconnect containers
- docker engine can organize containers in networks, by creating isloated *networks* within the host and connecting containers's virtual devices to them (using linux bridges).
 - each network has its own IP subnet
 - a network can be local (to a single docker engine) or global (span multiple hosts)
 - networks can be connected
- the containers' names (aka *network aliases*) are used in DNS-based service discovery

# Network drivers
- each network is implemented by a *driver*
- there are some built-in drivers, 
  - bridge (default): network is just an ethernet switch managed by docker (the default bridge, unless a new one is created)
  - none: networking is just disabled
  - host: the container uses networking stack of the host (does not live in an isolated networking namespace)
  - macvlan:
  - ipvlan: 
  - overlay: used in Swarm clusters in order to span multiple docker hosts, it leverages the VXLAN virtualization technique
 - other drivers can be provided by plugins to the docker engine (Weave, Calico, OVS,)
- each network may have a custom IP address management (IPAM) toolset (depending on the plugin): dhcp, nat,

# Service discovery
- containers reach each other through names (instead of IP addresses)
- names resolution to IP addresses is done by the docker-provided DNS service (since Docker Engine 1.10)
- the docker engine automatically registers containers
	- container (full) name : container-name.network-name
- each network maps to a domain name

# Example on using networks
- In the following, we create a network, attach two containers and test connectivity using names.
```bash
$ docker network create dev
4c1ff84d6d3f1733d3e233ee039cac276f425a9d5228a4355d54878293a889ba
$ docker network ls
NETWORK ID          NAME                DRIVER
6bde79dfcf70        bridge              bridge
8d9c78725538        none                null
eb0eeab782f4        host                host
4c1ff84d6d3f        dev                 bridge
$ docker run -d --name es --net dev elasticsearch:2
8abb80e229ce8926c7223beb69699f5f34d6f1d438bfc5682db893e798046863
$ docker run -ti --net dev alpine sh
root@0ecccdfa45ef:/# ping es
PING es (172.18.0.2) 56(84) bytes of data.
64 bytes from es.dev (172.18.0.2): icmp_seq=1 ttl=64 time=0.221 ms
^C
--- es ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 2000ms
rtt min/avg/max/mdev = 0.114/0.149/0.221/0.052 ms
```

# Storage Model
- images are immutable -> container creates a new layer instead for modifications, which is lost once the container destroyed, unless persisted elsewhere
- Example:
```bash
$ docker run -d -p 1234:8080 \
         -v logs:/usr/local/tomcat/logs \
         -v webapps:/usr/local/tomcat/webapps \
         tomcat
```
- multiple containers sharing a single store
- container and host may need to share data
- are there situations where data cannot persist across container reboots?
- etc.

# Mount types
- the linux kernel mounts different filesystems backed by three types of storage: actual directories from the docker host path, raw file, temporary file in sytem memory
- docker engine assigns mount points to the created containers' process
- *bind mounts*: for existing host path to a container path
	- mount the binary artefact path within container,
	- mount container path locally to allow live editing containers
- *volume mounts*: ask docker engine-managed storage volume
	- useful for container generated data like database files
	- docker engine implements creation, deletion, snapshotting, migration, persistence, encryption, etc.
	- volumes persists beyond containers, are shared across containers
- *tmpfs mounts*: docker creates temporary fs in memory
- `--mount 'type=volume,src=<volume-name>,dst=<container-path>,volume-driver=<driver>`
- `-v [host-path]:[container-path]:[rw|ro]`
- `--mount type=bind,source=/path/on/host,target=/path/in/container`

# Volume drivers
- available volume drivers: local, AWS-S3, NFS
- volume attributes: rw/ro, etc.

# Dockerized development workflows
- docker changes the way developers code, test, delivers, deploy
- edit code develop within a container
  - live editing the application code
  - 
- use compilers from docker containers

# Docker compose: Running a multi containers applications
- **`docker-compose`** is a tool to handle *multiple* containers configuration from a single recipe (`docker-compose.yml`)
- the recipe describes the required *services*, and tells for each one the image (also built on the fly), and additional configuration: networking, volumes, env vars, etc.
- compose works with a single docker host 
```yaml
version: "3"

services:
  www:
    build: www
    ports:
      - ${PORT-8000}:5000
    user: nobody
    environment:
      DEBUG: 1
    command: python counter.py
    volumes:
      - ./www:/src

  redis:
    image: redis
```

# Docker compose file structure
- A compose file has multiple sections:
	- `version` (current is "3".)
	- `services` each service corresponds to a container.
	- `networks`, indicates to which networks containers should be connected. (By default, containers will be connected on a private, per-compose-file network.)
	- `volumes`, defines volumes to be used and/or shared by the containers. (optional)

# Handling more requirements: containers orchestration
- what if docker engine host fails?
- what if applications needs more computing resources to handle load? 'elastic' requirements, load balancing
- scheduling jobs
- also, automates deployment patterns implementation on the cloud.
- service mesh
- serverless trend
- different tools (**k8s**, Docker Swarm, Apache Mesos, ), different features.
