# Configuring and Managing Kubernetes Networking, Services, and Ingress
## Purpose and prerequisites
Kubernetes networking gives containerised applications a predictable way to communicate across nodes, services, and external entry points. Administrators and developers use it to deploy workloads that keep stable access patterns while Pods start, stop, scale, and move across the cluster.

A practitioner needs working knowledge of Linux, TCP/IP networking, containers, Kubernetes clusters, `kubectl`, Pods, Deployments, labels, and selectors. These foundations support three related responsibilities: cluster networking, application access through Services, and HTTP or HTTPS exposure through Ingress.
## Kubernetes networking model
Kubernetes separates application networking from the physical or virtual network underneath the cluster. The model expects each Pod to have its own IP address and expects Pods to communicate with other Pods across nodes without address translation inside the cluster. Node agents, including the kubelet and kube-proxy, must also reach the Pods that they manage on their own node.

This model simplifies application design. Applications do not need to discover node ports, track dynamic container ports, or understand the cluster topology. They can address other workloads by Service names, DNS records, or Pod IPs when direct troubleshooting requires it.

A cluster usually contains three important address spaces:
- The node network gives each Kubernetes node an address on the data centre or cloud virtual network.
- The Pod network gives each Pod an address from a range managed by the network implementation.
- The Service network gives Services virtual cluster IPs that stable clients can use.

These ranges must not overlap. The network plugin assigns Pod IPs, the API server assigns Service IPs, and the kubelet or cloud-controller-manager reports node addresses. Dual-stack clusters can use both IPv4 and IPv6, but every component must agree on the primary IP family.
## Pod communication and CNI
Containers in the same Pod share one network namespace. They share one IP address and one port space, so they communicate over `localhost`. A Pod sandbox, often implemented with an infrastructure or pause container, owns that namespace for the lifetime of the Pod. Application containers can restart without destroying the Pod network namespace.

Pods on the same node usually communicate through virtual Ethernet pairs connected to a bridge, router, tunnel, or eBPF data path created by the network implementation. Pods on different nodes communicate through routing, encapsulation, or an overlay network. The details vary by plugin, but the Kubernetes contract stays the same: Pod traffic reaches the destination Pod address without applications handling node-level routing.

Kubernetes uses Container Network Interface plugins to implement cluster networking. The container runtime now manages CNI configuration. Older kubelet command-line options for CNI management were removed in Kubernetes 1.24, so current clusters rely on the runtime and the installed plugin rather than kubelet network-plugin flags.

CNI plugins differ in capability. Some provide basic interface creation and IP address management. Others add network policy enforcement, encryption, eBPF acceleration, route distribution, observability, or traffic controls. Calico, Cilium, Flannel, and cloud provider plugins solve similar problems with different design choices.

A Calico-style overlay can use tunnel interfaces and routes to carry Pod packets between nodes. A cloud implementation can use cloud route tables so each Pod CIDR routes to the node that hosts it. In Azure Kubernetes Service, kubenet has become a legacy option and is scheduled for retirement. Azure CNI Overlay is the current default when no network plugin is specified in new AKS clusters.
## Network implementation examples
A network implementation must connect Kubernetes intent to concrete host networking. On each node, the implementation creates interfaces, assigns addresses, and installs routes or dataplane rules. The administrator verifies this work by comparing node addresses, Pod addresses, Pod CIDRs, routes, and the interfaces that carry traffic.

In an overlay design, each node can receive one or more route entries that point remote Pod CIDRs at a tunnel interface. Traffic from a Pod enters the local host stack, matches a route for the destination Pod CIDR, then enters the tunnel. The source node encapsulates the packet and sends it across the node network. The destination node decapsulates it and forwards it to the virtual interface that connects to the target Pod. The application still sees Pod IP to Pod IP communication.

In a routed cloud design, the cloud network can hold routes for each node's Pod CIDR. A node forwards local Pod traffic to the cloud virtual network, and the cloud route table sends the packet to the node that owns the destination Pod CIDR. The destination node forwards the packet to the Pod bridge or dataplane interface. This approach depends on provider integration and route scale limits.

On a single node, virtual Ethernet pairs commonly connect Pods to the host dataplane. One end appears in the Pod network namespace, often as `eth0`. The other end remains on the host and attaches to a bridge, a routing table, or a plugin-specific dataplane. A bridge inspection tool can show which host-side interfaces connect to the bridge, while `ip addr` and `ip route` show Pod and node addressing.

