# Preparing Docker Applications for Production
Production containers need more than a working image. A container image must run consistently, accept environment-specific configuration, expose useful logs, report its health, wait for dependencies when necessary, and fit into a routing model that can handle traffic securely. These practices let a platform such as Docker Compose, Docker Swarm, Kubernetes, or a managed cloud service operate the application rather than merely start a black-box process.

A production-ready image keeps application binaries and dependencies stable while pushing environment-specific values into the runtime platform. The same image should move from development to test to production. Configuration, secrets, logging destinations, health policies, scaling rules, and routing rules should belong to the deployment model, not to a rebuilt image. This separation gives teams repeatable releases and reduces the risk that production runs code that has not passed earlier tests.

Container platforms treat containers through standard runtime interfaces and image formats. Docker remains a common tool for building and running images, but modern Kubernetes clusters run containers through Container Runtime Interface-compliant runtimes such as containerd or CRI-O. A production model should therefore describe portable container behaviour rather than depend on Docker Engine-specific assumptions.

A container is not a full virtual machine. It is an isolated process environment built from image layers, runtime settings, namespaces, filesystems, network configuration, environment variables, and, on Windows, registry settings. That environment gives applications practical integration points. Files can appear in known paths. Environment variables can supply simple values. Standard output and standard error can become container logs. Health commands and endpoints can expose state to the platform. Network names and reverse proxies can route traffic without embedding host addresses in application code.
## Distributed application example
A small distributed application makes these practices easier to see. A web front end can display NASA's Astronomy Picture of the Day. A background API can fetch image details from NASA and cache the result so repeated visits do not overload the external service. A separate access-log API can record requests. The components can use different languages, such as Go for the web front end, Java for the image API, and Node.js for the logging API.

The language mix matters because production patterns should not depend on one framework. Each component can read configuration, emit logs, report health, and receive traffic through the same platform contract even when the internal libraries differ. The Go service can use TOML configuration. The Node.js service can use JSON. The Java service can use properties files. The platform should not care which format the application uses internally, provided the image exposes predictable integration points.

Each component can run as a separate image and a separate service. The web front end calls the image API and the access-log API by service name. The APIs can run on private internal ports. Only the reverse proxy or ingress layer needs public exposure. This structure supports independent scaling, replacement, and troubleshooting. If the access-log API fails, the web component can show degraded logging behaviour without necessarily taking down the whole user-facing page. If the image API becomes unhealthy, the platform can restart it or remove it from traffic while other components continue to run.

The same example also shows why production preparation belongs early in the build. A team that adds configuration, logging, health, and routing only after the application reaches production will usually create emergency scripts and manual fixes. A team that builds those contracts into images and deployment models can move the same application across environments with fewer surprises.
## Configuration
Configuration should come from the container environment rather than from separate images for each environment. An image can include safe defaults, but runtime settings should override them. This allows one image to run with different behaviour in development, system test, integration, and production.

A good configuration hierarchy usually has three layers:
- Default settings packaged with the image
- Override files mounted into the container filesystem
- Environment variables that override file-based settings

The hierarchy lets an application start with sensible defaults, take environment-wide values from files, and accept quick overrides through environment variables. Many modern application frameworks already support this model. Go applications often use libraries such as Viper. Node.js applications can use packages such as node-config. .NET and Spring Boot also provide strong configuration support, although each framework uses its own conventions.

A production image should keep its defaults deliberately modest. For example, it can set a low logging level, default to non-sensitive endpoints, and use placeholder values for external dependencies. Runtime configuration can then raise the logging level, point to real services, enable or disable metrics, and identify the environment. This approach keeps the image portable and avoids rebuilding merely to change a setting.

Environment variables suit small values, feature flags, endpoint names, and settings that operators often override. Names should use an application-specific prefix when the framework scans the whole environment. Prefixes reduce accidental collisions with variables intended for the operating system, the container runtime, or another process.

