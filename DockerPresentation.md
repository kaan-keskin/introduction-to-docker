

**2: Docker**

**Docker**

Docker is software that runs on Linux and Windows. It creates, manages, and can even orchestrate containers. The software is currently built from various tools from the *Moby* open-source project. Docker, Inc. is the company that created the technology and continues to create technologies and solutions that make it easier to get the code on your laptop running in the cloud.


**The Docker technology**

When most people talk about Docker, they’re referring to the technology that runs containers. However, there are at least three things to be aware of when referring to Docker as a technology:

1. The runtime

2. The daemon (a.k.a. engine)

3. The orchestrator

<img src=".\images\TheDocker.png" style="width:75%; height: 75%;">


The runtime operates at the lowest level and is responsible for starting and stopping containers (this includes building all of the OS constructs such as namespaces and cgroups). Docker implements a tiered runtime architecture with high-level and low-level runtimes that work together. The low-level runtime is called runc and is the reference implementation of Open Containers Initiative (OCI) runtime-spec. Its job is to interface with the underlying OS and start and stop containers. Every running container on a Docker node has a runc instance managing it. The higher-level runtime is called containerd. containerd does a lot more than runc. It manages the entire lifecycle of a container, including pulling images, creating network interfaces, and managing lower-level runc instances. containerd is pronounced “container-dee’ and is a graduated CNCF project used by Docker and Kubernetes as a container runtime.


A typical Docker installation has a single containerd process (docker-containerd) controlling the runc (docker-runc) instances associated with each running container. The Docker daemon (dockerd) sits above containerd and performs higher-level tasks such as; exposing the Docker remote API, managing images, managing volumes, managing networks, and more…

A major job of the Docker daemon is to provide an easy-to-use standard interface that abstracts the lower levels. Docker also has native support for managing clusters of nodes running Docker. These clusters are called swarms and the native technology is called Docker Swarm. Docker Swarm is easy-to-use and many companies are using it in real-world production. However, most people are choosing to use Kubernetes instead of Docker Swarm.

**The Open Container Initiative (OCI)**

The OCI is a governance council responsible for standardizing the low-level fundamental components of container infrastructure. In particular it focusses on *image format* and *container runtime* (don’t worry if you’re not comfortable with these terms yet, we’ll cover them in the book). It’s also true that no discussion of the OCI is complete without mentioning a bit of history. And as with all accounts of history, the version you get depends on who’s doing the talking.

From day one, use of Docker grew like crazy. More and more people used it in more and more ways for more and more things. So, it was inevitable that some parties would get frustrated. This is normal and healthy.

This put the container ecosystem in an awkward position with two competing standards. Getting back to the story, this threatened to fracture the ecosystem and present users and customers with a dilemma. While competition is usually a good thing, *competing standards* is usually not. They cause confusion and slowdown user adoption. Not good for anybody. With this in mind, everybody did their best to act like adults and came together to form the OCI — a lightweight agile council to govern container standards.
At the time of writing, the OCI has published two specifications (standards) -
• The image-spec
• The runtime-spec
An analogy that’s often used when referring to these two standards is rail tracks. these two standards are like agreeing on standard sizes and properties of rail tracks, leaving everyone else free to build better trains, better carriages, better signalling systems, better stations… all safe in the knowledge that they’ll work on the standardized tracks. Nobody wants two competing standards for rail track sizes! It’s fair to say that the two OCI speciﬁcations have had a major impact on the architecture and design of the core Docker product. As of Docker 1.11, the Docker Engine architecture conforms to the OCI runtime spec. The OCI is organized under the auspices of the Linux Foundation.



**3: Installing Docker**

There are lots of ways and places to install Docker. There’s Windows, Mac, and Linux. You can install in the cloud, on premises, and on your laptop. And there are manual installs, scripted installs, wizard-based installs.


• Docker Desktop installs on

**–** Windows 10

**–** Mac

• Server installs on

**–** Linux

**–** Windows Server 2019


**Docker Desktop**

Docker Desktop is a packaged product from Docker, Inc. It runs on 64-bit versions of Windows 10 and Mac, and it’s easy to download and install. Once the installation is complete, you have a single-engine Docker environment that is great for development purposes. It includes Docker Compose and you can choose to enable a single-node Kubernetes cluster.

Docker Desktop on Windows 10 can run native Windows containers as well as Linux containers. Docker Desktop on Mac can only run Linux containers.

**Windows pre-reqs**

Docker Desktop on Windows requires all of the following:

• 64-bit version of Windows 10 Pro/Enterprise/Education (does not work with Home edition)
• Hardware virtualization support must be enabled in your system’s BIOS
• The *Hyper-V* and *Containers* features must be enabled in Windows
The installer can enable the Hyper-V and Containers features, but it’s your responsibility to enable hardware virtualization in your BIOS (be very careful changing anything in your system’s BIOS).


**Installing Docker Desktop on Windows 10**

Perform a google search for “install Docker Desktop”. This will take you to the relevant download page where you can download the installer and follow the instructions. It’s that simple!

Once it’s up and running you can open a PowerShell prompt and type some simple docker commands.

`$ docker version`


Notice the output is showing OS/Arch: linux/amd64 for the **Server** component. This is because a default installation assumes you’ll be working with Linux containers. It does this by running the Docker daemon inside of a lightweight Linux Hyper-V VM.

`C:\> docker version`

You can now run and manage Windows containers (containers running Windows applications).

**Installing Docker Desktop on Mac**

Download the installer and follow the step-by-step instructions.

Once the installation is complete you may have to manually start Docker Desktop from the MacOS Launchpad.

Open a terminal window and run some regular Docker commands. Try the following.

`$ docker version`

Notice that the OS/Arch: for the **Server** component is showing as linux/amd64. This is because the daemon is running inside of the Linux VM we mentioned earlier. The **Client** component is a native Mac application and runs directly on the Mac OS Darwin kernel (OS/Arch: darwin/amd64).


**Installing Docker on Linux**

There are lots of ways to install Docker on Linux and most of them are easy. 

1. Update the apt package index.

`$ sudo apt-get update`


2. Install Docker from the oﬃcial repo.

`$ sudo apt-get install docker.io`


Docker is now installed and you can test by running some commands.

`$ sudo docker --version`

`$ sudo docker info`



**Installing Docker on Windows Server 2019**

Most of the public cloud platforms offer off-the-shelf copies of Windows Server 2019 with Docker pre- installed. Simply choose one of these – such as Microsoft Windows Server 2019 Base with Containers - ami-0b809eef92577a4f1 on AWS – and you’re good to go.

1. Install the Docker Provider

`PS C:\> Install-Module DockerMsftProvider -Force`

2. Install Docker

`PS C:\> Install-Package Docker -ProviderName DockerMsftProvider -Force`

Contains Docker EE for use with Windows Server.

3. Restart your machine

Congratulations, Docker is now installed and conﬁgured to automatically start when the system boots. Run some commands to verify Docker is working.

`PS C:\> docker version`

Docker is now installed and you are ready to start using Windows containers.

**Play with Docker**

Play with Docker (PWD) provides a free-to-use fully functional Docker playground that lasts for 4 hours. You 
can add multiple nodes and even cluster them in a swarm. 
Sometimes performance can be slow, but for a free-to-use service it is excellent!


**Part 2: The technical stuff**

**5: The Docker Engine**

The *Docker engine* is the core software that runs and manages containers. We often refer to it simply as *Docker*. If you know a thing or two about VMware, it might be useful to think of it as being like ESXi. The Docker engine is modular in design and built from many small specialised tools. Where possible, these are based on open standards such as those maintained by the Open Container Initiative (OCI).

In many ways, the Docker Engine is like a car engine — both are modular and created by connecting many small specialized parts:

• A car engine is made from many specialized parts that work together to make a car drive — intake manifolds, throttle body, cylinders, spark plugs, exhaust manifolds etc.

• The Docker Engine is made from many specialized tools that work together to create and run containers
— APIs, execution driver, runtimes, shims etc.

At the time of writing, the major components that make up the Docker engine are; the *Docker daemon*, *containerd*, *runc*, and various plugins such as networking and storage. Together, these create and run containers.

<img src=".\images\DockerEngine.png" style="width:75%; height: 75%;">

When Docker was ﬁrst released, the Docker engine had two major components:

• The Docker daemon (hereafter referred to as just “the daemon”)
• LXC

The Docker daemon was a monolithic binary. It contained all of the code for the Docker client, the Docker API, the container runtime, image builds, and **much** more.

LXC provided the daemon with access to the fundamental building-blocks of containers that existed in the Linux kernel. Things like *namespaces* and *control groups (cgroups)*.

<img src=".\images\DockerEngineLXC.png" style="width:75%; height: 75%;">


**Getting rid of LXC**

The reliance on LXC was an issue from the start. First up, LXC is Linux-speciﬁc.This was a problem for a project that had aspirations of being multi-platform. Second up, being reliant on an external tool for something so core to the project was a huge risk that could hinder development. As a result, Docker. Inc. developed their own tool called *libcontainer* as a replacement for LXC. The goal of *libcontainer* was to be a platform-agnostic tool that provided Docker with access to the fundamental container building-blocks that exist in the host kernel.

Libcontainer replaced LXC as the default *execution driver* in Docker 0.9.

**Getting rid of the monolithic Docker daemon**

Over time, the monolithic nature of the Docker daemon became more and more problematic:
1. It’s hard to innovate on
2. It got slower
3. It wasn’t what the ecosystem wanted
Docker, Inc. was aware of these challanges and began a huge effort to break apart the monolithic daemon and modularize it. The aim of this work was to break out as much of the functionality as possible from the daemon, and re-implement it in smaller specialized tools. These specialized tools can be swapped out, as well as easily re-used by third parties to build other tools. This plan follows the tried-and-tested Unix philosophy of building small specialized tools that can be pieced together into larger tools. This work of breaking apart and re-factoring the Docker engine has seen **all of the container execution and container runtime code entirely removed from the daemon and refactored into small, specialized tools**.

<img src=".\images\DockerEngineDaemon.png" style="width:75%; height: 75%;">


**The inﬂuence of the Open Container Initiative (OCI)**

While Docker, Inc. was breaking the daemon apart and refactoring code, the OCI was in the process of deﬁning two container-related speciﬁcations (a.k.a standards):
1. Image spec
2. Container spec

As of Docker 1.11 (early 2016), the Docker engine implements the OCI speciﬁcations as closely as possible. For example, the Docker daemon no longer contains any container runtime code — all container runtime code is implemented in a separate OCI-compliant layer. By default, Docker uses *runc* for this. runc is the *reference implementation* of the OCI container-runtime-spec.
As well as this, the *containerd* component of the Docker Engine makes sure Docker images are presented to *runc* as valid OCI bundles.



**runc**

As previously mentioned, *runc* is the reference implementation of the OCI container-runtime-spec. If you strip everything else away, runc is a small, lightweight CLI wrapper for libcontainer (remember that libcontainer originally replaced LXC as the interface layer with the host OS in the early Docker architecture). runc has a single purpose in life — create containers. And it’s damn good at it. And fast! But as it’s a CLI wrapper, it’s effectively a standalone container runtime tool. This means you can download and build the binary, and you’ll have everything you need to build and play with runc (OCI) containers. But it’s bare bones and very low-level, meaning you’ll have none of the richness that you get with the full-blown Docker engine.

**containerd**

As part of the effort to strip functionality out of the Docker daemon, all of the container execution logic was ripped out and refactored into a new tool called containerd (pronounced container-dee). Its sole purpose in life was to manage container lifecycle operations — start | stop | pause | rm....
containerd is available as a daemon for Linux and Windows, and Docker has been using it on Linux since the 1.11 release. In the Docker engine stack, containerd sits between the daemon and runc at the OCI layer. As previously stated, containerd was originally intended to be small, lightweight, and designed for a single task in life — container lifecycle operations. However, over time it has branched out and taken on more functionality. Things like image pulls, volumes and networks.

**Starting a new container (example)**

Now that we have a view of the big picture, and some of the history, let’s walk through the process of creating a new container. The most common way of starting containers is using the Docker CLI. The following docker container run command will start a simple new container based on the alpine:latest image.

`$ docker container run --name ctr1 -it alpine:latest sh`

