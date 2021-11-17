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