Configuration files suit larger and structured settings. They can use the format the application already understands, such as TOML, JSON, YAML, XML, or properties files. A platform can mount a file or a whole directory into the container. Mounting a directory is often easier to maintain than mounting individual files, especially when configuration grows or when teams need separate files for several components.

Modern Docker supports bind mounts and volumes. A bind mount maps a host file or directory into a container path. Docker supports both `--volume` and `--mount`, but the explicit `--mount` syntax is generally clearer because it names the mount type, source, and target. Bind mounts work well for local development and demonstrations. Managed platform objects such as ConfigMaps, Secrets, and managed volumes are more suitable for production.

Docker Compose captures the same runtime configuration in a YAML model. A service can declare its image, environment variables, networks, ports, volumes, and dependencies in one file. Current Compose uses the Compose Specification, and the common command is `docker compose`, not the legacy `docker-compose` command. Compose is useful for local multi-container environments and small deployments, but production teams should still decide whether they need a scheduler, orchestrator, secret management, rolling updates, and stronger recovery behaviour.

Some older applications cannot merge settings from multiple sources. They may expect a single configuration file in a fixed path. Those applications can still fit the same runtime model by adding a small configuration loader to the image. The loader runs before the main application. It reads the default file, mounted override files, and selected environment variables. It then writes the final configuration file to the path the application already expects.

A loader utility should be simple and explicit. It should report which sources it used, fail fast when required files are invalid, and write only the values the application needs. The container startup command should run the loader first and start the application only if configuration succeeds. That pattern keeps legacy applications manageable without changing their core code.

A multi-stage Dockerfile can build both the main application and the loader utility. The final runtime image can contain the compiled application, the loader, safe defaults, and an entry command that prepares configuration before launch. This keeps the build repeatable and keeps configuration support versioned with the application.

Kubernetes applies the same principles with platform objects. ConfigMaps store non-confidential configuration as key-value pairs or files. Pods can consume ConfigMaps through environment variables, command-line arguments, or mounted files. Secrets store sensitive values, but they are not automatically secure merely because they are called Secrets. Kubernetes stores Secrets unencrypted in etcd by default unless the cluster enables encryption at rest, and access control must prevent unauthorised reading. Teams should use Secrets for sensitive values, enable encryption at rest, limit role-based access, and consider external secret managers for higher-risk systems.

The boundary between configuration and secrets should be deliberate. A log level, feature flag, cache size, or public service name belongs in configuration. A password, token, private key, certificate key, or API credential belongs in a secret store. Images should not contain either environment-specific configuration or secrets because image registries, build caches, and vulnerability scanners can expose image contents to more systems than intended. Even a deleted layer can remain recoverable from image history if a secret was added during a build.

Secret delivery should also match the application. Some applications can read secrets from files with restrictive permissions. Others expect environment variables. File-based secrets can reduce accidental exposure through process listings and diagnostic dumps, but both methods need careful logging discipline. Applications should never print their complete effective configuration when that output might include secret values. Diagnostic endpoints should redact or omit sensitive keys.

A Kubernetes Deployment usually creates Pods, and each Pod contains one or more containers. A Service provides a stable network endpoint for a set of Pods. Cluster DNS creates names for Services and Pods so workloads can use names rather than fragile Pod IP addresses. A ConfigMap or Secret can be mounted into a Pod as files or exposed as environment variables. Mounted ConfigMap data is read-only. Teams should understand update behaviour, especially when using `subPath`, because some mounts do not receive updates automatically.

Good configuration practice produces several production benefits:
- The same image moves through every environment.
- The platform owns environment-specific values.
- Operators can change configuration without changing code.
- Sensitive data stays outside the image.
- Application models become easier to review in source control.
- Rollbacks can reuse the same tested image with earlier settings.

The key discipline is to make configuration explicit. A container should start with clear defaults, accept known overrides, fail when required values are missing, and avoid hiding important behaviour in ad hoc scripts or manual commands.
### Configuration patterns by component
A Go component can read a default `config.toml` file from the image, then read a mounted override file from a known directory, then read environment variables with an application-specific prefix. The application code can ask for logical settings such as `environment`, `metrics.enabled`, or API URLs without knowing which source supplied the final value. The framework or library merges the values during startup.

