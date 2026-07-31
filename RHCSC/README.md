# *Red Hat Certified Specialist in Containers* Course Notes
*v1.0 stable release*

These notes cover material from Pluralsight's 2 hour, self-paced [Red Hat Certified Specialist in Containers](https://www.pluralsight.com/paths/red-hat-certified-specialist-in-containers-ex188) course. They cover how to to create, configure, and manage containerized services using Red Hat OpenShift and other Red Hat technologies.
## Container images and registries
A container image is an immutable OCI image that combines a root filesystem, configuration, and metadata. Podman adds a writable layer when it creates a container from an image. Image builders record filesystem changes in read-only layers, and registries can store and transfer shared layers once. Build caches can reuse unchanged layers, which reduces build time, storage, and network traffic.

Each layer records a filesystem difference from its parent. Layers are not independent restore points and do not replace source control. An operator rolls back by running a known earlier image or rebuilding from controlled source. Containers share the host kernel, so an image must support the target operating-system architecture even though it carries its own user-space files and libraries.

An image digest identifies its content. A tag such as `1.4.0` or `latest` is a mutable name that can point to another digest later. Tags organise releases, but they do not provide version control. Production deployments should use meaningful release tags and, when exact identity is required, pin the verified digest.

A registry stores image manifests, configuration objects, layers, signatures, and related artefacts. A multi-architecture manifest can direct each host to an image built for its platform. Public and private describe access policies rather than different image formats. Registry authentication, repository permissions, TLS, and signature policy control who can distribute and retrieve content.

RHEL 10 supplies Podman, Buildah, Skopeo, CRIU, and supporting libraries through the `container-tools` meta-package:

```shell
dnf install container-tools
podman info
```

Red Hat Universal Base Image 10 provides redistributable micro, minimal, standard, and init variants. `registry.redhat.io` requires authentication, while `registry.access.redhat.com` provides unauthenticated access to suitable images. Fully qualified image references avoid ambiguous short-name resolution.

The standard UBI includes DNF and common utilities. UBI minimal uses `microdnf`, UBI micro omits a package manager, and UBI init supports applications that require `systemd`. Image authors should select the smallest supported variant that supplies the required runtime. They should update the base regularly because copying an old base into a new image does not add later security fixes.

```shell
podman login registry.redhat.io
podman pull registry.redhat.io/ubi10/ubi
podman search registry.access.redhat.com/ubi10
```

Podman reads system-wide registry settings from `/etc/containers/registries.conf`. A rootless user can override them in `$HOME/.config/containers/registries.conf`. Administrators should restrict unqualified searches to trusted registries and keep short-name enforcement enabled.

```toml
unqualified-search-registries = [
  "registry.access.redhat.com",
  "registry.redhat.io"
]
short-name-mode = "enforcing"

[[registry]]
location = "registry.redhat.io"
blocked = false
insecure = false
```

The `insecure` setting permits connections without valid TLS protection and should remain `false` for remote registries. The `policy.json` trust policy can require image signature verification.

`podman login` stores registry credentials in an authentication file for later pulls and pushes. Administrators should protect that file, avoid credentials on command lines, and use scoped service accounts in automation. `podman search` discovers repositories, while `podman info` displays the active registry and storage configuration. Search results do not establish image trust, so operators should confirm the publisher, supported architecture, digest, signature, and update status before deployment.
## Image management
Podman manages local images, while Skopeo can inspect remote images without pulling them.

| Operation | Command |
| --- | --- |
| List local images | `podman images` |
| Pull an image | `podman pull registry.redhat.io/ubi10/ubi` |
| Inspect a local image | `podman inspect IMAGE` |
| Inspect a remote image | `skopeo inspect docker://REGISTRY/IMAGE:TAG` |
| Display image layers | `podman image tree IMAGE` |
| Add a name or tag | `podman tag SOURCE TARGET:TAG` |
| Push an image | `podman push TARGET:TAG` |
| Remove an image | `podman rmi IMAGE` |

