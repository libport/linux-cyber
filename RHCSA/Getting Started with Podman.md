# Getting Started with Podman
> [!NOTE]
> A practical guide to using Podman for secure, reproducible container workflows, covering images, services, storage, networking, systemd integration, custom builds, Compose, and multi-container automation labs.
## Podman and Linux containers
Podman builds, runs, and manages Open Container Initiative containers and images. It offers a command-line interface familiar to Docker users, but its local Linux workflow does not depend on a central, permanently running daemon. An optional API service supports remote clients and Compose providers. Podman can also run as an unprivileged user, which reduces the authority available to a compromised container process.

A container packages an application with its user-space libraries, configuration, and executable files. It shares the host kernel rather than booting a separate kernel, so it usually starts faster and consumes fewer resources than a virtual machine. An image supplies immutable layers and configuration. A container adds a writable layer and runs one or more processes from that image.

Containers provide isolation, not an absolute security boundary. Linux namespaces separate resources such as process IDs, mount points, network interfaces, hostnames, interprocess communication, and user IDs. Control groups account for and control CPU, memory, process, and input-output use. Podman also applies mechanisms such as capabilities, seccomp, SELinux, or AppArmor according to the host configuration. An administrator must set explicit resource limits when a workload requires them because creating a control group does not automatically impose a useful CPU or memory ceiling.

The process started by the image normally appears as process ID 1 inside a private PID namespace. That process controls the container's lifecycle. When it exits, the container stops. The process may be an application, a shell, or an init system. Containers often run one primary service, but the technology does not require a single process.

Rootless Podman places container root inside a user namespace. UID 0 in the container maps to an unprivileged identity on the host, so it does not acquire host root authority. Rootless operation narrows the impact of many failures, but it does not make an unsafe image trustworthy. The container still shares the host kernel, can consume resources unless constrained, and can access any host content deliberately mounted with sufficient permissions.

Runtime options can strengthen a workload according to its needs:

```bash
podman run --rm \
  --read-only \
  --cap-drop all \
  --security-opt no-new-privileges \
  --memory 256m \
  --cpus 0.5 \
  --pids-limit 100 \
  quay.io/podman/hello
```

`--read-only` protects the container's root filesystem, although an application may need writable tmpfs or volume mounts. Dropping all capabilities creates a strong baseline, after which the operator can add only a required capability. `no-new-privileges` prevents processes from gaining privileges through mechanisms such as set-user-ID binaries. Memory, CPU, and process limits protect host availability. A production definition should also select a non-root application user, verify the image, and avoid broad device access, host namespaces, and privileged mode.

The `unshare` command demonstrates the kernel primitives beneath containers. A root shell can create private PID, mount, and network namespaces without using Podman:

```bash
sudo unshare --fork --pid --mount-proc bash
ps
exit

sudo unshare --fork --pid --mount-proc --net bash
ip address
exit
```

The first shell sees its own PID tree. The second also receives a separate network namespace. These commands demonstrate namespace isolation, but they do not assemble the storage, security policy, networking, lifecycle management, and image workflow that a container engine supplies.
## Lab and installation
A small RHEL 9, Fedora, Rocky Linux, or AlmaLinux virtual machine can support a command-line lab. Two CPU cores, 2 GB of memory, and about 20 GB of storage suit the basic exercises, although image builds benefit from more memory and disk space. Package names and available Podman versions depend on the distribution and its enabled repositories.

RHEL provides the `container-tools` package group and also offers Podman, Buildah, and Skopeo separately. A minimal installation uses:

```bash
sudo dnf install -y podman skopeo
podman version
podman info
```

`podman info` reveals the storage driver, cgroup version, network backend, rootless status, and other host settings. Rootless operation normally requires subordinate UID and GID ranges in `/etc/subuid` and `/etc/subgid`. A newly created RHEL user usually receives those ranges automatically. Rootless image and container storage normally resides under `$HOME/.local/share/containers/storage`, while rootful storage normally resides under `/var/lib/containers/storage`. Rootless and rootful users therefore maintain separate image, container, and network state.

That separation explains why an image pulled as a regular user does not appear in `sudo podman image ls`. The two commands address different stores. Administrators should avoid changing between rootless and rootful operation during one workflow unless the design requires it. File ownership, port access, cgroup delegation, networking, and systemd management also differ across the two modes.

