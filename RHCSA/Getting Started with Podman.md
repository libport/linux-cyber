# Getting Started with Podman
> [!NOTE]
> This file is a practical introduction to Podman that explains container isolation, image and container lifecycle management, networking, storage, systemd integration, custom image building, Compose-based multi-container labs, and the operational habits that keep container workflows predictable.
## Podman, container isolation and the host environment
Containers isolate workloads, but the isolation comes from the kernel rather than the container engine. Podman exposes those Linux primitives in a consistent interface. A process inside one container sees a limited view of the system, its own process tree, its own network interfaces and its own filesystem content. That isolation keeps applications from colliding over package versions, ports, libraries or service state.

The `unshare` command demonstrates the underlying model without any container tooling. A shell launched in a new PID namespace sees its own process tree, often with the shell itself as process 1 in that namespace. A shell launched in a new network namespace sees only the interfaces configured inside that namespace. This is the same class of separation that containers use every day. Podman simply manages those namespaces, the filesystem layers and the container metadata in a practical workflow.

A simple demonstration shows the idea:

```bash
sudo unshare --fork --pid --mount-proc bash
ps
exit

sudo unshare --fork --pid --mount-proc --net bash
ip addr
exit
```

The first shell creates a separate PID namespace. The second adds an isolated network namespace. These examples explain why containers can run conflicting software stacks on the same host. A Java 8 workload and a Java 17 workload can coexist in separate containers without forcing the host to choose one runtime. The same logic applies to Python stacks, web servers and test dependencies.

Podman extends that kernel-level isolation with a user-focused workflow. It supports rootless execution for many everyday operations. A regular user can often pull images, start containers, inspect metadata and manage runtime state without joining a privileged group or using a daemon controlled by another account. That rootless model matters in shared environments because it narrows the attack surface and aligns container work with normal user permissions.

The host still matters. Storage, network configuration, cgroup support and security policy shape what the containers can do. A Red Hat compatible system with SELinux enabled brings extra protections that affect bind mounts, service supervision and systemd-in-container scenarios. Those protections should stay enabled. The correct fix is to label or relabel mounted content properly, or to use the appropriate mount flags, rather than disabling SELinux.

Container work also benefits from disciplined naming. The host should treat containers as identifiable services rather than as anonymous test artefacts. Names such as `controller`, `ubuntu` and `www` communicate the service role immediately. Hostnames inside the container matter as well when an operator attaches to an interactive shell or uses SSH to enter a container for troubleshooting.
## Installing Podman and validating the environment
Podman installation is straightforward on current Linux distributions, especially on Red Hat compatible systems. The engine itself usually comes from the distribution repositories. A clean install should confirm the command, version and host details before any further work begins.

Typical validation looks like this:

```bash
podman --version
podman info
podman image ls
podman container ls -a
```

`podman --version` confirms that the command is present. `podman info` reports storage drivers, registries, cgroup settings, rootless details and host capabilities. The image and container listings confirm that the local storage starts either empty or in a known state.

Per-user storage is an important detail. Podman keeps local images and containers in storage associated with the current user. A regular user and root do not automatically share the same image cache. An image pulled by an unprivileged account may need to be pulled again when root manages a system-level service. This is not a defect. It is a direct result of separate storage and permissions.

Older material often installs `podman-compose` from an extra repository or with `pip`. Current Podman documentation describes `podman compose` as a wrapper around an external Compose provider such as `podman-compose` or `docker-compose`. In practice, Compose-style workflows still work well, but the provider model should be understood clearly. Compose support does not mean that Podman has absorbed the entire implementation natively.

The host should also include a few supporting tools. `git` is useful for pulling example repositories and lab files. `skopeo` inspects remote image metadata without downloading full images. Standard shell tools such as `tree`, `less`, `ss`, `curl` and `ip` make inspection easier. On Red Hat compatible systems, the package manager may appear as `dnf` even where `yum` remains as a compatibility command. Either path leads to the same underlying package management on current releases.

A consistent host setup reduces noise later. A shell-only system, a known Podman version, working DNS and a clean home directory make the container lifecycle easier to read. That matters because the most useful Podman habits come from observing what each command changes. Installing the engine is not the point. Establishing a predictable platform for images, containers, networks and volume mounts is the real foundation.
## Images, registries and metadata
Images are the source material for containers. They package a root filesystem plus configuration such as default commands, environment variables, entrypoints, labels and exposed ports. Podman stores those images locally after pulling them from a registry, then creates containers as writable runtime instances on top of them.

