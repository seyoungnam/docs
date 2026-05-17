# 3. Container Networking Basics


## Introduction to Containers

In this section, we will discuss the evolution of running applications that has led us to containers.

### Applications

There is the historical and operational friction between developers and system admins when deploying and operating multiple applications in a single OS.

**The Core Conflict: Stability vs Agility**

- **Deployment Friction**: Fragmented workflows, library version conflicts, and custom Bash scripts make application deployments inconsistent and difficult for new developers to learn.
- **Organizational Tension**: SysAdmins prioritize **system stability** and high resource utilization, while developers prioritize **feature velocity**.

**The Networking & Infra Bottleneck**

- **The Single Stack Problem**: A general-purpose OS has only one TCP/IP stack. When multiple applications are hosted on one machine to increase efficiency, they inevitably clash over **port availability**.
- **Coordination Overhead**: Managing these port conflicts requires constant, manual coordination between developers, sysadmins, and network engineers.

**The Evolution of the Solution**

- **The Shared Kernel**: Because a general-purpose kernel tries to support everything (drivers, protocols, schedulers), it becomes a crowded environment for competing applications.
- **The Shift to Virtualization**: **Hypervisors** emerged as the primary solution to these problems, allowing a single physical host to run multiple independent operating systems and networking stacks, thereby eliminating port conflicts and improving overall machine utilization.

### Hypervisor

![Hypervisor](../../assets/img/kubernetes/networking/03-container-networking-basics/hypervisor.png)

A hypervisor allows system administrators to share the underlying hardware with multiple guest operating systems. This resource sharing **increases the host machine's efficiency**, alleviating one of the sysadmins issues. Hypervisors also gave each application development team a separate networking stack, **removing the port conflict issues**. 

**Library versions, deployment, and other issues remain for the application developer**. How can they package and deploy everything their application needs while maintaining the efficiency introduced by the hypervisor and virtual machines? This concern led to the development of containers.

### Containers

![Containers](../../assets/img/kubernetes/networking/03-container-networking-basics/containers.png)

each container is independent. 

- Application developers can use whatever they need to run their application **without relying on underlying libraries** or host operating systems. 
- Each container also has **its own network stack**.

#### Terms associated with container

- **Container**: A running container image
- **Image**: A container image is the file that is pulled down from a registry server and used locally as a mount point when starting a container.
- **Container engine**: A container engine accepts user requests via command-line options to pull images and run a container.
- **Container runtime**: The container runtime is the low-level piece of software in a container engine that deals with running a container.
- **Base image**: A starting point for container images; to reduce build image sizes and complexity, users can start with a base image and make incremental changes on top of it.
- **Image layer**: Repositories are often referred to as images or container images, but actually they are made up of one or more layers. Image layers in a repository are connected in a parent-child relationship. Each image layer represents changes between itself and the parent layer.
- **Image format**: Container engines have their own container image format, such as LXD, RKT, and Docker.
- **Registry**: A registry stores container images and allows for users to upload, download, and update container images.
- **Repository**: Repositories can be equivalent to a container image. The important distinction is
that repositories are made up of layers and metadata about the image; this is the manifest.
- **Tag**: A tag is a user-defined name for different versions of a container image.
- **Container host**: The container host is the system that runs the container with a container engine.
- **Container orchestration**: This is what Kubernetes does! It dynamically schedules container workloads for a cluster of container hosts.

#### A list of high and low functionality

Low-level container runtime functionality:

- Creating containers
- Running containers
- examples: *LXC*, *runC*

High-level container runtime functionality:

- Formatting container images
- Building container images
- Managing container images
- Managing instances of containers
- Sharing container images
- examples: *containerd*, *CRI-O*, *Docker*, *lmctfy*, *rkt*

#### OCI

**The Open Container Initiative (OCI) promotes common, minimal, open standards, and specifications for container technology**. The idea for creating a formal specification for container image formats and runtimes **allows a container to be portable across all major operating systems and platforms** to ensure no undue technical barriers.

#### Docker

Docker, released in 2013, solved many of the problems that developers had running containers end to end. It has all this functionality for developers to create, maintain, and deploy containers. 

![Docker engine](../../assets/img/kubernetes/networking/03-container-networking-basics/docker-engine.png)

Docker began as a monolith application, building all the previous functionality into a single binary known as the Docker engine. The engine contained the Docker client or CLI that allows developers to build, run, and push containers and images. In the past few years, Docker has broken apart this monolith into separate components. It allows the developers to focus on building their apps, and system admins focus on deployment.

#### CRI-O

CRI-O is an OCI-based implementation of the Kubernetes Container Runtime Interface(CRI). The CRI-O is a lightweight CRI runtime made as a Kubernetes-specific high-level runtime built on gRPC and Protobuf over a UNIX socket. The below diagram points out where the CRI fits into the whole picture with the Kubernetes architecture.

