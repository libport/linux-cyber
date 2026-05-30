# Getting Started with Docker and AI
## Local AI deployment
Organisations increasingly deploy large language models and chatbots locally to protect data, reduce cloud dependency, control costs and meet container-first engineering standards. Docker Model Runner helps Docker teams run AI models locally without adding a separate model platform. It brings model pulling, caching, serving and application integration into the Docker workflow.

Docker Model Runner fits teams that already use Docker Desktop, Docker Engine, Docker Hub, Docker Compose and the Docker CLI. It does not replace every specialised model platform, but it reduces tool sprawl when local inference belongs beside containerised applications.
## What Docker Model Runner provides
Docker Model Runner manages and serves local AI models through Docker tooling. It pulls models from Docker Hub, OCI-compliant registries and supported Hugging Face repositories. It stores models locally, loads them when applications request inference and unloads them when they are idle.

It exposes compatible API formats so existing applications can talk to local models with minimal changes. Current Docker documentation describes OpenAI-compatible, Anthropic-compatible and Ollama-compatible APIs, plus Docker-native management endpoints. OpenAI-compatible clients should use the Docker Model Runner OpenAI base path under `/engines/v1`, for example `/engines/v1/chat/completions`.

The practical value is simple. A chatbot can keep its frontend and backend in containers while Docker Model Runner serves the model from the host. The application still behaves like a Docker application, but model execution can use host-side inference engines and hardware support.
## Why the model runs outside the container
AI models can run on CPUs, but many useful workloads need acceleration for acceptable latency and throughput. Container access to acceleration hardware differs across operating systems, drivers and devices. Docker Model Runner avoids putting all of that device support inside each model container. It runs the inference engine on the host and lets containerised applications call it over documented endpoints.

The inference engine performs the model execution. Docker Model Runner handles Docker integration, model lifecycle and API exposure. Current Docker documentation lists three supported engines:
- `llama.cpp` for efficient local development with quantised GGUF models
- `vLLM` for higher throughput serving with Safetensors models on supported Linux and NVIDIA GPU environments
- `Diffusers` for Stable Diffusion image generation with Safetensors models on supported environments

Support varies by platform and hardware. Teams should validate the engine, driver, model format and target device before treating a local setup as production-ready.
## Model storage and OCI artifacts
Docker treats models as managed artifacts. A `docker model pull` command downloads a model and stores it in the local model cache. At request time, Docker Model Runner loads the model into memory, runs inference through the selected engine and returns the response.

OCI registries matter because they already store much of the cloud-native supply chain. The same registry ecosystem that stores container images can also store adjacent artifacts such as Helm charts, SBOMs, signatures, policies and model files. Packaging models as OCI artifacts lets organisations reuse existing registry controls, access rules, metadata workflows and distribution practices rather than creating a new registry system only for models.

Docker Hub provides a curated AI model catalogue, but a verified publisher does not necessarily mean the model creator trained the model. Teams still need to assess model origin, licence, benchmark claims, safety properties and suitability for the task.
## Installing and enabling
Docker Model Runner can be enabled through Docker Desktop or installed with Docker Engine. On Docker Desktop, the setting now sits under the AI area of the settings interface. Host-side TCP support is optional and allows local host processes to reach the service on a configured port, commonly `12434`.

On Docker Engine, the `docker-model-plugin` package provides the CLI integration. After installation, `docker model version` checks the plugin and `docker model run ai/smollm2` provides a basic interactive test.

For Docker Desktop, containers can reach Docker Model Runner at `http://model-runner.docker.internal`. Host processes can use `http://localhost:12434` when TCP support is enabled. For Docker Engine on Linux, host processes can also use `http://localhost:12434`, while containers may need an explicit host gateway mapping before the same internal hostname works inside a Compose project.
## Pulling and testing models
Models can be pulled from the Docker Desktop Models interface or through the CLI. Model names usually include a repository, a model family and a tag that describes the variant. Variants often specify parameter count and quantisation.

Quantisation reduces the precision of model weights to reduce storage size and memory use. A 4-bit quantised model usually runs on smaller local hardware than the original higher-precision model, though quality and behaviour can change. GGUF is a common packaging format for models intended to run efficiently with `llama.cpp` on consumer and developer hardware.

The CLI command `docker model run` is useful for confirming that a model loads and responds. It should not be mistaken for a full chatbot experience. A real chat application needs conversation history, system instructions, context management, streaming and a user interface.