The lab should record its actual versions rather than assume a course version:

```bash
cat /etc/os-release
podman version
skopeo --version
podman info --format '{{.Host.CgroupsVersion}}'
```

Current commands and features can differ from Podman 4 demonstrations. Local manual pages and the documentation shipped for the installed packages take precedence when a distribution backports features or fixes.

Compose support requires an external provider. `podman compose` acts as a wrapper around a provider such as `podman-compose` or Docker Compose. The administrator should install a provider supported by the distribution, then confirm the selected implementation:

```bash
podman compose version
podman compose --help
```

The provider controls the supported Compose features, so its documentation and version remain relevant.
## Images and registries
An image registry stores named image repositories. Each repository can contain multiple tags, and each tag points to an image manifest. A tag such as `24.04` communicates a release choice, while a digest such as `sha256:...` identifies immutable manifest content. Tags can move, including `latest`, so a digest offers stronger reproducibility.

Podman accepts short image names, but short names can be ambiguous. System-wide registry configuration normally lives in `/etc/containers/registries.conf` and `/etc/containers/registries.conf.d/`. User configuration can override system defaults. Distribution-supplied short-name aliases can map a name such as `hello` to a fully qualified reference. Production definitions should use a fully qualified registry, namespace, repository, and tag or digest.

An image consists of content-addressed layers and a manifest that refers to them. Several images can share unchanged layers, which reduces local storage and transfer costs. A container adds a writable layer without copying the complete image. Images may contain a Linux user space, but they do not include a separate running kernel. A large base image can still carry unnecessary packages, vulnerabilities, and transfer cost, so smaller purpose-built images often improve deployment efficiency and reviewability.

The main image commands are:

| Purpose | Command |
| --- | --- |
| List local images | `podman image ls` |
| Search configured registries | `podman search ubuntu` |
| Pull a named image | `podman pull docker.io/library/ubuntu:24.04` |
| Inspect local image metadata | `podman image inspect docker.io/library/ubuntu:24.04` |
| Remove a local image | `podman image rm IMAGE` |

`podman images` remains an alias for `podman image ls`. Structured subcommands such as `podman image inspect` and `podman container ls` make scripts and explanations easier to follow.

Skopeo reads remote image metadata without first storing the image locally. It can inspect a tag, display its digest, list repository tags, and copy images between supported transports:

```bash
skopeo inspect docker://docker.io/library/ubuntu:24.04
skopeo inspect --format '{{.Digest}}' \
  docker://docker.io/library/ubuntu:24.04
skopeo list-tags docker://docker.io/library/ubuntu
```

The operator should select an explicit supported release, inspect its source and metadata, and pin a digest when exact replay requires it. A repository's tag list does not prove that an image is secure or suitable. Provenance, signatures, vulnerability information, update policy, architecture, and publisher documentation also require review.

A tag and digest can appear together:

```text
docker.io/library/ubuntu:24.04@sha256:<verified-digest>
```

The tag communicates the intended release, while the digest fixes the content. The operator must obtain the real digest from a trusted registry response or release process rather than copying the placeholder. Multi-architecture tags can resolve to a manifest list, so the selected operating system and architecture also influence the final image.

Local tags provide additional names for the same image ID:

```bash
podman image tag \
  docker.io/library/ubuntu:24.04 \
  localhost/ubuntu-lab:24.04
podman image ls
```

Tagging does not duplicate image layers. Removing one tag does not necessarily remove shared content while another tag or container still references it.

Podman can filter local metadata with Go templates:

```bash
podman image inspect \
  --format '{{.Config.Cmd}}' \
  docker.io/library/ubuntu:24.04
```

The image configuration can define an entry point, a default command, environment variables, a working directory, labels, exposed-port metadata, and other defaults. `podman run` can override many of these values.
## Creating and managing containers
`podman run` creates and starts a container. If the referenced image is absent, Podman pulls it first. A short smoke test can run the Podman hello image and remove the stopped container automatically:

```bash
podman run --rm quay.io/podman/hello
```

An interactive Ubuntu shell uses an allocated terminal and standard input:

```bash
podman run --rm -it \
  --name ubuntu \
  --hostname ubuntu \
  docker.io/library/ubuntu:24.04 \
  bash
```

