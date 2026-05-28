# Tactics and Tools for Troubleshooting Docker
Docker troubleshooting works best when engineers collect evidence before changing configuration. Container logs, inspection output, filesystem checks and network state usually reveal whether the fault sits in the image, the container command, the host, a registry, a mount or a network.
## Core workflow
Engineers should start with the smallest reliable signal and then widen the investigation.

- Check container logs for application output, startup errors and runtime exceptions.
- Inspect the container for its image, command, exit code, mounts, logging path, host settings and network settings.
- Check the container filesystem when path, permission or missing-file errors appear.
- Remove unused Docker objects when stale containers, old networks or dangling images may be influencing the result.
- Rebuild or recreate resources only after the current state has been recorded.
## Logs, inspection and the container filesystem
`docker logs` retrieves output written to a container's standard output and standard error streams. The `--tail` option limits the number of lines, `--follow` streams new output, `--details` adds available logging metadata, `--since` and `--until` filter by time, and `--timestamps` adds timestamps. Logs are the first practical check for most Docker failures because they show what the container process attempted to do.

`docker inspect` returns low-level JSON for Docker objects. Useful fields include `Id`, `Name`, `State`, `Config`, `HostConfig`, `NetworkSettings`, `Mounts` and `LogPath`. Go template formatting narrows the output. For example, `{{.Config.Image}}` returns the image name, `{{.State.ExitCode}}` returns the exit code and `{{json .Config}}` returns the runtime configuration as JSON.

Engineers can examine a running container with `docker exec -it <container> /bin/sh` or another shell available in the image, such as `/bin/bash`. Minimal images may not include Bash. If a container is stopped, `docker export` can write the filesystem to a tar archive, and `docker cp` can copy files between the container and the host when permissions allow it.
## Cleaning resources safely
Pruning reduces conflicts and frees disk space, but it can remove data or dependencies that other work still needs. `docker image prune` removes dangling images, and `docker image prune -a` removes all images not used by an existing container. `docker container prune` removes stopped containers. `docker network prune` removes unused networks. `docker volume prune` removes unused local volumes, and current Docker behaviour removes only anonymous volumes by default unless `--all` is specified.

`docker system prune` removes unused containers, networks and images. It does not remove volumes unless `--volumes` is supplied. Teams should avoid blind pruning on shared machines or production hosts. They should inspect the target resources first and preserve named volumes that contain important data.
## Build cache and image build failures
Docker builds execute Dockerfile instructions in order. For each instruction, Docker checks whether it can reuse the build cache. A cached layer is reused when the instruction and relevant inputs have not changed. For `COPY` and `ADD`, Docker calculates a checksum from file metadata, but modification time alone does not invalidate the cache. If a layer changes, later layers must be rebuilt because their parent layer has changed.

The build cache speeds up development, but it can hide stale dependencies or old base images. `--no-cache` forces Docker to re-run build steps that would otherwise be cached. It does not by itself guarantee a fresh base image, so `--pull` should be used when the latest base image is required. `--cache-from` can seed the cache from another image when builds share layers.

Build failures often come from instruction order. For example, an application build that runs before dependencies are installed will fail. Engineers should read the first failing Dockerfile instruction, confirm that paths exist in the build context and image, and verify that each command can run under the configured user.

Modern BuildKit changes the visibility of intermediate containers compared with the legacy builder. Detailed logs, targeted multi-stage builds, temporary diagnostic commands and `--progress=plain` are usually safer than relying on legacy intermediate-container inspection. Where an older workflow exposes intermediate image or container IDs, they can help inspect the filesystem around the last successful step.
## Docker Desktop, filesystem and YAML failures
Docker Desktop startup problems usually come from installation state, virtualisation, host security tools, incompatible versions or a stale daemon. On Windows, Docker Desktop depends on suitable Windows features and virtualisation support, commonly through WSL 2 or Hyper-V depending on the setup. Endpoint protection can also block networking or filesystem operations. On macOS, Docker Desktop must match a supported macOS version. A restart, clean uninstall and reinstall can resolve corrupted local state, but that step can remove settings and should not be the first response on a machine with active projects.

File access errors usually involve the wrong base image, wrong path, missing executable, insufficient permissions, incorrect user, bad volume mount or Windows line endings in scripts that run inside Linux containers. Engineers should inspect the container, confirm the command path, check user permissions, convert scripts to LF line endings where needed and test the command inside an interactive shell.

Tar writer errors during builds or file copies often indicate a daemon issue, a file lock, a stopped Docker service, mismatched permissions or another process holding files open. The practical checks are Docker Desktop status, active file handles, consistent host and container users, administrator rights where required, and a daemon restart after the evidence has been captured.

Compose files use YAML. YAML is sensitive to indentation and does not allow tabs for indentation. Parser errors should be fixed by checking the reported line and the surrounding block. A YAML-aware editor extension helps detect invalid keys, bad indentation and malformed strings before Docker Compose runs.
## Registry access and image trust
A `pull access denied` error usually means that the image name, tag, namespace or registry URL is wrong, or that the registry requires authentication. Engineers should confirm the exact image reference, run `docker login` or the relevant cloud CLI login, check proxy settings, review DNS configuration and consider lower concurrent download settings on slow links.

