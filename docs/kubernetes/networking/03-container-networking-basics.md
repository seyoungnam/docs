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

- *Container*: A running container image
- *Image*: A container image is the file that is pulled down from a registry server and used locally as a mount point when starting a container.
- *Container engine*: A container engine accepts user requests via command-line options to pull images and run a container.
- *Container runtime*: The container runtime is the low-level piece of software in a container engine that deals with running a container.
- *Base image*: A starting point for container images; to reduce build image sizes and complexity, users can start with a base image and make incremental changes on top of it.
- *Image layer*: Repositories are often referred to as images or container images, but actually they are made up of one or more layers. Image layers in a repository are connected in a parent-child relationship. Each image layer represents changes between itself and the parent layer.
- *Image format*: Container engines have their own container image format, such as LXD, RKT, and Docker.
- *Registry*: A registry stores container images and allows for users to upload, download, and update container images.
- *Repository*: Repositories can be equivalent to a container image. The important distinction is
that repositories are made up of layers and metadata about the image; this is the manifest.
- *Tag*: A tag is a user-defined name for different versions of a container image.
- *Container host*: The container host is the system that runs the container with a container engine.
- *Container orchestration*: This is what Kubernetes does! It dynamically schedules container workloads for a cluster of container hosts.

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

In short, a cgroup is a Linux kernel feature that limits, accounts for, and **isolates resource usage**. cgroups allow administrators to
control different CPU systems and memory for particulate processes. These separate subsystems maintain various cgroups in the kernel:

- CPU: The process can be guaranteed a minimum number of CPU shares.
- Memory: set up memory limits for a process.
- Disk I/O: It is controlled via the device's cgroup subsystem.



### Namespaces


### Setting Up Namespaces

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