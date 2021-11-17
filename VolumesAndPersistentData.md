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

## Volumes and Persistent Data

Stateful applications that persist data are becoming more and more important in the world of cloud-native and microservices applications. Docker is an important infrastructure technology in this space.

There are two main categories of data — persistent and non-persistent. 

**Persistent** is the data you need to *keep*. Things like; customer records, ﬁnancial data, research results, audit logs, and even some types of application *log* data. 

**Non-persistent** is the data you don’t need to keep. Both are important, and Docker has solutions for both. To deal with non-persistent data, every Docker container gets its own non-persistent storage. This is automatically created for every container and is tightly coupled to the lifecycle of the container. As a result, deleting the container will delete the storage and any data on it. 

To deal with persistent data, a container needs to store it in a **volume**. **Volumes are separate objects that have their lifecycles decoupled from containers.** This means you can create and manage volumes independently, and they’re not tied to the lifecycle of any container. 

Net result, you can delete a container that’s using a volume, and the volume won’t be deleted. Containers deal with persistent and non-persistent data, and you may ﬁnd it hard to see many differences with virtual machines.

### UnionFS with Copy-on-Write

UnionFS is used by Docker to overlay a base container image with storage layers, such as ephemeral storage layer, custom storage layer, and config layer at the time a new container is created. The ephemeral storage is reserved for the container’s Input/Output (I/O) operations and it is not recommended to be used for persistent data; instead, a volume should be mounted on the container to provide persistent storage not managed by UnionFS.

The Copy-on-Write (CoW) strategy of UnionFS allows users to “indirectly” modify the content of files available to the running container from the base container image storage layer. While the container image files are Read-Only, when a user attempts to modify such a file, no errors or mechanisms will prevent the user from doing so. Instead, the base container image file is copied and saved on the ephemeral storage layer of the container and the user is allowed to make changes to the new copy of the file. In essence, a copy of a file is being saved when a user attempts to edit a Read-Only file of the base container image, all while the base container image file remains intact.

This strategy is used at the operating system level for memory management and process management, and it is used by Docker to manage the storage for container images, running containers, and to minimize I/O and the size of each storage layer.

### Containers and non-persistent data

Containers are designed to be immutable. This is just a buzzword that means read-only — it’s a best practice not to change the conﬁguration of a container after it’s deployed. If something breaks or you need to change something, you should create a new container with the ﬁxes/updates and deploy it in place of the old container. You shouldn’t log into a running container and make conﬁguration changes! However, many applications require a read-write ﬁlesystem in order to simply run – they won’t even run on a read-only ﬁlesystem. This means it’s not as simple as making containers entirely read-only. Every Docker container is created by adding a thin read-write layer on top of the read-only image it’s based on. Figure below shows two running containers sharing a single read-only image.

<img src=".\images\ContainerLayerRO.png" style="width:75%; height: 75%;">

The writable container layer exists in the ﬁlesystem of the Docker host, and you’ll hear it called various names. These include *local storage*, *ephemeral storage*, and *graphdriver storage*. 

It’s typically located on the Docker host in these locations:

- Linux Docker hosts: **'/var/lib/docker/storage-driver/...'**

- Windows Docker hosts: **'C:\ProgramData\Docker\windowsfilter\\...'**

This thin writable layer is an integral part of a container and enables all read/write operations. If you, or an application, update ﬁles or add new ﬁles, they’ll be written to this layer. However, it’s tightly coupled to the container’s lifecycle — it gets created when the container is created and it gets deleted when the container is deleted. The fact that it’s deleted along with a container means that it’s not an option for important data that you need to keep (persist). If your containers don’t create persistent data, this thin writable layer of *local storage* will be ﬁne and you’re good to go. However, if your containers need to persist data, you need to read the next section.

### Docker Storage Drivers

Typically, not a lot of data needs to be written to the container’s writable layer. However, when such a requirement presents itself, then a storage driver needs to be used to control how the container images and the running containers are stored and managed on the host system.

While Docker supports a variety of storage drivers, each driver may be limited by its supported backing filesystems. 

The **overlay**, **overlay2** and **aufs** drivers are supported by **xfs** and **ext4** filesystems, **devicemapper** driver is backed by **direct-lvm**, while **vfs** is supported by any filesystem. 

Despite the implementation and feature differences between storage drivers, they all use the CoW strategy for stacked storage layers.

### Containers and persistent data

**Volumes** are the recommended way to persist data in containers. There are three major reasons for this:

- Volumes are independent objects that are not tied to the lifecycle of a container
- Volumes can be mapped to specialized external storage systems
- Volumes enable multiple containers on different Docker hosts to access and share the same data

At a high-level, you create a volume, then you create a container and mount the volume into it. The volume is mounted into a directory in the container’s ﬁlesystem, and anything written to that directory is stored in the volume. If you delete the container, the volume and its data will still exist.

<img src=".\images\DockerVolume.png" style="width:75%; height: 75%;">

### Creating and managing Docker volumes

Volumes are ﬁrst-class citizens in Docker. Among other things, this means they are their own object in the API and have their own docker volume sub-command.

```shell
$ docker volume create myvol
```

By default, Docker creates new volumes with the built-in local driver. As the name suggests, volumes created with the local driver are only available to containers on the same node as the volume. You can use the -d ﬂag to specify a different driver. Now that the volume is created, you can see it with the docker volume ls command and inspect it.

```shell
$ docker volume ls

$ docker volume inspect myvol
```

Notice that the Driver and Scope are both local. This means the volume was created with the local driver and is only available to containers on this Docker host. The Mountpoint property tells us where in the Docker host’s ﬁlesystem the volume exists.

