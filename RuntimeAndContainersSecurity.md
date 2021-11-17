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
