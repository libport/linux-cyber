# Getting Started with Docker Swarm
> [!NOTE]
> This guide explains how Docker Swarm orchestrates scalable, resilient multi-host applications through clustered nodes, desired-state services, declarative stacks, overlay networking, secure configuration, and automated workload management.
## Swarm mode in context
Docker Swarm mode turns a group of Docker Engine hosts into one orchestrated cluster. Managers maintain the cluster's desired state, assign work, and expose the Swarm management API. Workers run the assigned containers. The scheduler can distribute replicated services, run one task on each eligible node, execute finite jobs, replace failed tasks, publish ports across the cluster, and update an application in controlled stages.

Swarm mode suits teams that need orchestration within Docker Engine and prefer a comparatively compact operational model. It does not remove the need for sound infrastructure design. Production operators still need resilient managers, reliable storage, secure networking, image governance, monitoring, backups, capacity planning, and tested recovery procedures. Platform selection should account for workload requirements, available integrations, team experience, and the expected life of the system.

A swarm manages services rather than isolated containers. A service describes the desired image, command, replica mode, networks, published ports, resources, placement rules, update policy, secrets, and configuration. The scheduler converts that declaration into tasks and assigns each task to a node. Each task controls one container. When a task fails or a node disappears, the orchestrator creates a replacement task if the service still requires one.

This desired-state approach changes the operating model. An administrator declares an outcome, such as five web replicas, and the swarm continually works towards that outcome. Directly starting, renaming, or repairing individual service containers bypasses the orchestrator and produces short-lived changes. Service and stack commands should define lasting changes.
## Cluster architecture
### Nodes and roles
A node is a Docker Engine instance participating in the swarm. Every node holds one of two roles:
- A manager accepts cluster-management requests, maintains orchestration state, schedules tasks, and participates in the Raft consensus group.
- A worker receives task assignments from managers and reports task status.

Managers also act as workers by default, so the scheduler may place service tasks on them. A small non-critical swarm can use this mixed role. A production cluster often drains managers so application load cannot compete with control-plane work. Draining changes the placement of Swarm service tasks. It does not stop or migrate standalone containers that operators created outside Swarm mode.

The manager set stores the authoritative cluster state in a replicated Raft log. Raft requires a majority of managers to agree before the swarm commits management changes. This quorum protects consistency, but it also determines failure tolerance. An odd number of managers usually uses resources more efficiently than the next even number:

| Managers | Quorum | Manager failures tolerated |
| ---: | ---: | ---: |
| 1 | 1 | 0 |
| 2 | 2 | 0 |
| 3 | 2 | 1 |
| 4 | 3 | 1 |
| 5 | 3 | 2 |
| 7 | 4 | 3 |

Three or five managers serve most production swarms. Docker recommends no more than seven because additional managers increase consensus traffic without increasing workload capacity. Two managers create a fragile design because the loss of either manager removes quorum. Seven managers tolerate three failures but impose more coordination overhead than five.

When the manager set loses quorum, existing tasks can continue to run, but the cluster cannot reliably accept management operations or reconcile all desired-state changes. Restoring quorum takes priority. Operators should not promote extra managers during an outage without understanding the surviving Raft membership, because unplanned membership changes can complicate recovery.
### Services, tasks, and containers
A service is a durable declaration. A task is an immutable scheduling unit that moves through states such as `NEW`, `ASSIGNED`, `PREPARING`, `STARTING`, `RUNNING`, `FAILED`, `SHUTDOWN`, and `COMPLETE`. A task belongs to one service and runs on one node. The scheduler never moves the same task to another node. It creates a new task with a new identity when replacement becomes necessary.

This distinction explains common command output. `docker service ls` reports each service and its overall replica state. `docker service ps SERVICE` lists current and historical tasks. `docker ps` shows only containers on the Docker Engine addressed by the current client. A failed container can therefore remain in task history while a new container runs elsewhere.

Swarm supports four main service modes:
- `replicated` maintains a specified number of continuously running tasks.
- `global` maintains one continuously running task on every active node that satisfies placement rules.
- `replicated-job` runs a specified number of tasks to successful completion.
- `global-job` runs one task to successful completion on each eligible node, including an eligible node that joins later.

A replicated service with one replica per current node does not equal a global service. The replica count stays fixed when another node joins, while a global service automatically creates a task for the new eligible node.
### Reconciliation
Managers continuously compare observed state with desired state. If a five-replica service has only four running tasks, the scheduler creates another. If a worker fails, the managers mark its tasks unavailable and schedule replacements where constraints and resources permit. If a failed node returns after replacements have started, Swarm does not merge the old and new containers. It brings the service back to its declared replica count.

Reconciliation depends on accurate declarations. A container that starts successfully but cannot serve traffic may still appear healthy unless the image defines an effective health check. The scheduler can place a service without reservations on a node that lacks practical capacity. A service tied to host-local data may restart on another node without the data it needs. Orchestration can replace computation, but it cannot infer application correctness or relocate storage by itself.
## Trust, identity, and network prerequisites
### Mutual TLS and node identity
Swarm mode creates a public key infrastructure for node identity. The first manager establishes a root certificate authority. Joining nodes receive certificates and use mutual TLS for authenticated control-plane communication. Docker rotates node certificates automatically according to the configured expiry period.

Join tokens authorise a new engine to request either a worker or manager certificate. They are sensitive bootstrap credentials, not ordinary command examples for public documentation or shared logs. An operator should retrieve them only through an authorised manager, transmit them through a protected channel, and rotate them after suspected disclosure:

```shell
docker swarm join-token worker
docker swarm join-token manager
docker swarm join-token --rotate worker
docker swarm join-token --rotate manager
```

