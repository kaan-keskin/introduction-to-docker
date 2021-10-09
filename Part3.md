# Introduction to Docker - Part 3

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

> - Container Networking
> - Docker Overlay Networking
> - Volumes and Persistent Data
> - Deploying apps with Docker Stacks
> - Runtime and Containers Security
> - Security in Docker

## Container Networking

It’s always the network!

Any time there’s a an infrastructure problem, we always blame the network. Part of the reason is that networks are at the center of everything — **no network, no app!**

### Standards and Specifications

A typical computing machine, whether a physical host system or a Virtual Machine, is connected to at least one network in order for the system and any running applications to become part of the enterprise’s software ecosystem. The network allows systems to connect with each other, allows applications to communicate with each other, allows for data to be transmitted not only between neighboring systems but also to systems and applications running halfway across the world. In addition, a connected system can be managed remotely, and its resources and applications performance may be monitored. Networking, however, is not at random. Instead, network connectivity and network traffic are governed by networking standards and principles, topologies, protocols, policies, rules, etc.

**Since containers encapsulate running applications, similarly to individual Virtual Machines, they also need to be attached to at least one network. The container network enables communication between microservices running inside containers, containers running on the same host, running on different hosts, or a container and other applications and services from anywhere in the world. Connected containers are also easier to manage and monitor.**

Container networking is guided by sets of standards that specify how a container may join one or multiple networks simultaneously. There are two container networking standards we need to be concerned about: the **Container Network Model (CNM)** and the **Container Network Interface (CNI)**. Both networking models present pros and cons in terms of adoption and level of support. 

Note: You can read more about the container networking standards in **The Container Networking Landscape: CNI from CoreOS and CNM from Docker**.

https://thenewstack.io/container-networking-landscape-cni-coreos-cnm-docker/

### The Container Network Interface (CNI)

The Container Network Interface (CNI), a Cloud Native Computing Foundation Incubator project, is the container network specification introduced by CoreOS, adopted by projects such as Mesos, Amazon ECS, Kubernetes, Cloud Foundry, OpenShift, rkt, and supported by projects such as Calico, Cilium, Romana, CNI-Genie, VMware NSX, and Weave. It links container platforms such as Kubernetes and Cloud Foundry to multiple distinct container network implementations.

CNI consists of a specification and libraries for network plugin development. CNI is a much simpler specification, focusing only on the network connectivity of a container and the release of networking resources once the container is deleted.

### Docker Networking

In this chapter, we’ll look at the fundamentals of Docker networking. Things like the Container Network Model (CNM) and libnetwork. We’ll also get our hands dirty building some networks.

Docker runs applications inside of containers, and applications need to communicate over lots of diﬀerent networks. This means Docker needs strong networking capabilities.

Fortunately, Docker has solutions for container-to-container networks, as well as connecting to existing networks and VLANs. The later is important for containerized apps that interact with functions and services on external systems such as VM’s and physical servers.

Docker networking is based on an open-source pluggable architecture called the Container Network Model (CNM). **libnetwork is Docker’s real-world implementation of the CNM, and it provides all of Docker’s core networking capabilities.** Drivers plug in to libnetwork to provide speciﬁc network topologies.

To create a smooth out-of-the-box experience, Docker ships with a set of native drivers that deal with the most common networking requirements. These include single-host bridge networks, multi-host overlays, and options for plugging into existing VLANs. Ecosystem partners can extend things further by providing their own drivers.

Last but not least, libnetwork provides a native service discovery and basic container load balancing solution. That's this big picture. Let’s get into the detail.

### The theory

At the highest level, Docker networking comprises three major components:
- The Container Network Model (CNM)
- `libnetwork`
- Drivers

The CNM is the design speciﬁcation. It outlines the fundamental building blocks of a Docker network. `libnetwork` is a real-world implementation of the CNM, and is used by Docker. It’s written in Go, and implements the core components outlined in the CNM.

Drivers extend the model by implementing speciﬁc network topologies such as VXLAN overlay networks. Figure shows how they ﬁt together at a very high level.

<img src=".\images\DockerNetwork.png" style="width:75%; height: 75%;">

### The Container Network Model (CNM)

The Container Network Model (CNM) is the container network specification introduced by Docker and implemented by the libnetwork project. Companies and other projects such as Cisco, Calico, Weave, Kuryr, VMware, and Open Virtual Networking (OVN) adopted the CNM.

Everything starts with a design.

The design guide for Docker networking is the CNM. It outlines the fundamental building blocks of a Docker network, and you can read the full spec here: https://github.com/Docker/libnetwork/blob/master/docs/design.md

I recommend reading the entire spec, but at a high level, it deﬁnes three major building blocks:
- Sandboxes
- Endpoints
- Networks

The CNM specifies a set of objects for a container to join a network and be able to talk to other containers that are part of that network. 

The CNM specifies:

- **A network sandbox**, an isolated networking stack inside the container which may support multiple individual networks through endpoints. It includes; Ethernet interfaces, ports, routing tables, and DNS conﬁg.

- **A container endpoint**, specified by the CNM and attached to the network sandbox, is an interface paired with an interface on a network, allowing the container to connect to that particular network. **Endpoints** are virtual network interfaces (E.g. veth). Like normal network interfaces, they’re responsible for making connections. In the case of the CNM, it’s the job of the *endpoint* to connect a *sandbox* to a *network*.

- The network sandbox supports multiple endpoints, each paired with a different network, thus allowing one container to access multiple networks simultaneously, where the network objects are also specified by the CNM. **Networks** are a software implementation of an switch (802.1d bridge). As such, they group together and isolate a collection of endpoints that need to communicate. Figure shows the three components and how they connect.

<img src=".\images\DockerNetwork2.png" style="width:75%; height: 75%;">

The atomic unit of scheduling in a Docker environment is the container, and as the name suggests, the Container Network Model is all about providing networking to containers. Figure shows how CNM components relate to containers — sandboxes are placed inside of containers to provide network connectivity.

<img src=".\images\DockerNetwork3.png" style="width:75%; height: 75%;">

Container A has a single interface (endpoint) and is connected to Network A. Container B has two interfaces (endpoints) and is connected to Network A **and** Network B. The two containers will be able to communicate because they are both connected to Network A. However, the two *endpoints* in Container B cannot communicate with each other without the assistance of a layer 3 router.

It’s also important to understand that *endpoints* behave like regular network adapters, meaning they can only be connected to a single network. Therefore, if a container needs connecting to multiple networks, it will need multiple endpoints.

Figure extends the diagram again, this time adding a Docker host. Although Container A and Container B are running on the same host, their network Stacks are completely isolated at the OS-level via the sandboxes.

<img src=".\images\DockerNetwork4.png" style="width:75%; height: 75%;">

#### Libnetwork

The CNM is the design doc, and libnetwork is the canonical implementation. It’s open-source, written in Go, cross-platform (Linux and Windows), and used by Docker.

In the early days of Docker, all the networking code existed inside the daemon. This was a nightmare — the daemon became bloated, and it didn’t follow the Unix principle of building modular tools that can work on their own, but also be easily composed into other projects. As a result, it all got ripped out and refactored into an external library called libnetwork based on the principles of the CNM. Nowadays, all of the core Docker networking code lives in libnetwork.

As you’d expect, it implements all three of the components deﬁned in the CNM. It also implements native *service discovery*, *ingress-based container load balancing*, and the network control plane and management plane functionality.

#### Docker Networking Drivers

If libnetwork implements the control plane and management plane functions, then drivers implement the data plane. For example, connectivity and isolation is all handled by drivers. So is the actual creation of networks. The relationship is shown in Figure.

<img src=".\images\DockerNetwork5.png" style="width:75%; height: 75%;">

Docker ships with several built-in drivers, known as native drivers or *local drivers*. 

**On Linux they include:**
- bridge, 
- host,
- ipvlan,
- macvlan,
- null, 
- overlay. 

On Windows they include; nat, overlay, transparent, and l2bridge.

**Bridge**

The default network type that Docker containers attach to is the bridge network, implemented by the bridge driver. The main roles of a bridge network are to isolate the container network from the host system network, and to act as a DHCP server and assign unique IP addresses to containers as they are running attached to the bridge. This is a useful feature when containers of an application need to talk to each other while they all run on the same host system. Other features of bridge networks depend whether the bridge is default or user-defined. Most often, a user-defined bridge presents advantages over the default bridge, such as better traffic isolation, automatic DNS resolution, on-the-fly container connection/disconnection to and from the network, advanced and flexible configuration options, and even sharing of environment variables between containers.

**Host**

The host network driver option, as opposed to the bridge, eliminates the network isolation between the container and the host system by allowing the container to directly access the host network. In addition, it may help with performance optimization by eliminating the need for Network Address Translation (NAT) or proxying since the container ports are automatically published and available as host ports.

**Overlay**

Gaining a lot of popularity these days is the overlay network type supported by the overlay driver. It is the network type that spans multiple hosts, typically part of a cluster, allowing the containers' traffic to be routed between hosts as containers or services from one host attempt to talk to others running on another host in the cluster.

This network type is very popular with container orchestration platforms because it not only enables traffic across the entire cluster of host systems, but also provides traffic isolation and management through rules and policies.

**Macvlan**

The macvlan network driver allows a user to change the appearance of a container on the physical network. A container may appear as a physical device with its own MAC address on the network, thus enabling the container to be directly connected to the physical network instead of having its traffic routed through the host network.

**None**

The none driver option for container networking disables the networking of a container while allowing the very same container to use a custom third-party network driver, if needed, to implement its networking requirements.

**Network Plugins**

Network plugins expand the capabilities of Docker through third-party network drivers that integrate Docker with specialized network stacks. Certain network plugins may combine features otherwise available from single network drivers while improving network resiliency and policy management.

3rd-parties can also write Docker network drivers known as **remote drivers** or **plugins**. Weave Net is a popular example and can be downloaded from Docker Hub.

Each driver is in charge of the actual creation and management of all resources on the networks it is responsible for. For example, an overlay network called “prod-fe-cuda” will be owned and managed by the overlay driver. This means the overlay driver will be invoked for the creation, management, and deletion of all resources on that network.

