# Pluralsight Red Hat Certified Specialist in Containers Notes
These notes cover material from Pluralsight's 2 hour, self-paced video course. The notes cover how to to create, configure, and manage containerized services using Red Hat OpenShift and other Red Hat technologies.
## Implement and Manage Images with Podman
Podman manages Open Container Initiative images and containers without a central daemon. It can run rootless, which lets an unprivileged user build, pull and run containers with a smaller host security exposure than a root-owned runtime. Image management with Podman relies on four connected ideas: images, registries, Containerfiles and rootless execution.
### Images and Registries
A container image is a packaged filesystem and configuration set that contains the code, runtime, tools, libraries and settings needed to run software. Images use stacked layers. Each instruction in a build normally creates a new layer, and unchanged layers can be cached, reused and shared across images. This design reduces storage use, speeds builds and allows image changes to be traced or rolled back.

Registries store and distribute images. Public registries make images available to anyone, while private registries restrict who can push and pull content. Tags identify image versions inside a registry. Access controls, tagging and continuous integration or continuous delivery workflows let teams publish tested images and deploy known versions.

Podman searches registries configured in `registries.conf` when an image name does not include a registry domain. The system file normally lives at `/etc/containers/registries.conf`, and non-root users can use `$HOME/.config/containers/registries.conf`. The `unqualified-search-registries` setting controls default search locations. Individual registry blocks can define a prefix, location, blocking and insecure transport settings.

Common registry and image commands include:
- `podman login registry.example.com` authenticates to a registry.
- `podman info` shows Podman configuration, including registry settings.
- `podman search httpd` searches configured registries.
- `podman pull registry/image:tag` downloads an image.
- `podman push registry/image:tag` uploads an image when the account has permission.
- `podman images` lists local images.
- `podman inspect image` shows local image metadata.
- `skopeo inspect docker://registry/image:tag` inspects a remote image without pulling it.
- `podman rmi image` removes a local image.
- `podman image tree image` shows image layers.
### Tagging, Saving and Container State
Tags give human-readable names to image variants. They simplify updates, rollbacks and environment separation. Production deployments should use explicit tags, such as semantic versions or release identifiers, rather than depending on `latest`. Immutable release tags improve traceability because the same tag always represents the same build.

Useful tagging and pruning commands include:
- `podman image tag old-image:tag new-image:tag` adds another name or tag to an image.
- `podman image prune` removes dangling images.
- `podman image prune -a` removes unused images that no container uses.

Image backup and container-state capture solve different problems. `podman save -o image.tar image:tag` saves an image and its parent layers into an archive. `podman load -i image.tar` restores that archive into local image storage. This suits distribution, offline transfer and image backup.

`podman commit container new-image:tag` creates an image from the current filesystem state of a container. It preserves changes made after the container started, such as edited files or installed packages. The `--change` option can adjust selected image metadata, including `CMD`, `ENTRYPOINT`, `ENV`, `EXPOSE`, `LABEL`, `USER`, `VOLUME` and `WORKDIR`. Commit is useful for preserving a container state, although reproducible builds should still prefer Containerfiles.
### Building Images with Containerfiles
A Containerfile is a build recipe. Podman reads its instructions, applies them on top of a base image and produces a new image. Podman can also build from Dockerfile-compatible files. The base image determines the Linux distribution, package manager, filesystem layout, preinstalled runtime and support profile.

The main build command is:

```text
podman build -f Containerfile -t localhost/myimage:1.0 .
```

Core Containerfile instructions include:
- `FROM` selects the base image.
- `WORKDIR` sets the working directory for following instructions.
- `COPY` copies local files from the build context into the image.
- `ADD` can copy local files, fetch URLs and unpack archives, so `COPY` is clearer for ordinary local file copies.
- `LABEL` adds key-value metadata.
- `RUN` executes a command during the build and records the result in a new layer.
- `USER` sets the user for later instructions and the default runtime user.
- `EXPOSE` documents a port used by the application, but it does not publish that port on the host.
- `ENV` sets environment variables that persist in the image.
- `ARG` defines build-time variables. Values passed with `--build-arg` override defaults but do not automatically persist unless copied into `ENV` or files.
- `VOLUME` marks a path for external storage.
- `ENTRYPOINT` defines the executable that runs when the container starts.
- `CMD` supplies default arguments or a default command, and users can override it more easily than `ENTRYPOINT`.