The worker token allows a host to join as a worker. The manager token grants a path into the manager set and requires stronger protection. Existing nodes continue to use their issued certificates after token rotation. The new token controls later joins.

Docker encrypts the Raft log at rest by default. Manager autolock adds another barrier by requiring an unlock key after a manager restarts. This control protects the keys used to decrypt Raft data when the manager is offline. It also introduces a recovery dependency. The organisation must store the unlock key securely, restrict access, and test restart procedures before enabling autolock broadly.

```shell
docker swarm update --autolock=true
docker swarm unlock-key
docker swarm unlock
```

Mutual TLS does not make every workload flow confidential. It secures Swarm control traffic. Application traffic on an overlay network requires separate encryption when a workload needs confidentiality across hosts.
### Required connectivity
Every participating host needs stable addressing, time synchronisation, compatible Docker Engine versions, working name resolution where hosts use names, and bidirectional connectivity for Swarm traffic. Firewalls between nodes must allow:
- TCP port 2377 for manager communication and node joins
- TCP and UDP port 7946 for node discovery and gossip
- UDP port 4789 for VXLAN overlay traffic
- IP protocol 50 when encrypted overlay networking uses IPsec ESP

Port 4789 should remain confined to a trusted network because VXLAN traffic does not authenticate peers on its own. Internet-facing exposure creates avoidable risk. Network devices must also pass the selected maximum transmission unit without harmful fragmentation. Encapsulation adds overhead, so cloud networks, virtual private networks, and tunnels may require a reduced overlay MTU.

The address advertised by a manager or worker must remain reachable from the other nodes. A multi-homed host should advertise the interface intended for cluster communication rather than whichever address Docker happens to select. When management and application data need separate paths, `--advertise-addr` identifies manager traffic and `--data-path-addr` identifies overlay data traffic.
### Docker daemon authority
Access to a Docker daemon carries host-level consequences. A user who can control the daemon can normally mount host paths, start privileged containers, and acquire extensive control of that host. Remote administration through a Docker context over SSH is convenient because SSH supplies authentication and transport protection, but the remote account still receives Docker authority.

```shell
docker context create swarm-manager \
  --docker host=ssh://swarmadmin@manager1.example.net
docker context use swarm-manager
docker info
```

The account, SSH key, host key, agent, and local workstation all form part of the trust boundary. Administrators should use dedicated accounts, least privilege where the environment supports it, protected private keys, verified host keys, and audited access. Administrators must never expose an unauthenticated Docker TCP socket.

Mounting `/var/run/docker.sock` into a container grants that container powerful access to the host daemon. Demonstration tools that inspect a swarm through the socket can help in a disposable lab, but they should not become public production dashboards. A compromised socket-enabled container can control other containers and often the host.
### Certificate authority lifecycle
The swarm certificate authority establishes the identity of every node, so its lifecycle needs stronger controls than routine service configuration. `docker swarm ca` displays the current root certificate, while `docker swarm update --cert-expiry` changes the validity period for node certificates. Shorter periods reduce the useful life of a stolen node certificate but increase dependence on reliable renewal. Managers renew certificates automatically, and operators should monitor failures rather than assume rotation always succeeds.

The root certificate authority can use Swarm's internal key or an external signing service. An external authority can align issuance with organisational controls, but it adds availability, access, and recovery dependencies. The selected design should document who can rotate the root, where signing keys reside, how an offline recovery works, and which alerts identify approaching expiry.

Root rotation changes the trust foundation for the whole cluster. Docker supports phased rotation so nodes transition through cross-signed certificates, but an interrupted or poorly planned operation can isolate nodes. A production change should begin with healthy quorum, current backups, stable node communication, and a tested rollback or recovery procedure. Administrators should not rotate the root during an unrelated outage.

A node certificate proves membership and role, not application identity. Services still need their own authentication and TLS where clients, APIs, databases, or brokers require end-to-end trust. Combining the cluster certificate authority with application certificates can blur ownership and expand the impact of a cluster administration error.

Node decommissioning should remove both access and cluster membership. The operator drains the node, confirms replacement tasks, makes the engine leave, removes its node record, revokes its infrastructure credentials, and erases retained swarm data according to policy. Rotating join tokens does not revoke an already issued node certificate. A suspected active-node compromise requires containment and removal, followed by assessment of manager, secret, and workload exposure.
## Creating and joining a swarm
### Prepare the hosts
Cluster creation should start with infrastructure decisions rather than commands. Each host needs a unique hostname, a stable address, current security updates, and sufficient CPU, memory, disk, and network capacity. Managers need dependable storage for Raft state. All nodes need registry access for the images that services will use. Production managers should occupy separate failure domains where possible so one rack, zone, power source, or network device cannot remove quorum.

A practical pre-flight check covers the following conditions:
- Docker Engine runs correctly on every host.
- Required ports and protocols pass between the intended interfaces.
- Clocks remain synchronised.
- Hostnames and addresses resolve consistently.
- Kernel modules and security controls support overlay networking.
- Image registries, log destinations, monitoring systems, and storage services are reachable.
- Named owners control each backup and recovery process.

Disposable virtual machines offer a useful learning environment, but a lab should use a maintained operating-system release. Old examples built around end-of-life distributions, experimental registry tools, or temporary browser laboratories should not define production practice.
### Initialise the first manager
The first manager creates the swarm and becomes its initial leader:

```shell
docker swarm init \
  --advertise-addr 10.20.0.11 \
  --data-path-addr 10.30.0.11
```

If one interface carries both paths, the operator can omit `--data-path-addr`. The command returns a worker join command containing the worker token and manager address. Operators should handle that displayed token as a secret.