A Node.js component can follow the same shape with JSON files. The default file can ship inside the image, and an environment-specific file can arrive through a mounted directory. Some Node configuration libraries use a single environment variable such as `NODE_CONFIG` to hold JSON overrides. That is a framework convention, not a general container rule. The deployment model should document it so operators understand how to override values safely.

A Java component that uses Spring Boot can already read many configuration sources, but older Java applications may require a fixed properties file. A loader utility can preserve the container contract. It can read a mounted `application.properties` file, scan environment variables with a prefix, merge the values, and write the final file before the main application starts. The application then sees the single properties file it expects, while the platform still uses normal environment variables and mounted files.

This component-level detail produces a consistent operator experience. Each service can accept a mounted configuration directory and selected environment variables. Each service can report its effective non-sensitive configuration through a protected diagnostic endpoint. Each service can keep secrets out of those diagnostics. Operators can then inspect behaviour without learning the internals of each language runtime.

Configuration should also avoid several common mistakes. Images should not contain production credentials. Environment variables should not hold large structured documents when a mounted file would be clearer. ConfigMaps should not store passwords or tokens. Secrets should not be printed in startup logs. Startup scripts should not silently overwrite invalid settings. A platform should fail visibly when configuration is wrong because a misconfigured but running container can be harder to diagnose than a container that exits with a clear error.
## Logging
Container logging should start with standard output and standard error. A platform can automatically capture those streams, attach metadata, and make the logs available through commands, APIs, or a central logging pipeline. Applications that write only to private files inside the container are harder to inspect, rotate, ship, and correlate.

An application should write normal event logs to standard output and errors to standard error when that separation is useful. The runtime can then collect logs without knowing the application framework. Docker logging drivers capture container output. Kubernetes expects containers to write to standard output and standard error, and a node-level agent commonly forwards those logs to central storage.

This model changes application design. Instead of configuring each component to write to a different local file path, each component should send meaningful, structured log lines to the console. A JSON log format is often easier to query than plain text because it can carry fields such as timestamp, level, component, request ID, trace ID, user session, route, status code, and latency. Even when plain text remains necessary, logs should contain enough context to support troubleshooting without opening a shell in the container.

The sample application pattern uses several components, including a web front end, an API that fetches data, and an access-log API. Each component should expose logs through the same container mechanism despite using different languages. The platform then sees a consistent stream from every container rather than unrelated framework-specific files.

Some applications or frameworks still write to file sinks. When the application cannot be changed easily, the image can include a relay process. A relay tails one or more application log files and writes the content to standard output or standard error. That keeps platform logging consistent while allowing older applications to keep their existing file-based behaviour.

A log relay should not become a hidden dependency that masks failure. It should start predictably, handle missing files where appropriate, and exit when its required log source cannot be read. The container command can start both the application and the relay, but teams should avoid complex shell logic that becomes difficult to test. If multiple long-running processes must share one container, the image needs clear process supervision or a deliberate sidecar pattern.

File-based logging also needs careful storage behaviour. Container filesystems are ephemeral and can disappear when a container restarts or moves. Local files can fill disks if the runtime does not rotate them. Central log collection reduces that risk, but it does not remove the need for retention policies, sampling decisions, redaction, and storage limits.

Centralised logging becomes essential once an application has several replicas or components. Without it, operators must identify which node and container handled a request before they can find the relevant logs. A logging stack collects records from all containers, stores them in a searchable system, and provides a user interface for filtering, dashboards, and incident analysis.

A traditional EFK stack uses Elasticsearch, Fluentd, and Kibana. Many current platforms use variations such as Fluent Bit, OpenSearch, OpenSearch Dashboards, Loki, or managed cloud logging. The exact product matters less than the architecture. A node-level collector should harvest container logs, enrich them with metadata, forward them to durable storage, and make them searchable by application, component, environment, container, Pod, namespace, level, request ID, and time range.

