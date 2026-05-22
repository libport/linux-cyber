# Deploying a Containerized Application with Docker
## Prepare the application before containerisation
Containerisation works best when the application already runs correctly outside Docker. A developer should first confirm the complete local workflow: clone the repository, install Python, install dependencies, compile assets where required, then start the application.

For the Flask example, the source contains `app/main.py`, `requirements.txt`, static assets and templates. A local virtual environment isolates the Python packages during development. After activating the environment, `pip install -r requirements.txt` installs the dependencies, and `python app/main.py` starts the web server. Successful local execution proves that later Docker errors relate to the image, container configuration or runtime environment rather than the application itself.
## Build the image
A Dockerfile turns the local workflow into repeatable image build steps. The base image supplies Python and `pip`, while later instructions set a working directory, copy dependency files, install dependencies, copy source code and define the command that starts the application.

```Dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 3000
CMD ["python", "app/main.py"]
```

A container runs as an isolated process with its own filesystem, networking and process tree, so a separate virtual environment is usually unnecessary inside this simple image. Copying `requirements.txt` before the rest of the source also improves build caching. Docker can reuse the dependency layer when application files change but dependencies stay the same. Image layers hold filesystem changes and image configuration. Later Dockerfile instructions can override defaults from the base image, so the Flask app replaces the Python image's generic startup command.

`EXPOSE 3000` records port metadata only. It does not publish the port on the host. The `docker container run` command must still map a host port to the container port, such as `-p 3001:3000`.
## Run and manage containers
A developer builds the image from the directory that contains the Dockerfile:

```bash
docker image build -t flask-web .
```

The image can then start a container:

```bash
docker container run --rm -p 3001:3000 flask-web
```

The browser should use the host port, such as `localhost:3001`, not the internal container address printed by the Flask server. When the container runs in the foreground, `Ctrl+C` stops it. With `--rm`, Docker removes the stopped container automatically. Without `--rm`, the stopped container remains available for `docker container start` until a developer removes it.

Detached containers run in the background with a stable name:

```bash
docker container run --name web1 --detach -p 3001:3000 flask-web
```

`docker container ps` lists running containers. `docker container stop web1` stops one, and `docker container rm web1` removes a stopped container. A running container must stop before removal unless `docker container rm -f web1` force removes it.
## Use alternative Python packaging with uv
The same application can use the uv package manager instead of `pip`. The image then needs a base image that includes Python and the `uv` command, often from GitHub Container Registry rather than Docker Hub. Instead of `requirements.txt`, uv uses `pyproject.toml` and `uv.lock`. The Dockerfile copies those files, runs `uv sync`, copies the application source and starts the app with `uv run app/main.py`.

This pattern keeps the Dockerfile structure the same while adapting dependency installation and the runtime command to the chosen Python tooling.
## Share images through registries
Docker images usually start from a registry image such as `python:3.12-slim`. A registry stores repositories, and repositories store tagged images. Docker Hub hosts the official Python image. GitHub Container Registry can host images such as uv.

A developer can tag and push an application image to a registry:

```bash
docker image tag flask-web username/flask-web:latest
docker login
docker image push username/flask-web:latest
```

Tagging creates another name for the same image. It does not upload anything. `docker image push` uploads the image layers after authentication. When the local image is missing, `docker image pull username/flask-web:latest` downloads the layers again and makes the image available for new containers.

Version tags distinguish application releases:

```bash
docker image build -t username/flask-web:v2 .
docker image push username/flask-web:v2
```

Tags such as `v2`, `v3` and `v3.0.1` make deployments explicit. The default registry is Docker Hub, using the implicit `docker.io` prefix. A fully qualified name such as `ghcr.io/username/flask-web:v2` targets GitHub Container Registry instead.
## Update containers safely
A running container uses the image selected when that container was created. Building `v3` does not update an existing `web1` container that started from `v2`. The developer must stop and remove the old container, then create a new one from the desired tag.

```bash
docker container rm -f web1
docker container run --name web1 --detach -p 3001:3000 username/flask-web:v3
```

This lifecycle keeps the image immutable. Docker replaces containers rather than patching them in place.
## Diagnose application and runtime problems
`docker container logs web1` shows the application output for a named container. In the Flask example, `/adder/1/2` failed because the view returned a float. Flask expects a valid response type such as a string, list, dict, tuple, response object or WSGI callable. Converting the calculated float to a string fixes the endpoint.

Developers should avoid rebuilding an image for every small code change during normal application development. Local Python execution, a virtual environment and the framework's development tools usually provide a faster feedback loop. Docker images become most useful once the application is ready to package, test as a deployment artefact or run in a production-like environment.
## Configure environment variables
Environment variables change runtime behaviour without changing application code. Flask debug mode uses `FLASK_DEBUG=true`. Debug mode can show stack traces and reload code during development, but it must remain off in production.

A container can receive a variable at startup:

```bash
docker container run --rm -p 3001:3000 -e FLASK_DEBUG=true username/flask-web:v3
```

A Dockerfile can also set a default:

```Dockerfile
ENV FLASK_DEBUG=true
```

Runtime values override Dockerfile defaults. `docker image inspect username/flask-web:v3.1` displays the image configuration, including exposed ports, environment variables and the default command.
## Monitor container performance
Docker's help output lists useful subcommands for containers and images. `docker container stats` shows live CPU, memory, network and block I/O data for running containers. A load tool such as ApacheBench can generate traffic:

```bash
ab -n 10000 -c 4 http://localhost:3000/
```

`-n` sets the number of requests, and `-c` sets concurrency. Rising CPU and memory figures help reveal available capacity, bottlenecks and the effect of concurrent load.