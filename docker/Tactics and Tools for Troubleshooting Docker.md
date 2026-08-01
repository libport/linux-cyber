# Tactics and Tools for Troubleshooting Docker
> [!NOTE]
> This guide presents an evidence-first approach to diagnosing Docker failures across containers, images, builds, filesystems, registries, volumes, networking, and host environments while minimizing risky or destructive changes.
### Use an evidence-first workflow
Effective troubleshooting separates symptoms from causes and changes one variable at a time. An operator first records the exact command, complete error, Docker client and server versions, active context, operating system, recent changes, and expected result. The investigation then follows the failing path through the client, daemon, image, container, network, storage, and application.

Broad cleanup, reinstallation, and factory reset destroy evidence and can remove data. They belong near the end of a diagnosis, after checks and a backup. A workflow keeps the process controlled:
1. Reproduce the failure with the smallest safe command.
2. Capture logs, state, configuration, resource use, and timestamps.
3. Test one hypothesis with a reversible action.
4. Compare the result with the baseline.
5. Record the cause and durable correction.

The active Docker context deserves an early check because a valid command can reach the wrong daemon. `docker version` distinguishes client access from server access, while `docker info` exposes daemon, storage, security, and runtime details.

```shell
docker context show
docker version
docker info
docker system df
```
### Inspect containers, logs, and files
`docker logs` retrieves output that the container writes to standard output and standard error, subject to the configured logging driver. It does not expose application logs written only inside the container. Time filters and a bounded tail reduce noise.

```shell
docker logs --tail 200 --since 15m --timestamps CONTAINER
docker logs --follow CONTAINER
docker inspect CONTAINER
docker stats --no-stream CONTAINER
```

`docker inspect` returns the container's declaration and observed state. Useful fields include `.State.Status`, `.State.ExitCode`, `.State.Error`, `.State.OOMKilled`, `.RestartCount`, `.Config`, `.HostConfig`, `.Mounts`, and `.NetworkSettings`. Go templates select a field without manual JSON searching:

```shell
docker inspect --format '{{json .State}}' CONTAINER
docker inspect --format '{{json .Mounts}}' CONTAINER
docker inspect --format '{{json .NetworkSettings.Networks}}' CONTAINER
```

A running container can execute a diagnostic command with `docker exec`. The selected image may provide `sh` but not `bash`, and minimal images may omit common network tools.

```shell
docker exec -it CONTAINER sh
docker cp CONTAINER:/path/file ./evidence/
docker export --output container-files.tar CONTAINER
```

`docker cp` works with running or stopped containers. `docker export` captures the container filesystem but excludes mounted volume contents. A separate, tested process must protect volume data.

Exit status narrows the search but does not establish the root cause:

| Code | Meaning and next check |
| ---: | --- |
| 0 | The main process completed successfully. A long-running service may still have exited too early. |
| 1 | The application reported a general failure. Application logs provide the next evidence. |
| 125 | Docker could not run the container, often because the invocation or daemon failed. |
| 126 | Docker found the command but could not invoke it. Permissions and executable format need inspection. |
| 127 | Docker could not find the command. The image, path, entrypoint, and arguments need inspection. |
| 137 | The process received `SIGKILL`. Memory exhaustion, `docker kill`, or host action can cause it. |
| 139 | The process received `SIGSEGV`, usually from an application or native-library fault. |
| 143 | The process received `SIGTERM`, often during an orderly stop. Docker may send `SIGKILL` after the grace period. |
### Diagnose Docker Desktop and the daemon
A Desktop startup failure calls for version compatibility, virtualisation, available disk, security software, WSL or hypervisor health, and daemon logs before destructive repair. Docker Desktop diagnostics and operating-system logs preserve useful evidence. On Linux, daemon logs commonly appear through `journalctl -xu docker.service`.

Routine Docker use should not require every tool or terminal to run as an administrator. Docker daemon access grants extensive host authority, so teams should configure access deliberately rather than use blanket elevation. A restart may clear a transient fault. Clean or purge operations, factory reset, and reinstallation remove local state and settings, so the operator must understand and back up the affected images, containers, volumes, and configuration first.

If the daemon stops responding, the operator checks its service state, host resources, and logs before sending more commands. Debug logging can expose daemon activity, but it may increase volume and disclose operational details. A diagnostic bundle should receive the same access controls as other system evidence.

Useful escalation includes the smallest reproduction, exact timestamps, diagnostic identifier, relevant logs, redacted configuration, platform version, and actions already attempted. Official support depends on the applicable product and subscription. Community answers can suggest hypotheses, but the operator should verify commands against current documentation and test them in a disposable environment before changing production or deleting data.
### Troubleshoot builds and Compose
Build failures often arise from an incorrect build context, `.dockerignore` exclusion, instruction order, missing dependency, wrong platform, file permission, or Windows line ending. A `COPY` source must exist inside the build context. Its destination belongs to the image filesystem.

Errors involving a tar writer often occur while Docker packages or transfers the build context. The investigation checks unreadable files, changing generated files, broken symbolic links, insufficient disk, security software, and an unnecessarily large context. A focused `.dockerignore` reduces transfer time and prevents local secrets, repositories, dependencies, and build output from entering the context. The same host account should read every required source file, but granting global administrator access creates more risk than correcting the specific permission.

File-not-found errors require a distinction between host and container paths. Build sources come from the context, bind sources come from the daemon host, and command paths exist inside the image. Alpine and other minimal images commonly provide `/bin/sh` without `/bin/bash`. Linux containers also need Unix line endings and executable permission for scripts. `docker image inspect`, a targeted build stage, or a temporary shell can confirm the image contents without guessing.

