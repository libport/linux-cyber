# Certified Kubernetes Administrator: Kubernetes Foundations
## Kubernetes foundations
Kubernetes orchestrates containerised applications. It schedules pods, starts containers, replaces failed workloads and hides much of the infrastructure work involved in running distributed applications. Administrators and developers describe the required state of an application, then Kubernetes works to make the cluster match that state.

Kubernetes improves deployment speed because teams can move container images from development into production through a consistent control model. It also improves recovery. When a pod fails or a node becomes unavailable, controllers can create replacement pods and return the workload to its declared state. Kubernetes also abstracts storage, networking and placement decisions, so application code can rely on cluster services rather than fixed servers.
## Operating model
Kubernetes uses declarative configuration. A deployment defines the containers, replicas, storage and network exposure an application requires. This model gives the cluster a clear target instead of a one-off script of manual steps. Controllers then watch the current state and reconcile it with the desired state. When the current state differs, controllers create, update or delete resources through the Kubernetes API. Imperative commands can still change state, but declarative objects provide a stronger record of what the system should run.

The API server provides the central access point for cluster activity. Administrators, command-line tools and cluster components use it to read and change API objects. The API server validates requests and records object state in etcd, the cluster data store.

Core API objects provide the building blocks for applications:
- Pods group one or more containers into the smallest schedulable Kubernetes unit. Containers in the same pod share a network namespace and can communicate over localhost.
- Controllers keep workloads in the desired state. A ReplicaSet maintains a declared number of matching pods. A Deployment manages ReplicaSets and supports controlled application updates and rollbacks.
- Services provide a stable endpoint for pods that may be replaced, rescheduled or scaled. A Service can expose an IP address or DNS name and route traffic to healthy backend pods.
- Storage objects let applications use persistent data independently of a pod lifecycle. PersistentVolumes represent cluster storage. PersistentVolumeClaims request storage for workloads.

Pods are ephemeral. Kubernetes does not resurrect a failed pod. It creates a new pod to replace it, often with a different name, IP address and placement. Applications that need durable data must store that data outside the pod lifecycle. For multi-container pods, the pod works as one unit. If a required container fails, the pod can stop serving its application. Health probes let the kubelet test whether a container is alive and ready, then Kubernetes can restart or replace unhealthy workloads.
## Cluster architecture
A Kubernetes cluster contains a control plane and one or more worker nodes. The control plane manages the overall cluster state. Worker nodes run application pods and provide compute and memory capacity. Nodes may be physical or virtual machines.

The control plane includes four core components:
- The API server exposes the Kubernetes HTTP API and handles all cluster control requests.
- etcd stores API object state as a consistent key-value data store.
- The scheduler assigns unscheduled pods to suitable nodes by considering resource requests, available capacity and placement rules such as pod affinity and pod anti-affinity.
- The controller manager runs controller loops that watch state and request changes through the API server.

kubectl gives administrators a command-line interface to the API server. It retrieves cluster information and submits changes, including workload deployment and configuration updates.

Each node normally runs three important components. The kubelet watches the API server for pods scheduled to its node, starts and stops pod containers, reports node and pod status, and runs health probes. kube-proxy maintains network rules that implement Services and route traffic to backend pods, although some clusters replace or extend this behaviour through other networking implementations. The container runtime downloads images and runs containers through the Container Runtime Interface. Runtime choice remains separate from image format. Kubernetes no longer includes dockershim. The dockershim integration was announced for deprecation in v1.20 and removed in v1.24, while Docker-built images still run through CRI-compatible runtimes such as containerd or CRI-O.

Cluster add-ons extend the core platform. DNS is the most common add-on. CoreDNS usually provides in-cluster name resolution and service discovery by using data from the Kubernetes API. Ingress controllers or Gateway implementations can expose HTTP and HTTPS applications to clients outside the cluster.
## Pod and service operations
A deployment request starts with kubectl or another API client. The API server validates and stores the Deployment. The controller manager creates the required ReplicaSet and pods. The scheduler selects suitable nodes for those pods and records its decisions. The kubelet on each selected node sees the scheduled work, pulls the required images through the runtime and starts the containers.

If a node fails, its kubelet stops reporting status. Controllers detect that the actual replica count no longer matches the desired count. The ReplicaSet controller requests replacement pods, the scheduler places them on available nodes and kubelets start them. Many clusters taint control plane nodes so ordinary application pods do not run there. Removing that taint can suit laboratories, but production clusters usually keep control plane capacity for control services.

Services give applications a stable access point while pods change behind the scenes. A Service selects backend pods, then Kubernetes and the service proxy keep routing data current through EndpointSlice data. When a pod fails, the ReplicaSet replaces it and the Service stops sending traffic to the failed backend. When the replacement pod becomes ready, the Service can send traffic to it. This combination supports self-healing application behaviour and simple horizontal scaling. Scaling changes the number of pod replicas. The Service keeps the client-facing endpoint steady while the backend set expands or contracts.
## Networking foundations
Kubernetes assigns each pod a unique cluster-wide IP address. Its network model expects pods to communicate directly with pods on the same or different nodes without Network Address Translation. Node agents such as the kubelet must also communicate with pods on their node.

Communication patterns depend on placement. Containers in the same pod share a network namespace and communicate through localhost. Pods on the same node communicate through the pod network using their pod IP addresses. Pods on different nodes communicate through layer 2 or layer 3 connectivity provided by the cluster network. When the underlying network cannot provide direct reachability, an overlay network can present a consistent pod network across nodes.

Services add another networking layer. Clients use a stable Service address or DNS name. Kubernetes then routes requests to ready pods that back the Service and can balance traffic across them. Ingress and Gateway resources can add protocol-aware routing for traffic that enters from outside the cluster.