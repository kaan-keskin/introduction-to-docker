# Introduction to Docker - Part 2

Editors: **Taha Osman Sarıaslan, Semih Teker, Kaan Keskin**

Date: October 2021

Available at: https://github.com/kaan-keskin/introduction-to-docker

**Resources:**

> - Docker Deep Dive - Zero to Docker in a single book - Nigel Poulton @nigelpoulton
> - Containers Fundamentals (LFS253) - Linux Foundation
> - DevOps Lecture Notes - California Institute of Technology
> - Wikipedia - www.wikipedia.com

**LEGAL NOTICE: This document is created for educational purposes, and it can not be used for any commercial purposes. If you find this document useful in any means please support the original authors for ethical reasons.** 

**Please visit this page and buy kindle/digital version of the book:**
**https://www.amazon.com/Docker-Deep-Dive-Nigel-Poulton-ebook/dp/B01LXWQUFF/**

**Content**

> - Containerizing an app
> - Deploying Apps with Docker Compose
> - Docker Swarm

## Containerizing an app

Docker is all about taking applications and running them in containers. The process of taking an application and conﬁguring it to run as a container is called “containerizing”.

Containers are all about making apps simple to **build**, **ship**, and **run**. The process of containerizing an app looks like this:

1. Start with your application code and dependencies

2. Create a *Dockerﬁle* that describes your app, its dependencies, and how to run it

3. Feed the *Dockerﬁle* into the docker image build command

4. Push the new image to a registry (optional)

5. Run container from the image

Once your app is containerized (made into a container image), you’re ready to share it and run it as a container.

<img src=".\images\Containerizing.png" style="width:75%; height: 75%;"/>

### Containerize a single-container app

The rest of this chapter walks through the process of containerizing a simple Node.js web app. 
We’ll complete the following high-level steps:

- Clone the repo to get the app code
- Inspect the Dockerﬁle
- Containerize the app
- Run the app
- Test the app
- Look a bit closer
- Move to production with **Multi-stage Builds**
- A few best practices 

The example in this chapter is of a single-container app. 
The next chapter will include a slightly more complex multi-container app, and we’ll move on to an even more complicated app in the chapter on Docker Stacks.

### Getting the application code

The application used in this example is available on GitHub at:

• https://github.com/nigelpoulton/psweb.git

Clone the sample app from GitHub.

```sh
$ git clone https://github.com/nigelpoulton/psweb.git
```

The clone operation creates a new directory called psweb. Change directory into psweb and list its contents.

```sh
$ cd psweb
$ ls -l
```

This directory contains all of the application source code, as well as subdirectories for views and unit tests. Feel free to look at the ﬁles — the app is extremely simple. We won’t be using the unit tests in this chapter. Now that we have the app code, let’s look at its Dockerﬁle.

### Inspecting the Dockerﬁle

A **Dockerﬁle** is the starting point for creating a container image — it describes an application and tells Docker how to build it into an image. The directory containing the application and dependencies is referred to as the *build context*. It’s a common practice to keep your Dockerﬁle in the root directory of the *build context*. It’s also important that **Dockerﬁle** starts with a capital “**D**” and is all one word. “Dockerﬁle” and “Docker ﬁle” are not valid. Let’s look at the contents of the Dockerﬁle.

```sh
$ cat Dockerfile
```

Do not underestimate the impact of the Dockerﬁle as a form of documentation. It’s a great document for bridging the gap between dev and ops. It also has the power to speed up on-boarding of new developers etc. This is because the ﬁle accurately describes the application and its dependencies in an easy-to-read format. You should treat it like you treat source code and check it into a version control system.

All Dockerﬁles start with the FROM instruction. This will be the base layer of the image, and the rest of the app will be added on top as additional layers. This particular application is a Linux app, so it’s important that the FROM instruction refers to a Linux-based image. 

Next, the Dockerﬁle creates a LABEL that speciﬁes “nigelpoulton@hotmail.com” as the maintainer of the image. Labels are simple key-value pairs and are an excellent way of adding custom metadata to an image. It’s considered a best practice to list a maintainer of an image so that other potential users have a point of contact when working with it. 

The RUN apk add --update nodejs nodejs-npm instruction uses the Alpine apk package manager to install nodejs and nodejs-npm into the image. It creates a new image layer directly above the Alpine base layer, and installs the packages in this layer. 

The COPY . /src instruction creates another new layer and copies in the application and dependency ﬁles from the *build context*.

Next, the Dockerﬁle uses the WORKDIR instruction to set the working directory inside the image ﬁlesystem for the rest of the instructions in the ﬁle. This instruction does not create a new image layer. 

Then the RUN npm install instruction creates a new layer and uses npm to install application dependencies listed in the package.json ﬁle in the build context. It runs within the context of the WORKDIR set in the previous instruction, and installs the dependencies into the newly created layer. 

<img src=".\images\Dockerfile.png" style="width:75%; height: 75%;"/>

The application exposes a web service on TCP port 8080, so the Dockerﬁle documents this with the EXPOSE 8080 instruction. This is added as image metadata and not an image layer. Finally, the ENTRYPOINT instruction is used to set the main application that the image (container) should run. This is also added as metadata and not an image layer.

### Containerize the app/build the image

Now that we understand how it works, let’s build it! The following command will build a new image called web:latest. The period (.) at the end of the command tells Docker to use the shell’s current working directory as the *build context*. Be sure to include the trailing period (.) and be sure to run the command from the psweb directory that contains the Dockerﬁle and application code.

```sh
$ docker image build -t web:latest .
```

Check that the image exists in your Docker host’s local repository.

```sh
$ docker image ls
```

You can use the docker image inspect web:latest command to verify the conﬁguration of the image. It will list all of the settings that were conﬁgured from the Dockerﬁle. Look out for the list of image layers and the Entrypoint command.

### Dockerfile Overview

<img src=".\images\dockerfile-overview.png" style="width:75%; height: 75%;"/>

<img src=".\images\dockerfile-overview-2.png" style="width:75%; height: 75%;"/>

### Dockerfile Format

<img src=".\images\dockerfile-format.png" style="width:75%; height: 75%;"/>

### Dockerfile Instructions

<img src=".\images\dockerfile-instructions.png" style="width:75%; height: 75%;"/>

There are several instructions available to aid us in building the desired images. The full list of instructions is available in Docker's official Documentation. The instructions can be divided into two distinct categories - build time and run time instructions.

Build Time instructions are used to build the image from the Dockerfile and Run Time instructions are used while a container is starting from the pre-built image.

#### Build Time Instructions

**FROM**

This instruction initializes a new build stage and defines the base image used to build our resulting image. Dockerfile must start with this instruction, but it may be preceded only by ARG instructions to define arguments to be used by subsequent FROM instructions. Multiple FROM instructions can be found in Dockerfile.

```dockerfile
FROM [--platform=<platform>]<image>[:<tag>][AS <name>]
FROM [--platform=<platform>]<image>[@<digest>][AS <name>]

FROM --platform=linux/amd64 alpine:3.10 AS base-alpine
```

**ARG**

This instruction may be placed before FROM, or after it. When it is found before FROM, it is considered outside of the build stage, and its value cannot be used in any instruction after FROM. However, the default value of an ARG declared before the first FROM can be invoked with an ARG instruction followed by the ARG name and no value. When the ​ARG​ instruction is declared, we can pass a variable at build time:

```dockerfile
ARG <name>[=<default value>
ARG BUILD_NO
```
```shell
$ docker build --build-arg BUILD_NO=v2
```

**RUN**

This instruction is used to run commands inside the intermediate container created from the base image during the build process, and commit the results as a new image. RUN may be used in both shell and exec forms. In shell form the command is run by default in a shell such as /bin/sh -c in Linux or cmd /S /C in Windows. However, the shell can be modified by passing a desired shell, or with the SHELL command.

```dockerfile
RUN ["executable", "param1", "param2"]  # exec form
RUN <command>                           # shell form
RUN ["/bin/bash", "-c", "echo New Shell"] # exec form
RUN /bin/bash -c 'echo New Shell'.        # shell form 
```

**LABEL**

It adds metadata to the resulting image in the key-value pair format:

```dockerfile
LABEL <key1>=<value1> <key2>=<value2> <key3>=<value3> ...
LABEL version="1.0" env="dev"
```

**EXPOSE**

This instruction defines network ports we want to open from the container, for external entities to connect with the container on those exposed ports:

```dockerfile
EXPOSE <port> [<port>/<protocol>...]
EXPOSE 80/tcp
EXPOSE 80/udp
```

**COPY**

This instruction allows content to be copied from our build context to the intermediate container which would get committed to create the resulting image. It has the following forms:

```dockerfile
COPY [--chown=<user>:<group>]<src>... <dest>
COPY [--chown=<user>:<group>]["<src>",... "<dest>"]
```

The second form is required when the source name contains white spaces.

```dockerfile
COPY --chown=2000:mygroup /host/path/file* /container/filesystem/
```

**ADD**

Similar to COPY, but it provides more features, such as accepting a URL for source, and accepting a tar file as source which is extracted at destination.

```dockerfile
ADD [--chown=<user>:<group>]<src>... <dest>
ADD [--chown=<user>:<group>]["<src>",... "<dest>"]
ADD https://raw.githubusercontent.com/lfstudent/image.png /downloads/logo.png
```

**WORKDIR**

This instruction sets the working directory for any RUN​, CMD, ENTRYPOINT, COPY and ADD instructions that follow it in the Dockerfile:

