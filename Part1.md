# Introduction to Docker - Part 1

Editors: **Taha Osman Sarıaslan, Semih Teker, Kaan Keskin**

Date: September 2021

Available at: https://github.com/kaan-keskin/introduction-to-docker

**Resources:**

> - Docker Deep Dive - Zero to Docker in a single book - Nigel Poulton @nigelpoulton
> - Containers Fundamentals (LFS253) - Linux Foundation
> - DevOps Lecture Notes - California Institute of Technology
> - Wikipedia - www.wikipedia.com

**LEGAL NOTICE: This document is created for educational purposes, and it can not be used for any commercial purposes. If you find this document useful in any means please support original authors for ethical reasons.** 

**Please visit this page and buy kindle/digital version of the book:**
**https://www.amazon.com/Docker-Deep-Dive-Nigel-Poulton-ebook/dp/B01LXWQUFF/**

**Content**

> - Introduction
> - The Big Picture
> - The Docker Engine
> - Images
> - Containers

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

## The Big Picture

The aim of this chapter is to paint a quick big-picture of what Docker is all about before we dive in deeper in later chapters. We’ll break this chapter into two:

• The Ops perspective

• The Dev perspective

In the Ops Perspective section, we’ll download an image, start a new container, log in to the new container, run a command inside of it, and then destroy it.

In the Dev Perspective section, we’ll focus more on the app. We’ll clone some app-code from GitHub, inspect a Dockerﬁle, containerize the app, run it as a container.

If you want to follow along, all you need is a single Docker host with an internet connection. I recommend Docker Desktop for your Mac or Windows PC. However, the examples will work anywhere that you have Docker installed.

If you can’t install software and don’t have access to a public cloud, another great way to get Docker is Play With Docker (PWD). This is a web-based Docker playground that you can use for free. Just point your web browser to https://labs.play-with-Docker.com/ and you’re ready to go (you’ll need a Docker Hub or GitHub account to be able to login).

As we progress through the chapter, we may use the terms “Docker host” and “Docker node” interchangeably. Both refer to the system that you are running Docker on.

### The Ops Perspective

When you install Docker, you get two major components:

- the Docker client
- the Docker daemon (sometimes called the “Docker engine”)

The daemon implements the runtime, API and everything else required to run Docker.

In a default Linux installation, the client talks to the daemon via a local IPC/Unix socket at **`/var/run/docker.sock`**.

On Windows this happens via a named pipe at `npipe:////./pipe/docker\_engine`. Once installed, you can use
the `docker version` command to test that the client and daemon (server) are running and talking to each other.

```shell
$ docker version
```

If you get a response back from the `Client` and `Server`, you’re good to go.

If you are using Linux and get an error response from the Server component, make sure that Docker is up and running. Also, try the command again with sudo in front of it: **sudo docker version**. If it works with sudo you will need to add your user account to the local docker group, or preﬁx the remainder of the commands in the book with sudo.

#### Images

It’s useful to think of a Docker image as an object that contains an OS ﬁlesystem, an application, and all application dependencies. If you work in operations, it’s like a virtual machine template. A virtual machine template is essentially a stopped virtual machine. **In the Docker world, an image is eﬀectively a stopped container**. If you’re a developer, you can think of an image as a *class*.

Run the **`docker image ls`** command on your Docker host.

```shell
$ docker image ls
```

If you are working from a freshly installed Docker host, or Play With Docker, you will have no images and it will look like the previous output. Getting images onto your Docker host is called “pulling”. If you are following along with Linux, pull the `ubuntu:latest` image.

```shell
$ docker image pull ubuntu:latest

$ docker images
```

We’ll get into the details of where the image is stored and what’s inside of it in later chapters. For now, it’s enough to know that an image contains enough of an operating system (OS), as well as all the code and dependencies to run whatever application it’s designed for. The ubuntu image that we’ve pulled has a stripped-down version of the Ubuntu Linux ﬁlesystem, including a few of the common Ubuntu utilities. The mcr.microsoft.com/powershell:lts-nanoserver-1903 image contains a Windows Server Core OS plus PowerShell.

If you pull an application container such as nginx or mcr.microsoft.com/windows/servercore/iis, you will get an image that contains some OS, as well as the code to run either NGINX or IIS. It’s also worth noting that each image gets its own unique ID. When referencing images, you can refer to them using either IDs or names. If you’re working with image ID’s, it’s usually enough to type the ﬁrst few characters of the ID — as long as it’s unique, Docker will know which image you mean.

#### Containers

Now that we have an image pulled locally, we can use the `docker container run` command to launch a container from it.

For Linux:

```shell
$ docker container run -it ubuntu:latest /bin/bash
root@6dc20d508db0:/#
```

For Windows:

```powershell
\> docker container run -it mcr.microsoft.com/powershell:lts-nanoserver-1903 pwsh.exe 

PowerShell 7.0.0
Copyright (C) Microsoft Corporation. All rights reserved.
PS C:\>
```

Look closely at the output from the previous commands. You should notice that the shell prompt has changed in each instance. This is because the `-it` ﬂags switch your shell into the terminal of the container — you are literally inside of the new container!

Let’s examine that `docker container run` command.

`docker container run` tells the Docker daemon to start a new container. The `-it` ﬂags tell Docker to make the container interactive and to attach the current shell to the container’s terminal. Next, the command tells Docker that we want the container to be based on the `ubuntu:latest` image. Finally, we tell Docker which process we want to run inside of the container. For the Linux example we’re running a Bash shell, for the Windows container we’re running PowerShell.

Run a ps command from inside of the container to list all running processes.

Linux example:

```shell
root@6dc20d508db0:/# ps -elf
```

Windows example:

```powershell
PS C:\> ps
```

The Linux container only has two processes:

- PID 1. This is the /bin/bash process that we told the container to run with the docker container run command.
- PID 9. This is the `ps -elf` command/process that we ran to list the running processes.

The presence of the `ps -elf` process in the Linux output can be a bit misleading as it is a short-lived process that dies as soon as the ps command completes. This means the only long-running process inside of the container is the `/bin/bash` process.

The Windows container has a lot more going on. This is an artefact of the way the Windows Operating System works. However, even though the Windows container has a lot more processes than the Linux container, it is still a lot less than a regular **Windows Server**.

Press **`Ctrl-PQ`** to exit the container without terminating it. This will land your shell back at the terminal of your Docker host. You can verify this by looking at your shell prompt. Now that you are back at the shell prompt of your Docker host, run the ps command again. Notice how many more processes are running on your Docker host compared to the container you just ran.

**Windows containers run far fewer processes than Windows hosts, and Linux containers run far less than Linux hosts.**

In a previous step, you pressed `Ctrl-PQ` to exit from the container. Doing this from inside of a container will exit you from the container without killing it. You can see all running containers on your system using the **docker container ls** command.

```shell
$ docker container ls
CONTAINER ID    IMAGE           COMMAND     CREATED     STATUS  NAMES
6dc20d508db0    ubuntu:latest   "/bin/bash" 7 mins Up   7 min   vigilant_borg
```

The output above shows a single running container. This is the container that you created earlier. The presence of the container in this output proves that it’s still running. You can also see that it was created 7 minutes ago and has been running for 7 minutes.

You can also see running containers from host operating system with parameterized ps command for viewing process tree:

```shell
$ ps -fax
```

#### Attaching to running containers

You can attach your shell to the terminal of a running container with the docker container exec command. As the container from the previous steps is still running, let’s make a new connection to it.

Linux example:

This example references a container called “vigilant\_borg”. The name of your container will be diﬀerent, so remember to substitute “vigilant\_borg” with the name or ID of the container running on your Docker host.

```shell
$ docker container exec -it vigilant\_borg bash

root@6dc20d508db0:/#
```

Windows example:

This example references a container called “pensive\_hamilton”. The name of your container will be diﬀerent, so remember to substitute “pensive\_hamilton” with the name or ID of the container running on your Docker host.

```powershell
\> docker container exec -it pensive\_hamilton pwsh.exe

PowerShell 7.0.0
Copyright (C) Microsoft Corporation. All rights reserved.
PS C:\>
```

Notice that your shell prompt has changed again. You are logged into the container again. The format of the `docker container exec` command is: **`docker container exec <options> <container-name or container-id> <command/app>`**. In our examples, we used the `-it` options to attach our shell to the container’s shell. We referenced the container by name, and told it to run the bash shell (PowerShell in the Windows example). We could easily have referenced the container by its hex ID. Exit the container again by pressing `Ctrl-PQ`.

Your shell prompt should be back to your Docker host.

Run the `docker container ls` command again to verify that your container is still running.

```shell
$ docker container ls
```

Stop the container and kill it using the docker container stop and `docker container rm` commands. Remember to substitute the names/IDs of your own containers.

```shell
$ docker container stop vigilant\_borg
vigilant\_borg
```

```shell
$ docker container rm vigilant\_borg
vigilant\_borg
```

Verify that the container was successfully deleted by running the `docker container ls` command with the `-a` ﬂag. Adding `-a` tells Docker to list all containers, even those in the stopped state.

```shell
$ docker container ls -a
```