A simple Apache image can start from a Red Hat Universal Base Image, install `httpd`, copy `index.html` into `/var/www/html`, run as the `apache` user, expose port 8080, set `/usr/sbin/httpd` as the entrypoint and pass `-DFOREGROUND` through `CMD`.
### Build Arguments, Volumes and Multi-stage Builds
Build arguments make one Containerfile reusable. For example, `ARG buildname` can set a username during the build. A command such as `podman build --build-arg buildname=alex -t aleximage .` creates an image with a different build value from the default. Because `ARG` is build-time only, runtime settings should use `ENV` when the variable must remain visible after the image is built.

Volumes persist data outside a container filesystem. A `VOLUME /var/www/html` instruction tells Podman to create external storage for that path. Volumes created this way are anonymous unless a named volume or bind mount is supplied at runtime. Administrators can inspect them with `podman volume ls` and `podman volume inspect volume-name`.

Multi-stage builds reduce final image size and limit runtime dependencies. A first stage can compile or prepare content, while a later stage copies only the needed artefacts into a smaller runtime image. Each `FROM` starts a new stage, and `COPY --from=builder` copies files from a named stage. This pattern keeps compilers, package caches and build tools out of the deployed image.

A two-stage web build can create `index.html` in a builder stage, then copy it into a prebuilt Apache runtime image. The final image contains the served content and web server, not the intermediate build environment.
### Rootless Podman and Security
Rootless Podman runs the container runtime and container processes without host root privileges. The root user inside the container maps to an unprivileged user on the host. This reduces risk if an application escapes the container, although it does not remove the need for careful image selection, file permissions and network controls.

Rootless operation uses Linux user namespaces and subordinate ID ranges. Podman reads user mappings from `/etc/subuid` and group mappings from `/etc/subgid`. A normal user may appear as UID 0 inside the container while mapping to that user's UID on the host. Other container IDs map into the subordinate range. After manual mapping changes, `podman system migrate` refreshes Podman's stored state.

Useful commands include:
- `cat /etc/subuid` and `cat /etc/subgid` show configured ranges.
- `usermod --add-subuids start-end username` adds subordinate user IDs.
- `usermod --add-subgids start-end username` adds subordinate group IDs.
- `podman unshare command` runs a command in Podman's user namespace.
- `podman unshare chown uid:gid path` fixes ownership for a bind-mounted host directory.

Rootless containers have limits. Some applications expect real root privileges. Privileged ports such as 80 and 443 are unavailable by default to unprivileged users unless the host allows lower unprivileged ports. Rootless networking commonly uses `slirp4netns` or `pasta`, and rootless storage may use `fuse-overlayfs` where native overlay support is unsuitable.

Host directory mounts need explicit security care. The process inside the container accesses the mounted path as the container user mapped onto the host. Permissions must match that mapping. On SELinux systems, unlabeled host paths may block container reads and writes. The `:z` mount option applies a shared container label, while `:Z` applies a private label for one container. Relabel only application data paths, not broad system directories.

Bind-mounted data remains on the host after a container is removed. Administrators must clean unused files, back up important data and avoid mounting sensitive host paths into containers. Combining rootless execution, correct ID mapping, selective SELinux labelling and explicit image tags gives Podman a reproducible and safer workflow for building and managing container images.
### Practical Podman Workflows
A typical image-management workflow starts with registry configuration and authentication. The operator sets the default search registries, logs in where required, searches for a suitable base image, pulls it locally, inspects its metadata and checks its layers. The operator then tags the image with a meaningful local name before pushing it to an authorised registry or using it as a base for a new build.

