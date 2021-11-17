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

The CNM provides portability to applications across diverse infrastructures.

<img src=".\images\docker-container-network-model.png" style="width:75%; height: 75%;">

#### Libnetwork

The CNM is the design doc, and libnetwork is the canonical implementation. It’s open-source, written in Go, cross-platform (Linux and Windows), and used by Docker.

In the early days of Docker, all the networking code existed inside the daemon. This was a nightmare — the daemon became bloated, and it didn’t follow the Unix principle of building modular tools that can work on their own, but also be easily composed into other projects. As a result, it all got ripped out and refactored into an external library called libnetwork based on the principles of the CNM. Nowadays, all of the core Docker networking code lives in libnetwork.

As you’d expect, it implements all three of the components deﬁned in the CNM. It also implements native *service discovery*, *ingress-based container load balancing*, and the network control plane and management plane functionality.

Interaction between Docker Engine, CNM, and Network Drivers:

<img src=".\images\docker-network-infrastructure.png" style="width:75%; height: 75%;">

#### Docker Networking Drivers

If libnetwork implements the control plane and management plane functions, then drivers implement the data plane. For example, connectivity and isolation is all handled by drivers. So is the actual creation of networks. The relationship is shown in Figure.

<img src=".\images\DockerNetwork5.png" style="width:75%; height: 75%;"/>

<img src=".\images\docker-cnm-drivers.png" style="width:75%; height: 75%;">

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

A Bridge is a default Docker network that is present on any Linux host which runs a Docker Engine.

Understanding correlated terms:
- A bridge is a Docker network
- A bridge is also a Docker network driver/template, which creates a bridge network
- docker0 is the kernel building block that is used in implementing the bridge network

The default network type that Docker containers attach to is the bridge network, implemented by the bridge driver. The main roles of a bridge network are to isolate the container network from the host system network, and to act as a DHCP server and assign unique IP addresses to containers as they are running attached to the bridge. This is a useful feature when containers of an application need to talk to each other while they all run on the same host system. Other features of bridge networks depend whether the bridge is default or user-defined. Most often, a user-defined bridge presents advantages over the default bridge, such as better traffic isolation, automatic DNS resolution, on-the-fly container connection/disconnection to and from the network, advanced and flexible configuration options, and even sharing of environment variables between containers.

**Bridge Network**

<img src=".\images\docker-bridge-network.png" style="width:75%; height: 75%;"/>

**User-Defined Bridge Network**

<img src=".\images\docker-user-defined-bridge-network.png" style="width:75%; height: 75%;"/>

**Bridge Network Comparison**

| Features | Default | User-Defined |
| :--- | :---: | :---: |
| Better isolation and interoperability between containerized applications | No | Yes |
|Automatic DNS resolution between containers | No | Yes |
|Attachment and detachment of containers on the fly | No | Yes |
|Configurable bridge creation | No | Yes |
|Linked containers share environment variables | Yes | No |

**Bridge Network Driver: Use Case**

The discovery of service is done automatically by the Docker bridge because they are on the same network.

<img src=".\images\docker-bridge-network-use-case.png" style="width:75%; height: 75%;"/>

**Host**

The host network driver option, as opposed to the bridge, eliminates the network isolation between the container and the host system by allowing the container to directly access the host network. In addition, it may help with performance optimization by eliminating the need for Network Address Translation (NAT) or proxying since the container ports are automatically published and available as host ports.

While using a host network, the container shares the host’s networking namespace, and the container is
not allocated its own IP address.

Advantages:

- Optimizes the performance.
- Handles a large range of ports.
- Does not require network address translation (NAT).
- Does not require “userland-proxy” for each port.

--network host: This is passed with command docker service create to use a host network for a swarm service.

Features:

- An overlay network is used to manage swarm and service-related traffic.
- The Docker daemon host network and ports are used to send data for individual swarm service.

> NOTE: The host networking driver only works on Linux hosts.

<img src=".\images\docker-host-network.png" style="width:75%; height: 75%;"/>

**Overlay**

Gaining a lot of popularity these days is the overlay network type supported by the overlay driver. It is the network type that spans multiple hosts, typically part of a cluster, allowing the containers' traffic to be routed between hosts as containers or services from one host attempt to talk to others running on another host in the cluster.

This network type is very popular with container orchestration platforms because it not only enables traffic across the entire cluster of host systems, but also provides traffic isolation and management through rules and policies.

Open ports:

- Open TCP port 2377 for cluster management communications
- Open TCP and UDP port 7946 for communication among nodes
- Open UDP port 4789 for overlay network traffic

Initialize Docker daemon as a swarm manager using docker swarm init, or join the Docker daemon to an existing swarm using docker swarm join, before creating an overlay network.

Overlay network driver: It creates a distributed network among multiple Docker daemon hosts.

<img src=".\images\docker-overlay-network-overview.png" style="width:75%; height: 75%;"/>

Provisioning for an overlay network is automated by Docker Swarm control plane.

<img src=".\images\docker-overlay-network.png" style="width:75%; height: 75%;"/>

**Overlay Network Driver: Use Case**

<img src=".\images\docker-overlay-network-use-case.png" style="width:75%; height: 75%;"/>

**Egress and Ingress**

<img src=".\images\docker-swarm-overlay-network.png" style="width:75%; height: 75%;"/>

**Egress** in the world of networking implies traffic that exits an entity or a network boundary, while **Ingress** is traffic that enters the boundary of a network. 

