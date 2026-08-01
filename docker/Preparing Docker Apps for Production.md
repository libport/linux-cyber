# Preparing Docker Applications for Production
> [!NOTE]
> This guide explains how to make containerized applications production-ready through portable images, runtime configuration, secure secrets, centralized logging, meaningful health checks, resilient routing, and explicit deployment models.

Production containers need more than a working image. Each application must cooperate with the platform that configures it, collects its logs, assesses its health, and routes its traffic. This integration allows the platform to operate replicated workloads, roll out updates, replace failed instances, and separate application code from environment-specific settings.

A sound production design promotes one immutable image through development, testing, staging, and production. The image contains the same application binaries and runtime dependencies in every environment. Runtime configuration supplies the differences, including service addresses, release settings, log levels, credentials, and feature controls. An image digest provides a stronger identity than a mutable tag and allows operators to confirm that production runs the artefact that passed earlier checks.

Containers isolate processes and their resources while sharing the host kernel. They are not small virtual machines. Their short lifespans, replaceable filesystems, and platform-managed networking shape the application design. Persistent data belongs in durable storage, configuration belongs outside the image when it varies by environment, and operational output belongs on interfaces that the platform can observe.

Four capabilities establish the core production contract:
- Configuration enters through controlled runtime sources without rebuilding the image.
- Application logs reach standard output and standard error in a useful, structured form.
- Health signals distinguish startup, readiness, and liveness without causing avoidable failures.
- A managed gateway or reverse proxy provides routing, TLS, load balancing, and traffic policy.

These capabilities support, but do not replace, metrics, traces, resource controls, security hardening, backups, capacity planning, and incident response.
### A representative distributed application
A small image-gallery system illustrates the contract. A Go web component serves the user interface. A Java API retrieves and caches NASA's Astronomy Picture of the Day data. A Node.js API records access events. Each service runs in its own container, uses a stable service name to reach its dependencies, and exposes only the interface needed by other components.

The three languages use different configuration libraries and logging frameworks, but operators should see a consistent application. Every component receives an environment name and release identifier. Each component writes logs with a common correlation identifier. Each HTTP service exposes separate readiness and liveness endpoints. A gateway presents one public origin and directs website and API paths to the correct internal Services.

This separation creates useful failure boundaries. The web component can scale independently from the cache API. A failing access recorder should not necessarily take the gallery offline. The Java API can continue serving cached data during a short upstream outage if the product accepts stale responses. Those decisions belong in service contracts and health logic rather than in the container runtime.

The deployment model should also state ownership. An application team owns code, application configuration, and service-level signals. A platform team may own cluster logging, gateway infrastructure, certificate automation, and policy. Clear boundaries prevent application manifests from acquiring unrestricted host access while still allowing teams to publish the routes, settings, and telemetry their services require.

A local Compose deployment can exercise the same fundamental interfaces as Kubernetes. Compose provides service DNS, mounted files, secrets, health checks, and private networks. Kubernetes adds controllers, declarative rollouts, Services, ConfigMaps, Secrets, probes, and Gateway API resources. Exact manifests differ, but the image's operational contract should remain stable. A production migration then changes the platform model rather than the application binary.
## Runtime configuration
### A layered configuration model
An application should combine configuration sources according to an explicit precedence order. A practical order, from lowest to highest priority, is:
1. Non-sensitive defaults packaged in the image
2. An environment-specific configuration file mounted at runtime
3. Environment variables for targeted overrides
4. Command-line flags or an explicit administrative override, when the application supports them

The exact order depends on the framework. The application must document and test its own rules. A Go service can use Viper, a Node.js service can use node-config, and a Spring Boot service can use its native externalised-configuration support. Each component may use a different library, but the distributed application should expose a coherent operational model.

Defaults should let the process start safely in a local or diagnostic environment. They should not contain production credentials, private keys, or environment-specific endpoints. Runtime values then alter behaviour without changing the image. A configuration report may show non-sensitive effective values at startup, but it must redact credentials and tokens.

Environment variables work well for small scalar values such as a log level, feature flag, or service hostname. They become awkward for long, structured documents and can expose secrets through process inspection, debugging output, support bundles, or platform interfaces. Files suit structured TOML, JSON, YAML, XML, and properties data. Secret files or references to an external secret store suit sensitive values.