Several commands confirm the new state:

```shell
docker info
docker node ls
docker swarm ca
```

`docker info` reports whether Swarm mode is active and summarises managers and nodes. `docker node ls` runs on a manager and lists node readiness, availability, role, and manager status. The first manager normally shows both `Leader` and `Ready`.
### Join workers and managers
A worker joins by running the generated worker command on that host:

```shell
docker swarm join \
  --token WORKER_JOIN_TOKEN \
  --advertise-addr 10.20.0.21 \
  --data-path-addr 10.30.0.21 \
  10.20.0.11:2377
```

Additional managers use the manager token:

```shell
docker swarm join \
  --token MANAGER_JOIN_TOKEN \
  --advertise-addr 10.20.0.12 \
  --data-path-addr 10.30.0.12 \
  10.20.0.11:2377
```

The join address can name any reachable manager, not only the current leader. Once joined, managers replicate cluster state and participate in consensus. Workers receive assignments but do not hold voting membership.

A joining node should appear as `Ready` and `Active` in `docker node ls`. Those labels have distinct meanings. `Ready` reports node health from the manager's perspective. `Active` means the scheduler may place tasks there. `Pause` prevents new task assignments without moving existing tasks. `Drain` prevents new assignments and causes the scheduler to replace existing service tasks elsewhere.

```shell
docker node update --availability drain worker2
docker node update --availability active worker2
```

Draining is appropriate before maintenance, but it can overload remaining nodes if capacity is tight. The operator should confirm replica placement and service health before shutting down the drained host.
### Change roles carefully
Managers can promote workers and demote managers:

```shell
docker node promote worker3
docker node demote manager3
```

Role changes alter the quorum design and should follow an explicit plan. Operators must not demote a manager if doing so would leave the cluster without quorum. A new manager needs time to catch up with the Raft log before the existing set changes again. Routine manager rotation should change one node at a time and verify cluster health between changes.

A node can leave voluntarily:

```shell
docker swarm leave
```

A manager normally refuses to leave if that action could damage the swarm. `--force` bypasses safeguards and belongs only in understood recovery or decommissioning procedures. After a node leaves permanently, a manager can remove its stale record with `docker node rm`, adding `--force` only when operators cannot make that node leave the swarm.
### Labels and availability
Node labels express infrastructure properties that the scheduler cannot discover safely on its own. An administrator might label storage nodes, accelerator hosts, zones, or compliance domains:

```shell
docker node update --label-add disktype=ssd worker1
docker node update --label-add zone=west worker1
docker node update --label-add zone=east worker2
```

Labels should describe stable capabilities rather than temporary preferences. Placement constraints can make a service impossible to schedule if nodes lack labels or reservations exhaust capacity. A label taxonomy therefore needs ownership, validation, and change control.
## Manager resilience and recovery
Manager resilience combines quorum design, failure-domain separation, backups, and tested restoration. Replication protects the live cluster from some node failures, but it does not replace backups. A faulty command, corrupted state, lost credentials, or widespread infrastructure event can affect every replica.

Only managers contain the Raft data required to restore a swarm. Docker stores that state under the engine data directory, commonly `/var/lib/docker/swarm`. A consistent backup requires stopping Docker on the selected manager before copying the swarm data. Hot copies can capture an inconsistent database. The backup process must also retain any autolock material needed for restoration and protect the archive as sensitive data because it contains cluster configuration and secrets.

A recovery plan should record the following details:
- Which manager produces the backup
- How the process obtains a consistent snapshot
- Where the organisation retains encrypted copies
- Who can retrieve the autolock key
- How often operators test restoration
- Which applications require separate data backups
- How DNS, load balancers, registries, and external secret systems reconnect after recovery

Regular exercises should confirm that operators can identify the leader, restore quorum, retrieve protected keys, redeploy declarations, recover application data, and verify service health without depending on one person, host, or undocumented credential during an outage.

If failures permanently remove a majority of managers, normal operation cannot resume through worker promotion alone. An authorised operator can restore a surviving manager from backup or force a new cluster from surviving Raft state. That operation creates a new consensus group and requires careful validation. Operators should rehearse recovery procedures in isolation rather than invent them during an actual incident.

Applications also need their own recovery design. Swarm recreates tasks, not database records or host-local volumes. A task rescheduled from one host to another does not carry a local named volume with it. Stateful services need storage accessible from replacement nodes, an external managed data service, a distributed storage system, or application-native replication. Backups, restore tests, consistency controls, and data-loss objectives remain application responsibilities.
## Deploying and operating services
### Create a replicated service
An administrator creates a service from a manager:

```shell
docker service create \
  --name web \
  --replicas 3 \
  --publish published=8080,target=80 \
  nginx:1.27.5
```

The manager records the declaration and schedules three tasks. Each eligible node pulls the image if necessary. A mutable tag such as `latest` weakens repeatability because nodes can resolve it to different content over time. Production deployments should use a controlled version tag and an image digest when releases require stronger immutability.

Operators can examine service status at several levels:

```shell
docker service ls
docker service inspect --pretty web
docker service ps --no-trunc web
docker node ps worker1
```

`docker service inspect` shows the service specification, including image reference, mode, update policy, endpoint settings, and task template. `docker service ps` shows placement and task history. Truncated error messages can hide the useful cause, so `--no-trunc` helps with failed pulls, rejected tasks, and runtime errors.

The service name supplies a stable operational identity while individual task and container names change. Monitoring, routing, and automation should follow the service and task metadata rather than assume a specific container will survive.
### Scale and reconcile
A replicated service can change its desired count without recreation:

```shell
docker service scale web=6
docker service update --replicas 4 web
```

