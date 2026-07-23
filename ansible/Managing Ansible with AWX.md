## Managing Ansible with AWX
AWX provides a web interface, REST API, and task engine for Ansible automation. It centralises projects, inventories, credentials, job execution, schedules, notifications, and access control. AWX is one upstream project for Red Hat Ansible Automation Platform, not an open-source edition of the whole commercial platform.

The AWX project requires careful evaluation for new production deployments. Its official repository states that releases have been paused since the July 2024 release while maintainers undertake a large-scale architectural refactor. Organisations should confirm the project's current release status, security posture, upgrade path, and operational support before adoption.
### Why AWX helps Ansible scale
Command-line Ansible works well for individual operators and small collections of playbooks. Larger environments introduce coordination and governance problems:
- Projects spread across repositories become difficult to discover, classify, and maintain.
- Separate cron jobs and CI/CD pipelines obscure schedules, ownership, and dependencies.
- Local execution fragments job history, output, and troubleshooting evidence.
- Credentials distributed across projects increase exposure and complicate rotation.
- Teams need controlled delegation without giving every operator repository or infrastructure administration rights.
- Complex business processes can overload playbooks with branching and orchestration logic.
- Audit, retention, approval, and notification requirements become harder to apply consistently.

AWX addresses these problems through a shared control plane. It records job output and history, applies role-based access control, stores encrypted credential fields, schedules jobs, and sends status notifications. Workflow templates connect smaller job templates into conditional processes, while the API integrates automation with CI/CD, service management, chat, and monitoring systems.

Centralisation improves control but does not establish compliance by itself. Administrators must still design roles, protect secrets, limit network exposure, retain appropriate logs, test recovery, patch dependencies, and review automation changes.

Effective administration requires practical knowledge of Linux or another Unix-like environment, Ansible playbooks and inventories, Git-based project management, containers, and basic Kubernetes concepts. Teams also need familiarity with permissions, credentials, network services, and structured API data. AWX reduces operational friction, but it does not replace those foundations.

Event-driven use requires an external event source or integration layer to call AWX. A service-management approval, monitoring alert, repository event, or chat command can trigger an API request that launches an approved template. AWX provides the controlled execution point, while the external system detects and validates the event.
### AWX and Ansible Automation Platform
AWX and Red Hat Ansible Automation Platform serve different risk and support needs. AWX is a community project that exposes new development earlier and does not include a commercial support entitlement. Red Hat Ansible Automation Platform is a subscribed product with supported components and an enterprise lifecycle. Ansible Tower preceded the component now called automation controller. Tower did not become the entire Ansible Automation Platform, which includes capabilities beyond controller functions.

AWX therefore suits laboratories, evaluation, community-supported environments, and organisations prepared to operate the software themselves. The commercial platform better suits environments that require vendor support, certified content, defined lifecycles, and a wider integrated platform. Selection should reflect operational capacity, stability requirements, regulatory obligations, and acceptable risk.
### Deployment with AWX Operator
AWX Operator deploys and manages AWX on Kubernetes. Its basic installation uses Kustomize against a running cluster and requires a selected operator tag. Administrators should use a currently documented, compatible tag instead of copying the obsolete `2.5.3` example from the source material.

Minikube provides a practical local cluster for testing. Docker is one supported driver, not minikube's only dependency or deployment option. Linux also supports drivers such as KVM2, VirtualBox, QEMU, and bare metal, while macOS and Windows support their own driver sets. A Linux virtual machine with Docker and minikube remains a valid lab configuration when the host provides adequate CPU, memory, storage, and network access.

A concise lab deployment uses this sequence:
1. Install and verify a suitable minikube driver, minikube, and `kubectl`.
2. Start a cluster with explicit CPU and memory allocations.
3. Select an AWX Operator release tag and deploy the operator with Kustomize.
4. Create an AWX custom resource and apply it to the cluster.
5. Watch operator logs and pods until AWX and its database become ready.
6. Retrieve the generated administrator password from the relevant Kubernetes secret.
7. Sign in, replace default credentials, and configure durable access appropriate to the environment.