When you type commands like this into the Docker CLI, the Docker client converts them into the appropriate API payload and POSTs them to the API endpoint exposed by the Docker daemon. The API is implemented in the daemon and can be exposed over a local socket or the network. On Linux the socket is /var/run/docker.sock and on Windows it’s \pipe\docker\_engine. Once the daemon receives the command to create a new container, it makes a call to containerd. Remember that the daemon no-longer contains any code to create containers! Despite its name, *containerd* cannot actually create containers. It uses *runc* to do that. It converts the required Docker image into an OCI bundle and tells runc to use this to create a new container. runc interfaces with the OS kernel to pull together all of the constructs necessary to create a container (namespaces, cgroups etc.). The container process is started as a child-process of runc, and as soon as it is started runc will exit.

<img src=".\images\DockerEngineShim.png" style="width:75%; height: 75%;">

**One huge beneﬁt of this model**

Having all of the logic and code to start and manage containers removed from the daemon means that the entire container runtime is decoupled from the Docker daemon. We sometimes call this “daemonless containers”, and it makes it possible to perform maintenance and upgrades on the Docker daemon without impacting running containers! In the old model, where all of container runtime logic was implemented in the daemon, starting and stopping the daemon would kill all running containers on the host. This was a huge problem in production environments. Every daemon upgrade would kill all containers on that host — not good! Fortunately, this is no longer a problem.

**What’s this shim all about?**

Some of the diagrams in the chapter have shown a shim component. The shim is integral to the implementation of daemonless containers (what we just mentioned about decoupling running containers from the daemon for things like daemon upgrades). We mentioned earlier that *containerd* uses runc to create new containers. In fact, it forks a new instance of runc for every container it creates. However, once each container is created, the parent runc process exits. This means we can run hundreds of containers without having to run hundreds of runc instances. Once a container’s parent runc process exits, the associated containerd-shim process becomes the container’s parent. Some of the responsibilities the shim performs as a container’s parent include:

• Keeping any STDIN and STDOUT streams open so that when the daemon is restarted, the container doesn’t terminate due to pipes being closed etc.

• Reports the container’s exit status back to the daemon.

**How it’s implemented on Linux**

On a Linux system, the components we’ve discussed are implemented as separate binaries as follows:

• dockerd (the Docker daemon)

• docker-containerd (containerd)

• docker-containerd-shim (shim)

• docker-runc (runc)

You can see all of these on a Linux system by running a ps command on the Docker host. Obviously, some of them will only be present when the system has running containers.

**What’s the point of the daemon**

At the time of writing, some of the major functionality that still exists in the daemon includes; image management, image builds, the REST API, authentication, security, core networking, and orchestration.


**Securing client and daemon communication**

Let’s ﬁnish the chapter by looking at how to secure the daemon over the network.Docker implements a client-server model.

• The client component implements the CLI

• The server (daemon) component implements the functionality, including the public-facing REST API

The client is called docker (docker.exe on Windows) and the daemon is called dockerd (dockerd.exe on Windows). A default installation puts them on the same host and conﬁgures them to communicate over a local IPC socket:

• /var/run/docker.sock on Linux

• ./pipe/docker\_engine on Windows

It’s also possible to conﬁgure them to communicate over the network. By default, network communication occur over an unsecured HTTP socket on port 2375/tcp.

<img src=".\images\DockerSecurity.png" style="width:75%; height: 75%;">

An insecure conﬁguration like this might be suitable for labs, but it’s unacceptable for anything else. TLS to the rescue! Docker lets you force the client and daemon to only accept network connections that are secured with TLS. This is recommended for production environments, even if all traffic is traversing trusted internal networks. You can secure both the client and the daemon. Securing the client forces the client to only connect to Docker daemons with certiﬁcates signed by a trusted Certiﬁcate Authority (CA). Securing the daemon forces the daemon to only accept connections from clients presenting certiﬁcates from a trusted CA. A combination of both modes offers the most security.



**7: Containers**

A container is the runtime instance of an image. In the same way that you can start a virtual machine (VM) from a virtual machine template, you start one or more containers from a single image. The big difference between a VM and a container is that containers are faster and more lightweight — instead of running a full-blown OS like a VM, containers share the OS/kernel with the host they’re running on. It’s also common for containers to be based on minimalist images that only include software and dependencies required by the application.

<img src=".\images\DockerContainers.png" style="width:75%; height: 75%;">

The simplest way to start a container is with the docker container run command. The command can take a lot of arguments, but in its most basic form you tell it an image to use and a app to run: docker container run image app. The following command will start an Ubuntu Linux container running the Bash shell as its app.

`$ docker container run -it ubuntu /bin/bash`


In each of the examples, the -it ﬂags will connect your current terminal window to the container’s shell. Containers run until the app they are executing exits. In the previous examples, the Linux container will exit when the Bash shell exits, and the Windows container will exit when the PowerShell process terminates.

A simple way to demonstrate this is to start a new container and tell it to run the sleep command for 10 seconds.

`$ docker container run -it alpine:latest sleep 10`


You can manually stop a running container with the docker container stop command. You can then restart it with docker container start. To get rid of a container forever, you have to explicitly delete it with docker container rm.

**Containers vs VMs**

Containers and VMs both need a host to run on. This can be anything from your laptop, a bare metal server in your data center, all the way up to an instance in the public cloud. In fact, many cloud services now offer the ability to run containers on ephemeral serverless back-ends. Don’t worry if that sounds like techno-babble, it just means that the back-end is so highly virtualized that the concept of a host or node no longer has any meaning — your container simply runs, and you don’t need to care about the *how* or *where*.

Anyway… let’s assume a requirement where your business has a single physical server that needs to run 4 business applications. In the VM model, the physical server is powered on and the hypervisor boots (we’re skipping the BIOS and bootloader code etc.). Once booted, the hypervisor lays claim to all physical resources on the system such as CPU, RAM, storage, and NICs. It then carves these hardware resources into virtual versions that look smell and feel exactly like the real thing. It then packages them into a software construct called a virtual machine (VM). We take those VMs and install an operating system and application on each one. Assuming the scenario of a single physical server that needs to run 4 business applications, we’d create 4 VMs, install 4 operating systems, and then install the 4 applications. 

<img src=".\images\ContainerVsVMs.png" style="width:75%; height: 75%;">

Things are a bit different in the container model. The server is powered on and the OS boots. In the Docker world this can be Linux, or a modern version of Windows that supports the container primitives in its kernel. Similar to the VM model, the OS claims all hardware resources. On top of the OS, we install a container engine such as Docker. The container engine then takes **OS resources** such as the *process tree*, the *ﬁlesystem*, and the *network stack*, and carves them into isolated constructs called *containers*. Each container looks smells and feels just like a real OS. Inside of each *container* we run an application.

If we assume the same scenario of a single physical server needing to run 4 business applications, we’d carve the OS into 4 containers and run a single application inside each. 

<img src=".\images\ContainerVsVMs2.png" style="width:75%; height: 75%;">

At a high level, hypervisors perform **hardware virtualization** — they carve up physical hardware resources into virtual versions called VMs. On the other hand, containers perform **OS virtualization** — they carve OS resources into virtual versions called containers.


**Running containers**


The ﬁrst thing I always do when I log on to a Docker host is check that Docker is running.

```sh
$ docker version
```

As long as you get a response back in the Client and Server you should be good to go. If you get an error code in the Server section, there’s a good chance that the Docker daemon (server) isn’t running, or that your user account doesn’t have permission to access it. If you’re on a Linux machine and your user account doesn’t have permission to access the daemon, you need to make sure it’s a member of the local docker Unix group. If it isn’t, you can add it with `usermod -aG docker <user>` and then you’ll have to logout and log back in to your shell for the changes to take effect.

If your user account is already a member of the local docker group, the problem might be that the Docker daemon isn’t running. To check the status of the Docker daemon, run one of the following commands depending on your Docker host’s operating system.

Linux systems not using Systemd.


```sh
$ service docker status
$ systemctl is-active docker
```

If the Docker daemon is running, you’re ﬁne to continue.

**Starting a simple container**

The simplest way to start a container is with the docker container run command. The following command starts a simple container that will run a containerized version of Ubuntu Linux.

```sh
$ docker container run -it ubuntu:latest /bin/bash
```

docker container run tells Docker to run a new container. The -it ﬂags make the container interactive and attach it to your terminal. ubuntu:latest tell Docker which image to start the container from. Finally, /bin/bash is the respective applications each container will run. When you hit Return, the Docker client packaged up the command and POSTed it to the API server running on the Docker daemon. The Docker daemon accepted the command and searched the Docker host’s local image repository to see if it already had a copy of the requested image. In the examples cited, it didn’t, so it went to Docker Hub to see if it could ﬁnd it there. It found it, pulled it locally, and stored it in its local cache.

**Note:** In a standard, out-of-the-box Linux installation, the Docker daemon implements the Docker Remote API on a local IPC/Unix socket at /var/run/docker.sock. It’s possible to conﬁgure the Docker daemon to listen on the network. The default non-TLS network port for Docker is 2375, the default TLS port is 2376.

Once the image was pulled, the daemon instructed containerd and runc to create and start the container. If you’re following along, your terminal is now attached to the container — look closely and you’ll see that your shell prompt has changed. In the Linux example cited, the shell prompt has changed to root@50949b614477:/#.

The long number after the @ is the ﬁrst 12 characters of the container’s unique ID. Try executing some basic commands inside of the container. You might notice that some of them don’t work. This is because the images are optimized to be lightweight. As a result, they don’t have all of the normal commands and packages installed. The following example shows a couple of commands — one succeeds and the other one fails.

```sh
root@50949b614477:/# ls -l

total 64

lrwxrwxrwx

root@50949b614477:/# ping 

bash: ping: command not found
```

As you can see, the ping utility is not included as part of the oﬃcial Ubuntu image.

**Container processes**

When we started the Ubuntu container in the previous section, we told it to run the Bash shell (/bin/bash). This makes the Bash shell the **one and only process running inside of the container**. You can see this by running ps -elf from inside the container.
```sh
root@50949b614477:/# ps -elf
```

The ﬁrst process in the list, with PID 1, is the Bash shell we told the container to run. The second process is the ps -elf command we ran to produce the list. This is a short-lived process that exits as soon as the output is displayed. Long story short, this container is running a single process — /bin/bash.

**Note:** Windows containers are slightly different and tend to run quite a few background processes.

If you’re logged on to the container and type exit, you’ll terminate the Bash process and the container will exit (terminate). This is because a container cannot exist without its designated main process. This is true of Linux and Windows containers — **killing the main process in the container will kill the container**.

Press Ctrl-PQ to exit the container without terminating its main process. Doing this will place you back in the shell of your Docker host and leave the container running in the background. You can use the `docker container ls` command to view the list of running containers on your system.
```sh
$ docker container ls
```
It’s important to understand that this container is still running and you can re-attach your terminal to it with the `docker container exec` command.

```sh
$ docker container exec -it 50949b614477 bash

root@50949b614477:/#
```

As you can see, the shell prompt has changed back to the container. If you run the ps -elf command again you will now see **two** Bash processes. This is because the docker container exec command created a new Bash or PowerShell process and attached to that. This means typing exit in this shell will not terminate the container, because the original Bash or PowerShell process will continue running.

Type exit to leave the container and verify it’s still running with a docker container ls. It will still be running. If you are following along with the examples, you should stop and delete the container with the following two commands (you will need to substitute the ID of your container).
```sh
$ docker container stop 50949b614477

$ docker container rm 50949b614477
```
The containers started in the previous examples will no longer be present on your system.

**Container lifecycle**

In this section, we’ll look at the lifecycle of a container — from birth, through work and vacations, to eventual death. We’ve already seen how to start containers with the docker container run command. Let’s start another one so we can walk it through its entire lifecycle. The following examples will be from a Linux Docker host running an Ubuntu container. 
```sh
$ docker container run --name percy -it ubuntu:latest /bin/bash

root@9cb2d2fd1d65:/#
```
That's the container created, and we named it “percy” for persistent. Now let’s put it to work by writing some data to it. The following procedure writes some text to a new ﬁle in the /tmp directory and veriﬁes the operation succeeded. Be sure to run these commands from within the container you just started.

```sh
root@9cb2d2fd1d65:/# cd tmp

root@9cb2d2fd1d65:/tmp# ls -l

root@9cb2d2fd1d65:/tmp# echo "Sunderland is the greatest football team in the world" > newfile

root@9cb2d2fd1d65:/tmp# ls -l

root@9cb2d2fd1d65:/tmp# cat newfile
```

Now use the docker container stop command to stop the container and put in on *vacation*.
```sh
$ docker container stop percy
```

