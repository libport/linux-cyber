# Managing Container Images
> [!NOTE]
> This guide explains how disciplined Dockerfile design, reproducible builds, secure registries, image-trust controls, and Kubernetes credential policies make container images reliable, efficient, and safe deployment artifacts.
## Image architecture and selection
A container image packages an application's filesystem and runtime configuration as immutable, content-addressed layers. It does not contain a separate kernel or boot like a virtual machine. When a container starts, the engine adds a thin writable layer and runs one or more isolated processes. Removing the container discards that writable layer unless the application stores data in a volume or bind mount.

Layers support build caching, deduplication, and efficient distribution. A changed Dockerfile instruction invalidates its layer and the layers that follow it. `docker image history` shows how an image was assembled, while `docker image inspect` exposes configuration, platform data, labels, and digests.

Tags such as `1.4` provide convenient names but remain mutable. The `latest` tag has no special stability and can identify a different image after a later push. A digest such as `sha256:...` identifies exact content. Digest pinning strengthens reproducibility, but maintainers must deliberately refresh the digest to receive security fixes. Multi-platform images use a manifest list so the client can select an image for the host's operating system and processor architecture. Portability still depends on platform compatibility, runtime configuration, and external services.

Applications should externalise durable data, environment-specific configuration, and credentials. They should send operational logs to standard output and standard error unless the platform provides another managed destination. This separation allows one tested image to progress through development, testing, and production while deployment configuration changes independently.

Image selection should consider:
- A trusted, accountable publisher and a maintained release line
- Compatibility with the required operating system and architecture
- A small package set, clear licensing, and timely security updates
- Available vulnerability results, an SBOM, provenance, and a verifiable signature
- An explicit version or digest rather than an unqualified tag

Docker Official Images provide useful curation, but publisher status, popularity, and star counts do not establish security on their own.
## Reproducible Dockerfiles
A Dockerfile commonly defines an image build. Kubernetes manifests define runtime resources and do not replace that build recipe. A clean build context and a `.dockerignore` file keep unrelated source, credentials, version-control data, and generated files out of the image and cache.

The following Dockerfile builds a simple Apache image:

```dockerfile
FROM ubuntu:24.04
RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends apache2 \
    && rm -rf /var/lib/apt/lists/*
COPY index.html /var/www/html/index.html
EXPOSE 80
CMD ["apachectl", "-D", "FOREGROUND"]
```

The build and run commands remain separate:

```sh
docker build -t webserver:1.0 .
docker run -d --name web -p 127.0.0.1:8080:80 webserver:1.0
```

`EXPOSE 80` records intended container-port metadata. It does not publish the port. The `-p` option maps host port 8080 to container port 80, and binding to `127.0.0.1` limits access to the host. Applications should use published ports or an orchestrated network rather than relying on a container's bridge address.

| Instruction | Purpose and limits |
| --- | --- |
| `FROM` | Selects the base image and platform. A production build can pin its digest. |
| `RUN` | Executes build-time commands and creates a filesystem layer. |
| `COPY` | Copies local build-context files. It suits ordinary file transfer. |
| `ADD` | Adds features such as local tar extraction and remote URLs. Builds should use it only when those features are required. |
| `ARG` | Supplies a build-time value. It must not carry secrets because image metadata and build records can expose it. |
| `ENV` | Sets configuration that persists in later build steps and running containers. It also must not carry secrets. |
| `WORKDIR` | Sets the working directory for later instructions and at runtime. |
| `USER` | Selects the identity for later build steps and the default runtime process. Applications should use a non-root identity where practical. |
| `ENTRYPOINT` | Defines the primary executable. The operator can replace it with `--entrypoint`. |
| `CMD` | Supplies the default command or default arguments. Runtime arguments can replace it. |
| `VOLUME` | Declares a container mount point. It does not select a host directory. |
| `HEALTHCHECK` | Runs a command whose exit status marks the container healthy or unhealthy. |
| `LABEL` | Adds searchable ownership, version, licensing, and build metadata. |
| `STOPSIGNAL` | Chooses the signal that requests graceful termination. |
| `ONBUILD` | Records a trigger for a downstream build and requires careful use. |
| `SHELL` | Changes the default shell used by shell-form instructions. |

BuildKit secret or SSH mounts should provide sensitive build inputs without preserving them in image layers. Runtime secret stores should supply application credentials after deployment.
## Efficient, maintainable builds
Small, maintained base images reduce download time and attack surface, although compatibility and operational support remain essential. Multi-stage builds can compile an application in a tool-rich stage and copy only the runtime artefacts into the final stage.

