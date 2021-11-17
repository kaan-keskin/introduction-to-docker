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
