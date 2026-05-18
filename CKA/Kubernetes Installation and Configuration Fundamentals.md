# Kubernetes Installation and Configuration Fundamentals
## Scope and prerequisites
Kubernetes runs containerised applications across a cluster of machines. It schedules workloads, exposes them through stable network services, keeps them close to a declared state and hides much of the infrastructure detail from application teams.

A practitioner needs a working knowledge of Linux administration, TCP/IP networking, containers and the command line before building or operating a Kubernetes cluster. Production use also requires planning for capacity, security, high availability, backup, recovery and version upgrades.
## Kubernetes as a container orchestrator
Kubernetes acts as a container orchestrator. It starts, stops, places and replaces container-based workloads according to configuration stored in its API. Administrators and developers describe the desired state, then Kubernetes controllers work to make the cluster match that state.

The desired-state model gives Kubernetes three main strengths:
- It speeds deployment by moving container images from development into running workloads with repeatable configuration.
- It improves recovery by replacing failed Pods or rescheduling workloads when nodes fail.
- It abstracts infrastructure so application teams do not have to manage every server, network rule, storage attachment or load balancer directly.

Kubernetes configuration normally uses a declarative model. A manifest describes what should exist, such as a Deployment with three replicas, a Service on port 80 or a PersistentVolumeClaim for storage. Kubernetes accepts that configuration through the API server, stores object state in etcd and uses controllers to reconcile actual state with desired state.

Imperative commands still have a place. They help with exploration, troubleshooting and quick changes at the command line. Declarative configuration remains the stronger operating model because it can be reviewed, versioned, reused and applied consistently across environments.
## The Kubernetes API
The Kubernetes API defines the objects that make up the cluster and its workloads. These objects include Pods, Nodes, Deployments, ReplicaSets, Services, PersistentVolumes and PersistentVolumeClaims. Each object has a schema, metadata and a specification that describes its intended configuration.

The API server exposes the Kubernetes HTTP API, usually over HTTPS. Clients such as kubectl, controllers and node components communicate with the cluster through the API server. Common HTTP operations include GET, POST, PUT, PATCH and DELETE. The API server validates requests, applies admission controls where configured, then persists object state in etcd.

All control activity passes through the API server. This design gives Kubernetes a central, consistent control point for authentication, authorisation, validation, audit logging and state changes.
## Pods
A Pod is the smallest deployable unit in Kubernetes. It represents one or more containers that run together on the same node and share a network namespace, storage volumes and lifecycle. Most Pods contain one application container, but multi-container Pods support patterns such as sidecars, proxies and helper processes.

Pods are scheduled as units. The scheduler chooses a node that can satisfy the Pod's resource requests and placement constraints. The kubelet on that node then works with the container runtime to pull images and run the containers.

Pods are also ephemeral. Kubernetes does not repair a failed Pod in place. A controller usually creates a replacement Pod when the existing Pod fails, disappears or becomes unhealthy. The new Pod receives its own identity and IP address.

Kubernetes tracks both Pod state and Pod health. State describes whether the Pod and its containers are pending, running, succeeded, failed or unknown. Health checks use probes. A liveness probe can trigger a restart when an application stops responding. A readiness probe can remove a Pod from service endpoints until it can safely receive traffic. A startup probe can give slow-starting applications time to initialise before other health checks apply.
## Controllers and workload resources
Controllers run reconciliation loops. They watch the API for changes, compare actual state with desired state and request changes when the two differ.

ReplicaSets maintain a chosen number of matching Pod replicas. If the desired state requires three replicas and one Pod fails, the ReplicaSet controller creates another Pod.

Deployments manage ReplicaSets and make application rollout safer. A Deployment can create a new ReplicaSet for a new container image, shift traffic from the old version to the new version, pause or resume a rollout and roll back when necessary. Administrators commonly create Deployments rather than raw ReplicaSets because Deployments manage version transitions as well as replica count.