A reliable build workflow keeps source files in a clean build context. The Containerfile should copy only required files, install packages in as few clear steps as practical, remove package caches where appropriate and run the service as a non-root user. Image names should include a registry, repository and explicit tag when the image will leave a local workstation.

A state-preservation workflow treats `commit` as a capture tool rather than a substitute for source-controlled builds. After a container changes, `podman commit` can preserve the result as a new image, and `podman save` can export that image to a tar archive. The operator can later use `podman load`, create a new container from the restored image and confirm that the expected files, labels and runtime settings remain present.

A rootless web-service workflow needs both ownership and labelling. The host content directory should map to the user or group used inside the container. On SELinux hosts, the bind mount should use an appropriate label suffix, normally `:Z` for a private mount or `:z` for shared content. The service should listen on an unprivileged container port, such as 8080, and publish that port explicitly with `-p host-port:container-port`.
## Managing Containerized Applications with Podman
Podman manages Open Container Initiative containers and images without a central daemon. It can run containers as an ordinary user, work with Docker-compatible images and Containerfiles, group containers into pods, and integrate with systemd. These features make it useful for administrators, DevOps engineers, and developers who need portable application environments with a smaller administrative and security footprint than traditional virtual machines.

Containers package an application with its dependencies while sharing the host operating system kernel. Virtual machines run complete guest operating systems on a hypervisor, which gives strong isolation but adds resource overhead and slower startup. Containers isolate processes, filesystems, users, and networks through Linux kernel features, so many application instances can run efficiently on one host. Images that follow the Open Container Initiative specification can usually move between compatible engines, which improves portability across development, test, and production environments. Podman also supports pods, which group related containers so they can share selected resources. That model resembles Kubernetes pods and helps small services run together as one logical unit before they move into a larger orchestration platform.
### Core Podman Workflow
A basic Podman workflow starts with installing the container tooling on a Red Hat Enterprise Linux host, authenticating to a registry when required, pulling an image, creating a container, checking its state, and removing temporary resources when finished. On Red Hat Enterprise Linux 9, the `container-tools` package group provides Podman, Buildah, Skopeo, and related tools.

Test containers, unused images, temporary volumes, and throwaway networks should be removed after each exercise so later work starts from a known state. That habit prevents confusing results when names, ports, mounts, or stale data collide.

Common lifecycle commands include:
- `podman version` or `podman -v` to show the installed version.
- `podman login` to authenticate to a registry.
- `podman search` and `podman pull` to find and download images.
- `podman images` to list local images.
- `podman run` to create and start a container.
- `podman run --name NAME -d IMAGE` to create a named background container.
- `podman ps` to list running containers.
- `podman ps -a` to list running and exited containers.
- `podman stop`, `podman start`, and `podman restart` to control existing containers.
- `podman rm` to remove stopped containers.
- `podman rm -f` to force removal of a running or stopped container.
- `podman rmi` to remove an image.
### Logs, Events, Inspection, and Configuration
Operational work depends on visibility. `podman logs CONTAINER` reads a container's standard output and error streams, which helps diagnose failed startups, application faults, and runtime behaviour. Log options can limit output, add timestamps, or follow new log entries as they appear. `podman events` reports Podman-level activity such as container creation, starts, stops, removals, image pulls, volume changes, and pod activity. Administrators use events for debugging, audit trails, and automation triggers.

`podman inspect` exposes detailed JSON data about containers and images. It shows configuration, state, process entry point, user, mounts, networking, environment variables, labels, and image metadata. Related commands inspect other resources, such as `podman network inspect NETWORK` and `podman volume inspect VOLUME`. Filtering the JSON with tools such as `jq` makes inspection more useful, especially when only one field matters, such as a container IP address or image user. Inspection should confirm assumptions before a command changes state, because many failures come from incorrect names, ports, users, mounts, and network attachments rather than from Podman itself.

