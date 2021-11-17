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
