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
