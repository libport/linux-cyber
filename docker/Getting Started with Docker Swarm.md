# Getting Started with Docker Swarm
Docker Swarm extends Docker Engine from a single host to a managed cluster. It keeps the familiar Docker workflow while adding scheduling, service discovery, load balancing, rolling updates, secrets, configs, and cluster-level state management. A developer can still pull an image and run a container, but Swarm adds a control plane that decides where service tasks should run and keeps them aligned with the declared target state.

A standalone Docker host solves common deployment problems by isolating processes, file systems, network interfaces, ports, and runtime dependencies. Containers allow two applications to use different versions of the same runtime or library without overwriting each other. Registries such as Docker Hub make the model practical because prebuilt images provide repeatable file systems for applications such as NGINX, MySQL, MongoDB, Jenkins, and Microsoft SQL Server.

A single host still has limits. Disk, CPU, memory, platform support, availability, and failover requirements eventually outgrow one machine. A team can install Docker Engine on several hosts and manage each daemon separately, but that approach becomes slow and error-prone as the number of hosts increases. Swarm groups Docker hosts into a cluster and presents them as a shared pool of capacity.

Swarm separates two management layers. The node layer covers host creation, Docker Engine installation, networking, SSH access, and membership in the swarm. The application layer covers services, stacks, networks, volumes, secrets, configs, and jobs. Clear separation keeps infrastructure operations distinct from application deployment.

Swarm uses managers and workers. Managers maintain cluster state, accept management commands, schedule tasks, and participate in Raft consensus. Workers run assigned tasks. A manager can also run tasks unless its availability is set to drain. A worker cannot run cluster management commands such as `docker node ls`, because workers do not hold the authority or cluster state needed for that work.

Swarm mode does not replace normal Docker Engine behaviour. A host in swarm mode can still run standalone containers. The difference is responsibility. Standalone containers belong to the local daemon. Swarm services belong to the cluster, and managers reconcile them against the desired state.
## Creating a swarm
A single-node swarm provides the simplest entry point. Docker Desktop and a normal Linux Docker Engine can initialise swarm mode with one command:

```bash
docker swarm init
```

Before initialisation, `docker info` reports Swarm as inactive. After initialisation, the Swarm section reports active status and shows manager details. The first node becomes a manager because a swarm needs at least one manager to store state and accept management operations.

Multi-node labs need hosts that can reach each other over a common network. Vagrant can create local virtual machines for this purpose. A Vagrantfile can define manager nodes named `m1`, `m2`, and `m3`, plus worker nodes named `w1`, `w2`, and so on. Static private IP addresses make join commands predictable. CPU and memory settings should match the host machine. Provisioners can install Docker Engine and useful diagnostic tools.

For new labs, supported operating system images matter. An old Ubuntu 21.04 Vagrant box should be replaced with a current supported Ubuntu LTS box or another maintained distribution. Outdated images create avoidable package, security, and repository problems. The important concept is not the specific distribution. The important concept is a repeatable node definition that starts with Docker Engine installed and network reachability established.

A manager with multiple network interfaces often needs an explicit advertise address. A virtual machine may have a NAT interface for Vagrant access and a host-only interface for cluster traffic. Docker cannot always choose the correct address without help. The command should advertise the address that other swarm nodes can reach:

```bash
docker swarm init --advertise-addr 192.168.99.201
```

The advertised address is not a decorative label. Other nodes use it to contact the manager. Manager nodes should use stable addresses because a changed manager address can leave the swarm trying to contact stale endpoints. Dynamic addressing is less risky for workers, but managers form the control plane and need stable reachability.

Swarm generates separate join commands for workers and managers. The manager token carries more authority because a joining manager participates in cluster control. The worker token grants permission to run tasks but not to manage the swarm. Tokens should be treated as sensitive operational credentials.

A worker can join after retrieving a worker token from a manager:

```bash
docker swarm join-token worker
```

The output includes a `docker swarm join` command with a token and manager address. Running that command on a worker host adds the node to the swarm. A manager can confirm membership with:

```bash
docker node ls
```
## Managing nodes and contexts
SSH access supports direct node management. Vagrant can generate SSH configuration with `vagrant ssh-config`, including host aliases, forwarded ports, usernames, and private key paths. Storing those settings in SSH configuration lets normal `ssh m1` and `ssh w1` commands reach the virtual machines without repeated arguments.

Docker contexts make remote daemon access more convenient. A context can point the Docker CLI at a local daemon, an SSH endpoint, or another Docker endpoint. With SSH configured, a context can target a host alias:

```bash
docker context create m1 --docker host=ssh://m1
docker context create w1 --docker host=ssh://w1
```