`podman tag` adds another reference to the same local image. A push succeeds only when the destination repository exists and the authenticated account has permission to write to it. `podman image prune` removes dangling images. Adding `-a` removes every image unused by a container, so administrators should review its effect before confirming.

A controlled release flow pulls a fully qualified base, inspects its metadata, builds and tests the application image, assigns a release tag, and pushes it to an authorised repository. Deployment records should retain the resulting digest. Reassigning a tag does not modify existing local image content, but a later pull can resolve that tag to new content.

`podman inspect` reports local image configuration, including the user, command, entry point, labels, environment, ports, size, and architecture. `skopeo inspect` queries equivalent remote metadata before download. Neither command proves that an image is safe. Signature verification, vulnerability assessment, provenance checks, and application testing address different risks.

`podman save` writes an image, its layers, its history, and its tags to an archive. `podman load` restores that image. OCI archives suit OCI workflows, while Docker archives support Docker-compatible tools.

```shell
podman save --format oci-archive \
    -o site-image.tar localhost/site:1.0
podman load -i site-image.tar
```

`podman commit CONTAINER IMAGE` creates an image from changes in a container's writable filesystem and configuration. It does not capture live processes or memory, and it excludes mounted volume content by default. Containerfiles remain the preferred source for repeatable builds.

`podman export` serves a different purpose. It flattens a container filesystem into a tar archive without image history, tags, or layered metadata, and `podman import` creates a new image from that archive. Neither image archives nor exported filesystems replace backups of databases, named volumes, bind mounts, registry metadata, or deployment configuration.

A live-state migration can use `podman container checkpoint` and `podman container restore` when the host, runtime, CRIU, kernel, and workload support checkpointing. Persistent application data still requires an independent backup policy.

```shell
podman container checkpoint \
    --export web-checkpoint.tar.zst web
podman container restore \
    --import web-checkpoint.tar.zst
```
## Building images with Containerfiles
`podman build` uses Buildah libraries to process a `Containerfile` or `Dockerfile` and a build context. Each filesystem-changing instruction can create a layer. Instruction order therefore affects cache reuse and image size.

The build context limits the files available to `COPY` and `ADD`. A small context improves performance and reduces accidental disclosure. Stable inputs, such as package manifests, should appear before frequently changing application files when that order allows useful cache reuse. Every rebuild should still obtain required security updates under the organisation's release policy.

Core instructions have distinct roles:
- `FROM` starts a build stage from a base image.
- `WORKDIR` sets the directory for later instructions.
- `COPY` transfers local build-context content into the image.
- `ADD` also supports remote content and automatic archive extraction, so explicit `COPY` operations are easier to audit.
- `RUN` executes a build-time command and records its filesystem changes.
- `ARG` supplies a build-time value without adding it to the final runtime environment.
- `ENV` records a runtime environment variable in the image configuration.
- `LABEL` records searchable metadata.
- `USER` selects the default runtime user. A non-root user reduces risk.
- `EXPOSE` documents a container port but does not publish it on the host.
- `VOLUME` declares an image mount point. A named volume or bind mount controls persistent storage at run time.
- `ENTRYPOINT` selects the main executable, while `CMD` supplies its default arguments or provides the default command when no entry point exists.

Build arguments and environment variables must not carry passwords, tokens, or private keys. Podman build secrets provide temporary access without recording secret values in the image.

The JSON form of `ENTRYPOINT` and `CMD` passes arguments directly without an implicit shell and handles signals more reliably. Runtime arguments normally replace `CMD`, while `podman run --entrypoint` can replace the configured entry point. The operator remains responsible for publishing ports with `-p` and attaching storage with `--mount` or `-v`.

A multi-stage build uses more than one `FROM` instruction. The final stage copies only required artefacts from a named builder stage, which excludes compilers, source files, and other build dependencies from the runtime image.

```dockerfile
FROM registry.access.redhat.com/ubi10/ubi:latest AS builder
ARG PAGE_TITLE="RHEL 10"
RUN mkdir -p /output && \
    printf '<h1>%s</h1>\n' "$PAGE_TITLE" > /output/index.html

FROM registry.access.redhat.com/ubi10/httpd-24:latest
COPY --from=builder /output/index.html /var/www/html/index.html
USER 1001
EXPOSE 8080
```