```dockerfile
WORKDIR /path/to/workdir
```

In the example below, run.sh​ would run from /code​:

```dockerfile
RUN mkdir /code
COPY run.sh /code
WORKDIR /code
CMD run.sh
```

**ENV**

This instruction sets environment variables inside the container:

```dockerfile
ENV <key>=<value>
ENV WEB="192.168.2.3"
```

**VOLUME**

To create a mount point and mount external storage on it:

```dockerfile
VOLUME ["/path/data"]
VOLUME /path/data
```

**USER**

To set the user name or UID of the resulting image for any subsequent ​RUN​, ​CMD and ENTRYPOINT​ instructions that follow it in the Dockerfile.

```dockerfile
USER <user>[:<group>]
USER <UID>[:<GID>]

USER student:student
USER 1000:1000
```

**ONBUILD**

Defines instructions to be executed at a later time, when the resulting image from the current build process becomes the base image of any other build:

```dockerfile
ONBUILD <INSTRUCTION>
ONBUILD RUN pip install docker --upgrade
```

For example, let's take the image created from the ​ Dockerfile​ which has the instruction lfstudent/python:onbuild​ . This image becomes the base image in ​ Dockerfile​ , like the following:

```dockerfile
FROM lfstudent/python:onbuild
...
```

Then, while creating the image, we would see this message:

```shell
Step 1 : FROM lfstudent/python:onbuild
# Executing 1 build trigger...
Step 1 : RUN pip install docker --upgrade
---> Running in c5503d7c1475
...
```

**STOPSIGNAL**

This instruction allows us to set a system call signal that will be sent to a container to exit, such as 9, SIGNAME, or SIGKILL:

```dockerfile
STOPSIGNAL signal
```

This is useful for the definition of a custom exit signal for our container.

**HEALTHCHECK**

At times, while our container is running, the application inside may have crashed. Ideally, a container with such behavior should be stopped. To prevent a container from reaching that state, application readiness and liveliness are defined with the HEALTHCHECK​ instruction, available in the following forms:

```dockerfile
HEALTHCHECK [OPTIONS] CMD command
HEALTHCHECK NONE
HEALTHCHECK --interval=5m --timeout=3s CMD curl -f http://localhost/ || exit 1
```

**SHELL**