Kubernetes includes many other controllers. Some manage workloads, such as StatefulSets, DaemonSets and Jobs. Others manage infrastructure state, such as Nodes, Endpoints and Service-related resources. The same control-loop principle applies across them.
## Services
Services provide stable network access to a changing set of Pods. Pods can be created, replaced and deleted, so their IP addresses cannot act as durable application endpoints. A Service gives clients a stable virtual IP address and DNS name while Kubernetes updates the backend endpoints as matching Pods change.

A Service selects Pods using labels. When a Pod matches the selector and passes readiness checks, Kubernetes adds it to the Service endpoints. When a Pod fails, loses readiness or disappears, Kubernetes removes it from those endpoints.

Services also support load distribution across multiple Pod replicas. kube-proxy or an equivalent network component programs node-level rules so traffic sent to the Service reaches the backing Pods. Common Service types include ClusterIP for internal access, NodePort for node-level exposure and LoadBalancer for integration with cloud or external load balancers.

This abstraction supports self-healing behaviour. Users continue to access the Service endpoint while controllers replace failed Pods and the Service endpoint list changes behind it.
## Storage
Containers and Pods are temporary, but many applications need durable data. Kubernetes supports storage through volumes, PersistentVolumes and PersistentVolumeClaims.

A volume can give containers in a Pod shared access to files during the Pod's life. Some volume types disappear with the Pod, while others connect to external storage.

A PersistentVolume is cluster-level storage that exists independently of a Pod. A PersistentVolumeClaim requests storage with particular size and access characteristics. A Pod then references the claim. This separates workload definitions from the details of physical or cloud storage and lets administrators manage storage separately from application manifests.

Production clusters should also define storage classes, backup processes, recovery testing and access controls for data-bearing workloads.
## Cluster architecture
A Kubernetes cluster has a control plane and worker nodes. The control plane manages cluster state. Worker nodes run application workloads. A node can be a virtual machine or a physical machine.

The term control plane node replaces the older term master node. Older commands, hostnames or documentation may still use master, but control plane is the current term.

The main control plane components are:
- kube-apiserver, which exposes the Kubernetes API and handles cluster requests.
- etcd, which stores cluster state as key-value data.
- kube-scheduler, which assigns newly created Pods to suitable nodes.
- kube-controller-manager, which runs core controller loops.
- cloud-controller-manager, where configured, which integrates the cluster with cloud provider APIs.

The API server is stateless and can scale horizontally. etcd holds the authoritative state, so it needs careful backup, protection and recovery planning. The scheduler evaluates resource requests, available node capacity, taints, tolerations, affinity rules, anti-affinity rules and other scheduling constraints. The controller manager runs reconciliation loops that keep objects moving towards their desired state.
## Node components
Every node that runs Pods needs node-level services. In kubeadm-built clusters, control plane components commonly run as static Pods on control plane nodes, so those nodes also run node components.

The key node components are:
- kubelet, which manages Pods and containers on the node.
- kube-proxy or an alternative Service implementation, which supports Service networking.
- a container runtime, such as containerd or CRI-O, which runs containers through the Container Runtime Interface.

The kubelet watches the API server for Pods scheduled to its node. It pulls configuration, asks the runtime to start or stop containers, runs configured probes and reports node and Pod status back to the API server.

kube-proxy watches Service and endpoint data, then updates local network rules to route Service traffic to backing Pods. Linux clusters may use iptables, IPVS or nftables modes, depending on version and configuration. Some networking solutions replace kube-proxy with their own dataplane.

Modern Kubernetes requires a runtime that implements the Container Runtime Interface. Kubernetes removed the in-tree dockershim in version 1.24. Docker-built images still run in Kubernetes because images follow Open Container Initiative standards, but Docker Engine itself is not a CRI runtime unless an external CRI adapter such as cri-dockerd is used.
## Cluster add-ons
Cluster add-ons provide services that the cluster needs but does not implement directly inside the core binaries. The most common add-on is DNS. CoreDNS gives Pods and Services discoverable names inside the cluster.

Other add-ons may include:
- an ingress controller for HTTP and HTTPS routing into Services.
- a dashboard or other administrative interface.
- metrics collection for autoscaling and observability.
- Container Network Interface plugins for Pod networking.
- storage drivers and snapshot controllers.