Kubernetes logging commonly uses a DaemonSet for a collector so that each node runs one logging agent. The agent reads container log files produced by the runtime and forwards entries to the chosen backend. This avoids changing every application. It also ensures that newly scheduled Pods receive logging automatically once the collector runs on the node.

Central logging must protect sensitive data. Logs often contain request parameters, headers, tokens, identifiers, and exception details. Applications should avoid logging secrets and personal information. Logging pipelines should support access control, retention limits, and redaction where necessary. Debug logging should be temporary and controlled because it can create large volumes and leak details that normal production logs should not expose.

Logging should connect with metrics and traces where possible. Logs explain what happened, metrics show rates and trends, and traces show how one request moved through services. A container platform does not need every application to use the same library, but it does need a shared correlation convention. A request ID from the proxy or web front end should travel to downstream APIs and appear in each log record. That convention turns separate component logs into a single story during incident response.

A useful production logging model has these properties:
- Applications log to standard output and standard error.
- Log lines carry enough context to correlate events across components.
- The platform collects logs without manual container access.
- The central store supports search, filtering, dashboards, and retention policies.
- Logging does not require separate application images per environment.
- Sensitive data is excluded or protected.

The main goal is operational visibility. A production platform should show what the application is doing, which version is running, which requests failed, and whether failures concentrate in one component, node, or dependency.
### Log structure and collection flow
A useful log line is more than a message. It identifies the source component, severity, time, build version, environment, request or correlation ID, and the event that occurred. For an HTTP service, it can record method, path, status, duration, and client category without exposing sensitive data. Consistent fields let teams query across components, for example to follow one request from the web front end to the API and then to the access-log service.

The collection flow should be predictable. The application writes to standard output or standard error. The container runtime captures those streams. Docker or the Kubernetes node stores the stream in a runtime-specific log location. A collector reads the node logs, adds metadata such as namespace, Pod, container, image, and labels, and forwards the records to central storage. Search and dashboards then sit on top of that store.

This architecture means applications do not need to know where central logging runs. A local Compose environment can use `docker logs` or a lightweight collector. A Kubernetes environment can use a DaemonSet collector. A managed cloud environment can forward logs to the provider's logging service. The image remains the same because the platform, not the application, owns the shipping mechanism.

Hidden log sinks deserve special attention during migration. Some web servers, libraries, and legacy frameworks write access logs, error logs, or audit records to files even when application logs go to the console. Those files can mislead operators because the platform appears to collect logs while important evidence stays inside the container. A production review should check every logging path and either redirect it to standard streams or deliberately collect it.

Central logging should support incident response. Operators need to filter by time, deployment, component, version, and severity. They need to search all replicas at once. They need enough retention to investigate delayed reports. They also need limits. Excessive debug logs can increase cost and slow searches. A good platform combines log levels, sampling where suitable, and retention policies that match operational and compliance needs.
## Health, readiness, and dependency checks
A platform can keep an application available only when the application tells the truth about its state. A running process is not always a healthy service. It may be deadlocked, unable to reach a database, stuck during startup, or serving errors while the process still exists. Health checks expose the difference between a process that exists and a component that can perform useful work.

Health checks usually answer several different questions:
- Has the process started?
- Can the application respond to a basic request?
- Is the application ready to receive traffic?
- Are critical dependencies available?
- Should the platform restart this container?

A simple Docker `HEALTHCHECK` instruction runs a command inside the container. The command returns an exit code. A zero exit code means healthy, a non-zero code means unhealthy, and Docker records that state. There can be only one `HEALTHCHECK` instruction in a Dockerfile, although the command can call a script that performs several checks.

A basic health check might use `curl` to request an HTTP endpoint. This works well for simple web services and clearly shows whether the application can answer within a timeout. The image must contain the tool used by the check. Minimal images often exclude `curl`, so teams may need a small custom probe utility or an endpoint that a platform can call from outside the container.

