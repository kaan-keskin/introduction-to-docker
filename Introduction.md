# Introduction to Docker

Editors: **Taha Osman Sarıaslan, Semih Teker, Kaan Keskin**

Date: October 2021

Available at: https://github.com/kaan-keskin/introduction-to-docker

**Resources:**

> - Docker Deep Dive - Zero to Docker in a single book - Nigel Poulton @nigelpoulton
> - Containers Fundamentals (LFS253) - Linux Foundation
> - DevOps Lecture Notes - California Institute of Technology
> - Wikipedia - www.wikipedia.com

**LEGAL NOTICE: This document is created for educational purposes, and it can not be used for any commercial intentions. If you find this document useful in any means please support the original authors for ethical reasons.** 

**Please visit this page and buy kindle/digital version of the book:**
**https://www.amazon.com/Docker-Deep-Dive-Nigel-Poulton-ebook/dp/B01LXWQUFF/**

[Return README page.](README.md)

## Introduction

### The bad old days

Applications are at the heart of businesses. If applications break, businesses break. Sometimes they even go burst. These statements get truer every day! Most applications run on servers. **In the past we could only run one application per server.** The open-systems world of Windows and Linux just didn’t have the technologies to safely and securely run multiple applications on the same server. 

As a result, the story went something like this: Every time the business needed a new application, the IT department would buy a new server. Most of the time nobody knew the performance requirements of the new application, forcing the IT department to make guesses when choosing the model and size of the server to buy. As a result, IT did the only thing it could do — it bought big fast servers that cost a lot of money. After all, the last thing anyone wanted, including the business, was under-powered servers unable to execute transactions and potentially losing customers and revenue. So, IT bought big. This resulted in over-powered servers operating as low as 5-10% of their potential capacity. **A tragic waste of company capital and environmental resources!**

### Virtual Machines

Amid all of this, VMware, Inc. gave the world a gift — the virtual machine (VM). And almost overnight, the world changed into a much better place. We ﬁnally had a technology that allowed us to safely and securely run multiple business applications on a single server. This was a game changer. IT departments no longer needed to procure a brand-new oversized server every time the business needed a new application. More often than not, they could run new apps on existing servers that were sitting around with spare capacity. All of a sudden, we could squeeze massive amounts of value out of existing corporate assets.

As great as VMs are, they’re far from perfect! The fact that every VM requires its own dedicated operating system (OS) is a major ﬂaw. Every OS consumes CPU, RAM and other resources that could otherwise be used to power more applications. Every OS needs patching and monitoring. And in some cases, every OS requires a license. All of this results in wasted time and resources. The VM model has other challanges too. VMs are slow to boot, and portability isn’t great — migrating and moving VM workloads between hypervisors and cloud platforms is harder than it needs to be.

### Containers

For a long time, the big web-scale players, like Google, have been using container technologies to address the shortcomings of the VM model. In the container model, the container is roughly analogous to the VM. A major diﬀerence is that containers do not require their own full-blown OS. In fact, all containers on a single host share the host’s OS Kernel. This frees up huge amounts of system resources such as CPU, RAM, and storage. It also reduces potential licensing costs and reduces the overhead of OS patching and other maintenance. **Net result: savings on the time, resource, and capital fronts**.

Containers are also fast to start and ultra-portable. Moving container workloads from your laptop, to the cloud, and then to VMs or bare metal in your data center is a breeze.

### Linux containers

Modern containers started in the Linux world and are the product of an immense amount of work from a wide variety of people over a long period of time. Just as one example, **Google LLC** has contributed many container-related technologies to the Linux kernel. Without these, and other contributions, we wouldn’t have modern containers today.

Some of the major technologies that enabled the massive growth of containers in recent years include; **kernel namespaces**, **control groups**, **union ﬁlesystems**, and of course **Docker**. To re-emphasize what was said earlier — the modern container ecosystem is deeply indebted to the many individuals and organizations that laid the strong foundations that we currently build on. Thank you!