Podman builds from the current directory and tags the result locally. The runtime publishes host port 8080 to the image's port 8080.

```shell
podman build --build-arg PAGE_TITLE="RHEL 10" \
    -t localhost/site:1.0 .
podman run --rm -d --name site \
    -p 8080:8080 localhost/site:1.0
curl http://localhost:8080/
```

Small, secure images use a trusted UBI 10 base, copy only required files, run as a non-root user, and remove package-manager caches in the same `RUN` instruction that installs packages. A `.containerignore` file keeps unrelated or sensitive content out of the build context. Verified digests fix base-image identity. The RHEL 10 build options `--source-date-epoch` and `--rewrite-timestamp` can reduce timestamp variation in reproducible build workflows.

Image authors should also record useful OCI labels, define a health check when the application supports one, and keep data outside the writable container layer. Scanners can identify known vulnerable packages, but maintainers must rebuild and redeploy the image to apply fixes.
## Rootless Podman
Rootless Podman runs the engine and containers as a regular host user. A process with UID 0 inside the container maps to an unprivileged host ID rather than host root. The files `/etc/subuid` and `/etc/subgid` allocate subordinate UID and GID ranges. After an administrator changes those files, `podman system migrate` applies the new mappings.

The invoking user's host UID normally maps to root inside the rootless user namespace, while other container IDs map into subordinate ranges. Numeric ownership displayed from the host can therefore differ from ownership displayed inside the container. `podman unshare` runs a command in the same user-namespace mapping and allows administrators to diagnose that difference.

Rootless storage normally resides in `$HOME/.local/share/containers/storage`, separate from root's storage in `/var/lib/containers/storage`. RHEL 10 uses `pasta` as the default rootless network mode. `slirp4netns` remains available as an alternative. A rootless process cannot normally publish host ports below 1024, so applications should publish to an unprivileged host port such as 8080.

Rootless operation limits the effect of a container escape because the engine lacks host-root authority. It does not remove application vulnerabilities or justify running the workload as container root. Images should still select a non-root `USER`, drop unneeded capabilities, avoid privileged mode, and expose the fewest host resources.

Bind mounts need correct Unix ownership and SELinux labels. `podman unshare chown` changes ownership using IDs from Podman's user namespace. The `:Z` mount option applies a private container label, while `:z` applies a shared label for content used by several containers.

```shell
mkdir -p content
podman unshare chown -R 1001:0 content
podman run --rm -d --name web \
    -p 8080:8080 \
    -v ./content:/var/www/html:Z \
    registry.access.redhat.com/ubi10/httpd-24
```

The `:U` mount option can adjust bind-mount ownership automatically, but it recursively changes the host files and can be slow. Host directories also weaken filesystem isolation. Administrators should expose only required paths, grant the least access, retain SELinux enforcement, and manage backup and deletion separately from the container lifecycle.
## Containers and Podman
A container packages an application with its runtime, libraries, and configuration. Linux namespaces isolate processes, networks, mounts, and other resources, while control groups govern resource use. Containers share the host kernel, unlike virtual machines, which run separate guest kernels through a hypervisor. This design gives containers a smaller footprint and faster startup, but it does not make them equivalent to virtual machines or remove the need for layered security.

Podman manages OCI containers and images without a permanent central daemon. It supports rootless operation, pods, registries, Buildah, Skopeo, and systemd integration. Rootless containers reduce host privileges, but they do not remove every security risk. Podman accepts many Docker-style commands and image formats, although complete command and orchestration compatibility does not exist.

Rootful and rootless Podman maintain separate container storage and configuration. A container created with `sudo podman` does not appear in an unprivileged user's `podman ps` output. An administrator should choose the operating identity before creating images, containers, networks, volumes, and secrets, then use that identity consistently.

RHEL 10 supplies Podman through the `container-tools` meta-package:

```shell
sudo dnf install container-tools
podman info
podman version
```

Fully qualified image names identify the registry and repository explicitly. Public content from `registry.access.redhat.com` does not require authentication. Protected content from `registry.redhat.io` requires a Red Hat registry login.