Scale-out creates additional tasks. Scale-in selects excess tasks for shutdown. Swarm does not know whether a task holds an in-memory session or unfinished request, so the application and proxy should support graceful termination. A suitable stop signal, shutdown timeout, connection draining strategy, and stateless request path reduce disruption.

The scheduler can maintain the count only when eligible nodes have enough reserved resources. If constraints allow one node and that node fails, tasks stay pending. If every node lacks the requested memory reservation, tasks also stay pending. Increasing replicas cannot create capacity.

For a global service, scale flags do not apply. The scheduler derives the count from active eligible nodes. Global services suit node agents such as telemetry collectors, security sensors, and local proxies, provided the image can safely run once per node.
### Placement and capacity
Placement constraints impose hard requirements. Placement preferences influence distribution but do not make a placement mandatory. Resource reservations tell the scheduler what capacity a task requires. Limits constrain what a running container may consume.

```shell
docker service create \
  --name api \
  --replicas 4 \
  --constraint 'node.labels.zone!=edge' \
  --placement-pref 'spread=node.labels.zone' \
  --reserve-memory 256M \
  --limit-memory 512M \
  --reserve-cpu 0.25 \
  --limit-cpu 1 \
  registry.example.net/shop/api:4.8.2
```

The constraint excludes nodes labelled `zone=edge`. The preference spreads tasks across the remaining zone labels as far as other rules permit. Reservations guide scheduling. Limits protect neighbouring workloads, although a memory limit can terminate a container if the process exceeds it.

`--replicas-max-per-node` can prevent too many replicas of one service from landing on a single host. This option helps preserve fault tolerance when the cluster has enough nodes. If the requested replica count exceeds eligible slots, the remaining tasks stay pending rather than violate the maximum.

Placement design should account for correlated failure. Four replicas on one physical host do not provide host resilience. Four replicas across separate hosts but one shared power source still share a larger failure domain. Labels and spread preferences can represent zones, racks, or other operational boundaries.
### Restart policies and health checks
A restart policy controls when Swarm replaces a failed task, how long it waits, how many attempts it makes within a window, and which exit conditions qualify. A tight retry loop can amplify an outage by consuming CPU, flooding dependencies, and generating logs. Backoff and application-level retry discipline reduce that risk.

An image health check tests whether a running container can perform useful work. The test should be fast, local, and representative of the process without placing heavy load on dependencies. It should not declare failure because an optional remote system briefly slows down. When a service task becomes unhealthy, Swarm can replace it according to the service's restart policy.

Health checks need realistic start periods for applications that initialise caches, apply migrations, or compile assets. A test that starts too early can trigger a replacement loop. A test that only checks whether a process exists can miss a deadlocked or unusable service. Operators should validate both failure detection and recovery behaviour.
### Diagnose pending and rejected tasks
A task in `PENDING` usually indicates that the scheduler cannot find an eligible node. Common causes include unsatisfied constraints, missing node labels, insufficient reserved CPU or memory, an unavailable network, and a per-node replica maximum. `docker service ps --no-trunc` exposes the scheduler's message, while `docker node inspect` confirms labels, resources, availability, and platform details.

A task in `REJECTED` reached a node, but the node could not prepare or start it. Image pull failures, unavailable secrets or configs, invalid mounts, occupied host ports, unsupported platform variants, and runtime security rules can all cause rejection. The first error deserves attention because repeated replacements often produce many later task records with the same cause.

Diagnosis should preserve the declared state while gathering evidence. Manually running a similar container can test an image, but it does not prove that the service has the same networks, secrets, mounts, identity, resources, or security settings. A useful comparison inspects the full task specification and recreates only the minimum safe conditions in a controlled environment.

If a registry pull fails on one node, the operator should check that node's DNS, routing, trust store, proxy configuration, credentials, disk space, and architecture. A successful pull on the manager proves little about workers because each eligible node retrieves its own layers. Private registries also need consistent certificate-authority installation across the cluster.

If a task starts and exits repeatedly, the container's exit code and logs provide the immediate evidence. The investigation should then distinguish an application defect from a missing dependency, incompatible configuration, memory termination, failed health check, or signal-driven shutdown. Increasing the restart limit or disabling the health check can hide the signal without correcting the cause.

Operators should record the service specification before urgent changes. `docker service inspect` produces a durable snapshot of image identity, environment, mounts, networks, placement, resources, and update settings. That record supports comparison after recovery and reduces undocumented incident changes.
### Update a service
`docker service update` modifies the service declaration. Changing an image normally creates replacement tasks according to the update policy:

```shell
docker service update \
  --image registry.example.net/shop/api:4.9.0 \
  --update-parallelism 2 \
  --update-delay 10s \
  --update-monitor 30s \
  --update-failure-action rollback \
  --rollback-parallelism 1 \
  api
```

Parallelism controls how many tasks change together. Delay spaces the groups. The monitor period allows a new task to demonstrate stability before the update advances. A failure action can pause, continue, or roll back the update. Rollback returns to the specification before the most recent service update, not to an arbitrary historical release.

Update order introduces a capacity and availability trade-off. `stop-first` stops the old task before starting its replacement and avoids overlap. `start-first` starts the replacement first and can reduce interruption, but the node briefly needs resources for both tasks. The application must also tolerate overlapping versions and duplicate workers.

A force update replaces tasks without changing the image or other main settings:

```shell
docker service update --force web
```

This operation can refresh containers after an external dependency or node-level change, but it should not substitute for a versioned deployment. Mutable images combined with force updates can produce different binaries under one service specification.
### Remove a service
Removing a service deletes its desired state and stops its tasks:

```shell
docker service rm web
```