Despite all of this, containers remained complex and outside of the reach of most organizations. It wasn’t until Docker came along that containers were eﬀectively democratized and cessible to the masses.

Docker was the magic that made Linux containers usable for mere mortals. Put another way, Docker, Inc. made containers simple!

#### Control Groups (cgroups)

Control groups, known as cgroups, are a feature of the Linux kernel allowing the limitation, accounting, and isolation of resources used by groups of processes and their subgroups.

Cgroups are a fundamental feature of various Operating System level virtualization mechanisms, such as OpenVZ and LXC.

<img src=".\images\linux-cgroups.png" style="zoom:33%;"/>

Although cgroups may control single processes, they are used in the Operating System level virtualization predominantly to manage multiple processes together (hence the 'groups' in cgroups). 

**Cgroups allow the limitation of memory, disk I/O, and network usage for a group of processes.** In addition, cgroups may set usage quotas, and prioritize a process group to receive more CPU time or memory than other groups. Also, a group's resource usage can be measured for accounting and billing purposes, and its state can be controlled by freezing and restarting the group.

Recent implementations of the Linux kernel allow entire cgroups to be terminated as a whole unit, where all processes in a group together with its subgroups of children processes are terminated at once, thus easing the management of processes that are part of a particular workload. Also, in such cases of hierarchical groups, composed of a parent group and its children subgroups, the children subgroups inherit limits from their parent group.

Containers benefit from cgroups primarily because they allow system resources to be limited for processes grouped by a container. In addition, the processes of a container are treated and managed as a whole unit, and a container may be prioritized for preferential resource allocation.

#### Namespaces

Namespaces are a feature of the Linux kernel allowing groups of processes to have limited visibility of the host system resources. 

**Namespaces may limit the visibility of cgroups, hostname, process IDs, IPC mechanisms, network interfaces and routes, users, and mounted file systems.** To an isolated process running inside a namespace, a namespaced resource, such as the network, will appear as the process' own dedicated resource. Processes running inside a namespace are aware of any changes in the local namespaced system resources, however, such changes will not be visible to other processes or other namespaces.

In the context of containers, namespaces isolate processes from one container to prevent them from modifying the hostname, network interfaces, or mounts for processes running in other containers. The processes isolated inside a container can only see system resources namespaced for that particular container. Namespaces also help resolve PID conflicts, by allowing multiple processes running with the same PID to coexist on a host system, in separate namespaces, while at the global level of the host system they are each assigned different PIDs. As a result, multiple root processes are allowed to run on the same host system, isolated in separate namespaces, a situation that otherwise would not be possible since root is unique on a host system.

#### Unification File System (UnionFS)

**UnionFS is a feature found in the Linux, FreeBSD and NetBSD kernels, allowing the overlay of separate transparent file systems to produce an apparent single unified file system.**

When the content of several file systems, called branches, are virtually stacked, their contents appear to be merged, however, physically, they remain separate. The real physical separation, together with read-only and read-write access modes, help to prevent data corruption with the implementation of a **Copy-on-Write (CoW)** mechanism. This mechanism creates a physical copy of an existing file when it is being accessed to be changed. A copy of the file is made onto a real writable file system, part of the unionfs, and that copy is in fact being modified. However, virtually, through the unified file system, it appears as if the original file has been modified instead.

<img src=".\images\unionfs.png"/>

In a container, unionfs allows for changes to be made to the container image at runtime. The container image and other writable file systems are all stacked into a unionfs when the container is created and is running. The unified file system gives the impression that the actual container image is being modified. In reality, however, these changes are saved onto a real writable file system, part of the unionfs, while leaving the base container image file intact. In some environments, there is a possibility to export the new stacked files of the container into a new container image allowing users to create new and improved container images out of existing ones. All this while keeping image file sizes to a minimum as the new image file only stores the new changes with a link to the base image file.

**Windows containers**