While in service provider types of the network this is pretty clear, in the case of datacenter or cloud it is slightly different. In the cloud, Egress still means traffic that’s leaving from inside the private network out to the public internet, but Ingress means something slightly different. To be clear private networks here refers to resources inside the network boundary of a data center or cloud environment and its IP space is completely under the control of an entity who operates it.

Since traffic often is translated using NAT in and out of a private network like the cloud, a response back from a public endpoint to a request that was initiated inside the private network is not considered Ingress. If a request is made from the private network out to a public IP, the public server/endpoint responds back to that request using a port number that was defined in the request, and firewall allows that connection since its aware of an initiated session based on that port number. See picture below for reference.

<img src=".\images\egress-ingress-image-1.png" style="width:75%; height: 75%;"/>

With Egress out of the way, let’s define Ingress. As you might be guessing by now, Ingress refers to unsolicited traffic sent from an address in public internet to the private network – it is not a response to a request initiated by an inside system. In this case, firewalls are designed to decline this request unless there are specific policy and configuration that allows ingress connections. See picture below for reference.

<img src=".\images\egress-ingress-image-2.png" style="width:75%; height: 75%;"/>

**Macvlan**

The macvlan network driver allows a user to change the appearance of a container on the physical network. A container may appear as a physical device with its own MAC address on the network, thus enabling the container to be directly connected to the physical network instead of having its traffic routed through the host network.

Macvlan network is used to assign MAC address to the virtual network interface of containers. This helps the legacy applications to directly connect to the physical network.

Precautionary measures:

- Cut down the large number of unique MAC address to save the network from damage.
- Handle “promiscuous mode” via networking equipment to assign multiple MAC address to single
physical interface.

<img src=".\images\docker-macvlan-network.png" style="width:75%; height: 75%;"/>

Positive performance implications:

- MACVLAN has simple and lightweight architecture.
- MACVLAN drivers provide direct access between physical network and containers.
- MACVLAN containers receive routable IP addresses that are present on the subnet of the physical network.

Use cases of MACVLAN include:

- Low-latency applications.
- Network design which needs containers to be on the same subnet and use IPs as the external host network.

**MACVLAN Network Driver: Use Case**

<img src=".\images\docker-macvlan-network-use-case.png" style="width:75%; height: 75%;"/>

**None**

The none driver option for container networking disables the networking of a container while allowing the very same container to use a custom third-party network driver, if needed, to implement its networking requirements.

None provides the functionality of disabling networking.

Form a container with none network:

```shell
$ docker run --rm -dit \
--network none \
--name no-net-alpine \
alpine:latest \
ash
```

Using **'--network none'** will result in a container with no eth0.

**Network Plugins**

Network plugins expand the capabilities of Docker through third-party network drivers that integrate Docker with specialized network stacks. Certain network plugins may combine features otherwise available from single network drivers while improving network resiliency and policy management.

3rd-parties can also write Docker network drivers known as **remote drivers** or **plugins**. Weave Net is a popular example and can be downloaded from Docker Hub.

Each driver is in charge of the actual creation and management of all resources on the networks it is responsible for. For example, an overlay network called “prod-fe-cuda” will be owned and managed by the overlay driver. This means the overlay driver will be invoked for the creation, management, and deletion of all resources on that network.

In order to meet the demands of complex highly-ﬂuid environments, libnetwork allows multiple network drivers to be active at the same time. This means your Docker environment can sport a wide range of heterogeneous networks.

**Prune Networks**

Docker networks don’t take up much disk space, but they do create iptables rules, bridge network devices, and routing table entries.

The user can use the following command to clean up networks which aren’t used by any containers:

```shell
$ docker network prune
```

### Networking from Container Point of View

<img src=".\images\docker-network-container-view.png" style="width:75%; height: 75%;"/>

**Published ports**

Making a port available using --publish or -p will create a firewall rule that map a container port to the port present on a Docker host. 

Examples are provided in the following table:

| Flag value | Description |
| :--- | :--- |
| -p 8080:80 | TCP port 80 in the container is mapped to port 8080 on the Docker host. |
| -p 192.168.1.100:8080:80 | TCP port 80 in the container is mapped to port 8080 on the Docker host for connections to host IP 192.168.1.100. |
| -p 8080:80/udp | UDP port 80 in the container is mapped to port 8080 on the Docker host. |
| -p 8080:80/tcp -p 8080:80/udp | TCP port 80 in the container is mapped to TCP port 8080 on the Docker host and UDP port 80 in the container is mapped to UDP port 8080 on the Docker host. |

**IP address**

- An IP address is assigned to a container for every Docker network that it connects to.
- **--network** is used to connect a container to a single network.
- The **docker network connect** is used to connect a running container to multiple networks.
- The IP address can be specified -ipwhile connecting the container to a network by using **--ip** or **--ip6** flags.

**Hostname**

- Container ID in the Docker is the default hostname of the container.
- A hostname is overridden by using **--hostname**.
- Additional network alias is specified by using **--alias** flag for the container on an existing network.

**DNS services**

A container inherits the DNS settings of the Docker daemon, including the **/etc/hosts** and **/etc/resolv.conf**.

| Flag | Description |
| :--- | :--- |
| --dns | IP address of a DNS server. Multiple --dns flags are used to specify multiple DNS servers. |
| --dns-search | Searches non-fully-qualified hostnames Multiple --dns-search flags are used to specify multiple DNS search prefixes. |
| --dns-opt | Represents a DNS option and its value. |
| --hostname | Hostname of a container. |

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

### Docker overlay networking

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