The operation does not necessarily remove named volumes, external networks, registry images, or application data. Decommissioning needs a separate retention decision for each resource. Removing the last task before preserving required data can cause irreversible loss.
## Defining applications as stacks
### Stack files and compatibility
A stack groups related services, networks, volumes, configs, and secrets under one name. `docker stack deploy` reads a Compose-like YAML file and applies the declaration from a manager. It does not use the full current Compose Specification. Docker documents stack deployment as using the legacy Compose file version 3 format, so the stack command may reject or ignore features that `docker compose` accepts.

The deployment pipeline should validate the file with the actual target command and Engine version. A successful local `docker compose up` does not prove Swarm compatibility. Swarm also does not build images from `build` instructions. The pipeline must build, test, scan, push, and identify images before the stack deploys them.

A compact stack can define a web service and an internal API:

```yaml
version: "3.9"

services:
  web:
    image: registry.example.net/shop/web:3.4.1
    ports:
      - target: 8080
        published: 443
        protocol: tcp
        mode: ingress
    networks:
      - front
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
        failure_action: rollback
        monitor: 30s
      rollback_config:
        parallelism: 1
        delay: 5s
      restart_policy:
        condition: on-failure
      resources:
        reservations:
          memory: 128M
        limits:
          memory: 256M

  api:
    image: registry.example.net/shop/api:4.9.0
    networks:
      - front
      - data
    deploy:
      replicas: 4
      placement:
        preferences:
          - spread: node.labels.zone

networks:
  front:
    driver: overlay
  data:
    driver: overlay
```

The example publishes only the web tier. The API remains reachable through service discovery on shared overlay networks. Separating `front` and `data` reduces unnecessary connectivity. Network separation is a useful boundary, but it does not replace application authentication and authorisation.
### Deploy, inspect, and update a stack
The manager deploys the file under a stack name:

```shell
docker stack deploy \
  --compose-file stack.yml \
  --with-registry-auth \
  shop
```

`--with-registry-auth` sends the client's registry authentication details to Swarm agents when private-image pulls require them. Those credentials need limited scope and secure handling. A registry identity dedicated to deployment reduces the exposure created by forwarding a developer's broad credentials.

Stack objects receive a name prefix, such as `shop_web`. The following commands show the resulting state:

```shell
docker stack ls
docker stack services shop
docker stack ps --no-trunc shop
```

Redeploying the same stack reconciles changed declarations. Operators should review the plan, verify image availability, monitor the rollout, and confirm application behaviour. If a service disappears from the file, ordinary deployment may leave it in the stack. `--prune` removes services that the current file no longer defines, so a deployment using that option needs careful review.

```shell
docker stack deploy \
  --compose-file stack.yml \
  --prune \
  shop
```

Stack removal stops the stack's services and removes stack-managed networks, but external resources remain. Operators must verify named-volume and data-retention behaviour for the actual declaration before removal.
### Image discipline
Each node pulls service images independently. Every eligible node must reliably reach the private image registry. The same tag should resolve to the same manifest for the duration of a rollout. Content digests provide the strongest identity, while controlled immutable tags offer readable release labels.

The image process should cover provenance, supported base images, vulnerability scanning, signatures or attestations where required, software bills of materials, and retention. An image labelled official or popular does not remove the need for evaluation. Production teams should verify the publisher, source, maintenance status, architecture support, and security history.

Multi-platform images help heterogeneous clusters only when each platform variant behaves consistently. Placement constraints can restrict an image to compatible operating systems or processor architectures when required.
### Volumes and external resources
A stack declaration can create a named volume, but the volume driver determines where its data lives. With the default local driver, a volume belongs to one node. If Swarm replaces a task on another node, Docker can create a new volume with the same logical name on that node, but the new volume does not contain the old node's data.

Placement constraints can pin a stateful task to the node that holds its local data. That approach preserves access during normal rescheduling decisions but turns the node into a service dependency. When the node fails, Swarm cannot deliver the data elsewhere. This design may suit reconstructable caches or low-value laboratory data, not a service that claims transparent host failover.

Shared or remote volume drivers can expose storage from multiple nodes, but they introduce their own consistency, performance, fencing, credential, and availability requirements. The application may also require exclusive writers or database-native replication. Teams should test the storage design under node loss, network interruption, concurrent starts, and recovery instead of inferring resilience from a successful mount.

External resources allow a stack to refer to a network, volume, config, or secret whose lifecycle sits outside that stack. This separation helps when several stacks share an ingress network or a platform team controls a credential. The resource must already exist with the expected name before deployment, and stack removal will not delete it.

External ownership needs an explicit contract. The producing process defines creation, updates, access, backup, and retirement. The consuming stack defines the expected interface and failure response. Without that contract, a stack can deploy successfully while relying on an untracked object that another team later changes or removes.

Bind mounts tie a task to a path on each host and reduce portability. Every eligible node must provide equivalent content, permissions, labels, and mount propagation. Host configuration management can enforce those conditions, but an image, config, secret, or managed volume often expresses the dependency more safely.
## Swarm networking
### Network types
Swarm uses several network layers:
- The `ingress` overlay network carries routing-mesh traffic for published ports.
- User-defined overlay networks connect service tasks across nodes.
- The `docker_gwbridge` bridge connects overlay networks to a node's physical network.
- Host networking and third-party drivers cover specialised cases but change isolation and portability.

An overlay network spans Docker hosts participating in the swarm. Swarm creates the network control state on managers and instantiates local components on nodes that run attached tasks. Services on the same overlay use built-in DNS discovery. Services on different overlays cannot communicate through those networks unless another path connects them.