You can use the container’s name or ID with the docker container stop command. The format is `docker container stop <container-id or container-name> ``. Now run a docker container ls command to list all running containers.
```sh
$ docker container ls
```
The container is not listed in the output above because it’s in the stopped state. Run the same command again, only this time add the -a ﬂag to show all containers, including those that are stopped.
```sh
$ docker container ls -a
```
Now we can see the container showing as Exited (0). Stopping a container is like stopping a virtual machine. Although it’s not currently running, its entire conﬁguration and contents still exist on the local ﬁlesystem of the Docker host. This means it can be restarted at any time. Let’s use the docker container start command to bring it back from vacation.
```sh
$ docker container start percy

$ docker container ls
```
The stopped container is now restarted. Time to verify that the ﬁle we created earlier still exists. Connect to the restarted container with the docker container exec command.
```sh
$ docker container exec -it percy bash

root@9cb2d2fd1d65:/#
```
Your shell prompt will change to show that you are now operating within the namespace of the container. Verify the ﬁle you created earlier is still there and contains the data you wrote to it.
```sh
root@9cb2d2fd1d65:/# cd tmp

root@9cb2d2fd1d65:/# ls -l

root@9cb2d2fd1d65:/# cat newfile
```

As if by magic, the ﬁle you created is still there and the data it contains is exactly how you left it. This proves that stopping a container does not destroy the container or the data inside of it. While this example illustrates the persistent nature of containers, it’s important you understand two things:
1. The data created in this example is stored on the Docker hosts local ﬁlesystem. If the Docker host fails, the data will be lost.
2. Containers are designed to be immutable objects and it’s not a good practice to write data to them.

For these reasons, Docker provides *volumes* that exist separately from the container, but can be mounted into the container at runtime.

Now let’s kill the container and delete it from the system. You can delete a *running* container with a single command, by passing the -f ﬂag to `docker container rm`. However, it’s considered a best practice to take the two-step approach of stopping the container ﬁrst and then deleting it. This gives the application/process running in the container a ﬁghting chance of stopping cleanly. 

```sh
$ docker container stop percy

$ docker container rm percy

$ docker container ls -a
```

The container is now deleted. To summarize the lifecycle of a container… You can stop, start, pause, and restart a container as many times as you want.

**Stopping containers gracefully**

 When you kill a running container with `docker container rm <container> -f`, the container is killed without warning. `docker container stop` sends a **SIGTERM** signal to the main application process inside the container (PID 1). As we said, this gives the process a chance to clean things up and gracefully shut itself down. If it doesn’t exit within 10 seconds, it will receive a **SIGKILL**. This is effectively the bullet to the head. But hey, it got 10 seconds to sort itself out ﬁrst. `docker container rm <container> -f` doesn’t bother asking nicely with a **SIGTERM**, it goes straight to the **SIGKILL**.

**Self-healing containers with restart policies**

It’s often a good idea to run containers with a *restart policy*. This is a form of self-healing that enables Docker to automatically restart them after certain events or failures have occurred. Restart policies are applied per-container, and can be conﬁgured imperatively on the command line as part of docker-container run commands, or declaratively in YAML ﬁles for use with higher-level tools such as Docker Swarm, Docker Compose, and Kubernetes. At the time of writing, the following restart policies exist:

• always

• unless-stopped

• on-failed

The **always** policy is the simplest. It always restarts a stopped container unless it has been explicitly stopped, such as via a docker container stop command. An easy way to demonstrate this is to start a new interactive container, with the --restart always policy, and tell it to run a shell process. When the container starts you will be attached to its shell. Typing exit from the shell will kill the container’s PID 1 process and kill the container. However, Docker will automatically restart it because it has the --restart always policy. If you issue a docker container ls command, you’ll see that the container’s uptime is less than the time since it was created. Let’s put it to the test.

```sh
$ docker container run --name neversaydie -it --restart always alpine sh
```
Wait a few seconds before typing the exit command. Once you’ve exited the container and are back at your normal shell prompt, check the container’s status.
```sh
$ docker container ls
```
See how the container was created 35 seconds ago, but has only been up for 9 seconds. This is because the exit command killed it and Docker restarted it. Be aware that Docker has restarted the same container and not created a new one. In fact, if you inspect it with docker container inspect you can see the restartCount has been incremented. An interesting feature of the --restart always policy is that if you stop a container with docker container stop and the restart the Docker daemon, the container will be restarted. To be clear… you start a new container with the --restart always policy and then stop it with the docker container stop command. At this point the container is in the Stopped (Exited) state. However, if you restart the Docker daemon, the container will be automatically restarted when the daemon comes back up. You need to be aware of this. The main difference between the **always** and **unless-stopped** policies is that containers with the --restart unless-stopped policy will not be restarted when the daemon restarts if they were in the Stopped (Exited) state. That might be a confusing sentence, so let’s walk through an example.


1. Create the two new containers
```sh
$ docker container run -d --name always --restart always alpine sleep 1d

$ docker container run -d --name unless-stopped --restart unless-stopped alpine sleep 1d

$ docker container ls
```
2. Stop both containers
```sh
$ docker container stop always unless-stopped

$ docker container ls -a
```
3. Restart Docker.

The process for restarting Docker is different on different Operating Systems. This example shows how to stop Docker on Linux hosts running systemd. To restart Docker on Windows Server 2016 use restart-service Docker.
```sh
$ systemlctl restart docker

$ docker container ls -a
```

Notice that the “always” container (started with the --restart always policy) has been restarted, but the “unless-stopped” container has not. The **on-failure** policy will restart a container if it exits with a non-zero exit code. It will also restart containers when the Docker daemon restarts, even containers that were in the stopped state. 

**Web server example**

**Inspecting containers**

When building a Docker image, you can embed an instruction that lists the default app for any containers that use the image. You can see this for any image by running a `docker image inspect`.
```sh
$ docker image inspect nigelpoulton/pluralsight-docker-ci
```

The output is snipped to make it easier to ﬁnd the information we’re interested in. The entries after Cmd show the command/app that the container will run unless you override it with a different one when you launch the container with docker container run. Sometimes the default app is listed as Entrypoint instead of Cmd. It’s common to build images with default commands like this, as it makes starting containers easier. It also forces a default behavior and is a form of self documentation — i.e. you can *inspect* the image and know what app it’s designed to run.

**Tidying up**

Let’s look at the simplest and quiteest way to get rid of **every running container** on your Docker host. Be warned though, the procedure will forcibly destroy **all** containers without giving them a chance to clean up. **This should never be performed on production systems or systems running important containers.** Run the following command from the shell of your Docker host to delete **all** containers.
```sh
$ docker container rm $(docker container ls -aq) -f
```
 We already know the docker container rm command deletes containers. Passing it $(docker container ls -aq) as an argument, effectively passes it the ID of every container on the system. The -f ﬂag forces the operation so that even containers in the running state will be destroyed. Net result… all containers, running or stopped, will be destroyed and removed from the system.




**8: Containerizing an app**

Docker is all about taking applications and running them in containers. The process of taking an application and conﬁguring it to run as a container is called “containerizing”. In this chapter, we’ll walk through the process of containerizing a simple Linux-based web application.

Containers are all about making apps simple to **build**, **ship**, and **run**. The process of containerizing an app looks like this:

1. Start with your application code and dependencies

2. Create a *Dockerﬁle* that describes your app, its dependencies, and how to run it

3. Feed the *Dockerﬁle* into the docker image build command

4. Push the new image to a registry (optional)

5. Run container from the image

Once your app is containerized (made into a container image), you’re ready to share it and run it as a container.

<img src=".\images\Containerizing.png" style="width:75%; height: 75%;">


**Containerize a single-container app**

The rest of this chapter walks through the process of containerizing a simple Node.js web app. We’ll complete the following high-level steps:

• Clone the repo to get the app code

• Inspect the Dockerﬁle

• Containerize the app

• Run the app

• Test the app

• Look a bit closer

• Move to production with **Multi-stage Builds**

• A few best practices 
The example in this chapter is of a single-container app. The next chapter will include a slightly more complex multi-container app, and we’ll move on to an even more complicated app in the chapter on Docker Stacks.


**Getting the application code**

The application used in this example is available on GitHub at:

• https://github.com/nigelpoulton/psweb.git

Clone the sample app from GitHub.

$ git clone https://github.com/nigelpoulton/psweb.git

The clone operation creates a new directory called psweb. Change directory into psweb and list its contents.

$ cd psweb

$ ls -l

This directory contains all of the application source code, as well as subdirectories for views and unit tests. Feel free to look at the ﬁles — the app is extremely simple. We won’t be using the unit tests in this chapter. Now that we have the app code, let’s look at its Dockerﬁle.

**Inspecting the Dockerﬁle**

A **Dockerﬁle** is the starting point for creating a container image — it describes an application and tells Dockerhow to build it into an image. The directory containing the application and dependencies is referred to as the *build context*. It’s a common practice to keep your Dockerﬁle in the root directory of the *build context*. It’s also important that **Dockerﬁle** starts with a capital “**D**” and is all one word. “Dockerﬁle” and “Docker ﬁle” are not valid. Let’s look at the contents of the Dockerﬁle.
```sh
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
Do not underestimate the impact of the Dockerﬁle as a form of documentation. It’s a great document for bridging the gap between dev and ops. It also has the power to speed up on-boarding of new developers etc. This is because the ﬁle accurately describes the application and its dependencies in an easy-to-read format. You should treat it like you treat source code and check it into a version control system.

All Dockerﬁles start with the FROM instruction. This will be the base layer of the image, and the rest of the app will be added on top as additional layers. This particular application is a Linux app, so it’s important that the FROM instruction refers to a Linux-based image. Next, the Dockerﬁle creates a LABEL that speciﬁes “nigelpoulton@hotmail.com” as the maintainer of the image. Labels are simple key-value pairs and are an excellent way of adding custom metadata to an image. It’s considered a best practice to list a maintainer of an image so that other potential users have a point of contact when working with it. The RUN apk add --update nodejs nodejs-npm instruction uses the Alpine apk package manager to install nodejs and nodejs-npm into the image. It creates a new image layer directly above the Alpine base layer, and installs the packages in this layer. The COPY . /src instruction creates another new layer and copies in the application and dependency ﬁles from the *build context*.

Next, the Dockerﬁle uses the WORKDIR instruction to set the working directory inside the image ﬁlesystem for the rest of the instructions in the ﬁle. This instruction does not create a new image layer. Then the RUN npm install instruction creates a new layer and uses npm to install application dependencies listed in the package.json ﬁle in the build context. It runs within the context of the WORKDIR set in the previous instruction, and installs the dependencies into the newly created layer. 

<img src=".\images\Dockerfile.png" style="width:75%; height: 75%;">

The application exposes a web service on TCP port 8080, so the Dockerﬁle documents this with the EXPOSE 8080 instruction. This is added as image metadata and not an image layer. Finally, the ENTRYPOINT instruction is used to set the main application that the image (container) should run. This is also added as metadata and not an image layer.

**Containerize the app/build the image**

Now that we understand how it works, let’s build it! The following command will build a new image called web:latest. The period (.) at the end of the command tells Docker to use the shell’s current working directory as the *build context*. Be sure to include the trailing period (.) and be sure to run the command from the psweb directory that contains the Dockerﬁle and application code.
```sh
$ docker image build -t web:latest .
```
Check that the image exists in your Docker host’s local repository.
```sh
$ docker image ls
```
You can use the docker image inspect web:latest command to verify the conﬁguration of the image. It will list all of the settings that were conﬁgured from the Dockerﬁle. Look out for the list of image layers and the Entrypoint command.

**Pushing images**

Once you’ve created an image, it’s a good idea to store it in an image registry to keep it safe and make it available to others. Docker Hub is the most common public image registry, and it’s the default push location for docker image push commands. In order to push an image to Docker Hub, you need to login with your Docker ID. You also need to tag the image appropriately.

Let’s log in to Docker Hub and push the newly created image. In the following example’s you will need to substitute my Docker ID with your own. 
```sh
$ docker login
```

Before you can push an image, you need to tag it in a special way. This is because Docker needs all of the followinginformation when pushing an image:

• Registry

• Repository

• Tag


Docker is opinionated, so by default it pushes images to Docker Hub. You can push to other registries, but you have to explicitly set the registry URL as part of the `docker image push` command. The previous docker image ls output shows the image is tagged as web:latest. This translates to a repository called web and an image tagged as latest. As a result, docker image push will try and push the image to a repository called web on Docker Hub. However, don’t have access to the web repository, all of my images live in the second-level namespace.
```sh
$ docker image tag web:latest repo_name/web:latest
```
The format of the command is `docker image tag <current-tag> <new-tag>` and it adds an additional tag, it does not overwrite the original.
```sh
$ docker image ls
```
Now we can push it to Docker Hub. You can’t push images to repos in my Docker Hub namespace, you will have to tag the image to use your own.
```sh
$ docker image push repo_name/web:latest
```

