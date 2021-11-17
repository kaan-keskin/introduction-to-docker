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