The configuration contract should define:
- Every supported setting, type, default, and valid range
- The precedence of sources
- Whether the application reads a value only at startup or can reload it
- Whether a change requires a rolling restart
- Which values are confidential
- How validation failures appear in logs and status

The process should validate the complete effective configuration before accepting traffic. A missing required value, invalid port, malformed URL, or unsupported mode should cause a clear startup failure. Silent fallback can make a deployment appear healthy while it uses the wrong environment or dependency.

Configuration parsing should preserve types. The string `false` must not become true because a library treats every non-empty string as truthy. Durations need units, lists need unambiguous encoding, and numbers need safe bounds. A schema or typed options object can catch these errors during automated tests and again at startup. Validation messages should name the invalid setting without echoing its sensitive value.

Configuration ownership also affects change control. A feature flag may change frequently without an image release, while a database schema name may need coordination with a migration. Teams should classify settings by owner, risk, and reload behaviour. High-risk changes need peer review, progressive rollout, and a rollback value. An emergency override should expire so it does not become undocumented permanent configuration.
### Docker and Compose
Docker can inject environment variables and mount files when it creates a container. Compose records these settings with the service definition and makes a multi-container application repeatable. The Compose model can declare networks, volumes, configuration, health checks, secrets, and dependencies alongside the image reference.

Bind mounts are useful during development and for controlled single-host deployments, but they couple a container to a host path. Production mounts should be read-only unless the process must write to them. A missing or incorrectly permissioned host file can prevent startup, and a broad bind mount can expose more of the host than the application requires.

Compose secrets provide sensitive data to authorised services as files under `/run/secrets`. They provide a clearer boundary than ordinary environment variables. The application can receive a non-sensitive environment variable that names the secret file, then read the credential from that file. Secret source files must remain outside source control and require appropriate host permissions.

The `depends_on` field controls service creation and shutdown order. Short-form `depends_on` does not prove that a dependency can serve requests. Long-form conditions can wait for a dependency marked `service_healthy`, or for an initialisation service that completes successfully. Applications still need bounded connection retries because dependencies can fail after startup.

Compose files belong in source control when they contain non-sensitive application topology and defaults. Environment-specific secret values, local `.env` files, and private key material do not. A release process should render and review the effective Compose configuration without printing confidential values.
### Kubernetes configuration
Kubernetes separates workload objects from configuration objects. A Deployment commonly manages replicated, stateless Pods through a ReplicaSet. StatefulSets, DaemonSets, Jobs, and other controllers manage different workload patterns. A Service gives changing Pod replicas a stable network endpoint, while cluster DNS publishes consistent names for Services and selected Pods.

A ConfigMap holds non-confidential configuration. A Secret holds confidential data and supports separate access controls. Both can appear in a container as environment variables or mounted files. This flexibility allows one image to run with different settings in several namespaces or clusters.

Kubernetes Secrets are not a complete secret-management system. Base64 encoding does not encrypt data, and Secrets are stored unencrypted in etcd unless the cluster enables encryption at rest. Production clusters should apply least-privilege role-based access control, restrict which Pods can consume each Secret, enable encryption at rest, protect backups, audit access, and consider an external secret store. Plain-text credentials must not remain in Git-managed manifests.

Configuration updates have different effects:

| Delivery method | Behaviour after an object changes |
| --- | --- |
| Environment variable | The running process keeps the old value until the Pod is replaced |
| Mounted ConfigMap or Secret volume | Kubernetes updates the projected files after a delay, except for some mount patterns such as `subPath` |
| Application-specific reload | The process must detect or receive the change and apply it safely |
| Versioned immutable object | A workload update points to a new object and triggers a controlled rollout |

A mounted file changing does not guarantee that the application reloads it. Many frameworks read configuration only at startup. Versioned ConfigMaps and Secrets, combined with a checksum or name change in the Pod template, provide an auditable rolling update. Immutable objects also reduce accidental mutation.

Kubernetes manifests should state resource names, namespaces, selectors, ports, service accounts, probes, volume mounts, and security settings explicitly. A ConfigMap does not create service discovery, and a Service does not create the cluster DNS system. The DNS add-on publishes records from Kubernetes data, while the Service and EndpointSlice mechanisms route traffic to eligible backends.
### Adapting applications with limited configuration support
Some older applications read one fixed file and cannot merge environment variables or override files. A small configuration adaptor can translate runtime inputs into the file format the application expects. A multi-stage build can compile both the adaptor and application while keeping build tools out of the final image.

