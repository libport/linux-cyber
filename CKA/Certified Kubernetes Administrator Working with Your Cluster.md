# Certified Kubernetes Administrator: Working with Your Cluster
`kubectl` is the main command-line client for Kubernetes. It sends requests to the API server, which exposes the Kubernetes API and stores cluster state in etcd. Administrators use `kubectl` to create, read, update and delete API resources such as Pods, Deployments, Services, ReplicaSets and Nodes.

A standard command combines an operation, a resource type, an optional object name and optional flags:

```bash
kubectl get pods
kubectl get pod nginx -o yaml
kubectl create deployment nginx --image=nginx
```

Resource names also have short names. For example, `po` can refer to Pods, `svc` to Services and `no` to Nodes. Short names reduce typing, especially when Bash completion is enabled.
## Core kubectl operations
Common operations support day-to-day cluster administration:
- `kubectl get` lists resources and shows basic status.
- `kubectl describe` shows detailed configuration, status, conditions and events.
- `kubectl apply -f file.yaml` applies declarative configuration from a manifest.
- `kubectl create` creates resources imperatively or from a file.
- `kubectl run` creates a Pod from an image.
- `kubectl delete` removes a named resource.
- `kubectl explain` describes API resources and fields.
- `kubectl logs` reads container output written to stdout or stderr.
- `kubectl exec` runs a command inside a container.
- `kubectl expose` creates a Service for an existing workload.
- `kubectl scale` changes the replica count of a scalable workload.
- `kubectl edit` opens the live object definition in the configured editor.

The `-h` flag provides command help. For example, `kubectl get -h` shows supported options and examples. `kubectl explain pod.spec.containers` helps administrators build manifests by showing the structure and required fields for a Pod container specification.
## Discovering resources and API structure
`kubectl api-resources` lists the resource types supported by the cluster. Its output includes names, short names, API versions, whether a resource is namespaced and its kind. Nodes and persistent volumes are cluster-scoped. Pods, Deployments and Services usually live inside namespaces.

Namespaces group many resources for management and access control. `kubectl get pods` uses the current namespace, often `default`. System components usually run in `kube-system`, so `kubectl get pods -n kube-system` shows control plane and cluster service Pods such as CoreDNS and kube-proxy.

`kubectl get all --all-namespaces` lists many common resource types across namespaces, but it does not list every resource in the API. Discovery commands remain important when a cluster has custom resources or less common built-in objects.
## Contexts, namespaces and completion
`kubectl` uses the current context from kubeconfig to decide which cluster, user and namespace to use. Administrators should confirm the context before changing resources, especially when they work across development, testing and production clusters. `kubectl config current-context` shows the active context, while `kubectl cluster-info` confirms the API endpoint that receives requests.

Namespace flags reduce mistakes. `-n kube-system` limits a command to the system namespace. `--all-namespaces` expands a read command across namespaces. Write commands should name the target namespace explicitly when the default namespace is not intended.

Bash completion improves accuracy as well as speed. It completes command names, resource types, object names and flags after the shell loads the kubectl completion script. Completion is especially useful with generated Pod names, where Deployments add a ReplicaSet hash and Kubernetes adds a unique suffix.
## Output formats and dry runs
`kubectl` can change output with `-o` or `--output`:
- `-o wide` adds useful columns such as node placement, Pod IP addresses, operating system details or container runtime details.
- `-o yaml` returns a YAML representation of an object.
- `-o json` returns a JSON representation for tools and automation.

YAML and JSON matter because Kubernetes represents desired state as structured API objects. Administrators can inspect a live object, save its definition and reuse part of it in configuration management.

`--dry-run=client -o yaml` generates a manifest without creating the object in the cluster. It works well as a starting point for reliable YAML:

```bash
kubectl create deployment hello-world --image=example/hello-app:1.0 --dry-run=client -o yaml > deployment.yaml
kubectl expose deployment hello-world --port=80 --target-port=8080 --dry-run=client -o yaml > service.yaml
```