The active context controls where Docker commands run. `docker context use m1` changes the default target for later commands. A one-off context flag can target another host without changing the active context:

```bash
docker -c w1 info
```

This distinction prevents accidental context drift. `docker context use` changes future commands. `docker -c` overrides one command only. Scripts that loop over nodes often prefer one-off context arguments because they avoid leaving the shell pointed at the wrong environment.

Contexts clarify the difference between node administration and cluster administration. A command sent to a worker can inspect that worker's local Docker Engine and swarm status. A cluster command that lists nodes, updates node availability, or changes membership must go to a manager. A failed `docker node ls` on a worker is expected behaviour, not a broken cluster.

Visual tools can help during learning. The `dockersamples/visualizer` image shows nodes and swarm-managed tasks. It needs access to the Docker socket on a manager because the manager has the cluster information. Running the visualiser as a standalone container on a manager demonstrates an important point. Standalone containers still run on a swarm node, but they do not appear as swarm service tasks because Swarm does not manage them.

Adding and stopping nodes demonstrates reconciliation. When a new worker joins, managers add it to the cluster state and may place eligible tasks on it. When a worker stops, `docker node ls` eventually reports that it is down. Existing service tasks on that worker become unavailable, and managers schedule replacement tasks on other eligible nodes when the service model allows it. When the worker returns, it becomes available again and can receive work.

Managers can be added for fault tolerance. A single manager is simple, but it has no control-plane redundancy. Multiple managers use Raft consensus and require quorum. A majority of managers must remain available for management operations. Three managers tolerate one manager failure. Five managers tolerate two. Adding managers improves control-plane resilience but also increases consensus overhead, so production swarms usually use an odd number and avoid unnecessary manager count.

Workers can be promoted, and managers can be demoted, as long as the swarm retains a valid manager set. The last manager cannot be demoted because the swarm would lose its control plane. Availability settings also matter. Setting a node to `drain` tells Swarm to move service tasks away from that node. Draining a manager keeps it in the control plane but prevents it from running workload tasks.

Play with Docker provides a quick hosted lab environment for exploring swarm commands. It removes most local virtual machine setup and suits short experiments. Local labs still remain useful when repeatability, editor integration, network inspection, or hardware diversity matters.
## Services and tasks
Services are the main abstraction for running applications in Swarm. A service declares the image, command, published ports, networks, resource settings, update policy, placement rules, and replica count. Managers translate the declaration into tasks. A task is a scheduled container that belongs to a service.

The key difference between a container and a service is desired state. A container run command asks one daemon to start one container. A service definition asks the swarm to maintain a target state over time. If a node disappears, a task exits, or a service is updated, managers compare the actual state with the desired state and take corrective action.

`docker service create` resembles `docker container run` because Docker intentionally keeps the command model familiar. The options differ because a service belongs to the cluster rather than one host. A basic service can look like this:

```bash
docker service create --name weby --replicas 3 --publish published=8080,target=80 nginx
```

The command asks Swarm to run three task replicas and publish the service on port 8080. Managers schedule tasks on eligible nodes. The routing mesh can then accept traffic on the published port from any swarm node and route it to an active task.

Replicated services run a specified number of task replicas. Global services run one task on each available node that matches the placement constraints and resource requirements. Replicated services suit web applications, APIs, workers, and other horizontally scalable components. Global services suit node-level agents, log collectors, monitoring exporters, and utilities that should run once per eligible host.

Placement constraints narrow scheduling. A service can require a node label, a role, an architecture, or another property. Constraints help place workloads near storage, bind services to specific hardware, avoid unsuitable machines, or run a visualiser only on manager nodes. Impossible constraints leave a service pending because managers cannot satisfy the desired state.

A service update changes the desired state. Updating an image, environment variable, constraint, network, port, or replica count causes Swarm to create new tasks and stop outdated ones according to the update policy. `docker service update --force` triggers task replacement even when the service specification stays otherwise unchanged. That command can restart tasks or force re-placement after node or image conditions change.

Inspection reveals how Swarm stores service state. `docker service inspect` shows the current `Spec`, previous specification, version information, endpoint details, update settings, and convergence data. The specification describes what managers want. The task list shows how the swarm is trying to satisfy it.

`docker service ps` shows service tasks and task history. The default task history limit is five, which helps operators see replacements after updates, failures, and re-scheduling events. Custom output formats can make the task list easier to scan. JSON output and `jq` allow precise inspection of IDs, nodes, desired states, current states, errors, and images.

Service logs aggregate output across tasks. `docker service logs` can follow live output and show logs from several replicas without manually finding each underlying container. It suits early troubleshooting and demonstrations. Production log handling should usually send output to a central logging system, because task movement and node failure make host-local logs harder to manage manually.

