

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




**10: Docker Swarm**

Now that we know how to install Docker, pull images, and work with containers, the next thing we need is a

way to work with things at scale. That's where Docker Swarm comes into play.

As usual, we’ll split this chapter into the usual three parts:

• The TLDR

• The deep dive

• The commands

**Docker Swarm - The TLDR**

Docker Swarm is two main things:

\1. An enterprise-grade secure cluster of Docker hosts

\2. An engine for orestrating microservices apps

On the clustering front, Swarm groups one or more Docker nodes and lets you manage them as a cluster. Out-

of-the-box, you get an encrypted distributed cluster store, encrypted networks, mutual TLS, secure cluster join

tokens, and a PKI that makes managing and rotating certiﬁcates a breeze. You can even non-disruptively add

and remove nodes. It’s a beautiful thing.

While we cover some aspects of Swarm security in this chapter, we go a lot deeper in Chapter 15.

On the orchestration front, Swarm exposes a ri API that allows you to deploy and manage complex

microservices apps with ease. You can deﬁne your apps in declarative manifest ﬁles and deploy them to the

Swarm with native Docker commands. You can even perform rolling updates, rollbacks, and scaling operations.

Again, all with simple commands.

Docker Swarm competes directly with Kubernetes — they both orchestrate containerized applications. While it’s

true that Kubernetes has more momentum and a more active community and ecosystem, Docker Swarm is an

excellent technology and a lot easier to conﬁgure and deploy. It’s an excellent technology for small-to-medium

businesses and application deployments.

**Docker Swarm - The Deep Dive**

We’ll split the deep dive part of this chapter as follows:

• Swarm primer

• Build a secure swarm cluster

• Deploy some swarm services

• Troubleshooting





128

10: Docker Swarm

**Swarm primer**

On the clustering front, a *swarm* consists of one or more Docker *nodes*. These can be physical servers, VMs,

Raspberry Pi’s, or cloud instances. The only requirement is that all nodes have Docker installed and can

communicate over reliable networks.

Nodes are conﬁgured as *managers* or *workers*. *Managers* look after the control plane of the cluster, meaning

things like the state of the cluster and dispatching tasks to *workers*. *Workers* accept tasks from *managers* and

execute them.

The conﬁguration and state of the *swarm* is held in a distributed *etcd* database located on all managers. It’s kept

in memory and is extremely up-to-date. But the best thing about it is that it requires zero conﬁguration — it’s

installed as part of the swarm and just takes care of itself.

Something that’s game changing on the clustering front is the approach to security. TLS is so tightly integrated

that it’s impossible to build a swarm without it. In today’s security conscious world, things like this deserve

all the plaudits they get. *Swarm* uses TLS to encrypt communications, authenticate nodes, and authorize roles.

Automatic key rotation is also thrown in as the icing on the cake. And the best part… it all happens so smoothly

that you don’t even know it’s there.

On the application orchestration front, the atomic unit of seduling on a swarm is the *service*. This is a new

object in the API, introduced along with swarm, and is a higher level construct that wraps some advanced

features around containers. These include scaling, rolling updates, and simple rollbacks. It’s useful to think of a

*service* as an enhanced container.

A high-level view of a swarm is shown in Figure 10.1.

**Figure 10.1 High-level swarm**

That's enough of a primer. Let’s get our hands dirty with some examples.

**Build a secure Swarm cluster**

In this section, we’ll build a secure swarm cluster with three *manager nodes* and three *worker nodes*. You can

use a different lab with different numbers of *managers* and *workers*, and with different names and IPs, but the

examples that follow will use the values in Figure 10.2.





129

**Figure 10.2**

The nodes can be virtual machines, physical servers, cloud instances, or Raspberry Pi systems. The only

requirements are that they have Docker installed and can communicate over a reliable network. It’s also beneﬁcial

if name resolution is conﬁgured — it makes it easier to identify nodes in command outputs and helps when

troubleshooting.

On the networking front, you need the following ports open on routers and ﬁrewalls between nodes:

•

•

•

2377/tcp: for secure client-to-swarm communication

7946/tcp and udp: for control plane gossip

4789/udp: for VXLAN-based overlay networks

Docker Desktop for Mac and Windows only supports a single Docker node. You can initialize a single-node swarm

and follow along with most of the examples. Alternatively, you can try Play with Docker at https://labs.play-

with-Docker.com.

Once you’ve satisﬁed the pre-requisites, you can go ahead and build a swarm.

The process of building a swarm is called *initializing a swarm*, and the high-level process is this: Initialize the

ﬁrst manager node > Join additional manager nodes > Join worker nodes > Done.

**Initializing a new swarm**

Docker nodes that are not part of a swarm are said to be in *single-engine mode*. Once they’re added to a swarm

they’re automatically swited into *swarm mode*.

Running docker swarm init on a Docker host in *single-engine mode* will swit that node into *swarm mode*,

create a new *swarm*, and make the node the ﬁrst *manager* of the swarm.

Additional nodes can then be joined to the swarm as workers and managers. Joining a Docker host to an existing

swarm swites them into *swarm mode* as part of the operation.

The following steps will put **mgr1** into *swarm mode* and initialize a new swarm. It will then join **wrk1**, **wrk2**,

and **wrk3** as worker nodes — automatically puing them into *swarm mode* as part of the process. Finally, it will

add **mgr2** and **mgr3** as additional managers and swit them into *swarm mode*. At the end of the procedure all

6 nodes will be in *swarm mode* and operating as part of the same swarm.

This example will use the IP addresses and DNS names of the nodes shown in Figure 10.2. Yours may be different.





130

10: Docker Swarm

\1. Log on to **mgr1** and initialize a new swarm (don’t forget to use backtis instead of backslashes if you’re

following along with Windows in a PowerShell terminal).

$ docker swarm init \

--advertise-addr 10.0.0.1:2377 \

--listen-addr 10.0.0.1:2377

Swarm initialized: current node (d21lyz...c79qzkx) is now a manager.

The command can be broken down as follows:

•

docker swarm init: This tells Docker to initialize a new swarm and make this node the ﬁrst manager.

It also enables swarm mode on the node.

•

--advertise-addr: As the name suggests, this is the swarm API endpoint that will be advertised to

other nodes in the swarm. It will usually be one of the node’s IP addresses, but can be an external

load-balancer address. It’s an optional ﬂag unless you want to specify a load-balancer or speciﬁc IP

address on a node with multiple interfaces.

•

--listen-addr: This is the IP address that the node will accept swarm traffic on. If not explicitly

set, it defaults to the same value as --advertise-addr. If --advertise-addr is a load-balancer, you

must use --listen-addr to specify a local IP or interface for swarm traffic.

I recommend you be speciﬁc and always use both ﬂags.

The default port that swarm mode operates on is **2377**. This is customizable, but it’s convention to use

2377/tcp for secured (HTTPS) client-to-swarm connections.

\2. List the nodes in the swarm.

$ docker node ls

ID

HOSTNAME

STATUS AVAILABILITY MANAGER STATUS

Ready Active Leader

d21...qzkx \* mgr1

Notice that **mgr1** is currently the only node in the swarm, and is listed as the *Leader*. We’ll come back to

this in a second.

\3. From **mgr1** run the docker swarm join-token command to extract the commands and tokens required

to add new workers and managers to the swarm.

$ docker swarm join-token worker

To add a manager to this swarm, run the following command:

docker swarm join \

--token SWMTKN-1-0uahebax...c87tu8dx2c \

10.0.0.1:2377

$ docker swarm join-token manager

To add a manager to this swarm, run the following command:

docker swarm join \

--token SWMTKN-1-0uahebax...ue4hv6ps3p \

10.0.0.1:2377

Notice that the commands to join a worker and a manager are identical apart from the join tokens

(SWMTKN...). This means that whether a node joins as a worker or a manager depends entirely on which

token you use when joining it. **You should ensure that your join tokens are kept secure, as they’re the**

**only thing required to join a node to a swarm!**

\4. Log on to **wrk1** and join it to the swarm using the docker swarm join command with the worker join

token.





131

$ docker swarm join \

--token SWMTKN-1-0uahebax...c87tu8dx2c \

10.0.0.1:2377 \

--advertise-addr 10.0.0.4:2377 \

--listen-addr 10.0.0.4:2377

This node joined a swarm as a worker.

The --advertise-addr, and --listen-addr ﬂags optional. I’ve added them as I consider it best practice

to be as speciﬁc as possible when it comes to network conﬁguration.

\5. Repeat the previous step on **wrk2** and **wrk3** so that they join the swarm as workers. If you’re specifying the

--advertise-addr and --listen-addr ﬂags, make sure you use **wrk2** and **wrk3’s** respective IP addresses.

\6. Log on to **mgr2** and join it to the swarm as a manager using the docker swarm join command with the

manager join token.

$ docker swarm join \

--token SWMTKN-1-0uahebax...ue4hv6ps3p \

10.0.0.1:2377 \

--advertise-addr 10.0.0.2:2377 \

--listen-addr 10.0.0.2:2377

This node joined a swarm as a manager.

\7. Repeat the previous step on **mgr3**, remembering to use **mgr3’s** IP address for the advertise-addr and

--listen-addr ﬂags.

\8. List the nodes in the swarm by running docker node ls from any of the manager nodes in the swarm.

$ docker node ls

ID

HOSTNAME

STATUS AVAILABILITY MANAGER STATUS

0g4rl...babl8 \* mgr2

Ready

Ready

Ready

Ready

Ready

Ready

Active

Active

Active

Active

Active

Active

Reachable

Reachable

2xlti...l0nyp

8yv0b...wmr67

9mzwf...e4m4n

d21ly...9qzkx

e62gf...l5wt6

mgr3

wrk1

wrk3

mgr1

wrk2

Leader

Congratulations. You’ve just created a 6-node swarm with 3 managers and 3 workers. As part of the process, the

Docker Engine on each node was automatically put into *swarm mode* and the *swarm* was automatically secured

with TLS.

If you look in the MANAGER STATUS column you’ll see the three manager nodes are showing as either “Reachable”

or “Leader”. We’ll learn more about leaders shortly. Nodes with nothing in the MANAGER STATUS column are

*workers*. Also note the asterisk (\*) after the ID on the line showing **mgr2**. This tells you which node you are

logged on to and executing commands from. In this instance the command was issued from **mgr2**.

**Note:** It’s a pain to specify the --advertise-addr and --listen-addr ﬂags every time you join a

node to the swarm. However, it can be a much bigger pain if you get the network conﬁguration

of your swarm wrong. Also, manually adding nodes to a swarm is unlikely to be a daily task, so

it’s worth the extra up-front effort to use the ﬂags. It’s your oice though. In lab environments or

nodes with only a single IP you probably don’t need to use them.

Now that you have a *swarm* up and running, let’s take a look at manager high availability (HA).





132

10: Docker Swarm

**Swarm manager high availability (HA)**

So far, we’ve added three manager nodes to a swarm. Why three? And how do they work together?

Swarm *managers* have native support for high availability (HA). This means one or more can fail, and the

survivors will keep the swarm running.

technically speaking, swarm implements a form of active-passive multi-manager HA. This means that although

you have multiple *managers*, only one of them is *active* at any given moment. This active manager is called the

“*leader*”, and the leader is the only one that will ever issue live commands against the *swarm*. So, it’s only ever

the leader that changes the conﬁg, or issues tasks to workers. If a follower manager (passive) receives commands

for the swarm, it proxies them across to the leader.

This process is shown in Figure 10.3. Step 1 is the command coming in to a *manager* from a remote Docker

client. Step 2 is the non-leader manager receiving the command and proxying it to the leader. Step 3 is the leader

executing the command on the swarm.

**Figure 10.3**

If you look closely at Figure 10.3, you’ll notice that managers are either *leaders* or *followers*. This is Ra