You’ve just pulled a Docker image, started a container from it, attached to it, executed a command inside it, stopped it, and deleted it.

### The Dev Perspective

Containers are all about the apps.

In this section, we’ll clone an app from a Git repo, inspect its Dockerﬁle, containerize it, and run it as a container.

The Linux app can be cloned from: https://github.com/nigelpoulton/psweb.git

The Windows app can be cloned from: https://github.com/nigelpoulton/win-web.git

The rest of this section will focus on the Linux NGINX example. However, both examples are containerizing simple web apps, so the process is the same. Where there are diﬀerences in the Windows example we will highlight them to help you follow along.

Run all of the following commands from a terminal on your Docker host.

Clone the repo locally. This will pull the application code to your local Docker host ready for you to containerize it.

```shell
$ git clone https://github.com/nigelpoulton/psweb.git
```

Both Git repos contain a file called Dockerfile. This is a plain-text document that tells Docker how to build an app and dependencies into a Docker image.

List the contents of the Dockerfile.

```shell
$ cat Dockerfile
FROM alpine
LABEL maintainer="nigelpoulton@hotmail.com"
RUN apk add --update nodejs nodejs-npm
COPY . /src
WORKDIR /src
RUN npm install
EXPOSE 8080
ENTRYPOINT ["node", "./app.js"]
```

The contents of the Dockerﬁle in the Windows example are diﬀerent. However, this isn’t important at this stage. For now, it’s enough to understand that each line represents an instruction that Docker uses to build an image. At this point we have pulled some application code from a remote Git repo. We also have a Dockerﬁle containing instructions on how to build the app into a Docker image. Use the docker image build command to create a new image using the instructions in the Dockerﬁle. This example creates a new Docker image called `test:latest`.

The command is the same for the Linux and Windows examples, and be sure to run it from within the directory containing the app code and Dockerﬁle.

```shell
$ docker image build -t test:latest .
```

    **Note:** 
    It may take a long time for the build to ﬁnish in the Windows example. This is because of the image being pulled is several gigabytes in size.

Once the build is complete, check to make sure that the new test:latest image exists on your host.

```shell
$ docker image ls
```

You have a newly-built Docker image with the app and dependencies inside. Run a container from the image and test the app.

Linux example:

```shell
$ docker container run -d \

--name web1 \

--publish 8080:8080 \

test:latest
```

Open a web browser and navigate to the DNS name or IP address of the Docker host that you are running the container from, and point it to port 8080.

If you are following along with Docker for Windows or Docker for Mac, you will be able to use localhost:8080 or 127.0.0.1:8080. If you’re following along on Play With Docker, you will be able to click the 8080 hyperlink above the terminal screen.

<img src=".\images\HelloDockerLearners.png" style="width:75%; height: 75%;">


Windows example:

```powershell
\> docker container run -d \

--name web1 \

--publish 8080:80 \

test:latest
```

Open a web browser and navigate to the DNS name or IP address of the Docker host that you are running the container from, and point it to port 8080. You will see the following web page. The same rules apply if you’re following along with Docker Desktop or Play With Docker.

<img src=".\images\HelloDockerLearnersWindows.png" style="width:75%; height: 75%;">

Well done. You’ve taken some application code from a remote Git repo and built it into a Docker image. You then ran a container from it. We call this “containerizing an app”.

## The Docker Engine

The **Docker engine** is the core software that runs and manages containers. We often refer to it simply as **Docker**. If you know a thing or two about VMware, it might be useful to think of it as being like ESXi. The Docker engine is modular in design and built from many small specialised tools. Where possible, these are based on open standards such as those maintained by the Open Container Initiative (OCI). 

In many ways, the Docker Engine is like a car engine — both are modular and created by connecting many small specialized parts:

- A car engine is made from many specialized parts that work together to make a car drive — intake manifolds, throttle body, cylinders, spark plugs, exhaust manifolds etc.

- The Docker Engine is made from many specialized tools that work together to create and run containers — APIs, execution driver, runtimes, shims etc.

Docker Engine is a client-server application with these major components:

- Server: It is a type of long-running program called a daemon process (the dockerd command).

- REST API: It specifies the interfaces that programs can use to communicate with the daemon and instructs it on what to do next.

- CLI: It is a command line interface client that is used to write the docker commands.

Docker uses a client-server architecture. The docker client interacts with the Docker daemon that performs running, heavy lifting of building, and distribution of Docker containers.

<img src=".\images\docker-architecture.png" style="width:75%; height: 75%;">

> **Docker daemon:** It accepts the Docker API requests and manages Docker objects, such as images, containers, networks, and volumes. A daemon can also communicate with other daemons to manage Docker services.

> **Docker client:** It is the primary path for Docker users to interact with the Docker application.

> **Docker registries:** It stores Docker images. A Docker registry can be of classified into two categories:
> - **Local Registry:** It helps the user to pull the image.
> - **Docker Trusted Registry:** It is a feature the Docker Enterprise that helps the user to pull the image and scan the image.

> **Docker objects:** When the user uses Docker, in order to package the application and store it in isolated bundles, user creates and uses objects, such as images, containers, services, networks, volumes, plugins.

At the time of writing, the major components that make up the Docker engine are; the **Docker daemon**, **containerd**, **runc**, and various plugins such as networking and storage. Together, these create and run containers.

<img src=".\images\DockerEngine.png" style="width:75%; height: 75%;">

Docker Engine supports the tasks and workflows involved to build, ship, and run container-based applications. The engine creates a daemon process on the server side that hosts volumes of files, containers, networks, and storage.

<img src=".\images\docker-engine.png" style="width:75%; height: 75%;">

### First Release

When Docker was ﬁrst released, the Docker engine had two major components:

- The Docker daemon (hereafter referred to as just “the daemon”)
- LXC (Linux Native Container)

The Docker daemon was a monolithic binary. It contained all of the code for the Docker client, the Docker API, the container runtime, image builds, and much more. LXC provided the daemon with access to the fundamental building-blocks of containers that existed in the Linux kernel. Things like **namespaces** and **control groups (cgroups)**.

<img src=".\images\DockerEngineLXC.png" style="width:75%; height: 75%;">

### Getting rid of LXC

The reliance on LXC was an issue from the start. First up, LXC is Linux-speciﬁc. This was a problem for a project that had aspirations of being multi-platform. Second up, being reliant on an external tool for something so core to the project was a huge risk that could hinder development. As a result, Docker. Inc. developed their own tool called **libcontainer** as a replacement for LXC. The goal of *libcontainer* was to be a platform-agnostic tool that provided Docker with access to the fundamental container building-blocks that exist in the host kernel.

Libcontainer replaced LXC as the default *execution driver* in Docker 0.9.

### Getting rid of the monolithic Docker daemon

Over time, the monolithic nature of the Docker daemon became more and more problematic:

1. It’s hard to innovate on.
2. It got slower.
3. It wasn’t what the ecosystem wanted.

Docker, Inc. was aware of these challanges and began a huge effort to break apart the monolithic daemon and modularize it. The aim of this work was to break out as much of the functionality as possible from the daemon, and re-implement it in smaller specialized tools. These specialized tools can be swapped out, as well as easily re-used by third parties to build other tools. This plan follows the tried-and-tested Unix philosophy of building small specialized tools that can be pieced together into larger tools. This work of breaking apart and re-factoring the Docker engine has seen **all of the container execution and container runtime code entirely removed from the daemon and refactored into small, specialized tools**.

<img src=".\images\DockerEngineDaemon.png" style="width:75%; height: 75%;">

### The inﬂuence of the Open Container Initiative (OCI)

While Docker, Inc. was breaking the daemon apart and refactoring code, the OCI was in the process of deﬁning two container-related speciﬁcations (a.k.a standards):

1. Image spec
2. Container spec

As of Docker 1.11 (early 2016), the Docker engine implements the OCI speciﬁcations as closely as possible. For example, the Docker daemon no longer contains any container runtime code — all container runtime code is implemented in a separate OCI-compliant layer. By default, Docker uses *runc* for this. runc is the *reference implementation* of the OCI container-runtime-spec.

As well as this, the **containerd** component of the Docker Engine makes sure Docker images are presented to **runc** as valid OCI bundles.

### runc

As previously mentioned, **runc** is the reference implementation of the OCI container-runtime-spec. If you strip everything else away, runc is a small, lightweight CLI wrapper for libcontainer (remember that libcontainer originally replaced LXC as the interface layer with the host OS in the early Docker architecture). runc has a single purpose in life — create containers. And it’s damn good at it. And fast! But as it’s a CLI wrapper, it’s effectively a standalone container runtime tool. This means you can download and build the binary, and you’ll have everything you need to build and play with runc (OCI) containers. But it’s bare bones and very low-level, meaning you’ll have none of the richness that you get with the full-blown Docker engine.

### containerd

As part of the effort to strip functionality out of the Docker daemon, all of the container execution logic was ripped out and refactored into a new tool called containerd (pronounced container-dee). Its sole purpose in life was to manage container lifecycle operations — start | stop | pause | rm.