Swarm events expose cluster activity. `docker system events` can show service creation, task scheduling, health changes, network changes, and removal. Filtering and JSON formatting make event streams readable. Events help explain why the cluster changed, especially during node failures, updates, and service removals.

Removing a service removes its desired state. Managers then shut down related tasks and release the service-level resources that belong only to that service. The operation is separate from image cleanup, volume cleanup, and unrelated network cleanup. Operators should distinguish service removal from host maintenance.
## Stacks and declarative applications
Stacks group related swarm resources under one application name. A stack can define services, overlay networks, named volumes, secrets, and configs in a Compose-style file. `docker stack deploy` creates or updates the stack from that file on a swarm manager.

Stacks reduce command-line repetition. A service can be created with a long `docker service create` command, but a stack file records the configuration in version control. The file becomes the operational contract for the application. It is easier to review, share, edit, and redeploy than a sequence of one-off commands.

A stack deployment can look like this:

```bash
docker stack deploy --compose-file compose.yaml weby
```

The stack name prefixes resources that belong to the stack. A service named `web` in a stack named `weby` becomes `weby_web`. The prefix helps separate applications and avoids naming collisions. Commands such as `docker stack services`, `docker stack ps`, and `docker stack rm` manage stack-level state.

Editing a stack file and redeploying it updates existing services. That workflow makes Swarm more declarative. Operators change the desired state in the file, then ask Swarm to reconcile the cluster. Swarm creates new services, updates changed services, and can prune services that no longer appear in the file when the relevant option is used.

Not every Compose feature maps to Swarm. The stack deployment command accepts Compose files, but Swarm ignores or rejects options that belong to local Compose rather than the Swarm orchestrator. A common example is image building. Local Compose can build an image from a Dockerfile. A swarm node usually needs a published image that each node can pull, so stack deployments should reference registry images. The image should include multi-platform support when the swarm contains mixed architectures.

Stacks reveal Swarm's reconciliation model clearly. A manager can be drained and Swarm will relocate service tasks from that node to other eligible nodes. Restoring node availability allows Swarm to schedule future work there. The cluster keeps comparing actual task placement with desired state and acts when the two differ.

A stack file also records service scale. Replicas can be declared under the deployment section. Updating the replica count and redeploying the stack asks Swarm to start or stop tasks until it reaches the new count. Manual scaling can be useful during experiments, but versioned stack files provide a clearer record for team operations.

The SwarmKit design documentation helps explain the underlying model. Swarm works with objects such as nodes, services, tasks, networks, and orchestration components. Managers accept declarations, store state, schedule tasks, and perform reconciliation. Understanding this model prevents confusion when Swarm replaces a task instead of changing the original container in place.
## Images, platforms, and troubleshooting
A swarm only runs a service when at least one eligible node can run the requested image. Image availability is therefore a cluster concern. If a service references a private image, each node needs registry access and credentials. If nodes use different CPU architectures, the image needs matching manifests or platform-specific tags.

Mixed hardware labs expose this issue quickly. A service that runs on an x86 laptop may fail on Raspberry Pi nodes if the image lacks ARM support. Docker Hub and other registries can show supported platforms. `docker manifest inspect` can also reveal available architectures. Multi-platform images simplify heterogeneous swarms because managers can schedule tasks across different node types without changing the service specification.

Task-level troubleshooting starts with `docker service ps`. A task stuck in pending or rejected state usually reports a reason. Common causes include unsupported image platforms, failed image pulls, invalid constraints, missing secrets, missing configs, unavailable networks, insufficient resources, port conflicts in host publishing mode, and container exit failures.

`docker service inspect` helps confirm what Swarm is trying to do. The service specification may differ from the operator's expectation after several updates. Inspecting the current specification avoids troubleshooting a stale assumption. For a stack, the stack file should also be checked because a redeployment can reapply settings that override manual changes.

Logs and events complement task status. Logs show application output. Task status shows scheduler and runtime outcomes. Events show the timeline of cluster changes. Together they show whether a failure came from image resolution, scheduler placement, container startup, application behaviour, or later node instability.

A failed service update can leave old tasks running or create new failed tasks depending on the update policy and failure action. Update settings should match application risk. Conservative rolling updates suit production web services. Faster replacement suits disposable workers and lab services. Rollback behaviour should be tested before a production incident requires it.

Node labels improve troubleshooting and scheduling clarity. Labels such as `region`, `disk`, `arch`, or `role` express intent. Constraints based on labels make placement visible. They also make incorrect assumptions visible when no nodes match.