A minimal cluster needs networking and DNS to run ordinary workloads effectively. Other add-ons depend on operational requirements.
## Pod and Service operation
A typical deployment sequence starts when kubectl sends a manifest to the API server. The API server validates the request and stores the new object in etcd. A Deployment controller observes the Deployment and creates a ReplicaSet. The ReplicaSet controller creates the required Pod objects. The scheduler assigns those Pods to nodes. Each node's kubelet sees the scheduled work, asks the runtime to pull the image and starts the containers.

If a node fails, the control plane eventually marks it unhealthy. Controllers detect that the desired number of available replicas no longer exists. They create replacement Pods, and the scheduler places those Pods on suitable nodes. A taint normally prevents ordinary user workloads from being scheduled on control plane nodes. That taint can be removed in small test clusters, but production clusters generally keep workloads away from the control plane.

When a Deployment backs a Service, Kubernetes updates Service endpoints as Pods appear, become ready, fail or disappear. Clients keep using the stable Service address while Kubernetes adjusts the backend set.
## Kubernetes networking
Kubernetes uses a flat Pod networking model. Each Pod gets its own IP address. Pods should communicate with other Pods without network address translation inside the cluster. Node agents must also communicate with the Pods running on their node.

The main networking cases are:
- Containers inside the same Pod communicate over localhost.
- Pods on the same node communicate through the node's Pod network bridge or dataplane.
- Pods on different nodes communicate through routed or overlaid Pod networks.
- External clients reach workloads through Services, ingress controllers, gateways or load balancers.

Direct routing can satisfy the Kubernetes network model when the underlying infrastructure supports Pod CIDR reachability. Overlay networking can provide that reachability when administrators do not control the physical network or cloud fabric. Overlay networks use tunnelling or encapsulation to carry Pod traffic between nodes.

Common CNI options include Calico, Cilium, Flannel and cloud-provider networking plugins. Administrators must choose Pod and Service address ranges that do not overlap with existing networks. They should also decide how to enforce network policy, expose ingress traffic and integrate with DNS, firewalls and load balancers.
## Installation choices
Kubernetes can run on laptops, virtual machines, bare metal or managed cloud services. The right choice depends on purpose, skill, governance, cost and operational maturity.

Desktop Kubernetes suits learning, testing and local development. Options such as Docker Desktop, minikube and kind provide fast local clusters, but they do not represent production operations fully.

Self-managed clusters can run on on-premises servers, virtual machines or cloud infrastructure-as-a-service instances. This model gives administrators strong control over versions, networking, storage, security and topology. It also makes them responsible for upgrades, backups, monitoring, control plane availability and incident response.

Managed Kubernetes services shift much of the control plane operation to the provider. Examples include Amazon Elastic Kubernetes Service, Google Kubernetes Engine and Azure Kubernetes Service. Managed services reduce infrastructure burden and integrate with cloud identity, networking and load balancing. They also limit some version choices and implementation details to the provider's supported options.
## Installation planning
Installation planning should address networking, scale, availability, backup and security before any node joins a cluster.

Network planning must cover Pod CIDRs, Service CIDRs, node IPs, DNS, routing, firewall rules and any overlay or CNI requirements. Address ranges must not overlap with corporate, cloud or lab networks.

Capacity planning must cover the number of nodes, CPU, memory, storage, image pull capacity and failure tolerance. A cluster that can run the workload during normal operation may still fail service objectives when one node becomes unavailable.

Availability planning must cover the control plane. A single control plane node can support a lab, but it creates a single point of failure. Production clusters should use multiple API server instances and a resilient etcd topology. etcd requires regular backup and tested restore procedures.

Security planning must cover authentication, authorisation, certificates, admission controls, secrets management, network policy, image provenance and least-privilege access.
## Base requirements for kubeadm clusters
A kubeadm cluster usually starts with Linux nodes that meet the version and resource requirements of the intended Kubernetes release. A lab can run with modest resources, such as two CPUs and 2 GB of RAM per node, but production sizing must follow the workload and add headroom for failure, upgrades and system components.