```shell
podman login registry.redhat.io
podman pull registry.access.redhat.com/ubi10/ubi
podman run --name release-check --rm \
  registry.access.redhat.com/ubi10/ubi cat /etc/redhat-release
```

An image contains read-only layers and metadata. A running container adds a writable layer. Stopping and restarting that container retains the layer, but removing the container discards it. Volumes or bind mounts must hold data that must outlive the container.

| Task | Command |
|---|---|
| Search a registry | `podman search registry.access.redhat.com/ubi10` |
| List local images | `podman images` |
| Create and start a container | `podman run [options] IMAGE [command]` |
| List running or all containers | `podman ps` or `podman ps -a` |
| Stop or restart a container | `podman stop NAME` or `podman restart NAME` |
| Start an existing container | `podman start NAME` |
| Remove a container | `podman rm NAME` |
| Remove an image | `podman rmi IMAGE` |

`podman run --rm` removes a container after it exits. `podman rm -f` stops and removes a container, so an administrator should confirm the target and persistent storage first.

Container creation should encode the complete runtime configuration. Recreating a container from a reviewed command, Quadlet, Kubernetes YAML file, or Compose file gives operations a repeatable deployment path. Interactive changes made with `podman exec` can help diagnosis, but they disappear when Podman replaces the container. Permanent application changes belong in a rebuilt image or mounted configuration.

Least privilege applies to every deployment. An administrator should avoid `--privileged`, mount only required paths, publish only required ports, drop unneeded Linux capabilities, set resource limits, and prefer read-only mounts where the application does not write. Image provenance, update policy, and vulnerability management remain important because a container still executes software on the host kernel.
## Observation and configuration
Podman exposes container state through logs, events, inspection, process lists, and resource statistics:

```shell
podman ps -a
podman logs --timestamps --tail 100 web
podman logs --follow web
podman events --since 1h
podman inspect web
podman top web
podman stats --no-stream web
```

`podman logs` returns available output and exits unless `--follow` is present. It has no content-search option, so a shell tool can filter its output:

```shell
podman logs web 2>&1 | grep -i error
```

Events record lifecycle operations such as creation, startup, exit, removal, network changes, and volume changes. The default events logger commonly uses the system journal. Logs and events support diagnosis, but neither forms a tamper-resistant audit record by itself.

Time filters narrow an investigation without inventing unsupported log options. `podman logs --since 30m --until 10m NAME` selects container output by time, while `podman events --filter container=NAME` limits the event stream. A central log platform should collect, protect, retain, and correlate records when operational or compliance requirements exceed local journal retention.

`podman inspect` returns structured JSON for containers, images, networks, and volumes. A format expression extracts a specific value:

```shell
podman inspect --format '{{.State.Status}}' web
podman inspect --format '{{.Config.User}}' web
podman network inspect appnet
podman volume inspect app-data
```

Environment variables suit non-sensitive configuration such as feature flags, service names, and ports. Values supplied with `--env`, an environment file, or a Compose file can appear in inspection output, shell history, process environments, or source control. Credentials belong in Podman secrets or an external secret manager.

RHEL 10 provides MariaDB 10.11 as `registry.redhat.io/rhel10/mariadb-1011`. This image uses `MARIADB_ROOT_PASSWORD` for the root password. A Podman secret can supply that variable without placing the value on the command line:

```shell
umask 077
read -r -s DB_PASSWORD
printf '%s' "$DB_PASSWORD" | podman secret create mariadb-root -
unset DB_PASSWORD
podman volume create mariadb-data
podman run -d --name mariadb \
  --secret mariadb-root,type=env,target=MARIADB_ROOT_PASSWORD \
  --volume mariadb-data:/var/lib/mysql \
  registry.redhat.io/rhel10/mariadb-1011
```

An application that accepts a credential file should receive the secret as a mounted file instead of an environment variable.
## Networking and application exposure
Each container receives its own network namespace unless another mode is selected. RHEL 10 uses `pasta` as the default rootless network mode. Rootful Podman uses a bridge by default. A user-defined bridge network provides an isolated application network, and Aardvark DNS resolves container names when DNS remains enabled.

