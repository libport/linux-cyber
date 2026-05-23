# Managing Container Images
Container images give container platforms a repeatable way to package operating system files, application code, libraries, dependencies and configuration. A well-designed image lets a team build once, test consistently and deploy the same artefact across workstations, private hosts, public clouds and Kubernetes clusters.

A container image remains immutable. When a runtime starts a container from an image, it adds a writable layer for the container's runtime changes. This design gives containers much of their speed and predictability. Docker can reuse layers that already exist on a host, so a pull may download only missing metadata or layers rather than an entire software stack.

Image reliability depends on disciplined image design. Teams should build images from source-controlled Dockerfiles rather than creating them from modified running containers with `docker commit`. A Dockerfile provides a reproducible build record. A manually changed container does not.
## Dockerfiles and image structure
A Dockerfile is a plain-text build specification. Docker reads its instructions in order and normally begins from a `FROM` line that selects a base image. Common instructions include:
- `FROM` selects the base image or build stage.
- `RUN` executes build commands.
- `COPY` copies local files into the image.
- `ADD` copies files and can also fetch remote sources or unpack archives, so `COPY` is clearer for ordinary local files.
- `ENV` sets environment variables.
- `ARG` defines build-time variables.
- `CMD` supplies default commands for a running container.
- `ENTRYPOINT` defines the main executable for a container.
- `EXPOSE` documents the port the application listens on, but it does not publish that port by itself.
- `LABEL` adds metadata. The older `MAINTAINER` instruction is deprecated.
- `USER` sets the user and group for following build steps and for runtime commands.
- `WORKDIR` sets the working directory for later instructions.
- `VOLUME` declares a mount point, while a host path is mounted with runtime options such as `-v`.
- `HEALTHCHECK` tests whether the containerised process still responds as expected.

A simple Apache image might start with an Ubuntu base, update package indexes, install Apache, copy an `index.html` file into the web root, document port 80 and start Apache in the foreground. In practice, production Dockerfiles should also minimise installed packages, remove unnecessary package metadata, run services as non-root users where possible and keep the final image focused on one clear purpose.

Build context also matters. Docker sends the selected context to the builder, so large source directories can slow builds and leak unwanted files into the process. A `.dockerignore` file should exclude logs, caches, local credentials, dependency directories and generated artefacts that the image does not need. This keeps builds faster and reduces the chance that sensitive or irrelevant files reach an image layer.

The build should also separate configuration from application binaries. Runtime settings that differ between environments belong in variables, secrets or orchestration manifests rather than baked into the image. This keeps the same image usable across development, test and production while allowing each environment to supply its own credentials, endpoints and scaling settings.
## Reliable and efficient builds
Image tags need careful handling. The `latest` tag is not a stable promise. It is a mutable reference that publishers may retarget. Relying on it can introduce unexpected changes between development and production. Specific version tags improve predictability, but tags can still move. Digest pinning gives the strongest reproducibility because a digest identifies the exact image content.

Reproducibility must still be balanced with patch management. Pinning an old base image can freeze vulnerabilities into the build. A mature workflow monitors base images and dependencies, rebuilds images when fixes are available and records the exact images deployed.

Image size affects pull time, storage and startup behaviour. Each `RUN` instruction can add a layer, so related package operations should usually be combined with `&&`, then cleaned in the same instruction. Multi-stage builds can keep compilers, test tools and temporary files out of the runtime image. Smaller base images such as Alpine can help, but compatibility and security support matter more than size alone.

Trusted sources reduce supply-chain risk. Docker Official Images and other trusted publisher programmes provide better-maintained starting points than unknown community images. Popularity signals such as star counts may help discovery, but they do not replace provenance checks, vulnerability monitoring and policy controls.
## Registries and persistent storage
A registry stores and distributes images. Docker Hub is the default public registry for many workflows, and private registries let organisations keep images near their build and runtime environments. Cloud registries, hosted registries and self-managed registries all serve the same core function: they provide a place to push, pull, share and govern container images.

