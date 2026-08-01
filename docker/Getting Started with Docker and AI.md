# Getting Started with Docker and AI
> [!NOTE]
> This guide explains how Docker Model Runner enables private, container-integrated local AI deployment while highlighting model management, application integration, hardware, security, and operational considerations.
## Local inference with Docker Model Runner
Docker Model Runner (DMR) lets teams manage and serve local generative AI models through familiar Docker workflows. It integrates with Docker Desktop, Docker Engine, Docker Hub, OCI registries, the Docker CLI, and Docker Compose. Applications can call its compatible APIs without adopting a separate model-management stack.

Local inference can reduce cloud expenditure, latency, and external transfer of prompts or proprietary data. It does not guarantee privacy or security. Model downloads, Docker Engine metadata requests, application integrations, updates, and telemetry settings can still create network traffic. Operators must inspect the entire system before describing it as offline or air-gapped.

Local deployment also shifts responsibility. The organisation supplies accelerator capacity, memory, storage, power, patching, monitoring, backups, and support. A small stable workload may cost less locally, while a large or highly variable workload may benefit from cloud elasticity. Model quality also varies. A compact local model can serve classification, summarisation, retrieval, or narrow chat tasks well, yet perform below a larger hosted model on complex reasoning. A sound decision compares data-residency requirements, workload shape, model quality, total cost, and operational capability.

DMR supports language, embedding, and image-generation workloads. Its APIs include OpenAI, Anthropic, and Ollama-compatible formats, as well as native model-management endpoints. Compatibility does not ensure that every client feature will work because models differ in tool use, structured output, context capacity, and other capabilities.
## Architecture and hardware
DMR pulls models from Docker Hub, another OCI-compliant registry, or Hugging Face, stores them locally, and loads them when an inference request arrives. An inference engine executes each model and uses compatible CPU or accelerator hardware. The application sends prompts to DMR and receives generated output through an HTTP API.

DMR separates model distribution from inference. The Docker CLI and API manage model artefacts, while the selected engine converts prompts into tokens, performs inference, and generates tokens for the response. This separation lets Docker retain a consistent management surface while engines optimise for different formats and hardware. It also means that an engine, model, and device must form a supported combination. An OCI artefact alone does not make every model executable on every host.

The execution boundary depends on the operating system. On Linux, DMR and its inference engines run in a container. On macOS and Windows, the engines run outside containers in operating-system sandboxes. Containerised applications can still call DMR through a Docker-provided endpoint.

DMR currently supports three inference engines:

| Engine | Primary model format | Typical use | Hardware |
| --- | --- | --- | --- |
| llama.cpp | GGUF | Efficient local inference with quantised models | CPU, NVIDIA, AMD, Apple Silicon, and Vulkan-compatible devices |
| vLLM | Safetensors and Hugging Face models | High-throughput language-model serving | NVIDIA CUDA on supported Linux or Windows environments |
| Diffusers | DDUF | Text-to-image generation | NVIDIA CUDA on supported Linux systems |

Small quantised models can run effectively on a CPU, although a supported GPU usually improves latency and throughput. Model size, quantisation, context length, concurrency, memory bandwidth, and available RAM or VRAM determine practical performance. Hardware tests should use representative prompts and workloads.
## Installation and initial use
Administrators should use a current, supported Docker release and follow the installation instructions for the host platform. Docker Desktop exposes DMR under Settings > AI. Host-side TCP access remains optional and should stay disabled unless a trusted host process needs it. Docker Engine users install the current `docker-model-plugin` package from Docker's repository.

The CLI confirms the service status, pulls a model, lists local models, and runs an interactive smoke test:

```shell
docker model status
docker model pull ai/smollm2:360M-Q4_K_M
docker model list
docker model run ai/smollm2:360M-Q4_K_M
```

`docker model run` provides a useful loading and response check. An application normally calls an API instead. DMR loads a pulled model on demand, so the first request can take longer than later requests. Memory pressure, engine configuration, and idle-unloading behaviour can alter subsequent latency.
## Model selection and distribution
A model tag often records parameter count, quantisation, or an engine variant. GGUF quantisation reduces numerical precision and usually lowers memory use. Aggressive quantisation can also reduce output quality, so teams should compare accuracy, speed, memory use, licence conditions, and task-specific performance.

Parameter count offers only a rough guide to resource demand. The runtime must also hold the context, intermediate state, and generated tokens. Long conversations can therefore exhaust memory even when model weights fit. Applications should cap prompt size, reserve space for output, summarise or retrieve older context when appropriate, and reject oversized requests cleanly. Context capacity does not prove that a model will use every distant token reliably.

OCI packaging allows existing registries, access controls, and distribution workflows to carry model artefacts. Registry tags can move even though manifests and blobs use content-addressed digests. Production deployments should pin a reviewed digest where the platform permits it and should record the model's publisher, licence, provenance, model card, intended use, and evaluation results. A verified publisher establishes publishing identity, not model safety or accuracy.

Model output can contain factual errors, insecure code, harmful content, or fabricated sources. Teams must validate consequential output and must not grant a model unsupervised authority over sensitive tools or data.
## API access
Host applications can reach an enabled TCP endpoint at `http://localhost:12434`. OpenAI-compatible clients use the base URL `http://localhost:12434/engines/v1`. Containers on Docker Desktop use `http://model-runner.docker.internal`, while Docker Engine deployments may need a `host-gateway` mapping. Platform documentation should govern the final address.