To define a new default shell for commands that run in Shell form​, if the defaults are not desired (default in Linux is ​ /bin/sh -c, while in Windows is ​ cmd /S /C​. With the​ SHELL​ instruction we may change the default shell used to run commands in shell form.

```dockerfile
SHELL ["/bin/bash", "-c" ]
```

#### Run Time Instructions

**CMD**

This is a runtime instruction executed when the container is started from the resulting image. There can only be one CMD​ instruction in a Dockerfile​. If there are more than one CMD​ instructions, then only the last one would take effect. The CMD​ instruction provides defaults to the container, which get executed from the resulting image. 

CMD is used in three forms:
- Shell form: CMD command param1 param2
- Exec form (preferred): CMD ["executable","param1","param2"]
- As default parameters to ENTRYPOINT - CMD ["param1","param2"]

In the Dockerfile of nginx​ container image we noticed the following:

```dockerfile
CMD ["nginx", "-g", "daemon off;"]
```

It means that when the container starts from the nginx​ image, by default, it runs the nginx -g daemon off; command to start the nginx daemon​. We can override the defaults defined by CMD​. In the following example, we are starting the /bin/sh​ application instead of the nginx daemon at start time.

```shell
$ docker run -it nginx /bin/sh
```

**ENTRYPOINT**

Similar to CMD​, ENTRYPOINT​ is also a runtime instruction. But the executable/command provided during the build time cannot be overridden at runtime. Although we can change the arguments to it, whatever is passed after the executable/command, is considered an argument to the executable/command. There can only be one ENTRYPOINT​ instruction. 

It is used in two forms: 
- Shell form: ENTRYPOINT command param1 param2
- Exec form: ENTRYPOINT ["executable", "param1", "param2"]

CMD​ can be used in conjunction with ENTRYPOINT​, but in that case, CMD​ provides the default arguments to ENTRYPOINT.

### Exclude Files and Directories from Build with .dockerignore

During an image build, the Docker client zips the referenced context folder and sends it to the Docker Host. If we want to exclude some files and directories during the ​ docker image build​ process, then we should list them in the .dockerignore​ file in the referenced folder:

```shell
$ cat .dockerignore

code
tmp
test.py
```

### Pushing images

Once you’ve created an image, it’s a good idea to store it in an image registry to keep it safe and make it available to others. Docker Hub is the most common public image registry, and it’s the default push location for docker image push commands. In order to push an image to Docker Hub, you need to login with your Docker ID. You also need to tag the image appropriately.

Let’s log in to Docker Hub and push the newly created image. In the following example’s you will need to substitute my Docker ID with your own. 

```sh
$ docker login
```

Before you can push an image, you need to tag it in a special way. 
This is because Docker needs all of the followinginformation when pushing an image:

- Registry
- Repository
- Tag

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

### Run the app

The containerized application is a web server that listens on TCP port 8080. You can verify this in the app.js ﬁle in the build context you cloned from GitHub. The following command will start a new container called c1 based on the web:latest image you just created. It maps port 80 on the Docker host, to port 8080 inside the container. This means that you’ll be able to point a web browser at the DNS name or IP address of the Docker host running the container and access the app.

**Note:** If your host is already running a service on port 80, you can specify a different port as part of the docker container run command. For example, to map the app to port 5000 on the Docker host, use the -p 5000:8080 ﬂag.

```shell
$ docker container run -d --name c1 -p 80:8080 web:latest
```

The -d ﬂag runs the container in the background, and the -p 80:8080 ﬂag maps port 80 on the host to port 8080 inside the running container. Check that the container is running and verify the port mapping.

```sh
$ docker container ls
```

The output above is snipped for readability, but shows that the app container is running. Note that port 80 is mapped, on all host interfaces (0.0.0.0:80).

### Test the app

Open a web browser and point it to the DNS name or IP address of the host that the container is running on. If the test does not work, try the following:

1. Make sure that the container is up and running with the docker container ls command. The container name is c1 and you should see the port mapping as 0.0.0.0:80->8080/tcp.

2. Check that ﬁrewall and other network security settings are not blocking traffic to port 80 on the Docker host.

3. Retry the command specifying a high numbered port on the Docker host (may be -p 5000:8080).

### Looking a bit closer

Now that the application is containerized, let’s take a closer look at how some of the machinery works. The docker image build command parses the Dockerﬁle one-line-at-a-time starting from the top. Comment lines start with the # character. All non-comment lines are **Instructions** and take the format INSTRUCTION argument. Instruction names are not case sensitive, but it’s normal practice to write them in UPPERCASE. This makes reading the Dockerﬁle easier.

**Some instructions create new layers, whereas others just add metadata to the image conﬁg ﬁle. Examples of instructions that create new layers are FROM, RUN, and COPY. Examples that create metadata include EXPOSE, WORKDIR, ENV, and ENTRYPOINT.** 

**The basic premise is this — if an instruction is adding *content* such as ﬁles and programs to the image, it will create a new layer. If it is adding instructions on how to build the image and run the application, it will create metadata.** You can view the instructions that were used to build the image with the docker image history command.

```sh
$ docker image history web:latest
```

Two things from the output above are worth noting. 

First of all, each line corresponds to an instruction in the Dockerﬁle (starting from the bottom and working up). The CREATED BY column even lists the exact Dockerﬁle instruction that was executed. 

Secondly, only 4 of the lines displayed in the output create new layers (the ones with non-zero values in the SIZE column). These correspond to the FROM, RUN, and COPY instructions in the Dockerﬁle. Although the other instructions might look like they create layers, they actually create metadata instead of layers. The reason that the docker image history output makes it looks like all instructions create layers is an artefact of the way builds and image layering used to work. 

Use the docker image inspect command to conﬁrm that only 4 layers were created.

```sh
$ docker image inspect web:latest
```

It is considered a good practice to use images from official repositories with the FROM instruction. You can view the output of the docker image build command to see the general process for building an image. 

As the following snippet shows, the basic process is: 

- spin up a temporary container 
- run the Dockerfile instruction inside of that container 
- save the results as a new image layer 
- remove the temporary container.

### Moving to production with Multi-stage Builds

When it comes to Docker images, big is bad! Big means slow. Big means hard to work with. And big means more potential vulnerabilities and possibly a bigger attack surface! For these reasons, Docker images should be small. The aim of the game is to only ship production images with the stuff **needed** to run your app in production. The problem is keeping images small *was* hard work. For example, the way you write your Dockerﬁles has a huge impact on the size of your images. A common example is that every RUN instruction adds a new layer. As a result, it’s usually considered a best practice to include multiple commands as part of a single RUN instruction — all glued together with double-ampersands (&&) and backslash (\) line-breaks. While this isn’t rocket science, it requires time and discipline. Another issue is that we don’t clean up after ourselves. We’ll RUN a command against an image that pulls some build-time tools, and we’ll leave all those tools in the image when we ship it to production. Not ideal! Multi-stage builds to the rescue! Multi-stage builds are all about optimizing builds without adding complexity. And they deliver on the promise! Here’s the high-level.

Multi-stage builds have a single Dockerﬁle containing multiple FROM instructions. Each FROM instruction is a new **build stage** that can easily COPY artefacts from previous **stages**. 

Example app is available at https://github.com/nigelpoulton/atsea-sample-shop-app.git and the Dockerﬁle is in the app directory. It’s a Linux-based application so, will only work on a Linux Docker host. It’s also quite old, so don’t deploy it to an important system, and be sure to delete it as soon as you’re ﬁnished. The ﬁrst thing to note is that the Dockerﬁle has three FROM instructions. Each of these constitutes a distinct **build stage**. Internally, they’re numbered from the top starting at 0. However, we’ve also given each stage a friendly name.

- Stage 0 is called storefront
- Stage 1 is called appserver
- Stage 2 is called production

The storefront stage pulls the node:latest image which is over 900MB in size. The resulting image is an even bigger than the base node:latest image as it contains lots of build stuff and not very much app code.

The appserver stage pulls the maven:latest image which is over 500MB in size. This produces another very large image with lots of build tools and very little actual production code.

The production stage starts by pulling the java:8-jdk-alpine image. This image is approximately 150MB -

Considerably smaller than the node and maven images used by the previous build stages. Finally, it sets the main application for the image to run when it’s started as a container. An important thing to note, is that COPY --from instructions are used to **only copy production-related application code** from the images built by the previous stages. They do not copy build artefacts that are not needed for production. It’s also important to note that we only need a single Dockerﬁle, and no extra arguments are needed for the docker image build command!

```sh
$ docker image build -t multi:stage .
```

Run a docker image ls to see the list of images pulled and created by the build operation.

```sh
$ docker image ls
```

The top line in the output above shows the node:latest image pulled by the storefront stage. The image below is the image produced by that stage (created by adding the code and running the npm install and build operations). Both are very large images with lots of build junk included. The 3rd and 4th lines are the images pulled and produced by the appserver stage. These are both large and contain lots of builds tools. The last line is the multi:stage image built by the ﬁnal build stage in the Dockerﬁle (stage2/production). You can see that this is signiﬁcantly smaller than the images pulled and produced by the previous stages. This is because it’s based off the much smaller java:8-jdk-alpine image and has only added the production-related app ﬁles from the previous stages. The net result is a small production image created by a single Dockerﬁle, a normal docker image build command, and zero additional scripting! Multi-stage builds were new with Docker 17.05 and are an excellent feature for building small production-worthy images.

### Leverage the build cache

When building an image, Docker steps through the instructions in your Dockerfile, executing each in the order specified. As each instruction is examined, Docker looks for an existing image in its cache that it can reuse, rather than creating a new (duplicate) image. If you do not want to use the cache at all, you can use the **--no-cache=true** option on the docker build command. However, if you do let Docker use its cache, it is important to understand when it can, and cannot, find a matching image.

### Squash the image

Squashing an image isn’t really a best practice as it has pros and cons. At a high level, Docker follows the normal process to build an image, but then adds an additional step that squashes everything into a single layer. Squashing can be good in situations where images are starting to have a lot of layers and this isn’t ideal. An example might be when creating a new base image that you want to build other images from in the future — this base is much better as a single-layer image. On the negative side, squashed images do not share image layers. This can result in storage ineﬃciencies and larger push and pull operations. Add the **--squash** ﬂag to the docker image build command if you want to create a squashed image. The squashed image will also need to send every byte to Docker Hub on a docker image push command, whereas the non-squashed image only needs to send unique layers.

<img src=".\images\ContainerSquash.png" style="width:75%; height: 75%;">

### Use no-install-recommends

If you are building Linux images, and using the apt package manager, you should use the **--no-install-recommends** ﬂag with the apt-get install command. This makes sure that apt only installs main dependencies (packages in the Depends ﬁeld) and not recommended or suggested packages. This can greatly reduce the number of unwanted packages that are downloaded into your images.

## Deploying Apps with Docker Compose

In this chapter, we’ll look at how to deploy multi-container applications using Docker Compose. Docker Compose and Docker Stacks are very similar. In this chapter we’ll focus on Docker Compose, which deploys and manages multi-container applications on Docker nodes running in **single-engine mode**. In a later chapter, we’ll focus on Docker Stacks. Stacks deploy and manage multi-container apps on Docker nodes running in **swarm mode**.

Modern cloud-native apps are made of multiple smaller services that interact to form a useful app. We call this pattern “microservices”. A simple example might be an app with the following seven services:

- Web front-end
- Ordering
- Catalog
- Back-end database
- Logging
- Authentication
- Authorization

Get all of these working together, and you have a *useful application*.

Deploying and managing lots of small microservices like these can be hard. This is where *Docker Compose* comes in to play.

Instead of gluing each microservice together with scripts and long docker commands, Docker Compose lets you describe an entire app in a single declarative conﬁguration ﬁle, and deploy it with a single command.

Once the app is *deployed*, you can *manage* its entire lifecycle with a simple set of commands. You can even store and manage the conﬁguration ﬁle in a version control system.

### Compose background

In the beginning was *Fig*. Fig was a powerful tool, created by a company called *Orchard*, and it was the best way to manage multi-container Docker apps. It was a Python tool that sat on top of Docker, and let you deﬁne entire multi-container apps in a single YAML ﬁle. You could then deploy and manage the lifecycle of the app with the fig command-line tool.

Behind the scenes, Fig would read the YAML ﬁle and use Docker to deploy and manage the app via the Docker API. It was a good thing. 

In fact, it was so good, that Docker, Inc. acquired Orchard and re-branded Fig as *Docker Compose*. The command-line tool was renamed from fig to docker-compose, and continues to be an external tool that gets bolted on top of the Docker Engine. Even though it’s never been fully integrated into the Docker Engine, it’s always been popular and widely used.

As things stand today, Compose is still an external Python binary that you have to install on a Docker host. You define multi-container (microservices) apps in a YAML file, pass the YAML file to the docker-compose command line, and Compose deploys it via the Docker API. 

However, April 2020 saw the announcement of the Compose Specification. It is aimed at creating an open standard for defining multi-container cloud-native apps. The ultimate aim being to greatly simplify the code-to-cloud process.

The speciﬁcation will be community-led and separate from the docker-compose implementation from Docker,Inc. This helps maintain better governance and clearer lines of demarcation. However, we should expect Docker to implement the ﬁll spec in docker-compose.

The spec itself is a great document to learn the details. 

Time to see it in action.

### Installing Compose

Docker Compose is available on multiple platforms. In this section we’ll demonstrate *some* of the ways to install it on Windows, Mac, and Linux. More installation methods exist, but the ones we show here will get you started.

### Installing Compose on Linux

Installing Docker Compose on Linux is a two-step process. First, you download the binary using the curl command. Then you make it executable using chmod. For Docker Compose to work on Linux, you’ll need a working version of the Docker Engine. The following command will download version 1.25.5 of Docker Compose and copy it to /usr/bin/local. You can check the releases page on [GitHub](https://github.com/docker/compose/releases)[¹³](#br123)[ ](#br123) for the latest version and replace the 1.25.5 in the URL with the version you want to install.

The command may wrap over multiple lines in the book. If you run the command on a single line you will need to remove any backslashes (\).

```sh
$ sudo curl -L \
"https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" \
-o /usr/local/bin/docker-compose
```

Now that you’ve downloaded the docker-compose binary, use the following chmod command to make it executable.

```sh
$ sudo chmod +x /usr/local/bin/docker-compose
```

Verify the installation and check the version.

```sh
$ docker-compose --version
docker-compose version 1.25.5, build 1110ad01
```

You’re ready to use Docker Compose on Linux.

You can also use pip to install Compose from its Python package. But I don’t want to waste valuable pages showing every possible installation method. Enough is enough, time to move on.

### Compose ﬁles

Compose uses YAML ﬁles to deﬁne multi-service applications. YAML is a subset of JSON, so you can also use JSON. However, all the examples in this chapter will be YAML. The default name for a Compose YAML ﬁle is docker-compose.yml. However, you can use the -f ﬂag to specify custom ﬁlenames.

The following example shows a very simple Compose ﬁle that deﬁnes a small Flask app with two microservices (web-fe and redis). The app is a simple web server that counts the number of visits to a web page and stores the value in Redis. We’ll call the app counter-app and use it as the example application for the rest of the chapter.

```yaml
version: "3.8"
services:
    web-fe:
        build: .
        command: python app.py
        ports:
            - target: 5000
              published: 5000
        networks:
            - counter-net
        volumes:
            - type: volume
              source: counter-vol
              target: /code
    redis:
        image: "redis:alpine"
        networks:
            counter-net:

networks:
    counter-net:

volumes:
    counter-vol:
```

We’ll skip through the basics of the ﬁle before taking a closer look.

The ﬁrst thing to note is that the ﬁle has 4 top-level keys:
- version
- services
- networks
- volumes

Other top-level keys exist, such as secrets and configs, but we’re not looking at those right now.

The version key is mandatory, and it’s always the ﬁrst line at the root of the ﬁle. This deﬁnes the version of the Compose ﬁle format (basically the API). You should normally use the latest version.

It’s important to note that the versions key does not deﬁne the version of Docker Compose or the Docker Engine. For information regarding compatibility between versions of the Docker Engine, Docker Compose, and the Compose ﬁle format, google “Compose ﬁle versions and upgrading”.

For the remainder of this chapter we’ll be using version 3 or higher of the Compose ﬁle format.

The top-level services key is where you deﬁne the diﬀerent application microservices. This example deﬁnes two services; a web front-end called web-fe, and an in-memory database called redis. Compose will deploy each of these services as its own container.

The top-level networks key tells Docker to create new networks. By default, Compose will create bridge networks. These are single-host networks that can only connect containers on the same Docker host. However, you can use the driver property to specify diﬀerent network types.

The following code can be used in your Compose ﬁle to create a new *overlay* network called over-net that allows standalone containers to connect to it (attachable).

```yaml
networks:
    over-net:
        driver: overlay
        attachable: true
```

The top-level volumes key is where you tell Docker to create new volumes.

### Our speciﬁc Compose ﬁle

The example ﬁle we’ve listed uses the Compose version 3.8 ﬁle format, deﬁnes two services, deﬁnes a network called counter-net, and deﬁnes a volume called counter-vol.

Most of the detail is in the services section, so let’s take a closer look at that.

The services section has two second-level keys:
- web-fe
- redis

each of these deﬁnes a service (container) in the app. It’s important to understand that Compose will deploy each of these as a container, and it will use the name of the keys as part of the container names. In our example, we’ve deﬁned two keys; web-fe and redis. This means Compose will deploy two containers, one will have web-fe in its name and the other will have redis.

Within the deﬁnition of the web-fe service, we give Docker the following instructions:

    - build: . This tells Docker to build a new image using the instructions in the Dockerfile in the current directory (.). The newly built image will be used in a later step to create the container for this service.

    - command: python app.py This tells Docker to run a Python app called app.py as the main app in the container. The app.py ﬁle must exist in the image, and the image must contain Python. The Dockerﬁle takes care of both of these requirements.
    
    - ports: Tells Docker to map port 5000 inside the container (target) to port 5000 on the host (published). This means that traﬃc sent to the Docker host on port 5000 will be directed to port 5000 on the container. The app inside the container listens on port 5000.
    
    - networks: Tells Docker which network to attach the service’s container to. The network should already exist, or be deﬁned in the networks top-level key. If it’s an overlay network, it will need to have the attachable ﬂag so that standalone containers can be attached to it (Compose deploys standalone containers instead of Docker Services).
    
    - volumes: Tells Docker to mount the counter-vol volume (source) to /code (target) inside the container. The counter-vol volume needs to already exist, or be deﬁned in the volumes top-level key at the bottom of the ﬁle.

In summary, Compose will instruct Docker to deploy a single standalone container for the web-fe service. It will be based on an image built from a Dockerﬁle in the same directory as the Compose ﬁle. This image will be started as a container and run app.py as its main app. It will expose itself on port 5000 on the host, attach to the counter-net network, and mount a volume to /code.

> **Note:** technically speaking, we don’t need the command: python app.py option. This is because the application’s Dockerﬁle already deﬁnes python app.py as the default app for the image. However, we’re showing it here so you know how it works. You can also use Compose to override CMD instructions set in Dockerﬁles.

The deﬁnition of the redis service is simpler:

    - image: redis:alpine This tells Docker to start a standalone container called redis based on the redis:alpine image. This image will be pulled from Docker Hub.
    
    - networks: The redis container will be attached to the counter-net network. 

As both services will be deployed onto the same counter-net network, they will be able to resolve each other by name. This is important as the application is conﬁgured to communicate with the redis service by name.

Now that we understand how the Compose ﬁle works, let’s deploy it!

### Deploying an app with Compose

In this section, we’ll deploy the app deﬁned in the Compose ﬁle from the previous section. To do this, you’ll need the following 4 ﬁles from https://github.com/nigelpoulton/counter-app:

- Dockerﬁle
- app.py
- requirements.txt
- Docker-compose.yml

Clone the Git repo locally.

```shell
$ git clone https://github.com/nigelpoulton/counter-app.git

Cloning into 'counter-app'...
remote: Counting objects: 9, done.
remote: Compressing objects: 100% (8/8), done.
remote: Total 9 (delta 1), reused 5 (delta 0), pack-reused 0
Unpacking objects: 100% (9/9), done.
Checking connectivity... done.
```

Cloning the repo will create a new sub-directory called counter-app. This will contain all of the required ﬁles and will be considered your *build context*. Compose will also use the name of the directory (counter-app) as the project name. We’ll see this later, but Compose will prepend all resource names with counter-app\_.

Change into the counter-app directory and check the ﬁles are present.

```shell
$ cd counter-app
$ ls
app.py docker-compose.yml Dockerfile requirements.txt ...
```

Let’s quickly describe each ﬁle:

    - app.py is the application code (a Python Flask app)

    - docker-compose.yml is the Docker Compose ﬁle that describes how Docker should build and deploy the app

    - Dockerfile describes how to build the image for the web-fe service requirements.txt lists the Python packages required for the app 

Feel free to inspect the contents of each ﬁle.

The app.py ﬁle is obviously the core of the application. But docker-compose.yml is the glue that sticks all the application microservices together.

Let’s use Compose to bring the app up. You must run the all of the following commands from within the counter-app directory that you just cloned from GitHub.

```shell
$ docker-compose up &
```

It’ll take a few seconds for the app to come up, and the output can be quite verbose. You may also have to hit the `Return` key when the deployment completes.

We’ll step through what happened in a second, but ﬁrst let’s talk about the docker-compose command.

`docker-compose up` is the most common way to bring up a Compose app (we’re calling a multi-container app deﬁned in a Compose ﬁle a *Compose app*). It builds or pulls all required images, creates all required networks and volumes, and starts all required containers.

By default, `docker-compose up` expects the name of the Compose ﬁle to docker-compose.yml. If your Compose ﬁle has a diﬀerent name, you need to specify it with the -f ﬂag. The following example will deploy an application from a Compose ﬁle called prod-equus-bass.yml

```shell
$ docker-compose -f prod-equus-bass.yml up
```

It’s also common to use the -d ﬂag to bring the app up in the background. For example:

```shell
docker-compose up -d
```

```shell
docker-compose -f prod-equus-bass.yml up -d
```

Our example brought the app up in the foreground (we didn’t use the -d ﬂag), but we used the & to give us the terminal window back. This forces Compose to output all messages to the terminal window, and we’ll refer back to these messages later.

Now that the app is built and running, we can use normal docker commands to view the images, containers networks, and volumes that Compose created.

```shell
$ docker image ls
```

We can see that three images were either built or pulled as part of the deployment.

The `counter-app\_web-fe:latest` image was created by the build: . instruction in the `docker-compose.yml` ﬁle. This instruction caused Docker to build a new image using the Dockerﬁle in the same directory. It contains the application code for the Python Flask web app, and was built from the python:alpine image. See the contents of the Dockerfile for more information.

```Dockerfile
FROM python:alpine                  << Base image          
ADD . /code                         << Copy app into image 
WORKDIR /code                       << Set working directory
RUN pip install -r requirements.txt << Install requirements
CMD ["python", "app.py"]            << Set the default app
```

I’ve added comments to the end of each line to help explain. They must be removed before deploying the app.

Notice how Compose has named the newly built image as a combination of the project name (counter-app), and the resource name as speciﬁed in the Compose ﬁle (web-fe). All resources deployed by Compose will follow this naming convention.

The redis:alpine image was pulled from Docker Hub by the image: "redis:alpine" instruction in the .Services.redis section of the Compose ﬁle.

The following container listing shows two running containers. The name of each is preﬁxed with the name of the project (name of the build context directory). Also, each one has a numeric suﬃx that indicates the instance number — this is because Compose allows for scaling.

```shell
$ docker container ls
```

The counter-app\_web-fe container is running the application’s web front end. This is running the app.py code and is mapped to port 5000 on all interfaces on the Docker host. We’ll connect to this in just a second. The following network and volume listings show the counter-app\_counter-net network and counter-app\_- counter-vol volume.

```shell
$ docker network ls
```

With the application successfully deployed, you can point a web browser at your Docker host on port 5000 and see the application in all its glory.

Pretty impressive ;-)

Hitting your browser’s refresh button will cause the counter to increment. Have a look at the app (app.py) to see how the counter data is stored in the Redis back-end.

If you brought the application up using the &, you will be able to see the HTTP 200 response codes being logged in the terminal window. These indicate successful requests, and you’ll see one for each time you load the web page.

```log
web-fe\_1 | 172.20.0.1 - - [29/Apr/2020 10:15:27] "GET / HTTP/1.1" 200 -
web-fe\_1 | 172.20.0.1 - - [29/Apr/2020 10:15:28] "GET / HTTP/1.1" 200 -
```

Congratulations. You’ve successfully deployed a multi-container application using Docker Compose!

### Managing an app with Compose

In this section, you’ll see how to start, stop, delete, and get the status of applications being managed by Docker Compose. You’ll also see how the volume we’re using can be used to directly inject updates to the app’s web front-end. As the application is already up, let’s see how to bring it down. To do this, replace the up sub-command with down.

```shell
$ docker-compose down
```

As you initially started the app with the &, it’s running in the foreground. This means you get verbose output to the terminal, giving you an excellent insight into how things work. 

It’s important to note that the counter-vol volume was **not** deleted. This is because volumes are intended to be long-term persistent data stores. As such, their lifecycle is entirely decoupled from the applications they serve. Running a docker volume ls will show that the volume is still present on the system. If you’d written any data to the volume, that data would still exist.

Also, any images that were built or pulled as part of the docker-compose up operation will still be present on the system. This means future deployments of the app will be faster.

Let’s look at a few other docker-compose sub-commands. Use the following command to bring the app up again, but this time in the background.

```shell
$ docker-compose up -d
```

See how the app started much faster this time — the` counter-vol volume` already exists, and all images already exist on the Docker host.

Show the current state of the app with the `docker-compose ps` command.

```shell
$ docker-compose ps
```

You can see both containers, the commands they are running, their current state, and the network ports they are listening on.

Use `docker-compose top` to list the processes running inside of each service (container).

```shell
$ docker-compose top
```

The PID numbers returned are the PID numbers as seen from the Docker host (not from within the containers). Use the `docker-compose stop` command to stop the app without deleting its resources. Then show the status of the app with `docker-compose ps`.

```shell
$ docker-compose stop
```

```shell
$ docker-compose ps
```

As you can see, stopping a Compose app does not remove the application deﬁnition from the system. It just stops the app’s containers. You can verify this with the docker container `ls -a` command. You can delete a stopped Compose app with `docker-compose rm`. This will delete the containers and networks the app is using, but it will not delete volumes or images. Nor will it delete the application source code in your project’s build context directory (`app.py`, `Dockerfile`, `requirements.txt`, and `docker-compose`.yml).

Restart the app with the `docker-compose restart` command.

```shell
$ docker-compose restart
```

```shell
$ docker-compose ps
```

Use the `docker-compose down` command to **stop and delete** the app with a single command.

```shell
$ docker-compose down
```

The app is now deleted. Only its images, volumes, and source code remain.

Let’s deploy the app one last time and see a lile more about how the volume works.

```shell
$ docker-compose up -d
```

If you look in the Compose ﬁle, you’ll see that it deﬁnes a volume called counter-vol and mounts it in to the web-fe container at /code.

```yaml
services:
    web-fe:
    <Snip>
        volumes:
            - type: volume
              source: counter-vol
              target: /code
<Snip>
volumes:
    counter-vol:
```

The ﬁrst time you deployed the app, Compose checked to see if a volume called counter-vol already existed. It did not, so Compose created it. You can see it with the docker volume ls command, and you can get more detailed information with docker volume inspect counter-app\_counter-vol.

```shell
$ docker volume ls
```

It’s also worth knowing that Compose builds networks and volumes **before** deploying services. This makes sense, as networks and volumes are lower-level infrastructure objects that are consumed by services (containers). The following snippet shows Compose creating the network and volume as its ﬁrst two tasks (even before building and pulling images).

```shell
$ docker-compose up -d
```

If we take another look at the service deﬁnition for web-fe, we’ll see that it’s mounting the counter-app volume into the service’s container at /code. We can also see from the Dockerﬁle that /code is where the app is installed and executed from. Net result, the app code resides on a Docker volume. See Figure.

<img src=".\images\DockerFileComposeFile.png" style="width:75%; height: 75%;">

This all means we can make changes to ﬁles in the volume, from the outside of the container, and have them reﬂected immediately in the app. Let’s see how that works. The next few steps will walk you through the following process. We’ll update the contents of app.py in the project’s working directory on the Docker host. We’ll copy the updated app.py to the volume on the Docker host. We’ll refresh the app’s web page to see the updated text. This will work because whatever you write to the volume on the Docker host will immediately appear in the volume mounted in the container. 

> **Note:** The following will not work if you are using Docker Desktop on a Mac or Windows 10 PC.

This is because Docker Desktop runs Docker inside of a lightweight VM and volumes exist inside the VM. Use your favourite text editor to edit the app.py ﬁle in the projects working directory. We’ll use vim in the example.

```shell
$ vim ~/counter-app/app.py
```

Change text between the double quote marks (“”) on line 22. The line starts with return "What's up...". Enter any text you like, as long as it’s within the double-quote marks, and save your changes. Now that you’ve updated the app, you need to copy it into the volume on the Docker host. each Docker volume is exposed at a location within the Docker host’s ﬁlesystem, as well as a mount point in one or more containers. Use the following docker volume inspect command to ﬁnd where the volume is exposed on the Docker host.

```shell
$ docker volume inspect counter-app\_counter-vol | grep Mount

"Mountpoint": "/var/lib/docker/volumes/counter-app\_counter-vol/\_data",
```

Copy the updated app ﬁle to the volume’s mount point on your Docker host (remember that this will not work on Docker Desktop). As soon as you perform the copy operation, the updated ﬁle will appear in the /code directory in the web-fe container. The operation will overwrite the existing /code/app.py ﬁle in the container.

```shell
$ cp ~/counter-app/app.py \

/var/lib/docker/volumes/counter-app\_counter-vol/\_data/app.py
```

The updated app ﬁle is now on the container. Connect to the app to see your change. You can do this by pointing your web browser to the IP of your Docker host on port 5000. Figure shows the updated app.

Obviously you wouldn’t do an update operation like this in production, but it’s a real time-saver in development. Congratulations. You’ve deployed and managed a simple multi-container app using Docker Compose. Before reminding ourselves of the major docker-compose commands, it’s important to understand that this was a very simple example. Docker Compose is capable of deploying and managing far more complex applications.

## Docker Swarm

Docker Swarm is two main things:

    1. An enterprise-grade secure cluster of Docker hosts

    2. An engine for orchestrating microservices apps

On the clustering front, Swarm groups one or more Docker nodes and lets you manage them as a cluster. Out-of-the-box, you get an encrypted distributed cluster store, encrypted networks, mutual TLS, secure cluster join tokens, and a PKI that makes managing and rotating certiﬁcates a breeze. You can even non-disruptively add and remove nodes. It’s a beautiful thing.

On the orchestration front, Swarm exposes a rich API that allows you to deploy and manage complex microservices apps with ease. You can deﬁne your apps in declarative manifest ﬁles and deploy them to the Swarm with native Docker commands. You can even perform rolling updates, rollbacks, and scaling operations. Again, all with simple commands.

Docker Swarm competes directly with Kubernetes — they both orchestrate containerized applications. While it’s true that Kubernetes has more momentum and a more active community and ecosystem, Docker Swarm is an excellent technology and a lot easier to conﬁgure and deploy. It’s an excellent technology for small-to-medium businesses and application deployments.

We’ll split the deep dive part of this chapter as follows:

- Swarm primer
- Build a secure swarm cluster
- Deploy some swarm services
- Troubleshooting

### Swarm primer

On the clustering front, a *swarm* consists of one or more Docker *nodes*. These can be physical servers, VMs, Raspberry Pi’s, or cloud instances. The only requirement is that all nodes have Docker installed and can communicate over reliable networks.

Nodes are conﬁgured as *managers* or *workers*. *Managers* look after the control plane of the cluster, meaning things like the state of the cluster and dispatching tasks to *workers*. *Workers* accept tasks from *managers* and execute them.

The conﬁguration and state of the *swarm* is held in a distributed *etcd* database located on all managers. It’s kept in memory and is extremely up-to-date. But the best thing about it is that it requires zero conﬁguration — it’s installed as part of the swarm and just takes care of itself.

Something that’s game changing on the clustering front is the approach to security. TLS is so tightly integrated that it’s impossible to build a swarm without it. In today’s security conscious world, things like this deserve all the plaudits they get. *Swarm* uses TLS to encrypt communications, authenticate nodes, and authorize roles.

Automatic key rotation is also thrown in as the icing on the cake. And the best part of it all happens so smoothly that you don’t even know it’s there.

On the application orchestration front, the atomic unit of scheduling on a swarm is the *service*. This is a new object in the API, introduced along with swarm, and is a higher level construct that wraps some advanced features around containers. These include scaling, rolling updates, and simple rollbacks. It’s useful to think of a *service* as an enhanced container.

A high-level view of a swarm is shown in Figure.

<img src=".\images\Swarm.png" style="width:75%; height: 75%;">

That's enough of a primer. Let’s get our hands dirty with some examples.

### Build a secure Swarm cluster

In this section, we’ll build a secure swarm cluster with three *manager nodes* and three *worker nodes*. You can use a diﬀerent lab with diﬀerent numbers of *managers* and *workers*, and with diﬀerent names and IPs, but the examples that follow will use the values in Figure.

<img src=".\images\SwarmMngWrk.png" style="width:75%; height: 75%;">

The nodes can be virtual machines, physical servers, cloud instances, or Raspberry Pi systems. The only requirements are that they have Docker installed and can communicate over a reliable network. It’s also beneﬁcial if name resolution is conﬁgured — it makes it easier to identify nodes in command outputs and helps when troubleshooting.

On the networking front, you need the following ports open on routers and ﬁrewalls between nodes:

- 2377/tcp: for secure client-to-swarm communication
- 7946/tcp and udp: for control plane gossip
- 4789/udp: for VXLAN-based overlay networks

Docker Desktop for Mac and Windows only supports a single Docker node. You can initialize a single-node swarm and follow along with most of the examples. Alternatively, you can try Play with Docker at https://labs.play-with-Docker.com.

Once you’ve satisﬁed the pre-requisites, you can go ahead and build a swarm. The process of building a swarm is called *initializing a swarm*, and the high-level process is this: Initialize the ﬁrst manager node > Join additional manager nodes > Join worker nodes > Done.

### Initializing a new swarm

Docker nodes that are not part of a swarm are said to be in *single-engine mode*. Once they’re added to a swarm they’re automatically switched into *swarm mode*. Running docker swarm init on a Docker host in *single-engine mode* will switch that node into *swarm mode*, create a new *swarm*, and make the node the ﬁrst *manager* of the swarm.

Additional nodes can then be joined to the swarm as workers and managers. Joining a Docker host to an existing swarm switches them into *swarm mode* as part of the operation.

The following steps will put **mgr1** into *swarm mode* and initialize a new swarm. It will then join **wrk1**, **wrk2**, and **wrk3** as worker nodes — automatically puing them into *swarm mode* as part of the process. Finally, it will add **mgr2** and **mgr3** as additional managers and switch them into *swarm mode*. At the end of the procedure all 6 nodes will be in *swarm mode* and operating as part of the same swarm.

This example will use the IP addresses and DNS names of the nodes shown in Figure. Yours may be diﬀerent.

1. Log on to **mgr1** and initialize a new swarm (don’t forget to use backticks instead of backslashes if you’re following along with Windows in a PowerShell terminal).

```shell
$ docker swarm init \
--advertise-addr 10.0.0.1:2377 \
--listen-addr 10.0.0.1:2377

Swarm initialized: current node (d21lyz...c79qzkx) is now a manager.
```

The command can be broken down as follows:

    - `docker swarm init`: This tells Docker to initialize a new swarm and make this node the ﬁrst manager. It also enables swarm mode on the node.

    - `--advertise-addr`: As the name suggests, this is the swarm API endpoint that will be advertised to other nodes in the swarm. It will usually be one of the node’s IP addresses, but can be an external load-balancer address. It’s an optional ﬂag unless you want to specify a load-balancer or speciﬁc IP address on a node with multiple interfaces.

    - `--listen-addr`: This is the IP address that the node will accept swarm traﬃc on. If not explicitly set, it defaults to the same value as `--advertise-addr`. If `--advertise-addr` is a load-balancer, you must use `--listen-addr` to specify a local IP or interface for swarm traﬃc.

I recommend you be speciﬁc and always use both ﬂags.

The default port that swarm mode operates on is **2377**. This is customizable, but it’s convention to use `2377/tcp` for secured (HTTPS) client-to-swarm connections.

2. List the nodes in the swarm.

```shell
$ docker node ls
```

Notice that **mgr1** is currently the only node in the swarm, and is listed as the *Leader*. We’ll come back to this in a second.

3. From **mgr1** run the docker swarm join-token command to extract the commands and tokens required to add new workers and managers to the swarm.

```shell
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
```

Notice that the commands to join a worker and a manager are identical apart from the join tokens (SWMTKN...). This means that whether a node joins as a worker or a manager depends entirely on which token you use when joining it. **You should ensure that your join tokens are kept secure, as they’re the only thing required to join a node to a swarm!**

4. Log on to **wrk1** and join it to the swarm using the docker swarm join command with the worker join token.

```shell
$ docker swarm join \
    --token SWMTKN-1-0uahebax...c87tu8dx2c \
    10.0.0.1:2377 \
    --advertise-addr 10.0.0.4:2377 \
    --listen-addr 10.0.0.4:2377

This node joined a swarm as a worker.
```

The `--advertise-addr`, and `--listen-addr` ﬂags optional. I’ve added them as I consider it best practice to be as speciﬁc as possible when it comes to network conﬁguration.

5. Repeat the previous step on **wrk2** and **wrk3** so that they join the swarm as workers. If you’re specifying the `--advertise-addr` and `--listen-addr` ﬂags, make sure you use **wrk2** and **wrk3’s** respective IP addresses.

6. Log on to **mgr2** and join it to the swarm as a manager using the docker swarm join command with the manager join token.

```shell
$ docker swarm join \
    --token SWMTKN-1-0uahebax...ue4hv6ps3p \
    10.0.0.1:2377 \
    --advertise-addr 10.0.0.2:2377 \
    --listen-addr 10.0.0.2:2377

This node joined a swarm as a manager.
```

7. Repeat the previous step on **mgr3**, remembering to use **mgr3’s** IP address for the `advertise-addr` and `--listen-addr` ﬂags.

8. List the nodes in the swarm by running docker node ls from any of the manager nodes in the swarm.

```shell
$ docker node ls
```

Congratulations. You’ve just created a 6-node swarm with 3 managers and 3 workers. As part of the process, the Docker Engine on each node was automatically put into *swarm mode* and the *swarm* was automatically secured with TLS.

If you look in the MANAGER STATUS column you’ll see the three manager nodes are showing as either “Reachable” or “Leader”. We’ll learn more about leaders shortly. Nodes with nothing in the MANAGER STATUS column are *workers*. Also note the asterisk (\*) after the ID on the line showing **mgr2**. This tells you which node you are logged on to and executing commands from.

> **Note:** It’s a pain to specify the --advertise-addr and --listen-addr ﬂags every time you join a node to the swarm. However, it can be a much bigger pain if you get the network conﬁguration of your swarm wrong. Also, manually adding nodes to a swarm is unlikely to be a daily task, so it’s worth the extra up-front eﬀort to use the ﬂags. It’s your choise though. In lab environments or nodes with only a single IP you probably don’t need to use them. 

Now that you have a *swarm* up and running, let’s take a look at manager high availability (HA).

### Swarm manager high availability (HA)

So far, we’ve added three manager nodes to a swarm. Why three? And how do they work together?

Swarm *managers* have native support for high availability (HA). This means one or more can fail, and the survivors will keep the swarm running.

Technically speaking, swarm implements a form of active-passive multi-manager HA. This means that although you have multiple *managers*, only one of them is *active* at any given moment. This active manager is called the “*leader*”, and the leader is the only one that will ever issue live commands against the *swarm*. So, it’s only ever the leader that changes the conﬁg, or issues tasks to workers. If a follower manager (passive) receives commands for the swarm, it proxies them across to the leader.

This process is shown in Figure. Step 1 is the command coming in to a *manager* from a remote Docker client. Step 2 is the non-leader manager receiving the command and proxying it to the leader. Step 3 is the leader executing the command on the swarm.

<img src=".\images\SwarmHA.png" style="width:75%; height: 75%;">

**If you look closely at Figure, you’ll notice that managers are either *leaders* or *followers*. This is Raft terminology, because swarm uses an implementation of the Raft consensus algorithm to maintain a consistent cluster state across multiple highly available managers.**

On the topic of HA, the following two best practices apply:
    1. Deploy an odd number of managers.
    2. Don’t deploy too many managers (3 or 5 is recommended)

**Having an odd number of *managers* reduces the chances of split-brain conditions.** For example, if you had 4 managers and the network partitioned, you could be left with two managers on each side of the partition. This is known as a split brain — each side knows there used to be 4 but can now only see 2. But crucially, neither side has any way of knowing if the other two are still alive and whether it holds a majority (quorum). A swarm cluster continues to operate during split-brain conditions, but you are no longer able to alter the conﬁguration or add and manage application workloads.

However, if you have 3 or 5 managers and the same network partition occurs, it is impossible to have an equal number of managers on both sides of the partition. This means that one side achieves quorum and full cluster management services remain available. The example on the right side of Figure shows a partitioned cluster where the left side of the split knows it has a majority of managers.

<img src=".\images\SwarmHA2.png" style="width:75%; height: 75%;">

As with all consensus algorithms, more participants means more time required to achieve consensus. It’s like deciding where to eat — it’s always quicker and easier for 3 people to make a quick decision than it is for 33! With this in mind, it’s a best practice to have either 3 or 5 managers for HA. 7 might work, but it’s generally accepted that 3 or 5 is optimal. You deﬁnitely don’t want more than 7, as the time taken to achieve consensus will be longer.

A ﬁnal word of caution regarding manager HA. While it’s obviously a good practice to spread your managers across availability zones within your network, you need to make sure the networks connecting them are reliable, as network partitions can be a diﬃcult to troubleshoot and resolve. This means, at the time of writing, the nirvana of hosting your active production applications and infrastructure across multiple cloud providers such as AWS and Azure is a bit of a daydream. Take the time and eﬀort to ensure your managers and workers are connected via reliable high-speed networks.

### Built-in Swarm security

Swarm clusters have a ton of built-in security that’s conﬁgured out-of-the-box with sensible defaults — CA settings, join tokens, mutual TLS, encrypted cluster store, encrypted networks, cryptographic node ID’s and more.

### Locking a Swarm

Despite all of this built-in native security, restarting an older manager or restoring an old backup has the potential to compromise the cluster. Old managers re-joining a swarm automatically decrypt and gain access to the Raft log time-series database — this can pose security concerns. Restoring old backups can also wipe the current swarm conﬁguration. 

To prevent situations like these, Docker allows you to lock a swarm with the Autolock feature. This forces restarted managers to present the cluster unlock key before being admited back into the cluster.

It’s possible to apply a lock directly to a new swarm by passing the `--autolock` ﬂag to the `docker swarm init` command. However, we’ve already built a swarm, so we’ll lock our existing swarm with the `docker swarm update` command.

Run the following command from a swarm manager.

```shell
$ docker swarm update --autolock=true

Swarm updated.
To unlock a swarm manager after it restarts, run the `docker swarm unlock` command and provide the following key:
    
    SWMKEY-1-5+ICW2kRxPxZrVyBDWzBkzZdSd0Yc7Cl2o4Uuf9NPU4

Please remember to store this key in a password manager, since without it you will not be able to restart the manager.
```

Be sure to keep the unlock key in a secure place. You can always check your current swarm unlock key with the `docker swarm unlock-key` command.

Restart one of your manager nodes to see if it automatically re-joins the cluster. You may need to prepend the command with `sudo`.

```shell
$ service docker restart
```

Try and list the nodes in the swarm.

```shell
$ docker node ls

Error response from daemon: Swarm is encrypted and needs to be unlocked before it can be used.
```

Although the Docker service has restarted on the manager, it has not been allowed to re-join the swarm. You can prove this even further by running the docker node ls command on another manager node. The restarted manager will show as down and unreachable.

Use the `docker swarm unlock` command to unlock the swarm for the restarted manager. You’ll need to run this command on the restarted manager, and you’ll need to provide the unlock key.

```
$ docker swarm unlock

Please enter unlock key: <enter your key>
```

The node will be allowed to re-join the swarm and will show as ready and reachable if you run another `docker node ls`.

locking your swarm and protecting the unlock key is recommended for production environments.

Now that you’ve got our *swarm* built and understand the infrastructure concepts of *leaders* and *manager HA*, let’s move on to the application aspect of *services*.

### Swarm services

Like we said in the swarm primer, *services* are a new construct introduced with Docker 1.12, and they only apply to *swarm mode*.

Services let us specify most of the familiar container options, such as *name, port mappings, attaching to networks,* and *images*. But they add important cloud-native features, including *desired state* and automatic reconciliation. For example, swarm services allow us to declaratively deﬁne a desired state for an application that we can apply to the swarm and let the swarm take care of deploying it and managing it.

Let’s look at a quick example. Assume you have an app with a web front-end. You have an image for the web server, and testing has shown that you need 5 instances to handle normal daily traﬃc. You translate this requirement into a single *service* declaring the image to use, and that the service should always have 5 running replicas. You issue that to the swarm as your desired state, and the swarm takes care of ensuring there are always 5 instances of the web server running.

We’ll see some of the other things that can be declared as part of a service in a minute, but before we do that, let’s see one way to create what we just described.

You can create services in one of two ways:
    1. Imperatively on the command line with docker service create
    2. Declaratively with a stack ﬁle

We’ll look at stack ﬁles in a later chapter. For now we’ll focus on the imperative method.

> **Note:** The command to create a new service is the same on Windows. However, the image used in this example is a Linux image and will not work on Windows. You can substitute the image for a Windows web server image and the command will work. Remember, if you are typing Windows commands from a PowerShell terminal you will need to use the backtick (‘) to indicate continuation on the next line.

```shell
$ docker service create --name web-fe \
    -p 8080:8080 \
    --replicas 5 \
    nigelpoulton/pluralsight-docker-ci

z7ovearqmruwk0u2vc5o7ql0p
```

Notice that many of the familiar docker container run arguments are the same. In the example, we speciﬁed `--name` and `-p` which work the same for standalone containers as well as services.

Let’s review the command and output.

We used docker service create to tell Docker we are declaring a new service, and we used the `--name` ﬂag to name it **web-fe**. We told Docker to map port 8080 on every node in the swarm to 8080 inside of each service replica. Next, we used the `--replicas` ﬂag to tell Docker there should always be 5 replicas of this service. Finally, we told Docker which image to use for the replicas — it’s important to understand that all service replicas use the same image and conﬁg!

After we hit Return, the command was sent to a manager node, and the manager acting as leader instantiated 5 replicas across the *swarm* — remember that swarm managers also act as workers. each worker or manager that received a work task pulled the image and started a container listening on port 8080. The swarm leader also ensured a copy of the service’s *desired state* was stored on the cluster and replicated to every manager. But this isn’t the end. All *services* are constantly monitored by the swarm — the swarm runs a background *reconciliation loop* that constantly compares the *observed state* of the service with the *desired state*. If the two states match, the world is a happy place and no further action is needed. If they don’t match, swarm takes actions to bring *observed state* into line with *desired state*.

As an example, if a *worker* hosting one of the 5 **web-fe** replicas fails, the *observed state* of the **web-fe** service will drop from 5 replicas to 4. This will no longer match the *desired state* of 5, so the swarm will start a new **web-fe** replica to bring the *observed state* back in line with *desired state*. This behavior is a key tenet of cloud-native applications and allows the service to self-heal in the event of node failures and the likes.

### Viewing and inspecting services

You can use the `docker service ls` command to see a list of all services running on a swarm.

```shell
$ docker service ls
```

The output shows a single running service as well as some basic information about state. Among other things, you can see the name of the service and that 5 out of the 5 desired replicas are in the running state. If you run this command soon after deploying the service it might not show all tasks/replicas as running. This is often due to the time it takes to pull the image on each node.

You can use the docker service ps command to see a list of service replicas and the state of each.

```shell
$ docker service ps web-fe
```

The format of the command is docker service ps <service-name or service-id>. The output displays each replica (container) on its own line, shows which node in the swarm it’s executing on, and shows desired state and the current observed state.

For detailed information about a service, use the docker service inspect command.

```shell
$ docker service inspect --pretty web-fe
```

The example above uses the --pretty ﬂag to limit the output to the most interesting items printed in an easy-to-read format. Leaving oﬀ the --pretty ﬂag will give a more verbose output. I highly recommend you read through the output of docker inspect commands as they’re a great source of information and a great way to learn what’s going on under the hood.

We’ll come back to some of these outputs later.

### Replicated vs global services

The default replication mode of a service is replicated. This deploys a desired number of replicas and distributes them as evenly as possible across the cluster.

The other mode is global, which runs a single replica on every node in the swarm.

To deploy a *global service* you need to pass the --mode global ﬂag to the docker service create command.

### Scaling a service

Another powerful feature of *services* is the ability to easily scale them up and down.

Let’s assume business is booming and we’re seeing double the amount of traﬃc Hitting the web front-end. Fortunately, scaling the **web-fe** service is as simple as running the `docker service scale` command.

```shell
$ docker service scale web-fe=10
```

This command will scale the number of service replicas from 5 to 10. In the background it’s updating the service’s *desired state* from 5 to 10. Run another `docker service ls` command to verify the operation was successful.

```shell
$ docker service ls
```

Running a `docker service ps` command will show that the service replicas are balanced across all nodes in the swarm evenly.

```shell
$ docker service ps web-fe
```

Behind the scenes, swarm runs a scheduling algorithm called “spread” that attempts to balance replicas as evenly as possible across the nodes in the swarm. At the time of writing, this amounts to running an equal number of replicas on each node without taking into consideration things like CPU load etc.

Run another `docker service scale` command to bring the number back down from 10 to 5.

```shell
$ docker service scale web-fe=5
```

Now that you know how to scale a service, let’s see how to remove one.

### Removing a service

Removing a service is simple — may be too simple.

The following `docker service rm` command will delete the service deployed earlier.

```shell
$ docker service rm web-fe
web-fe
```

Conﬁrm it’s gone with the `docker service ls` command.

```shell
$ docker service ls
```

Be careful using the `docker service rm` command as it deletes all service replicas without asking for conﬁrmation.

Now that the service is deleted from the system, let’s look at how to push rolling updates to one.

### Rolling updates

Pushing updates to deployed applications is a fact of life. And for the longest time it was really painful. I’ve lost more than enough weekends to major application updates, and I’ve no intention of doing it again.

Thanks to Docker *services*, pushing updates to well-designed microservices apps is easy.

To see this, we’re going to deploy a new service. But before we do that, we’re going to create a new overlay network for the service. This isn’t necessary, but I want you to see how it is done and how to attach the service to it.

```shell
$ docker network create -d overlay uber-net
43wfp6pzea470et4d57udn9ws
```

This creates a new overlay network called “uber-net” that we’ll use for the service we’re about to create. An overlay network creates a new layer 2 network that we can place containers on, and all containers on it will be able to communicate. This works even if all of the swarm nodes are on diﬀerent underlying networks. Basically, the overlay network creates a new layer 2 container network on top of potentially multiple diﬀerent underlying networks.

Figure shows four swarm nodes on two underlay networks connected by a layer 3 router. The overlay network spans all 4 swarm nodes creating a single ﬂat layer 2 network for containers to use.

<img src=".\images\SwarmOverlay.png" style="width:75%; height: 75%;">

Run a docker network ls to verify that the network created properly and is visible on the Docker host.

```shell
$ docker network ls
```

The uber-net network was successfully created with the swarm scope and is *currently* only visible on manager nodes in the swarm. It will be dynamically extended to worker nodes when they run workloads conﬁgured on the network.

Let’s create a new service and attach it to the network.

```shell
$ docker service create --name uber-svc \
    --network uber-net \
    -p 80:80 --replicas 12 \
    nigelpoulton/tu-demo:v1
```

Let’s see what we just declared with that docker service create command.

The ﬁrst thing we did was name the service and then use the --network ﬂag to tell it to place all replicas on the new uber-net network. We then exposed port 80 across the entire swarm and mapped it to port 80 inside of each of the 12 replicas we asked it to run. Finally, we told it to base all replicas on the nigelpoulton/tu-demo:v1 image.

Run a docker service ls and a docker service ps command to verify the state of the new service.

```shell
$ docker service ls
```

Passing the service -p 80:80 ﬂag will ensure that a **swarm-wide** mapping is created that maps all traﬃc, coming in to any node in the swarm on port 80, through to port 80 inside of any service replica.

This mode of publishing a port on every node in the swarm — even nodes not running service replicas — is called *ingress mode* and is the default. The alternative mode is *host mode* which only publishes the service on swarm nodes running replicas. Publishing a service in *host mode* requires the long-form syntax and looks like the following:

```shell
$ docker service create --name uber-svc \
    --network uber-net \
    --publish published=80,target=80,mode=host \
    --replicas 12 \
    nigelpoulton/tu-demo:v1
```

Open a web browser and point it to the IP address of any of the nodes in the swarm on port 80 to see the service running.

<img src=".\images\SwarmApp.png" style="width:75%; height: 75%;">

As you can see, it’s a simple voting application that will register votes for either “football” or “soccer”. Feel free to point your web browser to other nodes in the swarm. You’ll be able to reach the web service from any node because the -p 80:80 ﬂag creates an *ingress mode* mapping on every swarm node. This is true even on nodes that are not running a replica for the service — **every node gets a mapping and can therefore redirect your request to a node that is running the service**.

Let’s now assume that this particular vote has come to an end and your company wants to run a new poll. A new container image has been created for the new poll and has been added to the same Docker Hub repository, but this one is tagged as v2 instead of v1.

Let’s also assume that you’ve been tasked with pushing the updated image to the swarm in a staged manner —2 replicas at a time with a 20 second delay between each. You can use the following docker service update command to accomplish this.

```shell
$ docker service update \
    --image nigelpoulton/tu-demo:v2 \
    --update-parallelism 2 \
    --update-delay 20s uber-svc

overall progress: 4 out of 12 tasks
```

Let’s review the command. docker service update lets us make updates to running services by updating the service’s desired state. This example speciﬁes a new version of the image, tagged as v2 instead of v1. It also speciﬁed the --update-parallelism and --update-delay ﬂags to make sure that the new image was pushed to 2 replicas at a time with a 20 second cool-oﬀ period in between each set of two. Finally, it instructs the swarm to make the changes to the uber-svc service.

If you run a docker service ps uber-svc while the update is in progress, some of the replicas will be at v2 while some will still be at v1. If you give the operation enough time to complete (4 minutes), all replicas will eventually reach the new desired state of using the v2 image.

```shell
$ docker service ps uber-svc
```

You can witness the update happening in real-time by opening a web browser to any node in the swarm and Hitting refresh several times. Some of the requests will be serviced by replicas running the old version and some will be serviced by replicas running the new version. After enough time, all requests will be serviced by replicas running the updated version of the service.

Congratulations. You’ve just pushed a rolling update to a live containerized application. Remember, Docker Stacks take all of this to the next level in Chapter 14.

If you run a docker inspect --pretty command against the service, you’ll see the update parallelism and update delay settings are now part of the service deﬁnition. This means future updates will automatically use these settings unless you override them as part of the docker service update command.

```shell
$ docker service inspect --pretty uber-svc
```

You should also note a couple of things about the service’s network conﬁg. All nodes in the swarm that are running a replica for the service will have the uber-net overlay network that we created earlier. We can verify this by running docker network ls on any node running a replica.

You should also note the Networks portion of the docker inspect output. This shows the uber-net network as well as the swarm-wide `80:80` port mapping.

### Troubleshooting

Swarm Service logs can be viewed with the docker service logs command. However, not all logging drivers support the command.

By default, Docker nodes conﬁgure services to use the json-file log driver, but other drivers exist, including:

- journald (only works on Linux hosts running systemd)
- syslog
- splunk
- gelf
 
json-file and journald are the easiest to conﬁgure, and both work with the docker service logs command. The format of the command is docker service logs <service-name>.

If you’re using 3rd-party logging drivers you should view those logs using the logging platform’s native tools. The following snippet from a daemon.json conﬁguration ﬁle shows a Docker host conﬁgured to use syslog.

```yaml
{
    "log-driver": "syslog"
}
```

You can force individual services to use a diﬀerent driver by passing the --log-driver and --log-opts ﬂags to the docker service create command. These will override anything set in daemon.json.

Service logs work on the premise that your application is running as PID 1 in its container and sending logs to STDOUT and errors to STDERR. The logging driver forwards these “logs” to the locations conﬁgured via the logging driver.

The following docker service logs command shows the logs for all replicas in the svc1 service that experienced a couple of failures starting a replica.

```shell
$ docker service logs svc1
```

The output is trimmed to ﬁt the page, but you can see that logs from all three service replicas are shown (the two that failed and the one that’s running). Each line starts with the name of the replica, which includes the service name, replica number, replica ID, and name of host that it’s scheduled on. Following that is the log output.

It looks like the ﬁrst two replicas failed because they were trying to connect to another service that was still starting (a sort of race condition when dependent services are starting).

You can follow the logs (--follow), tail them (--tail), and get extra details (--details).

### Backing up Swarm

Backing up a swarm will backup the control plane objects required to recover the swarm in the event of a catastrophic failure of corruption. Recovering a swarm from a backup is an extremely rare scenario. However, business critical environments should always be prepared for worst-case scenarios.

You might be asking why backups are necessary if the control plane is already replicated and highly-available (HA). To answer that question, consider the scenario where a malicious actor deletes all of the Secrets on a swarm. HA cannot help in this scenario as the Secrets will be deleted from the cluster store that is automatically replicated to all manager nodes. In this scenario the highly-available replicated cluster store works against you — quickly propagating the delete operation. In this scenario you can either recreate the deleted objects from copies kept in a source code repo, or you can attempt to recover your swarm from a recent backup.

Managing your swarm and applications declaratively is a great way to prevent the need to recover from a backup. For example, storing conﬁguration objects outside of the swarm in a source code repository will enable you to redeploy things like networks, services, secrets and other objects. However, managing your environment declaratively and strictly using source control repos requires discipline.

Anyway, let’s see how to **backup a swarm**.

> Swarm conﬁguration and state is stored in **/var/lib/docker/swarm** on every manager node. The conﬁguration includes; Raft log keys, overlay networks, Secrets, Conﬁgs, Services, and more. A swarm backup is a copy of all the ﬁles in this directory.

As the contents of this directory are replicated to all managers, you can, and should, perform backups from multiple managers. However, as you have to stop the Docker daemon on the node you are backing up, it’s a good idea to perform the backup from non-leader managers. This is because stopping Docker on the leader will initiate a leader election. You should also perform the backup at a quiet time for the business, as stopping a manager can increase the risk of the swarm losing quorum if another manager fails during the backup.

The procedure we’re about to follow is designed for demonstration purposes and you’ll need to tweak it for your production environment. It also creates a couple of swarm objects so that a later step can prove the restore operation worked.

> **Warning**: The following operation carries risks. You should also ensure you perform test backup and restore operations regularly and test the outcomes.

The following commands will create the following two objects so you can prove the restore operation:

- An overlay network called “Unimatrix-01”
- A Secret called “missing drones” containing the text “Seven of Nine”

```shell
$ docker network create -d overlay Unimatrix-01

w9l904ff73e7stly0gnztsud7
```

```shell
$ printf "Seven of Nine" | docker secret create missing\_drones -

i8oj3b2lid27t5202uycw37lg
```

Let’s perform the swarm backup.

1. Stop Docker on a non-leader swarm manager.
    If you have any containers or service tasks running on the node, this action may stop them.

```shell
$ service docker stop
```

2. Backup the Swarm conﬁg.

    This example uses the Linux tar utility to perform the ﬁle copy that will be the backup. Feel free to use a diﬀerent tool.

```shell
$ tar -czvf swarm.bkp /var/lib/docker/swarm/
```

3. Verify the backup ﬁle exists.

```shell
$ ls -l
```

In the real world you should store and rotate this backup in accordance with any corporate backup policies.

At this point, the swarm is backed up and you can restart Docker on the node.

4. Restart Docker.

```shell
$ service docker restart
```

Now that you have a backup, let’s perform a test restore. The steps in this procedure demonstrate the operation. Performing a restore in the real world may be slightly diﬀerent, but the overall process will be similar.

> **Note**: You do not have to perform a restore operation if your swarm is still running and you only wish to add a new manager node. In this situation just add a new manager. A swarm restore is only for situations where the swarm is corrupted or otherwise lost and you cannot recover services from copies of conﬁg ﬁles stored in a source code repo.

We’ll use the swarm.bkp ﬁle from earlier to restore the swarm. **All swarm nodes must have their Docker daemon stopped and the contents of their /var/lib/docker/swarm directories deleted.**

The following must also be true for a recovery operation to work:

    1. You can only restore to a node running the same version of Docker the backup was performed on.

    2. You can only restore to a node with the same IP address as the node the backup was performed on.

Perform the following tasks from the swarm manager node that you wish to recover. 

> Remember that Docker must be stopped and the contents of **/var/lib/docker/swarm** must be deleted.

1. Restore the Swarm conﬁguration from backup.

    In this example, we’ll restore from a zipped tar ﬁle called swarm.bkp. Restoring to the root directory is required with this command as it will include the full path to the original ﬁles as part of the extract operation. This may be diﬀerent in your environment.
    
    ```shell
    $ tar -zxvf swarm.bkp -C /
    ```

2. Start Docker. The method for starting Docker can vary between environments.

    ```shell
    $ service docker start
    ```

3. Initialize a new Swarm cluster.

    Remember, you are not recovering a manager and adding it back to a working cluster. This operation is to recover a failed swarm that has no surviving managers. The --force-new-cluster ﬂag tells Docker to create a new cluster using the conﬁguration stored in /var/lib/docker/swarm/ that you recovered in step 1.

    ```shell
    $ docker swarm init --force-new-cluster

    Swarm initialized: current node (jhsg...3l9h) is now a manager.
    ```

4. Check that the network and service were recovered as part of the operation.

    ```shell
    $ docker network ls
    ```

    ```shell
    $ docker secret ls
    ```

5. Add new manager and worker nodes and take fresh backups.

Remember, test this procedure regularly and thoroughly. You do not want it to fail when you need it most!