A local registry can run from the registry image. The server commonly listens on port 5000. Clients tag images with the registry host, port and repository path before pushing them. For example, an image may be retagged from a local name to a registry-qualified name, then uploaded with `docker push`.

A registry container needs persistent storage. Without a mounted volume or durable storage backend, deleting the registry container can delete the stored images. Running the registry with a host-mounted directory or another supported storage backend preserves image data across container restarts and host maintenance.

A registry on a secure local network may be useful for development, but production registries should not rely on trusted networks alone. They need transport encryption, access control, backup, monitoring and clear retention rules.
## Registry security and image trust
Transport encryption protects image data as clients push and pull layers. A registry can use certificates from a recognised certificate authority, including certificates issued through Let's Encrypt. Certificate and key files are mounted into the registry container, and registry configuration points to those files. Self-signed certificates require extra client configuration and should be handled deliberately.

Authentication controls who can push and pull. The open-source registry implementation, now maintained as CNCF Distribution, supports native basic authentication with `htpasswd`. Basic authentication must run over TLS because the scheme sends credentials in HTTP headers. The registry expects bcrypt-formatted htpasswd entries.

Image signing protects against tampering and helps consumers verify provenance. Docker Content Trust signs and verifies images through Notary, but it does not perform vulnerability scanning. Docker has transitioned Notary to the CNCF Notary project, so modern workflows should treat image signing as one part of a broader supply-chain security model that also includes vulnerability scanning, policy enforcement, digest pinning and restricted registry access.

Credentials also need protection on client machines. Docker login data may be stored through a credential helper or in Docker configuration. Kubernetes and shell commands that include secrets can expose credentials through shell history or process listings. Teams should prefer credential stores, short-lived tokens, limited permissions and platform-native secret management.
## Kubernetes and private images
Kubernetes can run public images with a simple `kubectl run` command, but private images require registry credentials. A Pod or workload can reference an image from a private registry and specify `imagePullSecrets`. Kubernetes expects those secrets to exist in the same namespace as the Pod. The common secret type is `kubernetes.io/dockerconfigjson`.

A cluster can access private images in several ways:
- A Pod can reference an `imagePullSecrets` entry.
- A service account can provide image pull secrets for many Pods.
- Nodes can be configured with registry credentials.
- A kubelet credential provider can fetch credentials dynamically.
- Images can be pre-pulled onto nodes, although this requires strict operational control.

The Pod-level secret approach is usually the clearest option for application workloads. It keeps registry access tied to the namespace and workload rather than granting every Pod on a node the same access.

Kubernetes admission controllers can add policy checks before API requests take effect. `AlwaysPullImages` can force image pulls and help ensure the requester has permission to use the image. `ImagePolicyWebhook` can call an external policy service to decide whether an image may run, although Kubernetes disables it by default and recommends using it only where it fits a deliberate policy design. Third-party admission controllers can also rewrite tags to digests or block images from unapproved registries.
## Key practices
Effective image management depends on a small set of disciplined habits:
- Build images from source-controlled Dockerfiles.
- Use trusted base images and pin production workloads with digests where reproducibility matters.
- Monitor and rebuild images when base images or dependencies receive security fixes.
- Keep images small through focused builds, combined package operations and multi-stage builds.
- Store images in registries with persistent storage, backups and access controls.
- Encrypt registry traffic and require authentication for private registries.
- Verify image provenance through signing and policy controls.
- Use Kubernetes image pull secrets or credential providers for private registries.
- Avoid putting secrets directly into command histories, Dockerfiles, image layers or build arguments.

Container images are most valuable when teams treat them as governed deployment artefacts rather than convenient snapshots. Clear Dockerfiles, careful tagging, secure registries and Kubernetes policies turn images into reliable units of delivery across development, testing and production.