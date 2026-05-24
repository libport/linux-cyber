# Microservices with Docker
Containerised microservices separate an application into independently packaged services that run together through shared networking, configuration and storage. The example application contains an ASP.NET Core API, a PostgreSQL database and an NGINX web front end. Docker Compose runs the local development environment, Docker Swarm demonstrates cluster orchestration, and Google Kubernetes Engine (GKE) shows a managed Kubernetes deployment path.
### Local orchestration with Docker Compose
`docker init` can generate container files for an ASP.NET Core project, including a Dockerfile and Compose configuration. After the API port is set to `9000`, `docker compose up` builds the image, pulls the required .NET SDK base image, creates a container and attaches the logs. The running API responds on `localhost:9000`.

The API returns an HTTP 500 error when a route depends on a missing database. Compose fixes the environment by adding a PostgreSQL service and giving the API a connection string that uses the database service name as the host. Compose creates a project network, so services can resolve each other by name without exposing every internal port to the host.

A robust Compose file defines:
- an API service built from the API source directory
- a PostgreSQL service using a tagged image, such as `postgres:17.2`
- environment variables for the database name, user and password
- a named volume for PostgreSQL data
- an initialisation script mounted into `/docker-entrypoint-initdb.d/`
- an NGINX web service built from the web directory and published on `localhost:8080`

PostgreSQL initialisation scripts run when the database starts for the first time with an empty data directory. The named volume preserves the data when containers are recreated. `docker compose down` removes the service containers and default network, but named volumes remain unless the command includes `--volumes`. Images remain unless the command includes an image removal option such as `--rmi local`.

The final local environment has three containers. The API image and NGINX image are built locally, while the PostgreSQL image is pulled from Docker Hub. `docker compose up -d` starts or recreates the environment in detached mode. After a teardown, another `docker compose up` recreates the application from the Compose file and reuses the database volume.
### Running services with Docker Swarm
Docker Swarm mode coordinates containers across multiple Docker hosts. A manager node creates the cluster with `docker swarm init`, and worker nodes join with the generated `docker swarm join` command. `docker node ls` confirms the participating nodes.

Swarm manages workloads as services. A service declares the desired state, and Swarm creates tasks that run containers on suitable nodes. A visualisation service can run only on a manager when it needs access to the Docker socket. Placement constraints express that requirement.

A simple echo service demonstrates scaling and load balancing. `docker service create` publishes a port, and `docker service scale echo=2` raises the replica count. Requests to the published port can reach different replicas because Swarm balances traffic across the service tasks. `docker service ps echo` shows task placement, including the node running each replica. Scaling a service to zero leaves the service definition in place but runs no tasks. `docker service rm echo` removes it.

Swarm recreates failed or removed tasks to match the desired replica count. When a task container is forcibly deleted, Swarm schedules a replacement for the same service slot. This behaviour gives services self-healing characteristics, although it does not remove the need for proper health checks, persistent storage and operational monitoring.
### Deploying a stack to Swarm
A Swarm stack gives a multi-service application a declarative deployment file. The stack file resembles a Compose file, but it adds production-oriented details such as registry image names, deploy settings, placement constraints and configs.

The API and web images need registry-ready names so they can be pushed and pulled by Swarm nodes. Docker configs can inject non-sensitive files, such as an SQL initialisation file, into services. Passwords and connection credentials should use secrets or another secure mechanism rather than plain environment variables in production.

A PostgreSQL placement constraint can pin the database service to a node that has the expected local volume. This avoids accidentally starting the database on another node with empty storage. It is not a high-availability design. Production data needs reliable persistent storage, backup and recovery planning.

`docker compose -f stack.yaml build` and `docker compose -f stack.yaml push` build and publish the local images. `docker stack deploy -c stack.yaml staging` deploys the application as a named stack. Swarm prefixes service names with the stack name, producing services such as `staging_api`, `staging_db` and `staging_web`.

`docker stack services staging` lists the services, and `docker stack ps staging` lists the tasks. Early web tasks can fail if NGINX starts before the API name resolves. Swarm retries according to the service restart behaviour. Logs from a failed task usually reveal the upstream resolution failure. Scaling `staging_web` to three replicas spreads the web front end across the cluster. Removing the stack deletes the stack services, networks and configs, while volume lifecycle depends on the volume type and where it resides.
### Deploying to GKE
Kubernetes is the production orchestration model in the example. GKE Autopilot creates a managed cluster where Google manages nodes and infrastructure capacity. After authentication with the Google Cloud CLI, `gcloud container clusters create-auto` creates the cluster, and `gcloud container clusters get-credentials` configures `kubectl` access.

A dedicated namespace, such as `library1`, separates the application resources. Kubernetes manifests replace the Compose and stack files. The database, API, web front end and ingress each have their own manifest.

The PostgreSQL manifest defines the container image, environment variables and service. A local `kubectl port-forward` allows `psql` to test the database before external access exists. The SQL initialisation script creates the `books` table and sample records. In production, PostgreSQL should use a PersistentVolumeClaim or a managed database service so pod recreation does not destroy data.

The API manifest uses the published API image, container port and database connection settings. The web manifest deploys the NGINX image and exposes port `80` through a Kubernetes Service. The Ingress routes `/api` requests to the API service and other paths to the web service. GKE can take time to provision the external IP address, and `kubectl describe ingress` shows provisioning events while it synchronises.

A PostgreSQL exporter sidecar can expose database metrics on port `9187` at `/metrics` for Prometheus-compatible scraping. The exporter reports scrape health and database metrics, including database size metrics such as `pg_database_size_bytes`. Metrics can show the size difference before and after schema creation, but meaningful production monitoring also needs durable storage, secure credentials, alert thresholds and a Prometheus or managed monitoring backend.
### Key operational lessons
- Compose suits local development and repeatable multi-container startup.
- Swarm demonstrates service scheduling, scaling and replacement across Docker nodes.
- Stack files make Swarm deployments reproducible and closer to production practice.
- Kubernetes and GKE provide a stronger production model for managed cluster operations.
- Stateful services need deliberate storage, backup, security and migration design in every orchestrator.