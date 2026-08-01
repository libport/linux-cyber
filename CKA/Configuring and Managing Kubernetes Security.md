# Configuring and Managing Kubernetes Security
Kubernetes security depends on overlapping controls for identity, authorisation, software supply chains, workload configuration, network paths, secrets, and runtime activity. Each control reduces exposure or limits the effect of a compromised credential, image, or container.
## Secure cluster access
### Authentication and human access
Kubernetes authenticates a caller before an authoriser evaluates access. The API has ServiceAccount objects for workloads, but it has no User objects for people. Production clusters should connect to an external identity provider through a supported mechanism such as OpenID Connect. The provider can enforce multi-factor authentication, device and location rules, account suspension, and short-lived credentials.

Kubeconfig files can contain client keys, bearer tokens, or commands that obtain credentials. Administrators should restrict file permissions, remove stale entries, separate high-privilege and routine contexts, and avoid long-lived static credentials. Managed services can integrate their identity platform with `kubectl` through an exec credential plugin. For example, AKS can use Microsoft Entra ID and `kubelogin`. Disabling AKS local accounts prevents normal retrieval of the certificate-based cluster administrator credential, although the organisation still needs a tightly controlled, audited recovery process.
### RBAC and workload identity
RBAC grants permissions through additive rules. It has no deny rules. Rules match verbs, API groups, resources or non-resource URLs, and scope. Bindings assign those rules to users, groups, or ServiceAccounts.

| Object | Effective scope |
| --- | --- |
| `Role` | Defines permissions within one namespace |
| `RoleBinding` | Grants a Role, or a ClusterRole, within one namespace |
| `ClusterRole` | Defines reusable namespaced permissions or access to cluster-scoped resources |
| `ClusterRoleBinding` | Grants a ClusterRole across the cluster |

Administrators should grant the narrowest required verbs and resources, prefer group bindings for people, and reserve `cluster-admin` for exceptional recovery. Wildcards and broad rights to create Pods can enable indirect privilege escalation. Built-in roles also require review because some permissions expose Secrets or allow workloads to act through powerful ServiceAccounts.

Regular access reviews should examine bindings for unauthenticated groups, dormant identities, cross-namespace grants, and sensitive verbs such as `bind`, `escalate`, and `impersonate`. Namespace separation does not constrain a ClusterRoleBinding, node access, or another cluster-scoped privilege.

Each API-using workload should receive a dedicated ServiceAccount and a namespaced binding. Workloads that do not call the Kubernetes API should set `automountServiceAccountToken: false`. Modern clusters mount short-lived, automatically rotating, audience-bound tokens through projected volumes. Legacy Secret-based ServiceAccount tokens remain long-lived and should only support unavoidable compatibility needs.

Teams can test intended access with `kubectl auth can-i`, including authorised impersonation through `--as`. The result covers authorisation only. Admission policy, quotas, validation, and runtime conditions can still reject an operation. Kubernetes audit logging should record security-relevant API activity, while log storage, retention, and alerting should support investigation without exposing sensitive request data.
## Secure application deployment
### Build and scan trustworthy images
An image contains operating-system packages, language runtimes, libraries, and application code that follow independent release cycles. A clean scan can become stale when researchers publish a new vulnerability. Security teams therefore need both pre-deployment checks and continuing assessment of deployed workloads.

A secure pipeline should build from maintained minimal bases, generate a software bill of materials, scan each immutable image, sign it, and attach verifiable provenance and scan attestations. Workload manifests should reference a digest rather than a mutable tag. Admission control can then verify identity, integrity, provenance, and required attestations before a workload runs. An approved registry alone does not prove that an image is safe.

Vulnerability gates should consider severity, exploitability, exposure, available fixes, and approved exceptions. A critical count alone can block low-risk workloads or permit exploitable lower-severity findings. Teams should define remediation times, rescan when vulnerability data changes, and retain results for audit and trend analysis.