Over the past few years, Microsoft Corp. has worked extremely hard to bring Docker and container technologies to the Windows platform. At the time of writing, Windows containers are available on the Windows desktop and Windows Server platforms (certain versions of Windows 10 and later, and Windows Server 2016 and later). In achieving this, Microsoft has worked closely with Docker, Inc. and the open-source community.

The core Windows kernel technologies required to implement containers are collectively referred to as *Windows Containers*. The user-space tooling to work with these *Windows Containers* can be Docker. This makes the Docker experience on Windows almost exactly the same as Docker on Linux. This way developers and sysadmins familiar with the Docker toolset from the Linux platform can feel at home using Windows containers. 

**Windows containers vs Linux containers**

It’s vital to understand that a running container shares the kernel of the host machine it is running on. This means that a containerized Windows app will not run on a Linux-based Docker host, and vice-versa — Windows containers require a Windows host, and Linux containers require a Linux host. 

It is possible to run Linux containers on Windows machines. For example, Docker Desktop running on Windows has two modes — “Windows containers” and “Linux containers”. Depending on your version of Docker Desktop, Linux container run either inside a lightweight Hyper-V VM or using the Windows Subsystem for Linux (WSL). The WSL option is newer and the strategic option for the future as it doesn’t require a Hyper-V VM and oﬀers better performance and compatibility.

**What about Mac containers?**

There is currently no such thing as Mac containers.

However, you can run Linux containers on your Mac using *Docker Desktop*. This works by seamlessly running your containers inside of a lightweight Linux VM on your Mac. It’s extremely popular with developers, who can easily develop and test Linux containers on their Mac.

**What about Kubernetes**

Kubernetes is an open-source project out of Google that has quickly emerged as the de facto orchestrator of containerized apps. That's just a fancy way of saying *Kubernetes is the most popular tool for deploying and managing containerized apps*.

    Note: 
    A containerized app is an application running as a container. At the time of writing, Kubernetes uses Docker as its default container runtime — the low-level technology that pulls images and starts and stops containers. However, Kubernetes has a pluggable container runtime interface (CRI) that makes it easy to swap-out Docker for a diﬀerent container runtime. In the future, Docker might be replaced by containerd as the default container runtime in Kubernetes. For now it’s enough to know that containerd is the small specialized part of Docker that does the low-level tasks of starting and stopping containers.

The important thing to know about Kubernetes, at this stage, is that it’s a higher-level platform than Docker, and it currently uses Docker for its low-level container-related operations.

### Full Virtualization vs. Operating System-Level Virtualization

**Although Containers are not considered to be Virtual Machines (VMs), not even light-weight VMs, their similarities cannot be overlooked.** 

Both VMs and Containers are products of virtualization methods and provide resources isolation for running applications, but the main differences surround the underlying technologies and management methods.

<img src=".\images\full-virtualization-vs-os-level-virtualizaiton.png"/>

#### Virtual Machines vs. Containers

**A Virtual Machine is created on top of a hypervisor software, which may be installed over a host operating system (OS) or directly on a bare-metal system without a host OS.** 

**The hypervisor emulates hardware such as CPU, memory, storage disk, and networking from the available physical resources of the host system.** Then, a guest operating system such as Linux or Windows is installed on top of the emulated hardware. 

A typical application runs inside such a VM, and it requires extensive overhead to reach the physical hardware or the outside world considering that it has to go through so many layers of abstraction - the guest OS, then the hypervisor, and finally the host OS. 

A hypervisor may support multiple VMs running in parallel, each with its different guest OS, to the extent of the physical resources available on the host system to support all the running guests’ resources needs, together with the needs of the host operating system and of the hypervisor.

**By contrast, a container is a light-weight environment that virtualizes and isolates resources for a running application - typically a microservice.** 

A container image, which is the source or template of the container, allows an application to be boxed and shipped with all its dependencies. Once deployed, a container runs directly on the host operating system, without the need of a guest OS and a hypervisor, thus dramatically reducing its operations’ overhead. Without a hypervisor, it is the host operating system’s role, via its virtualization capabilities, to provide the isolation and resource allocation to individual containers. As a result, the user space component of the container should be compatible with the host operating system.