Resource reservations prevent overscheduling. A service can reserve CPU or memory so the scheduler only places tasks on nodes with enough available capacity. Without reservations, Swarm may place more work than a node can safely handle, and the Linux kernel may kill processes when memory pressure becomes severe.

Standalone container habits should not be carried into Swarm without review. Binding a host path may work on one node and fail on another. Publishing a fixed host port in host mode can prevent multiple tasks from landing on the same node. Local image builds do not automatically reach every node. Swarm rewards portable images, overlay networks, registry-based distribution, and declarative configuration.
## Swarm networking
Swarm networking makes services reachable across nodes. Overlay networks allow containers on different Docker hosts to communicate as if they share a logical network. The ingress overlay network supports published service ports and routing mesh behaviour. Custom overlay networks support internal service-to-service communication.

The routing mesh is one of Swarm's most visible features. When a service publishes a port through the ingress network, every node can accept connections on that published port. The node that receives the connection does not need to be running a task for that service. Swarm routes the connection to an active task on an available node.

This model simplifies access during demos and small deployments. A load balancer can target all swarm nodes on the published port, and Swarm can route traffic to active tasks. It also has implications. Firewalls must allow the required swarm traffic between nodes, and external resources must reach the published port. Operators can bypass the routing mesh for specific services when direct host publishing better fits the design.

Port publishing has old and new syntax. The clearer long form distinguishes the external published port from the internal container target port:

```bash
docker service create --name web --publish published=8080,target=80 nginx
```

The target port is where the application listens inside the container. The published port is where the swarm exposes the service. Leaving the published port unspecified allows Docker to choose a random high port, which then requires inspection.

Internal overlay networks provide service discovery. Services on the same overlay network can find each other by service name. A frontend service can call a backend service using its service name instead of a specific container IP. Swarm handles task movement and service discovery behind that stable name.

The default endpoint mode uses a virtual IP. DNS returns a stable service address, and Swarm load balances traffic to tasks behind it. The DNS round-robin mode, `dnsrr`, returns task IP addresses directly. DNS round-robin suits cases where an application or external load balancer wants to handle balancing itself. It should be chosen deliberately because it changes how clients see service endpoints.

Connection behaviour affects load-balancing observations. A browser or HTTP client may reuse an existing TCP connection, so repeated refreshes might continue reaching the same task. New connections are more likely to show routing across replicas. Tools such as `curl`, connection-closing headers, or multiple client processes can make routing behaviour easier to inspect.

Network inspection commands reveal how Swarm builds the system. `docker network inspect --verbose ingress` can show peers, subnets, services, and load-balancing details. Inspecting a custom overlay network shows which services attach to it and which endpoints exist. These details help distinguish published external access from internal service communication.

Overlay networks do not remove the need for sound infrastructure networking. Swarm nodes need open ports for cluster management, discovery, and overlay traffic. Firewalls, NAT, VPNs, and cloud security groups must match the swarm design. A lab may appear simple because nodes share a host-only network, but production networks require explicit planning.
## Jobs
Swarm jobs run work to completion. They use service machinery but differ from long-running services. A normal service aims to keep tasks running. A job aims to run tasks until they exit successfully, then leave them in the completed state.

Jobs support replicated and global modes. A replicated job runs a specified number of task iterations. A global job runs one task on each eligible node. Replicated jobs suit batch work, migrations, tests, one-off data processing, and load generation. Global jobs suit per-node operations that should complete once on every matching node.

A minimal replicated job can look like this:

```bash
docker service create --name check --mode replicated-job alpine true
```

A job that needs several successful runs can specify replicas. A job that should limit simultaneous execution can use CLI concurrency controls. This prevents a large batch from starting all tasks at once and overwhelming the target service or cluster.

Jobs can test services through backend overlay networks. A stack can deploy a web application and a job service on the same internal network. The job can call the service by name, generate load, or validate a hypothesis about scalability. Because the job runs inside the swarm, it tests internal service discovery and routing rather than only external ingress.

Logs remain useful for jobs. `docker service logs JOBNAME` can show output from completed job tasks. `docker service ps JOBNAME` shows task instances and their completed or failed states. Completed tasks remain visible until removed, which helps post-run inspection.

Jobs have update caveats. Updating a job can reset task state and stop in-progress work. Rollout and rollback concepts are less meaningful for jobs because the work completes rather than staying online. Operators should design jobs to be repeatable, idempotent where possible, and safe to restart after partial failure.

Global services and global jobs differ in lifecycle. A global service keeps one running task on every eligible node and starts a new one when a matching node appears. A global job runs a task to completion on each eligible node. If a new matching node joins later, Swarm can start a task for that node as well. That behaviour suits node onboarding checks and per-node initialisation tasks.