Generated manifests still need review. They often include a minimal, valid structure rather than the complete policy, labels, resource requests, probes and security settings needed for production.
## Inspecting cluster state
`kubectl cluster-info` shows the API server endpoint for the current context. `kubectl get nodes` shows node names, status, roles, age and Kubernetes version. `kubectl get nodes -o wide` adds information such as internal IP addresses, operating system images, kernel versions and container runtimes.

`kubectl describe node c1-cp1` gives deeper detail about a node. It reports labels, annotations, conditions, addresses, capacity, allocatable resources, system information and Pods running on that node. Conditions reveal problems such as memory pressure, disk pressure or network unavailability. Capacity shows physical resources. Allocatable resources show what Kubernetes can schedule for workloads after system reservations.

`kubectl describe pod name` gives scheduling and runtime detail for a Pod. Events show the scheduler assigning the Pod to a node, the kubelet pulling or reusing an image, the container runtime creating the container and the kubelet starting it. These events are often the quickest path to failed image pulls, scheduling failures and container start errors.
## Reading ownership and readiness
Kubernetes object names often reveal ownership. A Deployment named `hello-world` creates a ReplicaSet with the Deployment name plus a Pod template hash. That ReplicaSet creates Pods whose names include the same prefix plus a unique suffix. The `Controlled By` field in `kubectl describe pod` confirms the owner rather than relying only on the name.

Status columns give a quick health check. A Pod readiness value such as `1/1` means one of one containers is ready. A Deployment readiness value such as `20/20` means all requested replicas are available. Restart counts show container restarts, while age shows when the object was created, not how long the current container process has run.

Events place those facts in time. They show scheduling decisions, image pull attempts, container creation and start operations. Events do not replace logs, but they often explain why logs are absent. For example, a Pod stuck in `ImagePullBackOff` may never start the application process that would produce logs.
## Logs and container access
`kubectl logs pod-name` reads logs from a container in a Pod. For multi-container Pods, the command should include `-c container-name`. Logs help diagnose application startup, configuration and runtime issues when the application writes useful output to stdout or stderr.

`kubectl exec -it pod-name -- /bin/sh` starts an interactive shell inside a container when the image includes a shell. Administrators can inspect files, environment variables, DNS resolution and network interfaces from the container's point of view. The command runs through the Kubernetes API and the kubelet on the target node, so it does not require direct SSH access to that node.

Direct node access with tools such as `crictl` can confirm which containers run on a specific node, but Kubernetes administration should usually start with API objects and `kubectl`. Direct runtime inspection is useful for low-level troubleshooting.
## Imperative and declarative management
Imperative commands make immediate changes from the command line. They suit quick experiments and urgent repairs:

```bash
kubectl create deployment hello-world --image=example/hello-app:1.0
kubectl run hello-world-pod --image=example/hello-app:1.0
```

A Deployment creates and manages Pods through a ReplicaSet. A bare Pod created with `kubectl run` has no higher-level controller. If the Pod fails or a node disappears, Kubernetes has less information about the desired application state than it has for a Deployment.

Declarative management stores the desired state in manifests and applies them to the cluster:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

Declarative configuration scales better than repeated one-off commands because it creates an auditable source of truth. Teams can review, version and reuse manifests. They can also reapply the same files to correct drift.
## Manifest structure
A basic Pod manifest includes:
- `apiVersion`, which identifies the API group and version.
- `kind`, which identifies the object type.
- `metadata`, which names and labels the object.
- `spec`, which describes the desired state.

A Pod `spec` must define at least one container. Each container needs a unique name within the Pod and an image. A Deployment manifest uses `apiVersion: apps/v1`, `kind: Deployment`, metadata, a desired replica count, a selector and a Pod template. The selector matches labels on Pods created from the template. The template contains the Pod metadata and Pod specification used for each replica.