BuildKit forms cache keys from Dockerfile instructions and relevant inputs. `COPY` and `ADD` respond to source content and metadata, while most `RUN` cache decisions use the instruction text rather than checking remote package changes. Once a layer misses the cache, later layers rebuild. BuildKit remains the supported default, so operators should not disable it for legacy intermediate containers.

```shell
docker build --check .
docker build --progress=plain .
docker build --no-cache --pull .
docker build --target STAGE --progress=plain .
```

`--no-cache` bypasses reusable layers, while `--pull` requests a newer base image. Multi-stage targets isolate the failing stage. Build secrets and SSH mounts protect credentials better than build arguments or environment variables, which can persist in image metadata or history.

Compose v2 uses `docker compose`, not the retired `docker-compose` spelling. `docker compose config` parses files, resolves variables, merges overrides, and renders the model that Compose will apply. YAML requires spaces rather than tabs, but it does not impose four-space indentation or universal single quoting.

```shell
docker compose config --quiet
docker compose --progress=plain build
docker compose up
```
### Resolve registry and certificate failures
A failed pull may indicate a misspelt reference, missing tag, unsupported platform, absent authentication, denied authorisation, proxy fault, DNS failure, certificate error, registry outage, or rate limit. `docker login` should use an appropriate account or scoped token. Swarm secret inspection does not reveal a registry credential, and it applies only to Swarm secrets.

Current Docker Hub limits allow 100 pulls per six hours for an unauthenticated IPv4 address or IPv6 `/64`, 200 for an authenticated Personal account, and unlimited pulls for authenticated Pro, Team, and Business accounts, subject to fair use. A `429` response can also indicate the separate abuse limit. Authentication, caching, controlled retries, and efficient automation provide safer remedies than repeated pulls.

A private registry signed by an internal certificate authority needs a trusted CA certificate. On Linux, Docker Engine uses `/etc/docker/certs.d/REGISTRY:PORT/ca.crt` for this trust. Docker treats `.crt` files as CA roots and `.cert` plus `.key` files as client credentials. Docker Desktop may also require trust in the host certificate store. An application inside a container maintains its own trust store, so host trust does not automatically fix application TLS. Corporate TLS inspection requires the organisation's authorised CA, not a public resolver or an arbitrary certificate.
### Check mounts before pruning data
`docker inspect --format '{{json .Mounts}}'` shows each mount's type, source, destination, and access mode. `docker volume inspect` describes a Docker-managed volume. Its host mountpoint is not a path that must exist inside the image.

Bind mounts may target a file or directory when source and destination types agree. Mounting over existing container content obscures that content. An empty volume mounted over populated image content receives a copy by default, while `volume-nocopy` suppresses that behaviour. Overlapping mounts can hide one another, so specific and parent mounts need deliberate ordering. Host paths, ownership, labels, and sharing settings also vary across Linux, macOS, and Windows.

Pruning is a storage operation, not a universal repair step. `docker system prune` removes stopped containers, unused networks, dangling images, and unused build cache. `-a` expands image removal, and `--volumes` adds unused anonymous volumes. `docker volume prune` removes unused anonymous volumes by default, while `--all` includes unused named volumes. Filters can narrow several prune commands. An operator should list dependencies, back up required data, read the confirmation, and prefer the narrowest object-specific command.
### Isolate network failures by layer
The bridge driver connects containers on one host. A user-defined bridge supplies automatic name and alias resolution and stronger application separation than the default bridge. Host mode shares the host network stack. Overlay networks connect eligible Docker hosts through Swarm mode. Macvlan and IPvlan integrate containers with underlay addressing but impose platform and network constraints. The `none` driver disables normal networking.

```shell
docker network ls
docker network inspect NETWORK
docker port CONTAINER
docker inspect --format '{{json .NetworkSettings.Ports}}' CONTAINER
```

A published mapping uses `HOST_PORT:CONTAINER_PORT`. Dockerfile `EXPOSE` documents an intended port but does not publish it. If the host port is busy, `docker ps`, `docker port`, `ss` or `lsof` on Linux or macOS, and `Get-NetTCPConnection` on Windows identify the owner. Restarting or resetting Docker should not precede that check.

On a user-defined network, Docker's embedded DNS resolves container names and aliases, not the network's name. `/etc/resolv.conf` shows the resolver configuration. A fixed public resolver such as `8.8.8.8` is not a general correction because internal names, virtual private networks, and corporate policy may require approved DNS servers.

Connectivity testing should progress from name resolution to route, TCP connection, TLS handshake, and application response. `ping` tests ICMP only, and many images or networks omit or block it. A controlled diagnostic image on the same network can supply `getent`, `nslookup`, `dig`, `curl`, and `openssl` without modifying the application container.

Route tables show reachable prefixes rather than every address already allocated. Docker IP address management normally assigns container addresses, but a Docker subnet that overlaps a host, virtual private network, or corporate route can divert traffic. `ip addr`, `ip route`, `route print`, or `netstat -rn` can reveal the relevant host path. The operator compares the network's subnet and gateway with those routes, then creates a non-overlapping network instead of assigning arbitrary container addresses.

An IP connection that succeeds while a name lookup fails points towards DNS. A successful TCP connection followed by a certificate failure points towards trust, hostname, or time. An HTTP error confirms network delivery and shifts attention to the application. This layered interpretation avoids treating every failed request as an IP problem.

Docker refuses to remove a network that connected containers still reference. `docker network rm` therefore requires container disconnection or removal. For a Compose project, `docker compose down` removes project containers and its non-external networks, while `docker compose up` recreates the declared state. `--force-recreate` can replace containers when reconciliation does not. Routine system pruning, daemon restarts, and disabled automatic updates add risk without proving the cause of a network error.