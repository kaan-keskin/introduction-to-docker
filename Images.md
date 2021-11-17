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