A minimal image workflow includes search, pull and inspect:

```bash
podman search ubuntu
podman pull docker.io/library/ubuntu:20.04
podman image ls
podman image inspect docker.io/library/ubuntu:20.04
```

`podman search` helps identify repository names. `podman pull` retrieves a specific image and tag. `podman image ls` shows what now exists locally. `podman image inspect` exposes the JSON metadata that controls how the image behaves when a container starts. This is where the default command, environment, architecture and labels become visible.

The distinction between unqualified and fully qualified image names matters. A short name such as `ubuntu` may resolve through registry configuration and local short-name aliases. That is convenient, but explicit names such as `docker.io/library/ubuntu:20.04` are safer in shared or automated environments because they document exactly which registry and tag supply the image. Reproducibility improves as ambiguity falls.

Podman also preserves compatibility with older command styles. `podman images` still works, but `podman image ls` is clearer because it places the action under the image object. The same pattern appears across the CLI with `podman container ls`, `podman network ls` and `podman volume ls`. The older aliases remain useful for quick work, yet the newer form is easier to read in documentation and scripts.

Remote inspection is where `skopeo` becomes valuable. It can inspect a registry reference without pulling the full image, which is ideal when tag selection matters:

```bash
skopeo inspect docker://docker.io/library/ubuntu:latest
skopeo inspect --format '{{.RepoTags}}' docker://docker.io/library/ubuntu
```

This exposes available tags, metadata and image details directly from the registry. A lab that depends on Ubuntu 20.04 or Fedora 38 should pin those tags explicitly rather than accepting `latest`, which will shift over time. Stable tags make test results easier to reproduce and reduce the risk that a later package base breaks an automation exercise.

Images also show how little content a container may need. A tiny demonstration image can be only kilobytes in size. A fuller Fedora or Ubuntu base image is still far smaller than a virtual machine. That scale difference explains why containers are useful for short-lived labs. The host can download multiple operating system bases quickly and keep them on hand without the cost of multiple disk-heavy guests.

Inspection is also the start of trust and verification. The metadata reveals whether an image runs `bash`, a web server, an init process or a custom binary. It reveals environment defaults such as `TZ`. It reveals exposed ports and labels. Container work becomes more reliable once the image metadata is treated as a first-class part of the design rather than as an afterthought.
## Creating, naming and managing containers
A container is a runtime instance of an image with its own writable layer and metadata. It may start in the foreground for interactive work or in detached mode for service-style operation. The lifecycle remains simple once the key commands are understood.

An interactive container starts like this:

```bash
podman run -it --name ubuntu --hostname ubuntu docker.io/library/ubuntu:20.04
cat /etc/os-release
exit
```

`-it` gives the container a terminal. `--name` assigns a meaningful container name. `--hostname` sets the shell prompt and the internal hostname. The container remains available after exit unless `--rm` is added. This distinction matters. A stopped container can be started again and retains changes made to its writable layer.

Disposable tests benefit from `--rm`:

```bash
podman run --rm -it --name scratch docker.io/library/ubuntu:20.04
```

Once the main process exits, Podman removes the container automatically. This keeps the environment tidy and allows the same name to be reused immediately. It is ideal for quick checks and short-lived command-line experiments.

Detached operation suits persistent services and longer sessions:

```bash
podman run -dit --name ubuntu --hostname ubuntu docker.io/library/ubuntu:20.04
podman ps
podman attach ubuntu
```

The container stays running in the background. `podman attach` reconnects to the main process. If the main process is an interactive shell, leaving with `exit` stops the container because the main process ends. Leaving with the detach sequence preserves the running container. A safer day-to-day pattern is to use `podman exec` for ad hoc shell access into a service container while leaving the main process undisturbed.

Operational commands follow the lifecycle directly:

```bash
podman stop ubuntu
podman start ubuntu
podman rm ubuntu
podman container prune -f
```

`stop` ends the running workload cleanly. `start` restarts a created or stopped container. `rm` removes the container metadata and writable layer. `podman container prune -f` removes all stopped containers at once. This is useful after a test run, especially when many named containers accumulate.

Inspection and monitoring complete the picture:

```bash
podman top ubuntu
podman logs ubuntu
podman inspect ubuntu
```

`podman top` shows the processes running inside the container. In a microservice-style container, this is often only one main process plus a small number of helpers. `podman logs` shows the container output, which is essential for troubleshooting service startup. `podman inspect` exposes runtime configuration, networks, mounts, state and other metadata in JSON.