The adaptor should:
- Read a default template, mounted overrides, and permitted environment variables
- Validate names, types, ranges, and required values
- Write the generated file with restrictive permissions
- Avoid logging secret values
- Fail before the application starts if configuration is invalid
- Replace itself with the application process so signals reach the application correctly

A dedicated entrypoint program is safer than a long shell expression that backgrounds processes or obscures exit codes. On Kubernetes, an init container can generate configuration into a shared `emptyDir` volume before the main container starts. The Pod should remain unready until the application has loaded valid settings.

This adaptor pattern adds code and operational risk. Native framework configuration remains preferable because it usually handles parsing, precedence, validation, and reload behaviour more reliably. Teams should not duplicate configuration mechanisms that the application platform already supplies.

An adaptor also needs atomic output. Writing directly over the target file can leave partial data if the process stops halfway. The adaptor can write a temporary file on the same filesystem, set its mode and ownership, validate the completed content, then rename it over the target. The application should not start until that operation succeeds. If several containers share storage, each instance should generate its own local file or use coordination that prevents concurrent writers.
### Promoting the same image
Separating configuration from code supports image promotion. A build pipeline creates an image once, scans and tests it, signs or attests it where required, and deploys the same digest to each environment. Deployment definitions provide environment-specific settings. Rebuilding the image for production would produce a different artefact and weaken the evidence gathered during testing.

Promotion does not prove that an application is secure or correct. Tests must cover the production configuration shape, dependency versions, permissions, networking, and failure modes. Operators should record the image digest, configuration version, secret version, and deployment revision so an incident can be traced to a precise release.
## Application logging
### The container logging contract
Docker captures a container's standard output and standard error through its configured logging driver. Kubernetes follows the same basic application contract through the container runtime. The entrypoint process, and child processes that inherit its streams, should write operational events to those streams while running in the foreground.

An application that writes only to an internal file, syslog, the Windows Event Log, or ETW can appear silent to the platform's normal container-log pipeline. Native console output is therefore the preferred design. Most modern logging frameworks can select a console sink and adjust severity through configuration.

Useful logs require useful application events. A platform cannot infer a request identifier, business failure, cache miss, dependency timeout, or authorisation decision if the code never records it. At the same time, excessive debug output increases cost, hides important events, and can disclose data. Production defaults should usually record informational lifecycle events, warnings, and errors. Temporary diagnostic changes should be time-bounded and reversible.

Structured, one-event-per-line JSON works well for distributed systems. Each event should use stable field names and include relevant context, such as:
- Timestamp with time zone
- Severity
- Service and version
- Environment or cluster identifier
- Request, trace, or correlation identifier
- Event name or code
- Outcome and duration
- Dependency name when relevant

The application should not log passwords, API keys, session tokens, private data, or complete request bodies by default. Redaction belongs close to the logging call and should undergo tests. Untrusted values need encoding or structured fields so attackers cannot forge extra log lines.
### Log levels and configuration
Framework and application logs often need different thresholds. A Java service may suppress routine framework messages at `WARN` while retaining its own events at `INFO`. A Node.js service may enable `DEBUG` briefly for a component under investigation. The effective configuration should remain consistent across replicas so an operator can compare like with like.

Changing a log level does not require rebuilding an image. A mounted configuration file, environment variable, or platform-specific dynamic setting can control it. If the framework reads the level only at startup, a rolling restart applies the change. A diagnostic level should expire or be returned to its normal value after the investigation.
### Relaying file-based logs
An application that cannot write to the console may need a relay. The relay follows a file or subscribes to an operating-system log source, then writes each entry to its own standard output. Microsoft Log Monitor supports several Windows log sources for Windows containers. A Linux container can use a suitable relay utility when application changes are impossible.

Running the application in the background and a `tail` process in the foreground has serious limitations. Docker then observes the relay as the initial process. An application crash may leave the relay and container running, while a relay crash may stop a healthy application container. Shell wrappers can also mishandle termination signals, orphan child processes, and lose the application's exit status. A purpose-built supervisor can coordinate both processes, but changing the application to use console logging remains cleaner.