`--name` assigns a stable container identifier for management and network DNS. `--hostname` sets the hostname visible inside the container. They are separate settings. `--rm` removes the container when its primary process exits, which suits temporary tests but not work that must preserve the writable layer.

`podman create` separates definition from execution. It is useful when automation must inspect or prepare a container before the first start:

```bash
podman create \
  --name ubuntu \
  --hostname ubuntu \
  docker.io/library/ubuntu:24.04 \
  sleep infinity
podman container start ubuntu
```

`podman run` performs both operations in one command. `--pull=always` checks the registry before creation, while `--pull=never` requires an existing local image. A controlled deployment normally resolves and approves an image first instead of allowing an unreviewed moving tag to change during startup.

The container inventory distinguishes running containers from all known containers:

```bash
podman container ls
podman container ls --all
```

`podman ps` and `podman ps -a` provide equivalent shorter forms. A stopped container still retains its writable layer and metadata. Starting it again restores that state:

```bash
podman container stop ubuntu
podman container start ubuntu
podman container rm ubuntu
```

Removing the container deletes its writable layer. Applications should store durable or shared data in named volumes or controlled bind mounts rather than relying on that layer.

A clear state model prevents accidental data loss:

| State | Meaning |
| --- | --- |
| Created | Metadata exists, but the primary process has not started |
| Running | The primary process remains active |
| Exited | The process stopped, but metadata and the writable layer remain |
| Removed | Container metadata and the writable layer no longer exist |

`podman wait` blocks until a container exits and returns its exit code. `podman kill` sends a signal immediately, while `podman stop` first sends the configured stop signal and allows a grace period. Graceful stopping gives a service time to flush data and close connections.

A detached container runs in the background:

```bash
podman run -dit \
  --name ubuntu \
  --hostname ubuntu \
  docker.io/library/ubuntu:24.04 \
  bash
```

`podman attach ubuntu` reconnects the terminal to the primary process. The default `Ctrl+P`, then `Ctrl+Q` sequence detaches without stopping the container. Typing `exit` in this example terminates Bash, so the container stops. `podman exec -it ubuntu bash` starts a separate shell in a running container and usually provides a safer administrative session because exiting that shell does not terminate the primary process.

Inspection and monitoring commands expose different evidence:

```bash
podman container inspect ubuntu
podman container top ubuntu
podman container stats ubuntu
podman container logs ubuntu
```

`inspect` returns configuration and runtime metadata. `top` lists the container's processes. `stats` reports resource use. `logs` reads output captured from the container's configured logging path, normally the primary process's standard output and standard error. It does not provide a complete audit trail of every command run through `podman exec`.

Useful troubleshooting starts with the container state and exit code, then proceeds to logs, configuration, mounts, ports, and events:

```bash
podman inspect \
  --format '{{.State.Status}} {{.State.ExitCode}}' \
  ubuntu
podman port ubuntu
podman events --since 10m
```

The operator should preserve the failing state long enough to inspect it. Automatically removing a failed container can discard useful metadata. Health checks can test application behaviour rather than process existence, and systemd or another supervisor can respond to a failed health policy.

`podman container prune` removes stopped containers after confirmation. A targeted `podman container rm NAME` is safer when the operator knows the exact object to delete.
## Ports, web content, and volumes
A web server image can run without installing the server on the host. The following container serves a host directory with Apache HTTP Server:

```bash
mkdir -p "$HOME/podman/www"
printf '%s\n' '<h1>Podman web test</h1>' \
  > "$HOME/podman/www/index.html"

podman run -d \
  --name web \
  --tz=local \
  -p 127.0.0.1:8080:80 \
  -v "$HOME/podman/www:/usr/local/apache2/htdocs:ro,Z" \
  docker.io/library/httpd:2.4

curl http://127.0.0.1:8080/
```

The port mapping publishes container port 80 as host port 8080. Including `127.0.0.1` restricts access to the host's loopback interface. A mapping written as `-p 8080:80` binds to all host addresses by default, not only localhost. The operator should select the intended host address and enforce any additional firewall policy.

An image's `EXPOSE` instruction records port metadata. It does not open a container firewall or publish the port to the host. `-p` publishes a specific mapping, while `-P` can publish all exposed ports to automatically selected host ports.

`--tz=local` sets the container time zone through Podman's runtime option. Environment variables use `--env NAME=value`, but setting a variable has an effect only when the image or application reads it. Image documentation defines supported variables and mount paths.