Job-based load tests can reveal routing and scaling effects. Running more job tasks against a replicated backend can show whether additional replicas improve throughput. It can also expose connection reuse, bottlenecks, resource limits, slow startup, or network constraints. The result should be read as a cluster experiment, not as a substitute for full production-grade performance testing.
## Secrets and configs
Secrets and configs inject runtime data into services without baking that data into images. They keep images generic and reduce the need for hardcoded passwords, environment-specific files, or manual bind mounts.

Secrets hold sensitive values such as passwords, tokens, certificates, and keys. Docker sends secrets to managers over mutual TLS, stores them in the encrypted Raft log, and mounts decrypted values into authorised task containers. On Linux containers, secrets appear by default under `/run/secrets/<secret_name>` in an in-memory filesystem. A task only receives a secret when its service has been granted access to that secret.

A stack can reference an external secret so the sensitive value stays out of the stack file:

```yaml
secrets:
  db_password:
    external: true
```

The service then grants access to the secret by name. The application reads the mounted file at runtime. This approach avoids committing database passwords to version control and keeps deployment configuration separate from secret values.

Secret management has operational rules. A running service can be updated to add or remove secret access. A secret that an active service uses cannot be removed until service access changes. Rotation usually creates a new secret version, updates services to use it, then removes the old secret after no service depends on it. Versioned secret names reduce confusion during rotation.

Configs hold non-sensitive configuration data such as application settings, web server configuration, feature flags, or text files. Configs resemble secrets in the service model, but they are not encrypted at rest and do not use RAM disks. They are appropriate for non-sensitive data only. A password placed in a config is a security mistake.

Both secrets and configs can be defined at the top level of a stack file and then granted to individual services. This two-step model matters. Defining a secret or config makes it available to the stack. Granting it to a service controls which tasks receive it. Least privilege should guide access. A service should receive only the data it needs.

Inside a container, mounts make the injected data visible as files. Inspecting the container can show where the mount appears. The application should read the file path rather than expect the value to be present as an environment variable. File-based injection reduces accidental exposure through process listings, logs, and environment dumps.

Configs and secrets allow the same image to move across environments. The image can stay unchanged while each environment supplies different database credentials, endpoint addresses, or application settings. That separation supports repeatable builds and safer deployments.
## Operational patterns
Swarm operations work best when the cluster is treated as a desired-state system. The operator declares the target state and lets managers reconcile. Manual container fixes on individual nodes may work briefly, but they do not change the desired state and may disappear on the next reconciliation.

Common node operations include listing nodes, inspecting labels and status, changing availability, promoting workers, demoting managers, and removing unavailable nodes. Manager commands should be run against a manager context. Worker contexts suit local Docker Engine inspection and host-specific diagnostics.

Common service operations include creating services, scaling replicas, updating images, adding constraints, changing published ports, inspecting tasks, reading logs, and removing services. The safest recurring workflow stores those settings in stack files where possible.

Monitoring needs several viewpoints:
- Node status shows cluster membership and availability.
- Service lists show declared applications and replica counts.
- Task lists show scheduling and runtime outcomes.
- Logs show application output.
- Events show changes over time.
- Network inspection shows connectivity and discovery state.

No single command explains every failure. A rejected task may need image inspection. A pending task may need constraint or resource review. A running task with bad responses may need application logs. A service unreachable from outside may need published port, routing mesh, firewall, or load balancer checks.

Manager placement deserves care. Managers maintain state and consensus. Production swarms should usually keep managers stable, well-resourced, and distributed across failure domains. Draining managers can reserve them for control-plane work while workers handle application tasks. Small labs can let managers run tasks, but production designs should consider the risk of control-plane resource starvation.

Rolling updates reduce disruption when services support them. A service update policy can control parallelism, delay, monitoring windows, and failure response. Applications should handle termination signals, externalise state, and become healthy only when ready. Swarm can replace tasks, but the application must still behave well during startup, shutdown, and dependency failure.

Images should be tagged deliberately. The `latest` tag may make demonstrations faster, but fixed version tags or digests make production behaviour more predictable. Stack redeployments with moving tags can produce surprising results if different nodes pull at different times. Registries, image policies, and rollout practices should make the running version auditable.

Secrets should never be placed in images or committed stack files. Sensitive values should be created as swarm secrets or supplied through a controlled secret-management process. Configs should carry only non-sensitive data. Both should be named clearly and granted sparingly.
## Core command map
The following commands cover the main learning path.