```shell
docker network create \
  --driver overlay \
  --subnet 10.40.0.0/24 \
  app-net
```

Docker recommends `/24` blocks for overlay networks using the default virtual IP endpoint mode. A design requiring more endpoints should use multiple overlay networks or consider DNS round-robin with an external load balancer rather than simply enlarging the subnet.

The `--attachable` option allows manually started containers to join an overlay network. That flexibility expands the set of principals that can reach the services, so teams should enable it only when a concrete operational need exists.

```shell
docker network create \
  --driver overlay \
  --attachable \
  diagnostics-net
```
### Service discovery
Swarm registers each service name in its embedded DNS. By default, a lookup returns a virtual IP. The virtual IP remains stable for the service while Swarm load balances connections among healthy tasks. Clients therefore address `api` rather than discover individual container addresses.

DNS round-robin mode returns task addresses instead of a virtual IP:

```shell
docker service create \
  --name search \
  --network app-net \
  --endpoint-mode dnsrr \
  --replicas 3 \
  registry.example.net/search:2.1.0
```

`dnsrr` suits an external or application-aware load balancer that needs backend addresses. The client must respect DNS changes and failed-task removal. Cached addresses, long DNS time-to-live behaviour, and connection pools can delay failover. DNS round-robin also has compatibility limits with the ingress routing mesh, so the publishing design needs validation.

Service load balancing works at the connection level. One long-lived TCP connection remains associated with one backend. A browser or proxy using keep-alive may therefore send many requests to one task even when fresh connections distribute across tasks. Uneven request counts do not by themselves prove that the scheduler placed replicas incorrectly.
### Published ports and the routing mesh
Ingress publishing exposes a service port on every swarm node. A request can arrive at a node that runs no task for the service. The routing mesh forwards that connection to an available task through the ingress network.

```shell
docker service create \
  --name web \
  --publish published=8080,target=80,mode=ingress \
  --replicas 3 \
  nginx:1.27.5
```

An external load balancer can target all suitable node addresses on port 8080. Because every node accepts the published port in ingress mode, the load balancer does not need task-level discovery. Health checks should still remove failed nodes and account for maintenance states.

Host publishing bypasses the routing mesh:

```shell
docker service create \
  --name edge \
  --mode global \
  --publish published=8443,target=8443,mode=host \
  registry.example.net/edge:5.2.0
```

In host mode, only a node with a task listens through that published mapping. An external load balancer must know which nodes run tasks. A fixed published port can conflict if multiple tasks of the same service land on one node, so global mode or a one-replica-per-node rule often accompanies host publishing.

Ingress mode adds a network hop in some request paths and hides direct task placement from clients. Host mode can improve path control and preserve source characteristics in some designs, but it transfers more service-discovery work to the external load balancer. The choice should follow measured requirements for latency, observability, topology, and failure handling.
### Overlay encryption
Swarm encrypts control-plane communication, but overlay application data remains unencrypted by default. A user-defined overlay can enable IPsec encryption:

```shell
docker network create \
  --driver overlay \
  --opt encrypted \
  secure-app-net
```

Encrypted overlays impose a performance cost. Operators should benchmark representative throughput, latency, CPU use, and packet size before adoption. All hosts must permit IP protocol 50 and use kernels that support the required encryption path. Encryption protects packets between nodes. It does not replace TLS between applications when end-to-end identity, termination control, or traffic inspection requires it.
### Troubleshooting connectivity
Network diagnosis should move from declarations to paths. The operator can confirm the service's network attachments, task placement, node readiness, published-port mode, and DNS response before capturing packets or changing firewall rules.

Useful commands include:

```shell
docker network inspect app-net
docker service inspect --pretty web
docker service ps --no-trunc web
docker node ls
```

A short-lived diagnostic service or authorised container on the same network can test DNS, ports, routes, and application responses. Operators should control diagnostic images and remove them after use. Attaching an arbitrary troubleshooting container to a sensitive overlay can bypass intended segmentation.

Common causes include blocked 7946 or 4789 traffic, an incorrect advertised address, an MTU mismatch, stale external load-balancer targets, missing network attachment, port-mode assumptions, and an application bound only to loopback inside its container. A connection test should distinguish DNS resolution, TCP establishment, TLS negotiation, and application response rather than describe every failure as a network fault.
## Updates, rollbacks, and observability
### Design updates before deployment
A service update should define acceptable concurrency, delay, failure thresholds, monitoring time, order, and rollback behaviour before an incident. The stack format expresses these controls under `deploy.update_config` and `deploy.rollback_config`. Command-line flags provide equivalent controls for directly managed services.

Small update groups limit the blast radius but prolong a rollout. Large groups finish quickly but can remove too much capacity at once. `start-first` may preserve capacity but needs temporary headroom. `stop-first` uses less peak capacity but can reduce availability. A database migration can make rollback unsafe even when Swarm can restore the former container image, so schema changes need a compatible application strategy.

A successful scheduler rollout does not prove a successful release. Health checks can confirm local process behaviour, but only service-level telemetry can confirm request success, latency, queue depth, data correctness, and user outcomes. Deployment automation should observe both Swarm state and application indicators.

If an update pauses, the operator should inspect failed tasks before resuming or rolling back:

```shell
docker service ps --no-trunc api
docker service inspect --pretty api
docker service logs --since 15m api
docker service rollback api
```

Rollback restores only the previous service specification. It cannot reverse external side effects, data migrations, messages, or configuration changes outside that specification.
### Logs
`docker service logs` aggregates output from the tasks of a service or shows output for one task. Docker supports this command only for services using the `json-file` or `journald` logging driver.

```shell
docker service logs \
  --since 30m \
  --timestamps \
  --follow \
  api
```

