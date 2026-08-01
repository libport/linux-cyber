# Docker security implementation
> [!NOTE]
> This guide explains how to secure Docker across the container lifecycle through trusted images, vulnerability scanning, hardened Dockerfiles, protected hosts, encrypted access, secrets management, least privilege, and robust logging.
## Image supply-chain security
Trusted registries, Docker Official Images, and verified publishers improve provenance, but badges, downloads, and stars do not establish security. Teams should verify the publisher, maintenance, signatures, and provenance, then pin the reviewed image by digest. Mutable tags such as `latest` can identify different content after a later push.

Docker Scout and Trivy inventory packages, create or consume a software bill of materials (SBOM), and match components against vulnerability data. Zero findings mean only that the scanner detected no known issues with its current coverage and feeds. Scanners can miss custom binaries, new flaws, insecure configuration, and exploitable application logic.

Teams should prioritise remediation through exploit evidence, reachability, exposure, business impact, vendor status, available fixes, and CISA's Known Exploited Vulnerabilities Catalog. Severity alone does not define organisational risk. Vendor advisories can reflect backported fixes better than upstream version comparisons.

A minimal, supported, and compatible base image usually reduces attack surface, but Alpine or a hardened image does not suit every application. Dockerfiles should use multi-stage builds, omit unnecessary packages, prefer `COPY` for local files, verify remote artefacts, and exclude secrets. Builds should pin dependencies, then refresh them for security updates. On DNF systems, `--setopt=install_weak_deps=False` suppresses weak dependencies. Applications should use a non-root `USER` and `COPY --chown` where required.
## Host and daemon hardening
The current CIS Docker Benchmark provides a baseline, not proof of security or compliance. Docker Bench automates many checks, but its current release targets an older benchmark. Operators must interpret results for the deployed Docker version, review manual checks, and protect the privileged host access the tool requires.

Hosts should run supported, patched kernels and Docker Engine releases. Rootless mode runs the daemon and containers without root privileges. Where it does not fit, `"userns-remap": "default"` maps container root to an unprivileged host identity. Operators must plan storage ownership and compatibility first. `/var/lib/docker`, the Docker socket, configuration, certificates, and backups require strict access control.

The local Unix socket remains the preferred daemon endpoint. Remote access should use an SSH Docker context or mutual TLS with protected keys, valid subject alternative names, rotation, and revocation. An unauthenticated TCP socket grants host control. Daemon TLS protects Docker API traffic, not communication between containers.

Swarm uses mutual TLS for node identity and management traffic, but does not encrypt overlay application data by default. `--opt encrypted` adds IPsec for supported Linux nodes, with a performance cost that operators should test.
## Container, secret, and logging controls
Runtime controls should apply least privilege in layers:

| Control | Secure use |
| --- | --- |
| Identity | Run as a non-root user and consider rootless Docker or user namespaces. |
| Capabilities | Start with `--cap-drop ALL`, then add only required capabilities. |
| Privilege | Avoid `--privileged`, host namespaces, devices, and Docker socket mounts. |
| Filesystem | Use a read-only root filesystem and writable volumes or `tmpfs` only where needed. |
| Kernel controls | Retain the default seccomp profile, enforce AppArmor or SELinux, and set `no-new-privileges`. |
| Resources | Limit memory, CPU, processes, and open files to reduce denial-of-service risk. |

Compose mounts an authorised secret at `/run/secrets/<name>`, but its host source still needs protection. Permissions are not inherently root-only. Swarm encrypts secrets in transit and in its Raft log, then mounts them in memory for authorised Linux tasks. Applications should use supported `_FILE` variables, rotate secrets, and never print, copy, commit, or log values.

`docker logs` and `docker service logs` retrieve container standard output and standard error. Docker events and daemon logs are separate sources. Systemd hosts expose daemon records through `journalctl -u docker.service`. Operators should centralise records, restrict access, synchronise time, define retention, alert on security events, and exclude sensitive data.

Docker defaults to `json-file` without rotation, which can exhaust disk space. The `local` driver rotates by default. Deployments retaining `json-file` should set `max-size` and `max-file`. Remote drivers need authenticated encryption, appropriate buffering, and delivery monitoring.