<img src=".\images\DockerRepo.png" style="width:75%; height: 75%;">

Now that the image is pushed to a registry, you can access it from anywhere with an internet connection. You can also grant other people access to pull it and push changes. The examples in the rest of the chapter will use the shorter of the two image tags (web:latest).

**Run the app**

The containerized application is a web server that listens on TCP port 8080. You can verify this in the app.js ﬁle in the build context you cloned from GitHub. The following command will start a new container called c1 based on the web:latest image you just created. It maps port 80 on the Docker host, to port 8080 inside the container. This means that you’ll be able to point a web browser at the DNS name or IP address of the Docker host running the container and access the app.

**Note:** If your host is already running a service on port 80, you can specify a different port as part of the docker container run command. For example, to map the app to port 5000 on the Docker host, use the -p 5000:8080 ﬂag.
```sh
$ docker container run -d --name c1 -p 80:8080 web:latest
```
The -d ﬂag runs the container in the background, and the -p 80:8080 ﬂag maps port 80 on the host to port 8080 inside the running container. Check that the container is running and verify the port mapping.
```sh
$ docker container ls
```
The output above is snipped for readability, but shows that the app container is running. Note that port 80 is mapped, on all host interfaces (0.0.0.0:80).

**Test the app**

Open a web browser and point it to the DNS name or IP address of the host that the container is running on. If the test does not work, try the following:

1. Make sure that the container is up and running with the docker container ls command. The container name is c1 and you should see the port mapping as 0.0.0.0:80->8080/tcp.

2. Check that ﬁrewall and other network security settings are not blocking traffic to port 80 on the Docker host.

3. Retry the command specifying a high numbered port on the Docker host (may be -p 5000:8080).


**Looking a bit closer**

Now that the application is containerized, let’s take a closer look at how some of the machinery works. The docker image build command parses the Dockerﬁle one-line-at-a-time starting from the top. Comment lines start with the # character. All non-comment lines are **Instructions** and take the format INSTRUCTION argument. Instruction names are not case sensitive, but it’s normal practice to write them in UPPERCASE. This makes reading the Dockerﬁle easier.

Some instructions create new layers, whereas others just add metadata to the image conﬁg ﬁle. Examples of instructions that create new layers are FROM, RUN, and COPY. Examples that create metadata include EXPOSE, WORKDIR, ENV, and ENTRYPOINT. The basic premise is this — if an instruction is adding *content* such as ﬁles and programs to the image, it will create a new layer. If it is adding instructions on how to build the image and run the application, it will create metadata. You can view the instructions that were used to build the image with the docker image history command.
```sh
$ docker image history web:latest
```

Two things from the output above are worth noting. First. Each line corresponds to an instruction in the Dockerﬁle (starting from the bottom and working up). The CREATED BY column even lists the exact Dockerﬁle instruction that was executed.Second. Only 4 of the lines displayed in the output create new layers (the ones with non-zero values in the SIZE column). These correspond to the FROM, RUN, and COPY instructions in the Dockerﬁle. Although the other instructions might look like they create layers, they actually create metadata instead of layers. The reason that the docker image history output makes it looks like all instructions create layers is an artefact of the way builds and image layering used to work. Use the docker image inspect command to conﬁrm that only 4 layers were created.
```sh
$ docker image inspect web:latest
```
It is considered a good practice to use images from official repositories with the FROM instruction. You can view the output of the docker image build command to see the general process for building an image.As the following snippet shows, the basic process is: spin up a temporary container > run the Dockerfile instruction inside of that container > save the results as a new image layer > remove the temporary container.

**Moving to production with Multi-stage Builds**

When it comes to Docker images, big is bad! Big means slow. Big means hard to work with. And big means more potential vulnerabilities and possibly a bigger attack surface! For these reasons, Docker images should be small. The aim of the game is to only ship production images with the stuff **needed** to run your app in production. The problem is… keeping images small *was* hard work. For example, the way you write your Dockerﬁles has a huge impact on the size of your images. A common example is that every RUN instruction adds a new layer. As a result, it’s usually considered a best practice to include multiple commands as part of a single RUN instruction — all glued together with double-ampersands (&&) and backslash (\) line-breaks. While this isn’t rocket science, it requires time and discipline. Another issue is that we don’t clean up after ourselves. We’ll RUN a command against an image that pulls some build-time tools, and we’ll leave all those tools in the image when we ship it to production. Not ideal! Multi-stage builds to the rescue! Multi-stage builds are all about optimizing builds without adding complexity. And they deliver on the promise! Here’s the high-level…

Multi-stage builds have a single Dockerﬁle containing multiple FROM instructions. Each FROM instruction is a new **build stage** that can easily COPY artefacts from previous **stages**. Let’s look at an example! This example app is available at https://github.com/nigelpoulton/atsea-sample-shop-app.git and the Dockerﬁle is in the app directory. It’s a Linux-based application so, will only work on a Linux Docker host. It’s also quite old, so don’t deploy it to an important system, and be sure to delete it as soon as you’re ﬁnished. The ﬁrst thing to note is that the Dockerﬁle has three FROM instructions. Each of these constitutes a distinct **build stage**. Internally, they’re numbered from the top starting at 0. However, we’ve also given each stage a friendly name.

• Stage 0 is called storefront

• Stage 1 is called appserver

• Stage 2 is called production

The storefront stage pulls the node:latest image which is over 900MB in size. The resulting image is an even bigger than the base node:latest image as it contains lots of build stuff and not very much app code.

The appserver stage pulls the maven:latest image which is over 500MB in size. This produces another very large image with lots of build tools and very little actual production code.

The production stage starts by pulling the java:8-jdk-alpine image. This image is approximately 150MB -

considerably smaller than the node and maven images used by the previous build stages. Finally, it sets the main application for the image to run when it’s started as a container. An important thing to note, is that COPY --from instructions are used to **only copy production-related application code** from the images built by the previous stages. They do not copy build artefacts that are not needed for production. It’s also important to note that we only need a single Dockerﬁle, and no extra arguments are needed for the docker image build command!

```sh
$ docker image build -t multi:stage .
```
Run a docker image ls to see the list of images pulled and created by the build operation.
```sh
$ docker image ls
```

The top line in the output above shows the node:latest image pulled by the storefront stage. The image below is the image produced by that stage (created by adding the code and running the npm install and build operations). Both are very large images with lots of build junk included. The 3rd and 4th lines are the images pulled and produced by the appserver stage. These are both large and contain lots of builds tools. The last line is the multi:stage image built by the ﬁnal build stage in the Dockerﬁle (stage2/production). You can see that this is signiﬁcantly smaller than the images pulled and produced by the previous stages. This is because it’s based off the much smaller java:8-jdk-alpine image and has only added the production-related app ﬁles from the previous stages. The net result is a small production image created by a single Dockerﬁle, a normal docker image build command, and zero additional scripting! Multi-stage builds were new with Docker 17.05 and are an excellent feature for building small production-worthy images.


**Leverage the build cache**

When building an image, Docker steps through the instructions in your Dockerfile, executing each in the order specified. As each instruction is examined, Docker looks for an existing image in its cache that it can reuse, rather than creating a new (duplicate) image. If you do not want to use the cache at all, you can use the --no-cache=true option on the docker build command. However, if you do let Docker use its cache, it is important to understand when it can, and cannot, find a matching image.


**Squash the image**

Squashing an image isn’t really a best practice as it has pros and cons. At a high level, Docker follows the normal process to build an image, but then adds an additional step that squashes everything into a single layer. Squashing can be good in situations where images are starting to have a lot of layers and this isn’t ideal. An example might be when creating a new base image that you want to build other images from in the future — this base is much better as a single-layer image. On the negative side, squashed images do not share image layers. This can result in storage ineﬃciencies and larger push and pull operations. Add the --squash ﬂag to the docker image build command if you want to create a squashed image. The squashed image will also need to send every byte to Docker Hub on a docker image push command, whereas the non-squashed image only needs to send unique layers.

<img src=".\images\ContainerSquash.png" style="width:75%; height: 75%;">

**Use no-install-recommends**

If you are building Linux images, and using the apt package manager, you should use the no-install-recommends ﬂag with the apt-get install command. This makes sure that apt only installs main dependencies (packages in the Depends ﬁeld) and not recommended or suggested packages. This can greatly reduce the number of unwanted packages that are downloaded into your images.



**13: Volumes and persistent data**

Stateful applications that persist data are becoming more and more important in the world of cloud-native and microservices applications. Docker is an important infrastructure technology in this space.

There are two main categories of data — persistent and non-persistent. Persistent is the data you need to *keep*. Things like; customer records, ﬁnancial data, research results, audit logs, and even some types of application *log* data. Non-persistent is the data you don’t need to keep. Both are important, and Docker has solutions for both. To deal with non-persistent data, every Docker container gets its own non-persistent storage. This is automatically created for every container and is tightly coupled to the lifecycle of the container. As a result, deleting the container will delete the storage and any data on it. To deal with persistent data, a container needs to store it in a *volume*. Volumes are separate objects that have their lifecycles decoupled from containers. This means you can create and manage volumes independently, and they’re not tied to the lifecycle of any container. Net result, you can delete a container that’s using a volume, and the volume won’t be deleted.Containers deal with persistent and non-persistent data, and you may ﬁnd it hard to see many differences with virtual machines.


**Containers and non-persistent data**

Containers are designed to be immutable. This is just a buzzword that means read-only — it’s a best practice not to change the conﬁguration of a container after it’s deployed. If something breaks or you need to change something, you should create a new container with the ﬁxes/updates and deploy it in place of the old container. You shouldn’t log into a running container and make conﬁguration changes! However, many applications require a read-write ﬁlesystem in order to simply run – they won’t even run on a read-only ﬁlesystem. This means it’s not as simple as making containers entirely read-only. Every Docker container is created by adding a thin read-write layer on top of the read-only image it’s based on. Figure shows two running containers sharing a single read-only image.
<img src=".\images\ContainerLayerRO.png" style="width:75%; height: 75%;">

**Figure 13.1 Ephemeral container storage**

The writable container layer exists in the ﬁlesystem of the Docker host, and you’ll hear it called various names. These include *local storage*, *ephemeral storage*, and *graphdriver storage*. It’s typically located on the Docker host in these locations:

```sh
• Linux Docker hosts: /var/lib/docker/<storage-driver>/...

• Windows Docker hosts: C:\ProgramData\Docker\windowsfilter\...
```
This thin writable layer is an integral part of a container and enables all read/write operations. If you, or an application, update ﬁles or add new ﬁles, they’ll be written to this layer. However, it’s tightly coupled to the container’s lifecycle — it gets created when the container is created and it gets deleted when the container is deleted. The fact that it’s deleted along with a container means that it’s not an option for important data that you need to keep (persist). If your containers don’t create persistent data, this thin writable layer of *local storage* will be ﬁne and you’re good to go. However, if your containers need to persist data, you need to read the next section.


**Containers and persistent data**

*Volumes* are the recommended way to persist data in containers. There are three major reasons for this:

• Volumes are independent objects that are not tied to the lifecycle of a container

• Volumes can be mapped to specialized external storage systems

• Volumes enable multiple containers on different Docker hosts to access and share the same data

At a high-level, you create a volume, then you create a container and mount the volume into it. The volume is mounted into a directory in the container’s ﬁlesystem, and anything written to that directory is stored in the volume. If you delete the container, the volume and its data will still exist.

<img src=".\images\DockerVolume.png" style="width:75%; height: 75%;">

**Creating and managing Docker volumes**

Volumes are ﬁrst-class citizens in Docker. Among other things, this means they are their own object in the API and have their own docker volume sub-command.
```sh
$ docker volume create myvol
```
By default, Docker creates new volumes with the built-in local driver. As the name suggests, volumes created with the local driver are only available to containers on the same node as the volume. You can use the -d ﬂag to specify a different driver. Now that the volume is created, you can see it with the docker volume ls command and inspect it.
```sh
$ docker volume ls
$ docker volume inspect myvol
```

Notice that the Driver and Scope are both local. This means the volume was created with the local driver and is only available to containers on this Docker host. The Mountpoint property tells us where in the Docker host’s ﬁlesystem the volume exists.