In order to meet the demands of complex highly-ﬂuid environments, libnetwork allows multiple network drivers to be active at the same time. This means your Docker environment can sport a wide range of heterogeneous networks.

### Single-host bridge networks

The simplest type of Docker network is the single-host bridge network.

The name tells us two things:
- **Single-host** tells us it only exists on a single Docker host and can only connect containers that are on the same host.
- **Bridge** tells us that it’s an implementation of an 802.1d bridge (layer 2 switch).

Docker on Linux creates single-host bridge networks with the built-in bridge driver, whereas Docker on Windows creates them using the built-in nat driver. For all intents and purposes, they work the same.

Figure shows two Docker hosts with identical local bridge networks called “mynet”. Even though the networks are identical, they are independent isolated networks. This means the containers in the picture cannot communicate directly because they are on diﬀerent networks.

<img src=".\images\SingleHostBridgeNetwork.png" style="width:75%; height: 75%;">

Every Docker host gets a default single-host bridge network. On Linux it’s called “bridge”, and on Windows it’s called “nat” (yes, those are the same names as the drivers used to create them). By default, this is the network that all new containers will be connected to unless you override it on the command line with the --network ﬂag.

The following listing shows the output of a docker network ls command on newly installed Linux and Windows Docker hosts. The output is trimmed so that it only shows the default network on each host. Notice how the name of the network is the same as the driver that was used to create it — this is a coincidence and not a requirement.

```shell
$ docker network ls
```

The docker network inspect command is a treasure trove of great information. I highly recommended reading through its output if you’re interested in low-level detail.

```shell
docker network inspect bridge
```

Docker networks built with the bridge driver on Linux hosts are based on the battle-hardened *linux bridge* technology that has existed in the Linux kernel for nearly 20 years. This means they’re high performance and extremely stable. It also means you can inspect them using standard Linux utilities. For example.

```shell
$ ip link show docker0
```

The default “bridge” network, on all Linux-based Docker hosts, maps to an underlying *Linux bridge* in the kernel called “**Docker0**”. We can see this from the output of docker network inspect.

```shell
$ docker network inspect bridge | grep bridge.name
```

The relationship between Docker’s default “bridge” network and the “Docker0” bridge in the Linux kernel is shown in Figure.

<img src=".\images\SingleHostBridgeNetwork2.png" style="width:75%; height: 75%;">

Figure extends the diagram by adding containers at the top that plug into the “bridge” network. The “bridge” network maps to the “Docker0” Linux bridge in the host’s kernel, which can be mapped back to an Ethernet interface on the host via port mappings.

<img src=".\images\SingleHostBridgeNetwork3.png" style="width:75%; height: 75%;">

Let’s use the docker network create command to create a new single-host bridge network called “localnet”.

```shell
$ docker network create -d bridge localnet
```

The new network is created and will appear in the output of any future docker network ls commands. If you are using Linux, you will also have a new *Linux bridge* created in the kernel.

Let’s use the Linux brctl tool to look at the Linux bridges currently on the system. You may have to manually install the brctl binary using apt-get install bridge-utils, or the equivalent for your Linux distro.

```shell
$ brctl show
```

The output shows two bridges. The ﬁrst line is the “Docker0” bridge that we already know about. This relates to the default “bridge” network in Docker. The second bridge (br-20c2e8ae4bbb) relates to the new localnet Docker bridge network. Neither of them have spanning tree enabled, and neither have any devices connected (interfaces column).

At this point, the bridge conﬁguration on the host looks like Figure.

<img src=".\images\SingleHostBridgeNetwork4.png" style="width:75%; height: 75%;">

Let’s create a new container and attach it to the new localnet bridge network. 

If you’re following along on Windows, you should substitute “alpine sleep 1d” with “mcr.microsoft.com/powershell:nanoserver pwsh.exe -Command Start-Sleep 86400”.

```shell
$ docker container run -d --name c1 \
    --network localnet \
    alpine sleep 1d
```

This container will now be on the localnet network. You can conﬁrm this with a docker network inspect.

```shell
$ docker network inspect localnet --format '{{json .Containers}}'

{
    "4edcbd...842c3aa": {
        "Name": "c1",
        "EndpointID": "43a13b...3219b8c13",
        "MacAddress": "02:42:ac:14:00:02",
        "IPv4Address": "172.20.0.2/16",
        "IPv6Address": ""
    }
}
```

The output shows that the new “c1” container is on the localnet bridge/nat network.

It you run the Linux brctl show command again, you’ll see c1’s interface attached to the br-20c2e8ae4bbb bridge.

```shell
$ brctl show
```

This is shown in the figure below.

<img src=".\images\SingleHostBridgeNetwork5.png" style="width:75%; height: 75%;">

If we add another new container to the same network, it should be able to ping the “c1” container by name. This is because **all new containers are automatically registered with the embedded Docker DNS service, enabling them to resolve the names of all other containers on the same network**.

> **CRITICAL INFORMATION:** 

> **The default bridge network on Linux does not support name resolution via the Docker DNS service. All other *user-deﬁned* bridge networks do.** The following demo will work because the container is on the user-deﬁned localnet network.

Let’s test it.

Create a new interactive container called “c2” and put it on the same localnet network as “c1”.

```shell
$ docker container run -it --name c2 \
    --network localnet \
    alpine sh
```

Your terminal will switch into the “c2” container.

From within the “c2” container, ping the “c1” container by name.

```shell
$ ping c1
```

It works! This is because the c2 container is running a local DNS resolver that forwards requests to an internal Docker DNS server. This DNS server maintains mappings for all containers started with the `--name` or `--net-alias` ﬂag.

Try running some network-related commands while you’re still logged on to the container. It’s a great way of learning more about how Docker container networking works. The following snippet shows the ipconfig command ran from inside the “c2” Windows container previously created. You can Ctrl+P+Q out of the container and run another docker network inspect localnet command to match the IP addresses.

```powershell
PS C:\> ipconfig
```

So far, we’ve said that containers on bridge networks can only communicate with other containers on the same network. However, you can get around this using *port mappings*.

Port mappings let you map a container to a port on the Docker host. Any traﬃc Hitting the Docker host on the conﬁgured port will be directed to the container. The high-level ﬂow is shown in the figure below.

<img src=".\images\SingleHostBridgeNetwork6.png" style="width:75%; height: 75%;">

In the diagram, the application running in the container is operating on port 80. This is mapped to port 5000 on the host’s 10.0.0.15 interface. The end result is all traﬃc hitting the host on 10.0.0.15:5000 being redirected to the container on port 80.

Let’s walk through an example of mapping port 80 on a container running a web server, to port 5000 on the Docker host. The example will use NGINX on Linux. If you’re following along on Windows, you’ll need to substitute nginx with a Windows-based web server image such as mcr.microsoft.com/windows/servercore/iis:nanoserver.

Run a new web server container and map port 80 on the container to port 5000 on the Docker host.

```shell
$ docker container run -d --name web \
    --network localnet \
    --publish 5000:80 \
    nginx
```

Verify the port mapping.

```shell
$ docker port web

80/tcp -> 0.0.0.0:5000
```

This shows that port 80 in the container is mapped to port 5000 on all interfaces on the Docker host.

Test the conﬁguration by pointing a web browser to port 5000 on the Docker host. To complete this step, you’ll need to know the IP or DNS name of your Docker host. If you’re using Docker Desktop on Mac or Windows, you’ll be able to use localhost:5000 or 127.0.0.1:5000.

<img src=".\images\SingleHostBridgeNetworkApp.png" style="width:75%; height: 75%;">

Any external system can now access the NGINX container running on the localnet bridge network via a port mapping to TCP port 5000 on the Docker host.

Mapping ports like this works, but it’s clunky and doesn’t scale. For example, only a single container can bind to any port on the host. This means no other containers on that host will be able to bind to port 5000. This is one of the reason’s that single-host bridge networks are only useful for local development and very small applications.

### Multi-host overlay networks

Overlay networks are multi-host. They allow a single network to span multiple hosts so that containers on diﬀerent hosts can communicate directly. They’re ideal for container-to-container communication, including container-only applications, and they scale well.

Docker provides a native driver for overlay networks. This makes creating them as simple as adding the --d overlay ﬂag to the docker network create command.

### Connecting to existing networks

The ability to connect containerized apps to external systems and physical networks is vital. A common example is a partially containerized app — the containerized parts need a way to communicate with the non-containerized parts still running on existing physical networks and VLANs.

The built-in MACVLAN driver (transparent on Windows) was created with this in mind. It makes containers ﬁrst-class citizens on the existing physical networks by giving each one its own MAC address and IP addresses.

<img src=".\images\ConnectExistNetwork.png" style="width:75%; height: 75%;">

On the positive side, MACVLAN performance is good as it doesn’t require port mappings or additional bridges — you connect the container interface through to the hosts interface (or a sub-interface). However, on the negative side, it requires the host NIC to be in **promiscuous mode**, which isn’t always allowed on corporate networks and public cloud platforms. So MACVLAN is great for your corporate data center networks (assuming your network team can accommodate promiscuous mode), but it might not work in the public cloud.

Let’s dig a bit deeper with the help of some pictures and a hypothetical example.

Assume we have an existing physical network with two VLANS:
- VLAN 100: 10.0.0.0/24
- VLAN 200: 192.168.3.0/24

<img src=".\images\ConnectExistNetwork2.png" style="width:75%; height: 75%;">

Next, we add a Docker host and connect it to the network.

<img src=".\images\ConnectExistNetwork3.png" style="width:75%; height: 75%;"/>

We then have a requirement for a container running on that host to be plumbed into VLAN 100. To do this, we create a new Docker network with the macvlan driver. However, the macvlan driver needs us to tell it a few things about the network we’re going to associate it with. 

Things like:
- Subnet info
- Gateway
- Range of IP’s it can assign to containers
- which interface or sub-interface on the host to use

The following command will create a new MACVLAN network called “macvlan100” that will connect containers to VLAN 100.

```shell
$ docker network create -d macvlan \
    --subnet=10.0.0.0/24 \
    --ip-range=10.0.0.0/25 \
    --gateway=10.0.0.1 \
    -o parent=eth0.100 \
    macvlan100
```

This will create the “macvlan100” network and the eth0.100 sub-interface. The conﬁg now looks like this.