Trivy Operator can scan current workloads and publish `VulnerabilityReport` resources. Those reports support inventory and ongoing detection, but they do not provide a safe first-deployment gate. A new image may have no report until a workload starts, so a policy that queries only existing reports can allow unknown images. CI scanning or signed scan attestations should close that gap.
### Enforce workload configuration
Admission policy can reject unsafe manifests before the API stores them. Pod Security Admission provides built-in enforcement of the Pod Security Standards. Engines such as Kyverno can add organisation-specific validation, mutation, image verification, exceptions, and reports. Current Kyverno validation rules use `failureAction: Audit` to report violations and `failureAction: Enforce` to block them.

Teams should introduce policies in audit or warning mode, measure violations, repair workloads, define narrow and time-limited exceptions, and then enforce. High-value controls include:
- Running as a non-root user with tested user and group IDs
- Setting `allowPrivilegeEscalation: false`
- Applying `seccompProfile.type: RuntimeDefault`
- Dropping unnecessary Linux capabilities, preferably starting with `ALL`
- Avoiding privileged containers and host namespace or host path access
- Using a read-only root file system where the application supports it
- Requiring resource limits and approved immutable images

Policy webhooks extend the API server and can affect availability. Operators should run them redundantly, monitor latency and failures, restrict their permissions and network access, test upgrades, and choose failure behaviour according to the risk of bypass versus outage.
## Contain running workloads
### NetworkPolicy
Kubernetes permits Pod traffic by default when no NetworkPolicy isolates it. A compatible network plugin must enforce the API. Each namespace should start with deliberate default-deny ingress and egress policies, followed by explicit allowances for required application flows.

Policies select Pods through labels and combine allowed traffic additively. A connection between isolated Pods needs matching egress permission at the source and ingress permission at the destination. Default-deny egress also blocks DNS, so clusters need rules for the actual DNS architecture, commonly including UDP and TCP port 53. Administrators should test every permitted and denied path after changes.

NetworkPolicy primarily controls network-layer and transport-layer traffic through selectors, IP blocks, and ports. It does not authenticate application identities, inspect all application protocols, or guarantee that allowed destinations cannot relay traffic. Service meshes, egress gateways, application authentication, and runtime monitoring can address requirements beyond the standard API.
### Protect secrets
Kubernetes Secrets encode values with base64, which does not encrypt them. Secret objects remain suitable when administrators enable API-data encryption at rest, protect etcd and backups, apply restrictive RBAC, and prevent disclosure through logs, manifests, and command history. Applications should mount secrets as files when practical because environment variables can leak through process inspection and do not update during rotation.

An external store such as Azure Key Vault, AWS Secrets Manager, or HashiCorp Vault can centralise policy, rotation, versioning, and access logs. The Secrets Store CSI Driver mounts selected values through a provider and a `SecretProviderClass`. Workload identity should authenticate each ServiceAccount to the provider without static cloud credentials.

External storage does not remove secret exposure from the consuming Pod. A principal that can execute commands in the container, read the process, or control the node may retrieve mounted values. Linux nodes normally expose CSI-mounted values through `tmpfs`, but swap configuration can persist them. Windows nodes may write them to persistent storage. Automatic rotation requires explicit driver configuration, and applications must reread changed files. Synchronising values into Kubernetes Secrets can support environment variables or other consumers, but it restores a copy to the Kubernetes API and etcd.
## Maintain defence in depth
Security controls need continuous verification. Teams should collect API audit events, admission reports, vulnerability findings, identity-provider logs, network telemetry, secret-store access records, and runtime alerts. They should test controls against realistic attack paths, review exceptions, rotate credentials, update dependencies, and rehearse incident response. Clear ownership and measured rollout preserve both security and service availability.

Runtime detection can identify unexpected shells, process execution, sensitive file access, privilege changes, and unusual connections after admission. Teams should tune rules against normal workload behaviour, send alerts outside the cluster, preserve enough context for investigation, and link detections to containment procedures. Detection supplements preventive controls because a permitted image or connection can still carry malicious activity.