Each node needs:
- a supported Linux distribution and kernel configuration.
- a unique hostname, MAC address and product UUID.
- network connectivity between all nodes.
- required kernel modules and sysctl settings for bridged traffic.
- a CRI-compatible container runtime, commonly containerd or CRI-O.
- kubelet, kubeadm and kubectl installed at compatible versions.
- swap disabled or explicitly supported through kubelet configuration for the chosen release and operating model.

Kubernetes packages should be pinned or held after installation. Upgrades should follow the Kubernetes upgrade process rather than unattended operating system package updates.
## Important ports
Firewalls and security groups must allow required component traffic. The most important default ports are:

| Component | Default port or range | Typical use |
| --- | --- | --- |
| kube-apiserver | TCP 6443 | Cluster API access |
| etcd client API | TCP 2379 | API server to etcd |
| etcd peer traffic | TCP 2380 | etcd member communication |
| kubelet API | TCP 10250 | Control plane to kubelet |
| kube-scheduler | TCP 10259 | Scheduler health and metrics, usually local or restricted |
| kube-controller-manager | TCP 10257 | Controller manager health and metrics, usually local or restricted |
| NodePort Services | TCP and UDP 30000-32767 | External access through node ports |

Older material may mention scheduler port 10251 and controller manager port 10252. Current secure defaults use 10259 and 10257. Actual exposure should follow the Kubernetes release, the distribution and local hardening settings.
## Building a kubeadm cluster
A kubeadm installation normally follows a clear sequence:
1. Prepare each node's operating system.
2. Install and configure the container runtime.
3. Install kubelet, kubeadm and kubectl.
4. Run kubeadm init on the first control plane node.
5. Configure kubectl with the generated admin kubeconfig.
6. Install a CNI plugin so Pods can communicate.
7. Join worker nodes with kubeadm join.
8. Confirm node, Pod and system component status.

containerd commonly needs a generated config.toml file and a cgroup driver setting that matches kubelet. On many systemd-based distributions, containerd should use SystemdCgroup = true.

kubeadm init performs several phases. It runs pre-flight checks, creates certificates, writes kubeconfig files, generates static Pod manifests for control plane components, starts the control plane, taints the control plane node, creates join tokens and applies essential add-ons such as CoreDNS and kube-proxy.

The certificate authority created by kubeadm secures cluster communication and issues certificates for components. Organisations can integrate kubeadm with an external public key infrastructure when policy requires it.

kubeconfig files tell clients and components how to reach the API server and authenticate. The admin.conf file gives cluster administrator access. kubelet.conf, scheduler.conf and controller-manager.conf support component authentication. These files live under /etc/kubernetes on the relevant nodes.

Static Pod manifests live under /etc/kubernetes/manifests on control plane nodes. The kubelet watches this directory and starts the listed Pods directly. This allows the control plane to start after a reboot even before the API server can schedule ordinary Pods.

After kubeadm init, administrators usually copy admin.conf into the user's .kube/config path, then apply the CNI manifest. The control plane becomes ready only when networking and DNS components start successfully.

Worker nodes join with kubeadm join. The join command includes the API server address, a bootstrap token and the CA certificate hash. During the join process, the node obtains cluster information, submits a certificate signing request, receives kubelet credentials and begins reporting to the control plane. The node may show NotReady until the CNI plugin finishes deploying on that node.
## Managed cloud clusters
Managed services create Kubernetes clusters through provider portals, command-line tools or infrastructure-as-code workflows. The basic pattern stays consistent across providers:
- Authenticate to the cloud account.
- Create or select a resource group, project or account boundary.
- Choose a region, Kubernetes version, node pool and networking mode.
- Create the cluster.
- Download or merge kubeconfig credentials.
- Use kubectl against the selected context.

In managed services, the provider often hides control plane nodes and control plane Pods. kubectl get nodes usually shows only worker nodes or node pool machines. System namespaces still contain provider-managed add-ons such as DNS, networking, metrics and agents.

Cluster contexts matter when operating both local and cloud clusters. kubectl config get-contexts lists configured clusters, and kubectl config use-context selects the target for future kubectl commands.
## Working with kubectl
kubectl is the main command-line tool for interacting with Kubernetes. It sends requests to the API server to create, read, update and delete resources.

