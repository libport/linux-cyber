# Microservices with Docker
> [!NOTE]
> This guide shows how to build, connect, scale, and deploy a containerized microservices application across Docker Compose, Docker Swarm, and Google Kubernetes Engine while addressing networking, storage, resilience, and monitoring.
## Architecture
A microservices architecture divides a system into small, autonomous services aligned with business capabilities. Each service owns a clear boundary. Teams can build, deploy, scale, and revise each service independently. Running a web front end, an API, and PostgreSQL in separate containers creates a multi-container application, but container count alone does not establish microservice boundaries. Independent ownership, deployment, and data responsibilities provide that distinction.

Containers package each component consistently. Docker Compose coordinates local development, Docker Swarm orchestrates Docker Engine clusters, and Kubernetes manages workloads on platforms such as Google Kubernetes Engine (GKE).
## Local development with Docker Compose
`docker init` can generate a Dockerfile, `.dockerignore`, and Compose configuration for a detected application such as ASP.NET Core. Teams should review generated files, pin supported base images, remove unnecessary build content, and run application processes as non-root users where feasible.

Compose defines services, networks, volumes, configurations, and secrets in YAML. A catalogue application can use three services:

| Service | Role | Deployment requirement |
| --- | --- | --- |
| `web` | Serves static files through NGINX | Builds from the web directory and publishes the public port |
| `api` | Runs the catalogue API | Builds from source and connects to PostgreSQL through `db:5432` |
| `db` | Stores catalogue data | Uses a maintained PostgreSQL image and a named data volume |

Compose creates a default network and registers each service name in internal DNS. Services should connect by name, not by changing container IP addresses. Only components that require host access need published ports. The database normally remains internal.

PostgreSQL credentials should not appear as fixed values in the Compose file. Development environments can use restricted local secrets or an untracked environment file, while production systems should use an orchestrator or external secret store. The PostgreSQL image runs scripts under `/docker-entrypoint-initdb.d` only when it initialises an empty data directory. Schema evolution after initialisation requires an idempotent migration process.

A named volume preserves database files when Compose recreates or removes containers. `docker compose down` removes the project's containers and networks by default, but retains named volumes and images. `--volumes` removes named volumes, and `--rmi` removes images, so destructive options require deliberate use.

Startup order does not prove readiness. Compose starts a dependency before its consumer, but the database might not yet accept connections. A database health check combined with `depends_on` and `condition: service_healthy` can gate startup. APIs and web proxies should still retry transient failures and terminate cleanly.

| Command | Effect |
| --- | --- |
| `docker compose up -d --build` | Builds changed images and starts the application in the background |
| `docker compose ps` | Shows service containers and their state |
| `docker compose logs -f api` | Streams API logs |
| `docker compose down` | Removes service containers and the default network |
## Cluster orchestration with Docker Swarm
`docker swarm init --advertise-addr <manager-ip>` creates a swarm manager with a stable advertised address. Worker nodes join with a generated token, which operators must protect and rotate if exposed. A single manager suits a lab, but production requires an odd manager quorum, commonly three or five managers distributed across failure domains.

A Swarm service declares an image, replica count, ports, placement rules, update behaviour, and resource constraints. Swarm schedules each replica as a task, replaces failed tasks, and maintains the requested count. Scaling a replicated service changes that count. Published ports use the routing mesh, which accepts traffic on swarm nodes and routes requests to service tasks.

Useful commands include `docker service ls`, `docker service ps <service>`, `docker service logs <service>`, `docker service scale web=3`, and `docker service rm <service>`. A visualiser that mounts the Docker socket receives powerful host control and belongs only in a tightly controlled demonstration environment.

A stack groups services, networks, volumes, configs, and secrets. `docker stack deploy -c stack.yaml staging` runs only on a manager. It does not build images and ignores `build`, so a pipeline must build, test, tag, and push images to a registry first. Stack deployment still uses the legacy Compose version 3 format rather than the complete current Compose Specification.

Docker configs distribute non-sensitive files such as NGINX or initialisation configuration. Swarm does not encrypt configs at rest. Docker secrets protect passwords and keys, and only authorised service tasks can access them. A placement constraint that binds PostgreSQL to one node can reconnect it to a node-local volume, but it also creates a failure dependency. Production databases need durable shared storage, tested backups, recovery procedures, or a managed database service.
## Production deployment on GKE
GKE Autopilot manages cluster nodes and infrastructure. Current commands should state the location and project explicitly:

```sh
gcloud container clusters create-auto prod --location=REGION --release-channel=regular --project=PROJECT_ID
gcloud container clusters get-credentials prod --location=REGION --project=PROJECT_ID
```

A namespace isolates the catalogue's namespaced resources. Deployments manage stateless API and web replicas, Services provide stable internal discovery, and an Ingress or Gateway implementation routes external HTTP and HTTPS traffic. An Ingress resource has no effect without a controller. Production entry points need DNS, trusted TLS, controlled exposure, and provider-specific configuration.

PostgreSQL must not rely on a Pod's writable filesystem. A production design should use a managed PostgreSQL service or a supported operator with PersistentVolumeClaims, high availability, backups, and restore testing. A migration Job or application migration tool should apply versioned schema changes instead of manual `psql` commands.

Kubernetes Secrets separate credentials from ordinary workload configuration, but base64 encoding does not encrypt them. Clusters should enable encryption at rest, apply least-privilege RBAC, restrict each Secret to the containers that need it, and prefer short-lived credentials or an external secret provider. Workloads also need readiness, liveness, and startup probes, plus realistic CPU and memory requests and limits. A HorizontalPodAutoscaler can then scale stateless Deployments from relevant resource or application metrics.

`kubectl apply -f` creates or updates declared resources. Operators can diagnose deployments with `kubectl get pods`, `kubectl get services`, `kubectl get ingress`, `kubectl describe`, and `kubectl logs`. `kubectl get all` covers only a common resource set, not every object in a namespace. `kubectl port-forward` supports temporary local inspection and does not provide production exposure.

A PostgreSQL exporter can run beside the database and expose Prometheus metrics on port 9187. It should use a dedicated, least-privilege database account and read its password from a mounted secret file. Prometheus should scrape the endpoint through cluster networking. `pg_exporter_last_scrape_error` reports scrape failure, while `pg_database_size_bytes` reports database size. Dashboards, alerts, retention, and trend analysis provide stronger operations than manual snapshots.

Deleting manifests removes declared workloads but can leave external resources or persistent data, depending on their policies. Deleting the GKE cluster stops cluster charges, while registries, disks, load balancers, databases, and reserved addresses may require separate cleanup.