<img src=".\images\ConnectExistNetwork4.png" style="width:75%; height: 75%;"/>

MACVLAN uses standard Linux sub-interfaces, and you have to tag them with the ID of the VLAN they will connect to. In this example we’re connecting to VLAN 100, so we tag the sub-interface with macvlan100 (etho.100).

We also used the --ip-range ﬂag to tell the MACVLAN network which sub-set of IP addresses it can assign to containers. It’s vital that this range of addresses be reserved for Docker and not in use by other nodes or DHCP servers, as there is no management plane feature to check for overlapping IP ranges.

The macvlan100 network is ready for containers, so let’s deploy one with the following command.

```shell
$ docker container run -d --name mactainer1 \
    --network macvlan100 \
    alpine sleep 1d
```

The underlying network (VLAN 100) does not see any of the MACVLAN magic, it only sees the container with its MAC and IP addresses. And with that in mind, the “mactainer1” container will be able to ping and communicate with any other systems on VLAN 100.

> **Note:** If you can’t get this to work, it might be because the host NIC is not in promiscuous mode. Remember that public cloud platforms don’t usually allow promiscuous mode.

<img src=".\images\ConnectExistNetwork5.png" style="width:75%; height: 75%;">

At this point, we’ve got a MACVLAN network and used it to connect a new container to an existing VLAN. However, it doesn’t stop there. The Docker MACVLAN driver is built on top of the tried-and-tested Linux kernel driver with the same name. As such, it supports VLAN trunking. This means we can create multiple MACVLAN networks and connect containers on the same Docker host to them.

<img src=".\images\ConnectExistNetwork6.png" style="width:75%; height: 75%;">

That Pretty much covers MACVLAN. Windows oﬀers a similar solution with the transparent driver.

**Container and Service logs for troubleshooting**

A quick note on troubleshooting connectivity issues before moving on to Service Discovery. If you think you’re experiencing connectivity issues between containers, it’s worth checking the Docker daemon logs as well as container logs.

On Windows systems, the daemon logs are stored under **'∼AppData\Local\Docker'**, and you can view them in the Windows Event Viewer. 

> On Linux, it depends what init system you’re using. If you’re running a systemd, the logs will go to **journald** and you can view them with the **'journalctl -u docker.service'** command. 

If you’re not running systemd you should look under the following locations:
- Ubuntu systems running upstart: **'/var/log/upstart/docker.log'**
- RHEL-based systems: **'/var/log/messages'**
- Debian: **'/var/log/daemon.log'**

You can also tell Docker how verbose you want daemon logging to be. To do this, edit the daemon conﬁg ﬁle (`daemon.json`) so that “debug” is set to “true” and “log-level” is set to one of the following:
- `debug` The most verbose option
- `info` The default value and second-most verbose option
- `warn` Third most verbose option
- `error` Fourth most verbose option
- `fatal` Least verbose option

The following snippet from a daemon.json enables debugging and sets the level to debug. It will work on all Docker platforms.

```yaml
{
    "debug": true,
    "log-level": "debug",
}
```

Be sure to restart Docker after making changes to the ﬁle.

That was the daemon logs. What about container logs?

Logs from standalone containers can be viewed with the **'docker container logs'** command, and Swarm service logs can be viewed with the **'docker service logs'** command. However, Docker supports lots of logging drivers, and they don’t all work with the docker logs command.

As well as a driver and conﬁguration for daemon logs, every Docker host has a default logging driver and conﬁguration for containers. 

Some of the drivers include:
- **json-file** (default)
- **journald** (only works on Linux hosts running systemd)
- syslog
- splunk
- gelf
- local
- awslogs
- fluentd
- gcplogs
- logentries

json-file and journald are probably the easiest to conﬁgure, and they both work with the docker logs and docker service logs commands. The format of the commands is **'docker logs container-name'** and **'docker service logs service-name'**.

If you’re using other logging drivers you can view logs using the 3-rd party platform’s native tools.

The following snippet from a daemon.json shows a Docker host conﬁgured to use syslog.

```yaml
{
    "log-driver": "syslog",
}
```

**You can conﬁgure an individual container, or service, to start with a particular logging driver with the **--log-driver** and **--log-opts** ﬂags. These will override anything set in daemon.json.**

**Container logs work on the premise that your application is running as PID 1 inside the container and sending logs to STDOUT, and errors to STDERR. The logging driver then forwards these “logs” to the locations conﬁgured via the logging driver.**

If your application logs to a ﬁle, it’s possible to use a symlink to redirect log-ﬁle writes to STDOUT and STDERR.

The following is an example of running the docker logs command against a container called “vantage-db” conﬁgured to use the json-file logging driver.

```shell>
$ docker logs vantage-db
```

There’s a good chance you’ll ﬁnd network connectivity errors reported in the daemon logs or container logs.

### Docker Network Modes

Newer features of Docker networking modes aim to further manage network capabilities in multi-host clusters, especially with Docker Swarm.

#### Internal Mode

We can isolate an overlay network with the internal mode, when we want to restrict containers’ access to the outside world. Otherwise, when a container is connected to an overlay network, Docker connects that container to an additional bridge network to ensure the container has access to the outside world. 

#### Ingress Mode

This mode targets the networking of swarms, and implements a routing-mesh across the hosts of a swarm cluster. While similar to an overlay network in supported options, only one ingress can be defined, and it cannot be removed for as long as services are using it.

### Service discovery

As well as core networking, libnetwork also provides some important network services.

> **Service discovery** allows all containers and Swarm services to locate each other by name. The only requirement is that they be on the same network.

Under the hood, this leverages Docker’s embedded DNS server and the DNS resolver in each container. Figure shows container “c1” pinging container “c2” by name. The same principle applies to Swarm Services.

<img src=".\images\ServiceDiscovery.png" style="width:75%; height: 75%;"/>

Let’s step through the process.

- **Step 1:** The ping c2 command invokes the local DNS resolver to resolve the name “c2” to an IP address. **All Docker containers have a local DNS resolver.**

- **Step 2:** If the local resolver doesn’t have an IP address for “c2” in its local cache, it initiates a recursive query to the Docker DNS server. **The local resolver is pre-conﬁgured to know how to reach the Docker DNS server.**

- **Step 3:** The Docker DNS server holds name-to-IP mappings for all containers created with the --name or --net-alias ﬂags. This means it knows the IP address of container “c2”.

- **Step 4:** The DNS server returns the IP address of “c2” to the local resolver in “c1”. It does this because the two containers are on the same network — if they were on diﬀerent networks this would not work.

- **Step 5:** The ping command issues the ICMP echo request packets to the IP address of “c2”. Every Swarm service and standalone container started with the --name ﬂag will register its name and IP with the Docker DNS service. This means all containers and service replicas can use the Docker DNS service to ﬁnd each other.

However, **service discovery is *network-scoped*.** This means that name resolution only works for containers and Services on the same network. If two containers are on diﬀerent networks, they will not be able to resolve each other.

One last point on service discovery and name resolution:

It’s possible to conﬁgure Swarm services and standalone containers with customized DNS options. 

For example, **the --dns ﬂag lets you specify a list of custom DNS servers to use in case the embedded Docker DNS server cannot resolve a query.** This is common when querying names of services outside of Docker. 

**You can also use the --dns-search ﬂag to add custom search domains for queries against unqualiﬁed names (i.e. when the query is not a fully qualiﬁed domain name).**

**On Linux, these all work by adding entries to the '/etc/resolv.conf' ﬁle inside every container.**

The following example will start a new standalone container and add the infamous 8.8.8.8 Google DNS server, as well as nigelpoulton.com as search domain to append to unqualiﬁed queries.

```shell
$ docker container run -it --name c1 \
    --dns=8.8.8.8 \
    --dns-search=nigelpoulton.com \
    alpine sh
```

### Ingress load balancing

Swarm supports two publishing modes that make services accessible outside of the cluster:
- Ingress mode (default)
- Host mode

**Services published via *ingress mode* can be accessed from any node in the Swarm — even nodes **not** running a service replica.** 

**Services published via *host mode* can only be accessed by Hitting nodes running service replicas.** 

Figure below shows the diﬀerence between the two modes.

<img src=".\images\IngressLoadBalance.png" style="width:75%; height: 75%;"> 

Ingress mode is the default. This means any time you publish a service with -p or --publish it will default to *ingress mode*. 

To publish a service in *host mode* you need to use the long format of the --publish ﬂag **and** add mode=host. 

Let’s see an example using *host mode*.

```shell
$ docker service create -d --name svc1 \
    --publish published=5000,target=80,mode=host \
    nginx
```

A few notes about the command. docker service create lets you publish a service using either a *long form* *syntax* or *short form syntax*. 

The short form looks like this: -p 5000:80 and we’ve seen it a few times already. However, you cannot publish a service in *host mode* using short form.

The long form looks like this: --publish published=5000,target=80,mode=host. It’s a comma-separate list with no whitespace after each comma. 

The options work as follows:
- published=5000 makes the service available externally via port 5000
- target=80 makes sure that external requests to the published port get mapped back to port 80 on the service replicas
- mode=host makes sure that external requests will only reach the service if they come in via nodes running a service replica.

Ingress mode is what you’ll normally use.

> Behind the scenes, *ingress mode* uses a **layer 4 routing mesh** called the **Service Mesh** or the **Swarm Mode Service Mesh**. Figure shows the basic traﬃc ﬂow of an external request to a service exposed in ingress mode.

<img src=".\images\IngressLoadBalance2.png" style="width:75%; height: 75%;">

Let’s quickly walk through the diagram.

1. The command at the top deploys a new Swarm service called “svc1”. It’s attaching the service to the overnet network and publishing it on port 5000.

2. **Publishing a Swarm service like this (--publish published=5000,target=80) will publish it on port 5000 on the ingress network. As all nodes in a Swarm are attached to the ingress network, this means the port is published *swarm-wide*.**

3. Logic is implemented on the cluster ensuring that any traﬃc Hitting the ingress network, via **any node**, on port 5000 will be routed to the “svc1” service on port 80.

4. At this point, a single replica for the “svc1” service is deployed, and the cluster has a mapping rule that says “*all traﬃc hitting the ingress network on port 5000 needs routing to a node running a replica for the* *“svc1” service*”.