Common operations include:
- kubectl get, which lists resources.
- kubectl describe, which shows detailed resource state and events.
- kubectl create, which creates resources imperatively.
- kubectl apply, which applies declarative manifests.
- kubectl delete, which deletes resources.
- kubectl explain, which displays API documentation for resource fields.
- kubectl logs, which reads container logs from stdout and stderr.
- kubectl exec, which runs a command inside a container.
- kubectl scale, which changes replica counts.
- kubectl edit, which opens a live object for direct editing.

The general syntax is kubectl, followed by an operation, a resource type, an optional resource name and flags. For example, kubectl get pods lists Pods in the current namespace. kubectl get pods -o wide adds node and IP details. kubectl get pod example -o yaml prints a Pod as YAML.

kubectl can also help users discover the API. kubectl api-resources lists resource types and short names. kubectl explain deployment.spec.template describes fields in a manifest. Built-in help and shell completion make command-line work faster and reduce syntax errors.
## Application deployment with manifests
Kubernetes manifests commonly use YAML. A manifest usually contains apiVersion, kind, metadata and spec.

A Deployment manifest states the API version, identifies the object as a Deployment, names it and defines the desired workload. The spec sets the replica count, selectors and Pod template. The Pod template defines labels and container configuration, including image, name, ports and other runtime settings.

The usual flow is:
1. Write or generate a manifest.
2. Review the manifest.
3. Apply it with kubectl apply -f filename.yaml.
4. Inspect the resulting Deployment, ReplicaSet, Pods and events.
5. Expose the workload through a Service when stable access is required.

kubectl can generate starter YAML without creating an object. For example, kubectl create deployment hello-world --image=image-name --dry-run=client -o yaml prints a Deployment manifest. Redirecting that output to a file creates a useful starting point for declarative configuration.

Applying a Deployment creates a Deployment object. The Deployment creates a ReplicaSet. The ReplicaSet creates Pods. The scheduler assigns the Pods to nodes. The kubelet starts containers through the runtime. A Service can then select the Pods and distribute requests among ready replicas.

Imperative deployment remains useful for tests. kubectl create deployment can quickly create a Deployment, and kubectl run can start a bare Pod. Bare Pods are not normally suitable for durable application operation because no controller replaces them after failure.

Declarative configuration scales better. The same manifest can create, update and document a workload. Changing replicas from 1 to 20 in a Deployment manifest does not affect the cluster until the changed file is applied. After application, Kubernetes creates or removes Pods until the actual replica count matches the manifest.

Services can also be created declaratively. kubectl expose deployment can generate Service YAML with --dry-run=client -o yaml. Once applied, the Service tracks matching Pods and updates endpoints as the Deployment scales.
## Lab topology and package preparation
A practical kubeadm lab can use one control plane node and several worker nodes. Each node should resolve the others by DNS or by hosts-file entries. The control plane node runs the API server endpoint, and worker nodes contact that endpoint over TCP 6443 during normal operation and when joining the cluster.

A repeatable node build starts with operating system preparation. Administrators disable or configure swap according to kubelet requirements, load required kernel modules, set sysctl values for bridged IPv4 traffic and confirm that each node has a unique hostname and network identity. They then install the container runtime and configure its cgroup driver to match kubelet. On systemd-based Linux distributions, matching the runtime and kubelet to the systemd cgroup driver avoids resource-management conflicts.

Package installation should use repositories that match the intended Kubernetes minor version. kubelet, kubeadm and kubectl should normally stay on compatible versions, with kubeadm controlling cluster bootstrap and upgrades. Package holds prevent unattended upgrades from moving one component ahead of the planned upgrade sequence.

The kubelet may appear inactive after package installation and before cluster bootstrap. That state can be normal because the kubelet has no Pod manifests or assigned work yet. After kubeadm writes static Pod manifests and the node joins a cluster, the kubelet starts managing Pods and reports node status to the API server.
## kubeadm bootstrap details
kubeadm init builds the first control plane node. It checks privileges, CPU, memory, networking, ports and the container runtime. It then creates a certificate authority unless administrators supply external certificates. It uses that authority to issue certificates for the API server, etcd, front-proxy components and cluster clients.

