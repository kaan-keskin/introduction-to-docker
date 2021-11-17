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