Health checks should be cheap, fast, and reliable. They should not perform heavy queries or depend on slow external systems unless the check is specifically about readiness. A liveness check that fails because a downstream service is temporarily unavailable can cause restart loops. Restarting a healthy application does not fix an external outage. It can increase load and make the incident worse.

Readiness checks answer a different question. They tell the platform whether the container should receive traffic now. A service can be alive but not ready because it is starting, warming a cache, applying migrations, connecting to dependencies, or draining work before shutdown. A readiness check should fail until the component can handle requests safely. The platform can then withhold traffic without killing the container.

Dependency checks are especially important for distributed applications. A web front end may need an API. An API may need a database, object store, identity provider, or message broker. Some dependencies must be present before the application starts. Others can be optional or degraded. The application should distinguish these cases. A required dependency can block readiness. An optional dependency might only change a feature flag or show a degraded state.

Legacy applications can use the same loader pattern for health and readiness. A small utility can call endpoints, open sockets, test files, or inspect processes. A startup wrapper can wait for dependencies before launching the main process. A health command can report whether the main application still responds. This approach is useful when the original application has no native health endpoint.

Docker Compose supports health checks in service definitions, and it can use health conditions to order startup in some dependency models. Teams should not confuse startup ordering with production self-healing. A standalone Docker Engine marks a container unhealthy, but it does not automatically restart it simply because the health check fails. Restart policies react to process exits. Orchestrators such as Kubernetes and Swarm can use health signals to restart, reschedule, or stop sending traffic when they are configured to do so.

Kubernetes uses probes rather than Dockerfile `HEALTHCHECK` instructions as the primary health model. The kubelet can run liveness, readiness, and startup probes. A liveness probe can restart a container when it fails. A readiness probe can remove a Pod from Service endpoints so it stops receiving traffic. A startup probe can protect slow-starting applications from premature liveness failures. Kubernetes can run probes through HTTP, TCP, gRPC, or commands, depending on the application and cluster version.

This difference matters. A Dockerfile health check can help local Docker workflows and Swarm-style systems, but Kubernetes probe definitions belong in the Pod specification. Production teams should define the checks in the deployment model that the platform actually uses.

A strong health model has these practices:
- Liveness checks identify unrecoverable application failure.
- Readiness checks prevent traffic from reaching unprepared containers.
- Startup checks allow slow applications to initialise safely.
- Dependency checks distinguish required and optional services.
- Probe timeouts and failure thresholds avoid false positives.
- Checks produce useful logs when they fail.
- The platform response matches the type of failure.

Self-healing relies on honest signals and sensible reactions. Restarting a broken process can restore service. Restarting an application because its database is briefly slow can create more damage. Production checks should measure the right state and give the platform enough information to act proportionately.
### Designing checks that do not cause harm
A health endpoint should return a clear status and complete quickly. It should avoid work that creates load during an outage. A liveness endpoint can check an in-memory flag, the main event loop, or a lightweight internal operation. It should not require every downstream dependency to be perfect. Its purpose is to tell the platform whether restarting this container is likely to help.

A readiness endpoint can be stricter. It can check that required dependencies are reachable, migrations have completed, caches have warmed, and the service can accept requests. When readiness fails, the platform should stop sending new traffic but allow the container to keep running. This is useful during deployments, rolling updates, dependency interruptions, and graceful shutdown.

A startup probe prevents premature liveness failures. Some applications need extra time to compile templates, run first-time setup, or build caches. Without a startup probe, a liveness probe might kill the container before the application has a fair chance to start. With a startup probe, Kubernetes can wait for initial readiness and then begin normal liveness monitoring.

Dependency checks need judgement. A service that cannot reach its database may be unready because it cannot serve requests. A service that cannot reach an analytics endpoint may still serve users and record events for later delivery. A service that cannot reach an external image API may return cached content while reporting degraded status. These distinctions help the platform and operators choose the right action.

