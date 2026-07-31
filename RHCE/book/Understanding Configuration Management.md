# Understanding Configuration Management
## Automation and infrastructure as code
Manual administration and ad hoc shell scripts become difficult to audit, repeat, and scale across large estates. Scripts also tend to encode procedures instead of a desired end state, and they may handle changed starting conditions poorly.

Configuration management records the intended state as machine-readable text and applies only the changes required to reach it. Teams should keep this infrastructure-as-code content in a version-control system, usually Git, so they can review changes, trace history, test revisions, and restore earlier versions. CVS names a particular legacy version-control product, not the general category.

This approach supports DevOps by connecting software delivery with infrastructure operations. Automated building, testing, release, deployment, configuration, and monitoring shorten feedback cycles and reduce manual variation.
## Ansible architecture
Ansible uses four core elements:
- A control node runs `ansible-core` and initiates automation.
- An inventory identifies managed nodes and organises them into groups.
- YAML playbooks contain plays and ordered tasks.
- Modules perform work, while plug-ins alter Ansible behaviour.

Collections distribute related modules, plug-ins, roles, and playbooks. Ansible usually needs no agent on managed nodes. It connects to POSIX systems through SSH, to Windows through PSRP, WinRM, or supported SSH configurations, and to network or cloud services through connection plug-ins and APIs. Most POSIX modules require Python on the managed node, although Ansible itself need not be installed there.

Agentless operation reduces installed software, but it does not remove dependencies. Credentials, network access, privilege escalation, Python or PowerShell, and module-specific libraries must suit the chosen transport and task. Teams should protect credentials and grant only necessary privileges.

RHEL 10 supplies Python 3.12 as its default implementation and includes `ansible-core` 2.16. A RHEL 10 control node supports RHEL 9 and RHEL 10 managed nodes. Installing `rhel-system-roles` also installs `ansible-core` and the `redhat.rhel_system_roles` collection, which provides supported roles for services including networking, storage, SELinux, firewalls, Podman, logging, and time synchronisation.
## Effective automation
Declarative tasks specify a desired state. Purpose-built modules, such as package, service, file, and user modules, usually detect whether change is necessary. Repeating an idempotent task therefore leaves an already compliant system unchanged. Not every module or playbook is idempotent, especially when it runs arbitrary commands, so teams must test repeated execution and use check mode where supported.

Readable playbooks, meaningful task names, fully qualified collection names, version control, and small reusable roles improve maintenance. Playbooks may import roles and other content, so they need not be self-contained.
## Platform and use cases
`Ansible Engine` and `Ansible Tower` are obsolete product names. `ansible-core` provides command-line automation. Red Hat Ansible Automation Platform adds automation controller, role-based access control, credential handling, scheduling, logging, APIs, and execution environments. AWX remains the upstream community project. Execution environments package Ansible Core, Runner, collections, Python libraries, and system dependencies in shareable container images.

Ansible primarily manages configuration, but it also deploys applications, orchestrates services, provisions cloud, virtual, network, and bare-metal resources through suitable APIs, supports delivery pipelines, and responds to events. Provisioning calls the relevant platform rather than pushing a virtual-machine configuration file to a new host.