#### Operating System-Level Virtualization

**Operating system-level virtualization refers to a kernel’s capability to allow the creation and existence of multiple isolated virtual environments on the same host.** Such environments, known as containers, zones, partitions, virtual kernels, or jails, encapsulate running programs and conceal from them the true nature of their virtual environment, creating an illusion of a real computing environment. As opposed to programs running inside a real environment, where they see all resources such as CPU, network, connected devices, and files, programs running inside a virtual environment are limited to its content and assigned devices.

A real Operating System may allow or deny access to resources, or it may hide resources from programs. An Operating System level virtualized environment may have allocated only a set of the available resources, thus limiting programs’ access to them.
On a real OS, one or many isolated virtual environments may be created, with each running one or multiple programs. These programs may run concurrently, separately and may even interact with each other.

Operating System level virtualization is typically used to limit usage and securely isolate resources shared between multiple programs or users, and to separate programs to run in their own assigned virtual environments for better security and resource management.

**While Operating System level virtualization requires less overhead than full virtualization because everything is managed at the kernel level without the need to install a guest OS, it limits the OS of the virtual environments to the host Operating System.** Also, the OS-level virtualization introduces a stacked storage management model, implemented by the file-level Copy-on-Write (CoW) mechanism. This CoW mechanism ensures minimal storage space usage when saving files that are being modified. Changes and new data are stored on disk, but data duplication is prevented by making heavy usage of links for referencing original data that remains unchanged.

#### Mechanisms Implementing Operating System-Level Virtualization

While one of the first known mechanisms to implement operating system-level virtualization dates from the early 1980s, the majority of such mechanisms known today were released after the turn of the 21st century. As opposed to the newer virtualization mechanisms, their predecessor only implemented a limited amount of virtualization features for UNIX-like systems. This reveals the level of interest, or lack thereof, the OS-level virtualization received until about two decades ago, when it experienced a major comeback . Both open source software and commercial software mechanisms with heavily expanded virtualization features were released for Linux, Windows, and FreeBSD systems.

Next, let’s explore some of the virtualization mechanisms that paved the way for today’s containers, presented in chronological order, and not in order of importance.

### Docker

Docker is software that runs on Linux and Windows. It creates, manages, and can even orchestrate containers. The software is currently built from various tools from the **Moby** open-source project. Docker, Inc. is the company that created the technology and continues to create technologies and solutions that make it easier to get the code on your laptop running in the cloud.

<img src=".\images\feature-of-docker.png" style="width:75%; height: 75%;"/>

- Docker is a platform for developers and sysadmins to develop, ship, and run applications by using containers.
- Docker helps the user to quickly assemble applications from its components and eliminates the friction during code shipping.
- Docker aids the user to test and deploy the code into production.

#### Docker Functionalities

<img src=".\images\docker-functionalities.png" style="width:75%; height: 75%;">

#### Docker Properties

<img src=".\images\docker-properties.png" style="width:75%; height: 75%;">

### The Docker Technology

When most people talk about Docker, they’re referring to the technology that runs containers. However, there are at least three things to be aware of when referring to Docker as a technology:

1. The runtime
2. The daemon (a.k.a. engine)
3. The orchestrator

<img src=".\images\TheDocker.png" />

The runtime operates at the lowest level and is responsible for starting and stopping containers (this includes building all of the OS constructs such as namespaces and cgroups). Docker implements a tiered runtime architecture with high-level and low-level runtimes that work together. The low-level runtime is called runc and is the reference implementation of Open Containers Initiative (OCI) runtime-spec. Its job is to interface with the underlying OS and start and stop containers. Every running container on a Docker node has a runc instance managing it. The higher-level runtime is called containerd. containerd does a lot more than runc. It manages the entire lifecycle of a container, including pulling images, creating network interfaces, and managing lower-level runc instances. containerd is pronounced “container-dee’ and is a graduated CNCF project used by Docker and Kubernetes as a container runtime.