All volumes created with the local driver get their own directory under /var/lib/docker/volumes on Linux, and C:\ProgramData\Docker\volumes on Windows. This means you can see them in your Docker host’s ﬁlesystem. You can even access them directly from your Docker host, although this is not normally recommended.

Now the volume is created, it can be used by one or more containers.There are two ways to delete a Docker volume docker volume prune will delete **all volumes** that are not mounted into a container or service replica, so **use with caution!** docker volume rm lets you specify exactly which volumes you want to delete. Neither command will delete a volume that is in use by a container or service replica. As the myvol volume is not in use, delete it with the prune command.
```sh
$ docker volume prune
```
```sh
$ docker volume rm
```
 However, it’s also possible to deploy volumes via Dockerﬁles using the VOLUME instruction. The format is VOLUME <container-mount-point>. Interestingly, you cannot specify a directory on the host when deﬁning a volume in a Dockerﬁle. This is because *host* directories are different depending on what OS your Docker host is running – it could break your builds if you speciﬁed a directory on a Docker host that doesn’t exist. As a result, deﬁning a volume in a Dockerﬁle requires you to specify host directories at deploy-time.

**Demonstrating volumes with containers and services**
```sh
$ docker container run -dit --name voltainer --mount source=bizvol,target=/vol alpine
```
The command uses the --mount ﬂag to mount a volume called “bizvol” into the container at either /vol or c:\vol. The command completes successfully despite the fact there is no volume on the system called bizvol. This raises an interesting point:

• If you specify an existing volume, Docker will use the existing volume

• If you specify a volume that doesn’t exist, Docker will create it for you
```sh
$ docker volume ls
```
Although containers and volumes have separate lifecycle’s, you cannot delete a volume that is in use by a container.
```sh
$ docker volume rm bizvol
```
The volume is brand new, so it doesn’t have any data. Let’s exec onto the container and write some data to it.
```sh
$ docker container exec -it voltainer sh

/# echo "I promise to write a review of the book on Amazon" > /vol/file1

/# ls -l /vol

/# cat /vol/file1
```
Type exit to return to the shell of your Docker host, and then delete the container with the following command.
```sh
$ docker container rm voltainer -f
```

Even though the container is deleted, the volume still exists:
```sh
$ docker container ls -a
$ docker volume ls
```

Because the volume still exists, you can look at its mount point on the host to check if the data is still there. Run the following commands from the terminal of your Docker host. The ﬁrst one will show that the ﬁle still exists, the second will show the contents of the ﬁle.
```sh
$ ls -l /var/lib/docker/volumes/bizvol/\_data/
$ cat /var/lib/docker/volumes/bizvol/\_data/file1
```

It’s even possible to mount the bizvol volume into a new service or container. The following command creates a new Docker service, called hellcat, and mounts bizvol into the service replica at /vol. You’ll need to be running in swarm mode for this command to work. If you’re running in single-engine mode you can use a docker container run command instead.
```sh
$ docker service create --name hellcat --mount source=bizvol,target=/vol alpine sleep 1d
$ docker service ps hellcat
```
In this example, the replica is running on node1. Log on to node1 and get the ID of the service replica container.
```sh
node1$ docker container ls
```

Notice that the container name is a combination of service-name, replica-number, and replica-ID separated by periods. Exec onto the container and check that the data is present in /vol. We’ll use the service replica’s container ID in the exec example. 
```sh
node1$ docker container exec -it df6 sh
/# cat /vol/file1
```
Excellent, the volume has preserved the original data and made it available to a new container.

**Sharing storage across cluster nodes**

Integrating external storage systems with Docker makes it possible to share volumes between cluster nodes. These external systems can be cloud storage services or enterprise storage systems in your on-premises data centers. As an example, a single storage LUN or NFS share can be presented to multiple Docker hosts, allowing it to be used by containers and service replicas no-matter which Docker host they’re running on. Figure shows a single external shared volume being presented to two Docker nodes. These Docker nodes can then make the shared volume available to either, or both containers.

<img src=".\images\ShareStorage.png" style="width:75%; height: 75%;">


Building a setup like this requires a lot of things. You need access to a specialised storage systems and knowledge of how it works and presents storage. You also need to know how your applications read and write data to the shared storage. Finally, you need a volumes driver plugin that works with the external storage system.bDocker Hub is the best place to ﬁnd volume plugins. Login to Docker Hub, select the view to show plugins instead of containers, and ﬁlter results to only show Volume plugins. Once you’ve located the appropriate plugin for your storage system, you create any conﬁguration ﬁles it might need, and install it with docker plugin install. Once the plugin is registered, you can create new volumes from the storage system using docker volume create with the -d ﬂag. The following example installs the Pure Storage Docker volume plugin. This plugin provides access to storage volumes on either a Pure Storage FlashArray or FlashBlade storage system. Plugins only work with the correct external storage systems.

1. The Pure Storage plugin requires a conﬁguration ﬁle called pure.json in the Docker host’s /etc/pure-docker-plugin/directory. This ﬁle contains the information required for the plugin to locate the external storage system, authenticate, and access resources.

2. Install the plugin and grant the required permissions.
```sh
$ docker plugin install purestorage/docker-plugin:latest --alias pure --grant-all-permissions
$ docker plugin ls
```

1. Create a new volume with the plugin (you can also do this as part of the container creation process). This example creates a new 25GB volume called “fastvol” on the registered Pure Storage backend.
```sh
$ docker volume create -d pure -o size=25GB fastvol
```
Different storage drivers support different options, but this should be enough to give you a feel for how they work.


**Potential data corruption**

A major concern with any conﬁguration that shares a single volume among multiple containers is **data corruption**.Assume the following example based on Figure above. The application running in ctr-1 on node-1 updates some data in the shared volume. However, instead of writing the update directly to the volume, it holds it in its local buffer for faster recall (this is common in many operating systems). At this point, the application in ctr-1 thinks the data has been written to the volume. However, before ctr-1 on node-1 ﬂushes its buffers and commits the data to the volume, the app in ctr-2 on node-2 updates the same data with a different value and commits it directly to the volume. At this point, both applications *think* they’ve updated the data in the volume, but in reality only the application in ctr-2 has. A few seconds later, ctr-1 on node-1 ﬂushes the data to the volume, overwriting the changes made by the application in ctr-2. However, the application in ctr-2 is totally unaware of this! This is one of the ways data corruption happens. To prevent this, you need to write your applications in a way to avoid things like this.




**14: Deploying apps with Docker Stacks**

Deploying and managing cloud-native microservices applications comprising lots of small integrated services at scale is hard. Fortunately, Docker Stacks are here to help. They simplify application management by providing; *desired state, rolling updates, simple, scaling operations, health checks,* and more! All wrapped in a nice declarative model.

Testing and deploying simple apps on your laptop is easy, but that’s for amateurs. Deploying and managing multi-service apps in real-world production environments… that’s for pro’s! Fortunately, stacks are here to help!. They let you deﬁne complex multi-service apps in a single declarative ﬁle. They also provide a simple way to deploy the app and manage its entire lifecycle — initial deployment > health checks > scaling > updates > rollbacks and more!

The process is simple. Deﬁne the desired state of your app in a *Compose ﬁle*, then deploy and manage it with the docker stack command. That's it. The Compose ﬁle includes the entire stack of microservices that make up the app. It also includes all of the volumes, networks, secrets, and other infrastructure the app needs. The docker stack deploy command is used to deploy the entire app from the single ﬁle. To accomplish all of this, stacks build on top of Docker Swarm, meaning you get all of the security and advanced features that come with Swarm. In a nutshell, Docker is great for application development and testing. Docker Stacks are great for scale and production. Architecturally speaking, stacks are at the top of the Docker application hierarchy. They build on top of *services*, which in turn build on top of containers.

**Overview of the sample app**

For the rest of the chapter, we’ll be using the popular demo app. Beneath the covers, it’s a cloud-native microservices app that leverages certiﬁcates and secrets. The high-level application architecture is shown in Figure.

<img src=".\images\StackExample.png" style="width:75%; height: 75%;">

As you can see, it comprises 5 *Services*, 3 networks, 4 secrets, and 3 port mappings. We’ll see each of these in detail when we inspect the stack ﬁle.

**Note:** When referring to *services* in this chapter, we’re talking about the Docker service object that is one or more identical containers managed as a single object on a swarm cluster.

```sh
$ git clone https://github.com/dockersamples/atsea-sample-shop-app.git
```

The application consists of several directories and source ﬁles. Feel free to explore them all. However, we’re going to focus on the docker-stack.yml ﬁle that deﬁnes the app and its requirements. We’ll refer to this as the *stack ﬁle*. At the highest level, it deﬁnes 4 top-level keys; version, services, networks, secrets.

**Version** indicates the version of the Compose ﬁle format. This has to be 3.0 or higher to work with stacks. **Services** is where you deﬁne the stack of services that make up the app. **Networks** lists the required networks, and **secrets** deﬁnes the secrets the app uses. If you expand each top-level key, you’ll see how things map to Figure above. The stack ﬁle has ﬁve services called “reverse\_proxy”, “database”, “appserver”, “visualizer”, and “payment\_gateway”. The stack ﬁle has three networks called “front-tier”, “back-tier”, and “payment”. So does Figure 14.1. Finally, the stack ﬁle has four secrets called “postgres\_password”, “staging\_token”, “revprox\_key”, and “revprox\_cert”. It’s important to understand that the stack ﬁle captures and deﬁnes many of the requirements of the entire application. As such, it’s self-documenting and a great tool for bridging the gap between dev and ops. 

**Looking closer at the stack ﬁle**

Stacks ﬁles are very similar to Compose ﬁles. The only requirement is that the version: key specify a value of “3.0” or higher. One of the ﬁrst things Docker does when deploying an app from a stack ﬁle is create any required networks listed under the networks key. If the networks don’t already exist, Docker creates them.

**Networks**

The stack ﬁle describes three networks; front-tier, back-tier, and payment. By default, they’ll all be created as overlay networks by the overlay driver. But the payment network is special — it requires an encrypted data plane. As mentioned in the chapter on overlay networking, the control plane of all overlay networks is encrypted by default, but you have to explicitly encrypt the data plane. The control plane is for network management traffic and the data plane is for application traffic. Encrypting the data plane has a potential performance overhead. To encrypt the data plane, you have two choices:

• Pass the -o encrypted ﬂag to the docker network create command.

• Specify encrypted: 'yes' under driver\_opts in the stack ﬁle.

The overhead incurred by encrypting the data plane depends on various factors such traffic type and traffic ﬂow. You should perform extensive testing to understand the performance overhead that encrypting data plane traffic has on your workload. It’s not uncommon for this to be approximately 10%. As previously mentioned, all three networks will be created before the secrets and services.


**Secrets**

Secrets are deﬁned as top-level objects, and the stack ﬁle we’re using deﬁnes four secrets. Notice that all four are deﬁned as external. This means that they must already exist before the stack can be deployed. It’s possible for secrets to be created on-demand when the application is deployed — just replace external: true with file: `<filename>`. However, for this to work, a plaintext ﬁle containing the unencrypted value of the secret must already exist on the host’s ﬁlesystem. This has obvious security implications.

**Services**

Services are where most of the action happens. Each service is a JSON collection (dictionary) that contains a bunch of keys. We’ll step through each one and explain what each of the options does.

**The reverse\_proxy service**

As you can see, the reverse\_proxy service deﬁnes an image, ports, secrets, and networks. The image key is the only mandatory key in the service object. As the name suggests, it deﬁnes the Docker image that will be used to build the replicas for the service. Remember that a service is one or more identical containers. Docker is opinionated, so unless you specify otherwise, the **image** will be pulled from Docker Hub. You can specify images from 3rd-party registries by prepending the image name with the DNS name of the registry’s API endpoint such as gcr.io for Google’s container registry. One difference between Docker Stacks and Docker Compose is that stacks do not support **builds**. This means all images have to be built prior to deploying the stack. 

The **ports** key deﬁnes two mappings:

• 80:80 maps port 80 across the swarm to port 80 on each service replica.

• 443:443 maps port 443 across the Swarm to port 443 on each service replica.


By default, all ports are mapped using *ingress mode*. This means they’ll be mapped and accessible from every node in the Swarm — even nodes not running a replica. The alternative is *host mode*, where ports are only mapped on swarm nodes running replicas for the service. However, *host mode* requires you to use the long-form syntax. For example, mapping port 80 in *host mode* using the long-form syntax would be like this:

