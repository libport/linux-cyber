# Docker Basic Concepts and Configuration
> [!NOTE]
> This guide introduces Docker’s core architecture, installation, image-building, container operations, networking, storage, security, and orchestration options.
## Containers, images, and Docker Engine
Docker packages an application and its dependencies in an isolated container. A container is not a virtual server with its own kernel. It runs as one or more isolated processes and shares a compatible host kernel. Linux namespaces separate resources, while control groups limit CPU, memory, and other resource use. This design usually gives containers less overhead than virtual machines.

An image is an immutable, layered template. A container is a runnable instance of an image plus runtime settings, networking, and storage. A Dockerfile provides reproducible build instructions, but neither an image nor a container is simply a script. A registry stores and distributes images. Docker Hub offers public, private, official, verified, and community repositories.

Docker Engine uses a client-server architecture. The `docker` command sends API requests to the Docker daemon, `dockerd`, which manages images, containers, networks, and volumes and delegates low-level execution to a container runtime. Docker Desktop supplies the required Linux environment for Linux containers on macOS and Windows, using a virtual machine or WSL 2. Native Windows containers instead share the Windows kernel.

The following command starts a detached web container and maps host port 8080 to container port 80:

```bash
docker run -d --name app0 -p 8080:80 IMAGE
```

Docker uses a local image when available and otherwise pulls missing, reusable layers from a registry. A client can then request `http://localhost:8080`. Port publishing through `-p` makes the service reachable. The Dockerfile instruction `EXPOSE 80` only documents the intended container port.
## Installation on Ubuntu
Docker's official Ubuntu instructions remove conflicting packages, configure Docker's signed `apt` repository, and install Docker Engine, the CLI, containerd, Buildx, and Compose. Distribution packages such as `docker.io` can follow different release schedules. Administrators should follow the current official procedure and enable the service at boot when the installation has not already done so.

Docker commands normally require `sudo` because the daemon socket belongs to `root`. Adding a user with `sudo usermod -aG docker USERNAME` removes that inconvenience after a new login, but membership grants root-level privileges. Rootless mode provides a safer alternative when its limitations are acceptable.
## Building and operating a workload
A compact Apache image can use this Dockerfile:

```dockerfile
FROM ubuntu:24.04
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y apache2 \
    && rm -rf /var/lib/apt/lists/*
COPY index.html /var/www/html/index.html
EXPOSE 80
CMD ["apachectl", "-D", "FOREGROUND"]
```

`docker build -t myserver:1.0 .` builds from the Dockerfile and local build context. On supported Linux builds, `docker build` uses Buildx and BuildKit by default. `docker buildx build` also supports selected builders, platforms, and output types. `docker run -d --name web -p 8080:80 myserver:1.0` launches the image. `docker ps` lists running containers, `docker ps -a` includes stopped containers, `docker logs web` displays container output, and `docker stop web` stops the process.
## Configuration, storage, and orchestration
`docker info` reports current engine, security, logging, network, and storage settings. A manual Linux Engine installation reads daemon settings from `/etc/docker/daemon.json`. Docker Desktop exposes engine settings through its dashboard. After validating changed JSON, an administrator restarts Docker. New logging defaults apply only to subsequently created containers.

Docker defaults to the `json-file` logging driver, while Docker recommends the rotating `local` driver for many general workloads. Other integrations include `journald`, syslog, CloudWatch, and Splunk. `docker logs` reads supported container logs, while `journalctl -u docker` helps diagnose the Linux daemon.

Bridge networks connect and isolate containers on one host. Host networking removes that network isolation. Overlay networks connect participating Docker daemons, principally for Swarm services. Fresh Docker Engine 29 installations use the containerd image store by default. Upgraded systems can retain the classic `overlay2` driver.

Docker Compose defines services, networks, and volumes for a multi-container application in YAML. Docker Swarm orchestrates Docker services across hosts, while Kubernetes manages larger containerised systems through its own platform and networking model.