A Podman pod groups related containers and gives them a shared network namespace. Containers in the pod normally communicate through `localhost`, and the pod publishes ports for the group. Pods suit tightly coupled processes, while separate containers on user-defined networks provide stronger network separation and independent addressing. Neither model supplies cluster scheduling or high availability across hosts.

```shell
podman network create appnet
podman run -d --name web --network appnet \
  -p 127.0.0.1:8080:8080 \
  registry.access.redhat.com/ubi10/httpd-24
podman run --rm --network appnet \
  registry.access.redhat.com/ubi10/ubi \
  curl -s http://web:8080/
podman port web
```

Containers on the same DNS-enabled network address one another by name and use the destination container's internal port. Published host ports do not control container-to-container traffic. `127.0.0.1:8080:8080` limits host access to the local system. Omitting the host address binds the port on all host addresses by default. Remote access also requires an appropriate `firewalld` rule.

Podman can attach or detach a running container:

```shell
podman network connect appnet client
podman network disconnect appnet client
```

`podman network inspect appnet` reveals subnets, gateways, attached containers, and DNS status. A network created with `--disable-dns` cannot resolve container names. Podman must recreate the network to change that property.
## Persistent storage
Podman supports named volumes and bind mounts:

| Storage | Management | Best use |
|---|---|---|
| Container writable layer | Podman removes it with the container | Caches and disposable state |
| Named volume | Podman creates, inspects, exports, imports, and removes it | Application data with minimal host coupling |
| Bind mount | The administrator manages the host path | Selected host content, configuration, and controlled file exchange |

Podman's graph root and execution mode determine a named volume's host path, so automation should use volume names rather than a hard-coded storage directory.

```shell
podman volume create app-data
podman volume ls
podman volume inspect app-data
podman run -d --name app \
  --volume app-data:/var/lib/app \
  IMAGE
```

A bind mount requires suitable host ownership, permissions, and SELinux labels. The `:Z` option applies a private container label. The `:z` option applies a shared label for content used by multiple containers. `:U` recursively changes host ownership to match the container's user namespace, so it needs careful review before use.

```shell
mkdir -p "$HOME/web-content"
podman run -d --name web \
  -p 127.0.0.1:8080:8080 \
  -v "$HOME/web-content:/var/www/html:Z" \
  registry.access.redhat.com/ubi10/httpd-24
```

For rootless containers, `podman unshare` helps examine or change ownership as it appears inside the user namespace:

```shell
podman unshare stat "$HOME/web-content"
podman unshare chown -R CONTAINER_UID:CONTAINER_GID "$HOME/web-content"
```

A consistent backup requires the application to flush or stop writes before export. Podman can then move a named volume through a tar archive:

```shell
podman stop mariadb
podman volume export mariadb-data -o mariadb-data.tar
podman volume create mariadb-restored
podman volume import mariadb-restored mariadb-data.tar
```

`podman volume rm VOLUME` removes a volume. Podman refuses to remove an in-use volume unless forced, and forced removal can destroy application data.

Mount availability alone does not guarantee safe concurrent access. Applications that share a volume must coordinate locking, transactions, ownership, and file formats. Backups should include restoration tests, encryption, retention, and off-host copies. A volume export captures files, but it does not validate database consistency or restore the container's image, network, secrets, and runtime settings.
## Secrets
Podman stores secrets separately from images and does not include them in image commits or exports. A secret can enter a container as a file under `/run/secrets/NAME` or as an environment variable when an application requires that interface.

```shell
podman secret create app-token token.txt
podman secret ls
podman secret inspect app-token
podman run --secret app-token IMAGE
podman secret rm app-token
```

`podman secret inspect` displays metadata, not the secret value. Access should follow least privilege, protected source files should have restrictive permissions, and operators should avoid credentials in images, command arguments, logs, environment files, and Compose files.