These commands demonstrate a crucial point. A container is not just an image plus a process. It is a managed object with state, mounts, networks, name resolution, security labels and lifecycle events. Treating containers as first-class managed services, rather than as disposable black boxes, makes Podman much easier to operate at scale.
## Running services, publishing ports and persisting content
Service containers turn Podman from a shell tool into an application platform. A web server is the clearest example because it combines image selection, environment variables, port publishing, logs and mounted content in one small workflow.

A typical service start command looks like this:

```bash
podman pull docker.io/library/httpd:latest
podman run -d --name www -p 8080:80 docker.io/library/httpd:latest
curl http://localhost:8080
```

The published port maps host port 8080 to port 80 in the container. This is host-to-container access. The host keeps its own network namespace and forwards traffic into the isolated service. The container does not need a public IP address on the host network to be reachable. A browser, `curl` or a command-line client can test the result immediately.

Environment variables customise the service without rebuilding the image. Time zone settings are a common example because logs and application output should match the environment where the service runs. Container images often expose such settings in their metadata, which makes inspection useful before deployment.

A running container can also be inspected or entered without interrupting the main service:

```bash
podman exec -it www /bin/sh
podman logs www
ss -tln
```

`podman exec` starts an additional process inside the container. This is generally preferable to `attach` for service containers because it avoids interacting with the main process directly. `podman logs` shows the service output. `ss -tln` confirms that the host is listening on the published port.

Static or local content should not live only inside the container layer if the service needs to be updated or replaced. Bind mounts and named volumes keep data outside the container lifecycle. A bind mount for web content might look like this:

```bash
podman run -d --name www \
  -p 8080:80 \
  -v /home/tux/podman/www:/usr/local/apache2/htdocs:Z \
  docker.io/library/httpd:latest
```

The host directory becomes visible inside the container at the web root. Updating files on the host updates what the service serves. Replacing the container does not destroy the content. The `:Z` suffix requests a private SELinux relabel for the mount. `:z` requests a shared label when multiple containers need the same content. On SELinux-enabled hosts, these flags are usually safer and more maintainable than disabling SELinux or leaving the labels unchanged.

Named volumes solve a related problem for service data that should not depend on a specific host path:

```bash
podman volume create webdata
podman run -d --name www -p 8080:80 -v webdata:/usr/local/apache2/htdocs docker.io/library/httpd:latest
```

The volume lifecycle is separate from the container. A recreated container can mount the same volume and recover the data. This is suitable for service state, caches and content that belongs to the application rather than to the host filesystem layout.

The larger lesson is that a container service is a combination of image, process, published ports, environment and data mounts. Podman keeps these concerns separate, which makes the service easier to understand and replace. The host stays clean. The data stays durable. The service remains reproducible.
## Service supervision and systemd integration
Long-running containers need supervision. Podman supports integration with systemd so that the host can start, stop and restart containers as services. Earlier workflows often used `podman generate systemd` to emit a service file from an existing container. That command still exists, but current Podman documentation recommends Quadlet for new systemd-based container units.

A generated-unit workflow looks like this:

```bash
podman run -d --name www -p 8080:80 docker.io/library/httpd:latest
podman generate systemd --name www > ~/.config/systemd/user/www.service
systemctl --user daemon-reload
systemctl --user enable --now www.service
```

This approach is simple and still effective for basic setups. It converts an existing container definition into a systemd unit, reloads the user manager and enables the service. The host then supervises the container in the same way it supervises other user services.

Quadlet moves the definition closer to a declarative unit model. Instead of generating a unit from an already created container, a `.container` file describes the image, ports, volumes, restart behaviour and other options. Systemd then generates the concrete unit at reload time. This reduces drift between a hand-maintained unit file and the original container definition.

The choice between system services and user services depends on the workload. Rootless user units suit many development and lab cases because they align with unprivileged Podman usage. System-wide units may be appropriate when the service must start before login or when it needs system-managed resources. The important point is consistency. The same service should not be managed manually with ad hoc `podman stop` and `podman start` commands while systemd also believes it owns the lifecycle.

Root and non-root execution also affect image storage. A root-managed unit may need to pull its own copy of the image because root and the regular account keep separate local stores. This often surprises first-time users. It is normal behaviour and should be accounted for in the deployment flow.