5. The red line shows traﬃc hitting node1 on port 5000 and being routed to the service replica running on node2 via the ingress network.

It’s vital to know that the incoming traﬃc could have hit any of the four Swarm nodes on port 5000 and we would get the same result. This is because the service is published *swarm-wide* via the ingress network.

It’s also vital to know that if there were multiple replicas running, as shown in the figure below, the traﬃc would be balanced across all replicas.

<img src=".\images\IngressLoadBalance3.png" style="width:75%; height: 75%;">

## Docker overlay networking

Overlay networks are at the beating heart of many cloud-native microservices apps. In this chapter we’ll cover the fundamentals of native Docker overlay networking.

Docker overlay networking on Windows has feature parity with Linux. This means the examples we’ll use in this chapter will all work on Linux and Windows.

In the real world, it’s vital that containers can communicate with each other reliably and securely, even when they’re on diﬀerent hosts that are on diﬀerent networks. This is where overlay networking comes in to play. It allows you to create a ﬂat, secure, layer-2 network, spanning multiple hosts. Containers connect to this and can communicate directly.

Docker oﬀers native overlay networking that is simple to conﬁgure and secure by default.

Behind the scenes, it’s built on top of libnetwork and drivers. libnetwork is the canonical implementation of the Container Network Model (CNM) and drivers are pluggable components that implement diﬀerent networking technologies and topologies. Docker oﬀers native drivers, including the overlay driver.

In March 2015, Docker, Inc. acquired a container networking startup called *Socket Plane*. Two of the reasons behind the acquisition were to bring *real networking* to Docker, and to make container networking simple enough that even developers could do it.

They over-achieved on both.

However, hiding behind the simple networking commands are a lot of moving parts. The kind of stuﬀ you need to understand before doing production deployments and attempting to troubleshoot issues.

### Build and test a Docker overlay network in Swarm mode*

For the following examples, we’ll use two Docker hosts, on two separate Layer 2 networks, connected by a router.

See Figure, and note the diﬀerent networks that each node is on.

<img src=".\images\DockerOverlayNetworking.png" style="width:75%; height: 75%;">

You can follow along with either Linux or Windows Docker hosts. Linux should have at least a 4.4 Linux kernel (newer is always better) and Windows should be Windows Server 2016 or later with the latest hotﬁxes installed. You can also follow along on your Mac or Windows PC with Docker Desktop. However, you won’t see the full beneﬁts as they only support a single Docker host.

### Build a Swarm

The ﬁrst thing to do is conﬁgure the two hosts into a two-node swarm. This is because swarm mode is a pre-requisite for overlay networks.

We’ll run the `docker swarm init` command on **node1** to make it a *manager*, and then we’ll run the docker swarm join command on **node2** to make it a *worker*. This is not a production-grade setup, but it is enough for a learning lab. You’re encouraged to test with more managers and workers and expand on the examples.

If you are following along in your own lab, you’ll need to swap the IP addresses and the likes with the correct values for your environment.

Run the following command on **node1**.

```shell
$ docker swarm init \
    --advertise-addr=172.31.1.5 \
    --listen-addr=172.31.1.5:2377

Swarm initialized: current node (1ex3...o3px) is now a manager.
```

Run the next command on **node2**. You will need to ensure the following ports are enabled on any ﬁrewalls:

- **2377/tcp** for management plane comms
- **7946/tcp** and 7946/udp for control plane comms (SWIM-based gossip)
- **4789/udp** for the VXLAN data plane

```shell
$ docker swarm join \
    --token SWMTKN-1-0hz2ec...2vye \
    172.31.1.5:2377
```

This node joined a swarm as a worker.

We now have a two-node Swarm with **node1** as a manager and **node2** as a worker.

### Create a new overlay network

Now let’s create a new *overlay network* called **uber-net**.

Run the following command from **node1** (manager).

```shell
$ docker network create -d overlay uber-net

c740ydi1lm89khn5kd52skrd9
```

That's it. You’ve just created a brand-new overlay network that is available to all hosts in the Swarm and has its control plane encrypted with TLS (AES in GCM mode with keys automatically rotated every 12 hours). If you want to encrypt the data plane, you just add the -o encrypted ﬂag to the command. However, data plane encryption isn’t enabled by default because of the performance overhead. It’s highly recommended that you extensively test performance before enabling data plane encryption. However, if you do enable it, it’s protected by the same AES in GCM mode with key rotation.

> If you’re unsure about terms such as **control plane** and **data plane**, control plane traﬃc is cluster management traﬃc, whereas data plane traﬃc is application traﬃc. **By default, Docker overlay networks encrypt cluster management traﬃc but not application traﬃc.** You must explicitly enable encryption of application traﬃc.

You can list all networks on each node with the docker network ls command.

```shell
$ docker network ls
```

The newly created network is at the bottom of the list called **uber-net**. The other networks were automatically created when Docker was installed and when the swarm was initialized.

> If you run the docker network ls command on **node2**, you’ll notice that it can’t see the **uber-net** network. This is because new overlay networks are only extended to worker nodes when they are tasked with running a container on it. **This lazy approach to extended overlay networks improves network scalability by reducing the amount of network gossip.**

### Attach a service to the overlay network

Now that you have an overlay network, let’s create a new *Docker service* and attach it to the network. The example will create the service with two replicas (containers) so that one runs on **node1** and the other runs on **node2**. This will automatically extend the **uber-net** overlay to **node2**

Run the following commands from **node1**.

```shell
$ docker service create --name test \
    --network uber-net \
    --replicas 2 \
    ubuntu sleep infinity
```

The command creates a new service called **test**, attaches it to the **uber-net** overlay network, and creates two replicas (containers) based on the image provided. In both examples, you issued a sleep command to the containers to keep them running and stop them from exiting.

Because we’re running two replicas (containers), and the Swarm has two nodes, one replica will be scheduled on each node.

Verify the operation with a docker service ps command.

```shell
$ docker service ps test
```

When Swarm starts a container on an overlay network, it automatically extends that network to the node the container is running on. This means that the **uber-net** network is now visible on **node2**.

Standalone containers that are not part of a swarm service cannot attach to overlay networks unless they have the attachable=true property. The following command can be used to create an attachable overlay network that standalone containers can also attach to.

```shell
$ docker network create -d overlay --attachable uber-net
```

Congratulations. You’ve created a new overlay network spanning two nodes on separate physical underlay networks. You’ve also attached two containers to it. How easy was that!

### Test the overlay network

Let’s test the overlay network with the ping command.

As shown in the Figure below, we’ve got two Docker hosts on separate networks, and a single overlay network spanning both. We’ve got one container connected to the overlay network on each node. Let’s see if they can ping each other.

<img src=".\images\TestDockerOverlayNetwork.png" style="width:75%; height: 75%;">

You can run the test by pinging the remote container by name. However, the examples will use IP addresses as it gives us an excuse to learn how to ﬁnd a containers IP address.

Run a docker network inspect to see the subnet assigned to the overlay and the IP addresses assigned to the two containers in the test service.

```shell
$ docker network inspect uber-net
```

The output is heavily snipped for readability, but you can see it shows **uber-net**’s subnet is 10.0.0.0/24.

This doesn’t match either of the physical underlay networks shown in Figure(172.31.1.0/24 and 192.168.1.0/24). You can also see the IP addresses assigned to the two containers.

Run the following two commands on **node1** and **node2**. These will get the container’s ID’s and conﬁrm the IP address from the previous command. Be sure to use the container ID’s from your own lab in the second command.

```shell
$ docker container ls
```

```shell
$ docker container inspect \
    --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <container-name>

    10.0.0.3
```

Run these commands on both nodes to conﬁrm the IP addresses of both containers.

Figure shows the conﬁguration so far. Subnet and IP addresses may be diﬀerent in your lab.

<img src=".\images\TestDockerOverlayNetwork2.png" style="width:75%; height: 75%;">

As you can see, there is a Layer 2 overlay network spanning both hosts, and each container has an IP address on this overlay network. This means the container on **node1** will be able to ping the container on **node2** using its 10.0.0.4 address. This works despite the fact that both *nodes* are on diﬀerent Layer 2 underlay networks.

Let’s prove it.

Log on to the container on **node1** and ping the remote container.

To do this on the Linux Ubuntu container you’ll need to install the ping utility (**iputils-ping** and **iproute2** packages). If you’re following along with the Windows PowerShell example the ping utility is already installed.

Remember that the container IDs will be diﬀerent in your environment.

```shell
$ docker container exec -it 396c8b142a85 bash

root@396c8b142a85:/# apt-get update && apt-get install iputils-ping -y
```

Congratulations. The container on **node1** can ping the container on **node2** via the overlay network. 

> **If you created the network with the -o encrypted ﬂag, the exchange will have been encrypted.**

You can also trace the route of the ping command from within the container. This will report a single hop, proving that the containers are communicating directly via the overlay network — blissfully unaware of any underlay networks that are being traversed.

> **Note:** You’ll need to install **traceroute** for the Linux example to work.

```shell
$ root@396c8b142a85:/# traceroute 10.0.0.4
```

So far, you’ve created an overlay network with a single command. You then added containers to it. The containers were scheduled on two hosts that were on two diﬀerent Layer 2 underlay networks. Once you worked out the container’s IP addresses, you proved that they could communicate directly via the overlay network.

### The theory of how it all works

Now that you’ve seen how easy it is to build and use a secure overlay network, let’s ﬁnd out how it’s all put together behind the scenes.

Some of the detail in this section will be speciﬁc to Linux. However, the same overall principles apply to Windows.

### VXLAN primer

First and foremost, **Docker overlay networking uses VXLAN tunnels to create virtual Layer 2 overlay networks.**

So, before we go any further, let’s do a quick VXLAN primer.

At the highest level, VXLANs let you create a virtual Layer 2 network on top of an existing Layer 3 infrastructure. 

The example we used earlier created a new 10.0.0.0/24 Layer 2 network on top of a Layer 3 IP network comprising two Layer 2 networks — 172.31.1.0/24 and 192.168.1.0/24. This is shown in the figure below.

<img src=".\images\DockerOverlayWorks.png" style="width:75%; height: 75%;">