The same inspection pattern helps across CNI plugins:
- `kubectl get nodes -o wide` confirms node addresses on the node network.
- `kubectl get pods -o wide` confirms Pod addresses and placement.
- `kubectl describe node` shows reported Pod CIDRs, node addresses, and implementation-specific annotations.
- `ip addr` shows host interfaces, Pod-facing interfaces, bridges, and tunnel devices.
- `ip route` shows how the node reaches local and remote Pod CIDRs.

These checks reveal whether failure sits in Kubernetes object state, host routing, cloud routing, CNI health, or the application itself.
## Cluster DNS
Kubernetes provides DNS inside the cluster so workloads can use names instead of addresses. CoreDNS commonly serves this function through a Service named `kube-dns` in the `kube-system` namespace. Kubernetes keeps the historical Service name for compatibility, even when CoreDNS runs behind it.

The DNS system creates records for Services and Pods. Normal Services receive A or AAAA records according to the cluster IP family. A typical Service name has this form:

`service-name.namespace.svc.cluster.local`

A Pod can resolve Services in its own namespace by short name. It must include the namespace when it needs a Service in another namespace. Namespaces therefore form DNS subdomains under the cluster domain.

CoreDNS stores its configuration in a ConfigMap, usually `coredns` in `kube-system`. Administrators customise forwarding and stub domains by editing this ConfigMap. For example, the cluster can forward unresolved names to node resolvers, public resolvers, or a specific internal DNS server. More complex environments can use conditional forwarding for selected domains.

Pod DNS behaviour comes from `dnsPolicy` and optional `dnsConfig` settings. The usual default, `ClusterFirst`, sends cluster-domain lookups to the cluster DNS Service and sends other queries according to the node or resolver configuration. `Default` inherits the node resolver configuration. `None` lets the Pod specification define nameservers, searches, and options directly.

DNS is usually better than Service environment variables for service discovery. The kubelet injects Service environment variables only when it starts a Pod. If a Service appears after the Pod starts, that Pod does not receive new environment variables until it restarts. DNS records update without restarting client Pods.
## DNS configuration workflow
A cluster DNS configuration changes through the CoreDNS ConfigMap. The CoreDNS Pods mount the ConfigMap as a configuration file, then reload after the mounted file changes. Administrators can watch CoreDNS logs to confirm that CoreDNS has accepted the new configuration. They should test both ordinary cluster names and any custom forwarding rules after the reload.

Forwarding supports common enterprise DNS patterns. A cluster can forward all non-cluster names to the node resolver, which usually reflects the organisation's DNS configuration. It can also forward selected domains to specific nameservers. For example, an internal corporate domain can resolve through a private DNS server while public names resolve through a separate upstream resolver.

A Pod-level DNS override should remain rare because it changes service discovery behaviour for that workload. `dnsPolicy: None` with `dnsConfig` gives a Pod explicit nameservers and search suffixes. This helps when a workload must query a special resolver, but it can also break cluster Service discovery when configured poorly. A safer pattern keeps `ClusterFirst` for most workloads and customises CoreDNS or namespace-specific configuration where appropriate.

Administrators can validate DNS from a purpose-built debugging Pod. A test Pod with `nslookup`, `dig`, or similar tooling can query `kube-dns`, confirm Service names, confirm namespace search behaviour, and compare short names with fully qualified names. Direct queries against the `kube-dns` Service IP help separate cluster DNS problems from node resolver problems.

DNS names for Services and Pods serve different purposes. A Service DNS name provides a stable name for a changing backend group. A Pod DNS name identifies a particular Pod address and helps diagnostics, but application code should normally use Service names. Service names absorb Pod replacement, readiness changes, and scaling.
## Services
Pods are ephemeral. A Deployment can create, replace, scale, and delete Pods at any time, which changes Pod names and addresses. A Kubernetes Service gives clients a stable endpoint for a changing set of backend Pods.

A Service defines a selector that matches labels on Pods. Kubernetes tracks the matching backend addresses using EndpointSlices. The older Endpoints API is deprecated, so current clients and controllers should use EndpointSlices for scalable and dual-stack aware endpoint data.

The Service exposes one or more ports. The `port` field defines the port that clients use on the Service. The `targetPort` field defines the port on the backend Pods. Protocols can include TCP, UDP, or SCTP, depending on the cluster and implementation.