A typical Docker installation has a single containerd process (docker-containerd) controlling the runc (docker-runc) instances associated with each running container. The Docker daemon (dockerd) sits above containerd and performs higher-level tasks such as; exposing the Docker remote API, managing images, managing volumes, managing networks, and more.

A major job of the Docker daemon is to provide an easy-to-use standard interface that abstracts the lower levels. Docker also has native support for managing clusters of nodes running Docker. These clusters are called swarms and the native technology is called Docker Swarm. Docker Swarm is easy-to-use and many companies are using it in real-world production. However, most people are choosing to use Kubernetes instead of Docker Swarm.

### The Open Container Initiative (OCI)

The Open Container Initiative (OCI) was introduced in 2015 by Docker together with other leaders in the container industry. One of the container runtimes implementing the OCI specification is runC.

The OCI is a governance council responsible for standardizing the low-level fundamental components of container infrastructure. In particular it focusses on **image format** and **container runtime**. It’s also true that no discussion of the OCI is complete without mentioning a bit of history. And as with all accounts of history, the version you get depends on who’s doing the talking.

From day one, use of Docker grew like crazy. More and more people used it in more and more ways for more and more things. So, it was inevitable that some parties would get frustrated. This is normal and healthy.

This put the container ecosystem in an awkward position with two competing standards. Getting back to the story, this threatened to fracture the ecosystem and present users and customers with a dilemma. While competition is usually a good thing, **competing standards** is usually not. They cause confusion and slowdown user adoption. Not good for anybody. With this in mind, everybody did their best to act like adults and came together to form the OCI — a lightweight agile council to govern container standards.

At the time of writing, the OCI has published three specifications (standards):
- The Runtime Specification
- The Image Specification
- The Distribution Specification

The OCI incorporates the Runtime Specification (runtime-spec), the Image Specification (image-spec), and the most recent Distribution Specification (distribution-spec).

**The Runtime Specification** defines how to run a "filesystem bundle" that is unpacked on disk. An OCI implementation would download and unpack an OCI image into an OCI Runtime filesystem bundle. Then, an OCI Runtime would run the OCI Runtime Bundle.

**The Image Specification** helps with the development of compatible tools to ensure consistent container image conversion into containers.

**The Distribution Specification** standardizes how container images are distributed through image registries.

An analogy that’s often used when referring to these two standards is rail tracks. these two standards are like agreeing on standard sizes and properties of rail tracks, leaving everyone else free to build better trains, better carriages, better signalling systems, better stations. All safe in the knowledge that they’ll work on the standardized tracks. Nobody wants two competing standards for rail track sizes! It’s fair to say that the two OCI speciﬁcations have had a major impact on the architecture and design of the core Docker product. As of Docker 1.11, the Docker Engine architecture conforms to the OCI runtime spec. The OCI is organized under the Linux Foundation.

### Architecture, Description and Main Features of Container Runtimes

**A container runtime is guided by a runtime specification, which describes the configuration, execution environment and the lifecycle of the container.** The role of a container runtime is to provide an environment supporting basic operations with images and the running containers, that is both configurable and consistent, where container processes are able to run. 

**A runtime’s consistency is one of the top benefits for running containers. Regardless of the underlying infrastructure - whether an on-prem Data Center or a Cloud Infrastructure as a Service (IaaS), containers’ behavior is expected to be the same, thus allowing users to develop and test containers on any system across all tiers - from development to production.**

A container runtime is designed to perform several default operations under the hood as a response to user commands. The container runtime extracts the container image content and stores it on an overlay filesystem, that utilizes the Copy-on-Write mechanism for virtual file integrity. When the runtime executes a container, it interacts with the kernel to set resource limits, build isolation layers through virtualization mechanisms like control groups and namespaces in order to run a containerized application as specified by the container image.