Probe parameters also matter. Too short a timeout can turn normal latency spikes into restarts. Too many retries can leave broken containers in rotation. Too frequent a check can add needless traffic. Too slow a check can delay recovery. Teams should test probe settings under load, during dependency outages, and during rolling updates.

Health failures should be observable. The check command or endpoint should log enough information to show why it failed, without flooding logs on every probe. Dashboards should show probe failure counts, restart counts, readiness transitions, and deployment events together. This helps operators distinguish a bad release from a dependency outage, an overloaded node, or a routing problem.
## Routing and reverse proxies
Production applications need a clean traffic entry point. A container platform may run many services on the same hosts or cluster. Each service may have several replicas, internal ports, and deployment versions. Clients should not need to know where containers run. A reverse proxy or ingress layer should accept incoming traffic, apply external policies, and route each request to the right component.

A reverse proxy sits between clients and application containers. It can route by host name, path, header, or other request attributes. It can terminate Transport Layer Security, enforce redirects, add or remove headers, cache responses, compress content, and balance traffic across replicas. It can also provide one public endpoint for several internal services.

Nginx is a common reverse proxy. A basic containerised Nginx proxy can listen on a host port and forward requests to application containers by service name. In a Compose network, services can resolve each other by name. A proxy can route `/api` to one service, `/logs` to another, and static or front-end traffic to the web component. This keeps application containers focused on application logic and leaves external routing to infrastructure.

Static proxy configuration works when routes change rarely. The configuration file can be packaged with the proxy image or mounted at runtime. Runtime mounting is usually better because it avoids rebuilding the proxy image for every environment. The proxy configuration should be versioned with the application model, reviewed like other deployment code, and promoted through environments.

Dynamic reverse proxies reduce manual configuration. Traefik can watch Docker or Kubernetes metadata and configure routes from labels, annotations, or custom resources. A service can declare its routing requirements alongside its deployment definition. The proxy discovers the service and updates routes without a hand-written proxy file. This suits platforms where services appear, scale, and move frequently.

Different reverse proxies suit different environments. Nginx is mature, fast, widely understood, and strong for static configuration. Traefik integrates well with dynamic container environments. HAProxy, Envoy, Caddy, Kong, and cloud load balancers also fit many production designs. The choice should reflect operational skill, observability needs, security features, protocol support, certificate management, and the platform's native integration.

Routing should not hide service health. A proxy should send traffic only to healthy and ready backends. In Kubernetes, Services select ready Pods, and readiness probes influence endpoint membership. In other systems, the proxy or orchestrator may need explicit health-check configuration. A proxy that continues to route traffic to an unhealthy container can defeat the rest of the health model.

Kubernetes separates internal service discovery from external ingress. A Service exposes a stable endpoint for Pods inside the cluster and sometimes outside it, depending on the Service type. DNS records let workloads call Services by name. An Ingress object describes external HTTP or HTTPS routing rules. An Ingress does nothing on its own. The cluster must run an Ingress controller, such as ingress-nginx, Traefik, HAProxy, Kong, or a cloud provider controller, to implement the rules.

Modern Kubernetes Ingress resources use the `networking.k8s.io/v1` API. Older examples that use deprecated beta API versions should be updated. Ingress can route by host and path, and it can attach TLS configuration. Controller-specific annotations or resources often provide advanced behaviour such as rewrites, timeouts, rate limits, authentication, WebSocket support, and caching. Teams should document controller-specific features because they can reduce portability between clusters.

Transport Layer Security belongs in the routing design. A production platform should define where TLS terminates, how certificates renew, whether internal traffic also uses encryption, and how proxies forward client protocol information. Automatic certificate management can reduce risk, but the renewal process must be observable and tested.

Caching also belongs at the proxy layer when responses can safely be reused. A proxy can cache slow or expensive responses, reduce load on application containers, and improve latency. Cache rules must respect privacy, authentication, invalidation, and freshness. APIs that return user-specific or sensitive data should not be cached without explicit controls.

