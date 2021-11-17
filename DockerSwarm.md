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