The beauty of VXLAN is that it’s an encapsulation technology that existing routers and network infrastructure just see as regular IP/UDP packets and handle without issue.

To create the virtual Layer 2 overlay network, a VXLAN *tunnel* is created through the underlying Layer 3 IP infrastructure. You might hear the term *underlay network* used to refer to the underlying Layer 3 infrastructure — the networks that the Docker hosts are connected to.

Each end of the VXLAN tunnel is terminated by a VXLAN Tunnel Endpoint (VTEP). It’s this VTEP that performs the encapsulation/de-encapsulation and other magic required to make all of this work.

<img src=".\images\DockerOverlayWorks2.png" style="width:75%; height: 75%;">

### Walk through our two-container example

In the example from earlier, you had two hosts connected via an IP network. each host ran a single container, and you created a single VXLAN overlay network for the containers.

To accomplish this, a new *sandbox* (network namespace) was created on each host. As mentioned in the previous chapter, a *sandbox* is like a container, but instead of running an application, it runs an isolated network stack — one that’s sandboxed from the network stack of the host itself.

> A virtual switch (a.k.a. virtual bridge) called **Br0** is created inside the sandbox. A VTEP is also created with one end plumbed into the **Br0** virtual switch, and the other end plumbed into the host network stack (VTEP). The end in the host network stack gets an IP address on the underlay network the host is connected to, and is bound to a UDP socket on port 4789. The two VTEPs on each host create the overlay via a VXLAN tunnel as seen in Figure.

<img src=".\images\DockerOverlayWorks3.png" style="width:75%; height: 75%;">

At this point, the VXLAN overlay is created and ready for use.

Each container then gets its own virtual Ethernet (veth) adapter that is also plumbed into the local **Br0** virtual switch. The topology now looks like the figure given below, and it should be getting easier to see how the two containers can communicate over the VXLAN overlay network despite their hosts being on two separate networks.

<img src=".\images\DockerOverlayWorks4.png" style="width:75%; height: 75%;">

### Communication example

Now that we’ve seen the main plumbing elements, let’s see how the two containers communicate.

For this example, we’ll call the container on node1 “**C1**” and the container on node2 “**C2**”. And let’s assume **C1** wants to ping **C2** like we did in the practical example earlier in the chapter.

<img src=".\images\DockerOverlayWorks5.png" style="width:75%; height: 75%;">

**C1** creates the ping requests and sets the destination IP address to be the 10.0.0.4 address of **C2**. It sends the traﬃc over its veth interface which is connected to the **Br0** virtual switch. The virtual switch doesn’t know where to send the packet as it doesn’t have an entry in its MAC address table (ARP table) that corresponds to the destination IP address. As a result, it ﬂoods the packet to all ports. The VTEP interface is connected to **Br0** knows how to forward the frame, so responds with its own MAC address. This is a *proxy ARP* reply and results in the **Br0** switch *learning* how to forward the packet. As a result, **Br0** updates its ARP table, mapping 10.0.0.4 to the MAC address of the local VTEP.

Now that the **Br0** switch has *learned* how to forward traﬃc to **C2**, all future packets for **C2** will be transmitted directly to the local VTEP interface. The VTEP interface knows about **C2** because all newly started containers have their network details propagated to other nodes in the Swarm using the network’s built-in gossip protocol.

The packet is sent to the VTEP interface, which encapsulates the frames so they can be sent over the underlay transport infrastructure. At a fairly high level, this encapsulation includes adding a VXLAN header to the individual Ethernet frames. The VXLAN header contains the VXLAN network ID (VNID) which is used to map frames from VLANs to VXLANs and vice versa. each VLAN gets mapped to VNID so that the packet can be de-encapsulated on the receiving end and forwarded to the correct VLAN. This maintains network isolation.

The encapsulation also wraps the frame in a UDP packet with the IP address of the remote VTEP on node2 in the *destination IP ﬁeld*, as well as the UDP port 4789 socket information. This encapsulation allows the data to be sent across the underlying networks without the underlying networks having to know anything about VXLAN.

When the packet arrives at node2, the kernel sees that it’s addressed to UDP port 4789. The kernel also knows that it has a VTEP interface bound to this socket. As a result, it sends the packet to the VTEP, which reads the VNID, de-encapsulates the packet, and sends it on to its own local **Br0** switch on the VLAN that corresponds the VNID. From there it is delivered to container C2.

And that is how VXLAN technology is leveraged by native Docker overlay networking. 

Hopefully that’s enough to get you started with any potential production Docker deployments. It should also give you the knowledge required to talk to your networking team about the networking aspects of your Docker infrastructure. 

One ﬁnal thing. Docker also supports Layer 3 routing within the same overlay network. For example, you can create an overlay network with two subnets, and Docker will take care of routing between them. The command to create a network like this could be 

```shell
docker network create --subnet=10.1.1.0/24 --subnet=11.1.1.0/24 -d overlay prod-net
```

This would result in two virtual switches, **Br0** and **Br1**, being created inside the *sandbox*, and routing happens by default.

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

## Deploying apps with Docker Stacks

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

**Version** indicates the version of the Compose ﬁle format. This has to be 3.0 or higher to work with stacks. **Services** is where you deﬁne the stack of services that make up the app. **Networks** lists the required networks, and **secrets** deﬁnes the secrets the app uses. If you expand each top-level key, you’ll see how things map to Figure above. The stack ﬁle has ﬁve services called “reverse_proxy”, “database”, “appserver”, “visualizer”, and “payment_gateway”. The stack ﬁle has three networks called “front-tier”, “back-tier”, and “payment”. So does Figure. Finally, the stack ﬁle has four secrets called “postgres_password”, “staging_token”, “revprox_key”, and “revprox_cert”. It’s important to understand that the stack ﬁle captures and deﬁnes many of the requirements of the entire application. As such, it’s self-documenting and a great tool for bridging the gap between dev and ops. 

**Looking closer at the stack ﬁle**

Stacks ﬁles are very similar to Compose ﬁles. The only requirement is that the version: key specify a value of “3.0” or higher. One of the ﬁrst things Docker does when deploying an app from a stack ﬁle is create any required networks listed under the networks key. If the networks don’t already exist, Docker creates them.

**Networks**

The stack ﬁle describes three networks; front-tier, back-tier, and payment. By default, they’ll all be created as overlay networks by the overlay driver. But the payment network is special — it requires an encrypted data plane. As mentioned in the chapter on overlay networking, the control plane of all overlay networks is encrypted by default, but you have to explicitly encrypt the data plane. The control plane is for network management traffic and the data plane is for application traffic. Encrypting the data plane has a potential performance overhead. To encrypt the data plane, you have two choices:

• Pass the -o encrypted ﬂag to the docker network create command.

• Specify encrypted: 'yes' under driver_opts in the stack ﬁle.

The overhead incurred by encrypting the data plane depends on various factors such traffic type and traffic ﬂow. You should perform extensive testing to understand the performance overhead that encrypting data plane traffic has on your workload. It’s not uncommon for this to be approximately 10%. As previously mentioned, all three networks will be created before the secrets and services.


**Secrets**

Secrets are deﬁned as top-level objects, and the stack ﬁle we’re using deﬁnes four secrets. Notice that all four are deﬁned as external. This means that they must already exist before the stack can be deployed. It’s possible for secrets to be created on-demand when the application is deployed — just replace external: true with file: `<filename>`. However, for this to work, a plaintext ﬁle containing the unencrypted value of the secret must already exist on the host’s ﬁlesystem. This has obvious security implications.

**Services**

Services are where most of the action happens. Each service is a JSON collection (dictionary) that contains a bunch of keys. We’ll step through each one and explain what each of the options does.

**The reverse_proxy service**

As you can see, the reverse_proxy service deﬁnes an image, ports, secrets, and networks. The image key is the only mandatory key in the service object. As the name suggests, it deﬁnes the Docker image that will be used to build the replicas for the service. Remember that a service is one or more identical containers. Docker is opinionated, so unless you specify otherwise, the **image** will be pulled from Docker Hub. You can specify images from 3rd-party registries by prepending the image name with the DNS name of the registry’s API endpoint such as gcr.io for Google’s container registry. One difference between Docker Stacks and Docker Compose is that stacks do not support **builds**. This means all images have to be built prior to deploying the stack. 

The **ports** key deﬁnes two mappings:

• 80:80 maps port 80 across the swarm to port 80 on each service replica.

• 443:443 maps port 443 across the Swarm to port 443 on each service replica.


By default, all ports are mapped using *ingress mode*. This means they’ll be mapped and accessible from every node in the Swarm — even nodes not running a replica. The alternative is *host mode*, where ports are only mapped on swarm nodes running replicas for the service. However, *host mode* requires you to use the long-form syntax. For example, mapping port 80 in *host mode* using the long-form syntax would be like this:

ports:
- target: 80
- published: 80
- mode: host

The long-form syntax is recommended, as it’s easier to read and more powerful (it supports ingress mode **and** host mode). However, it requires at least version 3.2 of the Compose ﬁle format. The **secrets** key deﬁnes two secrets — revprox_cert and revprox_key. These secrets must already exist on the swarm and must also be deﬁned in the top-level secrets section of the stack ﬁle. Secrets get mounted into service replicas as a regular ﬁle. The name of the ﬁle will be whatever you specify as the target value in the stack ﬁle, and the ﬁle will appear in the replica under /run/secrets on Linux, and C:\ProgramData\Docker\secrets on Windows. Linux mounts /run/secrets as an in-memory ﬁlesystem, but Windows does not. The secrets deﬁned in this service will be mounted in each service replica as /run/secrets/revprox_cert and /run/secrets/revprox_key. To mount one of them as /run/secrets/uber_secret you would deﬁne it in the stack ﬁle as follows: 
secrets:

- source: revprox_cert
- target: uber_secret

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