Kube-proxy or an alternative data plane programs the node-level forwarding behaviour. Traditional kube-proxy implementations use iptables or IPVS. Some modern CNIs replace parts of this path with eBPF. The result is similar for clients: traffic sent to a Service reaches one of the ready endpoints that backs it.
### ClusterIP
`ClusterIP` is the default Service type. It assigns a virtual IP from the Service CIDR and exposes the application inside the cluster. Internal clients send traffic to the ClusterIP and Service port. The node data plane forwards the request to one of the ready backend Pods.

ClusterIP suits internal APIs, databases, application tiers, and workloads that will be exposed externally through Ingress or another gateway. The ClusterIP is stable for the life of the Service, while backend Pods can change continuously.
### NodePort
`NodePort` builds on ClusterIP. Kubernetes allocates a port from the configured node port range, which defaults to 30000-32767. Every node listens on that port and forwards traffic to the Service. A client outside the cluster can connect to any reachable node IP and the assigned node port.

NodePort works well for development clusters and for environments that provide their own external load balancer. It exposes a port on every node, so administrators should restrict access with firewall rules, security groups, or network policy where appropriate.
### LoadBalancer
`LoadBalancer` builds on Service behaviour and asks a supported infrastructure provider to provision an external or internal load balancer. The load balancer receives a public or private address and sends traffic into the cluster. Kubernetes publishes the assigned address in the Service status after provisioning completes.

Historically, most cloud implementations created a NodePort behind every LoadBalancer Service. Kubernetes now supports disabling NodePort allocation for LoadBalancer Services when the provider can send traffic directly to Pods or otherwise does not need NodePorts. Provider behaviour controls health checks, IP assignment, and other load-balancing details.
### ExternalName, headless Services, and Services without selectors
`ExternalName` maps a Kubernetes Service name to an external DNS name by returning a CNAME record. It supports internal discovery for a resource outside the cluster. It can surprise HTTP and TLS clients because the name used by the client may not match the upstream host name or certificate.

A headless Service sets `clusterIP` to `None`. Kubernetes does not allocate a cluster IP, kube-proxy does not proxy it, and DNS returns the backing endpoint addresses. Headless Services support clients that need to choose endpoints themselves, including many StatefulSet and database patterns.

A Service without a selector lets an administrator define EndpointSlices manually. This pattern exposes endpoints that Kubernetes does not select from Pods, such as services outside the cluster or bespoke backend groups. Multiple manually defined endpoints still allow load distribution through the Service abstraction.
## Service definition and dataplane details
A Service manifest links a stable frontend to a selected backend group. The Service selector must match labels on Pods, usually labels created by a Deployment template. When the selector matches, Kubernetes records the ready backend addresses in EndpointSlices and the node dataplane uses that information for forwarding.

A minimal Service definition includes a name, a type, a selector, and at least one port. A Deployment can have many labels for ownership, versioning, or release management, but the Service selector must match the labels that identify the backend set. Overly broad selectors can send traffic to unintended Pods. Overly narrow selectors can leave the Service without endpoints.

The `port` and `targetPort` fields separate client-facing access from container implementation. A Service can listen on port 80 while Pods listen on 8080. This lets administrators change container ports without changing the client contract, as long as the Service and Pod definitions stay aligned. Named ports can make this alignment clearer when several containers or ports exist.

Multiple ports on one Service need names so clients and controllers can distinguish them. A web application might expose HTTP and metrics through separate ports. The Service can publish both while still selecting the same backend Pods. Controllers and tools then read the Service port names rather than guessing by number.

Readiness controls whether a Pod receives Service traffic. A Pod can match the selector and still stay out of EndpointSlices until its readiness probe succeeds. This protects clients during startup, warm-up, or dependency failures. Liveness and startup probes serve different purposes, but readiness drives endpoint availability.

The data path for a ClusterIP request usually starts on the node where the client Pod runs. The node dataplane matches the virtual Service IP and port, chooses a ready endpoint, rewrites the destination as needed, and forwards the packet over the Pod network. The selected endpoint can live on the same node or another node. Return traffic follows the dataplane design used by the CNI and kube-proxy or its replacement.

Session affinity can keep a client connected to the same backend based on client IP, but default Service behaviour distributes requests across ready endpoints according to the dataplane implementation. Production applications should not assume perfect round-robin behaviour unless the implementation documents it and the traffic pattern supports it.

A LoadBalancer Service adds infrastructure integration but does not replace the Service model. The cloud controller or provider-specific controller watches the Service, creates the balancer, publishes the resulting address, and connects the balancer to cluster backends. Provider annotations often control internal versus external addresses, health checks, IP allocation, and protocol-specific options.
## Service discovery and troubleshooting
Service discovery works best when applications use DNS names rather than literal IP addresses. A client in the same namespace can use `service-name`. A client in another namespace can use `service-name.namespace` or the full DNS name.