`minikube service` can expose a service for access from the same host. `kubectl port-forward` listens on localhost by default. Remote access requires an explicit `--address` value, suitable firewall rules, and careful protection because broad bindings can expose the service to the network. A properly configured ingress with TLS is safer for sustained shared access.

AWX is community software, so claims about a universally "supported" production installation method are misleading. Kubernetes and AWX Operator form the documented installation path, but each organisation remains responsible for architecture, high availability, backups, upgrades, monitoring, and recovery.

The lab procedure should not become a production blueprint. Production design must address persistent database and project storage, ingress, TLS, DNS, secrets, resource requests and limits, node failure, database recovery, image provenance, and cluster upgrades. Administrators should record the AWX, operator, Kubernetes, PostgreSQL, and execution-environment versions as a tested compatibility set.
### Resource model and access control
An organisation provides an administrative boundary for resources and permissions. Teams group users so administrators can assign roles efficiently. A user may belong to multiple organisations and teams, while external identity providers can reduce local account administration.

Inventories define managed hosts, groups, and variables. They may contain manually managed hosts or draw data from project files and supported inventory sources. A project links AWX to automation content, commonly in Git. Synchronisation creates or updates AWX's local project copy. Administrators can request a sync, schedule one, or enable update-on-launch with a cache timeout to limit repeated source-control traffic.

A job template defines one repeatable execution. It normally references a project and playbook, inventory, credentials, execution environment, variables, limits, verbosity, forks, and other runtime controls. A template can expose selected fields at launch and can attach surveys for structured input. Job templates reference projects, but they are not simply children stored inside a project.

Roles should follow least privilege. Operators who only launch approved templates should not receive permission to edit projects, credentials, or inventories. Credential use can be delegated without revealing secret values, although administrators must still secure encryption keys, backups, external secret integrations, and superuser access.

Execution environments package Ansible Core, collections, Python dependencies, and system libraries into container images. They make job dependencies reproducible across workers and reduce differences between an operator's workstation and AWX. Teams should build images through a controlled pipeline, scan them, publish immutable versions to a trusted registry, and reference fixed tags or digests. Using `latest` weakens reproducibility and can change job behaviour without a template edit.
### Projects, inventories, and templates
A project should point to a controlled source repository and a deliberate revision, branch, tag, or commit policy. Automatic update-on-launch maximises freshness but may introduce unreviewed changes at execution time. Scheduled or pipeline-triggered synchronisation offers more control. Protected branches, signed changes, tests, and release tags reduce supply-chain and operational risk.

Project-sourced inventories allow existing Git-managed inventory files or inventory plugins to feed an AWX inventory. A sync imports the resulting hosts and groups. Teams should avoid maintaining the same hosts manually and dynamically without a clear ownership rule, because conflicting sources can create drift.

Job templates let one playbook serve different environments through separate inventories, credentials, limits, variables, and execution environments. Check mode can simulate supported tasks, but it does not guarantee a complete dry run. Modules without check-mode support may do nothing and report nothing, registered-variable logic may behave differently, and tasks forced with `check_mode: false` can still change systems. Production changes therefore require testing beyond check mode.

Workflow templates arrange job templates as nodes. Links can run subsequent nodes on success, on failure, or always. Parallel branches reduce elapsed time when tasks are independent. Convergence rules decide whether any or all parent nodes must satisfy their conditions before a downstream node starts. Surveys can convert approved user choices into variables, enabling controlled self-service automation without embedding every branch in one large playbook.

Workflow design should keep each node focused and independently testable. Failure branches can collect diagnostics, open incidents, or perform bounded recovery. Approval nodes can pause sensitive changes. An `always` link should receive special scrutiny because it can run after a failed predecessor. Converging branches also need explicit assumptions about partial success, idempotence, and safe retries.
### Jobs, schedules, and notifications
A job is one execution of a job template. AWX displays live and historical output, affected hosts, task events, status, duration, and launch details. Filters and event data support troubleshooting, while retained history improves accountability and trend analysis. Workflow jobs add a visual view of node progress and retain links to each underlying job.

Schedules can launch job templates, workflow templates, project updates, and inventory updates at defined times or intervals. Administrators should set time zones deliberately, review daylight-saving behaviour where relevant, define end conditions, and prevent overlapping executions when a previous job may still be running.