Local container logs are not a durable cluster-wide observability system. Node loss can remove them, unlimited files can exhaust disk, and task replacement separates related output across containers. Docker's default `json-file` driver does not rotate logs unless configured. The `local` driver rotates by default and uses a more space-efficient format, but `docker service logs` does not support it. Production systems often forward structured logs to a central destination and set local retention limits.

Applications should write operational output to standard output and standard error, add timestamps or rely on the collector's timestamps consistently, include correlation identifiers, and avoid secrets. Logging volume needs limits because an outage can multiply output across replicas.
### Events and metrics
`docker events` streams daemon and Swarm events and can filter by object type, event, label, or time. Local events appear only on the node where they occur, while Swarm-scoped events appear on managers. The daemon returns only the last 256 stored events, so the command cannot serve as a long-term audit log.

```shell
docker events \
  --filter type=service \
  --filter type=task \
  --since 30m
```

An external monitoring system should collect host metrics, daemon health, node availability, task state, restart rates, resource saturation, network errors, storage capacity, certificate status, and application indicators. Alerts should identify an actionable condition rather than echo every task transition. A replaced task may show normal reconciliation, while a rising replacement rate may reveal a systemic fault.

Dashboards should expose desired versus running replicas, pending tasks, manager quorum, update progress, and capacity by failure domain. An educational visualiser can display placement, but production observability needs authentication, retention, alerting, and no direct exposure of the Docker socket.
## One-off and distributed jobs
Jobs run tasks that complete and exit successfully instead of remaining active. A replicated job executes a total number of iterations. A global job executes once on each eligible node and also runs on a qualifying node that joins later.

```shell
docker service create \
  --name thumbnail-backfill \
  --mode replicated-job \
  --replicas 100 \
  --max-concurrent 5 \
  registry.example.net/tools/thumbs:1.6.0
```

The example creates 100 successful iterations and allows at most five to run concurrently. The service CLI exposes `--max-concurrent`, but the Compose deploy specification does not. A stack file can still define `replicated-job` and `global-job` modes. Current support supersedes guidance that excludes jobs from stack declarations.

A task that exits with code zero reaches `COMPLETE` and does not restart. A failure can restart according to the job's restart policy. Completed tasks remain recorded until the job is explicitly removed, which preserves status but can increase task history.

Updating a job stops tasks still in progress, creates a new set of tasks, and resets completion status. Jobs do not use rolling update or rollback policies. A force update runs the job again with the same main parameters.

Job containers should remain idempotent because retries, operator actions, or partial failures can repeat work. Each unit should claim work safely, record completion outside the container, tolerate duplicate execution, and expose clear exit codes. Results must live in an external data store or durable volume because the completed container is not a reliable result archive.

A global job can perform node-local maintenance, inspection, or cache preparation. Its placement constraints should exclude managers or specialised hosts when the command does not belong there. A destructive maintenance job needs especially strict review because one declaration can execute across the cluster.

Distributed benchmarks require careful interpretation. More replicas do not guarantee proportional throughput. Shared databases, locks, network links, registry limits, client generators, warm-up periods, connection reuse, and CPU contention can dominate results. A useful benchmark fixes the workload, records uncertainty, monitors every shared component, and repeats trials under representative conditions.
## Configs and secrets
### Swarm configs
A config stores non-sensitive operational content in Swarm and mounts it into authorised service tasks as a file. Configs suit proxy files, feature settings, templates, and other data that should travel with the service declaration.

```shell
docker config create web-nginx.conf ./nginx.conf
docker service create \
  --name web \
  --config source=web-nginx.conf,target=/etc/nginx/nginx.conf \
  nginx:1.27.5
```

Swarm stores configs in the encrypted Raft log, but a config is not a secret. Administrators and processes with suitable access can inspect it. Passwords, private keys, tokens, and similar credentials belong in secrets or an external secret system.

Configs are immutable. Updating a local file does not change the existing Swarm object. Rotation creates a new config, updates the service to use it, verifies the rollout, and removes the old config after no service references it. Versioned names such as `web-nginx-2026-08-01` make that sequence explicit.
### Swarm secrets
A Swarm secret stores sensitive data and grants it only to selected services. Swarm encrypts secrets in transit and stores them in the encrypted Raft log. On a Linux task, Docker mounts an authorised secret into an in-memory filesystem under `/run/secrets` by default. The secret is not inserted into the image or a normal environment variable.

```shell
printf 'value supplied through a protected process' | docker secret create db-password -
docker service create \
  --name database \
  --secret db-password \
  registry.example.net/database:8.4
```

Real secret creation should avoid shell history, terminal recording, process arguments, build logs, and casual standard output. A secure automation path can send data from an authorised secret source directly to Docker without writing an unprotected temporary file.

On Windows, Docker cannot mount secrets through a RAM disk. A running Windows container receives secrets in clear text on its root disk, and Docker removes them when the container stops. Docker recommends BitLocker on the volume containing the Docker root directory to protect those files at rest.

Secret access follows service authorisation. A task receives only the secrets attached to its service. Removing a secret grant and updating the service replaces tasks so the old mount disappears. A service process still needs least privilege because any process that can read the mounted path can disclose its contents.

Secrets are immutable. Rotation creates a new version, grants it to the service, updates the application to use it, confirms success, removes the old grant, and later deletes the unused secret.

```shell
docker secret create db-password-v2 ./protected-input
docker service update \
  --secret-add source=db-password-v2,target=db-password \
  --secret-rm db-password-v1 \
  database
docker secret rm db-password-v1
```

The target name can keep the path stable while the source object changes. Rotation must respect the dependency's acceptance window. A database may need to accept both credentials briefly, while a certificate rollout may need a trust bundle that recognises both issuers.