The appserver service uses an image, attaches to three networks, and mounts a secret. It also introduces several additional features under the deploy key. Let’s take a closer look at the new stuff under the deploy key.First up, services.appserver.deploy.replicas = 2 will set the desired number of replicas for the service to 2. If omitted, the default value is 1. If you need to change the number of replicas after you’ve deployed the service, you should do so declaratively. This means updating services.appserver.deploy.replicas ﬁeld in the stack ﬁle with the new value, and then redeploying the stack. We’ll see this later, but re-deploying a stack does not affect services that you haven’t made a change to. services.appserver.deploy.update_config tells Docker how to act when updating the service. For this service, Docker will update two replicas at-a-time (parallelism) and will perform a ‘rollback’ if it detects the update is failing. Rolling back will start new replicas based on the previous deﬁnition of the service. The default value for failure_action is pause, which will stop further replicas being updated. The other option is continue. You specify other options as part of update_config. These include inserting a delay, a failure monitor period, and controlling the order of starting updated replicas before terminating older replicas or vice versa. The services.appserver.deploy.restart-policy object tells Swarm how to restart replicas (containers) if and when they fail. The policy for this service will restart a replica if it stops with a non-zero exit code (condition: on-failure). It will try to restart the failed replica 3 times, and wait up to 120 seconds to decide if the restart worked. It will wait 5 seconds between each of the three restart attempts. 

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

When Docker stops a container, it issues a SIGTERM to the application process with PID 1 inside the container. The application then has a 10-second grace period to perform any clean-up operations. If it doesn’t handle the signal, it will be forcibly terminated after 10 seconds with a SIGKILL. The stop_grace_period property overrides this 10 second grace period. The volumes key is used to mount pre-created volumes and host directories into a service replica. In this case, it’s mounting /var/run/docker.sock from the Docker host, into /var/run/docker.sock inside of each service replica. This means any reads and writes to /var/run/docker.sock in the replica will be passed through to the same directory in the host. /var/run/docker.sock happens to be the IPC socket that the Docker daemon exposes all of its API endpoints on. This means giving a container access to it gives the container the ability to issue commands to the Docker daemon. This has signiﬁcant security implications and is not recommended in the real world. Fortunately, this is just a demo app in a lab environment. The reason this service requires access to the Docker daemon is because it provides a graphical representation of services on the Swarm. To do this, it needs to be able to query the Docker daemon on a manager node. To accomplish this, a placement constraint forces all service replicas onto manager nodes, and the Docker socket is bind-mounted into each service replica.

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

Clone the app’s GitHub repo to your Swarm manager.
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

The docker stack ps command is a good place to start when troubleshooting services that fail to start. It gives an overview of every service in the stack, including which node each replica is scheduled on, current state, desired state, and error message. The following output shows two failed attempts to start a replica for the reverse_proxy service on the wrk-2 node.
```sh
$ docker stack ps seastack
```
For more detailed logs of a particular service you can use the docker service logs command. You pass it either the service name/ID, or replica ID. If you pass it the service name or ID, you’ll get the logs for all service replicas. If you pass it a particular replica ID, you’ll only get the logs for that replica.The following `docker service logs` command shows the logs for all replicas in the seastack_reverse_proxy service that had the two failed replicas in the previous output.
```sh
$ docker service logs seastack_reverse_proxy
```

The output is trimmed to ﬁt the page, but you can see that logs from all three service replicas are shown (the two that failed and the one that’s running). Each line starts with the name of the replica, which includes the service name, replica number, replica ID, and name of host that it’s scheduled on. Following that is the log output.

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

## Runtime and Containers Security

An often overlooked aspect of the application development process is Security. Due to its continuously growing complexity, security in general tends to cause a lot of anxiety with users who become overloaded with an overwhelming amount of additional tasks, to redundantly secure every piece of software developed and deployed. And the list continues with the security requirements of the development tools used, artifacts and images, repositories, client tools and client – repository connections, runtime daemons and engines, etc. Also, the fact that there is no tool to act as a one-stop-shop for all, or most, security aspects, does not help to alleviate this general sense of anxiety.

### Securing the Environment

Prior to securing the artifacts part of the software solution such as container images and containers, users should focus on securing the development environment, which is not just one specific thing, or one single layer of tools. The environment assumes everything from the ground up, whether on prem or cloud: hardware, operating systems, virtualization hypervisors, network fabric, storage services - just to name a few environment aspects.

### Securing the Container Runtime

Once those environmental aspects are secured, now it is time to focus on the tools used during the development process: container runtimes, container engines, command line clients, web and graphical user interfaces, artifact repositories – public and/or private, automation tools, etc.

Running container processes inherit their permissions from the user who is running the container engine – often as a daemon. As a result, one of the security best practices is to ensure the container engine and runtime are not run with root privileges, but run by regular non-root users.

A recently introduced feature in Docker, that quickly matured from experimental phase, allows the Docker Engine to run in a rootless mode to alleviate possible Docker daemon vulnerabilities by running the daemon in an isolated user namespace. This setup depends on a set of prerequisites that help with UID and GID mapping.

Podman, in contrast to Docker, does not rely on a daemon, and it supported both rootless and rooted modes since its early days, the reason it was introduced as a more secure containerization tool than the more popular Docker Engine. However, there are a few features that are not supported in rootless mode, as a few networking and storage related operations require a rooted environment. Podman’s rootless mode is also achieved with UID and GID mapping.

As their popularity grew in the development ecosystem, the container runtimes and containerization engines started receiving continuous critical security feature updates especially as a result of the growing number of security threats and vulnerabilities.

### Securing the Client Access

Another security concern is the client accessing the runtime. With runtimes being available as cloud services, remote hosts, or on-prem, and the client configured for remote or local access method, it is recommended that the client tool to access the runtime in a secure fashion.

Docker allows users to secure access to the Docker daemon socket through multiple methods. A popular method is via SSH, while a more complex method is via TLS over HTTPS. Docker client requests can be secured by default ensuring that every call is verified. Daemon and Client modes also allow for client authentication mechanism configuration, in addition to a secure Docker port configuration option.

### Running Secure Container Images

**Ensuring that we deploy containers from trusted images and image sources is another important security aspect of containerization tools.** 

Tools of the same containerization framework, such as Docker, or Podman and Buildah, are capable of content trust signature verification to ensure only signed images are run as containers by the runtime. 

Although we have access to work with multiple OCI-compatible runtimes and container images can be ported and run across runtimes, **cross-runtime content trust signature verification** is not yet fully supported by all tools, but it is a helpful feature the community desires to see implemented.

### Secure Containers

Container security relies heavily on the integrity of their source images and the security of the runtime engine. 

Runtime engines running in rootless mode ensure that the container processes get started and are run in isolated user namespaces. However, user remapping and container image content signatures are not fully responsible for container security.

Some users are debating the need to run containers in rooted vs. non-rooted mode, however, it is a clear security risk to allow a container process to run in rooted mode giving it full access over host resources. While running containers in rooted mode may be more of a routine that is part of the development process, it can be addressed with capabilities. A complex method indeed, capabilities replace the need for full root access of a container process with a controlled and limited root access that can be managed and set at a very granular level. The benefit of capabilities is that the container process will only receive the needed amount of privileges over the resources of the host that are critical for the container’s operations. All other resources will be accessed as a regular user with unprivileged restricted permissions.

### Security Tools

As the security features of the container runtimes and tools are far from being perfect, users rely on additional security tools when concerned about the security of their containerized applications.

Antivirus software can be used to scan container images, container filesystems and storage volumes, but it may adversely impact the performance of the containers during the scanning process.

Container processes may be secured by assigning AppArmor profiles to them. AppArmor, or Application Armor) is a popular Linux operating system security module used to protect system and user processes against security threats.

Another layer of container process security can be protected with **Seccomp**. **Secure computing is a Linux kernel feature designed to restrict the access of a container process over host resources.** Seccomp profiles can allow or block system calls that may modify kernel modules, kernel memory, kernel I/O, modify namespaces, etc. Both Seccomp and capabilities are equally complex and granular approaches, and a significant amount of functional overlap can be observed.

## Security in Docker

Good security is all about layers, and Docker has lots of layers. It supports all the major Linux security technologies as well as plenty of its own. And many of them are simple and easy to conﬁgure. In this chapter, we’ll look at some of the technologies that can make running containers on Docker very secure.

Security is all about layers. Generally speaking, the more layers of security the more secure something is. Docker offers a lot of security layers. Figure below shows some of the security-related technologies we’ll cover in the chapter.

<img src=".\images\Security.png" style="width:75%; height: 75%;">

Docker on Linux leverages most of the common Linux security and workload isolation technologies. These include **namespaces**, **control groups (cgroups)**, **capabilities**, **mandatory access control (MAC) systems**, and **seccomp**. For each one, **Docker implements sensible defaults for a seamless and *moderately secure* out-of-the-box experience**. However, you can customize each one to your own speciﬁc requirements. Docker itself adds some excellent additional security technologies.

**Docker Swarm Mode** is secure by default. You get all of the following with zero conﬁguration required; cryptographic node IDs, mutual authentication, automatic CA conﬁguration, automatic certiﬁcate rotation, encrypted cluster store, encrypted networks, and more.

**Docker Content Trust (DCT)** lets you sign your images and verify the integrity and publisher of images you consume.

**Image security scanning** analyses images, detects known vulnerabilities, and provides detailed reports.

**Docker secrets** are a way to securely share sensitive data and are ﬁrst-class objects in Docker. They’re stored in the encrypted cluster store, encrypted in-ﬂight when delivered to containers, stored in in-memory ﬁlesystems when in use, and operate a least-privilege model. 

There’s a lot more, but the important thing to know is that Docker works with the major Linux security technologies as well as providing its own extensive and growing set of security technologies. While the Linux security technologies tend to be complex, the native Docker security technologies tend to be simple.

We all know that security is important. We also know that security can be complicated and boring. When Docker decided to bake security into the platform, it decided to make it simple and easy. They knew that if security was hard to conﬁgure, people wouldn’t use it. As a result, most of the security technologies offered by the Docker platform are simple to use. They also ship with sensible defaults — meaning you get a *fairly secure* platform at zero effort. Of course, the defaults aren’t perfect, but they’re enough to serve as a safe starting point. From there you should customize them to your requirements.

- **Linux security technologies**
    - Namespaces
    - Control Groups
    - Capabilities
    - Mandatory Access Control
    - seccomp

- **Docker platform security technologies**
    - Swarm Mode
    - Image Scanning
    - Docker Content Trust
    - Docker Secrets