Kubernetes can place a logging sidecar beside the application and share a volume between them. The application writes a file, and the sidecar streams it to standard output. This design lets Kubernetes observe each process separately, but it duplicates I/O, consumes additional resources, and can duplicate records if rotation and checkpoints are wrong. Kubernetes-native sidecars improve lifecycle ordering, yet the design still needs back-pressure, rotation, and failure tests.
### Central collection
Local container logs do not provide durable, cluster-wide investigation. Containers disappear, nodes fail, and replicas spread events across machines. A central pipeline normally contains:
1. A collector on each node or a platform integration
2. Parsers and filters that normalise records and attach metadata
3. Durable storage with retention and access controls
4. Search, dashboards, and alerting

Fluent Bit or Fluentd can collect records and forward them to Elasticsearch and Kibana, OpenSearch, a cloud service, or another supported destination. Elasticsearch is one possible store, not a requirement. The data model, query needs, throughput, retention, cost, and licensing should drive the choice.

In Kubernetes, a node-level collector commonly runs as a DaemonSet and reads container logs exposed by the runtime. It enriches each record with namespace, Pod, container, node, image, and label metadata before forwarding it. The implementation must match the cluster's container runtime and managed-service architecture. Direct host-path access is privileged and should be read-only, narrowly scoped, and granted only when the collector requires it.

The collector's service account needs only the Kubernetes metadata permissions required for enrichment. Separate indexes, streams, or tenants can isolate application logs from control-plane logs and restrict access by team or sensitivity. Retention and rotation limits must apply at the node and central store so a noisy service cannot exhaust disk capacity.

A central system should preserve the original event, prevent silent loss, and expose delivery failures. Buffering protects against temporary store outages, but an unbounded buffer can fill a node. Sampling can control high-volume events, though errors and security-relevant events may require complete retention. Clock synchronisation, consistent timestamps, and correlation identifiers allow operators to reconstruct a request across services.

Parsing should happen deliberately. A collector that guesses formats can split multiline stack traces, misread timestamps, or discard fields after an application update. One JSON object per line avoids much of this ambiguity. When an older application emits multiline records, the collector needs a tested parser and a maximum record size. Schema changes should remain backward compatible long enough for stored dashboards and alerts to continue working.

Back-pressure needs an explicit policy. A collector may buffer, block, sample, or drop when the destination cannot keep up. Blocking an application's output can eventually stall the process, while dropping silently removes evidence. The selected collector should publish its own queue depth, retry count, rejected records, and storage use. Alerts should fire before a full buffer begins losing events.

The Elasticsearch, Fluent Bit, and Kibana pattern demonstrates the flow. Fluent Bit tails runtime-managed files on each node, attaches workload metadata, and sends application and platform events to separate destinations or indexes. Elasticsearch stores searchable records, and Kibana provides queries and dashboards. A production deployment needs authentication, TLS, persistent storage, replicas, lifecycle policies, capacity limits, and tested recovery. A single Elasticsearch Pod with ephemeral storage is suitable only for a demonstration.

Logs form one part of observability. Metrics show rates, saturation, and trends. Traces show a request's path across components. Logs provide detailed events. Production operations need all three, along with deployment events and platform state.
## Health, readiness, and self-healing
### Process state is not application health
A running process can still reject every request, deadlock, exhaust a worker pool, or serve corrupt responses. Container status alone therefore gives an incomplete signal. Docker health checks and Kubernetes probes let the platform test application behaviour.

Self-healing means that the platform performs a bounded recovery action when a trusted signal fails. It does not repair defective code or protect in-memory data. A restart can clear a transient fault, but it can also erase ephemeral state, repeat work, or create a restart loop. Durable data must live outside the container, and handlers should tolerate retries where the business operation permits them.

Health endpoints should be cheap, local, side-effect free, and safe under frequent execution. They should not expose configuration, stack traces, credentials, or internal topology. Separate detailed diagnostic endpoints can require authentication and network restrictions.
### Docker health checks
A Dockerfile `HEALTHCHECK` defines a command that runs inside the container. The command returns exit code `0` for success and `1` for failure. Exit code `2` is reserved. Docker tracks `starting`, `healthy`, and `unhealthy` states and retains recent check output for inspection.

The main controls are:

| Control | Purpose |
| --- | --- |
| `interval` | Time between normal checks |
| `timeout` | Maximum duration of one check |
| `retries` | Consecutive failures required before the container becomes unhealthy |
| `start_period` | Initialisation window before failures count towards `retries` |
| `start_interval` | Check frequency during the start period on supported Docker versions |