Administrators can inspect Service behaviour with these objects and commands:
- `kubectl get service` shows type, cluster IP, external IP, and exposed ports.
- `kubectl describe service` shows selectors, ports, and backend endpoint data.
- `kubectl get endpointslices` shows the current backend addresses tracked for Services.
- `kubectl get pods --show-labels` confirms whether Pod labels match the Service selector.

Scaling a Deployment creates or removes Pods. Kubernetes updates the EndpointSlices as the matching Pods become ready or disappear. When a Pod loses the label required by a Service selector, the Pod can remain alive but no longer receives Service traffic.

Direct Pod access helps isolate a failing replica. An administrator can inspect EndpointSlices, identify a backend Pod IP and target port, and run a test from inside the cluster. This approach bypasses Service load distribution and confirms whether one Pod differs from the others.
## Service discovery patterns
DNS gives applications a portable naming contract. An internal API named `orders` in the `payments` namespace can use `orders.payments.svc.cluster.local` as a stable name. A client in the same namespace can shorten that to `orders` because the Pod resolver search list includes its namespace. A client in a different namespace must include the namespace or full name to avoid resolving a different local Service.

Environment-variable discovery exposes Service configuration in a form older applications can consume. The kubelet creates variables such as service host, service port, and protocol-specific values when the Pod starts. This works for static environments, but it fails to reflect Services created later and can clutter application environments in large namespaces. DNS remains the safer default.

ExternalName Services help applications keep Kubernetes-style discovery names while the implementation lives outside the cluster. They do not create a proxy or load-balancing path. The DNS response redirects the client to the external name, and the client then connects using normal DNS behaviour. Because HTTP host headers and TLS certificate names matter, ExternalName fits best when the external service expects the same name or the client can set the correct upstream host.

Headless Services deliberately remove the virtual IP. They suit systems that need endpoint identity or client-side load selection. Databases, quorum systems, and StatefulSets often need stable individual endpoints, not a single balanced VIP. DNS returns the endpoint addresses, and the client or library decides how to use them.

Services without selectors create a controlled bridge between Kubernetes discovery and manually managed endpoints. They support migrations, hybrid applications, and external dependencies. Administrators should document them carefully because Kubernetes does not validate that the manually listed addresses behave like ordinary selected Pods.
## Ingress architecture
Ingress manages external HTTP and HTTPS access to Services. It does not run traffic by itself. An Ingress resource stores routing rules, while an Ingress controller implements those rules by configuring a proxy, load balancer, or cloud service.

A cluster can run more than one Ingress controller. An IngressClass associates an Ingress resource with the controller that should implement it. This lets different teams or workloads use different controllers, policies, or infrastructure.

An Ingress controller commonly exposes itself through a NodePort or LoadBalancer Service. External traffic reaches the controller first. The controller reads the request host, path, and protocol, then routes the request to the intended backend Service or endpoint according to the Ingress rules.

Ingress suits Layer 7 HTTP and HTTPS routing. It can consolidate many web applications behind one address, route requests by host name or path, and terminate TLS at the controller. Non-HTTP TCP or UDP workloads usually need a Service of type LoadBalancer, a Gateway API route that supports the protocol, or another purpose-built load-balancing design.

Gateway API now provides a more expressive, role-oriented model for advanced traffic management. It adds stable concepts such as GatewayClass, Gateway, HTTPRoute, and GRPCRoute. New designs should evaluate Gateway API when they need richer routing, clearer ownership boundaries, or features that previously required controller-specific Ingress annotations.

The community Ingress-NGINX project has been retired. Existing deployments may continue to run, but administrators should migrate to Gateway API or a maintained Ingress controller. This does not mean F5 NGINX Ingress is the same project. The two projects share NGINX as a data plane but have different governance and support models.
## Ingress rules
A simple Ingress can use a `defaultBackend` to send all unmatched requests to one Service. This pattern exposes a single HTTP application through the controller without host or path rules. It also provides a safer fallback for requests that do not match more specific rules.

Host-based routing uses the HTTP host header. For example, `red.example.com` and `blue.example.com` can route to different Services while sharing the same controller address. DNS records for each host should point to the Ingress controller address.