Notification templates send events through services such as email, Slack, PagerDuty, or generic webhooks. They can report starts, successes, failures, and workflow approvals where supported. Teams should test each template, restrict tokens, avoid leaking sensitive job output, and route alerts to owners who can act on them.

Operators can inspect job output by host, play, task, and event. Historical jobs reveal who launched automation, which template and revision ran, when execution began, and whether it succeeded. Troubleshooting should start with the failed event and its host-level data, then examine credentials, inventory resolution, project sync status, execution-environment dependencies, network reachability, and target permissions. Retrying a job without diagnosing a non-idempotent failure can compound damage.
### REST API
AWX exposes its browsable API at `/api/` and currently identifies version 2 through `/api/v2/`. The API supports resource discovery, filtering, pagination, creation, updates, launches, and synchronisation. Automation can use it to register projects, collect job metrics, launch templates from CI/CD or service-management events, synchronise inventories after repository releases, and register execution-environment image references.

Session authentication suits browser interaction. Basic authentication sends a Base64-encoded username and password with every request and should be confined to protected testing where the installed version permits it. Production integrations should use the strongest token or service authentication supported by that AWX release, keep credentials out of shell history and source code, validate TLS certificates, restrict scopes, and rotate secrets.

The correct curl option for an explicit HTTP method is uppercase `-X`, not lowercase `-x`. Lowercase `-x` configures a proxy. A basic authenticated health request for a lab instance takes this form:

```bash
curl -sS -X GET --user 'user:password' \
  'https://awx.example/api/v2/ping/'
```

Launching a job template sends `POST` to its launch endpoint. Synchronising an inventory source sends `POST` to that source's update endpoint. Creating an execution environment sends JSON containing a name and container image reference to the execution-environments collection. Scripts should discover or query object identifiers instead of hard-coding demonstration IDs, check HTTP status codes, handle pagination and retries, and record the AWX job identifier for later monitoring.

A launch client should first query the template or launch endpoint to determine required passwords, variables, inventory choices, or limits. It should then send an explicit JSON payload, validate the response, and poll the returned job URL until the job reaches a terminal state. A successful API request only confirms that AWX accepted the launch. The associated job may later fail, be cancelled, or time out.

Inventory synchronisation follows the same asynchronous pattern. A `POST` to an update endpoint creates an update job, so automation should monitor that job before assuming the inventory is current. Project updates and execution-environment registration also require permission checks, input validation, and clear handling of duplicate names or unavailable images.
### Practical adoption sequence
An incremental rollout reduces risk and gives teams time to refine roles and operating procedures:
1. Deploy a disposable lab and document the tested version set.
2. Create an organisation, connect external authentication where appropriate, and define teams with minimal roles.
3. Import one reviewed Git project and one non-production inventory.
4. Add credentials and a pinned execution environment without exposing secrets to template users.
5. Create a job template, run it against a limited host set, and inspect its output and history.
6. Test project and inventory synchronisation, including failure and rollback behaviour.
7. Add a workflow only when several validated templates need orchestration.
8. Add schedules and notifications with named owners and escalation paths.
9. Integrate the API with scoped credentials and observable job monitoring.
10. Complete backup, restore, upgrade, capacity, security, and incident exercises before wider use.

This sequence keeps the first implementation understandable. It also exposes weaknesses in playbook idempotence, repository controls, inventory ownership, credential boundaries, and execution-environment maintenance before AWX becomes a critical shared service.
### Operational priorities
Reliable AWX operation depends on more than successful job launches. Administrators should establish:
- tested database and encryption-key backups
- controlled upgrades and rollback plans
- TLS, network segmentation, and restricted Kubernetes access
- least-privilege roles and periodic access reviews
- protected repositories and pinned execution-environment images
- log retention, external monitoring, and alert ownership
- capacity limits, concurrency controls, and job cancellation procedures
- recovery exercises and documented service dependencies

These controls allow AWX to centralise Ansible automation while preserving the simplicity of focused playbooks. Projects hold reviewed automation content, inventories define targets, job templates standardise execution, workflows coordinate business processes, and the API connects AWX to wider operational systems.