The bind mount maps host content into the image's document root. `ro` prevents the container from changing that content. On an SELinux host, `Z` gives the content a private container label. Lowercase `z` assigns a shared label when several containers must access the same files. Podman's relabelling options are usually clearer than a permanent manual `chcon` change. An administrator should never disable SELinux to solve an ordinary volume-labelling problem and should not relabel broad system directories.

Bind mounts and named volumes serve different ownership models. A bind mount exposes a chosen host path, which makes source code and configuration easy to edit with host tools. A named volume lets Podman choose and manage the backing path, which reduces dependence on a host directory layout. An anonymous volume has no stable user-selected name and is easier to orphan.

Container and host user IDs can differ under rootless user namespaces. A failed write may reflect Unix ownership rather than SELinux. `podman unshare` helps an administrator inspect ownership from the rootless storage namespace. The `U` volume option can recursively change host ownership to match a container user, but it modifies the host filesystem and can be expensive. It requires a deliberate, narrow target. Idmapped mounts or a suitable application UID often provide cleaner alternatives.

A writable application normally separates immutable content from data:

| Content | Preferred treatment |
| --- | --- |
| Application binaries | Image layer, read-only at runtime |
| Static configuration | Read-only bind mount or image layer |
| Database or application data | Named volume or dedicated bind mount |
| Credentials | Runtime secret, not image layer |
| Temporary files | tmpfs or bounded writable directory |

Named volumes let Podman manage storage location and lifecycle:

```bash
podman volume create app-data
podman run --rm \
  -v app-data:/var/lib/example \
  docker.io/library/alpine:latest \
  sh -c 'date > /var/lib/example/created'
podman volume inspect app-data
```

The example uses a moving tag for brevity. A controlled deployment should pin an approved tag or digest. Removing a container does not normally remove a named volume.
## Starting containers with systemd
Quadlet provides the current declarative integration between Podman and systemd. Podman has deprecated the older `podman generate systemd` command but keeps it available. Quadlet reads files such as `.container`, `.network`, `.volume`, `.pod`, and `.build`, then generates standard systemd services.

A rootless web service can use `~/.config/containers/systemd/web.container`:

```ini
[Unit]
Description=Local Apache test service

[Container]
Image=docker.io/library/httpd:2.4
ContainerName=web
PublishPort=127.0.0.1:8080:80
Volume=%h/podman/www:/usr/local/apache2/htdocs:ro,Z
Timezone=local

[Service]
Restart=on-failure

[Install]
WantedBy=default.target
```

The user reloads the systemd manager and starts the generated service:

```bash
systemctl --user daemon-reload
systemctl --user start web.service
systemctl --user status web.service
```

The `[Install]` section instructs the generator to arrange startup for the relevant user target. A rootless service that must run after the user logs out may require administrator-approved lingering:

```bash
sudo loginctl enable-linger "$USER"
```

System-wide Quadlet files normally belong in `/etc/containers/systemd/` and use the system systemd manager. Rootless files belong in a supported user Quadlet path. Quadlet requires cgroup v2. The service can also declare resource limits, dependencies, health behaviour, secrets, networks, and volumes without embedding a long `podman run` command in a generated unit.

Systemd becomes the lifecycle authority for a Quadlet workload. Administrators should use `systemctl` to start and stop it rather than mixing manual `podman stop` operations with systemd management. Logs can appear through the container log driver and the journal:

```bash
journalctl --user -u web.service
systemctl --user restart web.service
systemctl --user stop web.service
```

The `.container` file remains the source definition. `systemctl --user daemon-reload` regenerates the service after a change. A restart then applies the new container configuration. Rootful units use `sudo systemctl` and the system journal instead.
## Building custom images
A `Containerfile` or `Dockerfile` records repeatable image-build instructions. Podman executes each instruction, creates image layers, and tags the result. A sound build uses an approved base image, combines package installation with cache cleanup, minimises installed software, validates copied configuration, and avoids embedding private keys, passwords, or tokens.

Build context controls which local files the builder can read. A `.containerignore` or `.dockerignore` file should exclude private keys, version-control data, caches, test output, and unrelated large files. A copied secret remains recoverable from an image layer even if a later instruction deletes it. Runtime secrets or supported build-secret mounts prevent that exposure.