ports:

\- target: 80

\- published: 80

\- mode: host

The long-form syntax is recommended, as it’s easier to read and more powerful (it supports ingress mode **and** host mode). However, it requires at least version 3.2 of the Compose ﬁle format. The **secrets** key deﬁnes two secrets — revprox\_cert and revprox\_key. These secrets must already exist on the swarm and must also be deﬁned in the top-level secrets section of the stack ﬁle. Secrets get mounted into service replicas as a regular ﬁle. The name of the ﬁle will be whatever you specify as the target value in the stack ﬁle, and the ﬁle will appear in the replica under /run/secrets on Linux, and C:\ProgramData\Docker\secrets on Windows. Linux mounts /run/secrets as an in-memory ﬁlesystem, but Windows does not. The secrets deﬁned in this service will be mounted in each service replica as /run/secrets/revprox\_cert and /run/secrets/revprox\_key. To mount one of them as /run/secrets/uber\_secret you would deﬁne it in the stack ﬁle as follows: 
secrets:

\- source: revprox\_cert

\- target: uber\_secret

The **networks** key ensures that all replicas for the service will be attached to the front-tier network. The network speciﬁed here must be deﬁned in the networks top-level key, and if it doesn’t already exist, Docker will create it as an overlay.

**The database service**

The database service also deﬁnes; an image, a network, and a secret. As well as those, it introduces environment variables and placement constraints. The **environment** key lets you inject environment variables into services replicas at runtime. This service uses three environment variables to deﬁne a database user, the location of the database password (a secret mounted into every service replica), and the name of the database. A better and more secure solution would be to pass all three values in as secrets, as this would avoid documenting the database name and database user in plaintext variables. The service also deﬁnes a *placement constraint* under the deploy key. This ensures that replicas for this service will always run on Swarm *worker* nodes.

Placement constraints are a great way of inﬂuencing scheduling decisions. Swarm currently lets you schedule against all of the following:

• Node ID. node.id == o2p4kw2uuw2a

• Node name. node.hostname == wrk-12

• Role. node.role != manager

• Engine labels. engine.labels.operatingsystem==ubuntu 16.04

• Custom node labels. node.labels.zone == prod1

Notice that == and != are both supported.

**The appserver service**

The appserver service uses an image, attaches to three networks, and mounts a secret. It also introduces several additional features under the deploy key. Let’s take a closer look at the new stuff under the deploy key.First up, services.appserver.deploy.replicas = 2 will set the desired number of replicas for the service to 2. If omitted, the default value is 1. If you need to change the number of replicas after you’ve deployed the service, you should do so declaratively. This means updating services.appserver.deploy.replicas ﬁeld in the stack ﬁle with the new value, and then redeploying the stack. We’ll see this later, but re-deploying a stack does not affect services that you haven’t made a change to. services.appserver.deploy.update\_config tells Docker how to act when updating the service. For this service, Docker will update two replicas at-a-time (parallelism) and will perform a ‘rollback’ if it detects the update is failing. Rolling back will start new replicas based on the previous deﬁnition of the service. The default value for failure\_action is pause, which will stop further replicas being updated. The other option is continue. You specify other options as part of update\_config. These include inserting a delay, a failure monitor period, and controlling the order of starting updated replicas before terminating older replicas or vice versa. The services.appserver.deploy.restart-policy object tells Swarm how to restart replicas (containers) if and when they fail. The policy for this service will restart a replica if it stops with a non-zero exit code (condition: on-failure). It will try to restart the failed replica 3 times, and wait up to 120 seconds to decide if the restart worked. It will wait 5 seconds between each of the three restart attempts. 

    restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s

**visualizer**

The visualizer service references an image, maps a port, deﬁnes an update conﬁg, and deﬁnes a placement constraint. It also mounts a volume and deﬁnes a custom grace period for container stop operations. 

    visualizer:
        image: dockersamples/visualizer:stable
        ports:
            - "8001:8080"
        stop_grace_period: 1m30s
        volumes:
            - "/var/run/docker.sock:/var/run/docker.sock"
        deploy:
            update_config:
                failure_action: rollback
            placement:
                constraints:
                    - 'node.role == manager'

When Docker stops a container, it issues a SIGTERM to the application process with PID 1 inside the container. The application then has a 10-second grace period to perform any clean-up operations. If it doesn’t handle the signal, it will be forcibly terminated after 10 seconds with a SIGKILL. The stop\_grace\_period property overrides this 10 second grace period. The volumes key is used to mount pre-created volumes and host directories into a service replica. In this case, it’s mounting /var/run/docker.sock from the Docker host, into /var/run/docker.sock inside of each service replica. This means any reads and writes to /var/run/docker.sock in the replica will be passed through to the same directory in the host. /var/run/docker.sock happens to be the IPC socket that the Docker daemon exposes all of its API endpoints on. This means giving a container access to it gives the container the ability to issue commands to the Docker daemon. This has signiﬁcant security implications and is not recommended in the real world. Fortunately, this is just a demo app in a lab environment. The reason this service requires access to the Docker daemon is because it provides a graphical representation of services on the Swarm. To do this, it needs to be able to query the Docker daemon on a manager node. To accomplish this, a placement constraint forces all service replicas onto manager nodes, and the Docker socket is bind-mounted into each service replica.

**payment_gateway**

The payment_gateway service speciﬁes an image, mounts a secret, attaches to a network, deﬁnes a partial deployment strategy, and then imposes a couple of placement constraints.

payment_gateway:
    image: dockersamples/atseasampleshopapp_payment_gateway
    secrets:
        - source: staging_token
        - target: payment_token
    networks:
        - payment
    deploy:
        update_config:
        failure_action: rollback
        placement:
            constraints:
                - 'node.role == worker'
                - 'node.labels.pcidss == yes'

We’ve seen all of these options before, except for the node.label in the placement constraint. Node labels are custom-deﬁned labels added to swarm nodes with the docker node update command. As such, they’re only applicable within the context of the node’s role in the Swarm (you can’t leverage them on standalone containers or outside of the Swarm). In this example, the payment_gateway service performs operations that require it to run on a swarm node that has been hardened to PCI DSS standards. To enable this, you can apply a custom *node label* to any swarm node meeting these requirements. We’ll do this when we build the lab to deploy the app.

As this service deﬁnes two placement constraints, replicas will only be deployed to nodes that match both. I.e. a **worker** node with the pcidss=yes node label. Now that we’re ﬁnished examining the stack ﬁle, you should have a good understanding of the application’s requirements. As mentioned previously, the stack ﬁle is a great piece of application documentation. We know the application has 5 services, 3 networks, and 4 secrets. We know which services attach to which networks, which ports need publishing, which images are required, and we even know that some services need to run on speciﬁc nodes.

**Deploying the app**

There’s a few pre-requisites that need taking care of before deploying the app:

• **Swarm mode:** We’ll deploy the app as a Docker Stacks, and stacks require Swarm mode.

• **Labels:** One of the Swarm worker nodes needs a custom node label.

• **Secrets:** The app uses secrets which need pre-creating before it can be deployed.


Let’s create a new three-node Swarm cluster.

1. Initialize a new Swarm.

Run the following command on the node that you want to be your Swarm manager.
```sh
$ docker swarm init
```
2. Add worker nodes.

Copy the docker swarm join command that displayed in the output of the previous command. Paste it into the two nodes you want to join as workers.

//Worker 1 (wrk-1)
```sh
wrk-1$ docker swarm join --token SWMTKN-1-2hl6...-...3lqg 172.31.40.192:2377
```
//Worker 2 (wrk-2)
```sh
wrk-2$ docker swarm join --token SWMTKN-1-2hl6...-...3lqg 172.31.40.192:2377
```
3. Verify that the Swarm is conﬁgured with one manager and two workers.
```sh
$ docker node ls
```

The Swarm is now ready. The payment_gateway service has a set of placement constraints forcing it to only run on **worker nodes** with the pcidss=yes node label. In this step we’ll add that node label to wrk-1. In the real world you would harden at least one of your Docker nodes to PCI standards before labelling it. However, this is just a lab, so we’ll skip the hardening step and just add the label to wrk-1.

1. Add the node label to wrk-1.
```sh
$ docker node update --label-add pcidss=yes wrk-1
```

Node labels only apply within the Swarm.

2. Verify the node label.
```sh
$ docker node inspect wrk-1
```
The wrk-1 worker node is now conﬁgured so that it can run replicas for the payment_gateway service. The application deﬁnes four secrets, all of which need creating before the app can be deployed:

• postgress_password

• staging_token

• revprox_cert

• revprox_key


Run the following commands from the manager node to create them.

1. Create a new key pair.

Three of the secrets will be populated with cryptographic keys. We’ll create the keys in this step and then place them inside of Docker secrets in the next steps.
```sh
$ openssl req -newkey rsa:4096 -nodes -sha256  -keyout domain.key -x509 -days 365 -out domain.crt
```
You’ll have two new ﬁles in your current directory. We’ll use them in the next step.

2. Create the revprox_cert, revprox_key, and postgress_password secrets.
```sh
$ docker secret create revprox_cert domain.crt cqblzfpyv5cxb5wbvtrbpvrrj

$ docker secret create revprox_key domain.key jqd1ramk2x7g0s2e9ynhdyl4p

$ docker secret create postgres_password domain.key njpdklhjcg8noy64aileyod6l
```
3. Create the staging_token secret.
```sh
$ echo staging | docker secret create staging_token -sqy21qep9w17h04k3600o6qsj
```
4. List the secrets.
```sh
$ docker secret ls
```


**Deploying the sample app**

If you haven’t already done so, clone the app’s GitHub repo to your Swarm manager.
```sh
$ git clone https://github.com/dockersamples/atsea-sample-shop-app.git

$ cd atsea-sample-shop-app
```
Stacks are deployed using the docker stack deploy command. In its basic form it accepts two arguments:

• name of the stack ﬁle

• name of the stack

The application’s GitHub repository contains a stack ﬁle called docker-stack.yml, so we’ll use this as stack ﬁle. We’ll call the stack seastack, though you can choose a different name if you don’t like that. Run the following commands from within the atsea-sample-shop-app directory on the Swarm manager. Deploy the stack (app).
```sh
$ docker stack deploy -c docker-stack.yml seastack
```

You can run  `docker network ls` and `docker service ls` commands to see the networks and services that were deployed as part of the app. A few things to note from the output of the command. The networks were created before the services. This is because the services attach to the networks, so need the networks to be created before they can start. Docker prepends the name of the stack to every resource it creates. In our example, the stack is called seastack, meaning all resources are named seastack_resource. For example, the payment network is called seastack_payment. Resources that were created prior to the deployment, such as secrets, do not get renamed. Another thing to note is the presence of a network called seastack_default. This isn’t deﬁned in the stack ﬁle, so why was it created? Every service needs to attach to a network, but the visualizer service didn’t specify one. Therefore, Docker created one called seastack_default and attached it to that. You can verify this by running a docker network inspect seastack_default command. You can verify the status of a stack with a couple of commands. docker stack ls lists all stacks on the system, including how many services they have. `docker stack ps <stack-name>` gives more detailed information about a particular stack, such as *desired state* and *current state*. 
```sh
$ docker stack ls

$ docker stack ps seastack
```

The docker stack ps command is a good place to start when troubleshooting services that fail to start. It gives an overview of every service in the stack, including which node each replica is seduled on, current state, desired state, and error message. The following output shows two failed attempts to start a replica for the reverse_proxy service on the wrk-2 node.
```sh
$ docker stack ps seastack
```
For more detailed logs of a particular service you can use the docker service logs command. You pass it either the service name/ID, or replica ID. If you pass it the service name or ID, you’ll get the logs for all service replicas. If you pass it a particular replica ID, you’ll only get the logs for that replica.The following `docker service logs` command shows the logs for all replicas in the seastack_reverse_proxy service that had the two failed replicas in the previous output.
```sh
$ docker service logs seastack_reverse_proxy
```

The output is trimmed to ﬁt the page, but you can see that logs from all three service replicas are shown (the two that failed and the one that’s running). Each line starts with the name of the replica, which includes the service name, replica number, replica ID, and name of host that it’s seduled on. Following that is the log output.

**Note:** You might have noticed that all of the replicas in the previous output showed as replica number 1. This is because Docker created one at a time and only started a new one when the previous one had failed.

It’s hard to tell because the output is trimmed to ﬁt the book, but it looks like the ﬁrst two replicas failed because they were relying on something in another service that was still starting (a sort of race condition when dependent services are starting). You can follow the logs (--follow), tail them (--tail), and get extra details (--details).