If `retries` equals three, the third consecutive counted failure marks the container unhealthy. A successful check resets the consecutive-failure count. A standalone Docker Engine records the unhealthy state and emits an event, but the health status alone does not restart the container. An orchestrator or external controller decides how to respond.

An HTTP check can use `curl`, but installing a large diagnostic package only for a probe increases image size and maintenance. A small static probe binary or a script using the application's existing runtime may reduce extra dependencies. For Kubernetes, distroless images can expose an HTTP endpoint that the kubelet checks without adding a client package to the image itself.

A custom probe must remain simpler than the application. Complex probe code creates another failure path. It should apply a strict timeout, handle connection errors, produce concise output, and avoid remote calls that consume rate limits.
### Kubernetes probe types
Kubernetes assigns distinct meanings and actions to three probes:

| Probe | Question | Failure effect |
| --- | --- | --- |
| Startup | Has the application completed startup? | Kubernetes suppresses liveness and readiness checks until startup succeeds. Repeated failure restarts the container according to policy. |
| Readiness | Can the application accept traffic now? | Kubernetes marks the Pod unready and removes it from eligible Service endpoints. The container keeps running. |
| Liveness | Is the application stuck and unlikely to recover without a restart? | The kubelet restarts the container after the configured failure threshold. |

The kubelet performs each probe. An `exec` probe runs a command in the container. HTTP, TCP, and gRPC probes make network checks according to their respective rules. An HTTP probe does not require `curl` in the image because the kubelet sends the request to the Pod address.

Probe controls include `initialDelaySeconds`, `periodSeconds`, `timeoutSeconds`, `successThreshold`, and `failureThreshold`. Kubernetes acts when the configured number of consecutive results reaches the threshold. A `failureThreshold` of two means two consecutive failures, not three.

A startup probe suits slow-starting applications. It gives the process a bounded initialisation period without weakening liveness for the rest of its life. Using a fixed sleep in an entrypoint delays every start and cannot report early success. A startup probe observes actual progress and lets normal checks begin as soon as the application starts successfully.
### Designing probe logic
Liveness should usually test only the process's ability to make progress. It might verify that the event loop responds, an internal queue advances, or a local health handler can execute. It should not call a public third-party API. If that dependency reaches a rate limit or suffers an outage, a dependency-based liveness check can restart every replica, increase traffic to the failing service, and take an otherwise recoverable application offline.

Readiness can include dependencies that are essential for serving new requests, but it needs care. A brief dependency failure across all replicas can remove every endpoint and cause a complete outage. Cached or degraded responses may be better than unready status. The design should distinguish between an optional dependency, a critical dependency, and a condition that the application can handle through retries or circuit breaking.

A dependency check in the container startup command can fail fast when a required local file or schema is absent. It should not wait indefinitely for a network service. Bounded retry with exponential backoff and jitter avoids synchronised reconnect storms. In Kubernetes, startup and readiness probes normally express service availability more accurately than a long shell chain in the entrypoint.

Probe endpoints should return success only when the stated condition holds. A readiness endpoint may return a non-success status during draining so the platform stops new traffic before termination. The application then handles `SIGTERM`, stops accepting work, completes or safely abandons in-flight requests, flushes essential buffers, and exits within the termination grace period.
### Rollouts and recovery
A Deployment performs a rolling update by creating new Pods and reducing old replicas according to its strategy. Readiness prevents traffic from reaching a new Pod before it can serve requests. `maxUnavailable`, `maxSurge`, readiness timing, and termination behaviour determine whether the rollout preserves capacity.

Liveness failures restart a container in the same Pod. A Deployment may create a replacement Pod when a Pod disappears or cannot remain scheduled, but a liveness failure does not itself create a new Pod. The distinction affects identity, volumes, restart counts, and investigation.

Operators should alert on repeated restarts, long unready periods, probe latency, and rollout stalls. A platform that repeatedly restarts a faulty component can hide symptoms while serving intermittent errors. Events, previous container logs, exit codes, and probe output should remain available long enough to identify the root cause.