### Linux security technologies

All *good* container platforms use *namespaces* and *cgroups* to build containers. The *best* container platforms will also integrate with other Linux security technologies such as *capabilities*, *Mandatory Access Control systems* like SELinux and AppArmor, and *seccomp*. As expected, Docker integrates with them all.

#### Namespaces

Kernel namespaces are at the very heart of containers. They slice up an operating system (OS) so that it looks and feels like multiple **isolated** operating systems. 

This lets us do really cool things like run multiple web servers on the same OS without having port conﬂicts. It also lets us run multiple apps on the same OS without them ﬁghting over shared conﬁg ﬁles and shared libraries.

A couple of quite examples:

- Namespaces let you run multiple web servers, each on port 443, on a single OS. To do this you just run each web server app inside of its own **network namespace**. This works because each *network namespace* gets its own IP address and full range of ports. You may have to map each one to a separate port on the Docker host, but each can run without being re-written or reconﬁgured to use a different port.

- You can run multiple applications, each requiring their own version of a shared library or conﬁguration ﬁle. To do this you run each application inside of its own **mount namespace**. **This works because each *mount namespace* can have its own isolated copy of any directory on the system (e.g. /etc, /var, /dev etc.)**

Figure shows a high-level example of two web server applications running on a single host and both using port 443. Each web server app is running inside of its own network namespace.

<img src=".\images\Namespaces.png" style="width:75%; height: 75%;">

Working directly with namespaces is hard. Fortunately, Docker hides this complexity and manages all of the namespaces required to build a useful container. 

Docker on Linux currently utilizes the following kernel namespaces:
- Process ID (pid)
- Network (net)
- Filesystem/mount (mnt)
- Inter-process Communication (ipc)
- User (user)
- UNIX Time-Sharing (uts)

The most important thing to understand is that **Docker containers are an organized collection of namespaces**. This means that you get all of this OS isolation for free with every container. 

**For example, every container has its own pid, net, mnt, ipc, uts, and potentially user namespace. In fact, an organized collection of these namespaces is what we call a “container”.** 

Figure shows a single Linux host running two containers.

<img src=".\images\NamespacesDocker.png" style="width:75%; height: 75%;">

How Docker uses each namespace:

- **Process ID namespace**: Docker uses the pid namespace to provide isolated process trees for each container. **This means every container gets its own PID 1. PID namespaces also mean that one container cannot see or access to the process tree of other containers. Nor can it see or access the process tree of the host it’s running on.**

- **Network namespace**: Docker uses the net namespace to provide each container its own isolated network stack. **This stack includes; interfaces, IP addresses, port ranges, and routing tables.** For example, every container gets its own eth0 interface with its own unique IP and range of ports. 

- **Mount namespace**: Every container gets its own unique isolated root (/) ﬁlesystem. **This means every container can have its own /etc, /var, /dev and other important ﬁlesystem constructs. Processes inside of a container cannot access the mount namespace of the Linux host or other containers — they can only see and access their own isolated ﬁlesystem.**

- **Inter-process Communication namespace**: Docker uses the ipc namespace for shared memory access within a container. It also isolates the container from shared memory outside of the container.

- **User namespace**: **Docker lets you use user namespaces to map users inside of a container to different users on the Linux host.** A common example is mapping a container’s root user to a non-root user on the Linux host.

- **UTS (UNIX Time-Sharing) namespace**: UTS (UNIX Time-Sharing) namespaces allow a single system to appear to have different host and domain names to different processes. Docker uses the uts namespace to provide each container with its own hostname.

#### Control Groups

If namespaces are about isolation, *control groups (cgroups)* are about settings limits. 

**Cgroups let us set limits so that no single container can use all resources. Containers are isolated from each other but all share a common set of OS resources — things like CPU, RAM, network bandwidth, and disk I/O.** Cgroups let us set limits on each of these so a single container cannot consume everything and cause a denial of service (DoS) attack.

#### Capabilities

**It’s a bad idea to run containers as root — root is all-powerful and therefore very dangerous. But, it can be challenging running containers as unprivileged non-root users.** 

For example, on most Linux systems, non-root users tend to be so powerless they’re practically useless. What’s needed, is a technology that lets us pick and choose which root powers a container needs in order to run.

**Under the hood, the Linux root user is a combination of a long list of *capabilities*.** 

Some of these *capabilities* include:
- CAP_CHOWN: lets you change ﬁle ownership
- CAP_NET_BIND_SERVICE: lets you bind a socket to low numbered network ports
- CAP_SETUID: lets you elevate the privilege level of a process
- CAP_SYS_BOOT: lets you reboot the system.

The list goes on and is long. Docker works with *capabilities* so that you can run containers as root, but strip out all the capabilities you don’t need. 

For example, if the only root privilege your container needs is the ability to bind to low numbered network ports, you should start a container and drop all root capabilities, then add back just the CAP_NET_BIND_SERVICE capability. This is an excellent example of implementing *least privilege* — you get a container running with only the capabilities required. 

**Docker also imposes restrictions so that containers cannot re-add the dropped capabilities.** 

#### Mandatory Access Control systems

Docker works with major Linux MAC technologies such as AppArmor and SELinux.

Depending on your Linux distribution, Docker applies a default AppArmor proﬁle to all new containers. According to the Docker documentation, this default proﬁle is “moderately protective while providing wide application compatibility”. 

Docker also lets you start containers without a policy applied, as well as giving you the ability to customize policies to meet speciﬁc requirements. This is also very powerful, but can also be prohibitively complex.

#### seccomp

**Docker uses seccomp, in ﬁlter mode, to limit the syscalls a container can make to the host’s kernel.**

As per the Docker security philosophy, all new containers get a default seccomp proﬁle conﬁgured with sensible defaults. This is intended to provide moderate security without impacting application compatibility. 

As always, you can customize seccomp proﬁles, and you can pass a ﬂag to Docker so that containers can be started without a seccomp proﬁle. As with many of the technologies already mentioned, seccomp is extremely powerful. However, the Linux syscall table is long, and conﬁguring the appropriate seccomp policies can be prohibitively complex.

#### Final thoughts on the Linux security technologies

Docker supports most of the important Linux security technologies and ships with sensible defaults that add security but aren’t too restrictive. Figure below shows how these technologies form multiple layers of potential security.

<img src=".\images\LinuxSecurity.png" style="width:75%; height: 75%;">

Some of these technologies can be complicated to customize as they require deep knowledge of how the Linux kernel works. Hopefully they will get simpler to conﬁgure in the future, but for now, the default conﬁgurations that ship with Docker might be a good place to start.

### Docker platform security technologies

#### Security in Swarm Mode

Docker Swarm allows you to cluster multiple Docker hosts and deploy applications declaratively. Every Swarm is comprised of *managers* and *workers* that can be Linux or Windows. Managers host the control plane of the cluster and are responsible for conﬁguring the cluster and dispatching work tasks. Workers are the nodes that run your application code as containers. As expected, *swarm mode* includes many security features that are enabled out-of-the-box with sensible defaults.

These include:
- Cryptographic node IDs
- TLS for mutual authentication
- Secure join tokens
- CA conﬁguration with automatic certiﬁcate rotation
- Encrypted cluster store (conﬁg DB)
- Encrypted networks

Let’s walk through the process of building a secure swarm and conﬁguring some of the security aspects. To follow along with the complete set of examples you’ll need at least three Docker hosts running Docker 17.03 or higher. The examples cited use three Docker hosts called “mgr1”, “mgr2”, and “wrk1”. Each one is running Docker 19.03.4. There is network connectivity between all three hosts, and all three can ping each other by name.

**Conﬁgure a secure Swarm**

Run the following command from the node you want to be the ﬁrst manager in the new swarm. In the example, we’ll run it from **mgr1**.

```shell
$ docker swarm init
```

That's it! That is literally all you need to do to conﬁgure a secure swarm. **mgr1** is conﬁgured as the ﬁrst manager of the swarm and also as the root certiﬁcate authority (CA). The swarm  itself has been given a cryptographic clusterID. **mgr1** has issued itself with a client certiﬁcate that identiﬁes it as a manager in the swarm, certiﬁcate rotation has been conﬁgured with the default value of 90 days, and a cluster conﬁg database has been conﬁgured and encrypted. A set of secure tokens have also been created so that new managers and new workers can be joined to the swarm. And all of this with a **single command!**

Let’s join **mgr2** as an additional manager. Joining new managers to a swarm is a two-step process. The ﬁrst step extracts the required token. The second step runs the docker swarm join command on the node you wish to add. As long as you include the manager join token as part of the command, **mgr2** will join the swarm as a manager. Run the following command from **mgr1** to extract the manager join token.

```shell
$ docker swarm join-token manager
```

The output gives you the exact command you need to run on nodes that you want to join as managers. The join token and IP address will be different in your lab.

The format of the join command is:

```shell
$ docker swarm join --token {manager-join-token} {ip-of-existing-manager}:{swarm-port}
```

The format of the token is:

```shell
SWMTKN-1-{hash-of-cluster-certificate}-{manager-join-token}
```

Copy the command and run it on “mgr2”:

```shell
$ docker swarm join --token SWMTKN-1-1dmtwu...r17stb-2axi5...8p7glz 172.31.5.251:2377
```

This node joined a swarm as a manager. **mgr2** has joined the swarm as an additional manager. In production clusters you should always run either 3 or 5 managers for high availability. Verify **mgr2** was successfully added by running a docker node ls on either of the two managers.

```sh
$ docker node ls
```

The output shows that **mgr1** and **mgr2** are both part of the swarm and are both managers. Two managers is possibly the worst number possible. Adding a swarm worker is a similar two-step process. Step 1 extracts the join token, and step 2 is to run a docker swarm join command on the node you want to join as a worker. 

Run the following command on either of the managers to expose the worker join token.

```shell
$ docker swarm join-token worker
```

Again, you get the exact command you need to run on nodes you want to join as workers. The join token and IP address will be different in your lab. 

Copy the command and run it on **wrk1** as shown:

```shell
$ docker swarm join --token SWMTKN-1-1dmtw...17stb-ehp8g...w738q 172.31.5.251:2377
```

This node joined a swarm as a worker. Run another docker node ls command from either of the swarm managers.

