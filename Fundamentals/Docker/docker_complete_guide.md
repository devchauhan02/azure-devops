# 🐳 Docker — The Complete Beginner's Guide
### *"From Zero to Containers — With Real Life, Real Questions & Real Scenarios"*

---

> **How to read this guide:**
> Every concept explains:
> - 🔴 **The Problem** — what pain existed before
> - 🟡 **What was used before** — older solutions & their limits
> - 🟢 **How Docker solves it** — the actual fix
> - 💡 **Real-Life Scenario** — something you can relate to
> - ❓ **Cross Questions** — questions a curious beginner would ask
> - 🔗 **Related Concepts** — how it connects to other tech ideas

---

## 📋 Table of Contents

1. [Docker Overview — The Big Picture](#1-docker-overview)
2. [Installing Docker Desktop or Docker Engine](#2-installing-docker)
3. [Virtual Machines Architecture — Deep Dive](#3-virtual-machines-architecture)
4. [Docker Containers Architecture — How it Really Works](#4-docker-containers-architecture)
5. [How Docker Runs on Different Operating Systems](#5-docker-on-different-os)
6. [Container Processes and Resources](#6-container-processes-and-resources)
7. [Docker Components — Client, Server, Host, Registry](#7-docker-components)
8. [Docker Commands vs Management Commands](#8-docker-commands)
9. [Basic Container and Image Commands](#9-basic-commands)
10. [Pulling Images from Docker Hub](#10-pulling-images)
11. [What is a Docker Image (Deep Dive)](#11-docker-image-deep-dive)
12. [Creating a Container from an Image](#12-creating-containers)
13. [What is CMD in a Docker Image](#13-what-is-cmd)
14. [What is a Docker Container (Deep Dive)](#14-docker-container-deep-dive)
15. [Port Mapping — Connecting the World to Your Container](#15-port-mapping)
16. [Environment Variables for Containers](#16-environment-variables)
17. [Volumes and Volume Mapping — Saving Your Data](#17-volumes)
18. [Running Applications Inside Containers](#18-running-applications)
19. [What is a Dockerfile](#19-what-is-dockerfile)
20. [Creating a Dockerfile — Step by Step](#20-creating-dockerfile)
21. [Launching a Container from Your Custom Image](#21-launching-custom-container)
22. [Connecting Python and MongoDB Containers](#22-connecting-containers)
23. [Custom Bridge Networks](#23-bridge-networks)
24. [Docker Compose and YAML](#24-docker-compose)
25. [Writing Documents to the Database via Compose](#25-compose-database)
26. [Ports and Volumes in Docker Compose](#26-compose-ports-volumes)
27. [Full Summary + Interview Questions](#27-summary-and-interview)

---

## 1. Docker Overview

### 🔴 The Problem: "It Works on MY Machine!"

Imagine you're a developer. You build a web app on your Windows laptop. You use:
- Python 3.9
- A specific database driver version
- A library that only works with a certain OS setting

You send the code to your teammate. They have a Mac. **It crashes.**  
You deploy to the Linux production server. **It crashes differently.**  
You spend 2 days debugging — not your code — but the **environment**.

This is called the **"It works on my machine"** problem. Before Docker, this was a daily nightmare.

---

### 🟡 What Was Used Before Docker?

**Option 1 — Manual Setup Docs**
Teams wrote long instructions like:
```
Step 1: Install Python 3.9 (not 3.10, exactly 3.9)
Step 2: Run pip install -r requirements.txt
Step 3: Install MySQL 5.7 (NOT 8.0)
Step 4: Set environment variable X to Y
Step 5: Hope it works 🙏
```
**Problem:** Human error. Different OS. Someone installs the wrong version. It breaks.

**Option 2 — Virtual Machines (VMs)**  
Package a full OS as a file, share it with teammates.  
**Problem:** VMs are gigabytes in size, take minutes to start, and waste enormous resources. *(Deep comparison coming in Section 3.)*

**Option 3 — Ansible, Chef, Puppet (Config Management Tools)**  
Scripts that *try* to automate environment setup.  
**Problem:** Complex to write, still depends on the base OS, still not perfectly reproducible across teams.

---

### 🟢 What Docker Does

Docker lets you **package your app + its environment + all its dependencies** into one portable unit called a **container**.

Build it once → runs the same way on your laptop, your teammate's Mac, a Linux server, AWS. Everywhere. Always.

```
Your App Code
    +
All Libraries it needs
    +
Exact OS environment settings
    +
Config files
─────────────────────────
= ONE Docker Container ✅
```

---

### 💡 Real-Life Analogy

**Think of shipping containers on a cargo ship.**

Before standardized containers, loading cargo onto ships was chaos — every item was different shapes, needed different handling, could be damaged. Then in the 1950s, **standardized steel containers** were invented. Now it doesn't matter what's inside — the container is the same shape, fits on any ship, any truck, any crane.

Docker is the same idea for software. Your app is the cargo. Docker is the standardized container. Any server (ship) can handle it.

---

### ❓ Cross Questions

**Q: Can't Python's `venv` solve this?**  
A: `venv` only isolates Python packages. It doesn't isolate the OS, system libraries, databases, or other services. Docker isolates *everything*.

**Q: Isn't Docker just for backend/server apps?**  
A: No. Docker works for frontend build tools, databases, message queues, machine learning models, game servers — anything that runs on Linux.

**Q: Who created Docker?**  
A: Docker was created by Solomon Hykes and launched in 2013. It was originally built on top of Linux containers (LXC) but later replaced that with its own container runtime called `containerd`.

---

### 🔗 Related Concepts

Docker is the foundation of modern **DevOps** — the practice of developers and operations teams collaborating smoothly. It also powers:
- **CI/CD pipelines** — automated build/test/deploy systems (Jenkins, GitHub Actions, GitLab CI)
- **Microservices** — each service lives in its own container
- **Kubernetes** — orchestrates thousands of containers across many servers
- **Cloud-native apps** — AWS ECS, Google Cloud Run, Azure Container Instances all run Docker containers

---

## 2. Installing Docker

### Docker Desktop vs Docker Engine — What's the Difference?

| | Docker Desktop | Docker Engine |
|---|---|---|
| **What is it** | Full GUI app with everything included | Just the core Docker daemon (CLI only) |
| **For whom** | Developers on Windows/Mac | Servers, Linux power users |
| **Includes** | Engine + CLI + GUI + Docker Compose | Engine + CLI only |
| **Cost** | Free for personal/small business | Free and open source |

### Installing Docker Desktop (Windows/Mac)

1. Go to [https://docs.docker.com/get-docker/](https://docs.docker.com/get-docker/)
2. Download Docker Desktop for your OS
3. Install and start it
4. Open a terminal and verify:

```bash
docker --version
# Docker version 24.0.5, build ced0996

docker run hello-world
# Should print: Hello from Docker!
```

### Installing Docker Engine (Linux/Ubuntu)

```bash
# Update package list
sudo apt-get update

# Install required packages
sudo apt-get install ca-certificates curl gnupg

# Add Docker's GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Add Docker repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io

# Run without sudo (optional but recommended)
sudo usermod -aG docker $USER
# Log out and log back in for this to take effect
```

---

### ❓ Cross Questions

**Q: Why does Docker Desktop need a Linux VM on Windows/Mac?**  
A: Docker containers use the **Linux kernel** features (namespaces, cgroups). Windows and Mac don't have Linux kernels natively, so Docker Desktop runs a tiny, lightweight Linux VM in the background. You never see it, but it's there. More on this in Section 5.

**Q: Can I use Docker without Docker Desktop?**  
A: On Linux, yes — Docker Engine is all you need. On Windows/Mac, you technically need Docker Desktop OR another tool like Rancher Desktop, Colima (on Mac), or Podman Desktop.

---

## 3. Virtual Machines Architecture

### 🔴 The Problem VMs Were Solving

Before VMs, if you wanted to run two different applications with conflicting requirements (say App A needs Java 8, App B needs Java 17), you needed **two separate physical servers**. Expensive!

### What is a Virtual Machine?

A Virtual Machine is a **software-based fake computer** that runs inside your real computer. It has:
- Its own fake CPU, RAM, and disk (managed by software called a hypervisor)
- Its own complete operating system (a full copy of Windows or Linux)
- Your app running on top of all that

### VM Architecture — Drawn Out

```
┌──────────────────────────────────────────────────────┐
│                PHYSICAL MACHINE                      │
│                                                      │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐     │
│  │   VM 1     │  │   VM 2     │  │   VM 3     │     │
│  │            │  │            │  │            │     │
│  │  App A     │  │  App B     │  │  App C     │     │
│  │  Libs      │  │  Libs      │  │  Libs      │     │
│  │  ──────    │  │  ──────    │  │  ──────    │     │
│  │  Guest OS  │  │  Guest OS  │  │  Guest OS  │     │
│  │  (2GB+)  ← 🔴 Full OS copies waste space  │     │
│  └────────────┘  └────────────┘  └────────────┘     │
│  ┌──────────────────────────────────────────────┐    │
│  │         Hypervisor (VMware/VirtualBox/KVM)   │    │
│  │     Manages fake hardware for each VM        │    │
│  └──────────────────────────────────────────────┘    │
│  ┌──────────────────────────────────────────────┐    │
│  │              Host Operating System           │    │
│  └──────────────────────────────────────────────┘    │
│  ┌──────────────────────────────────────────────┐    │
│  │              Physical Hardware               │    │
│  └──────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────┘
```

### Two Types of Hypervisors

**Type 1 Hypervisor (Bare Metal) — Used in Production**
Runs directly on hardware, no host OS needed.
- Examples: VMware ESXi, Microsoft Hyper-V, KVM
- Used by: Data centers, AWS, Azure, Google Cloud
- Faster and more efficient

**Type 2 Hypervisor (Hosted) — Used by Developers**
Runs on top of an existing OS.
- Examples: VirtualBox, VMware Workstation, Parallels
- Used by: Developers testing things locally
- Easier to use but slightly slower

### How Developers Use VMs in Practice

**Scenario: A developer at a company running Windows needs to test a Linux app**
1. Install VirtualBox on Windows
2. Create a VM with Ubuntu 22.04 (download a 2GB ISO)
3. Install the app inside Ubuntu
4. Test it
5. The VM is a `.vmdk` or `.ova` file they can share

**The problems they face:**
- The VM file is 4-8GB — hard to share
- Starting the VM takes 1-2 minutes
- The VM uses 2GB RAM just for its OS, before the app even starts
- Moving VMs between VMware and VirtualBox is painful (format issues)

---

### 💡 Real-Life Analogy

**VM = Building a whole new house for every guest**

You want to run 5 apps. With VMs, you build 5 houses (full OS each), each with their own kitchen, wiring, plumbing — the complete infrastructure. Very private, but insanely expensive and slow to build.

---

### ❓ Cross Questions

**Q: If VMs are slow and heavy, why does AWS still use them?**  
A: AWS, GCP, Azure *do* use containers heavily now (ECS, GKE, AKS). But VMs still exist for:
- Legacy apps that can't be containerized easily
- Very high security requirements (VMs are more isolated)
- Running Windows workloads (Windows containers exist but are less common)
- The infrastructure *underneath* containers (EC2 = VMs, then Docker runs on them)

**Q: What's the difference between VMware and VirtualBox?**  
A: VirtualBox is free and open-source, great for personal use. VMware is commercial, faster, and used in enterprise/data centers. Both are Type 2 hypervisors for desktop use. VMware ESXi is a Type 1 hypervisor for servers.

**Q: Can a VM run inside a VM?**  
A: Yes! This is called **nested virtualization**. It's supported by modern CPUs and is used in some cloud environments. It's slower but works.

---

### 🔗 Related Concepts

VMs relate to the concept of **hardware abstraction** — the idea of hiding real hardware behind a software layer. This is also what operating systems do (hide hardware from apps), and what cloud computing does (hide data center hardware from users).

---

## 4. Docker Containers Architecture

### The Key Insight: Sharing the Kernel

Docker's revolutionary idea: **containers don't need their own OS kernel**. They share the host machine's Linux kernel. This makes them tiny and fast.

### Container Architecture — Drawn Out

```
┌──────────────────────────────────────────────────────┐
│                PHYSICAL MACHINE                      │
│                                                      │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐     │
│  │Container 1 │  │Container 2 │  │Container 3 │     │
│  │            │  │            │  │            │     │
│  │  App A     │  │  App B     │  │  App C     │     │
│  │  Libs A    │  │  Libs B    │  │  Libs C    │     │
│  │  (MBs) ✅  │  │  (MBs) ✅  │  │  (MBs) ✅  │     │
│  └────────────┘  └────────────┘  └────────────┘     │
│  ┌──────────────────────────────────────────────┐    │
│  │           Docker Engine (containerd)         │    │
│  └──────────────────────────────────────────────┘    │
│  ┌──────────────────────────────────────────────┐    │
│  │       Host OS + Linux Kernel (SHARED ✅)     │    │
│  └──────────────────────────────────────────────┘    │
│  ┌──────────────────────────────────────────────┐    │
│  │              Physical Hardware               │    │
│  └──────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────┘
```

No full OS copies. Containers only carry what's unique to them — their app and their specific libraries.

### How Docker Achieves Isolation Without a Full OS

Docker uses two Linux kernel features to isolate containers:

**1. Namespaces — Isolation**
Namespaces make each container think it's the only process running. Docker uses:
- `pid` namespace → each container has its own process IDs (its PID 1 is isolated)
- `net` namespace → each container has its own network interface
- `mnt` namespace → each container has its own filesystem view
- `uts` namespace → each container has its own hostname
- `ipc` namespace → each container has its own inter-process communication
- `user` namespace → each container has its own user IDs

**2. cgroups (Control Groups) — Resource Limits**
cgroups control how much CPU, memory, and disk I/O a container can use.

```bash
# Example: Limit container to 512MB RAM and 50% of one CPU
docker run --memory="512m" --cpus="0.5" nginx
```

Without cgroups, one container could eat all the RAM and starve others.

---

### VM vs Docker — Final Side-by-Side

| Feature | Virtual Machine | Docker Container |
|---------|----------------|-----------------|
| Size | 2–10 GB | 5–500 MB |
| Boot time | 1–2 minutes | Milliseconds |
| OS overhead | Full OS per VM | Shared kernel |
| Isolation | Hardware-level (very strong) | Process-level (strong) |
| Portability | Medium (format varies) | Excellent |
| Performance | Slightly slower (hypervisor overhead) | Near-native |
| Best for | Full OS isolation, legacy apps | App deployment, microservices |

---

### 💡 Real-Life Analogy

**Containers = Apartments in a building. VMs = Separate houses.**

Apartments share the building's plumbing, electricity grid, and foundation (the Linux kernel). Each apartment is isolated — your neighbor can't walk into your home. But you're not building a whole new house with its own foundation for each person.

---

### ❓ Cross Questions

**Q: If containers share the kernel, can one container crash the host OS?**  
A: It's rare but technically possible with kernel exploits. Normal apps crashing inside containers don't affect the host. The Docker daemon and namespaces provide solid protection for typical use cases.

**Q: Are Docker containers the same as Linux containers (LXC)?**  
A: Docker originally used LXC but replaced it with its own container runtime called `libcontainer` (now `runc`). LXC is lower-level and harder to use. Docker adds a much nicer layer on top.

**Q: Can I run Docker on ARM chips (like Apple M1/M2)?**  
A: Yes! Docker has native ARM support. However, if you try to run an image built for x86/amd64 on ARM, Docker uses **emulation** (QEMU) which is slower. You can build multi-platform images using `docker buildx` that work on both architectures.

---

## 5. How Docker Runs on Different Operating Systems

### 🔴 The Problem

Docker containers use **Linux kernel features** (namespaces, cgroups). But developers work on Windows and Mac which don't have Linux kernels. So how does Docker run on them?

### Linux (Native)

On Linux, Docker runs natively. The containers use the host's Linux kernel directly. This is the fastest, most efficient setup.

```
Your Linux Machine
    └── Linux Kernel (native)
           └── Docker Engine
                  └── Containers (directly use kernel)
```

### macOS (via HyperKit/Apple Virtualization Framework)

macOS has its own kernel (XNU/Darwin), not Linux. Docker Desktop on Mac:
1. Creates a tiny lightweight Linux VM in the background (using Apple's Virtualization Framework or HyperKit)
2. Runs Docker Engine inside that Linux VM
3. Your containers run inside that VM

```
Your Mac
    └── macOS (XNU kernel)
           └── Tiny Linux VM (invisible to you)
                  └── Docker Engine
                         └── Containers
```

This is why Docker Desktop on Mac uses a bit more memory than on Linux — there's that hidden Linux VM.

### Windows (via WSL 2 or Hyper-V)

Docker Desktop on Windows has two modes:

**Mode 1: WSL 2 Backend (recommended)**
WSL 2 = Windows Subsystem for Linux 2. Microsoft built a real Linux kernel that runs inside Windows. Docker uses this.

```
Your Windows PC
    └── Windows Kernel
           └── WSL 2 (real Linux kernel from Microsoft)
                  └── Docker Engine
                         └── Containers
```

**Mode 2: Hyper-V Backend (older)**
Uses Hyper-V (Windows' built-in Type 1 hypervisor) to run a Linux VM, similar to the Mac approach.

---

### 💡 Real-Life Analogy

Imagine you need to run a car that's built for right-hand traffic but you're in a left-hand traffic country. You'd need a translator layer. Docker Desktop on Mac/Windows is that translator — it creates a small Linux environment so your containers (which are "right-hand traffic") can run just fine.

---

### ❓ Cross Questions

**Q: Does this mean Docker on Mac is slower than on Linux?**  
A: Yes, slightly. The extra VM layer adds a small overhead. For most development work you won't notice. For high I/O workloads (like lots of file reads/writes via volume mounts), the difference can be noticeable.

**Q: What is WSL 2 exactly?**  
A: Windows Subsystem for Linux 2 is a feature Microsoft built that lets you run a real Linux kernel on Windows. It's not emulation — it uses Hyper-V to run a lightweight real Linux kernel alongside Windows. You can install Ubuntu, Debian, etc. from the Microsoft Store and run Linux commands natively on Windows.

---

## 6. Container Processes and Resources

### Containers are Just Processes

Here's a mind-bending truth: **a Docker container is not a mini virtual machine**. It's literally just a regular Linux **process** (or group of processes) running on your host machine, with namespace isolation making it feel like its own world.

```bash
# On your host Linux machine, you can see container processes:
ps aux | grep nginx
# You'll see the nginx process from the container listed here!
```

The container's process thinks it's running in isolation, but the host OS sees it as just another process.

### Resource Management with cgroups

Docker uses cgroups to limit what resources a container can use:

```bash
# Limit memory to 256MB
docker run --memory="256m" myapp

# Limit to 1 CPU core
docker run --cpus="1.0" myapp

# Limit CPU shares (relative weight)
docker run --cpu-shares=512 myapp  # half the default weight

# Set memory + swap limit
docker run --memory="256m" --memory-swap="512m" myapp

# Inspect resource usage live
docker stats
```

**`docker stats` output:**
```
CONTAINER ID   NAME      CPU %   MEM USAGE / LIMIT   NET I/O         BLOCK I/O
a1b2c3d4e5f6   web       0.07%   22.4MiB / 256MiB    1.2kB / 880B    0B / 0B
```

### Why This Matters in Production

**Scenario:** You run 5 containers on one server. One has a memory leak and starts consuming all RAM. Without resource limits, it would crash the entire server. With `--memory` limits, Docker kills that container (out-of-memory error) and the other 4 keep running.

---

### ❓ Cross Questions

**Q: What happens when a container exceeds its memory limit?**  
A: The container is killed by the OOM (Out of Memory) killer. You'll see the container status as `Exited (137)`. The host and other containers are unaffected.

**Q: Can I change resource limits without restarting the container?**  
A: Yes! Use `docker update`:
```bash
docker update --memory="512m" my_container
```

**Q: Do containers on the same host share CPU?**  
A: Yes. By default, CPU time is shared fairly. With `--cpu-shares` and `--cpus`, you can control how much CPU each container gets when there's contention.

---

## 7. Docker Components

### The Full Picture

```
┌─────────────────────────────────────────────────────────┐
│                    YOUR COMPUTER                        │
│                                                         │
│  ┌──────────────┐         ┌───────────────────────────┐ │
│  │ Docker Client│ ──────► │   Docker Server (Daemon)  │ │
│  │  (docker CLI)│  REST   │      (dockerd)            │ │
│  └──────────────┘  API    └───────────┬───────────────┘ │
│                                       │                 │
│                            ┌──────────▼──────────────┐  │
│                            │      Docker Host        │  │
│                            │  ┌─────────┐ ┌───────┐  │  │
│                            │  │ Images  │ │ Nets  │  │  │
│                            │  └─────────┘ └───────┘  │  │
│                            │  ┌─────────┐ ┌───────┐  │  │
│                            │  │Containers│ │Volumes│  │  │
│                            │  └─────────┘ └───────┘  │  │
│                            └─────────────────────────┘  │
└──────────────────────────────────┬──────────────────────┘
                                   │ pull/push
                         ┌─────────▼──────────┐
                         │   Docker Registry   │
                         │   (Docker Hub,      │
                         │   private registry) │
                         └─────────────────────┘
```

---

### Docker Client

The **Docker Client** is the command-line tool (`docker`) you type commands into.

```bash
docker build .
docker run nginx
docker pull ubuntu
docker ps
```

Every time you type a `docker` command, the client translates it into a REST API call and sends it to the Docker Daemon. That's all the client does — it's just a messenger.

**Key point:** The client and server don't have to be on the same machine! You can run the Docker client on your laptop and control a Docker daemon on a remote server.

```bash
# Connect to a remote Docker daemon
export DOCKER_HOST=tcp://192.168.1.100:2376
docker ps  # now running against the remote server
```

---

### Docker Server (Daemon — `dockerd`)

The **Docker Daemon** is the background service that does all the real work. It:
- Listens for API calls from the Docker client
- Builds images from Dockerfiles
- Pulls images from registries
- Creates, starts, stops, and deletes containers
- Manages volumes and networks

You never interact with it directly. It runs silently in the background.

```bash
# Check if the daemon is running
sudo systemctl status docker

# View daemon logs
journalctl -u docker

# Start/stop the daemon
sudo systemctl start docker
sudo systemctl stop docker
```

The daemon talks to a lower-level component called **containerd**, which talks to **runc** (the actual container runtime). You don't usually interact with these directly.

```
dockerd (daemon)
   └── containerd (manages container lifecycle)
          └── runc (runs the actual container process)
```

---

### Docker Host

The **Docker Host** is the machine where Docker is installed and running — your laptop, a cloud VM, a server. It contains:
- The Docker daemon
- All images you've pulled or built
- All running and stopped containers
- Volumes (persistent data storage)
- Networks (for container communication)

---

### Docker Image

A **Docker Image** is a read-only, layered template used to create containers.

Think of it as a **snapshot** of a file system — it contains the OS files, app code, libraries, and configuration needed to run your app.

Key facts:
- Images are **read-only** — you can never modify a running image
- Images are made of **layers** (like a stack of file system snapshots)
- Images are identified by a **name and tag**: `nginx:1.25`, `python:3.9-slim`
- If no tag is given, Docker uses `latest` by default

```bash
docker images           # list local images
docker pull nginx       # download an image
docker rmi nginx        # remove an image
docker image inspect nginx  # detailed info about an image
```

---

### Docker Container

A **Docker Container** is a running instance of a Docker Image.

Relationship:
```
Image  →  Container
(Class)    (Object/Instance)

blueprint → building
recipe    → cooked dish
program   → process
```

You can create many containers from the same image. Each container is isolated but starts from the same template.

```bash
docker run nginx           # create + start a container
docker ps                  # list running containers
docker ps -a               # list all containers (including stopped)
docker stop <id>           # stop a running container
docker rm <id>             # delete a stopped container
docker inspect <id>        # full details about a container
```

---

### Docker Repository

A **Docker Repository** is a collection of related images with the same name but different tags.

Example — the `nginx` repository contains:
```
nginx:latest
nginx:1.25
nginx:1.24
nginx:1.23-alpine
nginx:stable
```

Each tag represents a different version or variant.

**Naming convention:**
```
[registry]/[username]/[repository]:[tag]

docker.io/library/nginx:latest          ← official image on Docker Hub
docker.io/myuser/myapp:v1.2.3           ← my image on Docker Hub
ghcr.io/mycompany/backend:production    ← GitHub Container Registry
```

---

### Docker Registry

A **Docker Registry** is the server that stores and serves Docker images. Think of it as GitHub, but for Docker images instead of code.

**Public Registries:**
- **Docker Hub** (`hub.docker.com`) — the default, biggest one
- **GitHub Container Registry** (ghcr.io)
- **Google Container Registry** (gcr.io)
- **AWS ECR** (Amazon Elastic Container Registry)

**Private Registries:**  
Companies run their own private registries so their proprietary images aren't publicly accessible. Tools: Harbor, JFrog Artifactory, AWS ECR.

```bash
# Log into Docker Hub
docker login

# Push an image to Docker Hub
docker push myusername/myapp:v1

# Pull from a specific registry
docker pull ghcr.io/owner/image:tag
```

---

### 💡 Real-Life Analogy for the Full Stack

| Docker Component | Real World Equivalent |
|------------------|-----------------------|
| Docker Client | A waiter taking your order |
| Docker Daemon | The kitchen cooking your food |
| Docker Host | The restaurant building |
| Docker Image | A recipe |
| Docker Container | A cooked dish |
| Docker Repository | A chef's recipe collection (all versions of one dish) |
| Docker Registry | A recipe library / cookbook store |

---

### ❓ Cross Questions

**Q: What's the difference between a Repository and a Registry?**  
A: A **registry** is the server (like Docker Hub — the whole website). A **repository** is one specific collection within that registry (like the `nginx` repository on Docker Hub). Docker Hub has millions of repositories.

**Q: Can two containers share the same image?**  
A: Yes! This is normal and efficient. 10 containers can all be created from the same image. They share the image's layers (read-only), and each container gets its own thin writable layer on top.

**Q: What is `containerd` and how is it different from Docker?**  
A: `containerd` is the low-level container runtime that Docker uses internally. It actually manages container lifecycle. Kubernetes can also use `containerd` directly without Docker. Docker is a higher-level tool that adds developer-friendly features (image building, Docker Compose, etc.) on top of `containerd`.

---

## 8. Docker Commands vs Management Commands

### 🔴 The Problem

As Docker grew, they added a lot of commands. The original commands like `docker run`, `docker ps` were all "flat". But managing images, containers, volumes, and networks needed organization. Docker introduced **management commands** to group related operations.

### Old Style (Still Works)

```bash
docker ps           # list containers
docker images       # list images
docker run nginx    # run a container
docker pull nginx   # pull an image
```

### New Style — Management Commands

```bash
# Containers
docker container ls          # same as docker ps
docker container run nginx   # same as docker run nginx
docker container stop <id>
docker container rm <id>
docker container inspect <id>
docker container logs <id>
docker container exec -it <id> bash

# Images
docker image ls              # same as docker images
docker image pull nginx
docker image rm nginx
docker image build -t myapp .
docker image inspect nginx
docker image history nginx

# Volumes
docker volume ls
docker volume create mydata
docker volume rm mydata
docker volume inspect mydata

# Networks
docker network ls
docker network create mynet
docker network connect mynet <container>
docker network disconnect mynet <container>
```

### Which Style Should You Use?

Both work identically. Management commands are more **organized and explicit**. Old-style commands are **faster to type**. In practice, most developers mix both. Management commands are preferred in documentation and production scripts because they're clearer.

---

### Alternative Commands Worth Knowing

```bash
# Shorthand for running and immediately removing a container after it exits
docker run --rm nginx

# Run in detached mode (background)
docker run -d nginx

# Run interactively with a terminal
docker run -it ubuntu bash

# Run with a name
docker run --name my-nginx nginx

# See container resource usage
docker stats

# See container logs
docker logs <container_name>

# Follow logs in real time
docker logs -f <container_name>

# Execute a command in a running container
docker exec -it <container_name> bash

# Copy files to/from containers
docker cp myfile.txt container_name:/path/inside/
docker cp container_name:/path/inside/file.txt ./local/
```

---

## 9. Basic Container and Image Commands

### Complete Workflow: Pull → Run → Inspect → Clean Up

```bash
# ─────────────────────────────────────────
# IMAGES
# ─────────────────────────────────────────

# Search for an image on Docker Hub
docker search nginx

# Pull an image (download from registry)
docker pull nginx
docker pull nginx:1.25           # specific version
docker pull nginx:alpine         # alpine variant (smaller)

# List local images
docker images
docker image ls

# Get detailed image info
docker image inspect nginx

# See the layers in an image
docker image history nginx

# Remove an image
docker rmi nginx
docker image rm nginx

# Remove all unused images (cleanup)
docker image prune
docker image prune -a            # remove ALL unused images


# ─────────────────────────────────────────
# CONTAINERS
# ─────────────────────────────────────────

# Create and start a container
docker run nginx

# Run in background (detached)
docker run -d nginx

# Run with a custom name
docker run -d --name my-webserver nginx

# List running containers
docker ps

# List all containers (running + stopped)
docker ps -a

# Stop a container
docker stop my-webserver

# Start a stopped container
docker start my-webserver

# Restart a container
docker restart my-webserver

# Remove a container
docker rm my-webserver

# Force remove a running container
docker rm -f my-webserver

# View container logs
docker logs my-webserver
docker logs -f my-webserver      # follow/tail

# Enter a running container interactively
docker exec -it my-webserver bash

# See container resource usage
docker stats
docker stats my-webserver


# ─────────────────────────────────────────
# CLEANUP
# ─────────────────────────────────────────

# Remove all stopped containers
docker container prune

# Remove everything unused (containers, images, networks, cache)
docker system prune

# Remove everything including unused images
docker system prune -a

# Check disk usage
docker system df
```

---

### 💡 Real-Life Scenario: Clean Up Your Docker Setup

After days of experimenting, your disk is full of unused images and stopped containers:

```bash
# See how much space Docker is using
docker system df
# TYPE            TOTAL   ACTIVE   SIZE      RECLAIMABLE
# Images          23      5        8.5GB     7.1GB (83%)
# Containers      47      3        412MB     398MB (96%)
# Volumes         8       3        2.1GB     1.2GB (57%)

# Clean up everything unused
docker system prune -a --volumes
# WARNING! This will remove all unused data.
# Freed 9.3GB of space
```

---

## 10. Pulling Images from Docker Hub

### What is Docker Hub?

Docker Hub (`hub.docker.com`) is the world's largest container image registry. It has:
- **Official images** — maintained by Docker/software vendors (nginx, mysql, python, node)
- **Verified publisher images** — by companies like Microsoft, Oracle
- **Community images** — by individual developers

### Finding the Right Image

When searching on Docker Hub, look for:
- ✅ **Official Image** badge — most trustworthy
- ⭐ High pull count — widely used
- 🕐 Recently updated — actively maintained
- 📄 Good documentation

### Pulling Strategies

```bash
# Pull latest (no tag = latest)
docker pull nginx

# Pull specific version (ALWAYS do this in production!)
docker pull nginx:1.25.3

# Pull a minimal Alpine Linux-based variant (much smaller!)
docker pull nginx:alpine
docker pull python:3.11-alpine

# Check image size difference:
# nginx:latest  →  187 MB
# nginx:alpine  →   41 MB  ← almost 5x smaller!

# Pull for a specific architecture
docker pull --platform linux/amd64 nginx
```

### Why Pin Versions in Production?

```bash
# ❌ BAD — "latest" can change without warning
docker pull nginx:latest

# ✅ GOOD — exact version, reproducible
docker pull nginx:1.25.3
```

If your production server uses `nginx:latest` and you auto-pull, one day Docker Hub updates `latest` to a new major version — your app might break. Always pin versions.

---

### ❓ Cross Questions

**Q: Are Docker Hub images safe to use?**  
A: Official images are generally safe — they're maintained by Docker or the software vendor. Community images should be treated with caution. Always check: who published it, when was it last updated, how many pulls does it have, and review the Dockerfile if available.

**Q: What is an Alpine image?**  
A: Alpine Linux is a minimal Linux distribution (~5MB). Alpine-based Docker images are much smaller because they use musl libc and busybox instead of glibc and full GNU utilities. The tradeoff: some software needs recompilation or doesn't work with musl. For most apps, Alpine is great for production to reduce image size and attack surface.

---

## 11. Docker Image (Deep Dive)

### Images are Made of Layers — The Union File System

Every instruction in a Dockerfile creates a new **layer**. Docker uses a **Union File System** (like OverlayFS on Linux) to stack these layers.

```
Layer 5: COPY app.py /app/          [your app code, 10KB]
Layer 4: RUN pip install flask       [flask + deps, 20MB]
Layer 3: RUN apt-get install python  [python, 30MB]
Layer 2: RUN apt-get update          [package lists, 8MB]
Layer 1: FROM ubuntu:22.04           [base OS, 72MB]
```

Each layer only stores the **diff** (what changed) from the previous layer. Layers are **content-addressed** — identified by a SHA256 hash of their contents.

### Why Layers Are Powerful

**Scenario:** You have 20 microservices, all based on `python:3.11-slim`:

```
Service A: python:3.11-slim → install common deps → install app A
Service B: python:3.11-slim → install common deps → install app B
...
Service T: python:3.11-slim → install common deps → install app T
```

Docker caches layers. The `python:3.11-slim` layer is downloaded **once** and shared across all 20 images. Same for "install common deps" if it's the same. Only the unique top layers are different. This saves enormous disk space.

### The Writable Container Layer

Images are read-only. When you start a container, Docker adds a **thin writable layer** on top:

```
Container writable layer  ← Your app writes here (deleted when container is removed)
─────────────────────────
Image layer 5 (read-only)
Image layer 4 (read-only)
Image layer 3 (read-only)
Image layer 2 (read-only)
Image layer 1 (read-only)
```

When a container modifies a file from the image layers, Docker uses **Copy-on-Write (CoW)**: it copies the file to the writable layer first, then modifies it. The original image layer is untouched.

### Inspecting Image Details

```bash
# Full JSON details
docker image inspect nginx

# See all layers and how they were created
docker image history nginx

# Example output:
# IMAGE          CREATED BY                                      SIZE
# 89da1fb6dcf6   /bin/sh -c #(nop) CMD ["nginx" "-g" "daemon…   0B
# <missing>      /bin/sh -c #(nop) STOPSIGNAL SIGQUIT           0B
# <missing>      /bin/sh -c set -x && addgroup --system --gi…   61.7MB
# <missing>      /bin/sh -c #(nop) ENV PKG_RELEASE=1~bookworm   0B
```

---

### ❓ Cross Questions

**Q: If layers are cached, when does the cache become invalid?**  
A: Docker invalidates cache for a layer (and all subsequent layers) when:
- The instruction changes
- For `COPY`/`ADD` instructions: the file contents change (Docker checksums files)
- You use `--no-cache` flag: `docker build --no-cache .`

**Q: What's the difference between an image ID and an image name/tag?**  
A: The **ID** (like `89da1fb6dcf6`) is a SHA256 hash of the image contents — it's permanent and unique. The **name:tag** (like `nginx:1.25`) is a human-readable alias that points to an ID. Multiple tags can point to the same ID. The `latest` tag is just a convention — it's not guaranteed to actually be the latest.

---

## 12. Creating a Container from an Image

### The `docker run` Command — Full Breakdown

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARGS]
```

| Component | Description |
|-----------|-------------|
| `OPTIONS` | Flags like `-d`, `-p`, `-v`, `--name` |
| `IMAGE` | The image name/tag to create a container from |
| `COMMAND` | Override the default command in the image |
| `ARGS` | Arguments to that command |

### What Happens When You `docker run`

1. Docker Client receives your command
2. Client sends API request to Docker Daemon
3. Daemon checks: is the image available locally?
4. If not: pulls the image from Docker Hub (or specified registry)
5. Daemon creates a container from the image (adds writable layer)
6. Daemon sets up networking, volumes as requested
7. Daemon starts the container process
8. (If `-it` is specified) attaches your terminal to the container

```bash
# Simple run
docker run nginx

# Detached (background)
docker run -d nginx

# Named container
docker run -d --name webserver nginx

# Interactive with terminal
docker run -it ubuntu bash

# With port mapping (host:container)
docker run -d -p 8080:80 nginx

# With environment variable
docker run -d -e MYSQL_ROOT_PASSWORD=secret mysql

# With volume mount
docker run -d -v /mydata:/data nginx

# Combining multiple options
docker run -d \
  --name webserver \
  -p 8080:80 \
  -e APP_ENV=production \
  -v /logs:/var/log/nginx \
  nginx:1.25
```

---

### Container Lifecycle

```
docker run → [Created] → [Running] → docker stop → [Stopped/Exited]
                                           ↑               |
                                           └───── docker start ─┘
                                                          |
                                                    docker rm → [Deleted]
```

```bash
docker create nginx            # create without starting
docker start my-container      # start a stopped container
docker pause my-container      # freeze (SIGSTOP)
docker unpause my-container    # unfreeze
docker stop my-container       # graceful stop (SIGTERM, then SIGKILL after 10s)
docker kill my-container       # immediate kill (SIGKILL)
docker rm my-container         # delete (must be stopped first)
docker rm -f my-container      # force delete even if running
```

---

## 13. What is CMD in the Docker Image?

### 🔴 The Problem

When you run `docker run nginx`, how does Docker know to start the nginx web server? You didn't tell it to run `nginx -g "daemon off;"`. Something in the image tells it what to do by default.

That something is **CMD**.

### CMD — The Default Command

`CMD` defines the **default command** that runs when a container starts. It's baked into the image.

```dockerfile
# Example: official nginx Dockerfile has this at the end:
CMD ["nginx", "-g", "daemon off;"]
```

When you do `docker run nginx`, Docker sees this CMD and runs `nginx -g "daemon off;"` automatically.

### Overriding CMD

You can override CMD by providing a command at the end of `docker run`:

```bash
# Instead of running nginx, run bash
docker run -it nginx bash

# Instead of default CMD, print nginx version
docker run nginx nginx -v

# Run ubuntu with bash (ubuntu's default CMD is bash anyway)
docker run -it ubuntu bash
```

### CMD vs ENTRYPOINT

This trips up many beginners. Both define what runs when a container starts.

| | CMD | ENTRYPOINT |
|--|-----|------------|
| **Purpose** | Default command (easily overridden) | Fixed executable (always runs) |
| **Can be overridden** | Yes, by adding command to `docker run` | Only with `--entrypoint` flag |
| **Use case** | Providing defaults | Defining the container's main purpose |

```dockerfile
# CMD only: entire command is overridable
CMD ["python", "app.py"]

# ENTRYPOINT only: python is always run, args can vary
ENTRYPOINT ["python"]
CMD ["app.py"]  # default argument to ENTRYPOINT
# docker run myimage = python app.py
# docker run myimage test.py = python test.py

# Common pattern: ENTRYPOINT sets the executable, CMD sets default args
ENTRYPOINT ["nginx"]
CMD ["-g", "daemon off;"]
```

---

### 💡 Real-Life Analogy

Think of a vending machine:
- The **ENTRYPOINT** is the machine itself — it's always a vending machine, that's fixed.
- The **CMD** is the default item selection — it might default to cola, but you can press other buttons to change it.

---

### ❓ Cross Questions

**Q: What happens if a Docker image has no CMD and no ENTRYPOINT?**  
A: Docker will give you an error when you try to run it without specifying a command. The container doesn't know what to do.

**Q: What's the difference between `CMD ["nginx"]` (exec form) and `CMD nginx` (shell form)?**  
A: Exec form `CMD ["nginx", "-g", "daemon off;"]` runs the command directly — the process becomes PID 1 in the container. Shell form `CMD nginx -g "daemon off;"` wraps in `/bin/sh -c`, so the shell is PID 1 and nginx is a child. This matters for signal handling (graceful shutdown). Always use exec form in production.

---

## 14. Docker Container (Deep Dive)

### What Exactly is a Container?

A container is an **isolated process** (or group of processes) that:
1. Has its own filesystem view (from the image layers + writable layer)
2. Has its own network interface
3. Has its own process namespace (its main process is PID 1 inside the container)
4. Shares the host's Linux kernel

When the main process (PID 1) inside a container exits, **the container stops**. This is fundamentally different from VMs, which keep running even without an active process.

### Entering a Running Container

```bash
# Get a shell inside a running container
docker exec -it my-container bash   # if bash is available
docker exec -it my-container sh     # sh is almost always available

# Run a specific command
docker exec my-container ls /var/log
docker exec my-container cat /etc/nginx/nginx.conf

# Run as a different user
docker exec -it --user root my-container bash
```

### Container Logs

```bash
# View all logs since start
docker logs my-container

# Follow logs in real time (like tail -f)
docker logs -f my-container

# Last 50 lines
docker logs --tail 50 my-container

# With timestamps
docker logs -t my-container

# Since a specific time
docker logs --since 30m my-container     # last 30 minutes
docker logs --since "2024-01-01" my-container
```

### Container States — In Detail

```bash
docker ps -a
# CONTAINER ID  IMAGE   COMMAND   CREATED   STATUS              PORTS   NAMES
# a1b2c3d4e5f6  nginx   "nginx…"  5m ago    Up 5 minutes        80/tcp  webserver
# b2c3d4e5f6a1  mysql   "docker…" 1h ago    Exited (1) 30m ago          mydb
# c3d4e5f6a1b2  ubuntu  "bash"    2h ago    Exited (0) 2h ago           test

# Status meanings:
# Up = Running
# Exited (0) = Stopped cleanly (success exit code)
# Exited (1) = Stopped with error
# Exited (137) = Killed (OOM or manual kill)
# Created = Created but never started
# Paused = Frozen
# Restarting = Crash loop
```

### Restart Policies

```bash
# Never restart (default)
docker run --restart=no nginx

# Restart only on failure
docker run --restart=on-failure nginx

# Restart on failure, max 5 times
docker run --restart=on-failure:5 nginx

# Always restart (even if manually stopped — be careful!)
docker run --restart=always nginx

# Restart unless manually stopped
docker run --restart=unless-stopped nginx  # ← most common for production
```

---

### ❓ Cross Questions

**Q: If I `docker stop` a container and `docker start` it again, does it retain its state?**  
A: Yes — the writable layer persists when a container is stopped and restarted. You'll find files you created inside it are still there. But if you `docker rm` the container, the writable layer is deleted. This is why volumes are important for data that must survive container deletion.

**Q: What's the difference between `docker stop` and `docker kill`?**  
A: `docker stop` sends `SIGTERM` (please stop gracefully), waits 10 seconds, then sends `SIGKILL` (forced stop). `docker kill` sends `SIGKILL` immediately — no grace period. Use `stop` in production for clean shutdown.

---

## 15. Port Mapping

### 🔴 The Problem

You start an nginx container. You try to open `http://localhost` in your browser. Nothing. Why?

By default, containers are **completely isolated from the network**. NGINX is listening on port 80 *inside* the container, but nothing on your host machine knows about it. There's no connection between the container's network and your machine's network.

### What is Port Mapping?

Port mapping (also called port publishing) creates a **bridge** between a port on your host machine and a port inside the container.

```
Your Browser → localhost:8080 → [HOST PORT 8080] → [CONTAINER PORT 80] → nginx process
```

### The `-p` Flag

```bash
# Syntax: -p HOST_PORT:CONTAINER_PORT
docker run -d -p 8080:80 nginx

# Now:
# localhost:8080 → nginx inside container on port 80 ✅

# Multiple port mappings
docker run -d -p 8080:80 -p 8443:443 nginx

# Map to a specific host IP (only accept from localhost)
docker run -d -p 127.0.0.1:8080:80 nginx

# Random host port (Docker assigns one)
docker run -d -p 80 nginx
docker port my-container   # see which host port was assigned
```

### Common Port Mappings

| Application | Common Container Port | Example Mapping |
|-------------|----------------------|-----------------|
| NGINX | 80, 443 | `-p 8080:80` |
| Apache | 80 | `-p 8080:80` |
| MySQL | 3306 | `-p 3306:3306` |
| PostgreSQL | 5432 | `-p 5432:5432` |
| MongoDB | 27017 | `-p 27017:27017` |
| Redis | 6379 | `-p 6379:6379` |
| Node.js app | 3000 | `-p 3000:3000` |
| React dev server | 3000 | `-p 3000:3000` |

---

### 💡 Full Real-Life Scenario: Running NGINX with Port Mapping

```bash
# Step 1: Run nginx in detached mode with port mapping
docker run -d --name webserver -p 8080:80 nginx

# Step 2: Verify it's running
docker ps
# CONTAINER ID  IMAGE  ...  PORTS                   NAMES
# a1b2c3d4e5f6  nginx  ...  0.0.0.0:8080->80/tcp   webserver

# Step 3: Open browser → http://localhost:8080
# You see the nginx welcome page! ✅

# Step 4: Check logs
docker logs webserver

# Step 5: Stop and clean up
docker stop webserver
docker rm webserver
```

---

### ❓ Cross Questions

**Q: Can two containers map to the same host port?**  
A: No! You'd get an error like `Bind for 0.0.0.0:8080 failed: port is already allocated`. Each host port can only be used by one process at a time. This is the same as trying to run two programs on the same port natively.

**Q: What's `0.0.0.0` in the ports output?**  
A: `0.0.0.0` means the port is bound to ALL network interfaces on the host — accessible from your local machine AND from other machines on the network. If you want to restrict to only local access: `-p 127.0.0.1:8080:80`.

**Q: Is it necessary to expose ports in the Dockerfile?**  
A: `EXPOSE` in a Dockerfile is **documentation only** — it doesn't actually publish ports. You still need `-p` at runtime. `EXPOSE` tells other developers "this container listens on port 80," but it doesn't do anything by itself.

---

## 16. Environment Variables for Containers

### 🔴 The Problem

Your app needs different database passwords in development vs production. You can't hardcode secrets in your code. You need a way to pass configuration to containers at runtime.

### What are Environment Variables?

Key-value pairs passed to a container that the app running inside can read. This follows the **12-Factor App** methodology — configuration should come from the environment, not be hardcoded.

### Passing Environment Variables

```bash
# Single variable
docker run -d -e MYSQL_ROOT_PASSWORD=mysecretpassword mysql

# Multiple variables
docker run -d \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=myapp \
  -e MYSQL_USER=appuser \
  -e MYSQL_PASSWORD=apppass \
  mysql

# From a file (safer for secrets!)
# Create a file: db.env
# MYSQL_ROOT_PASSWORD=secret
# MYSQL_DATABASE=myapp
docker run -d --env-file db.env mysql

# Read from current shell environment
export MY_SECRET=value123
docker run -d -e MY_SECRET nginx  # passes the value from shell
```

### Real Example: MySQL Container

```bash
# MySQL REQUIRES at least one of these environment variables:
# MYSQL_ROOT_PASSWORD or MYSQL_ALLOW_EMPTY_PASSWORD or MYSQL_RANDOM_ROOT_PASSWORD

docker run -d \
  --name my-mysql \
  -e MYSQL_ROOT_PASSWORD=rootpass123 \
  -e MYSQL_DATABASE=myappdb \
  -e MYSQL_USER=appuser \
  -e MYSQL_PASSWORD=apppass123 \
  -p 3306:3306 \
  mysql:8.0

# Verify it started
docker logs my-mysql  # look for "ready for connections"

# Connect to it
docker exec -it my-mysql mysql -u root -p
# Enter password: rootpass123
```

### Inspecting Environment Variables

```bash
# See all env vars in a running container
docker exec my-mysql env

# Or inspect the container config
docker inspect my-mysql | grep -A 20 '"Env"'
```

---

### 💡 Real-Life Scenario: Dev vs Prod Config

```bash
# Development
docker run -d \
  -e APP_ENV=development \
  -e DB_HOST=localhost \
  -e DB_PASSWORD=devpassword \
  -e DEBUG=true \
  myapp:latest

# Production
docker run -d \
  -e APP_ENV=production \
  -e DB_HOST=prod-db.company.com \
  -e DB_PASSWORD=superSecurePassword! \
  -e DEBUG=false \
  myapp:latest
```

Same image, different behavior based on environment.

---

### ❓ Cross Questions

**Q: Are environment variables secure for storing passwords?**  
A: Better than hardcoding them in code, but environment variables can be seen by `docker inspect` and in process listings. For production secrets, use proper secret management tools: Docker Secrets (with Swarm), Kubernetes Secrets, HashiCorp Vault, AWS Secrets Manager, etc.

**Q: What happens if a required environment variable is not set?**  
A: It depends on the app. Some apps (like MySQL) will refuse to start and print an error. Others may start with unexpected behavior or use default values.

---

## 17. Volumes and Volume Mapping

### 🔴 The Problem: Containers Don't Remember

**Scenario:** You start a MySQL container, add some data to the database, then stop and remove the container.  
You start a new MySQL container.  
**Your data is gone.** 😱

This happens because all writes inside a container go to the **writable container layer**, which is deleted when the container is removed.

For stateless apps (web servers serving static content), this is fine. For databases, file uploads, logs — this is catastrophic.

### What are Volumes?

Volumes are a way to **persist data outside the container's writable layer**, so data survives container restarts, replacements, and deletions.

Docker has three types:

**1. Docker Managed Volumes (the preferred way)**
Docker creates and manages a directory on the host. You don't control the path.

```bash
# Create a named volume
docker volume create mydata

# Use it
docker run -d -v mydata:/var/lib/mysql mysql

# Docker stores it at: /var/lib/docker/volumes/mydata/_data
```

**2. Bind Mounts (mount a specific host directory)**
You specify an exact path on your host machine.

```bash
# Mount current directory to /app inside container
docker run -d -v $(pwd)/myapp:/app node

# Mount a specific path
docker run -d -v /home/user/logs:/var/log/nginx nginx
```

**3. tmpfs Mounts (in-memory, not persistent)**
Data stored in memory only. Gone when container stops.

```bash
docker run -d --tmpfs /tmp nginx
```

### Volume Commands

```bash
# Create a volume
docker volume create mydata

# List all volumes
docker volume ls

# Inspect a volume (see where data is stored)
docker volume inspect mydata
# Shows: "Mountpoint": "/var/lib/docker/volumes/mydata/_data"

# Remove a volume
docker volume rm mydata

# Remove all unused volumes
docker volume prune
```

---

### 💡 Real-Life Scenario: NGINX with Volume for Config and Logs

```bash
# Create directories on host
mkdir -p /home/user/nginx/conf
mkdir -p /home/user/nginx/logs

# Create a custom nginx config
cat > /home/user/nginx/conf/nginx.conf << 'EOF'
server {
    listen 80;
    root /usr/share/nginx/html;
    index index.html;
}
EOF

# Run nginx with:
# - Port mapping (host 8080 → container 80)
# - Bind mount for config
# - Bind mount for logs
docker run -d \
  --name webserver \
  -p 8080:80 \
  -v /home/user/nginx/conf/nginx.conf:/etc/nginx/conf.d/default.conf \
  -v /home/user/nginx/logs:/var/log/nginx \
  nginx

# Now nginx uses YOUR config file
# And nginx logs are written to /home/user/nginx/logs on your HOST machine
# Even if the container is deleted, logs are safe
```

### Volumes in Development — Hot Reload

A killer use case: mount your source code into the container so changes take effect immediately without rebuilding the image.

```bash
# Mount current directory as the web root
docker run -d \
  --name dev-server \
  -p 3000:3000 \
  -v $(pwd):/app \
  -w /app \
  node:18 \
  npm start

# Now edit files on your host → changes reflected in the container immediately!
```

---

### ❓ Cross Questions

**Q: Named volumes vs bind mounts — when to use which?**  
A: Use **named volumes** for databases and app data in production — Docker manages the path, easier to backup, works the same across OSes. Use **bind mounts** during development to sync your source code, or when you need to share specific files from a known host path.

**Q: Can multiple containers share the same volume?**  
A: Yes! Multiple containers can mount the same volume. Be careful: if both containers write to the same file simultaneously without locking, data corruption can occur. Databases handle this with their own locking mechanisms.

**Q: Are volumes included in `docker commit` (creating an image from a running container)?**  
A: No. `docker commit` captures the container's writable layer but NOT volume data. Volumes are separate from images by design.

---

## 18. Running Applications Inside Containers

### The Full Development Workflow

**Scenario:** Run a Python Flask app inside Docker.

```bash
# Step 1: Run a Python container interactively
docker run -it --name flask-dev -p 5000:5000 python:3.11 bash

# You're now inside the container
# Step 2: Install Flask
pip install flask

# Step 3: Write a quick app
cat > /app.py << 'EOF'
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello from Docker!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
EOF

# Step 4: Run it
python app.py

# In your browser: http://localhost:5000 → "Hello from Docker!"
```

But wait — if you exit this container and remove it, your `app.py` and Flask installation are gone. This is exactly why we need Dockerfiles and proper images.

---

## 19. What is a Dockerfile?

### 🔴 The Problem

If you configure a container manually (`docker exec -it ... bash` and install things), you:
- Can't reproduce it automatically
- Can't track changes (no version control)
- Have to redo it every time you need a fresh container
- Can't share it with your team easily

### What is a Dockerfile?

A **Dockerfile** is a text file with a series of instructions that Docker uses to **automatically build an image**. It's like a recipe card that tells Docker exactly how to construct your app's environment.

```
Dockerfile → docker build → Docker Image → docker run → Container
```

### 🔗 Related Concept

Dockerfile is to Docker what a `Makefile` is to C/C++ builds — a set of instructions to automate the build process. Or think of it like a shell script that sets up your environment, but it builds an image layer by layer.

### Why Dockerfiles are Awesome

- ✅ **Reproducible** — same Dockerfile = same image, every time
- ✅ **Version controlled** — commit your Dockerfile to Git
- ✅ **Automated** — CI/CD pipelines build images from Dockerfiles automatically
- ✅ **Documented** — the Dockerfile IS the documentation of your environment
- ✅ **Shareable** — push the image to Docker Hub; anyone can use it

---

## 20. Creating a Dockerfile

### Every Instruction Explained

```dockerfile
# ──────────────────────────────────────────────────
# FROM — Base image to start from (REQUIRED, must be first)
# ──────────────────────────────────────────────────
FROM python:3.11-slim

# ──────────────────────────────────────────────────
# LABEL — Metadata about the image
# ──────────────────────────────────────────────────
LABEL maintainer="yourname@example.com"
LABEL version="1.0"
LABEL description="My Python Flask App"

# ──────────────────────────────────────────────────
# ENV — Set environment variables (available during build AND at runtime)
# ──────────────────────────────────────────────────
ENV APP_HOME=/app
ENV FLASK_ENV=production
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

# ──────────────────────────────────────────────────
# WORKDIR — Set the working directory (like cd)
#           Created automatically if it doesn't exist
# ──────────────────────────────────────────────────
WORKDIR $APP_HOME

# ──────────────────────────────────────────────────
# COPY — Copy files from host to image
# COPY <src on host> <dest in image>
# ──────────────────────────────────────────────────

# Copy requirements first (for better layer caching!)
COPY requirements.txt .

# ──────────────────────────────────────────────────
# RUN — Execute a command during BUILD time (creates a new layer)
# ──────────────────────────────────────────────────
RUN pip install --no-cache-dir -r requirements.txt

# Now copy the rest of the app (after installing deps for better caching)
COPY . .

# ──────────────────────────────────────────────────
# EXPOSE — Document which port the app listens on
# (Does NOT publish the port — just documentation)
# ──────────────────────────────────────────────────
EXPOSE 5000

# ──────────────────────────────────────────────────
# CMD — Default command when container starts
# Exec form (recommended): ["executable", "arg1", "arg2"]
# ──────────────────────────────────────────────────
CMD ["python", "app.py"]
```

### Full Real-World Example

**Project structure:**
```
my-flask-app/
├── Dockerfile
├── requirements.txt
├── app.py
└── templates/
    └── index.html
```

**requirements.txt:**
```
flask==3.0.0
gunicorn==21.2.0
```

**app.py:**
```python
from flask import Flask, render_template
import os

app = Flask(__name__)

@app.route('/')
def index():
    name = os.environ.get('APP_NAME', 'World')
    return f'Hello, {name}! Running in Docker.'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=False)
```

**Dockerfile:**
```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```

### Building the Image

```bash
# Build the image
# -t = tag (name:version)
# .  = build context (current directory, where Dockerfile is)
docker build -t my-flask-app:v1 .

# You'll see each layer being built:
# Step 1/6 : FROM python:3.11-slim
# Step 2/6 : WORKDIR /app
# Step 3/6 : COPY requirements.txt .
# Step 4/6 : RUN pip install ...
# Step 5/6 : COPY . .
# Step 6/6 : CMD ["python", "app.py"]
# Successfully built a1b2c3d4e5f6
# Successfully tagged my-flask-app:v1

# Verify image was created
docker images | grep my-flask-app
```

### The Layer Caching Trick — CRITICAL for Fast Builds

```dockerfile
# ❌ BAD ORDER — slow rebuilds
FROM python:3.11-slim
WORKDIR /app
COPY . .                           # copies everything first
RUN pip install -r requirements.txt  # runs after code changes = NO CACHE

# ✅ GOOD ORDER — fast rebuilds
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .            # copy only deps file first
RUN pip install -r requirements.txt  # this layer is cached if requirements.txt unchanged
COPY . .                           # copy app code (changes often, but pip is already cached)
```

With the good order: if you only change `app.py`, Docker caches the `pip install` layer and only rebuilds the last `COPY . .` layer. Build goes from 60 seconds to 2 seconds!

---

### ❓ Cross Questions

**Q: What is a `.dockerignore` file?**  
A: Like `.gitignore` but for Docker builds. It tells Docker which files NOT to include when building. Always use it to exclude:
```
# .dockerignore
.git
node_modules
__pycache__
*.pyc
.env
*.log
```
Without it, Docker might send gigabytes of `node_modules` to the build context, making builds very slow.

**Q: What's the difference between `RUN`, `CMD`, and `ENTRYPOINT`?**  
A: `RUN` runs during **image build** (creates a layer). `CMD` and `ENTRYPOINT` run when the **container starts** (not during build). `CMD` is the default command (overridable). `ENTRYPOINT` is the fixed executable.

**Q: What is a multi-stage build?**  
A: An advanced technique for smaller production images:
```dockerfile
# Stage 1: Build (has compilers, build tools — large)
FROM node:18 AS builder
WORKDIR /app
COPY . .
RUN npm install && npm run build

# Stage 2: Production (only the built output — small)
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
# Final image only has nginx + built files, no node_modules!
```
This can reduce image sizes from 1GB+ to under 50MB.

---

## 21. Launching a Container from Your Custom Image

```bash
# Build your image
docker build -t my-flask-app:v1 .

# Run it
docker run -d \
  --name flask-api \
  -p 5000:5000 \
  -e APP_NAME=Docker \
  my-flask-app:v1

# Verify
docker ps
curl http://localhost:5000
# Hello, Docker! Running in Docker.

# Check logs
docker logs flask-api

# Enter the container to debug
docker exec -it flask-api bash

# Tag it for Docker Hub
docker tag my-flask-app:v1 yourusername/my-flask-app:v1

# Push to Docker Hub
docker login
docker push yourusername/my-flask-app:v1
```

---

## 22. Connecting Python and MongoDB Containers

### 🔴 The Problem

You want to run a Python app and a MongoDB database. You start them as separate containers. Your Python app tries to connect to MongoDB. It fails.

Why? Because each container is on its own **isolated network**. The Python container can't reach the MongoDB container. They're in separate network bubbles.

```bash
# Start MongoDB
docker run -d --name mongodb mongo:6.0

# Start Python app
docker run -d --name python-app my-python-app

# Inside python-app, try to connect to mongodb:
# connection = MongoClient("mongodb://mongodb:27017")
# Error: [Errno -2] Name or service not known
# The name "mongodb" doesn't resolve!
```

### Why Can't They Talk?

By default, Docker creates a **bridge network** but each container only knows about itself. Container names are not automatically DNS-resolvable to each other on the default bridge network.

*(Solution: Custom bridge networks — coming in Section 23)*

---

## 23. Custom Bridge Networks

### Docker Networking Modes

| Network Driver | Description | Use Case |
|----------------|-------------|----------|
| **bridge** | Default. Containers on the same bridge can communicate. | Single host, isolated apps |
| **host** | Container shares host's network namespace. | Performance-critical, removes network isolation |
| **none** | No networking at all. | Totally isolated containers |
| **overlay** | Multi-host networking. | Docker Swarm, distributed apps |
| **macvlan** | Container gets its own MAC address. | Legacy apps expecting direct network access |

### The Default Bridge Network Problem

On the default `bridge` network, containers cannot resolve each other by name — only by IP address, and IPs change every time you restart containers.

### Custom Bridge Networks — The Solution

When you create a **custom bridge network**, Docker provides automatic **DNS resolution** between containers. You can use container names as hostnames!

```bash
# Step 1: Create a custom network
docker network create my-app-network

# List networks
docker network ls
# NETWORK ID   NAME              DRIVER  SCOPE
# a1b2c3d4     bridge            bridge  local  ← default
# b2c3d4e5     host              host    local
# c3d4e5f6     none              null    local
# d4e5f6a7     my-app-network    bridge  local  ← ours!

# Step 2: Start MongoDB on this network
docker run -d \
  --name mongodb \
  --network my-app-network \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=password \
  mongo:6.0

# Step 3: Start Python app on the SAME network
docker run -d \
  --name python-app \
  --network my-app-network \
  my-python-app

# Now inside python-app:
# MongoClient("mongodb://admin:password@mongodb:27017")
# ✅ "mongodb" resolves to the MongoDB container's IP automatically!
```

### Inspecting Networks

```bash
# See containers on a network
docker network inspect my-app-network

# Connect an existing container to a network
docker network connect my-app-network existing-container

# Disconnect
docker network disconnect my-app-network existing-container
```

---

### 💡 Real-Life Analogy

Think of Docker networks like office building floors. 

- The default bridge is like the ground floor lobby — everyone can be there, but they don't know each other's names, only faces (IP addresses).
- A custom network is like a private floor where everyone has a nameplate on their door — you can knock on "MongoDB's" door by name without knowing their room number.

---

### ❓ Cross Questions

**Q: What is the `--network host` option?**  
A: The container uses the host's network directly — no isolation. The container sees and uses the same IP addresses and ports as the host. This is the fastest option (no network translation overhead) but sacrifices isolation. Useful for performance-critical apps or when the app needs to see the host's actual network interface.

**Q: Can a container be on multiple networks?**  
A: Yes! A container can be connected to multiple networks simultaneously. This is useful when a container needs to communicate with different groups of services.

---

## 24. Docker Compose and YAML

### 🔴 The Problem

As your app grows, you need to run multiple containers together — a web server, a database, a cache, a message queue. Starting them all with long `docker run` commands becomes a nightmare:

```bash
# Imagine doing this every time you start your dev environment:
docker network create my-network
docker volume create db-data

docker run -d --name mongodb \
  --network my-network \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=pass \
  -v db-data:/data/db \
  mongo:6.0

docker run -d --name redis \
  --network my-network \
  redis:7

docker run -d --name backend \
  --network my-network \
  -p 5000:5000 \
  -e MONGO_URI=mongodb://admin:pass@mongodb:27017 \
  -e REDIS_URL=redis://redis:6379 \
  my-backend:v1

docker run -d --name frontend \
  --network my-network \
  -p 3000:3000 \
  my-frontend:v1
```

This is 4 commands that need to be run in the right order, every time. Horrible.

### What is Docker Compose?

**Docker Compose** is a tool that lets you define and run multi-container applications using a single YAML file. With one command (`docker compose up`), your entire application stack starts.

### What is YAML?

YAML (YAML Ain't Markup Language) is a human-readable data format using indentation to represent structure:

```yaml
# YAML basics
name: John           # string
age: 30              # number
is_developer: true   # boolean

address:             # nested object
  city: Hyderabad
  country: India

skills:              # list
  - Python
  - Docker
  - Kubernetes

# Key rules:
# - Indentation uses SPACES (never tabs!)
# - 2 spaces per level (convention)
# - Strings with special characters need quotes
```

### The `docker-compose.yml` File Structure

```yaml
version: "3.9"   # Docker Compose file format version

services:         # Define each container here
  service-name:
    image: ...    # or build: ...
    ports:
      - "host:container"
    environment:
      - KEY=value
    volumes:
      - data:/path/in/container
    networks:
      - my-network
    depends_on:
      - other-service

volumes:           # Named volumes
  data:

networks:          # Custom networks
  my-network:
```

---

## 25. Launching Services with Docker Compose

### Full Example: Python App + MongoDB

**Project structure:**
```
my-project/
├── docker-compose.yml
├── backend/
│   ├── Dockerfile
│   ├── app.py
│   └── requirements.txt
```

**backend/app.py:**
```python
from flask import Flask, jsonify, request
from pymongo import MongoClient
import os

app = Flask(__name__)

# Docker Compose makes "mongodb" resolvable as a hostname!
client = MongoClient(os.environ.get('MONGO_URI', 'mongodb://mongodb:27017'))
db = client['myapp']

@app.route('/users', methods=['GET'])
def get_users():
    users = list(db.users.find({}, {'_id': 0}))
    return jsonify(users)

@app.route('/users', methods=['POST'])
def create_user():
    user = request.json
    db.users.insert_one(user)
    return jsonify({'message': 'User created!'}), 201

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

**docker-compose.yml:**
```yaml
version: "3.9"

services:

  # MongoDB service
  mongodb:
    image: mongo:6.0
    container_name: mongodb
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: secretpassword
      MONGO_INITDB_DATABASE: myapp
    volumes:
      - mongo-data:/data/db     # persist data
    networks:
      - app-network
    restart: unless-stopped

  # Python Flask Backend
  backend:
    build:
      context: ./backend        # build from Dockerfile in ./backend
      dockerfile: Dockerfile
    container_name: backend
    ports:
      - "5000:5000"
    environment:
      MONGO_URI: mongodb://admin:secretpassword@mongodb:27017
      FLASK_ENV: development
    volumes:
      - ./backend:/app          # hot reload during dev
    networks:
      - app-network
    depends_on:
      - mongodb                 # wait for mongodb to start first
    restart: unless-stopped

volumes:
  mongo-data:                   # named volume for MongoDB data

networks:
  app-network:                  # custom bridge network
    driver: bridge
```

### Docker Compose Commands

```bash
# Start all services (in background)
docker compose up -d

# Start and rebuild images first
docker compose up -d --build

# View running services
docker compose ps

# View logs for all services
docker compose logs

# View logs for a specific service
docker compose logs backend

# Follow logs in real time
docker compose logs -f

# Stop all services (keeps containers and volumes)
docker compose stop

# Stop and REMOVE containers (keeps volumes)
docker compose down

# Stop, remove containers AND volumes (DELETES DATA!)
docker compose down -v

# Scale a service (run multiple instances)
docker compose up -d --scale backend=3

# Restart a specific service
docker compose restart backend

# Execute command in a service
docker compose exec backend bash
docker compose exec mongodb mongosh -u admin -p secretpassword
```

---

## 26. Writing Documents to the Database + Ports and Volumes in Compose

### Testing the App

```bash
# Start the stack
docker compose up -d

# Check all services are running
docker compose ps
# NAME       IMAGE           STATUS    PORTS
# mongodb    mongo:6.0       running
# backend    my-backend      running   0.0.0.0:5000->5000/tcp

# Write a user document to the database
curl -X POST http://localhost:5000/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Alice", "email": "alice@example.com", "city": "Hyderabad"}'
# {"message": "User created!"}

# Read all users
curl http://localhost:5000/users
# [{"name": "Alice", "email": "alice@example.com", "city": "Hyderabad"}]

# Stop and remove containers
docker compose down

# Start again — data is still there because of the named volume!
docker compose up -d
curl http://localhost:5000/users
# [{"name": "Alice", ...}]  ← Data survived! ✅
```

### Ports in Docker Compose

```yaml
services:
  nginx:
    image: nginx
    ports:
      - "8080:80"           # host:container
      - "8443:443"          # multiple mappings
      - "127.0.0.1:9090:80" # bind to specific IP only
      - "80"                # random host port
```

### Volumes in Docker Compose

```yaml
services:
  backend:
    volumes:
      # Bind mount (host path : container path)
      - ./src:/app/src

      # Named volume
      - app-data:/app/data

      # Read-only bind mount
      - ./config:/app/config:ro

  database:
    volumes:
      - db-data:/var/lib/mysql

volumes:
  app-data:        # Docker manages this
  db-data:
    driver: local  # explicit driver (local is default)
```

### Advanced Compose Features

```yaml
version: "3.9"

services:
  backend:
    image: my-backend
    
    # Health check — is the service actually ready?
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

    # Resource limits
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          memory: 256M

    # Depends on mongodb AND waits for it to be healthy
    depends_on:
      mongodb:
        condition: service_healthy

  mongodb:
    image: mongo:6.0
    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
      interval: 10s
      timeout: 5s
      retries: 5
```

---

### ❓ Cross Questions

**Q: What's the difference between `docker compose stop` and `docker compose down`?**  
A: `stop` only stops the containers (like turning off the power) but keeps them. `down` stops AND removes the containers and networks. `down -v` also removes volumes (deletes data!). Think: `stop` = pause, `down` = clean up.

**Q: Does `depends_on` wait for the service to be READY?**  
A: By default, `depends_on` only waits for the container to *start*, not for the service inside to be *ready*. MongoDB might start its container but need 5 more seconds to be accepting connections. Use `healthcheck` + `condition: service_healthy` to wait for actual readiness.

**Q: Can I use Docker Compose in production?**  
A: Docker Compose is great for development and small production deployments on a single server. For multi-server, high-availability production, use Kubernetes or Docker Swarm instead. Compose doesn't handle multiple servers.

**Q: What's the difference between Docker Compose v1 (`docker-compose`) and v2 (`docker compose`)?**  
A: v1 is the old standalone Python tool (`docker-compose` with hyphen). v2 is the newer Go-based plugin integrated directly into Docker CLI (`docker compose` with space). v2 is faster and the current standard. Use `docker compose` (no hyphen).

---

## 27. Full Summary + Interview Questions

### The Docker Mental Model — Everything Together

```
Dockerfile
    │
    │  docker build -t myapp:v1 .
    ▼
Docker Image  ──── docker push ──►  Docker Registry (Hub)
    │                                      │
    │  docker run                          │  docker pull
    ▼                                      ▼
Container ◄──────────────────────── Image on another machine
    │
    ├── Ports (-p) ──────────────► Accessible from host/internet
    ├── Volumes (-v) ────────────► Persistent data on host
    ├── Networks ────────────────► Talk to other containers
    └── Env vars (-e) ───────────► Runtime configuration


docker-compose.yml
    │
    │  docker compose up -d
    ▼
Multiple containers + networks + volumes
started together, configured together
```

---

### Key Concepts Summary Table

| Concept | One-Line Summary |
|---------|-----------------|
| **Container** | An isolated process with its own filesystem, network, and process space |
| **Image** | A read-only layered template for creating containers |
| **Dockerfile** | A script of instructions to build an image |
| **Docker Hub** | A public registry hosting millions of Docker images |
| **Volume** | Persistent storage that survives container lifecycle |
| **Port mapping** | Connecting host machine ports to container ports |
| **Env variable** | Runtime configuration passed to containers |
| **Docker Compose** | Tool to run multi-container apps from one YAML file |
| **Bridge network** | Virtual network for containers to communicate |
| **Layer caching** | Reusing unchanged build steps to speed up builds |
| **CMD** | Default command a container runs on start |
| **ENTRYPOINT** | Fixed executable that always runs in a container |

---

### Interview Questions — Common + Tricky

**Beginner Level:**

**Q1: What problem does Docker solve?**  
A: The "it works on my machine" problem. Docker packages an app and all its dependencies into a container that runs identically on any machine — dev laptop, CI server, production. It eliminates environment inconsistencies.

**Q2: What's the difference between a Docker image and a container?**  
A: An image is a read-only blueprint (like a class in OOP). A container is a running instance of that image (like an object). You can create many containers from one image.

**Q3: What is Docker Hub?**  
A: Docker Hub is the default public registry for Docker images. It hosts official images (nginx, mysql, python) and community-contributed images. You can push your own images there too.

**Q4: How is Docker different from a Virtual Machine?**  
A: VMs run a full OS copy per VM (GBs, slow). Containers share the host OS kernel (MBs, fast). VMs use a hypervisor; Docker uses namespaces and cgroups for isolation. VMs are stronger isolation; containers are lighter and faster.

---

**Intermediate Level:**

**Q5: What are Docker layers and why do they matter?**  
A: Images are built as stacked read-only layers. Each Dockerfile instruction creates a layer. Layers are cached and shared between images. This saves disk space and speeds up builds (only changed layers are rebuilt).

**Q6: How do you persist data in Docker?**  
A: Using volumes. Data written inside a container's writable layer is lost when the container is deleted. Volumes store data on the host, outside the container's lifecycle. Use `docker volume create` or `-v volume-name:/path/in/container`.

**Q7: How does container networking work?**  
A: Docker creates a virtual bridge network. Containers on the same custom bridge network can communicate using container names as DNS hostnames. By default, containers are isolated. Port mapping (`-p`) exposes container ports to the host.

**Q8: What is the purpose of `.dockerignore`?**  
A: Like `.gitignore`, it excludes files from the Docker build context. This prevents sending unnecessary files (like `node_modules`, `.git`, `.env`) to the Docker daemon, keeping builds fast and images lean.

**Q9: Explain the CMD vs ENTRYPOINT difference.**  
A: `CMD` sets the default command run when a container starts; it can be overridden by passing a command to `docker run`. `ENTRYPOINT` sets the fixed executable; it cannot be overridden without `--entrypoint`. Common pattern: ENTRYPOINT as the executable, CMD as default arguments.

---

**Advanced Level:**

**Q10: What happens when you run `docker run nginx`?**  
A: 
1. Docker client receives the command and sends API request to Docker daemon
2. Daemon checks if `nginx:latest` image exists locally
3. If not, pulls it from Docker Hub
4. Daemon creates a new container: sets up namespace isolation, writable layer, virtual network interface
5. Daemon starts the container process using `runc` (via containerd)
6. nginx's CMD (`nginx -g "daemon off;"`) runs as PID 1 inside the container

**Q11: Why should you not run applications as root inside containers?**  
A: If a container is compromised and the app runs as root, an attacker might exploit vulnerabilities to break out to the host. Running as a non-root user limits the blast radius. Use `USER` instruction in Dockerfile:
```dockerfile
RUN useradd -r -s /bin/false appuser
USER appuser
```

**Q12: What is a multi-stage Dockerfile and when would you use it?**  
A: Multi-stage builds use multiple `FROM` statements in one Dockerfile. Early stages do heavy work (compiling, installing build tools). Later stages copy only the artifacts needed. Final image is minimal — no build tools, no source code. Reduces production image size from GB to MB, reduces attack surface.

**Q13: How does Docker Compose handle service dependencies?**  
A: `depends_on` tells Compose to start services in order. But it only waits for the container to start, not for the application inside to be ready. For true readiness, use `healthcheck` in combination with `condition: service_healthy` in `depends_on`.

**Q14: What's the difference between Docker Compose and Kubernetes?**  
A: Docker Compose manages containers on a **single host**. Kubernetes orchestrates containers across **many hosts (a cluster)**, handling auto-scaling, self-healing, rolling updates, load balancing, and more. Compose is for dev and simple deployments; Kubernetes is for large-scale production.

---

### Real-Life Project: Full Stack App in Docker

Here's a complete `docker-compose.yml` for a real-world MERN (MongoDB, Express, React, Node) stack:

```yaml
version: "3.9"

services:

  frontend:
    build: ./frontend
    container_name: react-app
    ports:
      - "3000:3000"
    volumes:
      - ./frontend/src:/app/src   # hot reload
    environment:
      - REACT_APP_API_URL=http://localhost:5000
    networks:
      - app-net
    depends_on:
      - backend

  backend:
    build: ./backend
    container_name: node-api
    ports:
      - "5000:5000"
    volumes:
      - ./backend:/app
      - /app/node_modules           # don't override node_modules from host
    environment:
      - MONGODB_URI=mongodb://admin:pass@mongodb:27017/mydb?authSource=admin
      - JWT_SECRET=supersecretkey
      - NODE_ENV=development
    networks:
      - app-net
    depends_on:
      mongodb:
        condition: service_healthy
    restart: unless-stopped

  mongodb:
    image: mongo:6.0
    container_name: mongodb
    ports:
      - "27017:27017"
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=pass
      - MONGO_INITDB_DATABASE=mydb
    volumes:
      - mongo-data:/data/db
    networks:
      - app-net
    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    container_name: redis-cache
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    networks:
      - app-net
    restart: unless-stopped

volumes:
  mongo-data:
  redis-data:

networks:
  app-net:
    driver: bridge
```

**Start everything:**
```bash
docker compose up -d --build
```

**That's it. One command. Your entire full-stack development environment is running.**

---

### The Big Picture — Docker in the Modern Tech World

```
Developer writes code
        │
        ▼
Dockerfile describes the environment
        │
        ▼
docker build → creates Docker Image
        │
        ▼
Image pushed to Docker Hub / private registry
        │
        ▼
CI/CD pipeline (GitHub Actions, Jenkins)
pulls image, runs tests in containers
        │
        ▼
Passes? Deploy to production
        │
   ┌────┴──────────────┐
   ▼                   ▼
Single server       Kubernetes cluster
docker compose up   Manages 100s of containers
                    across many servers
                    Auto-scaling, self-healing
```

Docker is not just a tool — it's the foundation of how modern software is built, tested, and deployed.

---

*🎉 Congratulations! You now understand Docker from the very basics to building full multi-container applications. The best way to solidify this knowledge is to practice — spin up containers, write Dockerfiles, break things, fix them. That's how you really learn Docker.*