A strong routing model has these properties:
- External clients use stable host names and paths.
- Application containers avoid hard-coded public addresses.
- Internal services communicate through platform service discovery.
- The proxy or ingress controller routes only to ready backends.
- TLS, redirects, headers, and caching are defined centrally.
- Routing configuration lives in source control with the deployment model.
- Controller-specific behaviour is documented.

The result is a platform that can host several applications and components without exposing every container directly. Routing becomes an operational capability rather than scattered application code.
### Proxy configuration in practice
A static Nginx proxy can use an upstream block for each internal service and a server block for external routes. The proxy listens on a public port. It forwards requests to service names on the container network. In a Compose environment, those names come from services in the Compose file. In Kubernetes, the proxy or ingress controller targets Services rather than individual Pods. This gives the platform freedom to replace containers while clients keep using the same address.

Traefik takes a more dynamic approach. In Docker, labels can describe routers, services, entry points, and TLS settings. In Kubernetes, Traefik can watch Ingress resources or its own custom resources. The proxy updates its routing table as the platform changes. This reduces manual reloads and supports environments where teams deploy many services frequently.

Dynamic discovery is powerful, but it needs guardrails. Teams should control which containers or namespaces a proxy may watch. They should set safe defaults for exposure so an internal service does not become public by accident. They should review labels, annotations, and ingress resources as deployment code. They should also monitor proxy configuration because a wrong route can look like an application failure.

Ingress controllers vary in behaviour. The core Ingress resource defines common routing concepts, but annotations and advanced features are controller-specific. A rewrite rule for one controller may not work in another. A timeout, body-size limit, WebSocket setting, or certificate annotation may also differ. Production documentation should state which controller implements the rules and which features depend on that controller.

Routing also interacts with deployment strategy. During a rolling update, readiness checks should add new Pods only after they are ready. Old Pods should remain in service until replacements can handle traffic. During shutdown, applications should fail readiness first, finish in-flight requests, and then exit. The proxy, Service, and application must all support that sequence to avoid dropped requests.
## Application modelling
Production behaviour depends on both the image and the application model. The image contains the application, default configuration, startup commands, health utilities, and logging behaviour. The model describes how the platform should run the image. It includes configuration objects, secrets, environment variables, mounts, networks, Services, probes, replicas, routing rules, and resource settings.

The image and model should complement each other. The image should provide predictable integration points. The model should connect those points to the platform. For example, an image might read `/app/config/config.toml`, expose `/healthz` and `/readyz`, and write logs to standard output. The Kubernetes model can mount a ConfigMap at that path, define liveness and readiness probes for those endpoints, create a Service, and attach an Ingress rule.

This split also clarifies responsibilities. Developers usually understand the application code, default configuration, health endpoints, and logging format. Operations or platform teams usually understand cluster objects, secrets, probes, policies, routing, certificates, scaling, and monitoring. Containers blur the boundary, so both sides should agree on clear contracts.

A practical production contract should specify:
- Which environment variables the image accepts
- Which configuration file paths the image reads
- Which settings are safe defaults and which are required at runtime
- Which ports the application listens on
- Which endpoints report liveness and readiness
- Which logs appear on standard output and standard error
- Which dependencies block startup or readiness
- Which routes expose the component externally
- Which values are confidential and must come from Secrets

Source control should hold the application model. Environment-specific overlays can define values for development, test, and production while keeping the core structure consistent. Teams can use Compose files, Kubernetes manifests, Helm charts, Kustomize overlays, Jsonnet, CDK-style tools, or managed platform definitions. The tool matters less than reviewability, repeatability, and clear promotion between environments.

Container images should be immutable once built. Rebuilding an image for each environment undermines confidence because each environment then runs a different artefact. A better path builds once, scans once, tests once, and promotes the same digest through each environment while changing only platform-owned configuration and secrets.

The release process should make image identity visible. A mutable tag such as `latest` can hide which build runs in production. A versioned tag is better, and an immutable digest is stronger because it identifies the exact image content. Deployment records, logs, and dashboards should show the image version or digest so incident response can connect behaviour to a specific release.