![CRI in Kubernetes](../../assets/img/kubernetes/networking/03-container-networking-basics/cri-o.png)

---

## Container Primitives

No matter if you are using Docker or containerd, runC starts and manages the actual containers for them. In this section, we will review what runC takes care of for developers from a container perspective. Each of our containers has Linux primitives known as *control groups* and *namespaces*.

![Namespaces and control groups](../../assets/img/kubernetes/networking/03-container-networking-basics/namespaces-cgroups.png)

cgroups control access to resources in the kernel for our containers, and namespaces are individual slices of resources to manage separately from the root namespaces, i.e., the host.

### Control Groups

In short, a cgroup is a Linux kernel feature that limits, accounts for, and **isolates resource usage**. cgroups allow administrators to control different CPU systems and memory for particulate processes. These separate subsystems maintain various cgroups in the kernel:

- CPU: The process can be guaranteed a minimum number of CPU shares.
- Memory: set up memory limits for a process.
- Disk I/O: This and other devices are controlled via the device's cgroup subsystem.
- Network: This is maintained by the `net_cls` and marks packets leaving the cgroup.

`lscgroup` is a command-line tool that lists all the cgroups currently in the system. runC will create the cgroups for the container at creation time. **A cgroup controls how much of a resource a container can use**, while **namespaces control what processes inside the container can see**.

### Namespaces

Namespaces are features of the Linux kernel that isolate and virtualize system resources of a collection of processes. Here are examples of virtualized resources:

- **PID namespace(pid)**: Processes ID, for process isolation
- **Network namespace(net)**: Manages network interfaces and a separate networking stack
- **IPC namespace(ipc)**: Manages access to interprocess communication (IPC) resources
- **Mount namespace(mnt)**: Manages filesystem mount points
- **UTS namespace(uts)**: UNIX time-sharing; allows single hosts to have different host and domain names for different processes
- **UID namespaces(user)**: User ID; isolates process ownership with separate user and group assignments

Below commands are an example of how to inspect the namespaces for a process:

``` bash
sudo ps -p 1 -o pid,pidns
[sudo: authenticate] Password:
    PID      PIDNS
      1 4026531836

sudo ls -l /proc/1/ns
total 0
lrwxrwxrwx 1 root root 0 May 16 16:44 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx 1 root root 0 May 16 16:44 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx 1 root root 0 May 10 17:33 mnt -> 'mnt:[4026531832]'
lrwxrwxrwx 1 root root 0 May 16 16:44 net -> 'net:[4026531833]'
lrwxrwxrwx 1 root root 0 May 10 17:33 pid -> 'pid:[4026531836]'
lrwxrwxrwx 1 root root 0 May 16 16:45 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx 1 root root 0 May 16 16:44 time -> 'time:[4026531834]'
lrwxrwxrwx 1 root root 0 May 16 16:45 time_for_children -> 'time:[4026531834]'
lrwxrwxrwx 1 root root 0 May 10 17:33 user -> 'user:[4026531837]'
lrwxrwxrwx 1 root root 0 May 16 16:44 uts -> 'uts:[4026531838]'
```

- All information for a process is on the `/proc` filesystem in Linux.
- **Direct check**: The first command (`ps -p 1 -o pid,pidns`) explicitly asked the kernel for the PID Namespace ID associated with process 1. The result was `4026531836`.
- **Cross-Reference**: The second command (`ls -l /proc/1/ns`) lists the **Namespace Inodes** that define the isolation boundaries for PID 1. A number in the squared bracket is called inode. The target of the `pid ` link matches the previous result exactly: `pid:[4026531836]`.

![Cgroups and namespace powers combined](../../assets/img/kubernetes/networking/03-container-networking-basics/cgroups-namespaces-combined.png)

The above illustrates how a container is constructed within a Linux node by using **namespaces** (IPC, NET, PID, UID) to provide an isolated environment and **cgroups** to manage and limit its resource consumption at the kernel level.


### Setting Up Namespaces

![Root network namespace and container network namespace](../../assets/img/kubernetes/networking/03-container-networking-basics/root-vs-container-network-namespace.png)

The above diagram outlines a basic container network setup. The following steps show how to create the networking setup shown in the above diagram:

1. Create a host with a root network namespace.
1. Create a new network namespace.
1. Create a veth pair.
1. Move one side of the veth pair into a new network namespace.
1. Address side of the veth pair inside the new network namespace.
1. Create a bridge interface.
1. Address the bridge interface.
1. Attach the bridge to the host interface.
1. Attach one side of the veth pair to the bridge interface.
1. Profit.