**Managing the app**

We know that a *stack* is set of related services and infrastructure that gets deployed and managed as a unit. And while that’s a fancy sentence full of buzzwords, it reminds us that the stack is built from normal Docker resources networks, volumes, secrets, services etc. This means we can inspect them with their normal Docker commands:  docker network, docker volume, docker secret, docker service…


With this in mind, it’s possible to use the docker service command to manage services that are part of the stack. A simple example would be using the docker service scale command to increase the number of replicas in the appserver service. However, **this is not the recommended method!** The recommended method is the declarative method, which uses the stack ﬁle as the ultimate source of truth. As such, all changes to the stack should be made to the stack ﬁle, and then the updated stack ﬁle should be used to redeploy the app.


We’ll make the following changes:

• Increase the number of appserver replicas from 2 to 10

• Increase the stop grace period for the visualizer service to 2 minutes

Edit the docker-stack.yml ﬁle and update the following two values:

• services.appserver.deploy.replicas=10

• services.visualizer.stop_grace_period=2m

Save the ﬁle and redeploy the app.

```sh
$ docker stack deploy -c docker-stack.yml seastack
```
Re-deploying the app like this will only update the changed components. Run a docker stack ps to see the number of appserver replicas increasing.
```sh
$ docker stack ps seastack
```

Notice that there are two lines for the visualizer service. One line shows a replica that was shutdown, and the other line shows a replica that has been running. This is because the change we made to the visualizer service caused Swarm to terminate the existing replica and started a new one with the new stop_grace_period value. You can also see that there are now 10 replicas for the appserver service, and that they are in various states in the “CURRENT STATE” column — some are *running* whereas others are still *starting*. After enough time, the cluster will converge so that *current observed state* matches the new *desired state*. At that point, what is deployed and observed on the cluster will exactly match what is deﬁned in the stack ﬁle.  This declarative update pattern should be used for all updates to the app/stack. I.e. **all changes should be made declaratively via the stack ﬁle, and rolled out using docker stack deploy**.

The correct way to delete a stack is with the `docker stack rm` command. Be warned though! It deletes the stack without asking for conﬁrmation.
```sh
$ docker stack rm seastack
```

Notice that the networks and services were deleted, but the secrets weren’t. This is because the secrets were pre- created and existed before the stack was deployed. If your stack deﬁnes volumes at the top-level, these will not be deleted by `docker stack rm` either. This is because volumes are intended as long-term persistent data stores and exist independent of the lifecycle of containers, services, and stacks.




**15: Security in Docker**

Good security is all about layers, and Docker has lots of layers. It supports all the major Linux security technologies as well as plenty of its own. And the best thing… many of them are simple and easy to conﬁgure. In this chapter, we’ll look at some of the technologies that can make running containers on Docker very secure.

Security is all about layers. Generally speaking, the more layers of security the more secure something is. Docker offers a lot of security layers. Figure shows some of the security-related technologies we’ll cover in the chapter.

<img src=".\images\Security.png" style="width:75%; height: 75%;">

Docker on Linux leverages most of the common Linux security and workload isolation technologies. These include *namespaces*, *control groups (cgroups)*, *capabilities*, *mandatory access control (MAC) systems*, and *seccomp*. For each one, Docker implements sensible defaults for a seamless and *moderately secure* out-of-the-box experience. However, you can customize each one to your own speciﬁc requirements. Docker itself adds some excellent additional security technologies.

**Docker Swarm Mode** is secure by default. You get all of the following with zero conﬁguration required; cryptographic node IDs, mutual authentication, automatic CA conﬁguration, automatic certiﬁcate rotation, encrypted cluster store, encrypted networks, and more.

**Docker Content Trust (DCT)** lets you sign your images and verify the integrity and publisher of images you consume.

**Image security scanning** analyses images, detects known vulnerabilities, and provides detailed reports.

**Docker secrets** are a way to securely share sensitive data and are ﬁrst-class objects in Docker. They’re stored in the encrypted cluster store, encrypted in-ﬂight when delivered to containers, stored in in-memory ﬁlesystems when in use, and operate a least-privilege model. There’s a lot more, but the important thing to know is that Docker works with the major Linux security technologies as well as providing its own extensive and growing set of security technologies. While the Linux security technologies tend to be complex, the native Docker security technologies tend to be simple.


We all know that security is important. We also know that security can be complicated and boring. When Docker decided to bake security into the platform, it decided to make it simple and easy. They knew that if security was hard to conﬁgure, people wouldn’t use it. As a result, most of the security technologies offered by the Docker platform are simple to use. They also ship with sensible defaults — meaning you get a *fairly secure* platform at zero effort. Of course, the defaults aren’t perfect, but they’re enough to serve as a safe starting point. From there you should customize them to your requirements.

• Linux security technologies

**–** Namespaces

**–** Control Groups

**–** Capabilities

**–** Mandatory Access Control

**–** seccomp

• Docker platform security technologies

**–** Swarm Mode

**–** Image Scanning

**–** Docker Content Trust

**–** Docker Secrets

**Linux security technologies**

All *good* container platforms use *namespaces* and *cgroups* to build containers. The *best* container platforms will also integrate with other Linux security technologies such as *capabilities*, *Mandatory Access Control systems* like SELinux and AppArmor, and *seccomp*. As expected, Docker integrates with them all.

**Namespaces**

Kernel namespaces are at the very heart of containers. They slice up an operating system (OS) so that it looks and feels like multiple **isolated** operating systems. This lets us do really cool things like run multiple web servers on the same OS without having port conﬂicts. It also lets us run multiple apps on the same OS without them ﬁghting over shared conﬁg ﬁles and shared libraries.
A couple of quite examples:

• Namespaces let you run multiple web servers, each on port 443, on a single OS. To do this you just run each web server app inside of its own *network namespace*. This works because each *network namespace* gets its own IP address and full range of ports. You may have to map each one to a separate port on the Docker host, but each can run without being re-written or reconﬁgured to use a different port. 
• You can run multiple applications, each requiring their own version of a shared library or conﬁguration ﬁle. To do this you run each application inside of its own *mount namespace*. This works because each *mount namespace* can have its own isolated copy of any directory on the system (e.g. /etc, /var, /dev etc.)

Figure shows a high-level example of two web server applications running on a single host and both using port 443. Each web server app is running inside of its own network namespace.
<img src=".\images\Namespaces.png" style="width:75%; height: 75%;">

Working directly with namespaces is hard. Fortunately, Docker hides this complexity and manages all of the namespaces required to build a useful container. Docker on Linux currently utilizes the following kernel namespaces:

• Process ID (pid)

• Network (net)

• Filesystem/mount (mnt)

• Inter-process Communication (ipc)

• User (user)

• UTS (uts)

The most important thing to understand is that **Docker containers are an organized collection of namespaces**. This means that you get all of this OS isolation for free with every container. For example, every container has its own pid, net, mnt, ipc, uts, and potentially user namespace. In fact, an organized collection of these namespaces is what we call a “container”. Figure shows a single Linux host running two containers.

<img src=".\images\NamespacesDocker.png" style="width:75%; height: 75%;">

How Docker uses each namespace:

• Process ID namespace: Docker uses the pid namespace to provide isolated process trees for each container. This means every container gets its own PID 1. PID namespaces also mean that one container cannot see or access to the process tree of other containers. Nor can it see or access the process tree of the host it’s running on.

• Network namespace: Docker uses the net namespace to provide each container its own isolated network stack. This stack includes; interfaces, IP addresses, port ranges, and routing tables. For example, every container gets its own eth0 interface with its own unique IP and range of ports. 

• Mount namespace: Every container gets its own unique isolated root (/) ﬁlesystem. This means every container can have its own /etc, /var, /dev and other important ﬁlesystem constructs. Processes inside of a container cannot access the mount namespace of the Linux host or other containers — they can only see and access their own isolated ﬁlesystem.

• Inter-process Communication namespace: Docker uses the ipc namespace for shared memory access within a container. It also isolates the container from shared memory outside of the container.

• User namespace: Docker lets you use user namespaces to map users inside of a container to different users on the Linux host. A common example is mapping a container’s root user to a non-root user on the Linux host.

• UTS namespace: Docker uses the uts namespace to provide each container with its own hostname.


**Control Groups**

If namespaces are about isolation, *control groups (cgroups)* are about settings limits. Cgroups let us set limits so that no single container can use all resources. Containers are isolated from each other but all share a common set of OS resources — things like CPU, RAM, network bandwidth, and disk I/O. Cgroups let us set limits on each of these so a single container cannot consume everything and cause a denial of service (DoS) attack.

**Capabilities**

It’s a bad idea to run containers as root — root is all-powerful and therefore very dangerous. But, it can be challenging running containers as unprivileged non-root users. For example, on most Linux systems, non-root users tend to be so powerless they’re practically useless. What’s needed, is a technology that lets us pick and choose which root powers a container needs in order to run.

Under the hood, the Linux root user is a combination of a long list of *capabilities*. Some of these *capabilities* include:

• CAP_CHOWN: lets you change ﬁle ownership

• CAP_NET_BIND_SERVICE: lets you bind a socket to low numbered network ports

• CAP_SETUID: lets you elevate the privilege level of a process

• CAP_SYS_BOOT: lets you reboot the system.


The list goes on and is long. Docker works with *capabilities* so that you can run containers as root, but strip out all the capabilities you don’t need. For example, if the only root privilege your container needs is the ability to bind to low numbered network ports, you should start a container and drop all root capabilities, then add back just the CAP_NET_BIND_SERVICE capability. This is an excellent example of implementing *least privilege* — you get a container running with only the capabilities required. Docker also imposes restrictions so that containers cannot re-add the dropped capabilities. 

**Mandatory Access Control systems**

Docker works with major Linux MAC technologies such as AppArmor and SELinux.Depending on your Linux distribution, Docker applies a default AppArmor proﬁle to all new containers. According to the Docker documentation, this default proﬁle is “moderately protective while providing wide application compatibility”. Docker also lets you start containers without a policy applied, as well as giving you the ability to customize policies to meet speciﬁc requirements. This is also very powerful, but can also be prohibitively complex.


**seccomp**

Docker uses seccomp, in ﬁlter mode, to limit the syscalls a container can make to the host’s kernel. As per the Docker security philosophy, all new containers get a default seccomp proﬁle conﬁgured with sensible defaults. This is intended to provide moderate security without impacting application compatibility. As always, you can customize seccomp proﬁles, and you can pass a ﬂag to Docker so that containers can be started without a seccomp proﬁle. As with many of the technologies already mentioned, seccomp is extremely powerful. However, the Linux syscall table is long, and conﬁguring the appropriate seccomp policies can be prohibitively complex.

**Final thoughts on the Linux security technologies**

Docker supports most of the important Linux security technologies and ships with sensible defaults that add security but aren’t too restrictive. Figure 15.4 shows how these technologies form multiple layers of potential security.

<img src=".\images\LinuxSecurity.png" style="width:75%; height: 75%;">

Some of these technologies can be complicated to customize as they require deep knowledge of how the Linux kernel works. Hopefully they will get simpler to conﬁgure in the future, but for now, the default conﬁgurations that ship with Docker might be a good place to start.

**Docker platform security technologies**


**Security in Swarm Mode**

Docker Swarm allows you to cluster multiple Docker hosts and deploy applications declaratively. Every Swarm is comprised of *managers* and *workers* that can be Linux or Windows. Managers host the control plane of the cluster and are responsible for conﬁguring the cluster and dispatching work tasks. Workers are the nodes that run your application code as containers. As expected, *swarm mode* includes many security features that are enabled out-of-the-box with sensible defaults.

These include:

• Cryptographic node IDs

• TLS for mutual authentication

• Secure join tokens

• CA conﬁguration with automatic certiﬁcate rotation

• Encrypted cluster store (conﬁg DB)

• Encrypted networks

Let’s walk through the process of building a secure swarm and conﬁguring some of the security aspects. To follow along with the complete set of examples you’ll need at least three Docker hosts running Docker 17.03 or higher. The examples cited use three Docker hosts called “mgr1”, “mgr2”, and “wrk1”. Each one is running Docker 19.03.4. There is network connectivity between all three hosts, and all three can ping each other by name.

**Conﬁgure a secure Swarm**

Run the following command from the node you want to be the ﬁrst manager in the new swarm. In the example, we’ll run it from **mgr1**. 
```sh
$ docker swarm init
```