Layer order affects rebuild speed and cache reuse. Stable package installation should normally precede frequently changing application source. The build should clean package metadata and downloaded archives in the same `RUN` instruction that installs packages because deleting them in a later layer does not remove their bytes from the earlier layer. Every build should retain enough source information to reproduce and review the result.

An Ansible lab may need a controller and managed nodes that run systemd and SSH. That design serves operating-system automation tests, although most application containers should run their service directly without a full init system.

A simplified Fedora controller image can use:

```Dockerfile
FROM registry.fedoraproject.org/fedora:43

RUN dnf install -y \
      ansible-core \
      openssh-clients \
      openssh-server \
      python3 \
      sudo \
      systemd \
      vim-minimal \
    && dnf clean all \
    && systemctl enable sshd

RUN useradd -m -G wheel -s /bin/bash tux \
    && install -d -m 0700 -o tux -g tux /home/tux/.ssh

COPY --chown=tux:tux --chmod=0600 \
  id_ed25519.pub /home/tux/.ssh/authorized_keys
COPY --chmod=0440 tux-sudoers /etc/sudoers.d/tux

EXPOSE 22
CMD ["/usr/sbin/init"]
```

The associated `tux-sudoers` file can grant passwordless elevation only inside an isolated training environment:

```text
tux ALL=(ALL) NOPASSWD: ALL
```

The host validates that file before the build:

```bash
sudo visudo -cf tux-sudoers
podman build \
  --tag localhost/ansible-controller:43 \
  .
```

Only the public SSH key enters the image. The private key stays outside the build context and outside source control. A more mature workflow injects keys or credentials at runtime through Podman secrets, read-only mounts, an SSH agent, or another secret manager.

The build should also avoid a reusable account password. SSH public-key authentication can provide access without storing a password hash in an image. If a training exercise requires a password, the runtime should inject a short-lived secret and the lab should destroy it with the environment. Image history and registry access can expose any value supplied through an ordinary `ARG`, `ENV`, `RUN`, or copied file.

When the image runs `/usr/sbin/init`, Podman normally recognises the command and enables systemd mode. An explicit lab command can state the intent:

```bash
podman run -d \
  --name controller \
  --hostname controller \
  --systemd=always \
  --network ansible-lab \
  -p 127.0.0.1:2222:22 \
  localhost/ansible-controller:43
```

Systemd mode prepares temporary filesystems, stop signals, and cgroup access for the init process. On an SELinux-separated host, a systemd container that writes to its cgroup hierarchy may require this host-wide Boolean:

```bash
sudo setsebool -P container_manage_cgroup 1
```

The administrator should enable it only when the authorised workload requires it. The container does not need `--privileged` for the normal systemd mode described by Podman. Privileged mode disables major confinement controls and should not serve as a routine troubleshooting shortcut.

An Ubuntu managed-node image follows the same structure but uses an Ubuntu base, `apt`, the `sudo` group, and Ubuntu package and service names. Every base image should use a supported release and, when replay must produce identical content, a verified digest.

An SSH client should record and verify host keys. Disabling strict host-key checking and directing the known-hosts file to `/dev/null` conceals server identity changes. Ephemeral labs can use a separate known-hosts file and refresh it deliberately after a container rebuild.
## Podman networks
Podman gives containers outbound networking by default unless the operator selects `--network none`. Rootful containers normally use a bridge network. Current rootless Podman normally uses `pasta` for its default private network mode. The exact network data shown by `podman inspect`, including a conventional container IP address, depends on the mode and host configuration.

A user-defined bridge network creates a stable application boundary and supports container-to-container name resolution through its DNS plugin:

```bash
podman network create ansible-lab
podman network inspect ansible-lab
```

Podman can allocate a conflict-free subnet automatically, which is safer than hard-coding a subnet that may overlap an existing route. An administrator can still provide `--subnet`, `--gateway`, and address ranges when the surrounding network design requires them.

Network mode changes the isolation boundary. `--network none` creates no usable network interface. `--network host` shares the host network namespace and removes normal port isolation, so it can expose host-local services to the container. `--network container:NAME` reuses another container's network stack. A user-defined bridge keeps separate network namespaces while connecting selected containers through virtual interfaces and DNS.

Containers join the network at creation:

```bash
podman run -d \
  --name controller \
  --hostname controller \
  --network ansible-lab \
  -p 127.0.0.1:2222:22 \
  localhost/ansible-controller:43

podman run -d \
  --name ubuntu \
  --hostname ubuntu \
  --network ansible-lab \
  localhost/ansible-node:24.04
```