Some images support environment variables ending in `_FILE`, such as `MYSQL_ROOT_PASSWORD_FILE`, and read the value from the referenced secret file. This convention belongs to each image, not to Docker as a universal feature. The image documentation must confirm support and exact variable names.

An external secret manager may provide stronger central auditing, dynamic credentials, policy integration, or automated rotation. Swarm secrets still offer useful distribution within the cluster. The choice should follow threat modelling, compliance needs, recovery design, and operational capability.
## Failure handling and maintenance
### Node and zone loss
The response to node loss starts with the failure domain. One failed worker should trigger replacement tasks on eligible nodes. Several simultaneous failures may expose a shared switch, zone, storage array, or capacity assumption. Managers should remain reachable across the surviving infrastructure, and replacement nodes should have access to the same registries, networks, secrets, and durable data.

Swarm cannot distinguish a permanently failed node from a temporary network partition immediately. Scheduling replacements preserves desired capacity, but a partitioned application task may continue to affect an external system until the host or network isolates it. Stateful services need fencing, leases, leader election, or database controls that prevent two writers from acting independently.

After a worker returns, the scheduler can use it for new tasks, but services do not automatically return to their earlier distribution. A force update can rebalance a service, although it replaces healthy tasks and should follow capacity and disruption checks. Placement preferences guide later scheduling but do not continuously rebalance every existing task.
### Planned maintenance
Host maintenance should proceed one node at a time unless the cluster has verified capacity for a wider change. The operator drains the node, watches replacement tasks reach a healthy state, performs the host work, validates Docker and networking, and returns the node to active availability. Manager maintenance also checks quorum before and after every restart.

Draining a node does not guarantee graceful application behaviour. The service's stop grace period, update order, readiness, and load-balancer response determine whether clients see interruption. A maintenance runbook should include application checks, not only a `Ready` node state.

Engine upgrades should use versions supported by the organisation and tested with stack files, network drivers, logging drivers, volume plugins, and security tooling. Managers should upgrade in a sequence that retains quorum. Workers can normally change in broader groups only when service capacity and failure-domain distribution remain adequate.
### Incident containment
A compromised worker may expose the tasks, mounted secrets, local volumes, network access, and daemon credentials present on that host. Containment can remove the node from load balancers, isolate its network, drain it when management remains trustworthy, and remove it from the swarm. The response then rotates affected application credentials and verifies images, configs, and external systems.

A compromised manager has wider impact because managers hold cluster state and can change services, distribute secrets, add nodes, and control workloads. Response planning should assume cluster-wide administrative exposure. Depending on evidence and policy, recovery may require new manager infrastructure, rotated join tokens and credentials, a new certificate authority, rebuilt workloads, and restoration of validated declarations and data.

Incident actions should preserve evidence where legal and operational requirements permit. Rebuilding too early can erase daemon logs, task records, filesystem artefacts, and network state. Evidence preservation must remain compatible with containment, especially when an active host can continue harming external systems.
### Capacity exhaustion
Swarm reports desired state even when it cannot achieve it. Pending tasks, node disk pressure, image-pull failures, memory termination, and saturated overlay links need alerts before redundancy disappears. Capacity planning should reserve headroom for a node or zone failure, rolling `start-first` updates, and short demand spikes.

Disk planning includes image layers, container writable layers, logs, volumes, Raft data, and temporary pull space. Automatic image cleanup needs safeguards so it cannot remove required data or compete with deployments. Log rotation and central forwarding reduce one common source of exhaustion.

Load testing should include degraded modes. A cluster that handles normal traffic across six workers may fail its service objective when one worker drains and two replacement tasks start while images pull. Testing that sequence reveals cold-start time, registry demand, scheduler constraints, and downstream limits before a real outage.
## Production operating model
A production swarm needs deliberate ownership across infrastructure, application delivery, and incident response. The following baseline condenses the most important controls:
- Run three or five managers across independent failure domains, and keep enough healthy managers for quorum.
- Drain dedicated managers, unless a consciously designed small cluster needs them to run application tasks.
- Restrict Swarm ports to trusted networks, protect Docker daemon access, rotate exposed join tokens, and manage autolock keys securely.
- Patch Docker Engine and host operating systems through a staged process that preserves quorum and service capacity.
- Build and test images before deployment, identify them with controlled tags or digests, scan them, and protect registry credentials.
- Declare resource reservations, limits, placement rules, health checks, restart policies, update controls, and rollback behaviour.
- Use separate overlay networks for required communication paths, encrypt overlay data when risk requires it, and test the performance cost.
- Centralise logs and metrics, bound local log growth, alert on quorum and reconciliation failures, and retain audit evidence outside the daemon event buffer.
- Place durable data on storage that survives task rescheduling, and test application backups independently of Swarm backups.
- Rotate configs and secrets through versioned objects, avoid credential exposure in logs or images, and remove obsolete grants.
- Back up manager state consistently, protect the archive and unlock material, and rehearse loss-of-quorum recovery.
- Test node loss, manager loss, registry failure, network partitions, failed updates, capacity exhaustion, and restoration before relying on the design.

Swarm's main strength is its compact desired-state model. Services define ongoing workloads, jobs define finite work, stacks group application resources, overlay networks connect tasks, and managers reconcile declarations through Raft consensus. That model produces resilience only when declarations reflect real capacity, storage, health, security, and recovery requirements.

An effective deployment treats containers as replaceable while preserving state in durable systems. It treats manager quorum as a hard dependency, not a background detail. It also treats operational feedback as part of deployment, so an update completes only after both Swarm and the application demonstrate health.