That's it! That is literally all you need to do to conﬁgure a secure swarm. **mgr1** is conﬁgured as the ﬁrst manager of the swarm and also as the root certiﬁcate authority (CA). The swarm  itself has been given a cryptographic clusterID. **mgr1** has issued itself with a client certiﬁcate that identiﬁes it as a manager in the swarm, certiﬁcate rotation has been conﬁgured with the default value of 90 days, and a cluster conﬁg database has been conﬁgured and encrypted. A set of secure tokens have also been created so that new managers and new workers can be joined to the swarm. And all of this with a **single command!**

Let’s join **mgr2** as an additional manager. Joining new managers to a swarm is a two-step process. The ﬁrst step extracts the required token. The second step runs the docker swarm join command on the node you wish to add. As long as you include the manager join token as part of the command, **mgr2** will join the swarm as a manager. Run the following command from **mgr1** to extract the manager join token.
```sh
$ docker swarm join-token manager
```
The output gives you the exact command you need to run on nodes that you want to join as managers. The join token and IP address will be different in your lab.

The format of the join command is:

• docker swarm join --token  {manager-join-token}  {ip-of-existing-manager}: {swarm-port}

The format of the token is:

• SWMTKN-1-{hash-of-cluster-certificate}-{manager-join-token}

Copy the command and run it on “mgr2”:
```sh
$ docker swarm join --token SWMTKN-1-1dmtwu...r17stb-2axi5...8p7glz 172.31.5.251:2377
```
This node joined a swarm as a manager. **mgr2** has joined the swarm as an additional manager. In production clusters you should always run either 3 or 5 managers for high availability. Verify **mgr2** was successfully added by running a docker node ls on either of the two managers.
```sh
$ docker node ls
```

The output shows that **mgr1** and **mgr2** are both part of the swarm and are both managers. Two managers is possibly the worst number possible. Adding a swarm worker is a similar two-step process. Step 1 extracts the join token, and step 2 is to run a docker swarm join command on the node you want to join as a worker. Run the following command on either of the managers to expose the worker join token.
```sh
$ docker swarm join-token worker
```
Again, you get the exact command you need to run on nodes you want to join as workers. The join token and IP address will be different in your lab. Copy the command and run it on **wrk1** as shown:
```sh
$ docker swarm join --token SWMTKN-1-1dmtw...17stb-ehp8g...w738q 172.31.5.251:2377
```

This node joined a swarm as a worker. Run another docker node ls command from either of the swarm managers.
```sh
$ docker node ls
```

You now have a swarm with two managers and one worker. The managers are conﬁgured for high availability (HA) and the cluster store is replicated to both. The ﬁnal conﬁguration is shown in Figure .

<img src=".\images\SwarmSecurity.png" style="width:75%; height: 75%;">


**Looking behind the scenes at Swarm security**

Now that we’ve built a secure Swarm let’s take a minute to look behind the scenes at some of the security technologies involved.

**Swarm join tokens**

The only thing that is needed to join new managers and workers to an existing swarm is the relevant join token. For this reason, it’s vital that you keep your join-tokens safe. Do not post them on public GitHub repos or even internal source code repos that are not restricted. Every swarm maintains two distinct join tokens:

• One for joining new managers

• One for joining new workers

It’s worth understanding the format of the Swarm join token. Every join token is comprised of 4 distinct ﬁelds separated by dashes (-):

PREFIX - VERSION - SWARM ID - TOKEN

The preﬁx is always SWMTKN. This allows you to pattern-match against it and prevent people from accidentally posting it publicly. The VERSION ﬁeld indicates the  version of the swarm. The Swarm ID ﬁeld is a hash of the swarm’s certiﬁcate. The TOKEN ﬁeld is the part that determines whether it can join nodes as managers or workers. As the following shows, the manager and worker join tokens for a given Swarm are identical except for the ﬁnal TOKEN ﬁeld.

• MANAGER: SWMTKN-1-1dmtwusdc...r17stb-2axi53zjbs45lqxykaw8p7glz

• WORKER: SWMTKN-1-1dmtwusdc...r17stb-ehp8gltji64jbl45zl6hw738q

If you suspect that either of your join tokens has been compromised, you can revoke them and issue new ones with a single command. The following example revokes the existing *manager* join token and issues a new one.
```sh
$ docker swarm join-token --rotate manager
```

Successfully rotated manager join token. To add a manager to this swarm, run the following command:
```sh
docker swarm join --token SWMTKN-1-1dmtwu...r17stb-1i7txlh6k3hb921z3yjtcjrc7 172.31.5.251:2377
```
Existing managers do not need updating, however, you’ll need to use the new token to add new managers. Notice that the only difference between the old and new join tokens is the last ﬁeld. The hash of the Swarm ID remains the same. Join tokens are stored in the cluster store which is encrypted by default.


**TLS and mutual authentication**

Every manager and worker that joins a swarm is issued a client certiﬁcate. This certiﬁcate is used for mutual authentication. It identiﬁes the node, the swarm that it’s a member of, and role the node performs in the swarm (manager or worker). You can inspect a node’s client certiﬁcate on Linux nodes with the following command.
```sh
$ sudo openssl x509 -in /var/lib/docker/swarm/certificates/swarm-node.crt -text
```

The Subject data in the output uses the standard O, OU, and CN ﬁelds to specify the Swarm ID, the node’s role, and the node ID.

• The Organization (O) ﬁeld stores the Swarm ID

• The Organizational Unit (OU) ﬁeld stores the node’s role in the swarm

• The Canonical Name (CN) ﬁeld stores the node’s crypto ID.

<img src=".\images\SecurityTLS.png" style="width:75%; height: 75%;">


You can also see the certiﬁcate rotation period in the Validity section. You can match these values to the corresponding values shown in the output of a docker system info command.
```sh
$ docker system info
```

**Conﬁguring some CA settings**

You can conﬁgure the certiﬁcate rotation period for the Swarm with the docker swarm update command. The following example changes the certiﬁcate rotation period to 30 days.
```sh
$ docker swarm update --cert-expiry 720h
```
Swarm allows nodes to renew certiﬁcates early (slightly before they expire) so that not all nodes don’t try and update their certiﬁcates at the same time. You can conﬁgure an external CA when creating a new swarm by passing the --external-ca ﬂag to the docker swarm init command. The new docker swarm ca sub-command can be used to manage CA related conﬁguration. Run the command with the --help ﬂag to see a list of things it can do.
```sh
$ docker swarm ca --help
```

**The cluster store**

The cluster store is the brains of a swarm and is where cluster conﬁg and state are stored. It’s also critical to other Docker technologies such as overlay networking and Secrets. This is why swarm mode is required for so many advanced and security related Docker features. The moral of the story… if you’re not running in swarm mode, there’ll be a bun of Docker technologies and security features you won’t be able to use. The store is currently based on the popular etcd distributed database and is automatically conﬁgured to replicate itself to all managers in the swarm. It is also encrypted by default. Day-to-day maintenance of the cluster store is taken care of automatically by Docker. However, in production environments, you should have strong backup and recovery solutions in place for it. 

**Detecting vulnerabilities with image security scanning**

Image scanning is your primary weapon against vulnerabilities and security holes in your images. Image scanners work by inspecting images and searching for packages that have known vulnerabilities. Once you know about these, you can update the packages and dependencies to versions with ﬁxes. As good as image scanning is, it’s important to understand its limitations. For example, image scanning is focussed on images and does not detect security problems with networks, nodes, or orchestrators. Also, not all image scanners are equal — some perform deep binary-level scanning to detect packages, whereas others simply look at package names and do not closely inspect the content of images. At the time of writing, Docker Hub does not offer image scanning services. This may change in the future. Some on-premises private registry solutions offer built-in scanning, and there are third-party services that offer image scanning services. Figures are included as an example of the kind of reports image scanners can provide.

<img src=".\images\ScanImage.png" style="width:75%; height: 75%;">
<img src=".\images\ScanImage2.png" style="width:75%; height: 75%;">

In summary, image security scanning can be a great tool for deeply inspecting your images for known vulnerabilities. Beware though, with great knowledge comes great responsibility — once you become aware of vulnerabilities you are responsible for mitigating or ﬁxing them.

**Signing and verifying images with Docker Content Trust**

Docker Content Trust (DCT) makes it simple and easy to verify the integrity and the publisher of images that you download and run. This is especially important when pulling images over untrusted networks such as the internet. At a high level, DCT allows developers to sign images when they are pushed to Docker Hub or other container registries. These images can then be veriﬁed when they are pulled and ran. This high-level process is shown in Figure.

<img src=".\images\DockerContentTrust.png" style="width:75%; height: 75%;">


DCT can also be used to provide important *context*. This includes; whether or not an image has been signed for use in a particular environment such as “prod” or “dev”, or whether an image has been superseded by a newer version and is therefore stale. The following steps will walk you through conﬁguring Docker Content Trust, signing and pushing an image, and then pulling the signed image. To follow along, you’ll need a cryptographic key-pair to sign images. If you don’t already have a key-pair, you can use the docker trust sub-command to generate a new key-pair one. The following command generates a new key-pair called “nigel”.
```sh
$ docker trust key generate nigel
```
If you already have a key-pair, you can import and load it with docker trust key load key.pem --name nigel. Now that you’ve loaded a valid key-pair, you need to associate it with the image repository you’ll be pushing signed images to. This example uses the nigelpoulton/dct repo on Docker Hub and the nigel.pub key that was created by the previous docker trust key generate command. Your key ﬁle will be different. 
```sh
$ docker trust signer add --key nigel.pub nigel nigelpoulton/dct
```
The following command will sign the nigelpoulton/dct:signed image and push it to Docker Hub.
```sh
$ docker trust sign nigelpoulton/dct:signed
```

Once the image is pushed, you can inspect its signing data with the following command.
```sh
$ docker trust inspect nigelpoulton/dct:signed --pretty
```

You can force a Docker host to always sign and verify image push and pull operations by exporting the DOCKER_CONTENT_TRUST environment variable with a value of 1. In the real world, you’ll want to make this a more permanent feature of Docker hosts.
```sh
$ export DOCKER_CONTENT_TRUST=1
```
Once DCT is enabled, you’ll no longer be able to pull and work with unsigned images. You can test this behavior by attempting to pull the following two images:

• nigelpoulton/dct:unsigned

• nigelpoulton/dct:signed

If you have enabled DCT by settings the DOCKER_CONTENT_TRUST environment variable, you will not be able to pull the dct:unsigned image. However, you will be able to pull the image tagged as signed.
```sh
$ docker image pull nigelpoulton/dct:unsigned
```

Docker Content Trust is an important technology for helping you verify the images you are pulling from container registries. It’s simple to conﬁgure in its basic form, but more advanced features, such as *context*, can be more complex to conﬁgure.

**Docker Secrets**

Many applications need secrets — things like passwords, TLS certiﬁcates, SSH keys, and more. Early versions of Docker had no standardised way of making secrets available to apps in a secure way. It was common for developers to insert secrets into apps via plain text environment variables (we’ve all done it). This was far from ideal. Docker 1.13 introduced *Docker Secrets* as ﬁrst-class objects in the Docker API. Behind the scenes, secrets are encrypted at rest, encrypted in-ﬂight, mounted in containers to in-memory ﬁlesystems, and operate under a least-privilege model where they are only made available to services that have been explicitly granted access to them. It’s quite a comprehensive end-to-end solution, and it even has its own docker secret sub-command.

<img src=".\images\DockerSecret.png" style="width:75%; height: 75%;">

The following steps walk through the high-level workﬂow shown in Figure.

1. The blue secret is created and posted to the Swarm.

2. It gets stored in the encrypted cluster store (all managers have access to the cluster store).

3. The blue service is created and the secret is attached to it.

4. The secret is encrypted in-ﬂight while it is delivered to the tasks (containers) in the blue service.

5. The secret is mounted into the containers of the blue service as an unencrypted ﬁle at /run/secrets/. This is an in-memory tmpfs ﬁlesystem (this step is different on Windows Docker hosts as they do not have the notion of an in-memory ﬁlesystem like tmpfs).

6. Once the container (service task) completes, the in-memory ﬁlesystem is torn down and the secret ﬂushed from the node.

7. The red containers in the red service cannot access the secret.

The reason that secrets are surfaced in their un-encrypted form in running containers is so applications can use them without requiring methods to decrypt them. You can create and manage secrets with the docker secret sub-command, and you can attach them to services by specifying the --secret ﬂag to the `docker service create` command.

