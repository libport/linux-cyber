# Docker Basic Concepts and Configuration
## Containers and images
Docker packages applications as isolated processes that share the host kernel. A container is not a small virtual server. It is a runtime environment created from an image, with filesystem layers, settings and resource controls. This model lets teams run many workloads on one host, start them quickly, and move them between environments with fewer differences.

An image is a read-only template for creating containers. Docker downloads images from registries such as Docker Hub, unless a matching image or layer already exists locally. Layer reuse saves time and bandwidth. Repositories group related images and tags.

A web-server container can map host port 8080 to container port 80 with `-p 8080:80`. External clients connect to the host on port 8080, and Docker forwards traffic to the service on port 80 inside the container. The `-d` flag runs the container in detached mode, and `--name` assigns a stable name.
## Docker Engine and the daemon
Docker Engine uses a client-server architecture. The Docker command-line interface and other tools call the Docker API, and the daemon, `dockerd`, manages Docker objects such as images, containers, networks and volumes. On macOS and Windows, Docker Desktop runs Linux containers inside a managed Linux virtual machine, while Windows can also run native Windows containers in the supported mode.

Operators inspect the environment with `docker info`. The output shows storage, logging, networking, plug-ins and security settings. The daemon configuration normally lives at `/etc/docker/daemon.json` on Linux. Docker reads this JSON file at start-up, so invalid JSON can stop the service from starting. Changes usually require `systemctl restart docker`.
## Installing Docker on Linux
Official Docker packages usually provide the clearest installation path for current Docker Engine features. A careful installation removes old packages first, adds Docker's package repository and signing key, then installs Docker Engine, the CLI, containerd, Buildx and Compose. Distro packages such as `docker.io` may work, but they can lag behind Docker's current feature set.

Adding a user to the `docker` group removes the need for `sudo`, but it also grants powerful access to the daemon. That access can become root-equivalent on the host. Production systems should restrict Docker group membership and protect the daemon socket.
## Building workloads with a Dockerfile
A Dockerfile defines image build instructions in plain text. A simple Apache image might start from `ubuntu:24.04`, set non-interactive package installation, install `apache2`, copy a local `index.html` into `/var/www/html`, and start Apache in the foreground.

Pinning the base image to a version such as `ubuntu:24.04` improves reproducibility compared with relying on `latest`. The `EXPOSE 80` instruction documents that the container listens on port 80, but it does not publish the port to the host. Publishing happens when a container starts with `-p` or related run options.

Buildx builds the image with `docker buildx build -t my-server .`. The final dot sets the current directory as the build context. If the tag omits a version, Docker treats it as `latest`. A container can then run from that image with `docker run -d -p 8080:80 my-server`.
## Operating and customising containers
`docker ps` lists running containers. `docker ps -a` also shows stopped containers. Docker displays a container ID, image, command, creation time, status, port mappings and name. `docker stop` stops a running container.

Container logs are available with `docker logs <container>`. Docker daemon logs diagnose Docker service problems rather than application problems inside a container. On many Linux systems, daemon logs are available through `journalctl -xu docker.service`.

Docker supports several network drivers. Bridge networks let containers on one host communicate while remaining isolated from external networks unless ports are published. Host networking shares the host network stack. Overlay networking connects containers across multiple Docker hosts and supports clusters.

Storage and logging settings can be customised in `daemon.json`. Linux systems commonly use the `overlay2` storage driver, while logging commonly uses `json-file` unless configured otherwise. Other logging drivers can send container output to syslog, journald, AWS CloudWatch or Splunk.
## Orchestration options
Single containers suit small demonstrations and simple services. Larger applications often need multiple services, networks and volumes. Docker Compose defines and runs multi-container applications from a YAML file. Docker Swarm manages clusters of Docker Engines. Kubernetes, now a Cloud Native Computing Foundation project, orchestrates containerised applications at larger scale and automates deployment, scaling and management.