containerd is available as a daemon for Linux and Windows, and Docker has been using it on Linux since the 1.11 release. In the Docker engine stack, containerd sits between the daemon and runc at the OCI layer. As previously stated, containerd was originally intended to be small, lightweight, and designed for a single task in life — container lifecycle operations. However, over time it has branched out and taken on more functionality. Things like image pulls, volumes and networks.

### Starting a new container (example)

Now that we have a view of the big picture, and some of the history, let’s walk through the process of creating a new container. The most common way of starting containers is using the Docker CLI. The following docker container run command will start a simple new container based on the alpine:latest image.

```shell
$ docker container run --name ctr1 -it alpine:latest sh
```

When you type commands like this into the Docker CLI, the Docker client converts them into the appropriate API payload and POSTs them to the API endpoint exposed by the Docker daemon. The API is implemented in the daemon and can be exposed over a local socket or the network. 

On Linux the socket is `/var/run/docker.sock` and on Windows it’s `\pipe\docker\_engine`. Once the daemon receives the command to create a new container, it makes a call to containerd. Remember that the daemon no-longer contains any code to create containers! Despite its name, *containerd* cannot actually create containers. It uses *runc* to do that. It converts the required Docker image into an OCI bundle and tells runc to use this to create a new container. runc interfaces with the OS kernel to pull together all of the constructs necessary to create a container (namespaces, cgroups etc). The container process is started as a child-process of runc, and as soon as it is started runc will exit.

<img src=".\images\DockerEngineShim.png" style="width:75%; height: 75%;">

### One huge beneﬁt of this model

Having all of the logic and code to start and manage containers removed from the daemon means that the entire container runtime is decoupled from the Docker daemon. We sometimes call this “daemonless containers”, and it makes it possible to perform maintenance and upgrades on the Docker daemon without impacting running containers! In the old model, where all of container runtime logic was implemented in the daemon, starting and stopping the daemon would kill all running containers on the host. This was a huge problem in production environments. Every daemon upgrade would kill all containers on that host — not good! Fortunately, this is no longer a problem.

### What’s this shim all about?

Some of the diagrams in the chapter have shown a shim component. The shim is integral to the implementation of daemonless containers (what we just mentioned about decoupling running containers from the daemon for things like daemon upgrades). We mentioned earlier that *containerd* uses runc to create new containers. In fact, it forks a new instance of runc for every container it creates. However, once each container is created, the parent runc process exits. This means we can run hundreds of containers without having to run hundreds of runc instances. Once a container’s parent runc process exits, the associated containerd-shim process becomes the container’s parent. 

Some of the responsibilities the shim performs as a container’s parent include:

- Keeping any STDIN and STDOUT streams open so that when the daemon is restarted, the container doesn’t terminate due to pipes being closed etc.
- Reports the container’s exit status back to the daemon.

### How it’s implemented on Linux

On a Linux system, the components we’ve discussed are implemented as separate binaries as follows:

> - dockerd (the Docker daemon)
> - docker-containerd (containerd)
> - docker-containerd-shim (shim)
> - docker-runc (runc)

You can see all of these on a Linux system by running a ps command on the Docker host. Obviously, some of them will only be present when the system has running containers.

### What’s the point of the daemon

At the time of writing, some of the major functionality that still exists in the daemon includes; image management, image builds, the REST API, authentication, security, core networking, and orchestration.

### Securing client and daemon communication

Docker implements a client-server model.

- The client component implements the CLI
- The server (daemon) component implements the functionality, including the public-facing REST API

The client is called docker (docker.exe on Windows) and the daemon is called dockerd (dockerd.exe on Windows). A default installation puts them on the same host and conﬁgures them to communicate over a local IPC socket:

> - `/var/run/docker.sock` on Linux
> - `./pipe/docker\_engine` on Windows

It’s also possible to conﬁgure them to communicate over the network. By default, network communication occur over an unsecured HTTP socket on port 2375/tcp.

<img src=".\images\DockerSecurity.png" style="width:75%; height: 75%;">

An insecure conﬁguration like this might be suitable for labs, but it’s unacceptable for anything else. TLS to the rescue! Docker lets you force the client and daemon to only accept network connections that are secured with TLS. This is recommended for production environments, even if all traffic is traversing trusted internal networks. You can secure both the client and the daemon. Securing the client forces the client to only connect to Docker daemons with certiﬁcates signed by a trusted Certiﬁcate Authority (CA). Securing the daemon forces the daemon to only accept connections from clients presenting certiﬁcates from a trusted CA. A combination of both modes offers the most security.

## Images

**A container image is a template for a running container and it is created in the form of a tarball with configuration files.** The image includes container configuration options and runtime settings to guide the container’s behavior at runtime. Container runtimes load images to run them as containers, therefore at runtime, a container becomes a running instance of an image, and there may be many containers started from the same image.

There are several tools available to build container images. Some of the container image building tools support only one of the OCI image format or the now retired ACI image format, while other tools build container images based on both formats. Container image building tools offer various methods of image building. 

Docker may use a specific configuration file called a Dockerfile in conjunction with the docker build command, while Podman may use a Containerfile or Dockerfile with the podman build command. Buildah, however, can use a Dockerfile with the buildah bud (build-using-dockerfile) command. 

**Container images can be created from scratch, from another container image, or directly from an existing running container.** Building a container image can be pretty straight forward if default options are used during the build process, but it could become challenging if options are customized. Ultimately, there are also tools to convert images between the two distinct image formats, OCI and ACI.

**An image, just as any program file or package of files, consumes a single type of resource at rest - storage.** However, during certain operations with the image, network bandwidth may be consumed while downloading/uploading the image to/from a container image registry. 

**The storage of a container image follows a layered approach.** The first or bottom layer of an image file will have saved the configuration information of the base image - that is where everything started. This layer is mounted in the running container in a read-only mode. In time, features have been added to the image, and every feature has been saved as an additional layer, one on top of another, while the base image data remains intact. There may be images that started from the same base image, but in time they all took different paths, as various features have been added to serve different purposes for different projects. 

**The layered approach in saving new features helps in reducing the overall disk space footprint of the image. It also avoids data duplication, where only the latest layer’s changes are being saved on the disk and the rest of the bottom level layers are just referenced, and it speeds up the image build process by caching each build step.**

### Docker images

A Docker image is a unit of packaging that contains everything required for an application to run. An image holds instructions that are required to run an application. This includes; application code, application dependencies, and OS constructs. If you have an application’s Docker image, the only other thing you need to run that application is a computer running Docker.

If you’re a former VM admin, you can think of Docker images as similar to VM templates. A VM template is like a stopped VM — a Docker image is like a stopped container. If you’re a developer you can think of them as similar to *classes*.

You get Docker images by *pulling* them from an image registry. The most common registry is **`DockerHub`** but others exist. The *pull* operation downloads the image to your local Docker host where Docker can use it to start one or more containers.

Images are made up of multiple *layers* that are stacked on top of each other and represented as a single object. Inside of the image is a cut-down operating system (OS) and all of the ﬁles and dependencies required to run an application. Because containers are intended to be fast and lightweight, images tend to be small (Microsoft images tend to be huge).

We’ve mentioned a couple of times already that **images** are like stopped containers. In fact, you can stop a container and create a new image from it. With this in mind, images are considered *build-time* constructs, whereas containers are *run-time* constructs.

<img src=".\images\DockerImageContainer.png" style="width:75%; height: 75%;">

### Images and containers

Figure above shows high-level view of the relationship between images and containers. We use the `docker container run` and `docker service create` commands to start one or more containers from a single image. Once you’ve started a container from an image, the two constructs become dependent on each other and you cannot delete the image until the last container using it has been stopped and destroyed. Attempting to delete an image without stopping and destroying all containers using it will result in an error.

### Images are usually small

The whole purpose of a container is to run a single application or service. This means it only needs the code and dependencies of the app/service it is running — it does not need anything else. This results in small images stripped of all non-essential parts. For example, Docker images do not ship with 6 diﬀerent shells for you to choose from. In fact, many application images ship without a shell – if the application doesn’t need a shell to run it doesn’t need to be included in the image. General purpose images such as busybox and Ubuntu ship with a shell, but when you package your business applications for production, you will probably package them without a shell.

**Image also don’t contain a kernel — all containers running on a Docker host share access to the host’s kernel. For these reasons, we sometimes say images contain *just enough operating system* (usually just OS-related ﬁles and ﬁlesystem objects).**

> **Note:** Hyper-V containers run a single container inside of a dedicated lightweight VM. The container leverages the kernel of the OS running inside the VM.

The oﬃcial *Alpine Linux* Docker image is about 5MB in size and is an extreme example of how small Docker images can be. That's not a typo! It really is about 5 megabytes! Some images are even smaller, however, a more typical example might be something like the oﬃcial Ubuntu Docker image which is currently about 40MB. These are clearly stripped of most non-essential parts!

Windows-based images tend to be a lot bigger than Linux-based images because of the way that the Windows OS works. It’s not uncommon for Windows images to be several gigabytes and take a long time to pull.

### Pulling images

A cleanly installed Docker host has no images in its local repository. The local image repository on a Linux-based Docker host is usually located at `/var/lib/docker/<storage-driver>`.

On Windows-based Docker hosts this is `C:\ProgramData\docker\windowsfilter`. If you’re using Docker on your Mac or PC with Docker Desktop, everything runs inside of a VM.