```bash
docker info
docker swarm init
docker swarm init --advertise-addr <manager-ip>
docker swarm join-token worker
docker swarm join-token manager
docker swarm join --token <token> <manager-ip>:2377
docker node ls
docker node inspect <node>
docker node update --availability drain <node>
docker node update --availability active <node>
docker node promote <node>
docker node demote <node>
```

```bash
docker context ls
docker context create <name> --docker host=ssh://<host>
docker context use <name>
docker -c <name> info
docker -c <name> node ls
```

```bash
docker service create --name <service> --replicas <count> <image>
docker service create --name <service> --mode global <image>
docker service ls
docker service ps <service>
docker service inspect <service>
docker service logs <service>
docker service update --image <image> <service>
docker service update --force <service>
docker service update --constraint-add <constraint> <service>
docker service scale <service>=<count>
docker service rm <service>
```

```bash
docker stack deploy --compose-file <file> <stack>
docker stack services <stack>
docker stack ps <stack>
docker stack rm <stack>
```

```bash
docker network ls
docker network inspect --verbose ingress
docker secret create <name> <file>
docker secret ls
docker secret inspect <name>
docker secret rm <name>
docker config create <name> <file>
docker config ls
docker config inspect <name>
docker config rm <name>
```

These commands are most useful when paired with the underlying model. `docker swarm` manages cluster membership. `docker node` manages hosts in the cluster. `docker service` manages desired application state. `docker stack` manages grouped application resources. `docker network`, `docker secret`, and `docker config` manage supporting resources.
## Practical deployment model
A practical Swarm deployment begins with stable nodes, a reachable manager address, and a deliberate manager count. The cluster then needs image registry access, working overlay networking, and a clear context strategy for administrators. The application layer should start with a stack file rather than a long sequence of commands.

Each service should define an image, networks, ports, replica count, update policy, resource settings, and placement rules. State should be externalised or placed on storage that follows the service's requirements. Services that need secrets or configs should receive them explicitly. Services that should not run on managers should use constraints or drained managers.

External traffic should enter through a planned path. For simple cases, the routing mesh and a published port are enough. For production, an external load balancer or reverse proxy often provides TLS, health checks, and traffic policy. The load balancer can target all eligible swarm nodes on the published port, or it can use a design that bypasses routing mesh when direct task placement is required.

Internal traffic should use custom overlay networks. Frontend services should not automatically share networks with databases unless they need direct access. Separating networks limits accidental reachability and clarifies application structure. Service names should be stable, and endpoint mode should match client behaviour.

Updates should be rehearsed. A stack redeployment can change images, environment, replicas, networks, constraints, secrets, and configs. Swarm reconciles those changes, but the application must support the transition. A service that cannot tolerate multiple versions running at once needs a stricter deployment strategy.

Troubleshooting should start from declared state, then move to actual state. The stack file and service inspection show intent. Service tasks show scheduling and runtime results. Logs show application behaviour. Events show timing. Network inspection shows reachability. Node inspection shows cluster health and labels.

A healthy Swarm workflow favours simple, repeatable operations. It avoids manual per-node container management for clustered applications. It stores application structure in stack files. It keeps secrets out of code. It uses stable manager addresses and supported operating systems. It treats managers as the source of cluster truth and workers as task executors.
## Lab topology and feedback loops
A useful learning topology starts with one manager and one worker. The manager initialises the swarm, and the worker joins by using the worker token. Adding a second worker demonstrates growth. Adding more managers demonstrates control-plane fault tolerance. Stopping a worker demonstrates task replacement and node health reporting. Draining a manager demonstrates that a node can remain in the control plane while avoiding workload placement.

Command-line feedback should stay visible during these changes. A watch command that repeatedly runs `docker node ls` against the manager context can show status changes as they happen. The visualiser can show nodes and service tasks in a browser. These two views reinforce the same concept. Swarm changes cluster state quickly, and the manager view provides the most complete picture.

A good lab prompt shows the current Docker context. Without that cue, it is easy to send a management command to the wrong daemon. A shell prompt can read the active Docker context and show it beside the working directory. That small habit prevents confusing results when several local and remote daemons are in use.

Context-aware commands also make node joins cleaner. An operator can keep the active context on the worker that needs to join, then fetch the join token from the manager through a one-off context argument. The command output from the manager can be pasted directly into the worker context. This pattern avoids full SSH sessions and keeps the target clear.

A worker join should be verified from the manager. `docker info` on the worker confirms that swarm mode is active on that node. `docker node ls` on the manager confirms that the swarm recognises the worker. Both checks matter. The first confirms local membership. The second confirms cluster-level visibility.

A node halt simulates a power loss or sudden host shutdown. Swarm does not instantly delete the node from membership. It marks the node down when heartbeats fail and reconciles affected services. When the node returns, it can rejoin normal operation if its identity and state remain valid. This behaviour helps distinguish temporary failure from deliberate removal.

