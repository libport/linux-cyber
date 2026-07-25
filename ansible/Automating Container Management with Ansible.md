# Automating Container Management with Ansible
Ansible can install Docker Engine, configure hosts, and manage images, containers, Compose applications, registries, networks, and volumes across a fleet. Playbooks describe the required state in repeatable YAML, while ad hoc commands support focused inspection and testing. Effective automation depends on idempotent tasks, controlled privileges, protected credentials, and versions that match the current Ansible and Docker documentation.
### Architecture and prerequisites
A typical environment separates the Ansible control node from one or more Docker hosts. The control node stores the inventory, playbooks, roles, collections, templates, and secrets. It connects to managed hosts through SSH. Each managed host needs a Python version supported by the selected `ansible-core` release, unless a module uses another documented execution method.

Teams should pin compatible versions of `ansible-core`, the `community.docker` collection, Docker Engine, and Docker Compose. They should also test upgrades before applying them to production. The full Ansible package includes many collections, whereas a minimal `ansible-core` installation requires teams to install `community.docker` separately.

The inventory can place Docker hosts in a group such as `docker_hosts`. The control node should authenticate through SSH keys or another approved mechanism. Tasks that install packages, manage repositories, alter users, or write to protected paths normally require privilege escalation. Setting `become: true` requests escalation, while the `-K` command option prompts for the escalation password when one is required.
### Playbook design and validation
Playbooks should separate host bootstrap, Docker installation, image delivery, application deployment, and cleanup. This structure limits privilege escalation and allows teams to test each responsibility independently. Variables should hold platform differences, image references, ports, paths, users, and policy choices. Inventory and group variables can describe fleet-wide defaults, while host variables should capture genuine exceptions rather than conceal drift.

Tasks should prefer purpose-built Ansible modules to raw shell commands because modules report changes, support check mode where documented, and express desired state. Handlers should restart Docker only when a configuration change requires it. Tags can select installation, deployment, or verification work, and `serial` can limit how many hosts change at once. Source control, peer review, `ansible-lint`, and continuous integration should protect production playbooks.

Idempotence remains a design goal, not an automatic property. Options that force pulls, builds, exports, recreation, or restarts can report changes on every run. Teams should understand those choices, use immutable inputs where possible, and run a second test execution to detect unexpected changes. Check mode cannot predict every Docker action, so a staging environment remains essential.
### Installing Docker Engine
An installation playbook should follow the supported procedure for each operating system rather than translating one distribution's commands directly to another. On a current Ubuntu host, the playbook should:
1. Remove conflicting packages when necessary.
2. Install prerequisites such as CA certificates and `curl`.
3. Place Docker's signing key in an APT keyring.
4. Configure Docker's repository with a `Signed-By` reference.
5. Install Docker Engine, the CLI, `containerd.io`, Buildx, and the Compose plugin.
6. Start and enable the Docker service.
7. Verify the daemon with an information module or a controlled test container.

The older `apt_key` approach and legacy `docker-compose` Python package should not underpin a new deployment. Current `community.docker` modules often contain their own Docker API client code, while Compose automation requires the Docker CLI and Compose v2 plugin. Each module's requirements remain authoritative.

Repository and package tasks should pin a tested release when change control demands predictable upgrades. A playbook should verify the service with `community.docker.docker_host_info` or an equivalent check after installation. Package success alone does not prove that the daemon started, the socket is accessible, or the selected user has permission to use it.

Adding a user to the `docker` group permits access to the Docker socket without `sudo`, but it also grants root-level control over the host. Organisations should restrict this membership, audit it, and consider rootless Docker where appropriate. A new login session is usually required after group membership changes.
### Roles, collections, and execution environments
Ansible Galaxy distributes community collections and roles. The `community.docker` collection supplies fully qualified module names such as `community.docker.docker_container`. Roles can package installation and configuration tasks for reuse. Teams should inspect role source, maintenance history, defaults, handlers, supported platforms, and licence terms before adoption. They should pin reviewed versions and test them in a non-production environment.

A container can also run the Ansible control process. This execution environment improves portability and isolates dependencies, especially when a team tests several Ansible versions. A controlled image should include the required `ansible-core` version, collections, Python dependencies, SSH tooling, inventory, and playbooks.

Mounting the host's Docker socket into that container allows it to control the host daemon. This mount effectively grants host-level administrative power. Host networking is not inherently required for socket access and should be enabled only when the network design requires it. Safer designs use a dedicated remote connection protected by SSH or TLS, narrowly scoped credentials, and short-lived execution environments.
### Managing images
The `community.docker.docker_image` module can pull, tag, push, build, archive, and remove images, depending on its parameters. Related information, load, export, pull, push, remove, and tag modules provide more focused operations. Playbooks should use complete image references and immutable tags or digests for predictable deployments. The `latest` tag is convenient for experiments but weakens reproducibility.

An image workflow commonly performs these actions:
1. Create a build directory on the Docker host.
2. Copy the Dockerfile and every referenced build file.
3. Build and tag the image.
4. Authenticate to the destination registry.
5. Push the image.
6. Inspect the resulting image and record its immutable digest.

The standard image module builds through the Docker daemon API and does not use BuildKit or Buildx. Builds that need BuildKit should use `community.docker.docker_image_build`. Archive operations can export an image before local removal, and `community.docker.docker_image_load` can restore it. Export is not always idempotent, so playbooks should not treat an archive task as change-free on every repeated run.