Probe settings need load and failure testing. Aggressive intervals waste CPU and network capacity. Loose timeouts delay detection. A useful configuration allows normal latency variation, detects genuine failure within the service objective, and avoids correlated restarts across replicas. Randomised application retry behaviour further reduces recovery storms.
### Failure scenarios
Several failure cases clarify the probe boundaries. If a web handler enters a permanent internal failure state while its process stays alive, readiness can stop new traffic and liveness can restart the container. If one of three replicas fails readiness, the Service continues routing to the other two, provided they have enough capacity. When the failed replica recovers, a successful readiness result makes it eligible again.

If an access-count service stores its total only in memory, a liveness restart restores availability but resets the count. The restart exposes a state-design defect rather than fixing it. Durable counters need external storage, or the product must accept their loss. Similar concerns apply to queued work, local caches, temporary uploads, and transactions interrupted during termination.

If every replica uses a liveness probe that calls the same rate-limited public API every five seconds, the probes can exhaust the allowance. Each failed call then triggers restarts, which create more startup calls and prolong the outage. Liveness should stay local. A readiness signal can reflect an essential upstream dependency only when removing the replica helps and when the check itself does not worsen the dependency's load.

If startup normally takes between 20 and 90 seconds, a liveness probe with a 10-second initial delay can kill healthy instances before they finish. A startup probe can allow a bounded 120-second window, then hand control to a stricter liveness probe. The readiness probe can succeed later if warming a cache or joining a pool takes additional time.

A rollout can fail even when every probe is technically correct. New replicas may consume more memory, a database migration may be incompatible with old replicas, or a readiness endpoint may pass before caches warm. Pre-production tests should cover mixed-version operation, peak traffic, termination, rollback, dependency failure, and node disruption. Progress deadlines and availability budgets should stop a bad rollout before it removes the last sound replica.
## Routing production traffic
### Ports, services, and entry points
Applications inside separate containers can listen on the same private port because each container has its own network namespace. A conflict occurs when two processes try to bind the same host IP, protocol, and port tuple. Publishing every application container directly to a fixed host port therefore limits scaling and exposes internal services unnecessarily.

A reverse proxy or gateway provides a small set of public entry points, usually ports 80 and 443, then routes requests to private backends. Routing rules can use the hostname, path, method, headers, or other protocol-specific attributes. The gateway can also terminate TLS, balance load, apply authentication, enforce rate limits, add security headers, and collect access logs.

The application network should expose only the gateway publicly. Internal APIs should use private networks or ClusterIP Services and network policies. Administrative dashboards need authentication and should not share unrestricted public exposure with application routes.
### NGINX and static configuration
NGINX can map virtual hosts and paths to upstream services. For example, one hostname can reach a website while `/api` reaches a separate application service. Upstream groups can balance requests across replicas. Proxy caching can reduce latency and backend load for cacheable responses.

Caching needs an explicit policy. Personalised, authorised, or mutable responses can leak data or become stale when cached under an incomplete key. Operators should define cache keys, permitted status codes, freshness, invalidation, size limits, and bypass rules. Response headers should make cache behaviour observable.

Static NGINX configuration must be regenerated or reloaded when routing rules change. A graceful reload can apply valid configuration without dropping established connections, so a full container restart is not inherently required. Upstream DNS behaviour also depends on the NGINX version and resolver configuration. Current releases can re-resolve configured upstream names in supported configurations, while a platform-aware controller can generate upstream state from an API.

Configuration should undergo syntax validation before reload. Multiple gateway replicas prevent one bad process or node from becoming the only entry point. Health checks, connection draining, and disruption controls protect traffic during gateway updates.
### Traefik and dynamic discovery
Traefik can watch a provider such as Docker and build routers, services, and middleware from container or service labels. With `exposedByDefault` disabled, only workloads carrying an explicit enable label become routable. Host and path rules select a service, while middleware can redirect HTTP to HTTPS, strip a prefix, alter headers, or apply other policy.

Dynamic discovery removes the need to edit a central routing file for every replica change. Traefik updates its backend set when the provider reports container lifecycle events. Application definitions then carry route metadata close to the workload, which improves deployment autonomy but also grants application authors influence over shared ingress behaviour. Label constraints and governance should restrict which workloads the gateway accepts.