The bootstrap process writes kubeconfig files for administrators and control plane components. These files contain the API server location, certificate authority data and client credentials. admin.conf gives broad administrative access and must be protected like a privileged credential. Component kubeconfig files let the scheduler, controller manager and kubelet authenticate to the API server.

kubeadm also writes static Pod manifests for etcd, kube-apiserver, kube-controller-manager and kube-scheduler. The kubelet reads these files and starts the control plane Pods directly. This solves a bootstrap problem: the API server cannot schedule the components that must exist before the API server can operate.

After the control plane starts, kubeadm applies default cluster settings, creates a bootstrap token and prints a join command. The join command contains the API server endpoint, a token and a CA certificate hash. The token authorises a new node to begin bootstrapping. The CA hash helps the joining node verify that it connects to the intended cluster.

The control plane is not fully useful until a CNI plugin creates Pod networking. Applying a CNI manifest installs the networking components, allocates Pod IP addresses and enables cross-node Pod communication. DNS components such as CoreDNS usually become ready after the network layer works.
## Joining worker nodes
A worker node must have the same base preparation as the control plane node. It needs the runtime, kubelet, kubeadm, kubectl, kernel settings and network reachability to the API server. Administrators run kubeadm join on the worker node, not on the control plane.

During join, the node retrieves cluster discovery information and starts TLS bootstrapping. The kubelet submits a certificate signing request to the API server. After approval through the kubeadm-managed bootstrap flow, the node stores its client certificate and kubelet kubeconfig, then begins reporting status.

A newly joined node may show NotReady while the CNI plugin starts its node-level components and configures Pod networking. Once those components run, the node normally moves to Ready and can accept scheduled Pods. kubectl get nodes and kubectl get pods --all-namespaces help confirm that the node, network pods, kube-proxy and DNS components are healthy.

Repeated node joins should use fresh or valid bootstrap tokens. kubeadm token create --print-join-command can generate a current join command when the original output is unavailable or expired.
## Namespaces, labels and selectors
Namespaces divide cluster resources into logical scopes. They help separate system components, environments, teams or applications. The default namespace often holds simple test workloads, while kube-system holds cluster services such as CoreDNS, kube-proxy and networking components. Namespaces are not hard security boundaries on their own, but they work with role-based access control, quotas and network policy to support multi-team administration.

Labels attach key-value metadata to objects. Selectors use those labels to find related objects. Deployments use selectors to identify the Pods they manage. Services use selectors to identify Pods that should receive traffic. This loose coupling lets Kubernetes replace Pods without changing the Service or Deployment identity.

Good label design matters. Labels should identify application name, component, version, environment and ownership where useful. Consistent labels make querying, service selection, monitoring and policy easier.
## Reading cluster state
kubectl get gives a quick view of resource state. kubectl get nodes shows readiness, roles, age and version. The wide output adds useful details such as internal IP address, operating system image, kernel version and container runtime version. kubectl get pods shows readiness, status, restart count and age. Wide Pod output adds Pod IP and node placement.

kubectl describe explains why an object looks the way it does. A node description shows addresses, capacity, allocatable resources, conditions, taints, system information and running Pods. A Pod description shows container state, image, node, IP address, volumes, events and probe information. Events often reveal scheduling failures, image pull errors, readiness problems and permission issues.

kubectl logs supports application troubleshooting by reading container output. For multi-container Pods, the command needs a container name. kubectl exec supports direct inspection by running a command inside a container, although production environments should control this access carefully.

kubectl explain helps build correct manifests. It exposes API field documentation from the cluster itself, so it matches the installed API version. This is especially useful when writing nested fields such as deployment.spec.template.spec.containers.
## Imperative and declarative deployment patterns
Imperative commands create resources directly from command-line arguments. kubectl create deployment hello-world --image=example creates a Deployment quickly. kubectl run can create a single unmanaged Pod. kubectl expose deployment can create a Service from an existing Deployment. These commands help with learning and short-lived experiments.

Declarative deployment stores configuration in files. kubectl apply sends those files to the API server. When the file changes and is applied again, Kubernetes updates the stored desired state. This pattern supports review, automation and rollback through version control.