Joining another manager changes the quorum calculation. A two-manager swarm has poor fault tolerance because both managers are needed for a majority. A three-manager swarm can lose one manager and still keep quorum. That is why an odd number of managers matters. The operator should add managers in a way that increases quorum resilience rather than merely increasing the number of control-plane nodes.

Node promotion and demotion should be deliberate. Promoting a worker to manager grants control-plane authority and stores replicated cluster state on that node. Demoting a manager removes that authority. Production swarms should restrict manager access, harden manager hosts, and avoid promoting nodes simply to make command execution more convenient.
## Visualising services and placement
The visualiser is most useful after services exist. A standalone visualiser container can show the cluster but will not show itself as a service task. Converting the visualiser into a Swarm service makes placement visible. A constraint can keep the task on a manager because it needs access to manager APIs or the Docker socket.

The same image can be run in two different ways. `docker container run` creates a local container controlled by one daemon. `docker service create` creates a service controlled by the swarm. The commands may look similar, but they enter different management systems. The visual difference reinforces the distinction between local runtime state and cluster desired state.

A service with one replica may land on any eligible node. If the task appears on an unexpected node, the scheduler is not wrong. It is using the service specification, node availability, resource constraints, and placement rules available at the time. A placement constraint turns an expectation into declared state.

A service update can change placement without changing the image. Adding a constraint, removing a constraint, or forcing an update can make Swarm replace tasks and schedule them elsewhere. This behaviour proves that tasks are disposable units. The service, not the individual container, carries the application identity.

Task history explains service movement. After an update or failure, old tasks remain visible for a limited history window. A task may show shutdown, rejected, failed, or complete. New tasks show preparing, starting, running, or ready depending on timing and command output. Reading task history helps avoid the common mistake of treating the newest container as the only relevant evidence.

Custom table formats make task output practical. A compact view with task name, node, desired state, current state, error, and image often reveals the problem quickly. JSON output provides more detail for scripts. Both approaches are better than repeatedly inspecting individual containers across several nodes.

Service logs should be read with task movement in mind. A failing task may be replaced several times, and the active task may run on a different node from the failed one. Aggregated service logs reduce the need to SSH into hosts. When logs show that the application started correctly but traffic still fails, attention should move to networking, published ports, endpoint mode, firewalls, or client behaviour.

A service should be removed when it is no longer part of the desired application. Leaving old demo services running can consume ports, networks, and scheduler attention. `docker service rm` removes service intent. `docker stack rm` removes all services and networks managed by a stack. Persistent volumes and external resources may need separate cleanup.
## Stack examples and application shape
A small NGINX service can start as a single command and then move into a stack file. That migration shows why declarative files matter. The service name, image, port publication, replicas, constraints, and networks become visible in one place. Reviewing the file becomes easier than reconstructing command history.

A stack file can define a web service with an image, published port, and replica count. Redeploying the same stack after editing the image or replica count changes the live service. Swarm calculates the difference and applies it. The running application therefore follows the file instead of an operator's memory.

A visualiser stack file can define the visualiser as a service rather than a standalone container. The service can mount the Docker socket, publish port 8080, and apply a manager constraint. This creates a more Swarm-native deployment. The visualiser then appears as a task in the visualiser itself, which makes the difference between standalone containers and services even clearer.

A stack also suits a small echo application that reports its hostname, task slot, and node. Template values can append node or task information to container hostnames. The output then makes routing behaviour visible. A response can show which task handled a request, which node hosted it, and whether repeated requests reused a connection.

Scaling the echo service through the stack file creates more replicas. New tasks appear in the visualiser and in `docker service ps`. Requests through a published port can then show traffic reaching different tasks, subject to connection reuse and load-balancing behaviour. This experiment links the stack file, scheduler, routing mesh, and application output.

Images should be rebuilt and pushed before a stack redeploy when source code changes. In a multi-node swarm, building an image on one laptop does not make it available to other nodes. A registry provides the shared distribution point. A stack file should reference the registry image tag that all nodes can pull.

Mixed architectures make image selection more important. A Raspberry Pi worker needs an ARM-compatible image. An x86 manager needs an x86-compatible image. A multi-platform image manifest lets each node pull the right variant. If a service repeatedly rejects tasks on one hardware type, the image's supported platforms should be checked early.

Stack files can encode constraints that match hardware. A service that needs ARM nodes can target a label or platform. A service that needs managers can target `node.role == manager`. A service that should avoid managers can target worker nodes or rely on drained managers. Explicit placement rules turn lab knowledge into operational truth.
## Internal communication example
A web service that exposes an external port and a backend service on a private overlay network demonstrate two separate paths. External clients use the published port and ingress routing. Internal services use the backend overlay network and service discovery. Mixing these paths can hide mistakes, so each path should be tested separately.