The host reaches the controller through the published SSH port. The controller reaches the managed node by the DNS name `ubuntu`. Published ports expose selected services to the host or wider network. They are not required for traffic between containers on the same user-defined network.

The operator can add aliases when one service needs a stable role name:

```bash
podman run -d \
  --name ubuntu \
  --network ansible-lab \
  --network-alias managed-node \
  localhost/ansible-node:24.04
```

Other containers on the DNS-enabled network can resolve both `ubuntu` and `managed-node`. Static IP addresses are rarely necessary because DNS names survive address reallocation and express service intent.

A Podman pod solves a different problem. Containers in a pod can share namespaces, including one network stack, and communicate through localhost when configured to share networking. A user-defined network gives separate containers their own network stacks and connects them through a virtual network. The deployment model determines which design fits.
## Compose orchestration
A Compose file defines related services, image builds, networks, volumes, ports, and other runtime settings in YAML. `podman compose` passes the file and command to the configured external provider. The administrator should select and test the provider explicitly because compatibility can vary.

The project name scopes generated container, network, and volume names. Explicit service names and DNS aliases should carry application meaning, while generated resource names can retain the project prefix. Relative paths resolve from the Compose project directory, so automation should run from a known location or pass the file explicitly with `-f`.

The current Compose specification does not require a top-level `version` field. That field remains only for backward compatibility and produces an obsolete-field warning in current Docker Compose.

A compact two-node lab can use `compose.yaml`:

```yaml
name: podman-lab

services:
  controller:
    build: {context: ./fedora}
    image: localhost/ansible-controller:43
    hostname: controller
    ports: ["127.0.0.1:2222:22"]
    volumes: ["./playbooks:/srv/playbooks:Z"]
    networks: [lab]
  ubuntu:
    build: {context: ./ubuntu}
    image: localhost/ansible-node:24.04
    hostname: ubuntu
    networks: [lab]

networks:
  lab: {}
```

The relative build contexts contain each node's `Containerfile` and public assets. The playbook bind mount receives a private SELinux label. A source directory shared with several services would use `z` instead of `Z`.

Compose manages the project from the directory containing the file:

```bash
podman compose build
podman compose up -d
podman compose ps
podman compose logs
podman compose stop
podman compose start
podman compose down
```

`build` creates the custom images. `up -d` creates the project network and starts the services in the background. `stop` preserves the containers for a later `start`. `down` stops and removes the project's containers and network. Images and named volumes follow provider-specific flags and Compose rules, so the operator should not assume that `down` deletes all project data.

Compose expresses desired topology, but service order does not prove application readiness. A database process may start before it accepts connections. Health checks, retrying clients, and dependency conditions supported by the selected provider can coordinate readiness. Logs, exit codes, and `podman compose ps` remain necessary when part of the project fails.

Configuration should avoid hard-coded credentials. An environment file can separate non-sensitive settings, while Podman secrets or another secret facility should carry sensitive values. A committed example file can document required variable names without containing operational values.
## Ansible test lab
The controller image can carry Ansible while the host supplies playbooks through a bind mount. An inventory can group Fedora and Ubuntu nodes, assign the appropriate connection data, and define distribution-specific variables. For example, the Apache package and service use `httpd` on Fedora and `apache2` on Ubuntu.

A small INI inventory can identify both nodes:

```ini
[redhat]
controller ansible_connection=local

[debian]
ubuntu ansible_user=tux

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

Group variables can then hold distribution differences without duplicating tasks:

```yaml
# group_vars/redhat.yml
apache_package: httpd
apache_service: httpd
```

```yaml
# group_vars/debian.yml
apache_package: apache2
apache_service: apache2
```

The playbook refers to `apache_package` and `apache_service`, while Ansible selects the values for each inventory group. This pattern keeps operating-system differences at the data layer.

After the operator configures verified SSH keys between the controller and managed nodes, Ansible can test Python connectivity:

```bash
cd /srv/playbooks
ansible all -m ping
```

An ad hoc package task can exercise privilege elevation and each distribution's package manager:

```bash
ansible all \
  --become \
  --module-name package \
  --args 'name=w3m state=present'