Promotion should also cover configuration. A production failure can come from a bad image, a bad setting, a missing secret, a route change, or a probe change. Teams should review deployment-model changes with the same care as application code. A small change to a readiness probe or ingress rule can remove healthy containers from traffic or expose an internal endpoint. A rollback plan should therefore include both image rollback and configuration rollback.

Local development and production do not need identical platforms, but they need the same contract. Docker Compose can give developers a fast environment that exercises image defaults, mounted configuration, service names, logs, and health checks. Kubernetes can apply the same contracts with Deployments, Services, ConfigMaps, Secrets, probes, and Ingress. The bridge between the two is the container image interface. If the image reads the same paths, accepts the same variables, writes the same logs, and exposes the same ports, the surrounding platform can change without rewriting the application.

Operational documentation should focus on actions. It should tell a support engineer how to find the running image, view effective non-sensitive configuration, inspect logs, check readiness, identify dependencies, and trace a request through the proxy. It should also identify what not to do, such as rebuilding an image to change a URL, entering a production container to edit a file, or storing a token in a ConfigMap.
## Production checklist
A containerised application is ready for production when it satisfies the operational contract expected by its platform.

Configuration:
- The image contains safe defaults only.
- Runtime settings override image defaults through files and environment variables.
- The same image can run in every environment.
- Secrets stay out of images and ordinary ConfigMaps.
- Startup fails clearly when required configuration is missing or invalid.

Logging:
- The application writes useful logs to standard output and standard error.
- Log records include component, level, time, request context, and version where practical.
- File-only logs are relayed or replaced.
- A central system collects, stores, searches, and retains logs.
- Logs do not expose secrets or unnecessary personal information.

Health and readiness:
- Liveness checks detect failures that a restart can fix.
- Readiness checks stop traffic until the component can serve requests.
- Startup checks protect slow applications from premature restarts.
- Dependency checks distinguish required, optional, and degraded states.
- Kubernetes workloads define probes in the Pod specification.
- Standalone Docker workloads do not assume health checks restart containers automatically.

Routing:
- A reverse proxy or ingress layer owns external traffic.
- Services provide stable internal names and endpoints.
- TLS, redirects, headers, and caching are defined deliberately.
- Routes target ready backends only.
- Ingress resources have a matching controller.
- Deprecated Kubernetes API versions are updated.

Delivery:
- The deployment model lives in source control.
- Image tags or digests identify exactly what is running.
- Environment promotion changes configuration, not binaries.
- Rollback procedures include both image and configuration changes.
- Operational dashboards show logs, health, traffic, and failure rates.

Security and reliability should reinforce the same contract. An image should contain only what the application needs at runtime. Smaller images reduce attack surface and usually make deployments faster. Runtime containers should not need shell access for normal support tasks. If operators must enter a container to diagnose a routine issue, the image or platform model probably lacks an observable signal.

The platform should also provide enough metadata to make operations traceable. Labels and annotations can identify the application, component, owner, environment, version, source repository, and support contact. Log collectors, metrics systems, and ingress controllers can use that metadata to organise data. Consistent metadata also makes clean-up safer because teams can identify which objects belong together.

Resource settings complete the model even though they sit outside the original application code. CPU and memory requests help schedulers place workloads. Limits prevent one component from exhausting shared capacity. Readiness checks, resource settings, and scaling rules should align. A component that becomes slow under load may need more replicas, higher resource requests, back-pressure, or a better readiness policy.

The final test is failure behaviour. A prepared containerised application should keep useful logs when a dependency fails, report unready when it cannot serve, avoid leaking secrets during diagnostics, recover from process failure, and return to service through the normal deployment model. It should not require a special rebuild, a manual file edit inside a container, or a private command known only to one engineer.

Production containers work best when images are portable, platform models are explicit, and applications expose the signals the platform needs. Configuration, logging, health, and routing are not add-ons. They form the contract that lets a container platform run applications reliably.