When Podman services expose host ports, systemd integration becomes especially useful because it prevents manual restarts and port collisions. A supervised web service can be enabled on boot, restarted automatically after failure and inspected with standard systemd tools. The combination of `journalctl`, `systemctl status` and `podman logs` creates a complete troubleshooting path.

Systemd integration also frames an important design choice. A normal container does not need a full init system. One main process is usually enough and remains the simplest model. Systemd on the host should supervise the container, not necessarily live inside it. Running systemd inside a container is a specialised pattern for multi-service labs and test environments, not the default architecture for production application containers.
## Building custom images for multi-service test labs
Single-process images suit most services, but some labs need a fuller userspace. Configuration management tests often require SSH access, a non-root user, Python and a service manager. In that case, a custom image is appropriate.

`podman build` accepts both `Dockerfile` and `Containerfile` syntax. A structured build directory might include a base file, an SSH public key and a sudoers fragment. A Fedora-based controller image can install tools such as `systemd`, `openssh-server`, `sudo`, `python3`, `vim` and `ansible`. An Ubuntu target image can install the equivalent packages with the distribution's own package manager.

A simplified build process looks like this:

```bash
podman build -t localhost/fedora-controller -f Dockerfile .
podman build -t localhost/ubuntu-target -f Dockerfile .
podman image ls
```

The Dockerfile or Containerfile defines the whole image. Typical steps include:
- selecting a pinned base image such as `fedora:38` or `ubuntu:20.04`
- installing required packages
- creating a non-root user
- configuring passwordless sudo where appropriate for automation
- copying the authorised public key into the image
- enabling or defining the services needed in the lab
- setting the default command or entrypoint

This image design makes the test environment self-contained. The host does not need to install Ansible, SSH servers or the target packages directly. Rebuilding the image recreates the environment cleanly.

A multi-service image needs more care than a microservice image. Systemd-in-container setups require access to cgroup management and the correct SELinux or mount configuration. On Red Hat compatible hosts, enabling the relevant SELinux boolean may be necessary for certain systemd and cgroup operations in containers. The exact requirement depends on the host policy and the container design, but the core idea remains the same. The host security model should be configured correctly rather than bypassed.

SSH access is convenient for automation tests because tools such as Ansible can treat the container like a small remote host. A controller image can expose SSH on a published port and accept the copied public key. A run command might look like this:

```bash
podman run -d \
  --name controller \
  --hostname controller \
  -p 2222:22 \
  localhost/fedora-controller
```

The published port allows host-to-container SSH access. Similar target containers can expose different host ports or communicate through a shared Podman network. Once running, the images behave like lightweight lab nodes rather than like anonymous application wrappers.

This pattern departs from pure microservice design, but it is justified in a teaching or testing context. The goal is not to build the smallest production image. The goal is to create disposable systems that resemble managed Linux hosts closely enough to validate automation, SSH connectivity and service configuration.
## Networking and inter-container communication
Podman isolates container networking by default, but most useful labs need controlled communication. Published ports solve host-to-container access. User-defined networks solve container-to-container access.

Pods offer one model. Containers inside the same pod share a network namespace and can communicate over localhost. That is useful for tightly coupled components such as a web front end and a sidecar process. Separate containers that should behave like distinct hosts need a different model. A user-defined network gives them separate identities while still permitting direct communication.

A simple network creation sequence looks like this:

```bash
podman network create my-net
podman network ls
podman run -d --name controller --network my-net localhost/fedora-controller
podman run -d --name ubuntu --network my-net localhost/ubuntu-target
```

Podman creates a bridge-style network and assigns addressing details unless the command specifies explicit subnets or gateways. Once both containers join the same network, they can often resolve each other by container name and exchange traffic directly. This is the basis for a multi-node lab on a single machine.

Inspection confirms the result:

```bash
podman inspect controller
podman inspect ubuntu
```

The output reveals network attachments, assigned addresses and other runtime metadata. Published ports still serve a distinct purpose. A controller container may join `my-net` for container-to-container traffic while also publishing `2222:22` so the host can reach its SSH service. These two paths should not be confused. One is internal network membership. The other is explicit host port publishing.

Custom networks improve realism. Separate services can communicate by name rather than through host loopback hacks. A controller can reach `ubuntu` by container name over the shared network. That makes inventory files, service definitions and troubleshooting output clearer. It also keeps the lab closer to real distributed systems behaviour.

Networks are also disposable. Removing the containers does not necessarily remove the network unless the operator chooses to do so. This separation mirrors the broader Podman design. Images, containers, networks and volumes are related but distinct resources. That modular model makes cleanup and recreation straightforward:

```bash
podman rm -f controller ubuntu
podman network rm my-net
```

Good network design starts simple. One named network for the lab is often enough. Extra networks only help when there is a real need to separate traffic classes. The main value lies in predictability, name resolution and repeatable connectivity between otherwise isolated workloads.
## Compose workflows and an Ansible lab
Once several containers, volumes, networks and build contexts exist, manual commands become noisy. Compose solves that by describing the desired state in YAML. Podman supports Compose workflows through `podman compose`, which delegates to an external Compose provider. The YAML still expresses the design clearly and remains the most effective way to stand up and tear down a multi-container lab reliably.

A Compose file for this type of lab usually defines:
- a controller service built from a Fedora-based context
- an Ubuntu target service built from an Ubuntu-based context
- published ports for SSH access where needed
- a shared network for inter-container communication
- a bind mount that exposes the host playbooks directory inside the controller
- hostnames and container names that match the service roles

The build contexts point at the relevant Dockerfiles or Containerfiles. The bind mount exposes automation assets directly to the controller. The shared network gives the controller and the Ubuntu target predictable name resolution. One command then creates the full lab:

```bash
podman compose build
podman compose up -d
podman compose ps
```

Stopping or removing the stack is just as simple:

```bash
podman compose stop
podman compose start
podman compose down
```

This declarative model is the natural culmination of the earlier sections. Images define base systems. Containers define runtime state. Networks define connectivity. Volumes define persistence. Compose pulls those pieces into a repeatable bundle.

SELinux remains relevant here. A bind-mounted playbooks directory on a Red Hat compatible host needs a container-readable label. That can be handled with relabel flags in the volume definition or by prelabelling the directory. Either approach is valid as long as the policy remains enforced and the container can read the files it needs.

The Ansible demonstration shows why the lab matters. The controller container can hold `ansible.cfg`, an inventory file, variables and playbooks in a mounted directory. From inside the controller, Ansible can connect to the Ubuntu target over the shared network or published SSH port, gather facts, install packages and validate idempotence. The environment stays disposable. If the lab drifts or fails, rebuilding the images and recreating the services returns it to a known state.

A compact sequence inside the controller might include:

```bash
cd /playbooks
ansible all -m ping
ansible all -b -m package -a "name=httpd state=present"
ansible-playbook site.yml
```

The exact package names vary by distribution, but the principle is stable. Containers stand in for managed hosts. The controller applies configuration repeatedly until the target reaches the desired state. Re-running the same playbook should converge cleanly. This demonstrates both Podman's value as a lab substrate and Ansible's value as a configuration tool.

Cleanup matters once the exercise ends. Podman can remove stopped containers, unused images, networks and build cache through pruning commands. A careful cleanup returns the host to a lean state without manual hunting:

```bash
podman system prune -a
```

Used thoughtfully, this keeps the local machine ready for the next lab without losing the conceptual clarity of the workflow.
## Working habits that keep Podman predictable
A few operating habits make the platform far easier to manage. The first is pinning versions. Tags such as `latest` are convenient for quick experiments, but they undermine repeatability because the base system changes silently over time. Labs, demos and automation tests should name the exact image tag, or better, the exact digest when strict reproducibility matters. This applies equally to base images, service images and custom build stages.

The second habit is treating inspection as part of normal work. `podman inspect`, `podman image inspect`, `podman logs` and `podman info` answer most troubleshooting questions before more invasive debugging starts. They reveal whether the container used the right image, mounted the correct paths, joined the expected networks and published the intended ports. Container work becomes much less mysterious once this metadata is checked early rather than only after failure.

The third habit is separating disposable state from persistent state. Anything that must survive container replacement belongs in a bind mount, named volume, Compose definition or rebuildable image, not in the writable container layer alone. The writable layer is excellent for experimentation and short-term changes, but it is the least durable place to keep content that matters.

The fourth habit is using names that reflect intent. `www`, `controller` and `ubuntu` are easier to understand than auto-generated names, and hostnames should reflect the same role. This improves logs, prompt output, Compose files and Ansible inventories. Clear naming is a small step that pays off throughout the whole lifecycle.

The fifth habit is preferring declarative definitions as the lab grows. A single `podman run` command is perfect for a quick test. Several services with mounts, networks, hostnames and build contexts belong in a Compose file or a systemd unit. The more state moves into files, the easier it becomes to rebuild the environment, review changes and share the setup with other operators.