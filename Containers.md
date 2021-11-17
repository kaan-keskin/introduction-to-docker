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