Docker Hub still applies pull-rate limits to unauthenticated users and Docker Personal accounts. Pro, Team and Business accounts have unlimited pull rate under the current Docker Hub limits. Rate-limit errors should be handled by authenticating, using the right subscription or reducing unnecessary pulls in automation.

Older Docker workflows used Docker Content Trust and `docker trust inspect` to inspect signatures. Docker has been retiring Docker Content Trust for Docker Official Images, and Docker Engine 29 removed Docker Content Trust from the Docker CLI. Current image-integrity work should favour immutable digests, trusted publishers and modern signing or analysis tools such as Docker Scout, Sigstore or Notation where they fit the organisation's policy.
## Exit codes
Exit codes help separate Docker invocation errors from application errors.

- `0`: the foreground process completed successfully or no long-running foreground process remained.
- `1`: the application or command inside the container failed.
- `125`: Docker itself failed to run the container, often because of an invalid flag or daemon problem.
- `126`: Docker found the specified command but could not invoke it, often because of permissions.
- `127`: Docker could not find the specified command.
- `137`: the process received `SIGKILL`, often from manual killing or an out-of-memory condition.
- `139`: the process received `SIGSEGV`, which usually indicates a memory access violation.
- `143`: the process received `SIGTERM`, commonly after `docker stop` or a graceful shutdown request.
## Volumes
Volume errors commonly appear as invalid volume specifications, missing files, overwritten directories or permission failures. The host path and container path must be correct. Containers must mount directories as directories and files as files. A mount over a non-empty container directory hides the existing container contents while the mount is active.

`docker volume ls` lists volumes, and `docker volume inspect <volume>` shows details such as the mount point. `docker inspect` can show which mounts a container uses. Named volumes often hold important application data, so Docker does not remove them automatically. Engineers should prune volumes only after confirming that no container or future workflow needs the data.
## Networking
Docker provides several network drivers. `bridge` is the default driver and suits containers on the same host. User-defined bridge networks are better than the default bridge because they provide automatic DNS resolution between containers and better isolation. `host` removes network isolation from the Docker host. `overlay` connects containers across multiple Docker daemons, commonly in Swarm-style deployments. `ipvlan` gives direct control over IPv4 and IPv6 addressing. `macvlan` can make a container appear like a physical device on the network. `none` gives the container only a loopback interface.

`docker network ls` shows available networks, and `docker network inspect <network>` shows the driver, subnet, gateway and attached containers. `docker inspect` shows a container's `NetworkSettings` and `HostConfig`, including network mode, bind mounts and port bindings.

Port conflicts occur when another container or host process already uses the requested host port. Compose port syntax places the host port on the left and the container port on the right, as in `8080:80`. Engineers should stop or remove stale containers, check host processes with tools such as `lsof` on macOS and Linux or `netstat` on Windows, and change the host port when another service must keep it.

Docker Compose creates a project-level default network and registers service names with internal DNS. Containers on that network should address each other by service name rather than hard-coded IP address. DNS failures usually come from using the wrong network, hard-coding old IP addresses, overriding DNS carelessly or expecting the host's DNS behaviour to match container DNS behaviour.

Connectivity checks should test name resolution, IP reachability and the application port separately. ICMP tools such as `ping` may be unavailable in minimal images or blocked by policy, so application-level tools such as `curl`, `wget`, `nc` or service-specific clients often provide better evidence. Certificate errors require separate checks for host certificates, container trust stores, proxy interception and whether TLS terminates inside the container or before traffic reaches it.

`network not found` errors often follow manual stopping, manual network removal, daemon restarts or stale Compose resources. `docker compose down` removes the Compose containers and networks cleanly. `docker compose up --force-recreate` can rebuild the expected relationship between containers and networks after stale IDs or removed networks cause failures.
## Support and escalation
Good troubleshooting notes make escalation faster. Engineers should record the Docker version, Docker Desktop version, operating system, failing command, exact error text, relevant logs, Compose file fragments, Dockerfile fragments and recent changes. They should also record whether the failure reproduces after `docker compose down`, after a daemon restart and after rebuilding without cache.

Official Docker support depends on the organisation's subscription and product entitlement. Community support remains useful for development issues. Docker forums, Docker community channels and DevOps Stack Exchange can help when the question includes a minimal reproducible example, command output and environment details. Public questions should not include secrets, private registry tokens, internal hostnames or confidential paths.

Teams should treat destructive repair steps as escalation actions. Factory resets, clean reinstalls, broad prune commands and volume removal can solve corrupted local state, but they can also delete caches, containers, networks, images and data. A backup or documented recovery plan should come before those actions.
## Continuing practice
Docker troubleshooting improves with repeated practice across logs, inspection output, build cache behaviour, filesystem layout, volumes, registry access and networking. The strongest habit is to gather the current state, make one controlled change, and verify the result before moving to the next hypothesis.