Labels and selectors connect related resources. A Deployment selector identifies the Pods it manages. A Service selector identifies the Pods that should receive traffic. Clear labels make troubleshooting and automation easier.
## Deployment flow inside Kubernetes
When `kubectl apply -f deployment.yaml` creates or updates a Deployment, the request goes to the API server. The API server validates the object and persists state in etcd. Controllers and the scheduler watch the API server, not etcd directly.

The Deployment controller reconciles the desired Deployment state by creating or updating a ReplicaSet. The ReplicaSet controller reconciles the desired replica count by creating or deleting Pod objects. The scheduler watches for Pods without assigned nodes and binds each suitable Pod to a node.

The kubelet on each node watches the API server for Pods assigned to that node. When it finds assigned work, it asks the container runtime to pull the required image and start containers. The kubelet reports Pod and container status back through the API server.

Services use selectors to find matching Pods. Modern Kubernetes clusters represent Service backends with EndpointSlices, while older workflows may also show Endpoints. kube-proxy watches Service and EndpointSlice changes and programs node-level networking rules so traffic to a Service reaches one of its ready backends.
## Services and application access
A ClusterIP Service gives an application a stable virtual IP inside the cluster. It exposes a Service port and forwards traffic to a target port on matching Pods:

```bash
kubectl expose deployment hello-world --port=80 --target-port=8080
kubectl get service hello-world
kubectl describe service hello-world
```

The Service port is the port clients use. The target port is the port where the container listens, such as port 8080 for a web application. `kubectl describe service` shows the Service type, ClusterIP, ports and backend addresses. EndpointSlices show the Pod IP addresses and ports that currently back the Service.

When a Deployment scales from 1 replica to 20 or 40 replicas, the Service distributes traffic across the ready Pods selected by its labels. Accessing a Pod IP directly can help isolate a single backend during troubleshooting, but clients should normally use the Service address because Pod IPs are temporary.
## Imperative exposure and generated manifests
`kubectl expose deployment` can create a Service quickly from an existing Deployment. The command infers selectors from the workload and maps a stable Service port to the target port used by the containers. The same command can run with `--dry-run=client -o yaml` to create a Service manifest before any object reaches the API server. This pattern combines speed with reviewability.

Generated YAML should become a starting point, not a finished design. Production manifests usually need labels that match team conventions, resource requests and limits, readiness probes, liveness probes, security contexts and rollout settings. The generated object proves the structure, while review adds operational intent.
## Changing existing resources
Administrators can change a Deployment by editing its manifest and applying it again:

```bash
kubectl apply -f deployment.yaml
```

Changing `spec.replicas` from 1 to 20 in the manifest and reapplying it changes the live Deployment. Kubernetes then reconciles the cluster until the requested number of Pods is ready.

`kubectl edit deployment hello-world` changes the live object directly. `kubectl scale deployment hello-world --replicas=40` also changes the live object directly. These commands are useful, but they can create drift when the manifest or repository still records an older desired state. Declarative files should be updated after any urgent live change.

Deleting a Deployment removes its owned ReplicaSets and Pods through Kubernetes ownership and garbage collection. A Service is a separate object and must be deleted separately when it is no longer needed:

```bash
kubectl delete deployment hello-world
kubectl delete service hello-world
```
## Practical operating principles
Effective Kubernetes administration starts with the API model. `kubectl` is not just a shell tool. It is a client for structured resources, their desired state and their observed state.

Administrators should use `get` for a quick view, `describe` for detailed status and events, `logs` for application output, `exec` for in-container inspection and `explain` for API field discovery. They should prefer declarative manifests for repeatable work and reserve imperative edits for temporary, exploratory or emergency changes.

Strong troubleshooting follows the chain from intent to runtime state: manifest, API object, controller, ReplicaSet, Pod, scheduler decision, kubelet action, container runtime and Service routing. Each layer exposes enough information through the Kubernetes API to show where the actual state diverges from the desired state.