Internal DNS lets a task call another service by name. A stress-runner job can call `http://echo:8080` when both the job and service share an overlay network. The caller does not need to know task IDs or node IP addresses. Swarm updates the service endpoint as tasks move.

The default virtual IP mode gives clients one stable service address. Swarm balances connections behind that address. DNS round-robin mode returns task addresses instead. The right choice depends on the client. Most simple applications should use the default. Applications with their own discovery or balancing logic may need DNS round-robin.

Ingress routing and backend routing answer different questions. Ingress asks how outside traffic reaches the cluster. Backend routing asks how tasks in the cluster reach each other. A service can use both, one, or neither. A database should usually stay off ingress and attach only to networks used by services that require it.

Connection reuse affects experiments. A browser may keep a connection open and send repeated requests to the same task. That can make a scaled service look as though it is not balancing. New client connections, disabled keep-alive, or command-line loops that open fresh connections make distribution easier to see. The observation should account for client behaviour before blaming Swarm.

Network inspection can explain unexpected results. Inspecting the ingress network shows swarm-level routing details. Inspecting the stack's custom overlay network shows attached services and endpoints. A service attached to the wrong network cannot resolve or reach the intended backend by service name. A published port on the wrong service can expose an unintended endpoint.

Firewalls remain part of the design. Overlay networks use Docker's networking components, but packets still move through host networking, virtual switches, cloud security groups, or physical networks. A lab inside one laptop hides many of those concerns. Production environments should document required ports, address ranges, and load balancer paths.
## Job-based validation and load testing
Jobs give Swarm a built-in way to run finite work near the services under test. A replicated job can send a fixed number of requests to a backend service. A global job can run one validation task per eligible node. Both forms help test network reachability, service discovery, and basic capacity without creating a permanent worker service.

A stress-runner stack can define the web application, backend network, and job. The web application stays running. The job starts, makes requests, writes output, and exits. `docker service ps` shows completed and failed task instances. `docker service logs` shows the request results.

Reducing or increasing job scale changes the test pressure. A small job confirms connectivity. A larger job may reveal bottlenecks in the application, service discovery, routing, CPU, memory, or network. The result is most useful when the job records enough output to connect failures with timing and target endpoints.

Jobs should be designed to complete cleanly. A successful task exits with code 0 and is not restarted. A failed task can be retried according to restart settings. The command should return a meaningful exit code so Swarm can distinguish success from failure. Scripts that swallow errors make failed tests look successful.

Job results should not be confused with service health. A completed job means the job command completed. It does not prove ongoing readiness of the application unless the job's checks cover that claim. Jobs work well as smoke tests, migration runners, short data tasks, and lab load generators. Long-lived background processing should usually be a normal service.

Updating jobs requires care because Swarm may stop in-progress job tasks and create new ones. Operators should avoid updating a job that performs non-idempotent work unless the consequences are understood. Repeatable job design reduces the risk of partial completion.
## Runtime data and database example
A database service illustrates why runtime data should be separated from images. The image provides the database software. A volume stores database files. A secret supplies the password. A config can provide non-sensitive settings. The service definition ties those resources together without embedding environment-specific values in the image.

For a MySQL-style service, the root password should come from a secret rather than a literal stack file value. The service can receive a secret and point the application to the mounted file. The container reads the password at startup. The stack file states that the service needs a secret, but the secret's value stays outside the file.

Inspecting a running task can show the mounted secret path. Inside Linux containers, Docker commonly mounts secrets under `/run/secrets`. Reading that file as an administrator during a lab confirms the mechanism. In normal operations, applications should read the path automatically, and operators should avoid unnecessary manual exposure of secret values.

Secret rotation should avoid breaking running services. A common pattern creates a new secret with a versioned name, updates the service to consume the new secret, verifies the application, then removes access to the old secret and deletes it when safe. Reusing the same secret name for changed content is less clear because services may need explicit updates and operators may lose track of which value is active.

Configs follow a similar grant model but carry non-sensitive content. A web server config can be created as a config and mounted into the service. Updating a config usually means creating a new config object, updating the service to reference it, and removing the old one after the service no longer uses it. This mirrors safe secret rotation without treating non-sensitive data as secret.

The separation of image, secret, config, volume, and service definition gives Swarm applications cleaner boundaries. Images become portable. Secrets become controlled. Configs become inspectable. Volumes hold state. Services declare how the pieces fit together. That structure is easier to operate than images that contain passwords or node-specific files.