Direct API tests give a better integration check. A request to `/engines/v1/chat/completions` can name a local model, pass system and user messages, and receive a response through the OpenAI-compatible shape. Docker Model Runner can pull a missing model on demand, but production environments should usually pre-pull and test required models.
## Compose integration
Docker Compose can describe model dependencies alongside application services. Compose v2.38 and later supports a top-level `models` element for platforms that implement Compose models, including Docker Model Runner.

A Compose file can define frontend and backend services, attach them to a network and declare the model that the backend requires. The service-level `models` attribute grants the service access to the model, lets the platform provision it and injects environment variables that identify the model endpoint and name.

A typical local chatbot has three logical parts:
- A frontend service that exposes the chat interface, often on a host port such as `3000`
- A backend API service, for example a FastAPI service on an internal container network
- Docker Model Runner on the host, reached from containers through the documented model runner hostname or configured endpoint

The frontend sends prompts to the backend. The backend calls the model endpoint. Docker Model Runner loads the model if necessary, runs inference and streams the result back through the backend to the interface.

This design keeps prompts, responses and inference local after the model exists in the local cache. However, model pulls, registry requests and optional product telemetry may still reach external services unless administrators configure the environment for offline or air-gapped use.
## Open WebUI integration
Open WebUI provides a self-hosted chat interface that can connect to local model runners and OpenAI-compatible APIs. It offers a more complete experience than a custom test chatbot, including accounts, chat history, settings, prompt configuration and a familiar commercial-chat style interface.

A Compose deployment can run the Open WebUI container, persist `/app/backend/data` in a volume and point the application at Docker Model Runner. Docker documentation now shows an Ollama-compatible quick start by default and also documents OpenAI-compatible configuration. Either approach can work when the correct base URL and environment variables match the selected API mode.

Persistent storage matters. Without a volume, account data, conversation history, selected models and interface settings disappear when the container is removed. With a volume, the interface can survive Compose restarts and continue using the same local model configuration.

Open WebUI can list available local models, switch between them and save conversation context. The first prompt against a model can be slower because Docker Model Runner must load the model into memory. Later prompts are faster while the model remains loaded. Idle models unload to conserve resources.
## Operational guidance
Local models improve privacy and control, but they do not remove the need for governance. Teams should choose models based on licence, provenance, accuracy, language support, tool support, context length, memory footprint and latency. Benchmarks supplied by model publishers should be treated as starting points, not acceptance evidence.

Hardware sizing drives the user experience. Smaller quantised models run more easily, but they may produce weaker reasoning, shorter useful context or less reliable output. Larger models need more memory and stronger acceleration. Production use needs monitoring, repeatable deployment, model version control, access control, prompt and response logging policies, and a rollback path.

Compose makes the local stack repeatable. Application services, networks, volumes and model dependencies can live in one YAML definition and start with `docker compose up`. Cleanup can use `docker compose down`, with image and volume removal only when the operator intentionally wants to discard cached artefacts or persisted application data.
## Security and offline considerations
Local inference reduces exposure because prompts and responses do not need to leave the organisation after the model, application image and dependencies are present locally. It does not automatically create a secure environment. Administrators still need to control who can reach the model endpoint, which models developers can pull, which registries they can use and how application logs handle prompts and responses.

Model provenance matters as much as image provenance. A model may come from a verified publishing namespace while its weights originate from another organisation. The operator should check the creator, licence, training claims, supported languages, permitted uses and known safety limitations before deployment. A smaller model may be easier to run, but it may also fail more often on reasoning, coding or domain-specific tasks. A larger model may deliver better answers, but it can raise hardware cost, latency and operational complexity.

Air-gapped deployments need more preparation than a normal development machine. Required models, application images and base images should be pulled, scanned and approved before disconnection. Compose files should refer to approved tags or digests. Volumes should be treated as data stores, not disposable build artefacts, because they can contain user accounts, saved prompts, chat history and system configuration.
## Key takeaways
Docker Model Runner brings local model serving into the Docker ecosystem. It reduces the need for separate local LLM tools when an organisation already standardises on Docker. It runs inference through host-side engines, exposes compatible APIs and lets containerised applications call local models in a familiar way.

The main architectural pattern is clear. Applications remain containerised, while model execution runs through Docker Model Runner on the host. Models can be stored and distributed as OCI artifacts. Compose can declare model dependencies beside services. Open WebUI can provide a capable local chat interface on top of the same model runner.

The result is a practical local AI stack that supports privacy, cost control and developer familiarity, provided that teams validate hardware, model quality, licensing and operational controls before production use.