`podman secret create --replace` changes the stored secret, but existing containers keep the value supplied when Podman created them. Rotation therefore requires replacement of each dependent container. Podman secrets provide local host storage, not distributed secret management, policy enforcement, or automatic rotation across several systems.
## Multi-container applications
A multi-container application normally defines services, images, networks, volumes, configuration, secrets, dependencies, and published ports as one deployment. Shared user-defined networks support service discovery, while separate networks can limit communication between tiers.

Only entry-point services should publish host ports. Database and back-end services can remain on internal application networks and accept traffic from authorised peers by container name. Distinct networks can separate front-end, application, and data tiers. Resource limits and health checks should apply per service because one unhealthy or memory-intensive container can otherwise degrade the complete application.

Upstream Podman provides `podman compose` as a wrapper around an external provider such as `docker-compose` or `podman-compose`. RHEL 10 does not ship or support `podman-compose`. An organisation that installs an external provider should validate its Compose features and support policy before use:

```shell
podman compose -f compose.yaml config
podman compose -f compose.yaml up -d
podman compose -f compose.yaml ps
podman compose -f compose.yaml logs
podman compose -f compose.yaml down
```

Compose `depends_on` controls creation and startup order. It does not prove that a database or other dependency is ready to accept requests. Health checks, retry logic, and failure handling must enforce readiness. Persistent data belongs in declared volumes, and credentials belong in supported secret facilities.

RHEL-supported alternatives include Kubernetes YAML with `podman kube play` and Quadlet units. Quadlet can define containers, pods, networks, volumes, images, builds, and Kubernetes workloads under systemd.
## Troubleshooting
Effective diagnosis follows the failure from container state to the application and then to its dependencies:
1. `podman ps -a` identifies creation, startup, exit, and health states.
2. `podman logs NAME` and `journalctl` expose application and service errors.
3. `podman inspect NAME` verifies the image, command, user, environment, mounts, ports, networks, and exit code.
4. `podman exec -it NAME COMMAND` tests the running container from inside its namespaces.
5. `podman port NAME` and `podman network inspect NETWORK` verify publication, membership, and DNS.
6. `ls -Zd PATH`, `podman unshare stat PATH`, and image inspection reveal SELinux, permission, and ownership faults.
7. The administrator changes one cause, recreates the container when configuration changed, and repeats the original test.

A database that reports `permission denied` on a bind mount commonly lacks either the required ownership or an SELinux container label. The correction should set the image's required UID and GID, and use `:Z` or `:z` as appropriate. Disabling SELinux or granting world-writable permissions conceals the cause and weakens the host.

A failed name lookup commonly indicates that the containers use different networks, the network has DNS disabled, or the client uses the wrong service name. A successful name lookup followed by a refused connection points instead to the application state, listening address, or internal port. Host port publication and firewall rules become relevant only when traffic crosses the host boundary.
## Containers as systemd services
RHEL 10 uses Quadlet for new systemd-managed containers. `podman generate systemd` is deprecated. Quadlet source files describe the desired container, and a systemd generator creates transient `.service` units.

A rootless administrator stores `web.container` in `~/.config/containers/systemd/`:

```ini
[Unit]
Description=UBI 10 HTTP service

[Container]
ContainerName=web
Image=registry.access.redhat.com/ubi10/httpd-24
PublishPort=127.0.0.1:8080:8080
Volume=%h/web-content:/var/www/html:Z

[Service]
Restart=on-failure
TimeoutStartSec=900

[Install]
WantedBy=default.target
```

Systemd generates `web.service` after a reload:

```shell
systemctl --user daemon-reload
systemctl --user start web.service
systemctl --user status web.service
journalctl --user-unit web.service
sudo loginctl enable-linger "$USER"
```

Lingering allows the user's service manager to start at boot and continue after logout. A root-managed Quadlet belongs in `/etc/containers/systemd/`, uses ordinary `systemctl` commands, and normally specifies `WantedBy=multi-user.target`.

Generated Quadlet services are transient, so `systemctl enable web.service` does not provide boot persistence. The `[Install]` section in the Quadlet source supplies that relationship when the generator runs. Systemd then manages startup order, restart policy, resource controls, status, and journal integration.