An OpenAI-compatible chat request uses the full model identifier:

```shell
curl http://localhost:12434/engines/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "ai/smollm2:360M-Q4_K_M",
    "messages": [
      {"role": "user", "content": "Explain container images briefly."}
    ]
  }'
```

DMR does not require an API key, although some clients require a placeholder value. The absence of authentication creates a significant boundary. Any client that can reach the API can submit prompts and can pull, load, or run models. Administrators should bind TCP access only to trusted interfaces, restrict networks and firewalls, limit CORS origins, and place authenticated TLS termination in front of remote access.

Chat-completion requests remain stateless unless the client resends prior messages or maintains conversation state elsewhere. A user interface may appear conversational because it stores and includes that history. The model itself does not retain earlier exchanges through a standard stateless request.

A three-tier chat application normally places a browser interface in front of an application backend, which then calls DMR. The backend should authenticate users, validate request size and content, enforce quotas, construct the system instruction, select approved models, and record only necessary telemetry. Keeping DMR behind the backend prevents browsers from receiving unrestricted access to its unauthenticated management and inference surface. The backend can also trim conversation history to fit the context budget and can return a controlled error when inference fails.
## Compose integration
Docker Compose 2.38 or later can declare models as first-class application dependencies on a platform that implements the Compose models specification. A service references a top-level model and receives the endpoint and model identifier as environment variables:

```yaml
services:
  backend:
    image: example/chat-backend:1.0
    models:
      local_llm:
        endpoint_var: MODEL_ENDPOINT
        model_var: MODEL_NAME

models:
  local_llm:
    model: ai/smollm2:360M-Q4_K_M
    context_size: 4096
```

Compose asks the supporting platform to provision the model and make it available to the service. `context_size` sets the maximum token context, including input and generated output, and larger values consume more memory. Engine-specific `runtime_flags` allow further tuning but reduce portability and require validation against the selected engine.

Plain `depends_on` controls service start order, not readiness. A backend or user interface that requires another service to accept requests should use a healthcheck with `condition: service_healthy`, or should implement bounded retries and backoff. Services should also expose only necessary ports. A binding such as `127.0.0.1:3000:8080` keeps a development interface on the local host.

Compose interpolation can read ordinary configuration from an `.env` file, but that file should not hold production credentials in source control. A secrets manager or platform-specific secret mechanism should supply sensitive values. Application images should run as a non-root user where practical, mount filesystems read-only where possible, drop unnecessary capabilities, and use separate networks to limit service-to-service reachability.
## Deployment and validation workflow
1. The team defines the task, quality threshold, latency target, concurrency, privacy boundary, retention policy, and hardware budget.
2. Engineers shortlist models whose licences, capabilities, context limits, formats, and engine requirements fit those constraints.
3. Operators pull a specific model variant, inspect its identity and metadata, and record the resolved content digest.
4. Engineers run CLI and direct API smoke tests before adding application code, which separates runtime faults from integration faults.
5. The application backend sends representative requests, includes required conversation history, handles timeouts, and limits retries.
6. Compose declares services and models, injects endpoint details, protects internal networks, and adds readiness checks where services need them.
7. Evaluators test factual quality, refusal behaviour, prompt injection, sensitive-data handling, malformed input, long context, and concurrent load.
8. Operators establish logs, metrics, alerts, backups, patching, rollback, model replacement, and incident-response procedures before release.
## Open WebUI and local chat applications
Open WebUI can provide accounts, conversation history, model selection, and a browser-based chat interface over DMR. A Compose deployment can connect it through DMR's OpenAI or Ollama-compatible API and store application data in a named volume.

An operational deployment should pin a reviewed Open WebUI version or image digest instead of a mutable `main` tag. It should enable authentication outside a tightly isolated single-user environment, protect and back up the data volume, restrict model and web ports, and use TLS for remote access. The volume can contain credentials, settings, and conversations. Removing it with `docker compose down --volumes` deletes that persistent state.

Open WebUI and similar clients can add web search, document retrieval, tools, or external integrations. Those features may transmit prompts, documents, or generated data outside the host. Operators should disable unnecessary functions, review outbound connections, apply least privilege, and define retention rules for prompts and responses.

During diagnosis, operators should first confirm DMR status, list local models, and call the models endpoint directly. A successful list request followed by a failed completion often indicates an incompatible model, insufficient memory, an engine problem, or invalid request data. A failed connection usually indicates a disabled TCP listener, an incorrect container hostname, a missing host-gateway mapping, or a network rule. Browser-only failures may indicate a CORS restriction. Cold loading can explain a slow first response, but sustained latency requires measurement of token rate, queueing, context size, and device utilisation.
## Production controls
- Benchmark each model on representative tasks, hardware, context sizes, and concurrency levels.
- Pin application images and model artefacts, verify publishers and licences, and control registry access.
- Treat models and model files as untrusted supply-chain inputs until validation succeeds.
- Protect prompts, responses, embeddings, logs, conversation stores, and backups as potentially sensitive data.
- Constrain network reachability, authenticate remote access, encrypt traffic, and audit administrative actions.
- Defend tool-enabled applications against prompt injection, excessive permissions, unsafe actions, and data exfiltration.
- Apply CPU, memory, GPU, request-size, context, concurrency, and rate limits to reduce resource exhaustion.
- Monitor quality, latency, failures, resource use, and model changes, and retain a tested rollback path.