Environment variables configure containers at runtime without rebuilding images. They suit values such as database names, usernames, feature flags, and non-secret runtime settings. A command such as `podman run -e MYSQL_USER=app -e MYSQL_DATABASE=store IMAGE` passes values into the container process. Sensitive values need stronger handling through secrets rather than plain command-line arguments or image layers.
### Running Databases in Containers
Database images often require explicit runtime configuration. A MariaDB or PostgreSQL container may exit immediately when required variables are missing. The fastest diagnosis usually comes from `podman ps -a` followed by `podman logs CONTAINER`. The image documentation or `podman inspect IMAGE` can then confirm required variables, exposed ports, default user, entry point, and expected storage paths.

A corrected database run should set the required variables, assign a stable name, and attach persistent storage. It should also avoid hard-coded production credentials in shell history. Example values in demonstrations must remain examples only. Production deployments should use secrets, restricted permissions, and managed rotation.
### Container Networking and Exposed Services
Podman gives each container an isolated network namespace unless the command specifies another mode. A container on a bridge network receives its own interfaces, routing, and IP configuration. Containers on the same DNS-enabled network can reach each other by container name, which suits small multi-service stacks and local development environments.

Network management centres on a few commands:
- `podman network create NAME` creates a user-defined network.
- `podman network ls` lists networks.
- `podman network inspect NAME` shows driver, subnet, gateway, DNS status, and attached containers.
- `podman network connect NAME CONTAINER` attaches a container to a network.
- `podman network disconnect NAME CONTAINER` removes that attachment.
- `podman network rm NAME` removes a network.
- `podman network prune` removes unused networks.

A private container service becomes reachable from the host through port mapping. The `-p HOST_PORT:CONTAINER_PORT` option maps traffic from the host to the container during creation. For example, `podman run -d --name web -p 8080:8080 IMAGE` sends host traffic on port 8080 to the container's port 8080. Host firewall rules must also allow external clients when access should extend beyond localhost. `podman port CONTAINER` or `podman port -a` confirms active mappings.
### Persistent Storage
Containers have an ephemeral writable layer by default. Podman keeps changes made in that layer while the container exists, then removes them with the container. That behaviour suits caches, temporary files, and disposable test data. It does not suit databases, uploads, or application content that must survive replacement.

Persistent storage uses bind mounts or volumes. A bind mount maps a specific host path into the container. It gives the container direct access to host files, so host ownership, permissions, and SELinux labels matter. On SELinux-enabled hosts, `:Z` can relabel a private bind mount for container use, while `:z` suits shared mounts.

A Podman volume is a named storage resource managed by Podman. Administrators create it with `podman volume create VOLUME`, inspect it with `podman volume inspect VOLUME`, mount it with `--volume VOLUME:/path/in/container`, and remove it with `podman volume rm VOLUME`. The correct removal command is `podman volume rm`, not `podman rm`. Volumes can also be exported and imported as archives with `podman volume export` and `podman volume import`, which helps with migration and backup workflows. Bind mounts are useful when a container must read or write a known host directory. Volumes suit application data that Podman should manage and isolate from normal host paths.

Persistent storage improves:
- Durability, because data survives container deletion.
- Performance for write-heavy workloads, because data avoids the container copy-on-write layer.
- Sharing, because multiple containers can read or write a mounted dataset when permissions allow it.
### Secrets
Podman secrets store sensitive values outside image layers and normal command text. They suit passwords, tokens, certificates, SSH keys, and other credentials that should not appear in source code, image metadata, or routine logs. Secrets reduce exposure, simplify credential reuse across environments, and support rotation through a central Podman interface.

The core commands are:
- `podman secret create NAME FILE` to create a secret from a file.
- `podman secret ls` to list secrets.
- `podman secret inspect NAME` to inspect metadata.
- `podman secret rm NAME` to remove a secret.