All volumes created with the local driver get their own directory under **'/var/lib/docker/volumes'** on Linux, and **'C:\ProgramData\Docker\volumes'** on Windows. 

**This means you can see them in your Docker host’s ﬁlesystem. You can even access them directly from your Docker host, although this is not normally recommended.**

Now the volume is created, it can be used by one or more containers. 

There are two ways to delete a Docker volume: 
- **'docker volume prune'** will delete **all volumes** that are not mounted into a container or service replica, so **use with caution!**. 
- **'docker volume rm'** lets you specify exactly which volumes you want to delete. 

Neither command will delete a volume that is in use by a container or service replica. As the myvol volume is not in use, delete it with the prune command.

```shell
$ docker volume prune

$ docker volume rm
```

However, it’s also possible to deploy volumes via Dockerﬁles using the VOLUME instruction. The format is **'VOLUME container-mount-point'**. 

Interestingly, you cannot specify a directory on the host when deﬁning a volume in a Dockerﬁle. This is because *host* directories are different depending on what OS your Docker host is running – it could break your builds if you speciﬁed a directory on a Docker host that doesn’t exist. 

As a result, deﬁning a volume in a Dockerﬁle requires you to specify host directories at deploy-time.

### Demonstrating volumes with containers and services

```shell
$ docker container run -dit --name voltainer --mount source=bizvol,target=/vol alpine
```

The command uses the --mount ﬂag to mount a volume called “bizvol” into the container at either /vol or c:\vol. 

The command completes successfully despite the fact there is no volume on the system called bizvol. 

This raises an interesting point:
- If you specify an existing volume, Docker will use the existing volume
- If you specify a volume that doesn’t exist, Docker will create it for you.

```shell
$ docker volume ls
```

Although containers and volumes have separate lifecycle’s, you cannot delete a volume that is in use by a container.

```shell
$ docker volume rm bizvol
```

The volume is brand new, so it doesn’t have any data. Let’s exec onto the container and write some data to it.

```sh
$ docker container exec -it voltainer sh

# echo "I promise to write a review of the book on Amazon" > /vol/file1
# ls -l /vol
# cat /vol/file1
```

Type exit to return to the shell of your Docker host, and then delete the container with the following command.

```shell
$ docker container rm voltainer -f
```

Even though the container is deleted, the volume still exists:

```shell
$ docker container ls -a

$ docker volume ls
```

Because the volume still exists, you can look at its mount point on the host to check if the data is still there. Run the following commands from the terminal of your Docker host. The ﬁrst one will show that the ﬁle still exists, the second will show the contents of the ﬁle.

```shell
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

### Sharing storage across cluster nodes

Integrating external storage systems with Docker makes it possible to share volumes between cluster nodes. These external systems can be cloud storage services or enterprise storage systems in your on-premises data centers. As an example, a single storage LUN or NFS share can be presented to multiple Docker hosts, allowing it to be used by containers and service replicas no-matter which Docker host they’re running on. Figure below shows a single external shared volume being presented to two Docker nodes. These Docker nodes can then make the shared volume available to either, or both containers.

<img src=".\images\ShareStorage.png" style="width:75%; height: 75%;">

Building a setup like this requires a lot of things. You need access to a specialised storage systems and knowledge of how it works and presents storage. You also need to know how your applications read and write data to the shared storage. Finally, you need a volumes driver plugin that works with the external storage system.

Docker Hub is the best place to ﬁnd volume plugins. Login to Docker Hub, select the view to show plugins instead of containers, and ﬁlter results to only show Volume plugins. Once you’ve located the appropriate plugin for your storage system, you create any conﬁguration ﬁles it might need, and install it with docker plugin install. Once the plugin is registered, you can create new volumes from the storage system using docker volume create with the -d ﬂag. 

The following example installs the Pure Storage Docker volume plugin. This plugin provides access to storage volumes on either a Pure Storage FlashArray or FlashBlade storage system. Plugins only work with the correct external storage systems.

1. The Pure Storage plugin requires a conﬁguration ﬁle called pure.json in the Docker host’s /etc/pure-docker-plugin/directory. This ﬁle contains the information required for the plugin to locate the external storage system, authenticate, and access resources.

2. Install the plugin and grant the required permissions.

```shell
$ docker plugin install purestorage/docker-plugin:latest --alias pure --grant-all-permissions

$ docker plugin ls
```

Create a new volume with the plugin (you can also do this as part of the container creation process). This example creates a new 25GB volume called “fastvol” on the registered Pure Storage backend.

```sh
$ docker volume create -d pure -o size=25GB fastvol
```

Different storage drivers support different options, but this should be enough to give you a feel for how they work.

### Potential data corruption

A major concern with any conﬁguration that shares a single volume among multiple containers is **data corruption**.

Assume the following example based on Figure above. The application running in ctr-1 on node-1 updates some data in the shared volume. However, instead of writing the update directly to the volume, it holds it in its local buffer for faster recall (this is common in many operating systems). At this point, the application in ctr-1 thinks the data has been written to the volume. However, before ctr-1 on node-1 ﬂushes its buffers and commits the data to the volume, the app in ctr-2 on node-2 updates the same data with a different value and commits it directly to the volume. At this point, both applications *think* they’ve updated the data in the volume, but in reality only the application in ctr-2 has. A few seconds later, ctr-1 on node-1 ﬂushes the data to the volume, overwriting the changes made by the application in ctr-2. However, the application in ctr-2 is totally unaware of this! 

This is one of the ways data corruption happens. To prevent this, you need to write your applications in a way to avoid things like this.