terminology, because swarm uses an implementation of the [Ra](https://raft.github.io/)[ ](https://raft.github.io/)[consensus](https://raft.github.io/)[ ](https://raft.github.io/)[algorithm](https://raft.github.io/)[¹⁴](#br143)[ ](#br143)to maintain a consistent

cluster state across multiple highly available managers.

On the topic of HA, the following two best practices apply:

\1. Deploy an odd number of managers.

\2. Don’t deploy too many managers (3 or 5 is recommended)

Having an odd number of *managers* reduces the chances of split-brain conditions. For example, if you had 4

managers and the network partitioned, you could be left with two managers on each side of the partition. This

is known as a split brain — each side knows there used to be 4 but can now only see 2. But crucially, neither

side has any way of knowing if the other two are still alive and whether it holds a majority (quorum). A swarm

cluster continues to operate during split-brain conditions, but you are no longer able to alter the conﬁguration

or add and manage application workloads.

¹⁴<https://raft.github.io/>





133

However, if you have 3 or 5 managers and the same network partition occurs, it is impossible to have an equal

number of managers on both sides of the partition. This means that one side aieves quorum and full cluster

management services remain available. The example on the right side of Figure 10.4 shows a partitioned cluster

where the left side of the split knows it has a majority of managers.

**Figure 10.4**

As with all consensus algorithms, more participants means more time required to aieve consensus. It’s like

deciding where to eat — it’s always quiteer and easier for 3 people to make a quite decision than it is for 33!

With this in mind, it’s a best practice to have either 3 or 5 managers for HA. 7 might work, but it’s generally

accepted that 3 or 5 is optimal. You deﬁnitely don’t want more than 7, as the time taken to aieve consensus

will be longer.

A ﬁnal word of caution regarding manager HA. While it’s obviously a good practice to spread your managers

across availability zones within your network, you need to make sure the networks connecting them are reliable,

as network partitions can be a diﬃcult to troubleshoot and resolve. This means, at the time of writing, the nirvana

of hosting your active production applications and infrastructure across multiple cloud providers such as AWS

and Azure is a bit of a daydream. Take the time and effort to ensure your managers and workers are connected

via reliable high-speed networks.

**Built-in Swarm security**

Swarm clusters have a ton of built-in security that’s conﬁgured out-of-the-box with sensible defaults — CA

settings, join tokens, mutual TLS, encrypted cluster store, encrypted networks, cryptographic node ID’s and

more. See **Chapter 15: Security in Docker** for a detailed look at these.

**Locking a Swarm**

Despite all of this built-in native security, restarting an older manager or restoring an old backup has the potential

to compromise the cluster. Old managers re-joining a swarm automatically decrypt and gain access to the Ra log





134

10: Docker Swarm

time-series database — this can pose security concerns. Restoring old backups can also wipe the current swarm

conﬁguration.

To prevent situations like these, Docker allows you to lo a swarm with the Autolo feature. This forces restarted

managers to present the cluster unlo key before being admied back into the cluster.

It’s possible to apply a lo directly to a new swarm by passing the --autolock ﬂag to the docker swarm init

command. However, we’ve already built a swarm, so we’ll lo our existing swarm with the docker swarm

update command.

Run the following command from a swarm manager.

$ docker swarm update --autolock=true

Swarm updated.

To unlock a swarm manager after it restarts, run the `docker swarm unlock` command and

provide the following key:

SWMKEY-1-5+ICW2kRxPxZrVyBDWzBkzZdSd0Yc7Cl2o4Uuf9NPU4

Please remember to store this key in a password manager, since without it you will not be able

to restart the manager.

Be sure to keep the unlo key in a secure place. You can always check your current swarm unlo key with the

docker swarm unlock-key command.

Restart one of your manager nodes to see if it automatically re-joins the cluster. You may need to prepend the

command with sudo.

$ service docker restart

Try and list the nodes in the swarm.

$ docker node ls

Error response from daemon: Swarm is encrypted and needs to be unlocked before it can be used.

Although the Docker service has restarted on the manager, it has not been allowed to re-join the swarm. You

can prove this even further by running the docker node ls command on another manager node. The restarted

manager will show as down and unreachable.

Use the docker swarm unlock command to unlo the swarm for the restarted manager. You’ll need to run this

command on the restarted manager, and you’ll need to provide the unlo key.

$ docker swarm unlock

Please enter unlock key: <enter your key>

The node will be allowed to re-join the swarm and will show as ready and reachable if you run another docker

node ls.

Loing your swarm and protecting the unlo key is recommended for production environments.

Now that you’ve got our *swarm* built and understand the infrastructure concepts of *leaders* and *manager HA*,

let’s move on to the application aspect of *services*.





135

**Swarm services**

Everything we do in this section of the chapter gets improved on by Docker Stacks in Chapter 14. However, it’s

important that you learn the concepts here so that you’re prepared for Chapter 14.

Like we said in the swarm primer… *services* are a new construct introduced with Docker 1.12, and they only

apply to *swarm mode*.

Services let us specify most of the familiar container options, such as *name, port mappings, attaching to networks,*

and *images*. But they add important cloud-native features, including *desired state* and automatic reconciliation.

For example, swarm services allow us to declaratively deﬁne a desired state for an application that we can apply

to the swarm and let the swarm take care of deploying it and managing it.

Let’s look at a quite example. Assume you have an app with a web front-end. You have an image for the

web server, and testing has shown that you need 5 instances to handle normal daily traffic. You translate this

requirement into a single *service* declaring the image to use, and that the service should always have 5 running

replicas. You issue that to the swarm as your desired state, and the swarm takes care of ensuring there are always

5 instances of the web server running.

We’ll see some of the other things that can be declared as part of a service in a minute, but before we do that,

let’s see one way to create what we just described.

You can create services in one of two ways:

\1. Imperatively on the command line with docker service create

\2. Declaratively with a stack ﬁle

We’ll look at stack ﬁles in a later chapter. For now we’ll focus on the imperative method.

**Note:** The command to create a new service is the same on Windows. However, the image used in

this example is a Linux image and will not work on Windows. You can substitute the image for a

Windows web server image and the command will work. Remember, if you are typing Windows

commands from a PowerShell terminal you will need to use the backti (‘) to indicate continuation

on the next line.

$ docker service create --name web-fe \

-p 8080:8080 \

--replicas 5 \

nigelpoulton/pluralsight-docker-ci

z7ovearqmruwk0u2vc5o7ql0p

Notice that many of the familiar docker container run arguments are the same. In the example, we speciﬁed

--name and -p which work the same for standalone containers as well as services.

Let’s review the command and output.

We used docker service create to tell Docker we are declaring a new service, and we used the --name ﬂag

to name it **web-fe**. We told Docker to map port 8080 on every node in the swarm to 8080 inside of each service

replica. Next, we used the --replicas ﬂag to tell Docker there should always be 5 replicas of this service. Finally,





136

10: Docker Swarm

we told Docker which image to use for the replicas — it’s important to understand that all service replicas use

the same image and conﬁg!

After we hit Return, the command was sent to a manager node, and the manager acting as leader instantiated

5 replicas across the *swarm* — remember that swarm managers also act as workers. Each worker or manager

that received a work task pulled the image and started a container listening on port 8080. The swarm leader also

ensured a copy of the service’s *desired state* was stored on the cluster and replicated to every manager.

But this isn’t the end. All *services* are constantly monitored by the swarm — the swarm runs a background

*reconciliation loop* that constantly compares the *observed state* of the service with the *desired state*. If the two

states mat, the world is a happy place and no further action is needed. If they don’t mat, swarm takes actions

to bring *observed state* into line with *desired state*.

As an example, if a *worker* hosting one of the 5 **web-fe** replicas fails, the *observed state* of the **web-fe** service will

drop from 5 replicas to 4. This will no longer mat the *desired state* of 5, so the swarm will start a new **web-fe**

replica to bring the *observed state* back in line with *desired state*. This behavior is a key tenet of cloud-native

applications and allows the service to self-heal in the event of node failures and the likes.

**Viewing and inspecting services**

You can use the docker service ls command to see a list of all services running on a swarm.

$ docker service ls

ID

NAME

web-fe

MODE

replicated

REPLICAS

5/5

IMAGE

nigel...ci:latest

PORTS

\*:8080->8080/tcp

z7o...uw

The output shows a single running service as well as some basic information about state. Among other things,

you can see the name of the service and that 5 out of the 5 desired replicas are in the running state. If you run

this command soon after deploying the service it might not show all tasks/replicas as running. This is often due

to the time it takes to pull the image on each node.

You can use the docker service ps command to see a list of service replicas and the state of each.

$ docker service ps web-fe

ID

NAME

IMAGE

NODE

mgr2

wrk1

wrk2

mgr3

mgr1

DESIRED

Running

Running

Running

Running

Running

CURRENT

817...f6z

a1d...mzn

cc0...ar0

6f0...azu

dyl...p3e

web-fe.1

web-fe.2

web-fe.3

web-fe.4

web-fe.5

nigelpoulton/...

nigelpoulton/...

nigelpoulton/...

nigelpoulton/...

nigelpoulton/...

Running 2 mins

Running 2 mins

Running 2 mins

Running 2 mins

Running 2 mins

The format of the command is docker service ps <service-name or service-id>. The output displays each

replica (container) on its own line, shows which node in the swarm it’s executing on, and shows desired state

and the current observed state.

For detailed information about a service, use the docker service inspect command.





137

$ docker service inspect --pretty web-fe

ID:

z7ovearqmruwk0u2vc5o7ql0p

Name:

web-fe

Replicated

5

Service Mode:

Replicas:

Placement:

UpdateConfig:

Parallelism:

On failure:

1

pause

Monitoring Period: 5s

Max failure ratio: 0

Update order:

RollbackConfig:

Parallelism:

On failure:

stop-first

1

pause

Monitoring Period: 5s

Max failure ratio: 0

Rollback order:

ContainerSpec:

stop-first

Image:

nigelpoulton/pluralsight-docker-ci:latest@sha256:7a6b01...d8d3d

init: false

Resources:

Endpoint Mode: vip

Ports:

PublishedPort = 8080

Protocol = tcp

TargetPort = 8080

PublishMode = ingress

The example above uses the --pretty ﬂag to limit the output to the most interesting items printed in an easy-

to-read format. Leaving off the --pretty ﬂag will give a more verbose output. I highly recommend you read

through the output of docker inspect commands as they’re a great source of information and a great way to

learn what’s going on under the hood.

We’ll come back to some of these outputs later.

**Replicated vs global services**

The default replication mode of a service is replicated. This deploys a desired number of replicas and distributes

them as evenly as possible across the cluster.

The other mode is global, which runs a single replica on every node in the swarm.

To deploy a *global service* you need to pass the --mode global ﬂag to the docker service create command.

**Scaling a service**

Another powerful feature of *services* is the ability to easily scale them up and down.

Let’s assume business is booming and we’re seeing double the amount of traffic hiing the web front-end.

Fortunately, scaling the **web-fe** service is as simple as running the docker service scale command.





138

10: Docker Swarm

$ docker service scale web-fe=10

web-fe scaled to 10

overall progress: 10 out of 10 tasks

1/10: running

2/10: running

3/10: running

4/10: running

5/10: running

6/10: running

7/10: running

8/10: running

9/10: running

10/10: running

verify: Service converged

This command will scale the number of service replicas from 5 to 10. In the background it’s updating the service’s

*desired state* from 5 to 10. Run another docker service ls command to verify the operation was successful.

$ docker service ls

ID

z7o...uw

NAME

web-fe

MODE

replicated

REPLICAS

10/10

IMAGE

nigel...ci:latest

PORTS

\*:8080->8080/tcp

Running a docker service ps command will show that the service replicas are balanced across all nodes in the

swarm evenly.

$ docker service ps web-fe

ID

NAME

IMAGE

NODE DESIRED CURRENT

nwf...tpn web-fe.1 nigelpoulton/... mgr1 Running Running 7 mins

yb0...e3e web-fe.2 nigelpoulton/... wrk3 Running Running 7 mins

mos...gf6 web-fe.3 nigelpoulton/... wrk2 Running Running 7 mins

utn...6ak web-fe.4 nigelpoulton/... wrk3 Running Running 7 mins

2ge...fyy web-fe.5 nigelpoulton/... mgr3 Running Running 7 mins

64y...m49 web-fe.6 igelpoulton/...

wrk3 Running Running about a min

ild...51s web-fe.7 nigelpoulton/... mgr1 Running Running about a min

vah...rjf web-fe.8 nigelpoulton/... wrk2 Running Running about a mins

xe7...fvu web-fe.9 nigelpoulton/... mgr2 Running Running 45 seconds ago

l7k...jkv web-fe.10 nigelpoulton/... mgr2 Running Running 46 seconds ago

Behind the scenes, swarm runs a seduling algorithm called “spread” that aempts to balance replicas as evenly

as possible across the nodes in the swarm. At the time of writing, this amounts to running an equal number of

replicas on each node without taking into consideration things like CPU load etc.

Run another docker service scale command to bring the number back down from 10 to 5.





139

$ docker service scale web-fe=5

web-fe scaled to 5

overall progress: 5 out of 5 tasks

1/5: running

2/5: running

3/5: running

4/5: running

5/5: running

verify: Service converged

Now that you know how to scale a service, let’s see how to remove one.

**Removing a service**

Removing a service is simple — may be too simple.

The following docker service rm command will delete the service deployed earlier.

$ docker service rm web-fe

web-fe

Conﬁrm it’s gone with the docker service ls command.

$ docker service ls

ID

NAME

MODE

REPLICAS

IMAGE

PORTS

Be careful using the docker service rm command as it deletes all service replicas without asking for

conﬁrmation.

Now that the service is deleted from the system, let’s look at how to push rolling updates to one.

**Rolling updates**

Pushing updates to deployed applications is a fact of life. And for the longest time it was really painful. I’ve lost

more than enough weekends to major application updates, and I’ve no intention of doing it again.

Well… thanks to Docker *services*, pushing updates to well-designed microservices apps is easy.

To see this, we’re going to deploy a new service. But before we do that, we’re going to create a new overlay

network for the service. This isn’t necessary, but I want you to see how it is done and how to attach the service

to it.

$ docker network create -d overlay uber-net

43wfp6pzea470et4d57udn9ws

This creates a new overlay network called “uber-net” that we’ll use for the service we’re about to create. An

overlay network creates a new layer 2 network that we can place containers on, and all containers on it will be

able to communicate. This works even if all of the swarm nodes are on different underlying networks. Basically,





140

10: Docker Swarm

the overlay network creates a new layer 2 container network on top of potentially multiple different underlying

networks.

Figure 10.5 shows four swarm nodes on two underlay networks connected by a layer 3 router. The overlay network

spans all 4 swarm nodes creating a single ﬂat layer 2 network for containers to use.

**Figure 10.5**

Run a docker network ls to verify that the network created properly and is visible on the Docker host.

$ docker network ls

NETWORK ID

<Snip>

43wfp6pzea47

NAME

DRIVER

SCOPE

swarm

uber-net

overlay

The uber-net network was successfully created with the swarm scope and is *currently* only visible on manager

nodes in the swarm. It will be dynamically extended to worker nodes when they run workloads conﬁgured on

the network.

Let’s create a new service and attach it to the network.

$ docker service create --name uber-svc \

--network uber-net \

-p 80:80 --replicas 12 \

nigelpoulton/tu-demo:v1

dhbtgvqrg2q4sg07ttfuhg8nz

overall progress: 12 out of 12 tasks

1/12: running

2/12: running

<Snip>

12/12: running

verify: Service converged

Let’s see what we just declared with that docker service create command.





141

The ﬁrst thing we did was name the service and then use the --network ﬂag to tell it to place all replicas on the

new uber-net network. We then exposed port 80 across the entire swarm and mapped it to port 80 inside of each

of the 12 replicas we asked it to run. Finally, we told it to base all replicas on the nigelpoulton/tu-demo:v1 image.

Run a docker service ls and a docker service ps command to verify the state of the new service.

$ docker service ls

ID

NAME

REPLICAS IMAGE

dhbtgvqrg2q4 uber-svc 12/12

nigelpoulton/tu-demo:v1

$ docker service ps uber-svc

ID

NAME

IMAGE

NODE DESIRED

CURRENT STATE

Running 1 min

Running 1 min

Running 1 min

Running 1 min

Running 1 min

Running 1 min

Running 1 min

Running 1 min

Running 1 min

Running 1 min

Running 1 min

Running 1 min

0v...7e5 uber-svc.1

bh...wa0 uber-svc.2

23...u97 uber-svc.3

82...5y1 uber-svc.4

c3...gny uber-svc.5

e6...3u0 uber-svc.6

78...r7z uber-svc.7

2m...kdz uber-svc.8

b9...k7w uber-svc.9

ag...v16 uber-svc.10

e6...dfk uber-svc.11

e2...k1j uber-svc.12

nigelpoulton/...:v1 wrk3 Running

nigelpoulton/...:v1 wrk2 Running

nigelpoulton/...:v1 wrk2 Running

nigelpoulton/...:v1 mgr2 Running

nigelpoulton/...:v1 wrk3 Running

nigelpoulton/...:v1 wrk1 Running

nigelpoulton/...:v1 wrk1 Running

nigelpoulton/...:v1 mgr3 Running

nigelpoulton/...:v1 mgr3 Running

nigelpoulton/...:v1 mgr2 Running

nigelpoulton/...:v1 mgr1 Running

nigelpoulton/...:v1 mgr1 Running

Passing the service the -p 80:80 ﬂag will ensure that a **swarm-wide** mapping is created that maps all traffic,

coming in to any node in the swarm on port 80, through to port 80 inside of any service replica.

This mode of publishing a port on every node in the swarm — even nodes not running service replicas — is

called *ingress mode* and is the default. The alternative mode is *host mode* which only publishes the service on

swarm nodes running replicas. Publishing a service in *host mode* requires the long-form syntax and looks like

the following:

$ docker service create --name uber-svc \

--network uber-net \

--publish published=80,target=80,mode=host \

--replicas 12 \

nigelpoulton/tu-demo:v1

Open a web browser and point it to the IP address of any of the nodes in the swarm on port 80 to see the service

running.





142

10: Docker Swarm

**Figure 10.6**

As you can see, it’s a simple voting application that will register votes for either “football” or “soccer”. Feel free

to point your web browser to other nodes in the swarm. You’ll be able to reach the web service from any node

because the -p 80:80 ﬂag creates an *ingress mode* mapping on every swarm node. This is true even on nodes

that are not running a replica for the service — **every node gets a mapping and can therefore redirect your**

**request to a node that is running the service**.

Let’s now assume that this particular vote has come to an end and your company wants to run a new poll. A

new container image has been created for the new poll and has been added to the same Docker Hub repository,

but this one is tagged as v2 instead of v1.

Let’s also assume that you’ve been tasked with pushing the updated image to the swarm in a staged manner —

2 replicas at a time with a 20 second delay between each. You can use the following docker service update

command to accomplish this.

$ docker service update \

--image nigelpoulton/tu-demo:v2 \

--update-parallelism 2 \

--update-delay 20s uber-svc

overall progress: 4 out of 12 tasks

1/12: running

2/12: running

3/12: running

4/12: running

5/12: starting

6/12: ready

<Snip>

12/12:

Let’s review the command. docker service update lets us make updates to running services by updating the

service’s desired state. This example speciﬁes a new version of the image, tagged as v2 instead of v1. It also





143

speciﬁed the --update-parallelism and --update-delay ﬂags to make sure that the new image was pushed to

2 replicas at a time with a 20 second cool-off period in between each set of two. Finally, it instructs the swarm to

make the changes to the uber-svc service.

If you run a docker service ps uber-svc while the update is in progress, some of the replicas will be at v2

while some will still be at v1. If you give the operation enough time to complete (4 minutes), all replicas will

eventually reach the new desired state of using the v2 image.

$ docker service ps uber-svc

ID

NAME

IMAGE

nigel...v2

NODE DESIRED

mgr2 Running

CURRENT STATE

Running 13 secs

7z...nys uber-svc.1

0v...7e5 \\_uber-svc.1 nigel...v1

wrk3 Shutdown Shutdown 13 secs

bh...wa0 uber-svc.2

e3...gr2 uber-svc.3

nigel...v1

nigel...v2

wrk2 Running

wrk2 Running

Running 1 min

Running 13 secs

23...u97 \\_uber-svc.3 nigel...v1

wrk2 Shutdown Shutdown 13 secs

82...5y1 uber-svc.4

c3...gny uber-svc.5

e6...3u0 uber-svc.6

78...r7z uber-svc.7

2m...kdz uber-svc.8

b9...k7w uber-svc.9

ag...v16 uber-svc.10

e6...dfk uber-svc.11

e2...k1j uber-svc.12

nigel...v1

nigel...v1

nigel...v1

nigel...v1

nigel...v1

nigel...v1

nigel...v1

nigel...v1

nigel...v1

mgr2 Running

wrk3 Running

wrk1 Running

wrk1 Running

mgr3 Running

mgr3 Running

mgr2 Running

mgr1 Running

mgr1 Running

Running 1 min

Running 1 min

Running 1 min

Running 1 min

Running 1 min

Running 1 min

Running 1 min

Running 1 min

Running 1 min

You can witness the update happening in real-time by opening a web browser to any node in the swarm and

hiing refresh several times. Some of the requests will be serviced by replicas running the old version and some

will be serviced by replicas running the new version. After enough time, all requests will be serviced by replicas

running the updated version of the service.

Congratulations. You’ve just pushed a rolling update to a live containerized application. Remember, Docker Stacks

take all of this to the next level in Chapter 14.

If you run a docker inspect --pretty command against the service, you’ll see the update parallelism and

update delay settings are now part of the service deﬁnition. This means future updates will automatically use

these settings unless you override them as part of the docker service update command.

$ docker service inspect --pretty uber-svc

ID:

mub0dgtc8szm80ez5bs8wlt19

Name:

uber-svc

Replicated

12

Service Mode:

Replicas:

UpdateStatus:

State:

updating

Started:

Message:

About a minute

update in progress

Placement:

UpdateConfig:

Parallelism:

Delay:

2

20s

pause

On failure:





144

10: Docker Swarm

Monitoring Period: 5s

Max failure ratio: 0

Update order:

RollbackConfig:

Parallelism:

On failure:

stop-first

1

pause

Monitoring Period: 5s

Max failure ratio: 0

Rollback order:

ContainerSpec:

stop-first

Image:

nigelpoulton/tu-demo:v2@sha256:d3c0d8c9...cf0ef2ba5eb74c

init: false

Resources:

Networks: uber-net

Endpoint Mode: vip

Ports:

PublishedPort = 80

Protocol = tcp

TargetPort = 80

PublishMode = ingress

You should also note a couple of things about the service’s network conﬁg. All nodes in the swarm that are

running a replica for the service will have the uber-net overlay network that we created earlier. We can verify

this by running docker network ls on any node running a replica.

You should also note the Networks portion of the docker inspect output. This shows the uber-net network as

well as the swarm-wide 80:80 port mapping.

**Troubleshooting**

Swarm Service logs can be viewed with the docker service logs command. However, not all logging drivers

support the command.

By default, Docker nodes conﬁgure services to use the json-file log driver, but other drivers exist, including:

•

•

•

•

journald (only works on Linux hosts running systemd)

syslog

splunk

gelf

json-file and journald are the easiest to conﬁgure, and both work with the docker service logs command.

The format of the command is docker service logs <service-name>.

If you’re using 3rd-party logging drivers you should view those logs using the logging platform’s native tools.

The following snippet from a daemon.json conﬁguration ﬁle shows a Docker host conﬁgured to use syslog.





145

{

}

"log-driver": "syslog"

You can force individual services to use a different driver by passing the --log-driver and --log-opts ﬂags to

the docker service create command. These will override anything set in daemon.json.

Service logs work on the premise that your application is running as PID 1 in its container and sending logs to

STDOUT and errors to STDERR. The logging driver forwards these “logs” to the locations conﬁgured via the logging

driver.

The following docker service logs command shows the logs for all replicas in the svc1 service that experienced

a couple of failures starting a replica.

$ docker service logs svc1

svc1.1.zhc3cjeti9d4@wrk-2 | [emerg] 1#1: host not found...

svc1.1.zhc3cjeti9d4@wrk-2 | nginx: [emerg] host not found..

svc1.1.6m1nmbzmwh2d@wrk-2 | [emerg] 1#1: host not found...

svc1.1.6m1nmbzmwh2d@wrk-2 | nginx: [emerg] host not found..

svc1.1.1tmya243m5um@mgr-1 | 10.255.0.2 "GET / HTTP/1.1" 302

The output is trimmed to ﬁt the page, but you can see that logs from all three service replicas are shown (the two

that failed and the one that’s running). Each line starts with the name of the replica, which includes the service

name, replica number, replica ID, and name of host that it’s seduled on. Following that is the log output.

It’s hard to tell because it’s trimmed to ﬁt the book, but it looks like the ﬁrst two replicas failed because they were

trying to connect to another service that was still starting (a sort of race condition when dependent services are

starting).

You can follow the logs (--follow), tail them (--tail), and get extra details (--details).

**Backing up Swarm**

Baing up a swarm will backup the control plane objects required to recover the swarm in the event of a

catastrophic failure of corruption. Recovering a swarm from a backup is an extremely rare scenario. However,

business critical environments should always be prepared for worst-case scenarios.

You might be asking why backups are necessary if the control plane is already replicated and highly-available

(HA). To answer that question, consider the scenario where a malicious actor deletes all of the Secrets on a

swarm. HA cannot help in this scenario as the Secrets will be deleted from the cluster store that is automatically

replicated to all manager nodes. In this scenario the highly-available replicated cluster store works against you —

quitely propagating the delete operation. In this scenario you can either recreate the deleted objects from copies

kept in a source code repo, or you can aempt to recover your swarm from a recent backup.

Managing your swarm and applications declaratively is a great way to prevent the need to recover from a

backup. For example, storing conﬁguration objects outside of the swarm in a source code repository will enable

you to redeploy things like networks, services, secrets and other objects. However, managing your environment

declaratively and strictly using source control repos requires discipline.

Anyway, let’s see how to **backup a swarm**.





146

10: Docker Swarm

Swarm conﬁguration and state is stored in /var/lib/docker/swarm on every manager node. The conﬁguration

includes; Ra log keys, overlay networks, Secrets, Conﬁgs, Services, and more. A swarm backup is a copy of all

the ﬁles in this directory.

As the contents of this directory are replicated to all managers, you can, and should, perform backups from

multiple managers. However, as you have to stop the Docker daemon on the node you are backing up, it’s a good

idea to perform the backup from non-leader managers. This is because stopping Docker on the leader will initiate

a leader election. You should also perform the backup at a quiet time for the business, as stopping a manager can

increase the risk of the swarm losing quorum if another manager fails during the backup.

The procedure we’re about to follow is designed for demonstration purposes and you’ll need to tweak it for

your production environment. It also creates a couple of swarm objects so that a later step can prove the restore

operation worked.

**Warning**: The following operation carries risks. You should also ensure you perform test backup

and restore operations regularly and test the outcomes.

The following commands will create the following two objects so you can prove the restore operation:

• An overlay network called “Unimatrix-01”

• A Secret called “missing drones” containing the text “Seven of Nine”

$ docker network create -d overlay Unimatrix-01

w9l904ff73e7stly0gnztsud7

$ printf "Seven of Nine" | docker secret create missing\_drones -

i8oj3b2lid27t5202uycw37lg

Let’s perform the swarm backup.

\1. Stop Docker on a non-leader swarm manager.

If you have any containers or service tasks running on the node, this action may stop them.

$ service docker stop

\2. Baup the Swarm conﬁg.

This example uses the Linux tar utility to perform the ﬁle copy that will be the backup. Feel free to use a

different tool.

$ tar -czvf swarm.bkp /var/lib/docker/swarm/

tar: Removing leading `/' from member names

/var/lib/docker/swarm/

/var/lib/docker/swarm/docker-state.json

/var/lib/docker/swarm/state.json

<Snip>

\3. Verify the backup ﬁle exists.





147

$ ls -l

-rw-r--r-- 1 root

root

450727 May 4 14:06 swarm.bkp

In the real world you should store and rotate this backup in accordance with any corporate backup policies.

At this point, the swarm is backed up and you can restart Docker on the node.

\4. Restart Docker.

$ service docker restart

Now that you have a backup, let’s perform a test restore. The steps in this procedure demonstrate the operation.

Performing a restore in the real world may be slightly different, but the overall process will be similar.

**Note**: You do not have to perform a restore operation if your swarm is still running and you only

wish to add a new manager node. In this situation just add a new manager. A swarm restore is

only for situations where the swarm is corrupted or otherwise lost and you cannot recover services

from copies of conﬁg ﬁles stored in a source code repo.

We’ll use the swarm.bkp ﬁle from earlier to restore the swarm. **All swarm nodes must have their Docker**

**daemon stopped and the contents of their /var/lib/docker/swarm directories deleted.**

The following must also be true for a recovery operation to work:

\1. You can only restore to a node running the same version of Docker the backup was performed on

\2. You can only restore to a node with the same IP address as the node the backup was performed on

Perform the following tasks from the swarm manager node that you wish to recover. Remember that Docker

must be stopped and the contents of /var/lib/docker/swarm must be deleted.

\1. Restore the Swarm conﬁguration from backup.

In this example, we’ll restore from a zipped tar ﬁle called swarm.bkp. Restoring to the root directory

is required with this command as it will include the full path to the original ﬁles as part of the extract

operation. This may be different in your environment.

$ tar -zxvf swarm.bkp -C /

\2. Start Docker. The method for starting Docker can vary between environments.

$ service docker start

\3. Initialize a new Swarm cluster.

Remember, you are not recovering a manager and adding it back to a working cluster. This operation is

to recover a failed swarm that has no surviving managers. The --force-new-cluster ﬂag tells Docker

to create a new cluster using the conﬁguration stored in /var/lib/docker/swarm/ that you recovered in

step 1.





148

10: Docker Swarm

$ docker swarm init --force-new-cluster

Swarm initialized: current node (jhsg...3l9h) is now a manager.

\4. Check that the network and service were recovered as part of the operation.

$ docker network ls

NETWORK ID

z21s5v82by8q

NAME

Unimatrix-01

DRIVER

overlay

SCOPE

swarm

$ docker secret ls

ID

NAME

DRIVER

i8oj3b2lid27t5202uycw37lg

missing\_drones

Congratulations. The Swarm is recovered.

\5. Add new manager and worker nodes and take fresh backups.

Remember, test this procedure regularly and thoroughly. You do not want it to fail when you need it most!

**Docker Swarm - The Commands**

•

•

docker swarm init is the command to create a new swarm. The node that you run the command on

becomes the ﬁrst manager and is swited to run in *swarm mode*.

docker swarm join-token reveals the commands and tokens needed to join workers and managers to

existing swarms. To expose the command to join a new manager, use the docker swarm join-token

manager command. To get the command to join a worker, use the docker swarm join-token worker

command.

•

•

•

docker node ls lists all nodes in the swarm including which are managers and which is the leader.

docker service create is the command to create a new service.

docker service ls lists running services in the swarm and gives basic info on the state of the service

and any replicas it’s running.

•

•

docker service ps <service> gives more detailed information about individual service replicas.

docker service inspect gives very detailed information on a service. It accepts the --pretty ﬂag to

limit the information returned to the most important information.

•

•

•

•

docker service scale lets you scale the number of replicas in a service up and down.

docker service update lets you update many of the properties of a running service.

docker service logs lets you view the logs of a service.

docker service rm is the command to delete a service from the swarm. Use it with caution as it deletes

all service replicas without asking for conﬁrmation.

**Chapter summary**

Docker Swarm is Docker’s native technology for managing clusters of Docker nodes and deploying and managing

cloud-native applications. It is similar to Kubernetes.

At its core, Swarm has a secure clustering component, and an orchestration component.





149

The secure clustering component is enterprise-grade and offers a wealth of security and HA features that are

automatically conﬁgured and extremely simple to modify.

The orchestration component allows you to deploy and manage cloud-native microservices applications in a

simple declarative manner.

We’ll dig deeper into deploying cloud-native microservices apps in a declarative manner in Chapter 14.









**11: Docker Networking**

It’s always the network!

Any time there’s a an infrastructure problem, we always blame the network. Part of the reason is that networks

are at the center of everything — **no network, no app!**

In the early days of Docker, networking was hard — really hard. These days, it’s *almost* a pleasure ;-)

In this chapter, we’ll look at the fundamentals of Docker networking. Things like the Container Network Model

(CNM) and libnetwork. We’ll also get our hands dirty building some networks.

As usual, we’ll split the chapter into three parts:

• The TLDR

• The deep dive

• The commands

**Docker Networking - The TLDR**

Docker runs applications inside of containers, and applications need to communicate over lots of different

networks. This means Docker needs strong networking capabilities.

Fortunately, Docker has solutions for container-to-container networks, as well as connecting to existing networks

and VLANs. The laer is important for containerized apps that interact with functions and services on external

systems such as VM’s and physical servers.

Docker networking is based on an open-source pluggable architecture called the Container Network Model

(CNM). libnetwork is Docker’s real-world implementation of the CNM, and it provides all of Docker’s core

networking capabilities. Drivers plug in to libnetwork to provide speciﬁc network topologies.

To create a smooth out-of-the-box experience, Docker ships with a set of native drivers that deal with the most

common networking requirements. These include single-host bridge networks, multi-host overlays, and options

for plugging into existing VLANs. Ecosystem partners can extend things further by providing their own drivers.

Last but not least, libnetwork provides a native service discovery and basic container load balancing solution.

That's this big picture. Let’s get into the detail.

**Docker Networking - The Deep Dive**

We’ll organize this section of the chapter as follows:

• The theory

• Single-host bridge networks

• Multi-host overlay networks

• Connecting to existing networks

• Service Discovery

• Ingress networking





152

11: Docker Networking

**The theory**

At the highest level, Docker networking comprises three major components:

• The Container Network Model (CNM)

•

libnetwork

• Drivers

The CNM is the design speciﬁcation. It outlines the fundamental building blocks of a Docker network.

libnetwork is a real-world implementation of the CNM, and is used by Docker. It’s wrien in Go, and implements

the core components outlined in the CNM.

Drivers extend the model by implementing speciﬁc network topologies such as VXLAN overlay networks.

Figure 11.1 shows how they ﬁt together at a very high level.

**Figure 11.1**

Let’s look a bit closer at each.

**The Container Network Model (CNM)**

Everything starts with a design.

The design guide for Docker networking is the CNM. It outlines the fundamental building blocks of a Docker

network, and you can read the full spec here: https://github.com/Docker/libnetwork/blob/master/docs/design.md

I recommend reading the entire spec, but at a high level, it deﬁnes three major building blocks:

• Sandboxes

• Endpoints

• Networks

A **sandbox** is an isolated network stack. It includes; Ethernet interfaces, ports, routing tables, and DNS conﬁg.

**Endpoints** are virtual network interfaces (E.g. veth). Like normal network interfaces, they’re responsible for

making connections. In the case of the CNM, it’s the job of the *endpoint* to connect a *sandbox* to a *network*.





153

**Networks** are a software implementation of an swit (802.1d bridge). As such, they group together and isolate

a collection of endpoints that need to communicate.

Figure 11.2 shows the three components and how they connect.

**Figure 11.2 The Container Network Model (CNM)**

The atomic unit of seduling in a Docker environment is the container, and as the name suggests, the Container

Network Model is all about providing networking to containers. Figure 11.3 shows how CNM components relate

to containers — sandboxes are placed inside of containers to provide network connectivity.

**Figure 11.3**

Container A has a single interface (endpoint) and is connected to Network A. Container B has two interfaces

(endpoints) and is connected to Network A **and** Network B. The two containers will be able to communicate

because they are both connected to Network A. However, the two *endpoints* in Container B cannot communicate

with each other without the assistance of a layer 3 router.

It’s also important to understand that *endpoints* behave like regular network adapters, meaning they can only

be connected to a single network. Therefore, if a container needs connecting to multiple networks, it will need

multiple endpoints.

Figure 11.4 extends the diagram again, this time adding a Docker host. Although Container A and Container B

are running on the same host, their network stacks are completely isolated at the OS-level via the sandboxes.





154

11: Docker Networking

**Figure 11.4**

**Libnetwork**

The CNM is the design doc, and libnetwork is the canonical implementation. It’s open-source, wrien in Go,

cross-platform (Linux and Windows), and used by Docker.

In the early days of Docker, all the networking code existed inside the daemon. This was a nightmare — the

daemon became bloated, and it didn’t follow the Unix principle of building modular tools that can work on

their own, but also be easily composed into other projects. As a result, it all got ripped out and refactored into

an external library called libnetwork based on the principles of the CNM. Nowadays, all of the core Docker

networking code lives in libnetwork.

As you’d expect, it implements all three of the components deﬁned in the CNM. It also implements native

*service discovery*, *ingress-based container load balancing*, and the network control plane and management plane

functionality.

**Drivers**

If libnetwork implements the control plane and management plane functions, then drivers implement the data

plane. For example, connectivity and isolation is all handled by drivers. So is the actual creation of networks. The

relationship is shown in Figure 11.5.





155

**Figure 11.5**

Docker ships with several built-in drivers, known as native drivers or *local drivers*. On Linux they include; bridge,

overlay, and macvlan. On Windows they include; nat, overlay, transparent, and l2bridge. We’ll see how to

use some of them later in the chapter.

3rd-parties can also write Docker network drivers known as *remote drivers* or plugins. Weave Net is a popular

example and can be downloaded from Docker Hub.

Each driver is in arge of the actual creation and management of all resources on the networks it is responsible

for. For example, an overlay network called “prod-fe-cuda” will be owned and managed by the overlay driver.

This means the overlay driver will be invoked for the creation, management, and deletion of all resources on

that network.

In order to meet the demands of complex highly-ﬂuid environments, libnetwork allows multiple network drivers

to be active at the same time. This means your Docker environment can sport a wide range of heterogeneous

networks.

**Single-host bridge networks**

The simplest type of Docker network is the single-host bridge network.

The name tells us two things:

• **Single-host** tells us it only exists on a single Docker host and can only connect containers that are on the

same host.

• **Bridge** tells us that it’s an implementation of an 802.1d bridge (layer 2 swit).

Docker on Linux creates single-host bridge networks with the built-in bridge driver, whereas Docker on Windows

creates them using the built-in nat driver. For all intents and purposes, they work the same.

Figure 11.6 shows two Docker hosts with identical local bridge networks called “mynet”. Even though the

networks are identical, they are independent isolated networks. This means the containers in the picture cannot

communicate directly because they are on different networks.





156

11: Docker Networking

**Figure 11.6**

Every Docker host gets a default single-host bridge network. On Linux it’s called “bridge”, and on Windows it’s

called “nat” (yes, those are the same names as the drivers used to create them). By default, this is the network

that all new containers will be connected to unless you override it on the command line with the --network ﬂag.

The following listing shows the output of a docker network ls command on newly installed Linux and Windows

Docker hosts. The output is trimmed so that it only shows the default network on each host. Notice how the name

of the network is the same as the driver that was used to create it — this is a coincidence and not a requirement.

//Linux

$ docker network ls

NETWORK ID

333e184cd343

NAME

bridge

DRIVER

bridge

SCOPE

local

//Windows

\> docker network ls

NETWORK ID

095d4090fa32

NAME

nat

DRIVER

nat

SCOPE

local

The docker network inspect command is a treasure trove of great information. I highly recommended reading

through its output if you’re interested in low-level detail.

docker network inspect bridge

[

{

"Name": "bridge",

<< Will be nat on Windows

"Id": "333e184...d9e55",

"Created": "2018-01-15T20:43:02.566345779Z",

"Scope": "local",

"Driver": "bridge",

"EnableIPv6": false,

"IPAM": {

<< Will be nat on Windows

"Driver": "default",

"Options": null,

"Config": [

{





157

"Subnet": "172.17.0.0/16"

}

]

},

"Internal": false,

"Attachable": false,

"Ingress": false,

"ConfigFrom": {

"Network": ""

},

<Snip>

}

]

Docker networks built with the bridge driver on Linux hosts are based on the bale-hardened *linux bridge*

technology that has existed in the Linux kernel for nearly 20 years. This means they’re high performance and

extremely stable. It also means you can inspect them using standard Linux utilities. For example.

$ ip link show docker0

3: docker0: <BROADCAST,MULTICAST,UP,LOWER\_UP> mtu 1500 qdisc...

link/ether 02:42:af:f9:eb:4f brd ff:ff:ff:ff:ff:ff

The default “bridge” network, on all Linux-based Docker hosts, maps to an underlying *Linux bridge* in the kernel

called “**Docker0**”. We can see this from the output of docker network inspect.

$ docker network inspect bridge | grep bridge.name

"com.docker.network.bridge.name": "docker0",

The relationship between Docker’s default “bridge” network and the “Docker0” bridge in the Linux kernel is shown

in Figure 11.7.

**Figure 11.7**

Figure 11.8 extends the diagram by adding containers at the top that plug into the “bridge” network. The “bridge”

network maps to the “Docker0” Linux bridge in the host’s kernel, which can be mapped back to an Ethernet

interface on the host via port mappings.





158

11: Docker Networking

**Figure 11.8**

Let’s use the docker network create command to create a new single-host bridge network called “localnet”.

//Linux

$ docker network create -d bridge localnet

//Windows

\> docker network create -d nat localnet

The new network is created and will appear in the output of any future docker network ls commands. If you

are using Linux, you will also have a new *Linux bridge* created in the kernel.

Let’s use the Linux brctl tool to look at the Linux bridges currently on the system. You may have to manually

install the brctl binary using apt-get install bridge-utils, or the equivalent for your Linux distro.

$ brctl show

bridge name

docker0

br-20c2e8ae4bbb

bridge id

8000.0242aff9eb4f

8000.02429636237c

STP enabled

no

no

interfaces

The output shows two bridges. The ﬁrst line is the “Docker0” bridge that we already know about. This relates

to the default “bridge” network in Docker. The second bridge (br-20c2e8ae4bbb) relates to the new localnet

Docker bridge network. Neither of them have spanning tree enabled, and neither have any devices connected

(interfaces column).

At this point, the bridge conﬁguration on the host looks like Figure 11.9.





159

**Figure 11.9**

Let’s create a new container and attach it to the new localnet bridge network. If you’re following along

on Windows, you should substitute “alpine sleep 1d” with “mcr.microsoft.com/powershell:nanoserver

pwsh.exe -Command Start-Sleep 86400”.

$ docker container run -d --name c1 \

--network localnet \

alpine sleep 1d

This container will now be on the localnet network. You can conﬁrm this with a docker network inspect.

$ docker network inspect localnet --format '{{json .Containers}}'

{

"4edcbd...842c3aa": {

"Name": "c1",

"EndpointID": "43a13b...3219b8c13",

"MacAddress": "02:42:ac:14:00:02",

"IPv4Address": "172.20.0.2/16",

"IPv6Address": ""

}

},

The output shows that the new “c1” container is on the localnet bridge/nat network.

It you run the Linux brctl show command again, you’ll see c1’s interface attached to the br-20c2e8ae4bbb

bridge.

$ brctl show

bridge name

br-20c2e8ae4bbb

docker0

bridge id

8000.02429636237c

8000.0242aff9eb4f

STP enabled

no

no

interfaces

vethe792ac0

This is shown in Figure 11.10.





160

11: Docker Networking

**Figure 11.10**

If we add another new container to the same network, it should be able to ping the “c1” container by name. This is

because all new containers are automatically registered with the embedded Docker DNS service, enabling them

to resolve the names of all other containers on the same network.

**Beware:** The default bridge network on Linux does not support name resolution via the Docker

DNS service. All other *user-deﬁned* bridge networks do. The following demo will work because

the container is on the user-deﬁned localnet network.

Let’s test it.

\1. Create a new interactive container called “c2” and put it on the same localnet network as “c1”.

//Linux

$ docker container run -it --name c2 \

--network localnet \

alpine sh

//Windows

\> docker container run -it --name c2 `

--network localnet `

mcr.microsoft.com/powershell:nanoserver

Your terminal will swit into the “c2” container.

\2. From within the “c2” container, ping the “c1” container by name.

\> ping c1

Pinging c1 [172.26.137.130] with 32 bytes of data:

Reply from 172.26.137.130: bytes=32 time=1ms TTL=128

Reply from 172.26.137.130: bytes=32 time=1ms TTL=128

Control-C

It works! This is because the c2 container is running a local DNS resolver that forwards requests to an

internal Docker DNS server. This DNS server maintains mappings for all containers started with the

--name or --net-alias ﬂag.

Try running some network-related commands while you’re still logged on to the container. It’s a great way

of learning more about how Docker container networking works. The following snippet shows the ipconfig

command ran from inside the “c2” Windows container previously created. You can Ctrl+P+Q out of the container

and run another docker network inspect localnet command to mat the IP addresses.





161

PS C:\> ipconfig

Windows IP Configuration

Ethernet adapter Ethernet:

Connection-specific DNS Suffix . :

Link-local IPv6 Address . . . . . : fe80::14d1:10c8:f3dc:2eb3%4

IPv4 Address. . . . . . . . . . . : 172.26.135.0

Subnet Mask . . . . . . . . . . . : 255.255.240.0

Default Gateway . . . . . . . . . : 172.26.128.1

So far, we’ve said that containers on bridge networks can only communicate with other containers on the same

network. However, you can get around this using *port mappings*.

Port mappings let you map a container to a port on the Docker host. Any traffic hiing the Docker host on the

conﬁgured port will be directed to the container. The high-level ﬂow is shown in Figure 11.11

**Figure 11.11**

In the diagram, the application running in the container is operating on port 80. This is mapped to port 5000 on

the host’s 10.0.0.15 interface. The end result is all traffic hiing the host on 10.0.0.15:5000 being redirected

to the container on port 80.

Let’s walk through an example of mapping port 80 on a container running a web server, to port 5000 on the Docker

host. The example will use NGINX on Linux. If you’re following along on Windows, you’ll need to substitute

nginx with a Windows-based web server image such as mcr.microsoft.com/windows/servercore/iis:nanoserver.

\1. Run a new web server container and map port 80 on the container to port 5000 on the Docker host.





162

11: Docker Networking

$ docker container run -d --name web \

--network localnet \

--publish 5000:80 \

nginx

\2. Verify the port mapping.

$ docker port web

80/tcp -> 0.0.0.0:5000

This shows that port 80 in the container is mapped to port 5000 on all interfaces on the Docker host.

\3. Test the conﬁguration by pointing a web browser to port 5000 on the Docker host. To complete this step,

you’ll need to know the IP or DNS name of your Docker host. If you’re using Docker Desktop on Mac or

Windows, you’ll be able to use localhost:5000 or 127.0.0.1:5000.

**Figure 11.12**

Any external system can now access the NGINX container running on the localnet bridge network via

a port mapping to TCP port 5000 on the Docker host.

Mapping ports like this works, but it’s clunky and doesn’t scale. For example, only a single container can bind to

any port on the host. This means no other containers on that host will be able to bind to port 5000. This is one of

the reason’s that single-host bridge networks are only useful for local development and very small applications.

**Multi-host overlay networks**

We’ve got an entire chapter dedicated to multi-host overlay networks. So we’ll keep this section short.

Overlay networks are multi-host. They allow a single network to span multiple hosts so that containers on

different hosts can communicate directly. They’re ideal for container-to-container communication, including

container-only applications, and they scale well.

Docker provides a native driver for overlay networks. This makes creating them as simple as adding the --d

overlay ﬂag to the docker network create command.





163

**Connecting to existing networks**

The ability to connect containerized apps to external systems and physical networks is vital. A common example

is a partially containerized app — the containerized parts need a way to communicate with the non-containerized

parts still running on existing physical networks and VLANs.

The built-in MACVLAN driver (transparent on Windows) was created with this in mind. It makes containers ﬁrst-

class citizens on the existing physical networks by giving each one its own MAC address and IP addresses. We

show this in Figure 11.13.

**Figure 11.13**

On the positive side, MACVLAN performance is good as it doesn’t require port mappings or additional bridges —

you connect the container interface through to the hosts interface (or a sub-interface). However, on the negative

side, it requires the host NIC to be in **promiscuous mode**, which isn’t always allowed on corporate networks and

public cloud platforms. So MACVLAN is great for your corporate data center networks (assuming your network

team can accommodate promiscuous mode), but it might not work in the public cloud.

Let’s dig a bit deeper with the help of some pictures and a hypothetical example.

Assume we have an existing physical network with two VLANS:

• VLAN 100: 10.0.0.0/24

• VLAN 200: 192.168.3.0/24

**Figure 11.14**

Next, we add a Docker host and connect it to the network.





164

11: Docker Networking

**Figure 11.15**

We then have a requirement for a container running on that host to be plumbed into VLAN 100. To do this,

we create a new Docker network with the macvlan driver. However, the macvlan driver needs us to tell it a few

things about the network we’re going to associate it with. Things like:

• Subnet info

• Gateway

• Range of IP’s it can assign to containers

• Whi interface or sub-interface on the host to use

The following command will create a new MACVLAN network called “macvlan100” that will connect containers

to VLAN 100.

$ docker network create -d macvlan \

--subnet=10.0.0.0/24 \

--ip-range=10.0.0.0/25 \

--gateway=10.0.0.1 \

-o parent=eth0.100 \

macvlan100

This will create the “macvlan100” network and the eth0.100 sub-interface. The conﬁg now looks like this.





165

**Figure 11.16**

MACVLAN uses standard Linux sub-interfaces, and you have to tag them with the ID of the VLAN they will

connect to. In this example we’re connecting to VLAN 100, so we tag the sub-interface with .100 (etho.100).

We also used the --ip-range ﬂag to tell the MACVLAN network which sub-set of IP addresses it can assign to

containers. It’s vital that this range of addresses be reserved for Docker and not in use by other nodes or DHCP

servers, as there is no management plane feature to check for overlapping IP ranges.

The macvlan100 network is ready for containers, so let’s deploy one with the following command.

$ docker container run -d --name mactainer1 \

--network macvlan100 \

alpine sleep 1d

The conﬁg now looks like Figure 11.17. But remember, the underlying network (VLAN 100) does not see any

of the MACVLAN magic, it only sees the container with its MAC and IP addresses. And with that in mind, the

“mactainer1” container will be able to ping and communicate with any other systems on VLAN 100. Prey sweet!

**Note:** If you can’t get this to work, it might be because the host NIC is not in promiscuous mode.

Remember that public cloud platforms don’t usually allow promiscuous mode.





166

11: Docker Networking

**Figure 11.17**

At this point, we’ve got a MACVLAN network and used it to connect a new container to an existing VLAN.

However, it doesn’t stop there. The Docker MACVLAN driver is built on top of the tried-and-tested Linux kernel

driver with the same name. As such, it supports VLAN trunking. This means we can create multiple MACVLAN

networks and connect containers on the same Docker host to them as shown in Figure 11.18.





167

**Figure 11.18**

That prey much covers MACVLAN. Windows offers a similar solution with the transparent driver.

**Container and Service logs for troubleshooting**

A quite note on troubleshooting connectivity issues before moving on to Service Discovery.

If you think you’re experiencing connectivity issues between containers, it’s worth checking the Docker daemon

logs as well as container logs.

On Windows systems, the daemon logs are stored under ∼AppData\Local\Docker, and you can view them in

the Windows Event Viewer. On Linux, it depends what init system you’re using. If you’re running a systemd,

the logs will go to journald and you can view them with the journalctl -u docker.service command. If

you’re not running systemd you should look under the following locations:

• Ubuntu systems running upstart: /var/log/upstart/docker.log

• RHEL-based systems: /var/log/messages

• Debian: /var/log/daemon.log

You can also tell Docker how verbose you want daemon logging to be. To do this, edit the daemon conﬁg ﬁle

(daemon.json) so that “debug” is set to “true” and “log-level” is set to one of the following:

•

•

•

debug The most verbose option

info The default value and second-most verbose option

warn ird most verbose option





168

11: Docker Networking

•

•

error Fourth most verbose option

fatal Least verbose option

The following snippet from a daemon.json enables debugging and sets the level to debug. It will work on all

Docker platforms.

{

<Snip>

"debug":true,

"log-level":"debug",

<Snip>

}

Be sure to restart Docker after making changes to the ﬁle.

That was the daemon logs. What about container logs?

Logs from standalone containers can be viewed with the docker container logs command, and Swarm service

logs can be viewed with the docker service logs command. However, Docker supports lots of logging drivers,

and they don’t all work with the docker logs command.

As well as a driver and conﬁguration for daemon logs, every Docker host has a default logging driver and

conﬁguration for containers. Some of the drivers include:

•

•

•

•

•

json-file (default)

journald (only works on Linux hosts running systemd)

syslog

splunk

gelf

json-file and journald are probably the easiest to conﬁgure, and they both work with the docker logs and

docker service logs commands. The format of the commands is docker logs <container-name> and docker

service logs <service-name>.

If you’re using other logging drivers you can view logs using the 3-rd party platform’s native tools.

The following snippet from a daemon.json shows a Docker host conﬁgured to use syslog.

{

"log-driver": "syslog"

}

You can conﬁgure an individual container, or service, to start with a particular logging driver with the

--log-driver and --log-opts ﬂags. These will override anything set in daemon.json.

Container logs work on the premise that your application is running as PID 1 inside the container and sending

logs to STDOUT, and errors to STDERR. The logging driver then forwards these “logs” to the locations conﬁgured

via the logging driver.

If your application logs to a ﬁle, it’s possible to use a symlink to redirect log-ﬁle writes to STDOUT and STDERR.

The following is an example of running the docker logs command against a container called “vantage-db”

conﬁgured to use the json-file logging driver.





169

$ docker logs vantage-db

1:C 2 Feb 09:53:22.903 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo

1:C 2 Feb 09:53:22.904 # Redis version=4.0.6, bits=64, commit=00000000, modified=0, pid=1

1:C 2 Feb 09:53:22.904 # Warning: no config file specified, using the default config.

1:M 2 Feb 09:53:22.906 \* Running mode=standalone, port=6379.

1:M 2 Feb 09:53:22.906 # WARNING: The TCP backlog setting of 511 cannot be enforced because...

1:M 2 Feb 09:53:22.906 # Server initialized

1:M 2 Feb 09:53:22.906 # WARNING overcommit\_memory is set to 0!

There’s a good chance you’ll ﬁnd network connectivity errors reported in the daemon logs or container logs.

**Service discovery**

As well as core networking, libnetwork also provides some important network services.

*Service discovery* allows all containers and Swarm services to locate each other by name. The only requirement

is that they be on the same network.

Under the hood, this leverages Docker’s embedded DNS server and the DNS resolver in each container. Figure

11.19 shows container “c1” pinging container “c2” by name. The same principle applies to Swarm Services.

**Figure 11.19**

Let’s step through the process.

• **Step 1:** The ping c2 command invokes the local DNS resolver to resolve the name “c2” to an IP address.

All Docker containers have a local DNS resolver.

• **Step 2:** If the local resolver doesn’t have an IP address for “c2” in its local cache, it initiates a recursive

query to the Docker DNS server. The local resolver is pre-conﬁgured to know how to reach the Docker

DNS server.

• **Step 3:** The Docker DNS server holds name-to-IP mappings for all containers created with the --name or

--net-alias ﬂags. This means it knows the IP address of container “c2”.

• **Step 4:** The DNS server returns the IP address of “c2” to the local resolver in “c1”. It does this because the

two containers are on the same network — if they were on different networks this would not work.

• **Step 5:** The ping command issues the ICMP eo request paets to the IP address of “c2”.

Every Swarm service and standalone container started with the --name ﬂag will register its name and IP with

the Docker DNS service. This means all containers and service replicas can use the Docker DNS service to ﬁnd

each other.





170

11: Docker Networking

However, service discovery is *network-scoped*. This means that name resolution only works for containers and

Services on the same network. If two containers are on different networks, they will not be able to resolve each

other.

One last point on service discovery and name resolution…

It’s possible to conﬁgure Swarm services and standalone containers with customized DNS options. For example,

the --dns ﬂag lets you specify a list of custom DNS servers to use in case the embedded Docker DNS server

cannot resolve a query. This is common when querying names of services outside of Docker. You can also use

the --dns-search ﬂag to add custom search domains for queries against unqualiﬁed names (i.e. when the query

is not a fully qualiﬁed domain name).

On Linux, these all work by adding entries to the /etc/resolv.conf ﬁle inside every container.

The following example will start a new standalone container and add the infamous 8.8.8.8 Google DNS server,

as well as nigelpoulton.com as search domain to append to unqualiﬁed queries.

$ docker container run -it --name c1 \

--dns=8.8.8.8 \

--dns-search=nigelpoulton.com \

alpine sh

**Ingress load balancing**

Swarm supports two publishing modes that make services accessible outside of the cluster:

• Ingress mode (default)

• Host mode

Services published via *ingress mode* can be accessed from any node in the Swarm — even nodes **not** running a

service replica. Services published via *host mode* can only be accessed by hiing nodes running service replicas.

Figure 11.20 shows the difference between the two modes.





171

**Figure 11.20**

Ingress mode is the default. This means any time you publish a service with -p or --publish it will default to

*ingress mode*. To publish a service in *host mode* you need to use the long format of the --publish ﬂag **and** add

mode=host. Let’s see an example using *host mode*.

$ docker service create -d --name svc1 \

--publish published=5000,target=80,mode=host \

nginx

A few notes about the command. docker service create lets you publish a service using either a *long form*

*syntax* or *short form syntax*. The short form looks like this: -p 5000:80 and we’ve seen it a few times already.

However, you cannot publish a service in *host mode* using short form.

The long form looks like this: --publish published=5000,target=80,mode=host. It’s a comma-separate list with

no whitespace after each comma. The options work as follows:

•

•

published=5000 makes the service available externally via port 5000

target=80 makes sure that external requests to the published port get mapped back to port 80 on the

service replicas

•

mode=host makes sure that external requests will only reach the service if they come in via nodes running

a service replica.

Ingress mode is what you’ll normally use.





172

11: Docker Networking

Behind the scenes, *ingress mode* uses a layer 4 routing mesh called the **Service Mesh** or the **Swarm Mode Service**

**Mesh**. Figure 11.21 shows the basic traffic ﬂow of an external request to a service exposed in ingress mode.

**Figure 11.21**

Let’s quitely walk through the diagram.

\1. The command at the top deploys a new Swarm service called “svc1”. It’s attaching the service to the

overnet network and publishing it on port 5000.

\2. Publishing a Swarm service like this (--publish published=5000,target=80) will publish it on port 5000

on the ingress network. As all nodes in a Swarm are attached to the ingress network, this means the port

is published *swarm-wide*.

\3. Logic is implemented on the cluster ensuring that any traffic hiing the ingress network, via **any node**,

on port 5000 will be routed to the “svc1” service on port 80.

\4. At this point, a single replica for the “svc1” service is deployed, and the cluster has a mapping rule that

says “*all traffic hiing the ingress network on port 5000 needs routing to a node running a replica for the*

*“svc1” service*”.

\5. The red line shows traffic hiing node1 on port 5000 and being routed to the service replica running on

node2 via the ingress network.

It’s vital to know that the incoming traffic could have hit any of the four Swarm nodes on port 5000 and we

would get the same result. This is because the service is published *swarm-wide* via the ingress network.

It’s also vital to know that if there were multiple replicas running, as shown in Figure 11.22, the traffic would be

balanced across all replicas.





173

**Figure 11.22**

**Docker Networking - The Commands**

Docker networking has its own docker network sub-command. The main commands include:

•

•

docker network ls: Lists all networks on the local Docker host.

docker network create: Creates new Docker networks. By default, it creates them with the nat driver on

Windows and the bridge driver on Linux. You can specify the driver (type of network) with the -d ﬂag.

docker network create -d overlay overnet will create a new overlay network called overnet with the

native Docker overlay driver.

•

•

•

docker network inspect: Provides detailed conﬁguration information about a Docker network.

docker network prune: Deletes all unused networks on a Docker host.

docker network rm: Deletes speciﬁc networks on a Docker host.

**Chapter Summary**

The Container Network Model (CNM) is the master design document for Docker networking and deﬁnes the

three major constructs that are used to build Docker networks — *sandboxes*, *endpoints*, and *networks*.

libnetwork is the open-source library, wrien in Go, that implements the CNM. It’s used by Docker and is where

all of the core Docker networking code lives. It also provides Docker’s network control plane and management

plane.

Drivers extend the Docker network stack (libnetwork) by adding code to implement speciﬁc network types,

such as bridge networks and overlay networks. Docker ships with several built-in drivers, but you can also use

3rd-party drivers.

Single-host bridge networks are the most basic type of Docker network and are suitable for local development and

very small applications. They do not scale, and they require port mappings if you want to publish your services

outside of the network. Docker on Linux implements bridge networks using the built-in bridge driver, whereas

Docker on Windows implements them using the built-in nat driver.





174

11: Docker Networking

Overlay networks are all the rage and are excellent container-only multi-host networks. We’ll talk about them

in-depth in the next chapter.

The macvlan driver (transparent on Windows) allows you to connect containers to existing physical networks

and VLANs. They make containers ﬁrst-class citizens by giving them their own MAC and IP addresses.

Unfortunately, they require promiscuous mode on the host NIC, meaning they won’t work in the public cloud.

Docker also uses libnetwork to implement basic service discovery, as well as a service mesh for container-based

load balancing of ingress traffic.





**12: Docker overlay networking**

Overlay networks are at the beating heart of many cloud-native microservices apps. In this chapter we’ll cover

the fundamentals of native Docker overlay networking.

Docker overlay networking on Windows has feature parity with Linux. This means the examples we’ll use in this

chapter will all work on Linux and Windows.

We’ll split this chapter into the usual three parts:

• The TLDR

• The deep dive

• The commands

Let’s do some networking magic.

**Docker overlay networking - The TLDR**

In the real world, it’s vital that containers can communicate with each other reliably and securely, even when

they’re on different hosts that are on different networks. This is where overlay networking comes in to play. It

allows you to create a ﬂat, secure, layer-2 network, spanning multiple hosts. Containers connect to this and can

communicate directly.

Docker offers native overlay networking that is simple to conﬁgure and secure by default.

Behind the scenes, it’s built on top of libnetwork and drivers. libnetwork is the canonical implementation of the

Container Network Model (CNM) and drivers are pluggable components that implement different networking

technologies and topologies. Docker offers native drivers, including the overlay driver.

**Docker overlay networking - The deep dive**

In Mar 2015, Docker, Inc. acquired a container networking startup called *Soet Plane*. Two of the reasons

behind the acquisition were to bring *real networking* to Docker, and to make container networking simple enough

that even developers could do it.

They over-aieved on both.

However, hiding behind the simple networking commands are a lot of moving parts. The kind of stuff you need

to understand before doing production deployments and aempting to troubleshoot issues.

The rest of this chapter will be broken into two parts:

• Part 1: We’ll build and test a Docker overlay network

• Part 2: We’ll explain the magic that makes it work





176

12: Docker overlay networking

**Build and test a Docker overlay network in Swarm mode**

For the following examples, we’ll use two Docker hosts, on two separate Layer 2 networks, connected by a router.

See Figure 12.1, and note the different networks that each node is on.

**Figure 12.1**

You can follow along with either Linux or Windows Docker hosts. Linux should have at least a 4.4 Linux kernel

(newer is always better) and Windows should be Windows Server 2016 or later with the latest hotﬁxes installed.

You can also follow along on your Mac or Windows PC with Docker Desktop. However, you won’t see the full

beneﬁts as they only support a single Docker host.

**Build a Swarm**

The ﬁrst thing to do is conﬁgure the two hosts into a two-node swarm. This is because swarm mode is a pre-

requisite for overlay networks.

We’ll run the docker swarm init command on **node1** to make it a *manager*, and then we’ll run the docker

swarm join command on **node2** to make it a *worker*. This is not a production-grade setup, but it is enough for a

learning lab. You’re encouraged to test with more managers and workers and expand on the examples.

If you are following along in your own lab, you’ll need to swap the IP addresses and the likes with the correct

values for your environment.

Run the following command on **node1**.

$ docker swarm init \

--advertise-addr=172.31.1.5 \

--listen-addr=172.31.1.5:2377

Swarm initialized: current node (1ex3...o3px) is now a manager.

Run the next command on **node2**. You will need to ensure the following ports are enabled on any ﬁrewalls:

•

•

•

2377/tcp for management plane comms

7946/tcp and 7946/udp for control plane comms (SWIM-based gossip)

4789/udp for the VXLAN data plane





177

$ docker swarm join \

--token SWMTKN-1-0hz2ec...2vye \

172.31.1.5:2377

This node joined a swarm as a worker.

We now have a two-node Swarm with **node1** as a manager and **node2** as a worker.

**Create a new overlay network**

Now let’s create a new *overlay network* called **uber-net**.

Run the following command from **node1** (manager).

$ docker network create -d overlay uber-net

c740ydi1lm89khn5kd52skrd9

That's it. You’ve just created a brand-new overlay network that is available to all hosts in the Swarm and has

its control plane encrypted with TLS (AES in GCM mode with keys automatically rotated every 12 hours). If

you want to encrypt the data plane, you just add the -o encrypted ﬂag to the command. However, data plane

encryption isn’t enabled by default because of the performance overhead. It’s highly recommended that you

extensively test performance before enabling data plane encryption. However, if you do enable it, it’s protected

by the same AES in GCM mode with key rotation.

If you’re unsure about terms such as *control plane* and \* data plane\*, control plane traffic is cluster management

traffic, whereas data plane traffic is application traffic. By default, Docker overlay networks encrypt cluster

management traffic but not application traffic. You must explicitly enable encryption of application traffic.

You can list all networks on each node with the docker network ls command.

$ docker network ls

NETWORK ID

NAME

bridge

docker\_gwbridge

host

ingress

none

DRIVER

bridge

bridge

host

overlay

null

SCOPE

local

local

local

swarm

local

swarm

ddac4ff813b7

389a7e7e8607

a09f7e6b2ac6

ehw16ycy980s

2b26c11d3469

c740ydi1lm89

uber-net

overlay

The output will look more like this on a Windows server:

NETWORK ID

NAME

ingress

nat

none

uber-net

DRIVER

overlay

nat

null

overlay

SCOPE

swarm

local

local

swarm

8iltzv6sbtgc

6545b2a61b6f

96d0d737c2ee

nil5ouh44qco

The newly created network is at the bottom of the list called **uber-net**. The other networks were automatically

created when Docker was installed and when the swarm was initialized.

If you run the docker network ls command on **node2**, you’ll notice that it can’t see the **uber-net** network.

This is because new overlay networks are only extended to worker nodes when they are tasked with running a

container on it. This lazy approach to extended overlay networks improves network scalability by reducing the

amount of network gossip.





178

12: Docker overlay networking

**Attach a service to the overlay network**

Now that you have an overlay network, let’s create a new *Docker service* and attach it to the network. The example

will create the service with two replicas (containers) so that one runs on **node1** and the other runs on **node2**.

This will automatically extend the **uber-net** overlay to **node2**

Run the following commands from **node1**.

Linux example:

$ docker service create --name test \

--network uber-net \

--replicas 2 \

ubuntu sleep infinity

Windows example:

\> docker service create --name test `

--network uber-net `

--replicas 2 `

mcr.microsoft.com\powershell:nanoserver pwsh.exe -Command Start-Sleep 3600

**Note:** The Windows example uses the backti character to split parameters over multiple lines to

make the command more readable. The backti is how PowerShell escapes line feeds.

The command creates a new service called **test**, attaches it to the **uber-net** overlay network, and creates two

replicas (containers) based on the image provided. In both examples, you issued a sleep command to the containers

to keep them running and stop them from exiting.

Because we’re running two replicas (containers), and the Swarm has two nodes, one replica will be seduled on

each node.

Verify the operation with a docker service ps command.

$ docker service ps test

ID

NAME

IMAGE

NODE

DESIRED STATE CURRENT STATE

77q...rkx

97v...pa5

test.1 ubuntu node1

test.2 ubuntu node2

Running

Running

Running

Running

When Swarm starts a container on an overlay network, it automatically extends that network to the node the

container is running on. This means that the **uber-net** network is now visible on **node2**.

Standalone containers that are not part of a swarm service cannot attach to overlay networks unless they have

the attachable=true property. The following command can be used to create an attachable overlay network that

standalone containers can also attach to.

$ docker network create -d overlay --attachable uber-net

Congratulations. You’ve created a new overlay network spanning two nodes on separate physical underlay

networks. You’ve also attached two containers to it. How easy was that!





179

**Test the overlay network**

Let’s test the overlay network with the ping command.

As shown in Figure 12.2, we’ve got two Docker hosts on separate networks, and a single overlay network spanning

both. We’ve got one container connected to the overlay network on each node. Let’s see if they can ping each

other.

**Figure 12.2**

You can run the test by pinging the remote container by name. However, the examples will use IP addresses as

it gives us an excuse to learn how to ﬁnd a containers IP address.

Run a docker network inspect to see the subnet assigned to the overlay and the IP addresses assigned to the

two containers in the test service.

$ docker network inspect uber-net

[

{

"Name": "uber-net",

"Id": "c740ydi1lm89khn5kd52skrd9",

"Scope": "swarm",

"Driver": "overlay",

"EnableIPv6": false,

"IPAM": {

"Driver": "default",

"Options": null,

"Config": [

{

"Subnet": "10.0.0.0/24",

"Gateway": "10.0.0.1"

}

"Containers": {

"Name": "test.1.mfd1kn0qzgosu2f6bhfk5jc2p",

"IPv4Address": "10.0.0.3/24",

<Snip>





180

12: Docker overlay networking

},

},

"Name": "test.2.m49f4psxp3daixlwfvy73v4j8",

"IPv4Address": "10.0.0.4/24",

<Snip>

The output is heavily snipped for readability, but you can see it shows **uber-net**’s subnet is 10.0.0.0/24.

This doesn’t mat either of the physical underlay networks shown in Figure 12.2 (172.31.1.0/24 and

192.168.1.0/24). You can also see the IP addresses assigned to the two containers.

Run the following two commands on **node1** and **node2**. These will get the container’s ID’s and conﬁrm the IP

address from the previous command. Be sure to use the container ID’s from your own lab in the second command.

$ docker container ls

CONTAINER ID

396c8b142a85

IMAGE

ubuntu:latest

COMMAND

CREATED

STATUS

NAME

test.1.mfd...

"sleep infinity" 2 hours ago Up 2 hrs

$ docker container inspect \

--format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' 396c8b142a85

10.0.0.3

Run these commands on both nodes to conﬁrm the IP addresses of both containers.

Figure 12.3 shows the conﬁguration so far. Subnet and IP addresses may be different in your lab.

**Figure 12.3**

As you can see, there is a Layer 2 overlay network spanning both hosts, and each container has an IP address on

this overlay network. This means the container on **node1** will be able to ping the container on **node2** using its

10.0.0.4 address. This works despite the fact that both *nodes* are on different Layer 2 underlay networks.

Let’s prove it.

Log on to the container on **node1** and ping the remote container.

To do this on the Linux Ubuntu container you’ll need to install the ping utility. If you’re following along with

the Windows PowerShell example the ping utility is already installed.





181

Remember that the container IDs will be different in your environment.

Linux example:

$ docker container exec -it 396c8b142a85 bash

root@396c8b142a85:/# apt-get update && apt-get install iputils-ping -y

<Snip>

Reading package lists... Done

Building dependency tree

Reading state information... Done

<Snip>

Setting up iputils-ping (3:20190709-3) ...

Processing triggers for libc-bin (2.31-0ubuntu9) ...

root@396c8b142a85:/# ping 10.0.0.4

PING 10.0.0.4 (10.0.0.4) 56(84) bytes of data.

64 bytes from 10.0.0.4: icmp\_seq=1 ttl=64 time=1.06 ms

64 bytes from 10.0.0.4: icmp\_seq=2 ttl=64 time=1.07 ms

64 bytes from 10.0.0.4: icmp\_seq=3 ttl=64 time=1.03 ms

64 bytes from 10.0.0.4: icmp\_seq=4 ttl=64 time=1.26 ms

^C

root@396c8b142a85:/#

Windows example:

\> docker container exec -it 1a4f29e5a4b6 pwsh.exe

Windows PowerShell

PS C:\> ping 10.0.0.4

Pinging 10.0.0.4 with 32 bytes of data:

Reply from 10.0.0.4: bytes=32 time=1ms TTL=128

Reply from 10.0.0.4: bytes=32 time<1ms TTL=128

Reply from 10.0.0.4: bytes=32 time=2ms TTL=128

Reply from 10.0.0.4: bytes=32 time=2ms TTL=12

PS C:\>

Congratulations. The container on **node1** can ping the container on **node2** via the overlay network. If you created

the network with the -o encrypted ﬂag, the exchange will have been encrypted.

You can also trace the route of the ping command from within the container. This will report a single hop, proving

that the containers are communicating directly via the overlay network — blissfully unaware of any underlay

networks that are being traversed.

**Note:** You’ll need to install traceroute for the Linux example to work.

Linux example:





182

12: Docker overlay networking

$ root@396c8b142a85:/# traceroute 10.0.0.4

traceroute to 10.0.0.4 (10.0.0.4), 30 hops max, 60 byte packets

1

test-svc.2.97v...a5.uber-net (10.0.0.4) 1.110ms 1.034ms 1.073ms

Windows example:

PS C:\> tracert 10.0.0.3

Tracing route to test.2.ttcpiv3p...7o4.uber-net [10.0.0.4]

over a maximum of 30 hops:

1

<1 ms <1 ms <1 ms test.2.ttcpiv3p...7o4.uber-net [10.0.0.4]

Trace complete.

So far, you’ve created an overlay network with a single command. You then added containers to it. The containers

were seduled on two hosts that were on two different Layer 2 underlay networks. Once you worked out the

container’s IP addresses, you proved that they could communicate directly via the overlay network.

**The theory of how it all works**

Now that you’ve seen how easy it is to build and use a secure overlay network, let’s ﬁnd out how it’s all put

together behind the scenes.

Some of the detail in this section will be speciﬁc to Linux. However, the same overall principles apply to Windows.

**VXLAN primer**

First and foremost, Docker overlay networking uses VXLAN tunnels to create virtual Layer 2 overlay networks.

So, before we go any further, let’s do a quite VXLAN primer.

At the highest level, VXLANs let you create a virtual Layer 2 network on top of an existing Layer 3 infrastructure.

That's a lot of teno jargon that means you can create a simple network that hides horriﬁcally complex networks

beneath. The example we used earlier created a new 10.0.0.0/24 Layer 2 network on top of a Layer 3 IP network

comprising two Layer 2 networks — 172.31.1.0/24 and 192.168.1.0/24. This is shown in Figure 12.4.





183

**Figure 12.4**

The beauty of VXLAN is that it’s an encapsulation technology that existing routers and network infrastructure

just see as regular IP/UDP paets and handle without issue.

To create the virtual Layer 2 overlay network, a VXLAN *tunnel* is created through the underlying Layer 3 IP

infrastructure. You might hear the term *underlay network* used to refer to the underlying Layer 3 infrastructure

— the networks that the Docker hosts are connected to.

Each end of the VXLAN tunnel is terminated by a VXLAN Tunnel Endpoint (VTEP). It’s this VTEP that performs

the encapsulation/de-encapsulation and other magic required to make all of this work. See Figure 12.5.

**Figure 12.5**

**Walk through our two-container example**

In the example from earlier, you had two hosts connected via an IP network. Each host ran a single container,

and you created a single VXLAN overlay network for the containers.





184

12: Docker overlay networking

To accomplish this, a new *sandbox* (network namespace) was created on each host. As mentioned in the previous

chapter, a *sandbox* is like a container, but instead of running an application, it runs an isolated network stack —

one that’s sandboxed from the network stack of the host itself.

A virtual swit (a.k.a. virtual bridge) called **Br0** is created inside the sandbox. A VTEP is also created with one

end plumbed into the **Br0** virtual swit, and the other end plumbed into the host network stack (VTEP). The end

in the host network stack gets an IP address on the underlay network the host is connected to, and is bound to a

UDP socket on port 4789. The two VTEPs on each host create the overlay via a VXLAN tunnel as seen in Figure

12.6.

**Figure 12.6**

At this point, the VXLAN overlay is created and ready for use.

Each container then gets its own virtual Ethernet (veth) adapter that is also plumbed into the local **Br0** virtual

swit. The topology now looks like Figure 12.7, and it should be geing easier to see how the two containers

can communicate over the VXLAN overlay network despite their hosts being on two separate networks.





185

**Figure 12.7**

**Communication example**

Now that we’ve seen the main plumbing elements, let’s see how the two containers communicate.

**Warning!** This section gets quite technical, and it’s not necessary for you to understand all of this

detail for day-to-day operations.

For this example, we’ll call the container on node1 “**C1**” and the container on node2 “**C2**”. And let’s assume **C1**

wants to ping **C2** like we did in the practical example earlier in the chapter.

**Figure 12.8**





186

12: Docker overlay networking

**C1** creates the ping requests and sets the destination IP address to be the 10.0.0.4 address of **C2**. It sends the

traffic over its veth interface which is connected to the **Br0** virtual swit. The virtual swit doesn’t know

where to send the paet as it doesn’t have an entry in its MAC address table (ARP table) that corresponds to

the destination IP address. As a result, it ﬂoods the paet to all ports. The VTEP interface is connected to **Br0**

knows how to forward the frame, so responds with its own MAC address. This is a *proxy ARP* reply and results

in the **Br0** swit *learning* how to forward the paet. As a result, **Br0** updates its ARP table, mapping 10.0.0.4

to the MAC address of the local VTEP.

Now that the **Br0** swit has *learned* how to forward traffic to **C2**, all future paets for **C2** will be transmied

directly to the local VTEP interface. The VTEP interface knows about **C2** because all newly started containers

have their network details propagated to other nodes in the Swarm using the network’s built-in gossip protocol.

The paet is sent to the VTEP interface, which encapsulates the frames so they can be sent over the underlay

transport infrastructure. At a fairly high level, this encapsulation includes adding a VXLAN header to the

individual Ethernet frames. The VXLAN header contains the VXLAN network ID (VNID) which is used to map

frames from VLANs to VXLANs and vice versa. Each VLAN gets mapped to VNID so that the paet can be

de-encapsulated on the receiving end and forwarded to the correct VLAN. This maintains network isolation.

The encapsulation also wraps the frame in a UDP paet with the IP address of the remote VTEP on node2 in

the *destination IP ﬁeld*, as well as the UDP port 4789 socket information. This encapsulation allows the data to be

sent across the underlying networks without the underlying networks having to know anything about VXLAN.

When the paet arrives at node2, the kernel sees that it’s addressed to UDP port 4789. The kernel also knows

that it has a VTEP interface bound to this socket. As a result, it sends the paet to the VTEP, which reads the

VNID, de-encapsulates the paet, and sends it on to its own local **Br0** swit on the VLAN that corresponds the

VNID. From there it is delivered to container C2.

And that… ladies and gents… is how VXLAN technology is leveraged by native Docker overlay networking.

Hopefully that’s enough to get you started with any potential production Docker deployments. It should also

give you the knowledge required to talk to your networking team about the networking aspects of your Docker

infrastructure. On the topic of talking to your networking team… I recommend you don’t approach them thinking

that you now know everything about VXLAN. If you do that, you’ll probably embarrass yourself ;-)

One ﬁnal thing. Docker also supports Layer 3 routing within the same overlay network. For example, you can

create an overlay network with two subnets, and Docker will take care of routing between them. The command

to create a network like this could be docker network create --subnet=10.1.1.0/24 --subnet=11.1.1.0/24

-d overlay prod-net. This would result in two virtual swites, **Br0** and **Br1**, being created inside the *sandbox*,

and routing happens by default.

**Docker overlay networking - The commands**

•

docker network create is the command that we use to create a new container network. The -d ﬂag lets

you specify the driver to use, and the most common driver is the overlay driver. You can also specify

*remote* drivers from 3rd parties. For overlay networks, the control plane is encrypted by default. Just add

the -o encrypted ﬂag to encrypt the data plane (performance overheads may be incurred).

docker network ls lists all of the container networks visible to a Docker host. Docker hosts running in

*swarm mode* only see overlay networks if they are hosting containers attached to that particular network.

This keeps network-related gossip to a minimum.

•





187

•

•

docker network inspect shows you detailed information about a particular container network. This

includes *scope*, *driver*, *IPv4* and *IPv4* info, *subnet conﬁguration*, *IP addresses of connected containers\*\*,*

*\*VXLAN network ID*, and *encryption state*.

docker network rm deletes a network

**Chapter Summary**

In this chapter, we saw how easy it is to create new Docker overlay networks with the docker network create

command. We then learned how they are put together behind the scenes using VXLAN technology.

We’ve only scrated the surface of what can be done with Docker overlay networking.









**13: Volumes and persistent data**

Stateful applications that persist data are becoming more and more important in the world of cloud-native and

microservices applications. Docker is an important infrastructure technology in this space, so we’ll turn our

aention in this chapter to investigating how Docker handles applications that write persistent data.

We’ll split the chapter into the usual three parts:

• The TLDR

• The deep dive

• The commands

**Volumes and persistent data - The TLDR**

There are two main categories of data — persistent and non-persistent.

Persistent is the data you need to *keep*. Things like; customer records, ﬁnancial data, research results, audit logs,

and even some types of application *log* data. Non-persistent is the data you don’t need to keep.

Both are important, and Docker has solutions for both.

To deal with non-persistent data, every Docker container gets its own non-persistent storage. This is automatically

created for every container and is tightly coupled to the lifecycle of the container. As a result, deleting the

container will delete the storage and any data on it.

To deal with persistent data, a container needs to store it in a *volume*. Volumes are separate objects that have

their lifecycles decoupled from containers. This means you can create and manage volumes independently, and

they’re not tied to the lifecycle of any container. Net result, you can delete a container that’s using a volume, and

the volume won’t be deleted.

That's the TLDR. Let’s take a closer look.

**Volumes and persistent data - The Deep Dive**

There’s a popular opinion that containers aren’t good for stateful applications that persist data. This was true a

few years ago. However, things are changing, and technologies now exist that make containers a viable oice

for many stateful applications.

Am I saying containers are the best solution for all stateful applications? No. However, we’re about to see some

of the ways that containers deal with persistent and non-persistent data, and you may ﬁnd it hard to see many

differences with virtual machines.

We’ll start out with non-persistent data.





190

13: Volumes and persistent data

**Containers and non-persistent data**

Containers are designed to be immutable. This is just a buzzword that means read-only — it’s a best practice

not to change the conﬁguration of a container after it’s deployed. If something breaks or you need to change

something, you should create a new container with the ﬁxes/updates and deploy it in place of the old container.

You shouldn’t log into a running container and make conﬁguration changes!

However, many applications require a read-write ﬁlesystem in order to simply run – they won’t even run on

a read-only ﬁlesystem. This means it’s not as simple as making containers entirely read-only. Every Docker

container is created by adding a thin read-write layer on top of the read-only image it’s based on. Figure 13.1

shows two running containers sharing a single read-only image.

**Figure 13.1 Ephemeral container storage**

The writable container layer exists in the ﬁlesystem of the Docker host, and you’ll hear it called various names.

These include *local storage*, *ephemeral storage*, and *graphdriver storage*. It’s typically located on the Docker host

in these locations:

• Linux Docker hosts: /var/lib/docker/<storage-driver>/...

• Windows Docker hosts: C:\ProgramData\Docker\windowsfilter\...

This thin writable layer is an integral part of a container and enables all read/write operations. If you, or an

application, update ﬁles or add new ﬁles, they’ll be wrien to this layer. However, it’s tightly coupled to the

container’s lifecycle — it gets created when the container is created and it gets deleted when the container is

deleted. The fact that it’s deleted along with a container means that it’s not an option for important data that you

need to keep (persist).

If your containers don’t create persistent data, this thin writable layer of *local storage* will be ﬁne and you’re

good to go. However, if your containers need to persist data, you need to read the next section.

One ﬁnal word before moving to the next section.

This writable layer of local storage is managed on every Docker host by a storage driver (not to be confused with

a volume driver). If you’re running Docker in production on Linux, you’ll need to make sure you mat the right

storage driver with the Linux distribution on your Docker host. Use the following list as a *guide:*





191

• **Red Hat Enterprise Linux:** Use the overlay2 driver with modern versions of RHEL running Docker 17.06

or higher. Use the devicemapper driver with older versions. This applies to Oracle Linux and other Red

Hat related upstream and downstream distros.

• **Ubuntu:** Use the overlay2 or aufs drivers. If you’re using a Linux 4.x kernel or higher you should go

with overlay2.

• **SUSE Linux Enterprise Server:** Use the btrfs storage driver.

• **Windows** Windows only has one driver and it is conﬁgured by default.

**Containers and persistent data**

*Volumes* are the recommended way to persist data in containers. There are three major reasons for this:

• Volumes are independent objects that are not tied to the lifecycle of a container

• Volumes can be mapped to specialized external storage systems

• Volumes enable multiple containers on different Docker hosts to access and share the same data

At a high-level, you create a volume, then you create a container and mount the volume into it. The volume is

mounted into a directory in the container’s ﬁlesystem, and anything wrien to that directory is stored in the

volume. If you delete the container, the volume and its data will still exist.

Figure 13.2 shows a Docker volume existing outside of the container as a separate object. It is mounted into the

container’s ﬁlesystem at /data, and any data wrien to the /data directory will be stored on the volume and

will exist after the container is deleted.

**Figure 13.2 High-level view of volumes and containers**

In Figure 13.2, the /data directory is a Docker volume that can either be mapped to an external storage system

or a directory on the Docker host. Either way, its lifecycle is decoupled from the container. All other directories

in the container use the thin writable container layer in the local storage area on the Docker host.

The arrow from the volume to the /data directory is shown as a doed line to represent the decoupled relationship

between volumes and containers.

**Creating and managing Docker volumes**

Volumes are ﬁrst-class citizens in Docker. Among other things, this means they are their own object in the API

and have their own docker volume sub-command.

Use the following command to create a new volume called myvol.





192

13: Volumes and persistent data

$ docker volume create myvol

myvol

By default, Docker creates new volumes with the built-in local driver. As the name suggests, volumes created

with the local driver are only available to containers on the same node as the volume. You can use the -d ﬂag

to specify a different driver.

ird-party volume drivers are available as plugins. These provide Docker with seamless access external storage

systems such as cloud storage services and on-premises storage systems including SAN or NAS. This is shown in

Figure 13.3.

**Figure 13.3 Plugging external storage into Docker**

We’ll look at an example with a third-party driver in a later section.

Now that the volume is created, you can see it with the docker volume ls command and inspect it with the

docker volume inspect command.

$ docker volume ls

DRIVER

local

VOLUME NAME

myvol

$ docker volume inspect myvol

[

{

"CreatedAt": "2020-05-02T17:44:34Z",

"Driver": "local",

"Labels": {},

"Mountpoint": "/var/lib/docker/volumes/myvol/\_data",

"Name": "myvol",

"Options": {},

"Scope": "local"

}

]

Notice that the Driver and Scope are both local. This means the volume was created with the local driver and

is only available to containers on this Docker host. The Mountpoint property tells us where in the Docker host’s

ﬁlesystem the volume exists.





193

All volumes created with the local driver get their own directory under /var/lib/docker/volumes on Linux,

and C:\ProgramData\Docker\volumes on Windows. This means you can see them in your Docker host’s

ﬁlesystem. You can even access them directly from your Docker host, although this is not normally recommended.

We showed an example of this in the chapter on Docker Compose — we copied a ﬁle directly into a volume’s

directory on the Docker host and the ﬁle immediately appeared in the volume inside the container.

Now the volume is created, it can be used by one or more containers. We’ll see usage examples in a minute.

There are two ways to delete a Docker volume:

•

•

docker volume prune

docker volume rm

docker volume prune will delete **all volumes** that are not mounted into a container or service replica, so **use**

**with caution!** docker volume rm lets you specify exactly which volumes you want to delete. Neither command

will delete a volume that is in use by a container or service replica.

As the myvol volume is not in use, delete it with the prune command.

$ docker volume prune

WARNING! This will remove all volumes not used by at least one container.

Are you sure you want to continue? [y/N] y

Deleted Volumes:

myvol

Total reclaimed space: 0B

Congratulations, you’ve created, inspected, and deleted a Docker volume. And you did it all without interacting

with a container. This demonstrates the independent nature of volumes.

At this point, you know all of the commands to create, list, inspect, and delete Docker volumes. However,

it’s also possible to deploy volumes via Dockerﬁles using the VOLUME instruction. The format is VOLUME

<container-mount-point>. Interestingly, you cannot specify a directory on the host when deﬁning a volume

in a Dockerﬁle. This is because *host* directories are different depending on what OS your Docker host is running

– it could break your builds if you speciﬁed a directory on a Docker host that doesn’t exist. As a result, deﬁning

a volume in a Dockerﬁle requires you to specify host directories at deploy-time.

**Demonstrating volumes with containers and services**

Let’s see how to use volumes with containers and services.

The examples will be from a system with no pre-existing volumes, and everything we demonstrate applies to

both Linux and Windows.

Use the following command to create a new standalone container that mounts a volume called bizvol.

**Linux example:**





194

13: Volumes and persistent data

$ docker container run -dit --name voltainer \

--mount source=bizvol,target=/vol \

alpine

**Windows example:**

Use PowerShell for all Windows examples, and note the use of backtis (‘) to split commands across multiple

lines.

\> docker container run -dit --name voltainer `

--mount source=bizvol,target=c:\vol `

mcr.microsoft.com/powershell:nanoserver

The command uses the --mount ﬂag to mount a volume called “bizvol” into the container at either /vol or c:\vol.

The command completes successfully despite the fact there is no volume on the system called bizvol. This raises

an interesting point:

• If you specify an existing volume, Docker will use the existing volume

• If you specify a volume that doesn’t exist, Docker will create it for you

In this case, bizvol didn’t exist, so Docker created it and mounted it into the new container. This means you’ll

be able to see it with docker volume ls.

$ docker volume ls

DRIVER

local

VOLUME NAME

bizvol

Although containers and volumes have separate lifecycle’s, you cannot delete a volume that is in use by a

container. Try it.

$ docker volume rm bizvol

Error response from daemon: remove bizvol: volume is in use - [b44d3f82...dd2029ca]

The volume is brand new, so it doesn’t have any data. Let’s exec onto the container and write some data to it.

The example cited is Linux, if you’re following along on Windows just replace sh with pwsh.exe at the end of

the command. All other commands will work on Linux and Windows.

$ docker container exec -it voltainer sh

/# echo "I promise to write a review of the book on Amazon" > /vol/file1

/# ls -l /vol

total 4

-rw-r--r-- 1 root root

50 Jan 12 13:49 file1

/# cat /vol/file1

I promise to write a review of the book on Amazon

Type exit to return to the shell of your Docker host, and then delete the container with the following command.





195

$ docker container rm voltainer -f

voltainer

Even though the container is deleted, the volume still exists:

$ docker container ls -a

CONTAINER ID

IMAGE

COMMAND

CREATED

STATUS

$ docker volume ls

DRIVER

local

VOLUME NAME

bizvol

Because the volume still exists, you can look at its mount point on the host to check if the data is still there.

Run the following commands from the terminal of your Docker host. The ﬁrst one will show that the ﬁle still

exists, the second will show the contents of the ﬁle.

Be sure to use the C:\ProgramData\Docker\volumes\bizvol\\_data directory if you’re following along on

Windows. Also, this step won’t work on Docker Desktop for Mac and Windows 10. This is because Docker

Desktop runs Docker inside of a VM and the volume data directories exist inside the VM.

$ ls -l /var/lib/docker/volumes/bizvol/\_data/

total 4

-rw-r--r-- 1 root root 50 Jan 12 14:25 file1

$ cat /var/lib/docker/volumes/bizvol/\_data/file1

I promise to write a review of the book on Amazon

Great, the volume and data still exists.

It’s even possible to mount the bizvol volume into a new service or container. The following command creates a

new Docker service, called hellcat, and mounts bizvol into the service replica at /vol. You’ll need to be running in

swarm mode for this command to work. If you’re running in single-engine mode you can use a docker container

run command instead.

$ docker service create \

--name hellcat \

--mount source=bizvol,target=/vol \

alpine sleep 1d

overall progress: 1 out of 1 tasks

1/1: running

[====================================>]

verify: Service converged

We didn’t specify the --replicas ﬂag, so only a single service replica was deployed. Find which node in the

Swarm it’s running on.





196

13: Volumes and persistent data

$ docker service ps hellcat

ID

l3nh...

NAME

hellcat.1

NODE

node1

DESIRED STATE

Running

CURRENT STATE

Running 19 seconds ago

In this example, the replica is running on node1. Log on to node1 and get the ID of the service replica container.

node1$ docker container ls

CTR ID

IMAGE

COMMAND

STATUS

NAMES

df6..a7b

alpine:latest

"sleep 1d"

Up 25 secs

hellcat.1.l3nh...

Notice that the container name is a combination of service-name, replica-number, and replica-ID separated

by periods.

Exec onto the container and check that the data is present in /vol. We’ll use the service replica’s container ID in

the exec example. If you’re following along on Windows, remember to replace sh with pwsh.exe.

node1$ docker container exec -it df6 sh

/# cat /vol/file1

I promise to write a review of the book on Amazon

Excellent, the volume has preserved the original data and made it available to a new container.

I guess it’s time to jump over to Amazon and write that book review :-D

**Sharing storage across cluster nodes**

Integrating external storage systems with Docker makes it possible to share volumes between cluster nodes. These

external systems can be cloud storage services or enterprise storage systems in your on-premises data centers. As

an example, a single storage LUN or NFS share can be presented to multiple Docker hosts, allowing it to be used

by containers and service replicas no-maer which Docker host they’re running on. Figure 13.4 shows a single

external shared volume being presented to two Docker nodes. These Docker nodes can then make the shared

volume available to either, or both containers.

**Figure 13.4**





197

Building a setup like this requires a lot of things. You need access to a specialised storage systems and knowledge

of how it works and presents storage. You also need to know how your applications read and write data to the

shared storage. Finally, you need a volumes driver plugin that works with the external storage system.

Docker Hub is the best place to ﬁnd volume plugins. Login to Docker Hub, select the view to show plugins instead

of containers, and ﬁlter results to only show Volume plugins. Once you’ve located the appropriate plugin for

your storage system, you create any conﬁguration ﬁles it might need, and install it with docker plugin install.

Once the plugin is registered, you can create new volumes from the storage system using docker volume create

with the -d ﬂag.

The following example installs the Pure Storage Docker volume plugin. This plugin provides access to storage

volumes on either a Pure Storage FlashArray or FlashBlade storage system. Plugins only work with the correct

external storage systems.

\1. The Pure Storage plugin requires a conﬁguration ﬁle called pure.json in the Docker host’s /etc/pure-

docker-plugin/directory. This ﬁle contains the information required for the plugin to locate the

external storage system, authenticate, and access resources.

\2. Install the plugin and grant the required permissions.

$ docker plugin install purestorage/docker-plugin:latest --alias pure --grant-all-permissions

Plugin "purestorage/docker-plugin:3.8" is requesting the following privileges:

\- network: [host]

\- host pid namespace: [true]

\- mount: [/etc/pure-docker-plugin/pure.json]

\- mount: [/dev]

\- mount: [/sys]

\- allow-all-devices: [true]

\- capabilities: [CAP\_SYS\_ADMIN CAP\_SYS\_PTRACE]

Do you grant the above permissions? [y/N] y

\1. List the available plugins.

$ docker plugin ls

ID

NAME

pure:latest

DESCRIPTION

Pure Storage plugin for Docker

ENABLED

true

6b5e61aefbb3

\1. Create a new volume with the plugin (you can also do this as part of the container creation process). This

example creates a new 25GB volume called “fastvol” on the registered Pure Storage backend.

$ docker volume create -d pure -o size=25GB fastvol

fastvol

Different storage drivers support different options, but this should be enough to give you a feel for how they

work.





198

13: Volumes and persistent data

**Potential data corruption**

A major concern with any conﬁguration that shares a single volume among multiple containers is **data**

**corruption**.

Assume the following example based on Figure 13.4.

The application running in ctr-1 on node-1 updates some data in the shared volume. However, instead of writing

the update directly to the volume, it holds it in its local buffer for faster recall (this is common in many operating

systems). At this point, the application in ctr-1 thinks the data has been wrien to the volume. However, before

ctr-1 on node-1 ﬂushes its buffers and commits the data to the volume, the app in ctr-2 on node-2 updates the

same data with a different value and commits it directly to the volume. At this point, both applications *think*

they’ve updated the data in the volume, but in reality only the application in ctr-2 has. A few seconds later, ctr-1

on node-1 ﬂushes the data to the volume, overwriting the changes made by the application in ctr-2. However, the

application in ctr-2 is totally unaware of this! This is one of the ways data corruption happens.

To prevent this, you need to write your applications in a way to avoid things like this.

**Volumes and persistent data - The Commands**

•

docker volume create is the command we use to create new volumes. By default, volumes are created

with the local driver, but you can use the -d ﬂag to specify a different driver.

docker volume ls will list all volumes on the local Docker host.

docker volume inspect shows detailed volume information. Use this command to see many interesting

volume properties, including where a volume exists in the Docker host’s ﬁlesystem.

docker volume prune will delete **all** volumes that are not in use by a container or service replica. **Use**

**with caution!**

•

•

•

•

•

•

docker volume rm deletes speciﬁc volumes that are not in use.

docker plugin install will install new volume plugins from Docker Hub.

docker plugin ls lists all plugins installed on a Docker host.

**Chapter Summary**

There are two main types of data: persistent and non-persistent data. Persistent data is data that you need to keep,

non-persistent is data that you don’t need to keep. By default, all containers get a layer of writable non-persistent

storage that lives and dies with the container — we call this *local storage* and it’s ideal for non-persistent data.

However, if your containers create data that you need to keep, you should store the data in a Docker volume.

Docker volumes are ﬁrst-class citizens in the Docker API and managed independently of containers with their

own docker volume sub-command. This means that deleting a container will not delete the volumes it was using.

ird party volume plugins can provide Docker access to specialised external storage systems. They’re installed

from Docker Hub with the docker plugin install command and are referenced at volume creation time with

the -d command ﬂag.

Volumes are the recommended way to work with persistent data in a Docker environment.





**14: Deploying apps with Docker Stacks**

Deploying and managing cloud-native microservices applications comprising lots of small integrated services at

scale is hard.

Fortunately, Docker Stacks are here to help. They simplify application management by providing; *desired state,*

*rolling updates, simple, scaling operations, health checks,* and more! All wrapped in a nice declarative model.

Love it!

Don’t worry if these buzzwords are new to you or sound complicated, you’ll understand them all by the end of

the chapter.

We’ll split this chapter into the usual three parts:

• The TLDR

• The deep dive

• The commands

**Deploying apps with Docker Stacks - The TLDR**

Testing and deploying simple apps on your laptop is easy, but that’s for amateurs. Deploying and managing

multi-service apps in real-world production environments… that’s for pro’s!

Fortunately, stacks are here to help!. They let you deﬁne complex multi-service apps in a single declarative ﬁle.

They also provide a simple way to deploy the app and manage its entire lifecycle — initial deployment > health

checks > scaling > updates > rollbacks and more!

The process is simple. Deﬁne the desired state of your app in a *Compose ﬁle*, then deploy and manage it with the

docker stack command. That's it.

The Compose ﬁle includes the entire stack of microservices that make up the app. It also includes all of the

volumes, networks, secrets, and other infrastructure the app needs. The docker stack deploy command is used

to deploy the entire app from the single ﬁle. Simple.

To accomplish all of this, stacks build on top of Docker Swarm, meaning you get all of the security and advanced

features that come with Swarm.

In a nutshell, Docker is great for application development and testing. Docker Stacks are great for scale and

production.

**Deploying apps with Docker Stacks - The Deep Dive**

If you know Docker Compose, you’ll ﬁnd Docker Stacks really easy. In fact, in many ways, stacks are what

we always wished Compose was — fully integrated into Docker and able to manage the entire lifecycle of

applications.





200

14: Deploying apps with Docker Stacks

Aritecturally speaking, stacks are at the top of the Docker application hierary. They build on top of *services*,

which in turn build on top of containers.

We’ll divide this section of the chapter as follows:

• Overview of the sample app

• Looking closer at the stack ﬁle

• Deploying the app

• Managing the app

**Overview of the sample app**

For the rest of the chapter, we’ll be using the popular **AtSea Shop** demo app. It lives on [GitHub](https://github.com/dockersamples/atsea-sample-shop-app)[¹⁵](#br211)[ ](#br211)and is open-

sourced under the [Apae](https://github.com/dockersamples/atsea-sample-shop-app/blob/master/LICENSE)[ ](https://github.com/dockersamples/atsea-sample-shop-app/blob/master/LICENSE)[2.0](https://github.com/dockersamples/atsea-sample-shop-app/blob/master/LICENSE)[ ](https://github.com/dockersamples/atsea-sample-shop-app/blob/master/LICENSE)[license](https://github.com/dockersamples/atsea-sample-shop-app/blob/master/LICENSE)[¹⁶](#br211).

We’re using this app because it’s moderately complicated without being too big to list and describe in a book.

Beneath the covers, it’s a cloud-native microservices app that leverages certiﬁcates and secrets. The high-level

application architecture is shown in Figure 14.1.

**Figure 14.1 AtSea Shop high level architecture**

As you can see, it comprises 5 *Services*, 3 networks, 4 secrets, and 3 port mappings. We’ll see each of these in

detail when we inspect the stack ﬁle.

**Note:** When referring to *services* in this chapter, we’re talking about the Docker service object that

is one or more identical containers managed as a single object on a swarm cluster.

Clone the application’s GitHub repo so that you have all of the application source ﬁles on your local machine.

¹⁵<https://github.com/dockersamples/atsea-sample-shop-app>

¹⁶<https://github.com/dockersamples/atsea-sample-shop-app/blob/master/LICENSE>





201

$ git clone https://github.com/dockersamples/atsea-sample-shop-app.git

Cloning into 'atsea-sample-shop-app'...

remote: Enumerating objects: 30, done.

remote: Counting objects: 100% (30/30), done.

remote: Compressing objects: 100% (30/30), done.

remote: Total 672 (delta 20), reused 0 (delta 0), pack-reused 642

Receiving objects: 100% (672/672), 7.29 MiB | 1.46 MiB/s, done.

Resolving deltas: 100% (217/217), done.

The application consists of several directories and source ﬁles. Feel free to explore them all. However, we’re going

to focus on the docker-stack.yml ﬁle that deﬁnes the app and its requirements. We’ll refer to this as the *stack*

*ﬁle*.

At the highest level, it deﬁnes 4 top-level keys.

version:

services:

networks:

secrets:

**Version** indicates the version of the Compose ﬁle format. This has to be 3.0 or higher to work with stacks. **Services**

is where you deﬁne the stack of services that make up the app. **Networks** lists the required networks, and **secrets**

deﬁnes the secrets the app uses.

If you expand each top-level key, you’ll see how things map to Figure 14.1. The stack ﬁle has ﬁve services called

“reverse\_proxy”, “database”, “appserver”, “visualizer”, and “payment\_gateway”. So does Figure 14.1. The stack

ﬁle has three networks called “front-tier”, “back-tier”, and “payment”. So does Figure 14.1. Finally, the stack ﬁle

has four secrets called “postgres\_password”, “staging\_token”, “revprox\_key”, and “revprox\_cert”. So does Figure

14.1.

version: "3.2"

services:

reverse\_proxy:

database:

appserver:

visualizer:

payment\_gateway:

networks:

front-tier:

back-tier:

payment:

secrets:

postgres\_password:

staging\_token:

revprox\_key:

revprox\_cert:

It’s important to understand that the stack ﬁle captures and deﬁnes many of the requirements of the entire

application. As such, it’s self-documenting and a great tool for bridging the gap between dev and ops.

Let’s take a closer look at each section of the stack ﬁle.





202

14: Deploying apps with Docker Stacks

**Looking closer at the stack ﬁle**

Sta ﬁles are very similar to Compose ﬁles. The only requirement is that the version: key specify a value of

“3.0” or higher. See the [the](https://docs.docker.com/compose/compose-file/)[ ](https://docs.docker.com/compose/compose-file/)[Docker](https://docs.docker.com/compose/compose-file/)[ ](https://docs.docker.com/compose/compose-file/)[docs](https://docs.docker.com/compose/compose-file/)[¹⁷](#br213)[ ](#br213)for the latest information on Compose ﬁle versions and compatibility

with your versions of Docker.

One of the ﬁrst things Docker does when deploying an app from a stack ﬁle is create any required networks listed

under the networks key. If the networks don’t already exist, Docker creates them.

Let’s see the networks deﬁned in the stack ﬁle.

**Networks**

networks:

front-tier:

back-tier:

payment:

driver: overlay

driver\_opts:

encrypted: 'yes'

The stack ﬁle describes three networks; front-tier, back-tier, and payment. By default, they’ll all be created

as overlay networks by the overlay driver. But the payment network is special — it requires an encrypted data

plane.

As mentioned in the chapter on overlay networking, the control plane of all overlay networks is encrypted by

default, but you have to explicitly encrypt the data plane. The control plane is for network management traffic

and the data plane is for application traffic. Encrypting the data plane has a potential performance overhead.

To encrypt the data plane, you have two oices:

• Pass the -o encrypted ﬂag to the docker network create command.

• Specify encrypted: 'yes' under driver\_opts in the stack ﬁle.

The overhead incurred by encrypting the data plane depends on various factors such traffic type and traffic ﬂow.

You should perform extensive testing to understand the performance overhead that encrypting data plane traffic

has on your workload. It’s not uncommon for this to be approximately 10%.

As previously mentioned, all three networks will be created before the secrets and services.

Let’s look at the secrets.

**Secrets**

Secrets are deﬁned as top-level objects, and the stack ﬁle we’re using deﬁnes four:

¹⁷<https://docs.docker.com/compose/compose-file/>





203

secrets:

postgres\_password:

external: true

staging\_token:

external: true

revprox\_key:

external: true

revprox\_cert:

external: true

Notice that all four are deﬁned as external. This means that they must already exist before the stack can be

deployed.

It’s possible for secrets to be created on-demand when the application is deployed — just replace external: true

with file: <filename>. However, for this to work, a plaintext ﬁle containing the unencrypted value of the secret

must already exist on the host’s ﬁlesystem. This has obvious security implications.

We’ll see how to create these secrets when we come to deploy the app. For now, it’s enough to know that the

application deﬁnes four secrets that need pre-creating.

Let’s look at each of the services.

**Services**

Services are where most of the action happens.

Each service is a JSON collection (dictionary) that contains a bun of keys. We’ll step through each one and

explain what each of the options does.

**The reverse\_proxy service**

As you can see, the reverse\_proxy service deﬁnes an image, ports, secrets, and networks.

reverse\_proxy:

image: dockersamples/atseasampleshopapp\_reverse\_proxy

ports:

\- "80:80"

\- "443:443"

secrets:

\- source: revprox\_cert

target: revprox\_cert

\- source: revprox\_key

target: revprox\_key

networks:

\- front-tier

The image key is the only mandatory key in the service object. As the name suggests, it deﬁnes the Docker image

that will be used to build the replicas for the service. Remember that a service is one or more identical containers.

Docker is opinionated, so unless you specify otherwise, the **image** will be pulled from Docker Hub. You can

specify images from 3rd-party registries by prepending the image name with the DNS name of the registry’s API

endpoint such as gcr.io for Google’s container registry.





204

14: Deploying apps with Docker Stacks

One difference between Docker Stacks and Docker Compose is that stacks do not support **builds**. This means all

images have to be built prior to deploying the stack.

The **ports** key deﬁnes two mappings:

•

•

80:80 maps port 80 across the swarm to port 80 on each service replica.

443:443 maps port 443 across the Swarm to port 443 on each service replica.

By default, all ports are mapped using *ingress mode*. This means they’ll be mapped and accessible from every

node in the Swarm — even nodes not running a replica. The alternative is *host mode*, where ports are only mapped

on swarm nodes running replicas for the service. However, *host mode* requires you to use the long-form syntax.

For example, mapping port 80 in *host mode* using the long-form syntax would be like this:

ports:

\- target: 80

published: 80

mode: host

The long-form syntax is recommended, as it’s easier to read and more powerful (it supports ingress mode **and**

host mode). However, it requires at least version 3.2 of the Compose ﬁle format.

The **secrets** key deﬁnes two secrets — revprox\_cert and revprox\_key. These secrets must already exist on the

swarm and must also be deﬁned in the top-level secrets section of the stack ﬁle.

Secrets get mounted into service replicas as a regular ﬁle. The name of the ﬁle will be whatever you specify

as the target value in the stack ﬁle, and the ﬁle will appear in the replica under /run/secrets on Linux, and

C:\ProgramData\Docker\secrets on Windows. Linux mounts /run/secrets as an in-memory ﬁlesystem, but

Windows does not.

The secrets deﬁned in this service will be mounted in each service replica as /run/secrets/revprox\_cert and

/run/secrets/revprox\_key. To mount one of them as /run/secrets/uber\_secret you would deﬁne it in the

stack ﬁle as follows:

secrets:

\- source: revprox\_cert

target: uber\_secret

The **networks** key ensures that all replicas for the service will be attached to the front-tier network. The

network speciﬁed here must be deﬁned in the networks top-level key, and if it doesn’t already exist, Docker will

create it as an overlay.

**The database service**

The database service also deﬁnes; an image, a network, and a secret. As well as those, it introduces environment

variables and placement constraints.





205

database:

image: dockersamples/atsea\_db

environment:

POSTGRES\_USER: gordonuser

POSTGRES\_DB\_PASSWORD\_FILE: /run/secrets/postgres\_password

POSTGRES\_DB: atsea

networks:

\- back-tier

secrets:

\- postgres\_password

deploy:

placement:

constraints:

\- 'node.role == worker'

The **environment** key lets you inject environment variables into services replicas at runtime. This service uses

three environment variables to deﬁne a database user, the location of the database password (a secret mounted

into every service replica), and the name of the database.

environment:

POSTGRES\_USER: gordonuser

POSTGRES\_DB\_PASSWORD\_FILE: /run/secrets/postgres\_password

POSTGRES\_DB: atsea

A better and more secure solution would be to pass all three values in as secrets, as this would avoid documenting

the database name and database user in plaintext variables.

The service also deﬁnes a *placement constraint* under the deploy key. This ensures that replicas for this service

will always run on Swarm *worker* nodes.

deploy:

placement:

constraints:

\- 'node.role == worker'

Placement constraints are a great way of inﬂuencing seduling decisions. Swarm currently lets you sedule

against all of the following:

• Node ID. node.id == o2p4kw2uuw2a

• Node name. node.hostname == wrk-12

• Role. node.role != manager

• Engine labels. engine.labels.operatingsystem==ubuntu 16.04

• Custom node labels. node.labels.zone == prod1

Notice that == and != are both supported.

**The appserver service**

The appserver service uses an image, attaches to three networks, and mounts a secret. It also introduces several

additional features under the deploy key.





206

14: Deploying apps with Docker Stacks

appserver:

image: dockersamples/atsea\_app

networks:

\- front-tier

\- back-tier

\- payment

deploy:

replicas: 2

update\_config:

parallelism: 2

failure\_action: rollback

placement:

constraints:

\- 'node.role == worker'

restart\_policy:

condition: on-failure

delay: 5s

max\_attempts: 3

window: 120s

secrets:

\- postgres\_password

Let’s take a closer look at the new stuff under the deploy key.

First up, services.appserver.deploy.replicas = 2 will set the desired number of replicas for the service to 2.

If omied, the default value is 1.

If you need to change the number of replicas after you’ve deployed the service, you should do so declaratively.

This means updating services.appserver.deploy.replicas ﬁeld in the stack ﬁle with the new value, and then

redeploying the stack. We’ll see this later, but re-deploying a stack does not affect services that you haven’t made

a change to.

services.appserver.deploy.update\_config tells Docker how to act when updating the service. For this service,

Docker will update two replicas at-a-time (parallelism) and will perform a ‘rollback’ if it detects the update is

failing. Rolling back will start new replicas based on the previous deﬁnition of the service. The default value for

failure\_action is pause, which will stop further replicas being updated. The other option is continue.

update\_config:

parallelism: 2

failure\_action: rollback

You specify other options as part of update\_config. These include inserting a delay, a failure monitor period,

and controlling the order of starting updated replicas before terminating older replicas or vice versa.

The services.appserver.deploy.restart-policy object tells Swarm how to restart replicas (containers) if and

when they fail. The policy for this service will restart a replica if it stops with a non-zero exit code (condition:

on-failure). It will try to restart the failed replica 3 times, and wait up to 120 seconds to decide if the restart

worked. It will wait 5 seconds between each of the three restart aempts.





207

restart\_policy:

condition: on-failure

delay: 5s

max\_attempts: 3

window: 120s

**visualizer**

The visualizer service references an image, maps a port, deﬁnes an update conﬁg, and deﬁnes a placement

constraint. It also mounts a volume and deﬁnes a custom grace period for container stop operations.

visualizer:

image: dockersamples/visualizer:stable

ports:

\- "8001:8080"

stop\_grace\_period: 1m30s

volumes:

\- "/var/run/docker.sock:/var/run/docker.sock"

deploy:

update\_config:

failure\_action: rollback

placement:

constraints:

\- 'node.role == manager'

When Docker stops a container, it issues a SIGTERM to the application process with PID 1 inside the container.

The application then has a 10-second grace period to perform any clean-up operations. If it doesn’t handle the

signal, it will be forcibly terminated after 10 seconds with a SIGKILL. The stop\_grace\_period property overrides

this 10 second grace period.

The volumes key is used to mount pre-created volumes and host directories into a service replica. In this case,

it’s mounting /var/run/docker.sock from the Docker host, into /var/run/docker.sock inside of each service

replica. This means any reads and writes to /var/run/docker.sock in the replica will be passed through to the

same directory in the host.

/var/run/docker.sock happens to be the IPC socket that the Docker daemon exposes all of its API endpoints

on. This means giving a container access to it gives the container the ability to issue commands to the Docker

daemon. This has signiﬁcant security implications and is not recommended in the real world. Fortunately, this is

just a demo app in a lab environment.

The reason this service requires access to the Docker daemon is because it provides a graphical representation

of services on the Swarm. To do this, it needs to be able to query the Docker daemon on a manager node. To

accomplish this, a placement constraint forces all service replicas onto manager nodes, and the Docker socket is

bind-mounted into each service replica.

**payment\_gateway**

The payment\_gateway service speciﬁes an image, mounts a secret, attaches to a network, deﬁnes a partial

deployment strategy, and then imposes a couple of placement constraints.





208

14: Deploying apps with Docker Stacks

payment\_gateway:

image: dockersamples/atseasampleshopapp\_payment\_gateway

secrets:

\- source: staging\_token

target: payment\_token

networks:

\- payment

deploy:

update\_config:

failure\_action: rollback

placement:

constraints:

\- 'node.role == worker'

\- 'node.labels.pcidss == yes'

We’ve seen all of these options before, except for the node.label in the placement constraint. Node labels are

custom-deﬁned labels added to swarm nodes with the docker node update command. As such, they’re only

applicable within the context of the node’s role in the Swarm (you can’t leverage them on standalone containers

or outside of the Swarm).

In this example, the payment\_gateway service performs operations that require it to run on a swarm node that

has been hardened to PCI DSS standards. To enable this, you can apply a custom *node label* to any swarm node

meeting these requirements. We’ll do this when we build the lab to deploy the app.

As this service deﬁnes two placement constraints, replicas will only be deployed to nodes that mat both. I.e. a

**worker** node with the pcidss=yes node label.

Now that we’re ﬁnished examining the stack ﬁle, you should have a good understanding of the application’s

requirements. As mentioned previously, the stack ﬁle is a great piece of application documentation. We know the

application has 5 services, 3 networks, and 4 secrets. We know which services attach to which networks, which

ports need publishing, which images are required, and we even know that some services need to run on speciﬁc

nodes.

Let’s deploy it.

**Deploying the app**

There’s a few pre-requisites that need taking care of before deploying the app:

• **Swarm mode:** We’ll deploy the app as a Docker Sta, and stacks require Swarm mode.

• **Labels:** One of the Swarm worker nodes needs a custom node label.

• **Secrets:** The app uses secrets which need pre-creating before it can be deployed.

**Building a lab for the sample app**

In this section we’ll build a three-node Linux-based Swarm cluster that satisﬁes all of the application’s pre-reqs.

Once we’re done, the lab will look like this.





209

**Figure 14.2 Sample lab**

We’ll complete the following three steps:

• Create a new Swarm

• Add a node label

• Create the secrets

Let’s create a new three-node Swarm cluster.

\1. Initialize a new Swarm.

Run the following command on the node that you want to be your Swarm manager.

$ docker swarm init

Swarm initialized: current node (lhma...w4nn) is now a manager.

<Snip>

\2. Add worker nodes.

Copy the docker swarm join command that displayed in the output of the previous command. Paste it

into the two nodes you want to join as workers.

//Worker 1 (wrk-1)

wrk-1$ docker swarm join --token SWMTKN-1-2hl6...-...3lqg 172.31.40.192:2377

This node joined a swarm as a worker.

//Worker 2 (wrk-2)

wrk-2$ docker swarm join --token SWMTKN-1-2hl6...-...3lqg 172.31.40.192:2377

This node joined a swarm as a worker.

\3. Verify that the Swarm is conﬁgured with one manager and two workers.

Run this command from the manager node.





210

14: Deploying apps with Docker Stacks

$ docker node ls

ID

HOSTNAME

STATUS

Ready

Ready

Ready

AVAILABILITY

Active

Active

MANAGER STATUS

Leader

lhm...4nn \*

b74...gz3

o9x...um8

mgr-1

wrk-1

wrk-2

Active

The Swarm is now ready.

The payment\_gateway service has a set of placement constraints forcing it to only run on **worker nodes** with the

pcidss=yes node label. In this step we’ll add that node label to wrk-1.

In the real world you would harden at least one of your Docker nodes to PCI standards before labelling it.

However, this is just a lab, so we’ll skip the hardening step and just add the label to wrk-1.

Run the following commands from the Swarm manager.

\1. Add the node label to wrk-1.

$ docker node update --label-add pcidss=yes wrk-1

wrk-1

Node labels only apply within the Swarm.

\2. Verify the node label.

$ docker node inspect wrk-1

[

{

"ID": "b74rzajmrimfv7hood6l4lgz3",

"Version": {

"Index": 27

},

"CreatedAt": "2020-05-04T12:06:18.221580486Z",

"UpdatedAt": "2020-05-04T12:08:17.335295528Z",

"Spec": {

"Labels": {

"pcidss": "yes"

},

<Snip>

The wrk-1 worker node is now conﬁgured so that it can run replicas for the payment\_gateway service.

The application deﬁnes four secrets, all of which need creating before the app can be deployed:

•

•

•

•

postgress\_password

staging\_token

revprox\_cert

revprox\_key

Run the following commands from the manager node to create them.

\1. Create a new key pair.

ree of the secrets will be populated with cryptographic keys. We’ll create the keys in this step and then

place them inside of Docker secrets in the next steps.





211

$ openssl req -newkey rsa:4096 -nodes -sha256 \

-keyout domain.key -x509 -days 365 -out domain.crt

You’ll have two new ﬁles in your current directory. We’ll use them in the next step.

\2. Create the revprox\_cert, revprox\_key, and postgress\_password secrets.

$ docker secret create revprox\_cert domain.crt

cqblzfpyv5cxb5wbvtrbpvrrj

$ docker secret create revprox\_key domain.key

jqd1ramk2x7g0s2e9ynhdyl4p

$ docker secret create postgres\_password domain.key

njpdklhjcg8noy64aileyod6l

\3. Create the staging\_token secret.

$ echo staging | docker secret create staging\_token -

sqy21qep9w17h04k3600o6qsj

\4. List the secrets.

$ docker secret ls

ID

NAME

DRIVER

CREATED

UPDATED

njp...d6l

cqb...rrj

jqd...l4p

sqy...qsj

postgres\_password

revprox\_cert

revprox\_key

staging\_token

47 seconds ago

About a minute ago

About a minute ago

23 seconds ago

47 seconds ago

About a minute ago

About a minute ago

23 seconds ago

That's all of the pre-requisites taken care of. Time to deploy the app!

**Deploying the sample app**

If you haven’t already done so, clone the app’s GitHub repo to your Swarm manager.

$ git clone https://github.com/dockersamples/atsea-sample-shop-app.git

Cloning into 'atsea-sample-shop-app'...

remote: Enumerating objects: 30, done.

remote: Counting objects: 100% (30/30), done.

remote: Compressing objects: 100% (30/30), done.

remote: Total 672 (delta 20), reused 0 (delta 0), pack-reused 642

Receiving objects: 100% (672/672), 7.29 MiB | 33.19 MiB/s, done.

Resolving deltas: 100% (217/217), done.

$ cd atsea-sample-shop-app

Now that you have the code, you are ready to deploy the app.

Stacks are deployed using the docker stack deploy command. In its basic form it accepts two arguments:





212

14: Deploying apps with Docker Stacks

• name of the stack ﬁle

• name of the stack

The application’s GitHub repository contains a stack ﬁle called docker-stack.yml, so we’ll use this as stack ﬁle.

We’ll call the stack seastack, though you can choose a different name if you don’t like that.

Run the following commands from within the atsea-sample-shop-app directory on the Swarm manager.

Deploy the stack (app).

$ docker stack deploy -c docker-stack.yml seastack

Creating network seastack\_default

Creating network seastack\_back-tier

Creating network seastack\_front-tier

Creating network seastack\_payment

Creating service seastack\_database

Creating service seastack\_appserver

Creating service seastack\_visualizer

Creating service seastack\_payment\_gateway

Creating service seastack\_reverse\_proxy

You can run docker network ls and docker service ls commands to see the networks and services that were

deployed as part of the app.

A few things to note from the output of the command.

The networks were created before the services. This is because the services attach to the networks, so need the

networks to be created before they can start.

Docker prepends the name of the stack to every resource it creates. In our example, the stack is called seastack,

meaning all resources are named seastack\_<resource>. For example, the payment network is called seastack\_-

payment. Resources that were created prior to the deployment, such as secrets, do not get renamed.

Another thing to note is the presence of a network called seastack\_default. This isn’t deﬁned in the stack ﬁle,

so why was it created? Every service needs to attach to a network, but the visualizer service didn’t specify one.

Therefore, Docker created one called seastack\_default and attached it to that. You can verify this by running a

docker network inspect seastack\_default command.

You can verify the status of a stack with a couple of commands. docker stack ls lists all stacks on the system,

including how many services they have. docker stack ps <stack-name> gives more detailed information about

a particular stack, such as *desired state* and *current state*. Let’s see them both.

$ docker stack ls

NAME

seastack

SERVICES

5

ORCHESTRATOR

Swarm

$ docker stack ps seastack

NAME

NODE

DESIRED STATE

CURRENT STATE

seastack\_reverse\_proxy.1

seastack\_payment\_gateway.1

seastack\_visualizer.1

seastack\_appserver.1

seastack\_database.1

seastack\_appserver.2

wrk-2

wrk-1

mgr-1

wrk-2

wrk-2

wrk-1

Running

Running

Running

Running

Running

Running

Running 7 minutes ago

Running 7 minutes ago

Running 7 minutes ago

Running 7 minutes ago

Running 7 minutes ago

Running 7 minutes ago





213

The docker stack ps command is a good place to start when troubleshooting services that fail to start. It gives an

overview of every service in the stack, including which node each replica is seduled on, current state, desired

state, and error message. The following output shows two failed aempts to start a replica for the reverse\_proxy

service on the wrk-2 node.

$ docker stack ps seastack

NAME

NODE

DESIRED

STATE

CURRENT ERROR

STATE

reverse\_proxy.1

\\_reverse\_proxy.1

wrk-2

wrk-2

Shutdown

Shutdown

Failed

Failed

"task: non-zero exit (1)"

"task: non-zero exit (1)"

For more detailed logs of a particular service you can use the docker service logs command. You pass it either

the service name/ID, or replica ID. If you pass it the service name or ID, you’ll get the logs for all service replicas.

If you pass it a particular replica ID, you’ll only get the logs for that replica.

The following docker service logs command shows the logs for all replicas in the seastack\_reverse\_proxy

service that had the two failed replicas in the previous output.

$ docker service logs seastack\_reverse\_proxy

seastack\_reverse\_proxy.1.zhc3cjeti9d4@wrk-2 | [emerg] 1#1: host not found...

seastack\_reverse\_proxy.1.6m1nmbzmwh2d@wrk-2 | [emerg] 1#1: host not found...

seastack\_reverse\_proxy.1.6m1nmbzmwh2d@wrk-2 | nginx: [emerg] host not found..

seastack\_reverse\_proxy.1.zhc3cjeti9d4@wrk-2 | nginx: [emerg] host not found..

seastack\_reverse\_proxy.1.1tmya243m5um@mgr-1 | 10.255.0.2 "GET / HTTP/1.1" 302

The output is trimmed to ﬁt the page, but you can see that logs from all three service replicas are shown (the two

that failed and the one that’s running). Each line starts with the name of the replica, which includes the service

name, replica number, replica ID, and name of host that it’s seduled on. Following that is the log output.

**Note:** You might have noticed that all of the replicas in the previous output showed as replica

number 1. This is because Docker created one at a time and only started a new one when the

previous one had failed.

It’s hard to tell because the output is trimmed to ﬁt the book, but it looks like the ﬁrst two replicas failed because

they were relying on something in another service that was still starting (a sort of race condition when dependent

services are starting).

You can follow the logs (--follow), tail them (--tail), and get extra details (--details).

Now that the stack is up and running, let’s see how to manage it.

**Managing the app**

We know that a *stack* is set of related services and infrastructure that gets deployed and managed as a unit. And

while that’s a fancy sentence full of buzzwords, it reminds us that the stack is built from normal Docker resources

— networks, volumes, secrets, services etc. This means we can inspect them with their normal Docker commands:

docker network, docker volume, docker secret, docker service…





214

14: Deploying apps with Docker Stacks

With this in mind, it’s possible to use the docker service command to manage services that are part of the stack.

A simple example would be using the docker service scale command to increase the number of replicas in

the appserver service. However, **this is not the recommended method!**

The recommended method is the declarative method, which uses the stack ﬁle as the ultimate source of truth. As

such, all changes to the stack should be made to the stack ﬁle, and then the updated stack ﬁle should be used to

redeploy the app.

Here’s a quite example of why the imperative method (making changes via the CLI) is bad:

*Imagine you have a stack deployed from the* docker-stack.yml *ﬁle that you cloned from GitHub*

*earlier in the chapter. This means you have two replicas of the* appserver *service. If you use the*

docker service scale *command to change that to 4 replicas, the current observed state of the*

*cluster will be 4 running replicas, but the stack ﬁle will still deﬁne 2. Admiedly, that doesn’t sound*

*like the end of the world. However, imagine you then edit the stack ﬁle to use a newer image, and*

*roll it out the recommended way with the* docker stack deploy *command. As part of this rollout,*

*the number of* appserver *replicas in the cluster will be rolled back to 2, because you never updated*

*to the stack ﬁle to 4 replicas. For this kind of reason, it’s recommended to make all changes to the*

*application via the stack ﬁle, and to manage the stack ﬁle in a proper version control system.*

Let’s walk through the process of making a couple of declarative changes to the stack.

We’ll make the following changes:

• Increase the number of appserver replicas from 2 to 10

• Increase the stop grace period for the visualizer service to 2 minutes

Edit the docker-stack.yml ﬁle and update the following two values:

•

•

.services.appserver.deploy.replicas=10

.services.visualizer.stop\_grace\_period=2m

The relevant sections of the stack ﬁle will now look like this:

<Snip>

appserver:

image: dockersamples/atsea\_app

networks:

\- front-tier

\- back-tier

\- payment

deploy:

replicas: 2

<Snip>

visualizer:

<<Updated value

image: dockersamples/visualizer:stable

ports:

\- "8001:8080"

stop\_grace\_period: 2m

<Snip

<<Updated value

Save the ﬁle and redeploy the app.





215

$ docker stack deploy -c docker-stack.yml seastack

Updating service seastack\_reverse\_proxy (id: z4crmmrz7zi83o0721heohsku)

Updating service seastack\_database (id: 3vvpkgunetxaatbvyqxfic115)

Updating service seastack\_appserver (id: ljht639w33dhv0dmht1q6mueh)

Updating service seastack\_visualizer (id: rbwoyuciglre01hsm5fviabjf)

Updating service seastack\_payment\_gateway (id: w4gsdxfnb5gofwtvmdiooqvxs)

Re-deploying the app like this will only update the changed components.

Run a docker stack ps to see the number of appserver replicas increasing.

$ docker stack ps seastack

NAME

NODE

DESIRED STATE

Running

Shutdown

Running

Running

Running

Running

Running

Running

Running

Running

Running

Running

CURRENT STATE

seastack\_visualizer.1

seastack\_visualizer.1

seastack\_appserver.1

seastack\_appserver.2

seastack\_appserver.3

seastack\_appserver.4

seastack\_appserver.5

seastack\_appserver.6

seastack\_appserver.7

seastack\_appserver.8

seastack\_appserver.9

seastack\_appserver.10

mgr-1

mgr-1

wrk-2

wrk-1

wrk-2

wrk-1

wrk-2

wrk-1

wrk-2

wrk-1

wrk-2

wrk-1

Running 1 second ago

Shutdown 3 seconds ago

Running 24 minutes ago

Running 24 minutes ago

Running 1 second ago

Running 1 second ago

Running 1 second ago

Starting 7 seconds ago

Running 1 second ago

Starting 7 seconds ago

Running 1 second ago

Starting 7 seconds ago

The output has been trimmed so that it ﬁts on the page, and so that only the updated services are shown.

Notice that there are two lines for the visualizer service. One line shows a replica that was shutdown 3 seconds

ago, and the other line shows a replica that has been running for 1 second. This is because the change we made

to the visualizer service caused Swarm to terminate the existing replica and started a new one with the new

stop\_grace\_period value.

You can also see that there are now 10 replicas for the appserver service, and that they are in various states in

the “CURRENT STATE” column — some are *running* whereas others are still *starting*.

After enough time, the cluster will converge so that *current observed state* mates the new *desired state*. At that

point, what is deployed and observed on the cluster will exactly mat what is deﬁned in the stack ﬁle. This is a

happy place to be :-D

This declarative update paern should be used for all updates to the app/stack. I.e. **all changes should be made**

**declaratively via the stack ﬁle, and rolled out using docker stack deploy**.

The correct way to delete a stack is with the docker stack rm command. Be warned though! It deletes the stack

without asking for conﬁrmation.





216

14: Deploying apps with Docker Stacks

$ docker stack rm seastack

Removing service seastack\_appserver

Removing service seastack\_database

Removing service seastack\_payment\_gateway

Removing service seastack\_reverse\_proxy

Removing service seastack\_visualizer

Removing network seastack\_front-tier

Removing network seastack\_payment

Removing network seastack\_default

Removing network seastack\_back-tier

Notice that the networks and services were deleted, but the secrets weren’t. This is because the secrets were pre-

created and existed before the stack was deployed. If your stack deﬁnes volumes at the top-level, these will not

be deleted by docker stack rm either. This is because volumes are intended as long-term persistent data stores

and exist independent of the lifecycle of containers, services, and stacks.

Congratulations. You know how to deploy and manage a multi-service app using Docker Stacks.

**Deploying apps with Docker Stacks - The Commands**

•

docker stack deploy is the command for deploying **and** updating stacks of services deﬁned in a stack

ﬁle (usually called docker-stack.yml).

•

•

docker stack ls lists all stacks on the Swarm, including how many services they have.

docker stack ps gives detailed information about a deployed stack. It accepts the name of the stack as

its main argument, lists which node each replica is running on, and shows *desired state* and *current state*.

docker stack rm deletes a stack from the Swarm. It does not ask for conﬁrmation before deleting the

stack.

•

**Chapter Summary**

Stacks are the native Docker solution for deploying and managing cloud-native microservices applications with

multiple services. They’re baked into the Docker engine, and offer a simple declarative interface for deploying

and managing the entire lifecycle of an application.

You start with application code and a set of infrastructure requirements — things like networks, ports, volumes

and secrets. You containerize the application and group together all of the app services and infrastructure

requirements into a single declarative stack ﬁle. You set the number of replicas, as well as rolling update and restart

policies. You then take the ﬁle and deploy the application from it using the docker stack deploy command.

Future updates to the deployed app should be done declaratively by checking the stack ﬁle out of source control,

updating it, re-deploying the app, and checking the stack ﬁle back into source control.

Because the stack ﬁle deﬁnes things like number of service replicas, you should maintain separate stack ﬁles for

each of your environments, such as dev, test and prod.





**15: Security in Docker**

Good security is all about layers, and Docker has lots of layers. It supports all the major Linux security

technologies as well as plenty of its own. And the best thing… many of them are simple and easy to conﬁgure.

In this chapter, we’ll look at some of the technologies that can make running containers on Docker very secure.

When we get to the deep dive part of the chapter, we’ll divide things into two categories:

• Linux security technologies

• Docker security technologies

Large parts of the chapter will be speciﬁc to Linux. However, the **Docker security technologies** section is

platform agnostic and applies equally to Linux and Windows.

**Security in Docker - The TLDR**

Security is all about layers. Generally speaking, the more layers of security the more secure something is. Well…

Docker offers a lot of security layers. Figure 15.1 shows some of the security-related technologies we’ll cover in

the chapter.

**Figure 15.1**

Docker on Linux leverages most of the common Linux security and workload isolation technologies. These include

*namespaces*, *control groups (cgroups)*, *capabilities*, *mandatory access control (MAC) systems*, and *seccomp*. For

each one, Docker implements sensible defaults for a seamless and *moderately secure* out-of-the-box experience.

However, you can customize each one to your own speciﬁc requirements.





218

15: Security in Docker

Docker itself adds some excellent additional security technologies. And one of the best things about the Docker

security technologies is that they’re **amazingly simple to use!**

**Docker Swarm Mode** is secure by default. You get all of the following with zero conﬁguration required;

cryptographic node IDs, mutual authentication, automatic CA conﬁguration, automatic certiﬁcate rotation,

encrypted cluster store, encrypted networks, and more.

**Docker Content Trust (DCT)** lets you sign your images and verify the integrity and publisher of images you

consume.

**Image security scanning** analyses images, detects known vulnerabilities, and provides detailed reports.

**Docker secrets** are a way to securely share sensitive data and are ﬁrst-class objects in Docker. They’re stored in

the encrypted cluster store, encrypted in-ﬂight when delivered to containers, stored in in-memory ﬁlesystems

when in use, and operate a least-privilege model.

There’s a lot more, but the important thing to know is that Docker works with the major Linux security

technologies as well as providing its own extensive and growing set of security technologies. While the Linux

security technologies tend to be complex, the native Docker security technologies tend to be simple.

**Security in Docker - The deep dive**

We all know that security is important. We also know that security can be complicated and boring.

When Docker decided to bake security into the platform, it decided to make it simple and easy. They knew that

if security was hard to conﬁgure, people wouldn’t use it. As a result, most of the security technologies offered by

the Docker platform are simple to use. They also ship with sensible defaults — meaning you get a *fairly secure*

platform at zero effort. Of course, the defaults aren’t perfect, but they’re enough to serve as a safe starting point.

From there you should customize them to your requirements.

We’ll organize the rest of this chapter as follows:

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

All *good* container platforms use *namespaces* and *cgroups* to build containers. The *best* container platforms will

also integrate with other Linux security technologies such as *capabilities*, *Mandatory Access Control systems* like

SELinux and AppArmor, and *seccomp*. As expected, Docker integrates with them all.

In this section of the chapter we’ll take a *brief* look at some of the major Linux security technologies used

by Docker. We won’t go into detail, as I want the main focus of the chapter to be on the value-add security

technologies Docker adds.





219

**Namespaces**

Kernel namespaces are at the very heart of containers. They slice up an operating system (OS) so that it looks

and feels like multiple **isolated** operating systems. This lets us do really cool things like run multiple web servers

on the same OS without having port conﬂicts. It also lets us run multiple apps on the same OS without them

ﬁghting over shared conﬁg ﬁles and shared libraries.

A couple of quite examples:

• Namespaces let you run multiple web servers, each on port 443, on a single OS. To do this you just run

each web server app inside of its own *network namespace*. This works because each *network namespace*

gets its own IP address and full range of ports. You may have to map each one to a separate port on the

Docker host, but each can run without being re-wrien or reconﬁgured to use a different port.

• You can run multiple applications, each requiring their own version of a shared library or conﬁguration

ﬁle. To do this you run each application inside of its own *mount namespace*. This works because each

*mount namespace* can have its own isolated copy of any directory on the system (e.g. /etc, /var, /dev etc.)

Figure 15.2 shows a high-level example of two web server applications running on a single host and both using

port 443. Each web server app is running inside of its own network namespace.

**Figure 15.2**

**Note**: While namespaces isolate multiple processes on a single OS, the isolation they provide is not

very strong. For example, namespaces are not as good at workload isolation as virtual machines.

You should keep this in mind from a security perspective and not rely too heavily on the isolation

provided by namespaces.

Working directly with namespaces is hard. Fortunately, Docker hides this complexity and manages all of the

namespaces required to build a useful container.

Docker on Linux currently utilizes the following kernel namespaces:





220

15: Security in Docker

• Process ID (pid)

• Network (net)

• Filesystem/mount (mnt)

• Inter-process Communication (ipc)

• User (user)

• UTS (uts)

We’ll brieﬂy explain what each one does in a moment. But the most important thing to understand is that **Docker**

**containers are an organized collection of namespaces**. This means that you get all of this OS isolation for free

with every container.

For example, every container has its own pid, net, mnt, ipc, uts, and potentially user namespace. In fact, an

organized collection of these namespaces is what we call a “container”. Figure 15.3 shows a single Linux host

running two containers.

**Figure 15.3**

Let’s brieﬂy look at how Docker uses each namespace:

•

Process ID namespace: Docker uses the pid namespace to provide isolated process trees for each

container. This means every container gets its own PID 1. PID namespaces also mean that one container

cannot see or access to the process tree of other containers. Nor can it see or access the process tree of the

host it’s running on.

•

•

Network namespace: Docker uses the net namespace to provide each container its own isolated network

stack. This stack includes; interfaces, IP addresses, port ranges, and routing tables. For example, every

container gets its own eth0 interface with its own unique IP and range of ports.

Mount namespace: Every container gets its own unique isolated root (/) ﬁlesystem. This means every

container can have its own /etc, /var, /dev and other important ﬁlesystem constructs. Processes inside

of a container cannot access the mount namespace of the Linux host or other containers — they can only

see and access their own isolated ﬁlesystem.

•

•

Inter-process Communication namespace: Docker uses the ipc namespace for shared memory access

within a container. It also isolates the container from shared memory outside of the container.

User namespace: Docker lets you use user namespaces to map users inside of a container to different

users on the Linux host. A common example is mapping a container’s root user to a non-root user on the

Linux host.

•

UTS namespace: Docker uses the uts namespace to provide each container with its own hostname.

Remember… a container is a collection of namespaces packaged and ready to use.





221

**Control Groups**

If namespaces are about isolation, *control groups (cgroups)* are about seing limits.

ink of containers as similar to rooms in a hotel. While each room might appear isolated, every room shares

a common set of infrastructure resources — things like water supply, electricity supply, shared swimming pool,

shared gym, shared breakfast bar etc. Cgroups let us set limits so that (stiing with the hotel analogy) no single

container can use all of the water or eat everything at the breakfast bar.

In the real world, not the hotel analogy, containers are isolated from each other but all share a common set of OS

resources — things like CPU, RAM, network bandwidth, and disk I/O. Cgroups let us set limits on each of these

so a single container cannot consume everything and cause a denial of service (DoS) attack.

**Capabilities**

It’s a bad idea to run containers as root — root is all-powerful and therefore very dangerous. But, it can be

allenging running containers as unprivileged non-root users. For example, on most Linux systems, non-root

users tend to be so powerless they’re practically useless. What’s needed, is a technology that lets us pi and

choose which root powers a container needs in order to run.

Enter *capabilities!*

Under the hood, the Linux root user is a combination of a long list of *capabilities*. Some of these *capabilities*

include:

•

•

•

•

CAP\_CHOWN: lets you change ﬁle ownership

CAP\_NET\_BIND\_SERVICE: lets you bind a socket to low numbered network ports

CAP\_SETUID: lets you elevate the privilege level of a process

CAP\_SYS\_BOOT: lets you reboot the system.

The list goes on and is long.

Docker works with *capabilities* so that you can run containers as root, but strip out all the capabilities you

don’t need. For example, if the only root privilege your container needs is the ability to bind to low numbered

network ports, you should start a container and drop all root capabilities, then add back just the CAP\_NET\_-

BIND\_SERVICE capability.

This is an excellent example of implementing *least privilege* — you get a container running with only the

capabilities required. Docker also imposes restrictions so that containers cannot re-add the dropped capabilities.

While this is great, conﬁguring the correct set of capabilities can be prohibitively complex for many users.

**Mandatory Access Control systems**

Docker works with major Linux MAC technologies such as AppArmor and SELinux.

Depending on your Linux distribution, Docker applies a default AppArmor proﬁle to all new containers.

According to the Docker documentation, this default proﬁle is “moderately protective while providing wide

application compatibility”.

Docker also lets you start containers without a policy applied, as well as giving you the ability to customize

policies to meet speciﬁc requirements. This is also very powerful, but can also be prohibitively complex.





222

15: Security in Docker

**seccomp**

Docker uses seccomp, in ﬁlter mode, to limit the syscalls a container can make to the host’s kernel.

As per the Docker security philosophy, all new containers get a default seccomp proﬁle conﬁgured with sensible

defaults. This is intended to provide moderate security without impacting application compatibility.

As always, you can customize seccomp proﬁles, and you can pass a ﬂag to Docker so that containers can be

started without a seccomp proﬁle.

As with many of the technologies already mentioned, seccomp is extremely powerful. However, the Linux syscall

table is long, and conﬁguring the appropriate seccomp policies can be prohibitively complex.

**Final thoughts on the Linux security technologies**

Docker supports most of the important Linux security technologies and ships with sensible defaults that add

security but aren’t too restrictive. Figure 15.4 shows how these technologies form multiple layers of potential

security.

**Figure 15.4**

Some of these technologies can be complicated to customize as they require deep knowledge of how the Linux

kernel works. Hopefully they will get simpler to conﬁgure in the future, but for now, the default conﬁgurations

that ship with Docker might be a good place to start.

**Docker platform security technologies**

Let’s take a look at some of the major security technologies offered by the **Docker platform**.





223

**Security in Swarm Mode**

Docker Swarm allows you to cluster multiple Docker hosts and deploy applications declaratively. Every Swarm

is comprised of *managers* and *workers* that can be Linux or Windows. Managers host the control plane of the

cluster and are responsible for conﬁguring the cluster and dispatching work tasks. Workers are the nodes that

run your application code as containers.

As expected, *swarm mode* includes many security features that are enabled out-of-the-box with sensible defaults.

These include:

• Cryptographic node IDs

• TLS for mutual authentication

• Secure join tokens

• CA conﬁguration with automatic certiﬁcate rotation

• Encrypted cluster store (conﬁg DB)

• Encrypted networks

Let’s walk through the process of building a secure swarm and conﬁguring some of the security aspects.

To follow along with the complete set of examples you’ll need at least three Docker hosts running Docker 17.03

or higher. The examples cited use three Docker hosts called “mgr1”, “mgr2”, and “wrk1”. Each one is running

Docker 19.03.4. There is network connectivity between all three hosts, and all three can ping each other by name.

**Conﬁgure a secure Swarm**

Run the following command from the node you want to be the ﬁrst manager in the new swarm. In the example,

we’ll run it from **mgr1**.

$ docker swarm init

Swarm initialized: current node (7xam...662z) is now a manager.

To add a worker to this swarm, run the following command:

docker swarm join --token \

SWMTKN-1-1dmtwu...r17stb-ehp8g...hw738q 172.31.5.251:2377

To add a manager to this swarm, run 'docker swarm join-token manager'

and follow the instructions.

That's it! That is literally all you need to do to conﬁgure a secure swarm.

**mgr1** is conﬁgured as the ﬁrst manager of the swarm and also as the root certiﬁcate authority (CA). The swarm

itself has been given a cryptographic clusterID. **mgr1** has issued itself with a client certiﬁcate that identiﬁes

it as a manager in the swarm, certiﬁcate rotation has been conﬁgured with the default value of 90 days, and a

cluster conﬁg database has been conﬁgured and encrypted. A set of secure tokens have also been created so that

new managers and new workers can be joined to the swarm. And all of this with a **single command!**

Figure 15.5 shows how the lab looks now. Some of the details may be different in your lab.





224

15: Security in Docker

**Figure 15.5**

Let’s join **mgr2** as an additional manager.

Joining new managers to a swarm is a two-step process. The ﬁrst step extracts the required token. The second

step runs the docker swarm join command on the node you wish to add. As long as you include the manager

join token as part of the command, **mgr2** will join the swarm as a manager.

Run the following command from **mgr1** to extract the manager join token.

$ docker swarm join-token manager

To add a manager to this swarm, run the following command:

docker swarm join --token \

SWMTKN-1-1dmtwu...r17stb-2axi5...8p7glz \

172.31.5.251:2377

The output gives you the exact command you need to run on nodes that you want to join as managers. The join

token and IP address will be different in your lab.

The format of the join command is:

•

docker swarm join --token <manager-join-token> <ip-of-existing-manager>:<swarm-port>

The format of the token is:

•

SWMTKN-1-<hash-of-cluster-certificate>-<manager-join-token>

Copy the command and run it on “mgr2”:





225

$ docker swarm join --token SWMTKN-1-1dmtwu...r17stb-2axi5...8p7glz \

\> 172.31.5.251:2377

This node joined a swarm as a manager.

**mgr2** has joined the swarm as an additional manager. In production clusters you should always run either 3 or

5 managers for high availability.

Verify **mgr2** was successfully added by running a docker node ls on either of the two managers.

$ docker node ls

ID

HOSTNAME

mgr1

STATUS

Ready

Ready

AVAILABILITY

Active

Active

MANAGER STATUS

Leader

Reachable

7xamk...ge662z

i0ue4...zcjm7f \* mgr2

The output shows that **mgr1** and **mgr2** are both part of the swarm and are both managers. The updated

conﬁguration is shown in Figure 15.6.

**Figure 15.8**

Two managers is possibly the worst number possible. However, we’re just messing about in a demo lab, not

building a business critical production environment ;-)

Adding a swarm worker is a similar two-step process. Step 1 extracts the join token, and step 2 is to run a docker

swarm join command on the node you want to join as a worker.

Run the following command on either of the managers to expose the worker join token.





226

15: Security in Docker

$ docker swarm join-token worker

To add a worker to this swarm, run the following command:

docker swarm join --token \

SWMTKN-1-1dmtw...17stb-ehp8g...w738q \

172.31.5.251:2377

Again, you get the exact command you need to run on nodes you want to join as workers. The join token and IP

address will be different in your lab.

Copy the command and run it on **wrk1** as shown:

$ docker swarm join --token SWMTKN-1-1dmtw...17stb-ehp8g...w738q \

\> 172.31.5.251:2377

This node joined a swarm as a worker.

Run another docker node ls command from either of the swarm managers.

$ docker node ls

ID

HOSTNAME

mgr1

wrk1

STATUS

Ready

Ready

Ready

AVAILABILITY

Active

Active

MANAGER STATUS

Leader

7xamk...ge662z \*

ailrd...ofzv1u

i0ue4...zcjm7f

mgr2

Active

Reachable

You now have a swarm with two managers and one worker. The managers are conﬁgured for high availability

(HA) and the cluster store is replicated to both. The ﬁnal conﬁguration is shown in Figure 15.7.

**Figure 15.7**





227

**Looking behind the scenes at Swarm security**

Now that we’ve built a secure Swarm let’s take a minute to look behind the scenes at some of the security

technologies involved.

**Swarm join tokens**

The only thing that is needed to join new managers and workers to an existing swarm is the relevant join token.

For this reason, it’s vital that you keep your join-tokens safe. Do not post them on public GitHub repos or even

internal source code repos that are not restricted.

Every swarm maintains two distinct join tokens:

• One for joining new managers

• One for joining new workers

It’s worth understanding the format of the Swarm join token. Every join token is comprised of 4 distinct ﬁelds

separated by dashes (-):

PREFIX - VERSION - SWARM ID - TOKEN

The preﬁx is always SWMTKN. This allows you to paern-mat against it and prevent people from accidentally

posting it publicly. The VERSION ﬁeld indicates the version of the swarm. The Swarm ID ﬁeld is a hash of the

swarm’s certiﬁcate. The TOKEN ﬁeld is the part that determines whether it can join nodes as managers or workers.

As the following shows, the manager and worker join tokens for a given Swarm are identical except for the ﬁnal

TOKEN ﬁeld.

• MANAGER: SWMTKN-1-1dmtwusdc...r17stb-2axi53zjbs45lqxykaw8p7glz

• WORKER: SWMTKN-1-1dmtwusdc...r17stb-ehp8gltji64jbl45zl6hw738q

If you suspect that either of your join tokens has been compromised, you can revoke them and issue new ones

with a single command. The following example revokes the existing *manager* join token and issues a new one.

$ docker swarm join-token --rotate manager

Successfully rotated manager join token.

To add a manager to this swarm, run the following command:

docker swarm join --token \

SWMTKN-1-1dmtwu...r17stb-1i7txlh6k3hb921z3yjtcjrc7 \

172.31.5.251:2377

Existing managers do not need updating, however, you’ll need to use the new token to add new managers.

Notice that the only difference between the old and new join tokens is the last ﬁeld. The hash of the Swarm ID

remains the same.

Join tokens are stored in the cluster store which is encrypted by default.





228

15: Security in Docker

**TLS and mutual authentication**

Every manager and worker that joins a swarm is issued a client certiﬁcate. This certiﬁcate is used for mutual

authentication. It identiﬁes the node, the swarm that it’s a member of, and role the node performs in the swarm

(manager or worker).

You can inspect a node’s client certiﬁcate on Linux nodes with the following command.

$ sudo openssl x509 \

-in /var/lib/docker/swarm/certificates/swarm-node.crt \

-text

Certificate:

Data:

Version: 3 (0x2)

Serial Number:

80:2c:a7:b1:28...a8:af:89:a1:2a:51:89

Signature Algorithm: ecdsa-with-SHA256

Issuer: CN = swarm-ca

Validity

Not Before: May 5 10:31:00 2020 GMT

Not After : Aug 3 11:31:00 2020 GMT

Subject: O=mfbkgjm2tlametbnfqt2zid8x, OU=swarm-manager,

CN=7xamk8w3hz9q5kgr7xyge662z

Subject Public Key Info:

<SNIP>

The Subject data in the output uses the standard O, OU, and CN ﬁelds to specify the Swarm ID, the node’s role,

and the node ID.

• The Organization (O) ﬁeld stores the Swarm ID

• The Organizational Unit (OU) ﬁeld stores the node’s role in the swarm

• The Canonical Name (CN) ﬁeld stores the node’s crypto ID.

This is shown in Figure 15.8.

**Figure 15.8**





229

You can also see the certiﬁcate rotation period in the Validity section.

You can mat these values to the corresponding values shown in the output of a docker system info command.

$ docker system info

<SNIP>

Swarm: active

NodeID: 7xamk8w3hz9q5kgr7xyge662z

Is Manager: true

\# Relates to the CN field

\# Relates to the OU field

ClusterID: mfbkgjm2tlametbnfqt2zid8x # Relates to the O field

...

<SNIP>

...

CA Configuration:

Expiry Duration: 3 months

Force Rotate: 0

\# Relates to Validity field

Root Rotation In Progress: false

<SNIP>

**Conﬁguring some CA settings**

You can conﬁgure the certiﬁcate rotation period for the Swarm with the docker swarm update command. The

following example changes the certiﬁcate rotation period to 30 days.

$ docker swarm update --cert-expiry 720h

Swarm allows nodes to renew certiﬁcates early (slightly before they expire) so that not all nodes don’t try and

update their certiﬁcates at the same time.

You can conﬁgure an external CA when creating a new swarm by passing the --external-ca ﬂag to the docker

swarm init command.

The new docker swarm ca sub-command can be used to manage CA related conﬁguration. Run the command

with the --help ﬂag to see a list of things it can do.

$ docker swarm ca --help

Usage: docker swarm ca [OPTIONS]

Manage root CA

Options:

--ca-cert pem-file

Path to the PEM-formatted root CA

certificate to use for the new cluster

Path to the PEM-formatted root CA

key to use for the new cluster

--ca-key pem-file

--cert-expiry duration

Validity period for node certificates

(ns|us|ms|s|m|h) (default 2160h0m0s)

Exit immediately instead of waiting for

-d, --detach





230

15: Security in Docker

the root rotation to converge

Specifications of one or more certificate

signing endpoints

--external-ca external-ca

-q, --quiet

--rotate

Suppress progress output

Rotate the swarm CA - if no certificate

or key are provided, new ones will be generated

**The cluster store**

The cluster store is the brains of a swarm and is where cluster conﬁg and state are stored. It’s also critical to other

Docker technologies such as overlay networking and Secrets. This is why swarm mode is required for so many

advanced and security related Docker features. The moral of the story… if you’re not running in swarm mode,

there’ll be a bun of Docker technologies and security features you won’t be able to use.

The store is currently based on the popular etcd distributed database and is automatically conﬁgured to replicate

itself to all managers in the swarm. It is also encrypted by default.

Day-to-day maintenance of the cluster store is taken care of automatically by Docker. However, in production

environments, you should have strong backup and recovery solutions in place for it.

That's enough for now about swarm mode security.

**Detecting vulnerabilities with image security scanning**

Image scanning is your primary weapon against vulnerabilities and security holes in your images.

Image scanners work by inspecting images and searching for packages that have known vulnerabilities. Once

you know about these, you can update the packages and dependencies to versions with ﬁxes.

As good as image scanning is, it’s important to understand its limitations. For example, image scanning is focussed

on images and does not detect security problems with networks, nodes, or orestrators. Also, not all image

scanners are equal — some perform deep binary-level scanning to detect packages, whereas others simply look

at package names and do not closely inspect the content of images.

At the time of writing, Docker Hub does not offer image scanning services. This may change in the future. Some

on-premises private registry solutions offer built-in scanning, and there are third-party services that offer image

scanning services.

Figure 15.9 and Figure 15.10 are included as an example of the kind of reports image scanners can provide.





231

**Figure 15.9**

**Figure 15.10**

In summary, image security scanning can be a great tool for deeply inspecting your images for known

vulnerabilities. Beware though, with great knowledge comes great responsibility — once you become aware

of vulnerabilities you are responsible for mitigating or ﬁxing them.

**Signing and verifying images with Docker Content Trust**

Docker Content Trust (DCT) makes it simple and easy to verify the integrity and the publisher of images that

you download and run. This is especially important when pulling images over untrusted networks such as the

internet.

At a high level, DCT allows developers to sign images when they are pushed to Docker Hub or other container

registries. These images can then be veriﬁed when they are pulled and ran. This high-level process is shown in

Figure 15.11





232

15: Security in Docker

**Figure 15.11**

DCT can also be used to provide important *context*. This includes; whether or not an image has been signed for

use in a particular environment such as “prod” or “dev”, or whether an image has been superseded by a newer

version and is therefore stale.

The following steps will walk you through conﬁguring Docker Content Trust, signing and pushing an image, and

then pulling the signed image. To follow along, you’ll need a cryptographic key-pair to sign images.

If you don’t already have a key-pair, you can use the docker trust sub-command to generate a new key-pair

one. The following command generates a new key-pair called “nigel”.

$ docker trust key generate nigel

Generating key for nigel...

Enter passphrase for new nigel key with ID 1f78609:

Repeat passphrase for new nigel key with ID 1f78609:

Successfully generated and loaded private key.... public key available: /root/nigel.pub

If you already have a key-pair, you can import and load it with docker trust key load key.pem --name nigel.

Now that you’ve loaded a valid key-pair, you need to associate it with the image repository you’ll be pushing

signed images to. This example uses the nigelpoulton/dct repo on Docker Hub and the nigel.pub key that was

created by the previous docker trust key generate command. Your key ﬁle will be different.

$ docker trust signer add --key nigel.pub nigel nigelpoulton/dct

Adding signer "nigel" to nigelpoulton/dct...

Initializing signed repository for nigelpoulton/dct...

Enter passphrase for root key with ID aee3314:

Enter passphrase for new repository key with ID 1a18dd1:

Repeat passphrase for new repository key with ID 1a18dd1:

Successfully initialized "nigelpoulton/dct"

Successfully added signer: nigel to nigelpoulton/dct

The following command will sign the nigelpoulton/dct:signed image **and** push it to Docker Hub.





233

$ docker trust sign nigelpoulton/dct:signed

Signing and pushing trust data for local image nigelpoulton/dct:signed,

may overwrite remote trust data

The push refers to repository [docker.io/nigelpoulton/dct]

1a777bda846c: Mounted from nigelpoulton/dct

d23c343f7626: Mounted from nigelpoulton/dct

18dc259b4479: Mounted from nigelpoulton/dct

40a236c21a47: Mounted from nigelpoulton/dct

a9a7f132e4de: Mounted from nigelpoulton/dct

9a8b7b2b0c33: Mounted from nigelpoulton/dct

00891a9058ec: Mounted from nigelpoulton/dct

d87eb7d6daff: Mounted from nigelpoulton/dct

beee9f30bc1f: Mounted from nigelpoulton/dct

signed: digest: sha256:c9f8e18822...6cbb9a74cf size: 2202

Signing and pushing trust metadata

Enter passphrase for nigel key with ID 1f78609:

Successfully signed docker.io/nigelpoulton/dct:signed

Once the image is pushed, you can inspect its signing data with the following command.

$ docker trust inspect nigelpoulton/dct:signed --pretty

Signatures for nigelpoulton/dct:signed

SIGNED TAG

signed

DIGEST

SIGNERS

nigel

c9f8c18522...75aaccd6cbb9a74cf

List of signers and their keys for nigelpoulton/dct:signed

SIGNER

nigel

KEYS

1f786095c467

Administrative keys for nigelpoulton/dct:signed

Repository Key:

Root Key:

1a18dd1113...a91f489782

c2f53fd2f2...b0a720d344

You can force a Docker host to always sign and verify image push and pull operations by exporting the DOCKER\_-

CONTENT\_TRUST environment variable with a value of 1. In the real world, you’ll want to make this a more

permanent feature of Docker hosts.

$ export DOCKER\_CONTENT\_TRUST=1

Once DCT is enabled, you’ll no longer be able to pull and work with unsigned images. You can test this behavior

by aempting to pull the following two images:

• nigelpoulton/dct:unsigned

• nigelpoulton/dct:signed

If you have enabled DCT by seing the DOCKER\_CONTENT\_TRUST environment variable, you will not be able to

pull the dct:unsigned image. However, you will be able to pull the image tagged as signed.





234

15: Security in Docker

$ docker image pull nigelpoulton/dct:unsigned

No valid trust data for unsigned

Docker Content Trust is an important technology for helping you verify the images you are pulling from container

registries. It’s simple to conﬁgure in its basic form, but more advanced features, such as *context*, can be more

complex to conﬁgure.

**Docker Secrets**

Many applications need secrets — things like passwords, TLS certiﬁcates, SSH keys, and more.

Early versions of Docker had no standardised way of making secrets available to apps in a secure way. It was

common for developers to insert secrets into apps via plain text environment variables (we’ve all done it). This

was far from ideal.

Docker 1.13 introduced *Docker Secrets* as ﬁrst-class objects in the Docker API.

Behind the scenes, secrets are encrypted at rest, encrypted in-ﬂight, mounted in containers to in-memory

ﬁlesystems, and operate under a least-privilege model where they are only made available to services that have

been explicitly granted access to them. It’s quite a comprehensive end-to-end solution, and it even has its own

docker secret sub-command.

Figure 15.12 shows a high-level workﬂow:

**Figure 15.12**

The following steps walk through the high-level workﬂow shown in Figure 15.12.

\1. The blue secret is created and posted to the Swarm

\2. It gets stored in the encrypted cluster store (all managers have access to the cluster store)

\3. The blue service is created and the secret is attached to it





235

\4. The secret is encrypted in-ﬂight while it is delivered to the tasks (containers) in the blue service

\5. The secret is mounted into the containers of the blue service as an unencrypted ﬁle at /run/secrets/.

This is an in-memory tmpfs ﬁlesystem (this step is different on Windows Docker hosts as they do not have

the notion of an in-memory ﬁlesystem like tmpfs)

\6. Once the container (service task) completes, the in-memory ﬁlesystem is torn down and the secret ﬂushed

from the node

\7. The red containers in the red service cannot access the secret

The reason that secrets are surfaced in their un-encrypted form in running containers is so applications can use

them without requiring methods to decrypt them.

You can create and manage secrets with the docker secret sub-command, and you can attach them to services

by specifying the --secret ﬂag to the docker service create command.

**Chapter Summary**

Docker can be conﬁgured to be extremely secure. It supports all of the major Linux security technologies,

including; kernel namespaces, cgroups, capabilities, MAC, and seccomp. It ships with sensible defaults for all

of these, but you can customize them and even disable them.

Over and above the general Linux security technologies, Docker includes an extensive set of its own security

technologies. Swarm Mode is built on TLS and is extremely simple to conﬁgure and customize. Image scanning

can perform binary-level scans of images and provide detailed reports of known vulnerabilities. Docker Content

Trust lets you sign and verify content, and Docker Secrets allow you to securely share sensitive data with

containers and Swarm services.

The net result is that your Docker environment can be conﬁgured to be as secure or insecure as you desire — it

all depends on how you conﬁgure it.









**16: What next**

Hopefully you’re feeling conﬁdent with Docker and ready to take your next steps. Fortunately, taking those next

steps has never been easier.

**Practice makes perfect**

There’s no substitute for hands-on practice. Fortunately, as we showed in Chapter 3, it’s easier than ever to get

Docker and start developing your hands-on skills. I personally use Docker Desktop every day and regularly use

[Play](https://play-with-docker.com/)[ ](https://play-with-docker.com/)[with](https://play-with-docker.com/)[ ](https://play-with-docker.com/)[Docker](https://play-with-docker.com/)[¹⁸](#br248)[ ](#br248)when I need to test something quitely with multiple nodes.

**Video training**

Video training is another way to learn things and see stuff in action.

I’ve created a lot of high-quality Docker video training courses at [Pluralsight](http://app.pluralsight.com/author/nigel-poulton)[¹⁹](#br248). If you’re not a member of

Pluralsight then become one! It costs money, but it could be one of the most important career investments you

ever make. And if you’re unsure about parting with your hard-earned money… they always have a free trial

where you can get free access to my courses for a limited period.

**Get involved with the community**

There’s a vibrant Docker community full of helpful people. Get involved with Docker groups and ats on the

internet, and look-up your local Docker or cloud-native meetup (search Google for “Docker meetup near me”). I

regularly present at meetups and they’re a great place to network with people and learn.

**Kubernetes**

Now that you know a thing or two about Docker, a logical next-step might be Kubernetes. Without going into

detail, Kubernetes is similar to Docker Swarm but has a larger scope and a more active community. It’s also

notoriously hard to learn. However, now that you know Docker and how swarm orchestration works, learning

Kubernetes will be easier. That said, if you don’t **need** all the extras that Kubernetes brings, you might be better

stiing with Swarm.

¹⁸<https://play-with-docker.com/>

¹⁹<http://app.pluralsight.com/author/nigel-poulton>





238

16: What next

**Feedback and connecting**

Massive thanks for reading my book, I really hope it was useful. It’d be magic if you could leave a review on

Amazon.

Feel free to connect with me on [Twier](https://twitter.com/nigelpoulton)[²⁰](#br249), [LinkedIn](https://www.linkedin.com/in/nigelpoulton/)[ ](https://www.linkedin.com/in/nigelpoulton/)and other places you can ﬁnd me (I don’t accept Facebook

friends requests. It’s nothing personal, I just keep Facebook for family and old friends.

•

•

•

Twitter (@nigelpoulton)

LinkedIn (https://www.linkedin.com/in/nigelpoulton/)

nigelpoulton.com

• YouTube: Nigel Poulton - KubeTrainer

²⁰<https://twitter.com/nigelpoulton>





239