A container can receive a secret as an environment variable with syntax such as `--secret dbpass,type=env,target=MYSQL_PASSWORD`. It can also receive a secret as a mounted file. Linux containers mount file secrets under `/run/secrets/TARGET` unless the command specifies another target. Only containers that need a secret should receive it. Administrators should rotate secrets regularly and monitor access patterns. Secrets protect the delivery of sensitive data, but they do not remove the need for least privilege, short-lived credentials where possible, and careful control of shell history, backups, and diagnostic output.
### Multi-container Applications with Compose
Multi-container applications usually contain several services, networks, volumes, and dependencies. Compose files place that structure in YAML so the application can start and stop as one stack. Podman can run Compose workloads through an external Compose provider, such as Docker Compose or `podman-compose`. `podman-compose up -d` or `podman compose up -d` creates the declared services in detached mode when the relevant provider is installed. The matching `down` command tears the stack down.

A typical Compose file defines:
- Services such as a database and an administration UI.
- Images or build instructions for each service.
- Environment variables needed by each service.
- Named volumes for persistent data.
- User-defined networks for service communication.
- Port mappings for host access.
- Dependencies with `depends_on` when startup order matters.

For example, a PostgreSQL service can expose port 5432, mount a named volume under its data directory, and join a shared application network. A pgAdmin service can expose a web UI on a host port, use its own login variables, join the same network, and depend on the database service. The UI can then connect to the database by service name on the shared network.
### Troubleshooting Containers
Effective troubleshooting starts with state, then moves to evidence. `podman ps -a` shows whether a container is running, exited, or restarting. `podman logs` reveals application errors and missing configuration. `podman inspect` confirms image expectations, runtime configuration, mounts, users, and network attachments.

Database startup failures commonly come from missing variables, incorrect mount paths, file ownership, or SELinux labelling. If a database image runs as user 27 and a mounted host directory belongs only to a normal user, the container may fail with permission errors. `podman unshare chown -R 27:27 PATH` can adjust ownership inside the rootless user namespace, and `:Z` can apply an SELinux label that permits private container access.

Network failures often come from two causes. Containers may sit on different networks, or the shared network may not provide name resolution. `podman inspect CONTAINER` and `podman network inspect NETWORK` reveal both problems. The fix may require disconnecting containers from the wrong network, creating a DNS-enabled network, connecting both containers to it, and testing from inside the client with `podman exec -it CONTAINER /bin/bash` followed by a tool such as `curl`.

`podman cp` helps move files into or out of containers during diagnosis. `podman port` verifies published ports. Host firewall rules, local SELinux policy, and application listen addresses can also block access even when the container itself is healthy. A repeatable troubleshooting process changes one variable at a time, validates the result, and records the final command so the fix can be reproduced safely.
### Running Containers as Services
Long-running containers need predictable lifecycle management. systemd can start containers at boot, restart failed workloads, apply resource controls, centralise logs, and align container administration with other Linux services. This approach suits web servers, databases, and local infrastructure services that must survive user sessions and host restarts. Logs can then be read through normal systemd tooling, such as `journalctl`, which places container service messages beside other host events.

Modern Podman deployments should prefer Quadlet for new systemd integration. Quadlet uses declarative files, such as `.container`, `.volume`, and `.network`, which the Podman systemd generator converts into normal service units at boot or during `systemctl daemon-reload`. Rootless Quadlet files can live under `~/.config/containers/systemd/`. System-wide Quadlet files commonly live under `/etc/containers/systemd/`.

Older workflows may use `podman generate systemd --files --new --name CONTAINER` to create a service unit from an existing container. That command is now deprecated, but it still explains many existing systems. Generated root units belong under `/etc/systemd/system`, while generated user units belong under `$HOME/.config/systemd/user`. The path `/etc/system/systemd` is incorrect.

For user services, `systemctl --user daemon-reload`, `systemctl --user start SERVICE`, and `systemctl --user enable SERVICE` manage the unit. `loginctl enable-linger USER` allows user services to start at boot and keep running after logout. For system services, root uses `systemctl daemon-reload`, `systemctl start SERVICE`, and `systemctl enable SERVICE`.