`source: pull` retrieves an image from a registry, `source: build` builds from a supplied context, `source: local` operates on a local image, and `source: load` imports an archive where supported. Registry-qualified names distinguish private images from Docker Hub images. Authentication should occur before a pull or push, and a logout task can remove credentials after a short-lived job.
### Managing the container lifecycle
`community.docker.docker_container` declares a container's image, name, command, environment, ports, mounts, networks, health check, pull policy, and restart policy. Its `state` controls the lifecycle:
- `present` ensures that the container exists without requiring it to run.
- `started` ensures that it exists and runs.
- `stopped` ensures that it exists but does not run.
- `absent` removes it.

The module compares the existing container with the declared configuration. Some differences require Docker to replace the container because Docker cannot modify every setting in place. `recreate: true` forces replacement, while comparison policies control which differences Ansible considers significant. Operators should assume that changes to ports, environment variables, mounts, and similar creation-time settings can replace a container. Persistent data therefore belongs in named volumes or bind mounts, not in the writable container layer.

Useful restart policies include `always`, `on-failure`, `unless-stopped`, and the quoted value `"no"`. A health check can verify the application rather than only the container process. Information modules and `docker container inspect` help confirm the resulting state.

A safe rollout validates more than Ansible's changed status. It checks the selected image digest, published ports, health status, restart policy, network attachments, and mounts. For a fleet, a rolling strategy can update a small batch, test service health, and continue only after success. A failed health check should stop the rollout before the same configuration reaches every host.
### Deploying Compose applications
Docker Compose defines multi-container applications, networks, volumes, health checks, and dependencies in one project. Ansible can copy or template a Compose file and its configuration assets to each Docker host, then apply the project with `community.docker.docker_compose_v2`. The module also accepts an inline definition when a generated or compact configuration suits the deployment.

The former `community.docker.docker_compose` module relied on Compose v1, which reached end of life, and the collection removed that module. New playbooks should use the v2 module and install the Docker Compose CLI plugin. `state: present` brings the project up, while other states can stop, restart, or remove it. Pull, build, recreation, scaling, and health-wait options should reflect the deployment policy. Before rollout, operators should check for conflicting names, ports, networks, and volumes.

An external Compose file usually serves established applications best because Docker tooling can validate and run it independently. An inline definition can suit a small generated project. Templates should preserve valid Compose YAML, and validation should catch missing environment values before deployment. Removal options need special care because they can also remove images or data volumes.
### Registry authentication and private registries
`community.docker.docker_login` authenticates to Docker Hub or another registry. It can use a registry URL, write to a selected Docker configuration path, reauthorise a session, and log out with `state: absent`. Playbooks must not contain plaintext production passwords. Ansible Vault, an approved secret manager, protected runtime variables, and `no_log: true` reduce exposure through source control and task output.

A private registry can run as a container and store image data in a persistent volume. Production deployments should enable TLS, strong authentication, access control, backups, and retention policies. An unauthenticated registry on `localhost:5000` suits an isolated laboratory, not a network-accessible production service. A build pipeline can log in, tag an image with the registry address, push it, and verify the stored manifest or pull the image in a separate validation step.
### Networks and volumes
`community.docker.docker_network` creates and removes Docker networks. Containers can join networks through either the network's `connected` list or the container module's `networks` parameter. With `appends: true`, Ansible adds listed containers without disconnecting others. A strict network comparison removes unlisted network attachments, so teams should use it only when the playbook defines the complete intended set.

`community.docker.docker_volume` creates, configures, recreates, and removes named volumes. `community.docker.docker_volume_info` inspects them. The container module attaches named volumes and bind mounts. Driver options must match a real, prepared storage device or path, and protected locations may require escalation. Removing a volume can destroy persistent data, so automation should check usage, backup status, and retention requirements before setting `state: absent`.

Named volumes suit Docker-managed persistent data. Bind mounts suit host-managed configuration or content when the operator controls ownership, permissions, and path creation. Network and volume names should include enough project context to avoid collisions. Information modules and inspection commands should verify attachments after a change, especially when strict comparison or recreation can disconnect a service.
### Troubleshooting and safe operation
Troubleshooting should first locate the failing layer and host. YAML syntax, inventory, variable, SSH, and privilege errors often occur before Docker changes. Missing collections, unsupported versions, absent CLI plugins, and module dependencies can fail during module execution. Docker then introduces daemon availability, socket permissions, image, port, network, volume, registry, and application health failures.

`ansible-playbook --syntax-check`, check mode, diff mode, `ansible-lint`, verbose output, module information calls, Docker inspection commands, daemon logs, and targeted host testing narrow the cause. One failing host usually indicates local drift or damage, while uniform fleet failure often points to shared playbook data or dependencies.

The first investigation should preserve the original error, affected host, task name, module arguments with secrets redacted, and recent changes. Operators can then reproduce the smallest failing action on one non-production host. They should confirm Ansible connectivity and escalation before testing Docker, then check daemon status, API access, and application logs in that order. This sequence separates controller, transport, host, daemon, and workload failures.

Cleanup commands require caution. Docker prune operations can remove unused containers, images, networks, build cache, and, when explicitly requested, volumes. Automation should scope cleanup, preview effects where possible, protect required data, and avoid broad prune operations as a routine response to an unexplained failure.