Mounting `/var/run/docker.sock` into a proxy grants powerful access to the Docker API. Docker's default authorisation model gives an API client extensive control over the host, so a compromised proxy can become a host compromise. A production design should avoid a raw socket mount where possible. Alternatives include a protected API endpoint, an authorised socket proxy with a minimal method allowlist, SSH or mutually authenticated TLS, a rootless architecture, or a provider integration with narrower credentials. A read-only filesystem mount does not make Docker API operations read-only.

Traefik can obtain and renew certificates through an ACME certificate resolver. Production deployments need durable, protected certificate state, high-availability planning, challenge configuration, renewal monitoring, and issuer rate-limit awareness. A self-signed default certificate helps local testing but does not provide public trust and should not train users to bypass browser warnings.

Sticky sessions can help an older stateful application during migration, but they reduce load-distribution freedom and complicate failover. External session storage or stateless request handling usually gives stronger resilience. Gateway features should support the application design rather than preserve avoidable local state indefinitely.
### Kubernetes Services and Gateway API
A Kubernetes Service selects a changing group of Pods and provides a stable endpoint. Readiness influences whether a Pod endpoint is eligible for normal Service traffic. A ClusterIP Service remains reachable only inside the cluster unless another mechanism exposes it. A LoadBalancer Service asks the environment to provision or configure an external load balancer, subject to provider support.

The original Ingress API still works, but Kubernetes has frozen it and recommends Gateway API for new development. Gateway API separates infrastructure and application routing responsibilities:
- `GatewayClass` identifies the controller and implementation.
- `Gateway` defines listeners and the gateway instance.
- `HTTPRoute` maps hosts, paths, headers, and other HTTP conditions to Services.
- Other route kinds support additional protocols when the selected controller implements them.

This model lets a platform team own public addresses, listeners, TLS policy, and accepted namespaces while application teams own routes to their Services. Attachment rules create an explicit trust relationship between Gateways and Routes. Implementations vary, so teams should confirm conformance, supported features, upgrade policy, and security maintenance before selection.

The community ingress-nginx controller was retired on 24 March 2026. Its existing artefacts still run, but the project no longer publishes bug fixes or security updates. New deployments should use a maintained Gateway API implementation or another maintained ingress controller. Existing ingress-nginx users should inventory annotations and custom behaviour, select a supported replacement, test translated routes, and migrate without assuming exact feature equivalence.

An Ingress resource should not be confused with the retired ingress-nginx implementation. The stable Ingress API remains available, although it receives no new features. Organisations that retain it still need an actively maintained controller.
### TLS and traffic policy
TLS should protect public traffic with certificates from a trusted issuer. The gateway needs automated issuance and renewal, secure key storage, modern protocol settings, and alerts before expiry. Redirecting HTTP to HTTPS improves consistency, while HTTP Strict Transport Security requires careful rollout because clients cache it.

TLS termination at the gateway does not automatically secure traffic from the gateway to backends. The threat model determines whether internal traffic also needs TLS, mutual TLS, or a service mesh. Network policies should restrict which workloads can reach backends and the control plane.

Rate limits protect finite capacity, but a per-source-IP rule may group many users behind one address or fail when a proxy obscures the client. The gateway must trust forwarded headers only from known upstream proxies. Authentication, request-size limits, timeouts, connection limits, and web application protections should align across every replica.

Routing configuration deserves the same review and deployment discipline as application code. A faulty wildcard host, path prefix, rewrite, or namespace attachment can expose the wrong service. Automated tests should verify positive routes, rejected routes, TLS names, redirects, headers, limits, and default-backend behaviour.
### Request flow through a gateway
A request for `https://gallery.example/api/image` first reaches the public load balancer and gateway listener on port 443. The gateway selects a certificate for `gallery.example`, completes TLS, and matches the host and `/api/image` path. A route then chooses the internal image API Service. The Service supplies an eligible Pod endpoint, and the gateway forwards the request with controlled forwarding headers and timeouts.

The gateway can return a cached public response when policy allows it. Otherwise, the Java API may return its own cached NASA data or contact the upstream service. The response travels back through the gateway, which records an access event and applies response headers. The browser never connects directly to the Java Pod, and the Pod does not publish a host port.

A request for the website root follows a different route to the Go Service. A request with an unknown host should reach no application route and receive a controlled rejection. This deny-by-default behaviour prevents accidental exposure when a new service appears in the cluster.