You can use the following command to check if your Docker host has any images in its local repository.

```shell
$ docker image ls
```

The process of Getting images onto a Docker host is called *pulling*. So, if you want the latest Busybox image on your Docker host, you’d have to *pull* it. Use the following commands to *pull* some images and then check their sizes.

If you are following along on Linux and haven’t added your user account to the local docker Unix group, you may need to add sudo to the beginning of all the following commands.

Linux example:

```shell
$ docker image pull redis:latest

$ docker image pull alpine:latest

$ docker image ls
```

As you can see, the images just pulled are now present in the Docker host’s local repository. You can also see that the Windows images are a lot larger and comprise many more layers.

### Image naming

As part of each command, we had to specify which image to pull. Let’s take a minute to look at image naming. To do that we need a bit of background on how images are stored.

### Image registries

We store images in centralised places called *image registries*. This makes it easy to share and access them. The most common registry is Docker Hub (https://hub.Docker.com). Other registries exist, including 3rd party registries and secure on-premises registries. However, the Docker client is opinionated and defaults to using Docker Hub. We’ll be using Docker Hub for the rest of the book.

The output of the following command is snipped, but you can see that Docker is conﬁgured to use https://index.docker.io/v1/as its default registry when pushing and pulling images (this actually redirects to v2).

```shell
$ docker info
```

Image registries contain one or more *image repositories*. In turn, image repositories contain one or more images. That might be a bit confusing, so Figure below shows a picture of an image registry with 3 repositories, and each repository has one or more images.

<img src=".\images\DockerRegistry.png" style="width:75%; height: 75%;">

### Oﬃcial and unoﬃcial repositories

Docker Hub has the concept of *oﬃcial repositories* and *unoﬃcial repositories*. As the name suggests, *oﬃcial repositories* are the home to images that have been vetted and curated by Docker, Inc. This means they should contain up-to-date, high-quality code, that is secure, well-documented, and in-line with best practices.

*Unoﬃcial repositories* can be like the wild-west — you should not assume they are safe, well-documented or built according to best practices. That's not saying everything in *unoﬃcial repositories* is bad. There’s some excellent stuﬀ in *unoﬃcial repositories*. You just need to be very careful before trusting code from them. To be honest, you should always be careful when trusting software from the internet — even images from *oﬃcial repositories.* Most of the popular applications and base operating systems have their own *oﬃcial repositories* on Docker Hub.

They’re easy to spot because they live at the top level of the Docker Hub namespace. The following list contains a few of the *oﬃcial repositories*, and shows their URLs that exist at the top-level of the Docker Hub namespace:

> - **nginx:** https://hub.Docker.com/\_/nginx/
> - **busybox:** https://hub.Docker.com/\_/busybox/
> - **redis:** https://hub.Docker.com/\_/redis/
> - **mongo:** https://hub.Docker.com/\_/mongo/

On the other hand, personal images live in the wild west of *unoﬃcial repositories* and should **not** be trusted. Here are some examples of images:

- nigelpoulton/tu-demo — https://hub.Docker.com/r/nigelpoulton/tu-demo/
- nigelpoulton/pluralsight-Docker-ci — https://hub.Docker.com/r/nigelpoulton/pluralsight-Docker-ci/

Not only are images in personal repositories **not** vetted, **not** kept up-to-date, **not** secure, and **not** well documented. They also don’t live at the top-level of the Docker Hub namespace. Personal repositories all live within the second-level namespace. 

You’ll probably notice that the Microsoft images we’ve used do not exist at the top-level of the Docker Hub namespace. At the time of writing, they exist under the oﬃcial mcr.microsoft.com second-level namespace. This is due to legal reasons requiring them to be hosted outside of Docker Hub. However, they are integrated into the Docker Hub namespace to make the experience of pulling them as seamless as possible.

After all of that, we can ﬁnally look at how we address images on the Docker command line.

### Image naming and tagging

Addressing images from oﬃcial repositories is as simple as providing the repository name and tag separated by a colon (:). The format for docker image pull, when working with an image from an oﬃcial repository is:

```shell
$ docker image pull <repository>:<tag>
```

In the Linux examples from earlier, we pulled an Alpine and a Redis image with the following two commands:

```shell
$ docker image pull alpine:latest

$ docker image pull redis:latest
```

These two commands pull the images tagged as “latest” from the top-level “alpine” and “redis” repositories.

The following examples show how to pull various diﬀerent images from *oﬃcial repositories*:

```shell
$ docker image pull mongo:4.2.6

//This will pull the image tagged as `4.2.6` from the official `mongo` repository.

$ docker image pull busybox:latest

//This will pull the image tagged as `latest` from the official `busybox` repository.

$ docker image pull alpine

//This will pull the image tagged as `latest` from the official `alpine` repository.
```

A couple of points about those commands.

First, if you **do not** specify an image tag after the repository name, Docker will assume you are referring to the image tagged as latest. If the repository doesn’t have an image tagged as latest the command will fail.

Second, the latest tag doesn’t have any magical powers. Just because an image is tagged as latest does not guarantee it is the most recent image in a repository. For example, the most recent image in the alpine repository is usually tagged as edge. Moral of the story — take care when using the latest tag.

Pulling images from an *unoﬃcial repository* is essentially the same — you just need to prepend the repository name with a Docker Hub username or organization name. The following example shows how to pull the v2 image from the tu-demo repository owned by a not-to-be-trusted person whose Docker Hub account name is `nigelpoulton`.

```shell
$ docker image pull nigelpoulton/tu-demo:v2

//This will pull the image tagged as `v2` from the `tu-demo` repository within the `nigelpoulton` namespace
```

If you want to pull images from 3rd party registries (not Docker Hub), you need to prepend the repository name with the DNS name of the registry. For example, the following command pulls the 3.1.5 image from the google-containers/git-sync repo on the Google Container Registry (gcr.io).

```shell
$ docker image pull gcr.io/google-containers/git-sync:v3.1.5
```

Notice how the pull experience is exactly the same from Docker Hub and the Google Container Registry.

### Images with multiple tags

One ﬁnal word about image tags: A single image can have as many tags as you want. This is because tags are arbitrary alpha-numeric values that are stored as metadata alongside the image. 

Let’s look at an example. 

**Pull all of the images in a repository by adding the -a ﬂag to the docker image pull command.** 

Then run `docker image ls` to look at the images pulled. It’s probably not a good idea to pull all images from an mcr.microsoft.com repository because Microsoft images can be so large. Also, if the repository you are pulling contains images for multiple architectures and platforms, such as Linux and Windows, the command is likely to fail. We recommend you use the command and repository in the following example.

```shell
$ docker image pull -a nigelpoulton/tu-demo

$ docker image ls
```

A couple of things about what just happened:

First. The command pulled three images from the nigelpoulton/tu-demo repository: latest, v1, and v2.

Second. Look closely at the `IMAGE ID` column in the output of the `docker image ls` command. You’ll see that two of the IDs match. This is because two of the tags refer to the same image. Put another way, one of the images has two tags. If you look closely, you’ll see that the v2 and latest tags have the same IMAGE ID. **This means they’re two tags of the same image**.

This is a perfect example of the warning issued earlier about the latest tag. In this example, the latest tag refers to the same image as the v2 tag. This means it’s pointing to the older of the two images! Moral of the story, latest is an arbitrary tag and is not guaranteed to point to the newest image in a repository!

### Filtering the output of docker image ls

The format of filter flag is a key-value pair. Docker provides the `--filter` ﬂag to ﬁlter the list of images returned by `docker image ls`.

Filter option is used in docker images to filter:

- Images that are not tagged.
- Images that are labelled.
- Images by time.
- Images by reference.

The following example will only return dangling images.

```shell
$ docker image ls --filter dangling=true
```

A dangling image is an image that is no longer tagged, and appears in listings as <none>:<none>. A common way they occur is when building a new image giving it a tag that already exists. When this happens, Docker will build the new image, notice that an existing image already has the same tag, remove the tag from the existing image and give it to the new image.

Consider this example, you build a new application image based on `alpine:3.4` and tag it as `dodge:challenger`. Then you update the image to use `alpine:3.5` instead of `alpine:3.4`. When you build the new image, the operation will create a new image tagged as `dodge:challenger` and remove the tags from the old image. The old image will become a dangling image.

**You can delete all dangling images on a system with the `docker image prune` command. If you add the `-a` ﬂag, Docker will also remove all unused images (those not in use by any containers).**

> Docker currently supports the following ﬁlters:
> - `dangling`: Accepts true or false, and returns only dangling images (true), or non-dangling images (false).
> - `before`: Requires an image name or ID as argument, and returns all images created before it.
> - `since`: Same as above, but returns images created after the specified image.
> - `label`: Filters images based on the presence of a label or label and value. The docker image ls command does not display labels in its output.

For all other ﬁltering you can use `reference`.

Here’s an example using reference to display only images tagged as “latest”.

```shell
$ docker image ls --filter=reference="\*:latest"
```

You can also use the `--format` ﬂag to format output using Go templates. 

Format option is used in docker image to filter:

- Image ID
- Image repository
- Image tag
- Image digest
- Image disk size
- Time at which the image was created
- Time elapsed since the creation of the image

For example, the following command will only return the size property of images on a Docker host.

```shell
$ docker image ls --format "{{.Size}}"
```

Use the following command to return all images, but only display repo, tag and size.

```shell
$ docker image ls --format "{{.Repository}}: {{.Tag}}: {{.Size}}"
```

If you need more powerful ﬁltering, you can always use the tools provided by your OS and shell such as `grep` and `awk`.

### Searching Docker Hub from the CLI

The docker search command lets you search Docker Hub from the CLI. This has limited value as you can only pattern-match against strings in the “NAME” ﬁeld. However, you can ﬁlter output based on any of the returned columns.

In its simplest form, it searches for all repos containing a certain string in the “NAME” ﬁeld. For example, the following command searches for all repos with “nigelpoulton” in the “NAME” ﬁeld.

```shell
$ docker search nigelpoulton
```

The “NAME” ﬁeld is the repository name. This includes the Docker ID, or organization name, for unoﬃcial repositories. For example, the following command will list all repositories that include the string “alpine” in the name.

```shell
$ docker search alpine
```

Notice how some of the repositories returned are oﬃcial and some are unoﬃcial. You can use `--filter "is-official=true"` so that only oﬃcial repos are displayed.

Filter option is used in docker search to filter:

- Images according to the stars scored.
- Images according to their automation status.
- Images according to their official status.

```shell
$ docker search alpine --filter "is-official=true"
```

You can do the same again, but this time only show repos with automated builds.

```shell
$ docker search alpine --filter "is-automated=true"
```

One last thing about docker search. By default, Docker will only display 25 lines of results. However, you can use the `--limit` ﬂag to increase that to a maximum of 100.

Format option is used in docker search to filter. The format option --format helps in identifying:

- Image name
- Image description
- Image stars
- Official image
- Automated image

### Images and layers

A Docker image is just a bunch of loosely-connected read-only layers, with each layer comprising one or more ﬁles. This is shown in Figure below.

<img src=".\images\ImageLayers.png" style="width:75%; height: 75%;">


A Docker image is built from a series of layers. Each layer is an instruction in the Dockerfile of the image. Except the very last layer, each layer is read-only.

<img src=".\images\docker-image-layers.png" style="width:75%; height: 75%;">

Docker takes care of stacking these layers and representing them as a single uniﬁed object. There are a few ways to see and inspect the layers that make up an image. In fact, we saw one earlier when pulling images. The following example looks closer at an image pull operation.

```shell
$ docker image pull ubuntu:latest

latest: Pulling from library/ubuntu

952132ac251a: Pull complete

82659f8f1b76: Pull complete

c19118ca682d: Pull complete

8296858250fe: Pull complete

24e0251a0e2c: Pull complete

Digest: sha256:f4691c96e6bbaa99d...28ae95a60369c506dd6e6f6ab

Status: Downloaded newer image for ubuntu:latest

docker.io/ubuntu:latest
```

Each line in the output above that ends with “Pull complete” represents a layer in the image that was pulled. As we can see, this image has 5 layers. Figure below shows this in picture form with layer IDs.

<img src=".\images\ImageLayersSha.png" style="width:75%; height: 75%;">

**Identifying the Layers:** Layers of an image can be identified using the following commands:

<img src=".\images\docker-image-layer-identify.png" style="width:75%; height: 75%;">

Another way to see the layers of an image is to inspect the image with the `docker image inspect` command.

The following example inspects the same `ubuntu:latest` image.

```shell
$ docker image inspect ubuntu:latest
```

The trimmed output shows 5 layers again. Only this time they’re shown using their SHA256 hashes. **The docker image inspect command is a great way to see the details of an image.* 

**The docker history command is another way of inspecting an image and seeing layer data. However, it shows the build history of an image and is **not** a strict list of layers in the ﬁnal image.** For example, some Dockerﬁle instructions (“ENV”, “EXPOSE”, “CMD”, and “ENTRYPOINT”) add metadata to the image and do not result in permanent layers being created.

**All Docker images start with a base layer, and as changes are made and new content is added, new layers are added on top.** 

Consider the following oversimpliﬁed example of building a simple Python application. You might have a corporate policy that all applications are based on the oﬃcial `Ubuntu 20:04` image. This would be your image’s *base layer*. If you then add the Python package, this will be added as a second layer on top of the base layer. If you later add source code ﬁles, these will be added as additional layers. Your ﬁnal image would have three layers as shown in Figure below (remember this is an over-simpliﬁed example for demonstration purposes).

<img src=".\images\ImageLayersBase.png" style="width:75%; height: 75%;">

It’s important to understand that as additional layers are added, the *image* is always the combination of all layers stacked in the order they were added. Take a simple example of two layers as shown in Figure below. Each *layer* has 3 ﬁles, but the overall *image* has 6 ﬁles as it is the combination of both layers.

<img src=".\images\ImageLayersInside.png" style="width:75%; height: 75%;">

> **Note:** We’ve shown the image layers in Figure above in a slightly diﬀerent way to previous ﬁgures.

This is just to make showing the ﬁles easier.

In the slightly more complex example of the three-layer image in Figure below, the overall image only presents 6 ﬁles in the uniﬁed view. This is because File 7 in the top layer is an updated version of File 5 directly below (inline). In this situation, the ﬁle in the higher layer obscures the ﬁle directly below it. This allows updated versions of ﬁles to be added as new layers to the image.

<img src=".\images\ImageLayersInside2.png" style="width:75%; height: 75%;">

**Docker employs a storage driver that is responsible for stacking layers and presenting them as a single uniﬁed ﬁlesystem/image.** Examples of storage drivers on Linux include AUFS, overlay2, devicemapper, btrfs and zfs.

As their names suggest, each one is based on a Linux ﬁlesystem or block-device technology, and each has its own unique performance characteristics. The only driver supported by Docker on Windows is windowsfilter, which implements layering and CoW on top of NTFS.

No matter which storage driver is used, the user experience is the same.     

Figure below shows the same 3-layer image as it will appear to the system. i.e. all three layers stacked and merged, giving a single uniﬁed view.

<img src=".\images\ImageLayersInside3.png" style="width:75%; height: 75%;">

### Sharing image layers

Multiple images can, and do, share layers. This leads to eﬃciencies in space and performance. Let’s take a second look at the docker image pull command with the -a ﬂag that we ran previously to pull all tagged images in the nigelpoulton/tu-demo repository.

```shell
$ docker image pull -a nigelpoulton/tu-demo

latest: Pulling from nigelpoulton/tu-demo

aad63a933944: Pull complete

f229563217f5: Pull complete

...

Digest: sha256:c9f8e18822...6cbb9a74cf

v1: Pulling from nigelpoulton/tu-demo

aad63a933944: Already exists

f229563217f5: Already exists

...

fc669453c5af: Pull complete

Digest: sha256:674cb03444...f8598e4d2a

v2: Pulling from nigelpoulton/tu-demo

Digest: sha256:c9f8e18822...6cbb9a74cf

Status: Downloaded newer image for nigelpoulton/tu-demo

docker.io/nigelpoulton/tu-demo

$ docker image ls

```

Notice the lines ending in Already exists.

These lines tell us that Docker is smart enough to recognize when it’s being asked to pull an image layer that it already has a local copy of. In this example, Docker pulled the image tagged as latest ﬁrst. Then, when it pulled the v1 and v2 images, it noticed that it already had some of the layers that make up those images. This happens because the three images in this repository are almost identical, and therefore share many layers. In fact, the only diﬀerence between v1 and v2 is the top layer.

As mentioned previously, Docker on Linux supports many storage drivers. Each is free to implement image layering, layer sharing, and copy-on-write (CoW) behaviour in its own way. However, the overall result and user experience is essentially the same. Although Windows only supports a single storage driver, that driver provides the same experience as Linux.

### The Copy-on-Write (COW) Strategy

Copy-on-write is a strategy of sharing and copying files for maximum efficiency. If there is a file or directory within the image in a lower layer and another layer (including the writable layer) needs read access to it, it just uses the existing file.

When does a user use docker pull?

- To pull down an image from a repository, or create a container from an image that does not exist locally.
- To pull down each layer separately and store in Docker’s local storage area, which is usually /var/lib/docker/ on Linux hosts.

If the user builds images from the two Docker files, the user can use docker image and docker history commands to verify that the cryptographic IDs of the shared layers are the same.

### Pulling images by digest

So far, we’ve shown you how to pull images using their name (tag). This is by far the most common method, but it has a problem — tags are mutable! This means it’s possible to accidentally tag an image with the wrong tag (name). Sometimes, it’s even possible to tag an image with the same tag as an existing, but diﬀerent, image. This can cause problems!

As an example, imagine you’ve got an image called golftrack:1.5 and it has a known bug. You pull the image, apply a ﬁx, and push the updated image back to its repository using the **same tag**. Take a moment to consider what happened there. You have an image called golftrack:1.5 that has a bug. That image is being used by containers in your production environment. You create a new version of the image that includes a ﬁx. Then comes the mistake. You build and push the ﬁxed image back to its repository with the **same tag as the vulnerable image!**. This overwrites the original image and leaves you without a great way of knowing which of your production containers are using the vulnerable image and which are using the ﬁxed image — they both have the same tag!

This is where *image digests* come to the rescue.

**Docker 1.10 introduced a content addressable storage model. As part of this model, all images get a cryptographic *content hash*. For the purposes of this discussion, we’ll refer to this hash as the *digest*. As the digest is a hash of the contents of the image, it’s impossible to change the contents of the image without creating a new unique digest.** Put another way, you cannot change the content of an image and keep the old digest. This means digests are immutable and provide a solution to the problem we just talked about.

Every time you pull an image, the docker image pull command includes the image’s digest as part of the information returned. You can also view the digests of images in your Docker host’s local repository by adding the `--digests` ﬂag to the docker image ls command. These are both shown in the following example.

```shell
$ docker image pull alpine

$ docker image ls --digests alpine
```

The snipped output above shows the digest for the alpine image as - sha256:9a839e63da...9ea4fb9a54

Now that we know the digest of the image, we can use it when pulling the image again. This will ensure that we get **exactly the image we expect!**

The following example deletes the alpine:latest image from your Docker host and then shows how to pull it again using its digest instead of its tag. The actual digest is truncated so that it ﬁts on one line. Substitute this for the real digest of the version you pulled on your own system.

```shell
$ docker image rm alpine:latest

Untagged: alpine:latest

Untagged: alpine@sha256:c0537...7c0a7726c88e2bb7584dc96

Deleted: sha256:02674b9cb179d...abff0c2bf5ceca5bad72cd9

Deleted: sha256:e154057080f40...3823bab1be5b86926c6f860

...

$ docker image pull alpine@sha256:9a839e63da...9ea4fb9a54
```

### Multi-architecture images

One of the best things about Docker is its simplicity. However, as technologies grow, things get more complex. This happened for Docker when it started supporting multiple diﬀerent platforms and architectures such as Windows and Linux, on variations of ARM, x64, PowerPC, and s390x. All of a sudden, popular images had versions for diﬀerent platforms and architectures. As developers and operators, we had to make sure we were pulling the correct version for the platform and architecture we were using. This broke the smooth Docker experience.

> **Note:** We’re using the term “architecture” to refer to CPU architecture such as x64 and ARM. We use the term “platform” to refer to either the OS (Linux or Windows) or the combination of OS and architecture.

Multi-architecture images to the rescue!

Fortunately, Docker and Docker Hub have a slick way of supporting multi-arch images. This means a single image, such as golang:latest, can have an image for Linux on x64, Linux on PowerPC, Windows x64, Linux on diﬀerent versions of ARM, and more. To be clear, we’re talking about a single image tag supporting multiple platforms and architectures. We’ll see it in action in a second, but it means you can run a simple docker image pull goloang:latest from any platform or architecture and Docker will pull the correct image for your platform and architecture. 

To make this happen, the Registry API supports two important constructs:

- **manifest lists**
- **manifests**

The **manifest list** is exactly what it sounds like: a list of architectures supported by a particular image tag. each supported architecture then has its own *\*manifest* detailing the layers that make it up. Figure below uses the oﬃcial golang image as an example. On the left is the **manifest list** with entries for each architecture the image supports. The arrows show that each entry in the **manifest list** points to a **manifest** containing image conﬁg and layer data.

<img src=".\images\MultiArchImage.png" style="width:75%; height: 75%;">

Let’s look at the theory before seeing it in action.

Assume you are running Docker on a Raspberry Pi (Linux running on ARM architecture). When you pull an image, your Docker client makes the relevant calls to the Docker Registry API exposed by Docker Hub. If a **manifest list** exists for the image, it will be parsed to see if an entry exists for Linux on ARM. If an ARM entry exists, the **manifest** for that image is retrieved and parsed for the crypto ID’s of the layers that make up the image. Each layer is then pulled from Docker Hub. The following examples show how this works by starting a new container from the oﬃcial golang image and running the go version command inside the container. The output of the go version command shows the version of Go as well as the platform and CPU architecture of the container/host. The thing to note, is that both examples use the exact same docker container run command. We do not have to tell Docker that we need the Linux x64 or Windows x64 versions of the image. We just run normal commands and let Docker take care of Getting the right image for the platform and architecture we are running!

Linux on x64 example:

```shell
$ docker container run --rm golang go version

go version go1.14.2 linux/amd64
```

Windows on x64 example:

```powershell
\> docker container run --rm golang go version

go version go1.14.2 windows/amd64
```

The Windows Golang image is currently over 5GB in size and may take a long time to download. The ‘Docker manifest’ command lets you inspect the manifest list of any image on Docker Hub. The following example inspects the manifest list on Docker Hub for the golang image. You can see that Linux and Windows are supported on various CPU architectures. You can run the same command without the grep ﬁlter to see the full JSON manifest list.

```shell
$ docker manifest inspect golang | grep 'architecture\|os'

"architecture": "amd64",

"os": "linux"

"architecture": "arm",

"os": "linux",

"architecture": "arm64",

"os": "linux",

"architecture": "386",

"os": "linux"

"architecture": "ppc64le",

"os": "linux"

"architecture": "s390x",

"os": "linux"

"architecture": "amd64",

"os": "windows",

"os.version": "10.0.14393.3630"

"architecture": "amd64",

"os": "windows",

"os.version": "10.0.17763.1158"
```

All oﬃcial images have manifest lists. You can create your own builds for diﬀerent platforms and architectures with docker buildx and then use docker manifest to create your own manifest lists. The following command builds an image for ARMv7 called myimage:arm-v7 from the contents of the current directory. It’s based on code in the code in https://github.com/nigelpoulton/psweb.

```shell
$ docker buildx build --platform linux/arm/v7 -t myimage:arm-v7 .
```

The beauty of the command is that you don’t have to run it from an ARMv7 Docker node. In fact, the example shown was ran on Linux on x64 hardware. At the time of writing, buildx is an experimental feature and requires experimental=true setting in your `∼/.docker/config.json` ﬁle as follows.

```shell
{

"experimental": true

}
```

### Deleting Images

When you no longer need an image on your Docker host, you can delete it with the docker image rm command. rm is short for remove. Deleting an image will remove the image and all of its layers from your Docker host. This means it will no longer show up in docker image ls commands and all directories on the Docker host containing the layer data will be deleted. However, if an image layer is shared by more than one image, that layer will not be deleted until all images that reference it have been deleted. Delete the images pulled in the previous steps with the docker image rm command. The following example deletes an image by its ID, this might be diﬀerent on your system.

```shell
$ docker image rm 02674b9cb179

Untagged: alpine@sha256:c0537ff6a5218...c0a7726c88e2bb7584dc96

Deleted: sha256:02674b9cb179d57...31ba0abff0c2bf5ceca5bad72cd9

Deleted: sha256:e154057080f4063...2a0d13823bab1be5b86926c6f860
```

You can list multiple images on the same command by separating them with whitespace like the following.

```shell
$ docker image rm f70734b6a266 a4d3716dbb72
```

If the image you are trying to delete is in use by a running container you will not be able to delete it. Stop and delete any containers before trying the delete operation again. A handy shortcut for **deleting all images** on a Docker host is to run the docker image rm command and pass it a list of all image IDs on the system by calling docker image ls with the -q ﬂag. This is shown next.

If you are following along on a Windows system, this will only work in a PowerShell terminal. It will not work on a CMD prompt.

```shell
$ docker image rm $(docker image ls -q) -f
```

To understand how this works, download a couple of images and then run docker image ls -q.

```shell
$ docker image pull alpine

Using default tag: latest

latest: Pulling from library/alpine

e110a4a17941: Pull complete

Digest: sha256:3dcdb92d7432d5...3626d99b889d0626de158f73a

Status: Downloaded newer image for alpine:latest

...

$ docker image pull ubuntu

Using default tag: latest

latest: Pulling from library/ubuntu

952132ac251a: Pull complete

82659f8f1b76: Pull complete

c19118ca682d: Pull complete

8296858250fe: Pull complete

24e0251a0e2c: Pull complete

Digest: sha256:f4691c96e6bba...128ae95a60369c506dd6e6f6ab

Status: Downloaded newer image for ubuntu:latest

...

$ docker image ls -q

bd3d4369aebc

4e38e38c8ce0
```

See how docker image ls -q returns a list containing just the image IDs of all images pulled locally on the system. Passing this list to docker image rm will delete all images on the system as shown next.

```shell
$ docker image rm $(docker image ls -q) -f

Untagged: ubuntu:latest

Untagged: ubuntu@sha256:f4691c9...2128ae95a60369c506dd6e6f6ab

Deleted: sha256:bd3d4369aebc494...fa2645f5699037d7d8c6b415a10

Deleted: sha256:cd10a3b73e247dd...c3a71fcf5b6c2bb28d4f2e5360b

Deleted: sha256:4d4de39110cd250...28bfe816393d0f2e0dae82c363a

Deleted: sha256:6a89826eba8d895...cb0d7dba1ef62409f037c6e608b

Deleted: sha256:33efada9158c32d...195aa12859239d35e7fe9566056

Deleted: sha256:c8a75145fcc4e1a...4129005e461a43875a094b93412

Untagged: alpine:latest

Untagged: alpine@sha256:3dcdb92...313626d99b889d0626de158f73a

Deleted: sha256:4e38e38c8ce0b8d...6225e13b0bfe8cfa2321aec4bba

Deleted: sha256:4fe15f8d0ae69e1...eeeeebb265cd2e328e15c6a869f

...

$ docker image ls
```

## Containers

Most times, to ease the understanding of the concept, a container is compared to a regular Virtual Machine - which is a result of full virtualization. **A container, however, is the product of several OS-level virtualization features of the Linux kernel used in conjunction to build a lightweight isolated environment.** These environments provide secure runtimes from simple scripts to full-sized web-servers. 

**It is quite obvious why containers are compared to VMs: they have their own process trees, network interfaces, users, root, file systems, just to name a few reasons.** There are quite a few differences as well, between VMs and containers: containers use the host kernel and are bound to boot the host OS only, and are processes running on the host system managed individually or in groups, while VMs allow for the installation of guest OSes that may be different than the OS of the host system.

**A container is the runtime instance of an image.** In the same way that you can start a virtual machine (VM) from a virtual machine template, you start one or more containers from a single image. The big difference between a VM and a container is that containers are faster and more lightweight — instead of running a full-blown OS like a VM, containers share the OS/kernel with the host they’re running on. It’s also common for containers to be based on minimalist images that only include software and dependencies required by the application.

<img src=".\images\DockerContainers.png" style="width:75%; height: 75%;">

The simplest way to start a container is with the docker container run command. The command can take a lot of arguments, but in its most basic form you tell it an image to use and a app to run: docker container run image app. The following command will start an Ubuntu Linux container running the Bash shell as its app.

```shell
$ docker container run -it ubuntu /bin/bash
```

In each of the examples, the -it ﬂags will connect your current terminal window to the container’s shell. Containers run until the app they are executing exits. In the previous examples, the Linux container will exit when the Bash shell exits, and the Windows container will exit when the PowerShell process terminates. A simple way to demonstrate this is to start a new container and tell it to run the sleep command for 10 seconds.

```shell
$ docker container run -it alpine:latest sleep 10
```

You can manually stop a running container with the docker container stop command. You can then restart it with docker container start. To get rid of a container forever, you have to explicitly delete it with docker container rm.

### Containers vs VMs

**Containers and VMs both need a host to run on. This can be anything from your laptop, a bare metal server in your data center, all the way up to an instance in the public cloud. In fact, many cloud services now offer the ability to run containers on ephemeral serverless back-ends. Don’t worry if that sounds like techno-babble, it just means that the back-end is so highly virtualized that the concept of a host or node no longer has any meaning — your container simply runs, and you don’t need to care about the *how* or *where*.**

Anyway, let’s assume a requirement where your business has a single physical server that needs to run 4 business applications. In the VM model, the physical server is powered on and the hypervisor boots (we’re skipping the BIOS and bootloader code etc.). Once booted, the hypervisor lays claim to all physical resources on the system such as CPU, RAM, storage, and NICs. It then carves these hardware resources into virtual versions that look smell and feel exactly like the real thing. It then packages them into a software construct called a virtual machine (VM). We take those VMs and install an operating system and application on each one. Assuming the scenario of a single physical server that needs to run 4 business applications, we’d create 4 VMs, install 4 operating systems, and then install the 4 applications. 

<img src=".\images\ContainerVsVMs.png" style="width:75%; height: 75%;">

Things are a bit different in the container model. The server is powered on and the OS boots. In the Docker world this can be Linux, or a modern version of Windows that supports the container primitives in its kernel. Similar to the VM model, the OS claims all hardware resources. On top of the OS, we install a container engine such as Docker. The container engine then takes **OS resources** such as the *process tree*, the *ﬁlesystem*, and the *network stack*, and carves them into isolated constructs called *containers*. Each container looks smells and feels just like a real OS. Inside of each *container* we run an application.

If we assume the same scenario of a single physical server needing to run 4 business applications, we’d carve the OS into 4 containers and run a single application inside each. 

<img src=".\images\ContainerVsVMs2.png" style="width:75%; height: 75%;">

At a high level, hypervisors perform **hardware virtualization** — they carve up physical hardware resources into virtual versions called VMs. On the other hand, containers perform **OS virtualization** — they carve OS resources into virtual versions called containers.

#### Virtual Machine vs. Docker

<img src=".\images\VM-vs-Docker.png" style="width:75%; height: 75%;">

### Container Operations

Since a running container resembles a Virtual Machine and there are tools facilitating various management operations on VMs, then, as expected, there are a variety of operations available to manage a container’s lifecycle - operations facilitated by the container runtimes.

**While some operations may be basic in nature and allow us to create, start, run, pause, resume, stop, restart, and remove/delete a container, other operations may be advanced and allow us to interact directly with the application in the running container or further customize the behavior of the running container.**

**Advanced operations may allow us to:**
* run a container in the background
* run a container in daemon mode
* list and sort containers
* interact with the container’s environment by:
** inspecting a running container’s status
** listing processes inside a container
** opening an interactive terminal in the container’s environment
** limiting resources utilization such as CPU and memory
** setting access privileges.

For troubleshooting purposes, we can also view container events and logs to help determine the cause for any unexpected behavior. Based on the complexity of a container runtime, various operations may or may not be supported.

### Running containers

The ﬁrst thing I always do when I log on to a Docker host is check that Docker is running.

```shell
$ docker version
```

As long as you get a response back in the Client and Server you should be good to go. If you get an error code in the Server section, there’s a good chance that the Docker daemon (server) isn’t running, or that your user account doesn’t have permission to access it. If you’re on a Linux machine and your user account doesn’t have permission to access the daemon, you need to make sure it’s a member of the local docker Unix group. If it isn’t, you can add it with **`sudo usermod -aG docker $USER`** and then you’ll have to logout and log back in to your shell for the changes to take effect.

If your user account is already a member of the local docker group, the problem might be that the Docker daemon isn’t running. To check the status of the Docker daemon, run one of the following commands depending on your Docker host’s operating system.

Linux systems not using Systemd.

```shell
$ service docker status
$ systemctl is-active docker
```

If the Docker daemon is running, you’re ﬁne to continue.

### Starting a simple container

The simplest way to start a container is with the docker container run command. The following command starts a simple container that will run a containerized version of Ubuntu Linux.

```shell
$ docker container run -it ubuntu:latest /bin/bash
```

docker container run tells Docker to run a new container. The -it ﬂags make the container interactive and attach it to your terminal. ubuntu:latest tell Docker which image to start the container from. Finally, /bin/bash is the respective applications each container will run. When you hit Return, the Docker client packaged up the command and POSTed it to the API server running on the Docker daemon. The Docker daemon accepted the command and searched the Docker host’s local image repository to see if it already had a copy of the requested image. In the examples cited, it didn’t, so it went to Docker Hub to see if it could ﬁnd it there. It found it, pulled it locally, and stored it in its local cache.

> **Note:** In a standard, out-of-the-box Linux installation, the Docker daemon implements the Docker Remote API on a local IPC/Unix socket at /var/run/docker.sock. It’s possible to conﬁgure the Docker daemon to listen on the network. The default non-TLS network port for Docker is 2375, the default TLS port is 2376.

Once the image was pulled, the daemon instructed containerd and runc to create and start the container. If you’re following along, your terminal is now attached to the container — look closely and you’ll see that your shell prompt has changed. In the Linux example cited, the shell prompt has changed to root@50949b614477:/#.

The long number after the @ is the ﬁrst 12 characters of the container’s unique ID. Try executing some basic commands inside of the container. You might notice that some of them don’t work. This is because the images are optimized to be lightweight. As a result, they don’t have all of the normal commands and packages installed. The following example shows a couple of commands — one succeeds and the other one fails.

```shell
root@50949b614477:/# ls -l
root@50949b614477:/# ping 
```

As you can see, the ping utility is not included as part of the oﬃcial Ubuntu image.

### Container processes

When we started the Ubuntu container in the previous section, we told it to run the Bash shell (/bin/bash). This makes the Bash shell the **one and only process running inside of the container**. You can see this by running ps -elf from inside the container.

```shell
root@50949b614477:/# ps -elf
```

The ﬁrst process in the list, with PID 1, is the Bash shell we told the container to run. The second process is the ps -elf command we ran to produce the list. This is a short-lived process that exits as soon as the output is displayed. Long story short, this container is running a single process — /bin/bash.

**Note:** Windows containers are slightly different and tend to run quite a few background processes.

If you’re logged on to the container and type exit, you’ll terminate the Bash process and the container will exit (terminate). This is because a container cannot exist without its designated main process. This is true of Linux and Windows containers — **killing the main process in the container will kill the container**.

Press **Ctrl-PQ** to exit the container without terminating its main process. Doing this will place you back in the shell of your Docker host and leave the container running in the background. You can use the `docker container ls` command to view the list of running containers on your system.

```shell
$ docker container ls
```

It’s important to understand that this container is still running and you can re-attach your terminal to it with the `docker container exec` command.

```shell
$ docker container exec -it 50949b614477 bash
```

As you can see, the shell prompt has changed back to the container. If you run the ps -elf command again you will now see **two** Bash processes. This is because the docker container exec command created a new Bash or PowerShell process and attached to that. This means typing exit in this shell will not terminate the container, because the original Bash or PowerShell process will continue running.

Type exit to leave the container and verify it’s still running with a docker container ls. It will still be running. If you are following along with the examples, you should stop and delete the container with the following two commands (you will need to substitute the ID of your container).

```shell
$ docker container stop 50949b614477

$ docker container rm 50949b614477
```

The containers started in the previous examples will no longer be present on your system.

### Container lifecycle

In this section, we’ll look at the lifecycle of a container — from birth, through work and vacations, to eventual death. We’ve already seen how to start containers with the docker container run command. Let’s start another one so we can walk it through its entire lifecycle. The following examples will be from a Linux Docker host running an Ubuntu container.

```shell
$ docker container run --name percy -it ubuntu:latest /bin/bash
```

That's the container created, and we named it “percy” for persistent. Now let’s put it to work by writing some data to it. The following procedure writes some text to a new ﬁle in the /tmp directory and veriﬁes the operation succeeded. Be sure to run these commands from within the container you just started.

```shell
root@9cb2d2fd1d65:/# cd tmp
root@9cb2d2fd1d65:/tmp# ls -l
root@9cb2d2fd1d65:/tmp# echo "Sunderland is the greatest football team in the world" > newfile
root@9cb2d2fd1d65:/tmp# ls -l
root@9cb2d2fd1d65:/tmp# cat newfile
```

Now use the docker container stop command to stop the container and put in on *vacation*.

```shell
$ docker container stop percy
```

You can use the container’s name or ID with the docker container stop command. The format is **`docker container stop <container-id or container-name>`**. Now run a docker container ls command to list all running containers.

```shell
$ docker container ls
```

The container is not listed in the output above because it’s in the stopped state. Run the same command again, only this time add the -a ﬂag to show all containers, including those that are stopped.

```shell
$ docker container ls -a
```

Now we can see the container showing as Exited (0). Stopping a container is like stopping a virtual machine. Although it’s not currently running, its entire conﬁguration and contents still exist on the local ﬁlesystem of the Docker host. This means it can be restarted at any time. Let’s use the docker container start command to bring it back from vacation.

```shell
$ docker container start percy
$ docker container ls
```

The stopped container is now restarted. Time to verify that the ﬁle we created earlier still exists. Connect to the restarted container with the docker container exec command.

```shell
$ docker container exec -it percy bash
```

Your shell prompt will change to show that you are now operating within the namespace of the container. Verify the ﬁle you created earlier is still there and contains the data you wrote to it.

```shell
root@9cb2d2fd1d65:/# cd tmp
root@9cb2d2fd1d65:/# ls -l
root@9cb2d2fd1d65:/# cat newfile
```

As if by magic, the ﬁle you created is still there and the data it contains is exactly how you left it. This proves that stopping a container does not destroy the container or the data inside of it. While this example illustrates the persistent nature of containers, it’s important you understand two things:

1. The data created in this example is stored on the Docker hosts local ﬁlesystem. If the Docker host fails, the data will be lost.
2. Containers are designed to be immutable objects and it’s not a good practice to write data to them.

For these reasons, Docker provides *volumes* that exist separately from the container, but can be mounted into the container at runtime.

Now let’s kill the container and delete it from the system. You can delete a *running* container with a single command, by passing the -f ﬂag to `docker container rm`. However, it’s considered a best practice to take the two-step approach of stopping the container ﬁrst and then deleting it. This gives the application/process running in the container a ﬁghting chance of stopping cleanly. 

```shell
$ docker container stop percy
$ docker container rm percy
$ docker container ls -a
```

The container is now deleted. To summarize the lifecycle of a container. You can stop, start, pause, and restart a container as many times as you want.

### Stopping containers gracefully

When you kill a running container with `docker container rm <container> -f`, the container is killed without warning. `docker container stop` sends a **SIGTERM** signal to the main application process inside the container (PID 1). As we said, this gives the process a chance to clean things up and gracefully shut itself down. If it doesn’t exit within 10 seconds, it will receive a **SIGKILL**. This is effectively the bullet to the head. But hey, it got 10 seconds to sort itself out ﬁrst. `docker container rm <container> -f` doesn’t bother asking nicely with a **SIGTERM**, it goes straight to the **SIGKILL**.

### Self-healing containers with restart policies

It’s often a good idea to run containers with a *restart policy*. This is a form of self-healing that enables Docker to automatically restart them after certain events or failures have occurred. Restart policies are applied per-container, and can be conﬁgured imperatively on the command line as part of docker-container run commands, or declaratively in YAML ﬁles for use with higher-level tools such as Docker Swarm, Docker Compose, and Kubernetes. At the time of writing, the following restart policies exist:

- always
- unless-stopped
- on-failed

The **always** policy is the simplest. It always restarts a stopped container unless it has been explicitly stopped, such as via a docker container stop command. An easy way to demonstrate this is to start a new interactive container, with the --restart always policy, and tell it to run a shell process. When the container starts you will be attached to its shell. Typing exit from the shell will kill the container’s PID 1 process and kill the container. However, Docker will automatically restart it because it has the --restart always policy. If you issue a docker container ls command, you’ll see that the container’s uptime is less than the time since it was created. Let’s put it to the test.

```shell
$ docker container run --name neversaydie -it --restart always alpine sh
```

Wait a few seconds before typing the exit command. Once you’ve exited the container and are back at your normal shell prompt, check the container’s status.

```shell
$ docker container ls
```

See how the container was created 35 seconds ago, but has only been up for 9 seconds. This is because the exit command killed it and Docker restarted it. Be aware that Docker has restarted the same container and not created a new one. 

In fact, if you inspect it with docker container inspect you can see the restartCount has been incremented. An interesting feature of the --restart always policy is that if you stop a container with docker container stop and the restart the Docker daemon, the container will be restarted. 

To be clear, you start a new container with the --restart always policy and then stop it with the docker container stop command. At this point the container is in the Stopped (Exited) state. However, if you restart the Docker daemon, the container will be automatically restarted when the daemon comes back up. You need to be aware of this. 

The main difference between the **always** and **unless-stopped** policies is that containers with the --restart unless-stopped policy will not be restarted when the daemon restarts if they were in the Stopped (Exited) state. That might be a confusing sentence, so let’s walk through an example.

1. Create the two new containers

```shell
$ docker container run -d --name always --restart always alpine sleep 1d
$ docker container run -d --name unless-stopped --restart unless-stopped alpine sleep 1d
$ docker container ls
```

2. Stop both containers

```shell
$ docker container stop always unless-stopped
$ docker container ls -a
```

3. Restart Docker.

The process for restarting Docker is different on different Operating Systems. This example shows how to stop Docker on Linux hosts running systemd. To restart Docker on Windows Server 2016 use restart-service Docker.

```shell
$ systemlctl restart docker
$ docker container ls -a
```

Notice that the “always” container (started with the --restart always policy) has been restarted, but the “unless-stopped” container has not. The **on-failure** policy will restart a container if it exits with a non-zero exit code. It will also restart containers when the Docker daemon restarts, even containers that were in the stopped state. 

### Inspecting containers

When building a Docker image, you can embed an instruction that lists the default app for any containers that use the image. You can see this for any image by running a `docker image inspect`.

```shell
$ docker image inspect nigelpoulton/pluralsight-docker-ci
```

The output is snipped to make it easier to ﬁnd the information we’re interested in. The entries after Cmd show the command/app that the container will run unless you override it with a different one when you launch the container with docker container run. Sometimes the default app is listed as Entrypoint instead of Cmd. It’s common to build images with default commands like this, as it makes starting containers easier. It also forces a default behavior and is a form of self documentation — i.e. you can *inspect* the image and know what app it’s designed to run.

### Container Size on Disk

The user can use the **`docker ps -s`** command to view the approximate size of a running container. Two different columns which are related to the size of the container are:

```shell
$ docker ps -s
```

- **Size:** the amount of data (on disk) that is used for the writable layer of each container.
- **Virtual size:** the amount of data used for the read-only image data used by the container plus the container’s writable layer size.

The number of ways a container can take up disk space:

- Used for log files if the user uses the json-file logging driver.
- Used for volumes and bind mounts which are used by the container.
- Used for the container’s configuration files, which are typically small.
- Used for memory which are written to disk if swapping is enabled.

### Tidying up

Let’s look at the simplest and quickest way to get rid of **every running container** on your Docker host. Be warned though, the procedure will forcibly destroy **all** containers without giving them a chance to clean up. **This should never be performed on production systems or systems running important containers.** Run the following command from the shell of your Docker host to delete **all** containers.

```shell
$ docker container rm $(docker container ls -aq) -f
```

 We already know the docker container rm command deletes containers. Passing it $(docker container ls -aq) as an argument, effectively passes it the ID of every container on the system. The -f ﬂag forces the operation so that even containers in the running state will be destroyed. Net result… all containers, running or stopped, will be destroyed and removed from the system.