The main runtime process should handle termination signals correctly so the engine can stop the container cleanly. Exec-form `ENTRYPOINT` and `CMD` instructions avoid an unnecessary shell and usually preserve signal delivery. A container can include supporting processes when the design requires them, but each image should retain a clear operational responsibility.

Dockerfile order controls cache reuse. Stable dependency files should appear before frequently changed source files. Package-index refresh, installation, and cache removal belong in the same `RUN` instruction. Deleting files in a later layer does not remove bytes already stored in an earlier layer, so reducing layer count alone does not guarantee a smaller image. BuildKit cache mounts can accelerate repeated package operations without placing the cache in the final image.

Automated builds should replace manual `docker commit` workflows. Version-controlled Dockerfiles, pinned dependencies, repeatable tests, and regular rebuilds create an auditable path to new patches. Local maintenance uses `docker image ls`, `pull`, `inspect`, `history`, `tag`, `push`, `rm`, and carefully scoped `prune` operations.
## Registries and secure distribution
A registry stores image manifests and layers. A repository groups related image versions. A publisher normally authenticates, adds the registry and repository to the tag, and pushes the result:

```sh
docker tag webserver:1.0 registry.example.com/team/webserver:1.0
docker push registry.example.com/team/webserver:1.0
```

Teams can use Docker Hub, a cloud registry, or a self-hosted CNCF Distribution registry. Google Container Registry stopped serving images on 18 March 2025, so Google Cloud deployments should use Artifact Registry.

The following command starts a loopback-only Distribution registry for local testing:

```sh
docker run -d -p 127.0.0.1:5000:5000 --restart=always --name registry registry:3
```

A production registry requires more than a running container. It needs a fully qualified domain name, trusted TLS, authenticated and authorised access, persistent storage for `/var/lib/registry`, backups, monitoring, renewal of certificates, and an availability plan. Basic authentication must operate over TLS and use bcrypt password records. An `insecure-registries` engine setting belongs only in isolated testing. Credential helpers and `--password-stdin` reduce exposure of registry passwords.

Registry governance should define repository ownership, immutable release tags, retention periods, and deletion procedures. Garbage collection must follow the registry's documented process because removing a manifest does not always reclaim layer storage immediately. Replication or geographically appropriate mirrors can support recovery and reduce pull latency for large deployments.
## Supply-chain assurance
Secure distribution combines several independent controls. A digest fixes the selected artefact. Provenance records how a build produced it. An SBOM inventories components. Vulnerability analysis compares those components with current advisories. A signature links an artefact to an identity and verification policy. A valid signature does not show that an image lacks vulnerabilities or unsafe configuration.

Docker Content Trust relies on Notary v1 and is being retired. Docker states that its hosted Notary service will shut down on 8 December 2026. New workflows should adopt supported signing and verification systems such as Sigstore Cosign or Notation, then enforce identity, repository, and provenance requirements in policy. Docker Scout or another maintained scanner can analyse an SBOM and report known vulnerabilities. Teams should scan during build, before promotion, and again as advisory data changes.
## Kubernetes image access and policy
Kubernetes pulls public images directly and uses `imagePullSecrets` for private registries. The referenced Secret must exist in the same namespace as the Pod. A `kubernetes.io/dockerconfigjson` Secret can hold registry credentials, but base64 encoding does not encrypt them. Clusters should enable Secret encryption at rest, restrict access with least-privilege RBAC, prefer short-lived credentials or an external credential provider, and avoid placing passwords directly on command lines where shell history or process listings can expose them.

An `imagePullSecrets` entry authorises the kubelet to pull an image. It does not inject registry credentials into the application environment. Likewise, a `containerPort` field documents the application's listening port but does not expose it outside the Pod. A Service, and sometimes an Ingress or Gateway, provides network access.

An image reference can use a tag or digest. A digest gives the strongest identity. `imagePullPolicy: Always` resolves the image reference each time a container starts, although the node can reuse cached layers with the same digest. `IfNotPresent` can avoid registry access when the required content already exists. Mutable tags can still change between deployments, so release processes should record and promote digests.

Admission controllers evaluate authenticated and authorised API requests before storage. `AlwaysPullImages` can strengthen isolation in a multi-tenant cluster by requiring image-authorisation checks for every start. `ImagePolicyWebhook` can consult an external policy service, but Kubernetes disables it by default. Modern admission policy can restrict registries, reject mutable tags, require digests and signatures, and apply vulnerability or provenance gates. Such controls complement registry access, network security, runtime hardening, and continuous monitoring.