```shell
$ docker node ls
```

You now have a swarm with two managers and one worker. The managers are conﬁgured for high availability (HA) and the cluster store is replicated to both. The ﬁnal conﬁguration is shown in Figure .

<img src=".\images\SwarmSecurity.png" style="width:75%; height: 75%;">

**Looking behind the scenes at Swarm security**

Now that we’ve built a secure Swarm let’s take a minute to look behind the scenes at some of the security technologies involved.

**Swarm join tokens**

**The only thing that is needed to join new managers and workers to an existing swarm is the relevant join token. For this reason, it’s vital that you keep your join-tokens safe.** Do not post them on public GitHub repos or even internal source code repos that are not restricted. 

Every swarm maintains two distinct join tokens:
- One for joining new managers
- One for joining new workers

It’s worth understanding the format of the Swarm join token. 

Every join token is comprised of 4 distinct ﬁelds separated by dashes (-):

```shell
PREFIX - VERSION - SWARM ID - TOKEN
```

**The preﬁx** is always SWMTKN. This allows you to pattern-match against it and prevent people from accidentally posting it publicly. 

**The VERSION** ﬁeld indicates the version of the swarm. 

**The Swarm ID** ﬁeld is a hash of the swarm’s certiﬁcate. 

**The TOKEN** ﬁeld is the part that determines whether it can join nodes as managers or workers. 

As the following shows, the manager and worker join tokens for a given Swarm are identical except for the ﬁnal TOKEN ﬁeld.

```
MANAGER: SWMTKN-1-1dmtwusdc...r17stb-2axi53zjbs45lqxykaw8p7glz

WORKER: SWMTKN-1-1dmtwusdc...r17stb-ehp8gltji64jbl45zl6hw738q
```

**If you suspect that either of your join tokens has been compromised, you can revoke them and issue new ones with a single command.**

The following example revokes the existing *manager* join token and issues a new one.

```shell
$ docker swarm join-token --rotate manager
```

Successfully rotated manager join token. 

To add a manager to this swarm, run the following command:

```shell
docker swarm join --token SWMTKN-1-1dmtwu...r17stb-1i7txlh6k3hb921z3yjtcjrc7 172.31.5.251:2377
```

Existing managers do not need updating, however, you’ll need to use the new token to add new managers. Notice that the only difference between the old and new join tokens is the last ﬁeld. The hash of the Swarm ID remains the same. Join tokens are stored in the cluster store which is encrypted by default.

**TLS and mutual authentication**

Every manager and worker that joins a swarm is issued a client certiﬁcate. This certiﬁcate is used for mutual authentication. It identiﬁes the node, the swarm that it’s a member of, and role the node performs in the swarm (manager or worker). 

You can inspect a node’s client certiﬁcate on Linux nodes with the following command.

```shell
$ sudo openssl x509 -in /var/lib/docker/swarm/certificates/swarm-node.crt -text
```

**The Subject data in the output uses the standard O, OU, and CN ﬁelds to specify the Swarm ID, the node’s role, and the node ID.**
- The Organization (O) ﬁeld stores the Swarm ID
- The Organizational Unit (OU) ﬁeld stores the node’s role in the swarm
- The Canonical Name (CN) ﬁeld stores the node’s crypto ID.

<img src=".\images\SecurityTLS.png" style="width:75%; height: 75%;">

You can also see the certiﬁcate rotation period in the Validity section. You can match these values to the corresponding values shown in the output of a docker system info command.

```shell
$ docker system info
```

**Conﬁguring some CA settings**

You can conﬁgure the certiﬁcate rotation period for the Swarm with the docker swarm update command. The following example changes the certiﬁcate rotation period to 30 days.

```shell
$ docker swarm update --cert-expiry 720h
```

Swarm allows nodes to renew certiﬁcates early (slightly before they expire) so that not all nodes don’t try and update their certiﬁcates at the same time. You can conﬁgure an external CA when creating a new swarm by passing the --external-ca ﬂag to the docker swarm init command. The new docker swarm ca sub-command can be used to manage CA related conﬁguration. Run the command with the --help ﬂag to see a list of things it can do.

```shell
$ docker swarm ca --help
```

**The cluster store**

The cluster store is the brains of a swarm and is where cluster conﬁg and state are stored. It’s also critical to other Docker technologies such as overlay networking and Secrets. This is why swarm mode is required for so many advanced and security related Docker features. If you’re not running in swarm mode, there’ll be a bunch of Docker technologies and security features you won’t be able to use. The store is currently based on the popular etcd distributed database and is automatically conﬁgured to replicate itself to all managers in the swarm. It is also encrypted by default. Day-to-day maintenance of the cluster store is taken care of automatically by Docker. However, in production environments, you should have strong backup and recovery solutions in place for it. 

#### Detecting vulnerabilities with image security scanning

Image scanning is your primary weapon against vulnerabilities and security holes in your images. Image scanners work by inspecting images and searching for packages that have known vulnerabilities. Once you know about these, you can update the packages and dependencies to versions with ﬁxes. 

As good as image scanning is, it’s important to understand its limitations. For example, **image scanning is focussed on images and does not detect security problems with networks, nodes, or orchestrators.** Also, not all image scanners are equal — some perform deep binary-level scanning to detect packages, whereas others simply look at package names and do not closely inspect the content of images. 

Some on-premises private registry solutions offer built-in scanning, and there are third-party services that offer image scanning services. Figures are included as an example of the kind of reports image scanners can provide.

<img src=".\images\ScanImage.png" style="width:75%; height: 75%;">

<img src=".\images\ScanImage2.png" style="width:75%; height: 75%;">

In summary, image security scanning can be a great tool for deeply inspecting your images for known vulnerabilities. Beware though, with great knowledge comes great responsibility — once you become aware of vulnerabilities you are responsible for mitigating or ﬁxing them.

#### Signing and verifying images with Docker Content Trust

Docker Content Trust (DCT) makes it simple and easy to verify the integrity and the publisher of images that you download and run. This is especially important when pulling images over untrusted networks such as the internet. At a high level, DCT allows developers to sign images when they are pushed to Docker Hub or other container registries. These images can then be veriﬁed when they are pulled and ran. This high-level process is shown in Figure below.

<img src=".\images\DockerContentTrust.png" style="width:75%; height: 75%;">

DCT can also be used to provide important *context*. This includes; whether or not an image has been signed for use in a particular environment such as “prod” or “dev”, or whether an image has been superseded by a newer version and is therefore stale. The following steps will walk you through conﬁguring Docker Content Trust, signing and pushing an image, and then pulling the signed image. To follow along, you’ll need a cryptographic key-pair to sign images. If you don’t already have a key-pair, you can use the docker trust sub-command to generate a new key-pair one. 

The following command generates a new key-pair called “nigel”.

```shell
$ docker trust key generate nigel
```

If you already have a key-pair, you can import and load it with 

```shell
docker trust key load key.pem --name nigel
```

Now that you’ve loaded a valid key-pair, you need to associate it with the image repository you’ll be pushing signed images to. This example uses the nigelpoulton/dct repo on Docker Hub and the nigel.pub key that was created by the previous docker trust key generate command. Your key ﬁle will be different.

```shell
$ docker trust signer add --key nigel.pub nigel nigelpoulton/dct
```

The following command will sign the nigelpoulton/dct:signed image and push it to Docker Hub.

```shell
$ docker trust sign nigelpoulton/dct:signed
```

Once the image is pushed, you can inspect its signing data with the following command.

```shell
$ docker trust inspect nigelpoulton/dct:signed --pretty
```

You can force a Docker host to always sign and verify image push and pull operations by exporting the **DOCKER_CONTENT_TRUST** environment variable with a value of **1**. In the real world, you’ll want to make this a more permanent feature of Docker hosts.

```shell
$ export DOCKER_CONTENT_TRUST=1
```

**Once DCT is enabled, you’ll no longer be able to pull and work with unsigned images.** 

You can test this behavior by attempting to pull the following two images:

- nigelpoulton/dct:unsigned
- nigelpoulton/dct:signed

If you have enabled DCT by settings the DOCKER_CONTENT_TRUST environment variable, you will not be able to pull the dct:unsigned image. However, you will be able to pull the image tagged as signed.

```shell
$ docker image pull nigelpoulton/dct:unsigned
```

Docker Content Trust is an important technology for helping you verify the images you are pulling from container registries. It’s simple to conﬁgure in its basic form, but more advanced features, such as *context*, can be more complex to conﬁgure.

#### Docker Secrets

Many applications need secrets — things like passwords, TLS certiﬁcates, SSH keys, and more. 

Early versions of Docker had no standardised way of making secrets available to apps in a secure way. It was common for developers to insert secrets into apps via plain text environment variables (we’ve all done it). This was far from ideal. Docker 1.13 introduced **Docker Secrets** as ﬁrst-class objects in the Docker API. 

**Behind the scenes, secrets are encrypted at rest, encrypted in-ﬂight, mounted in containers to in-memory ﬁlesystems, and operate under a least-privilege model where they are only made available to services that have been explicitly granted access to them.** It’s quite a comprehensive end-to-end solution, and it even has its own docker secret sub-command.

<img src=".\images\DockerSecret.png" style="width:75%; height: 75%;">

The following steps walk through the high-level workﬂow shown in the figure above.

1. The blue secret is created and posted to the Swarm.

2. It gets stored in the encrypted cluster store (all managers have access to the cluster store).

3. The blue service is created and the secret is attached to it.

4. The secret is encrypted in-ﬂight while it is delivered to the tasks (containers) in the blue service.

5. The secret is mounted into the containers of the blue service as an unencrypted ﬁle at /run/secrets/. This is an in-memory tmpfs ﬁlesystem (this step is different on Windows Docker hosts as they do not have the notion of an in-memory ﬁlesystem like tmpfs).

6. Once the container (service task) completes, the in-memory ﬁlesystem is torn down and the secret ﬂushed from the node.

7. The red containers in the red service cannot access the secret.

The reason that secrets are surfaced in their un-encrypted form in running containers is so applications can use them without requiring methods to decrypt them. 

You can create and manage secrets with the docker secret sub-command, and you can attach them to services by specifying the --secret ﬂag to the `docker service create` command.
