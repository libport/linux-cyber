# Deploying a Containerized Application with Docker
> [!NOTE]
> This guide explains how to build, run, share, update, configure, troubleshoot, and monitor a containerized Python application using Docker.
## Validate the application
Containerisation starts with a working application. A clean checkout should restore its dependencies, load templates and static assets, and start through a documented entry point. A pip-based Flask project can verify those steps in an isolated local environment:

```bash
python -m venv .venv
. .venv/bin/activate
python -m pip install -r requirements.txt
python app/main.py
```

Fish and Windows shells use different activation commands. The repository should include dependency definitions and exclude `.venv`, caches, credentials, version-control data, and other unnecessary files through `.dockerignore`.

The local check should confirm the listening port, required environment, static-file paths, and response behaviour. Automated tests should exercise the application before containerisation so build failures remain distinct from application defects.

Flask's built-in server and debugger support local development only. A deployed image should run a production WSGI server such as Gunicorn. The server must listen on `0.0.0.0` inside the container so Docker can forward traffic to it.
## Build the image
A Dockerfile converts the verified procedure into repeatable build instructions. A production-oriented Dockerfile can assume that `requirements.txt` includes Gunicorn and that `app.main` exposes a Flask object named `app`:

```dockerfile
FROM python:3.12-slim

WORKDIR /app
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

COPY requirements.txt .
RUN python -m pip install --no-cache-dir -r requirements.txt

COPY app ./app
RUN useradd --create-home appuser
USER appuser

EXPOSE 3000
CMD ["gunicorn", "--bind=0.0.0.0:3000", "--access-logfile=-", "app.main:app"]
```

The versioned base tag selects the Python runtime and operating-system variant. A digest can pin the exact base image for stronger reproducibility, although maintainers must update that digest to receive security fixes. `WORKDIR` fixes the in-image path. Copying the dependency file before the source preserves the dependency layer when only application code changes. The final process runs as an unprivileged user.

The build context should contain only required inputs. Excluding local environments, test artefacts, editor files, and repository history reduces transfer time, prevents accidental inclusion, and makes cache invalidation easier to understand.

A container already isolates its filesystem and process environment, so a pip-based image does not normally need a second virtual environment. `EXPOSE` records intended port metadata but does not publish a port.

```bash
docker build -t flask-web:1.0.0 .
docker run --rm --name web -p 127.0.0.1:3001:3000 flask-web:1.0.0
```

The final dot supplies the current directory as the build context. The port mapping sends host requests on `127.0.0.1:3001` to port `3000` in the container. Omitting the host address normally publishes the port on every host interface, which can expose the service beyond the local machine. `--rm` removes the container after it stops. A web service does not require interactive `-i` or `-t` flags.
## Use uv as an alternative
An uv-managed project records metadata in `pyproject.toml` and resolved versions in `uv.lock`. An image can copy the uv binaries from Astral's official image into a versioned Python base, copy the dependency files before the application source, and run `uv sync --locked` to reject an outdated lockfile. `uv run --locked` then starts the application in the synchronised project environment. Pinning the uv image version or digest avoids an uncontrolled toolchain change.
## Publish and retrieve images
A registry stores image content, while a repository groups related images. A tag provides a human-readable reference to an image, and a digest identifies immutable content. Ordinary tags, including `latest`, can move to new content, so deployments should use explicit version tags or digests.

The `latest` tag has no automatic relationship to creation time or semantic versioning. Docker applies it only when an image reference omits a tag. Release automation should assign deliberate tags and record the pushed digest.

Docker Hub expects an account or organisation namespace:

```bash
docker tag flask-web:1.0.0 ACCOUNT/flask-web:1.0.0
docker login
docker push ACCOUNT/flask-web:1.0.0
docker pull ACCOUNT/flask-web:1.0.0
```

Tagging adds another reference to the same local image. Pushing uploads the image configuration and any layers that the registry does not already hold. Pulling downloads missing layers for the selected manifest. GitHub Container Registry uses a fully qualified reference such as `ghcr.io/OWNER/flask-web:1.0.0` and requires authentication to `ghcr.io` for private content or publishing.

Image layers primarily capture filesystem changes. A separate configuration object records settings such as environment variables, the startup command, and exposed ports. `docker image inspect IMAGE` displays this configuration. Build steps, image metadata, and registry history can reveal embedded values, so build arguments and `ENV` must not carry passwords, tokens, or private keys.
## Manage the container lifecycle
A detached container returns control to the shell and keeps running in the background:

```bash
docker run -d --name web1 -p 127.0.0.1:3001:3000 flask-web:1.0.0
```

| Command | Result |
| --- | --- |
| `docker ps` | Lists running containers |
| `docker ps -a` | Includes stopped containers |
| `docker logs -f web1` | Follows standard output and standard error |
| `docker stats web1` | Streams CPU, memory, network, storage, and process metrics |
| `docker stop web1` | Requests a graceful stop before forced termination |
| `docker start web1` | Restarts the existing container with its original image and settings |
| `docker rm web1` | Removes a stopped container |

An existing container cannot switch to a newly built image. An update builds and tests a new version, stops the old container, removes it, and creates a replacement from the new tag. `docker rm -f` kills a running container and should remain an exceptional recovery action. Compose or an orchestrator can automate replacement, health checks, rollback, and low-interruption releases.

A stable container name simplifies logs and lifecycle commands, but the name does not preserve application data. Deployments should place durable state in managed services or volumes before replacing a container.
## Diagnose and configure the application
Applications should write operational output to standard output and standard error so `docker logs` can retrieve it. A Flask route must return a supported response type. When a traceback reports that a view returned a float, the application should return text, JSON, or a proper response object, then run an automated test and build a new image.

Runtime options can override image defaults. The following local-only command replaces the production `CMD` with Flask's development server and enables debug mode through the Flask CLI:

```bash
docker run --rm -e FLASK_DEBUG=1 -p 127.0.0.1:3001:3000 flask-web:1.0.0 \
  flask --app app.main run --host=0.0.0.0 --port=3000
```

Debug mode belongs only in local development because Flask's interactive debugger permits browser-based code execution. A production image should keep debug mode off rather than setting `ENV FLASK_DEBUG=1`. Environment variables remain visible through inspection and process interfaces, so sensitive values require a secrets mechanism.

Fast development can run Python in a local virtual environment or synchronise source into a development container with Compose Watch. The release workflow should still build, test, scan, and publish an immutable image from committed inputs.

`docker stats` provides a live resource snapshot, not a capacity verdict. A load test should also record request rate, latency, error rate, warm-up behaviour, and resource limits under production-like conditions. CPU usage can exceed 100 percent on a multi-core host. A short ApacheBench run can expose obvious constraints, but it cannot establish production capacity by itself.

Explicit CPU and memory limits make tests easier to compare and protect neighbouring workloads. Health checks should test service readiness separately from raw process and resource metrics.