The same flow supports multiple replicas. The route names a Service rather than a Pod address, so Pod replacement does not alter the public rule. Readiness changes update endpoint eligibility. The gateway implementation watches the platform API or resolves the stable service name, then balances traffic among available backends. Deployments can scale without allocating a new public port for each replica.

Path rewriting requires precision. If the public route removes an `/api` prefix before forwarding, tests must cover `/api`, `/api/`, encoded paths, repeated slashes, query strings, and paths that should not match. A loose prefix rule can shadow another service or bypass an authentication policy. Hostnames should use exact or carefully reviewed wildcard rules.
### Production verification
Production checks should exercise the assembled system rather than isolated images alone. A staging deployment can use the production topology, security context, probe timings, log collector, and gateway policy with non-production credentials and controlled dependencies. Tests should confirm that configuration reaches each component with the intended precedence and that secret values never appear in rendered manifests, process output, or logs.

Failure injection can stop a process, block a dependency, delay startup, fill a log buffer, and remove a node. The expected platform action should follow each signal. Readiness should divert traffic without erasing useful state. Liveness should restart only an instance that cannot recover. A collector outage should not consume all node storage. A gateway update should preserve established traffic and reject invalid configuration.

Release tests should also observe scaling. New replicas must become ready before receiving load, and terminating replicas must leave endpoint sets before they stop. Requests should retain correlation identifiers across gateway and service boundaries. Cached responses should preserve privacy and freshness rules. Certificate selection should succeed for every public hostname, while unknown hosts receive no application content.

The verification record should capture evidence, including the image digest, manifest revision, configuration version, probe results, route tests, vulnerability status, and rollback outcome. Production approval then rests on a reproducible artefact and deployment model rather than on a successful local container run.

Teams should repeat these checks after changes to runtime versions, base images, cluster networking, storage classes, or gateway controllers. Platform upgrades can alter defaults, API behaviour, security policy, and failure timing even when application code stays unchanged. Regular rehearsal keeps operational assumptions aligned with the deployed environment and its dependencies.
## Production operating model
A production-ready container application combines the four capabilities into one controlled release model.
### Build and release
- The pipeline builds once and promotes an immutable digest.
- A multi-stage build keeps compilers, package managers, and temporary credentials out of the runtime image.
- The image runs as a non-root user where feasible and contains only required runtime components.
- Dependency, image, licence, and vulnerability checks run before promotion.
- Deployment records link the image, configuration, secrets, and routing revision.
- Rollback restores a compatible combination rather than only an older image.
### Runtime configuration and identity
- Non-sensitive defaults live with the image.
- Environment-specific values come from managed configuration.
- Secrets use dedicated secret mechanisms, encryption, least privilege, and rotation.
- Startup validation rejects invalid settings without printing confidential values.
- Service discovery uses stable platform names instead of replica IP addresses.
- Configuration changes follow an auditable rollout or a tested reload path.
### Observability
- Applications emit structured events to standard output and standard error.
- Log rotation and retention prevent local disk exhaustion.
- Central collectors add workload metadata and report delivery failures.
- Correlation identifiers connect logs, traces, and user-visible error responses.
- Metrics cover request rates, errors, latency, saturation, queue depth, and probe outcomes.
- Alerts identify user impact and sustained risk rather than every transient event.
### Resilience
- Startup probes protect slow initialisation.
- Readiness reflects the ability to accept new traffic.
- Liveness detects local deadlock or loss of progress without depending on remote public services.
- Probe thresholds match normal latency and recovery behaviour.
- The application handles termination signals and drains within its grace period.
- Persistent state survives container and Pod replacement.
- Replicas, disruption controls, and rollout settings preserve required capacity.
### Network delivery
- Only managed entry points publish public ports.
- Private services use isolated networks, ClusterIP Services, and policy controls.
- Gateways route by explicit host and path rules and reject unintended defaults.
- Trusted certificates and automated renewal protect public names.
- Cache, retry, timeout, and rate-limit policies account for idempotency and user identity.
- Docker API access and Kubernetes service accounts receive the minimum necessary authority.
- Gateway and controller components remain supported and receive security updates.

No single setting makes an application ready for production. Reliability emerges from consistent contracts between the application, image, runtime, and platform. Configuration defines intended behaviour, observability shows actual behaviour, probes provide bounded control signals, and routing connects only healthy services to users. Those contracts must remain secure, testable, and versioned throughout the release lifecycle.