Dry-run generation bridges the two approaches. A command such as kubectl create deployment hello-world --image=example --dry-run=client -o yaml prints a manifest without creating the object. Administrators can redirect the output to a file, edit it and then apply it. The same technique can generate starter Service manifests.

A simple Deployment manifest demonstrates how Kubernetes composes objects. apiVersion selects the API group and version. kind identifies the object type. metadata names and labels the object. spec defines replicas, selectors and a Pod template. The template has its own metadata and spec because it describes Pods that the Deployment will create. The container list defines names, images, ports, environment variables, probes and resource settings.
## Scaling, rollout and cleanup
Scaling changes desired replica count. Administrators can edit a manifest and apply it, run kubectl scale deployment name --replicas count, or edit the live object with kubectl edit. The declarative file remains the safest long-term source of truth. If kubectl edit or kubectl scale changes a live object but the stored manifest still says something different, the next apply may reverse the manual change.

When a Deployment scales up, the ReplicaSet creates more Pods. The scheduler spreads them according to resource availability and configured constraints. The Service endpoint set grows as new Pods become ready. When a Deployment scales down, Kubernetes removes excess Pods and updates the Service endpoints.

Deleting a Deployment cascades to its ReplicaSet and managed Pods. Deleting a bare Pod removes only that Pod because no higher-level controller owns it. Deleting a Service removes the stable endpoint, but it does not delete the Pods or Deployment behind it. Cleanup commands should target the correct owner object to avoid orphaned or unexpectedly running workloads.

Rollouts need attention beyond replica count. Production Deployments should define readiness probes, resource requests and limits, image tags or digests, update strategies, Pod disruption budgets where needed and observability hooks. These settings help Kubernetes make safer decisions during upgrades, failures and maintenance.
## Managed service operating differences
Managed Kubernetes services simplify control plane operation, but they do not remove operational responsibility. Application teams still manage workload manifests, namespaces, access policies, resource requests, secrets, upgrades within supported windows, node pools and cost. Provider-managed control planes reduce the burden of running etcd and API server replicas, yet cluster users still need to understand how the API behaves.

Cloud integrations affect Services and storage. A LoadBalancer Service can create a provider load balancer. A PersistentVolumeClaim can trigger dynamic volume provisioning through a storage class. Node pools can attach to autoscaling groups or virtual machine scale sets. These integrations make platform features easier to consume, but they also require clear cost, quota and security controls.

Context management becomes critical when operators work with multiple clusters. A command intended for a lab cluster can damage a production cluster if the current context is wrong. Operators should check kubectl config current-context before risky operations and use naming conventions that make contexts unambiguous.
## Reliability and security foundations
Reliable clusters require more than successful installation. etcd needs backups, restore procedures and monitoring. Control plane endpoints need redundancy. Worker nodes need patching, capacity monitoring and safe drain procedures. Cluster add-ons need lifecycle management just like application workloads.

Security begins at the API server. Strong authentication, role-based access control, least-privilege service accounts and audit logging help limit damage from mistakes or compromised credentials. kubeconfig files, bootstrap tokens and certificates should be stored securely and rotated according to policy.

Network policy can restrict Pod-to-Pod communication when the CNI plugin supports it. Without policy, many clusters allow broad east-west traffic by default. Image admission policies, signed images and vulnerability scanning reduce supply-chain risk. Secrets should use appropriate encryption and external secret management where organisational policy requires stronger controls.

Resource requests and limits protect cluster stability. Requests guide scheduling by telling Kubernetes how much CPU and memory a Pod needs. Limits constrain usage where appropriate. Quotas and limit ranges can stop one namespace or team from exhausting shared capacity.
## Operational discipline
Kubernetes rewards clear operating discipline. Administrators should prefer declarative manifests, version control, repeatable cluster builds, tested upgrades and monitored health. They should treat etcd as critical data, restrict API access, keep certificates and kubeconfig files secure and understand which components their platform provider manages.

The essential model remains simple. The API stores desired state. Controllers reconcile that state. The scheduler places Pods. Kubelets run them. Services provide stable access. Networking and storage plugins connect workloads to the infrastructure they need.