```

A playbook can install Apache, copy content, enable the service, and start it:

```bash
ansible-playbook apache.yml
```

Ansible's idempotence means a second run changes only resources that differ from the declared state. The containers provide disposable target systems, while the Compose file recreates their topology. This arrangement supports training and integration tests without maintaining several full virtual machines.

The first connectivity test should distinguish an SSH failure from a Python or privilege-elevation failure. `ansible-inventory --graph` confirms inventory membership, `ansible all -m ping` checks connection and Python execution, and a small `--become` command checks sudo policy. The operator can then run the full role or playbook with clearer evidence.

The host bind mount keeps playbook edits outside the controller's writable layer. Destroying and recreating the controller therefore tests whether the declared image, mount, network, and credentials contain everything required. A test that succeeds only after an undocumented interactive change has not produced a reproducible lab.

The lab still needs sound security boundaries. Passwords should not appear in image layers, Compose files, shell history, or source control. Passwordless sudo should remain limited to disposable training nodes. SSH host keys require verification. The operator should apply resource limits when concurrent builds or services could exhaust the host.
## End-to-end lab workflow
1. The administrator creates a dedicated unprivileged account, confirms its subordinate UID and GID ranges, and checks Podman, cgroup, storage, network, and SELinux settings. The account keeps the lab separate from rootful containers and unrelated user workloads.
2. The operator selects fully qualified base images from approved publishers. Skopeo retrieves remote metadata, and the release process records tags, digests, architectures, and relevant provenance. A digest pins any image that must replay with identical content.
3. Each build context contains a minimal Containerfile, public configuration, and a restrictive ignore file. The build excludes private keys, passwords, tokens, caches, and unrelated source. Package installation and cleanup occur in the same image layer.
4. The controller image contains Ansible, Python, SSH tools, and only the packages needed for the exercise. Managed-node images contain Python, SSH, sudo policy, and the services under test. A full init system runs only where operating-system automation requires it.
5. A user-defined network connects the controller and nodes. Service and container names provide DNS resolution, while a loopback-bound published port gives the host controlled access to the controller. The deployment publishes no managed-node port without a defined need.
6. Bind mounts expose playbooks and editable test content with read-only access wherever possible. Named volumes hold durable application data. SELinux uses `Z` for a private mount or `z` for shared content, and Unix ownership matches the container's user mapping.
7. Compose defines the complete topology, including builds, images, hostnames, networks, ports, and mounts. The selected provider builds and starts the project. Health checks and retrying clients handle application readiness after process startup.
8. The operator verifies inventory, SSH host identities, Python availability, and privilege elevation before running the main Ansible playbook. A second playbook run should report no unnecessary changes when the first run reached the declared state.
9. The operator destroys and recreates the containers from the declarations. Successful recovery confirms that interactive changes did not become hidden dependencies. Separately managed secrets return through the authorised runtime mechanism rather than through image layers.
10. Compose removes the disposable project after testing. Targeted inspection precedes broader pruning, and volume removal follows a data-retention decision. Version-controlled definitions and approved external secrets remain sufficient to rebuild the environment.
## Cleanup and recovery
Podman reports storage use before cleanup:

```bash
podman system df
podman container ls --all
podman image ls
podman network ls
podman volume ls
```

Targeted removal protects unrelated work:

```bash
podman container rm CONTAINER
podman image rm IMAGE
podman network rm NETWORK
podman volume rm VOLUME
```

Prune commands remove unused resources and require careful review:

```bash
podman container prune
podman image prune
podman system prune
podman system prune --all
```

`podman system prune` removes stopped containers, unused pods and networks, dangling images, and applicable build cache. `--all` also removes unused images that no container references. Neither command removes volumes by default because they may hold valuable data. `--volumes` adds unused volumes to the deletion scope. Running containers and resources they use remain protected, but an administrator should still inspect the confirmation list and avoid `--force` unless automation has already resolved the exact scope.

A declarative lab should be recoverable from its Containerfiles, Compose file, playbooks, public configuration, and separately managed secrets. That recovery test provides stronger evidence than retaining unidentified stopped containers and mutable image tags.

Cleanup should follow dependency order when manual commands replace Compose. Containers release ports, mounts, and network attachments first. The operator can then remove unused networks and images. Volumes remain until the operator has confirmed that they can discard the data or has backed it up. This order reduces forceful deletion and makes each removal easier to explain.