Path-based routing uses URL paths. A request to `/red` can route to a red Service, while `/blue` routes to a blue Service. Each path must include a `pathType`:
- `Exact` matches the full path with case sensitivity.
- `Prefix` matches path elements by prefix with case sensitivity.
- `ImplementationSpecific` delegates matching behaviour to the IngressClass implementation.

A path-based Ingress should usually include a default backend. Without one, requests that do not match a rule normally receive a 404 response. A default backend can serve an error page, route to a landing service, or redirect clients more gracefully.
## TLS with Ingress
Ingress can terminate TLS for HTTPS clients. The administrator creates a TLS Secret that contains the certificate and private key, then references that Secret in the Ingress `tls` section. The host listed in the TLS configuration should match the certificate name and the host rule.

When traffic reaches the controller on HTTPS, the controller presents the certificate, decrypts the client connection, and forwards the request to the backend Service according to the Ingress rule. The connection from the controller to the backend may be HTTP or HTTPS depending on the controller and configuration. Production clusters should use certificates issued by a trusted authority and automate renewal where possible.

Self-signed certificates can test the configuration but should not serve production clients. Test clients such as `curl` may need explicit name resolution and trust flags to connect to a self-signed endpoint during a lab.
## Ingress implementation and verification
An Ingress resource depends on a working backend Service before it can route useful traffic. The controller reads the Ingress, resolves the backend Service and port, and programs its proxy configuration. A Service with no ready endpoints produces failed requests even when the Ingress rule itself is valid.

Ingress status usually shows the address clients should use, but the exact meaning depends on the controller. In a local lab, the address might be a node address behind a NodePort Service. In a managed cloud cluster, it might be the public or private address of a cloud load balancer. Administrators should verify both the Ingress resource and the controller Service.

Host-based routing requires DNS outside the cluster. The public or private DNS record must point to the controller address. During testing, clients can inject a host header or override local name resolution, but production traffic needs ordinary DNS. Without the expected host header, the controller cannot match the intended host rule.

Path matching needs careful design. `Prefix` matches by path element, not by simple string prefix. `Exact` requires a case-sensitive match of the full path. A rule for `/blue` with `Exact` does not match `/blue/1`, and a rule for `/Blue` does not match `/blue`. Applications should define default backends or explicit error routes so unmatched requests fail predictably.

TLS Ingress requires three aligned pieces: a DNS name, a certificate that covers that name, and an Ingress rule for that name. The TLS Secret stores the certificate and private key. The Ingress `tls` section references the Secret and the host. The rule section maps the same host and path to a Service. A mismatch among these pieces can produce certificate warnings, wrong-backend routing, or 404 responses.

Ingress can reduce resource use compared with one LoadBalancer Service per web application. Several host names and paths can share one controller address and one controller deployment. It can also centralise TLS termination, redirects, access logs, and common HTTP policies. The trade-off is that the controller becomes a critical shared component that needs capacity planning, upgrades, monitoring, and clear ownership.

The controller implementation matters. Different controllers support different annotations, traffic policies, timeouts, rewrites, authentication integrations, and observability features. Portable Ingress features cover common HTTP routing, while advanced features often depend on controller-specific extensions. Gateway API reduces some of this portability problem by modelling richer routing as standard API resources.
## Operational guidance
Kubernetes networking works best when operators keep the control plane model clear and choose the data plane deliberately. The API objects define intent, while CNI plugins, kube-proxy replacements, cloud controllers, and Ingress or Gateway controllers implement that intent.

Good cluster designs apply these practices:
- Allocate non-overlapping node, Pod, and Service address ranges.
- Choose a CNI plugin for policy, routing, observability, and provider support requirements.
- Prefer DNS for service discovery and use environment variables only when their startup-time behaviour is acceptable.
- Use EndpointSlices rather than the deprecated Endpoints API for endpoint-aware automation.
- Select the least exposed Service type that meets the access requirement.
- Use ClusterIP for internal workloads, NodePort only when node exposure is intended, and LoadBalancer when infrastructure integration should create an external or internal balancer.
- Use Ingress or Gateway API for HTTP and HTTPS routing that needs hosts, paths, TLS, and shared entry points.
- Keep ingress controllers current and migrate away from unsupported controllers.
- Test routing from both inside and outside the cluster, then confirm DNS, EndpointSlices, Service selectors, and controller logs when traffic fails.

The key concept stays consistent across all three layers. Pods provide compute and receive addresses, Services provide stable discovery and traffic distribution, and Ingress or Gateway controllers provide HTTP and HTTPS entry points. Kubernetes keeps these layers separate so applications can scale and move without forcing clients to track every Pod or node change.