!!! Warning

    Please refer to [the "Installation Guide for the Linux users" section in the "1. Networking Introduction" page](01-networking-introduction.md/#application) to set up the `ubuntu/xenial64` Virtual Machine for our testing purpose.


Use Vagrant to ssh into this VM:
``` bash
vagrant ssh
Welcome to Ubuntu 16.04.7 LTS (GNU/Linux 4.4.0-210-generic x86_64)
...
New release '18.04.6 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


vagrant@ubuntu-xenial:~$
```

**IP forwarding** is an operating system's ability to accept incoming network packets on one interface, recognize them for another, and pass them on to that network accordingly. When enabled, IP forwarding allows a Linux machine to receive incoming packets and forward them. Let's enable it on our Ubuntu instance:
``` bash
$ sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 0
$ sudo sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"
$ sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1
```

With our install of the Ubuntu instance, we do not have any additional network namespaces:
``` bash
$ sudo ip netns list
```

`ip netns` allows us to control the namespaces on the server. Creating one is as easy as typing `ip netns add net1`:

``` bash
$ sudo ip netns add net1
# verify net1 namespace is created
$ sudo ip netns list
net1
```

Now that we have a new network namespace for our container, we will **need a veth pair for communication between the root network namespace and the container network namespace `net1`**. veth comes in pairs and acts as a conduit between network namespaces, so {==packets from one end are automatically forwarded to the other.==}

``` bash hl_lines="15-18"
# verify the current interfaces before the veth pair is created
$ ip link list
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 02:be:82:6b:cc:1d brd ff:ff:ff:ff:ff:ff
# create the veth pair
$ sudo ip link add veth0 type veth peer name veth1
# verify the veth pair creation
$ ip link list
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 02:be:82:6b:cc:1d brd ff:ff:ff:ff:ff:ff
3: veth1@veth0: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 06:22:dc:29:f5:ad brd ff:ff:ff:ff:ff:ff
4: veth0@veth1: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 1e:2e:17:f7:be:79 brd ff:ff:ff:ff:ff:ff
```

Interface 3 and 4 are the veth pairs in the command output. We can also see which are paired with each other, `veth1@veth0` and
`veth0@veth1`.

Now let's move `veth1` into the new network namespace created previously:
``` bash
# move veth1 to net1 network namespace
$ sudo ip link set veth1 netns net1
# verify veth1 is now in the net1 network namespace
$ sudo ip netns exec net1 ip link list
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
3: veth1@if4: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 06:22:dc:29:f5:ad brd ff:ff:ff:ff:ff:ff link-netnsid 0
```

The veth interface will need IP addressing in order to carry packets from the `net1` namespace to the root namespace
and beyond the host:
``` bash
sudo ip netns exec net1 ip addr add 192.168.1.100/24 dev veth1
```

As with host networking interfaces, they will need to be "turned on":
``` bash
sudo ip netns exec net1 ip link set dev veth1 up
```

Let's check the `veth1` interface status:
``` bash
$ sudo ip netns exec net1 ip link list veth1
3: veth1@if4: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state LOWERLAYERDOWN mode DEFAULT group default qlen 1000
    link/ether 06:22:dc:29:f5:ad brd ff:ff:ff:ff:ff:ff link-netnsid 0
```

- The state has now transitioned to `LOWERLAYERDOWN`.
- The status `NO-CARRIER` points in the right direction.
  - Ethernet needs a cable to be connected; our upstream veth pair is not on yet either.
  - The `veth1` interface is up and addressed but effectively still **unplugged**.

Let's turn up the `veth0` side of the pair now:
``` bash hl_lines="9-10"
# activate the veth0 interface so that it can begin sending and receiving traffic
$ sudo ip link set dev veth0 up
# verify
$ sudo ip link list
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 02:be:82:6b:cc:1d brd ff:ff:ff:ff:ff:ff
4: veth0@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 1e:2e:17:f7:be:79 brd ff:ff:ff:ff:ff:ff link-netnsid 0
```

Now the veth pair inside the `net1` namespace is UP:
``` bash hl_lines="4-5"
$ sudo ip netns exec net1 ip link list
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
3: veth1@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 06:22:dc:29:f5:ad brd ff:ff:ff:ff:ff:ff link-netnsid 0
```

Both sides of the veth pair report up; we need to connect the root namespace veth side to the bridge interface.

!!! Warning

Make sure to select the interface you're working with, in this case `enp0s3`; it may be different for others:

``` bash
# create and up the bridge
$ sudo ip link add br0 type bridge
$ sudo ip link set dev br0 up
# Add the interface to the bridge
$ sudo ip link set enp0s3 master br0

```


---

## Container Network Basics

### Docker Networking Model


### Overlay Networking


### Container Network Interface

---

## Container Connectivity

### Container to Container


### Container to Container Separate Hosts

---

## Conclusion