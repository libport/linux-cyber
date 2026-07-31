# Managing Software with Ansible
RHEL 10 supplies `ansible-core` 2.16 and supports RHEL 9 and RHEL 10 managed nodes. Software automation should control package state, repository trust, registration, and updates without exposing credentials or bypassing platform safeguards.
## Package management
RHEL 10 uses DNF for RPM content. Ansible provides these principal interfaces:

| Interface | Purpose |
|---|---|
| `ansible.builtin.dnf` | Installs, updates, removes, and downloads packages and groups on DNF systems |
| `ansible.builtin.package` | Uses the detected package manager for portable tasks |
| `ansible.builtin.package_facts` | Adds installed-package data to `ansible_facts.packages` |

The `yum` module name remains an alias for `ansible.builtin.dnf`, but the DNF fully qualified collection name states the RHEL 10 implementation clearly. `ansible.builtin.package` suits simple cross-platform tasks, while DNF-specific options require `ansible.builtin.dnf`.

A single task should pass several package names as a list. This approach lets DNF resolve one transaction and avoids the overhead of a loop:

```yaml
- name: Install web packages
  ansible.builtin.dnf:
    name: [httpd, mod_ssl]
    state: present
```

`state: present` installs an available version and preserves idempotence. `state: latest` updates the selected packages. A full update uses `name: "*"` with `state: latest`, but administrators should test it, schedule a maintenance window, and assess whether the new kernel or libraries require a reboot.

RHEL 10 distributes core content through BaseOS and additional user-space content through AppStream. Both repositories are required. Initial RHEL 10 Application Streams ship as RPMs, so obsolete RHEL 8 modular syntax such as `@php:7.3/devel` does not apply. Package groups remain available and can be installed by group name or ID.

`download_only: true` downloads packages without installing them. `download_dir` selects an explicit destination, but it is not a mandatory companion argument.

`state: absent` removes named packages. `update_only: true` prevents an update task from installing a package that is currently absent. Security-focused maintenance can combine `state: latest`, `security: true`, and an explicit host limit. Options such as `allow_downgrade` and `allowerasing` can alter dependency resolution, so administrators should use them only after reviewing the resulting transaction.
## Package facts
Normal fact gathering does not collect the installed package inventory. A separate task populates `ansible_facts.packages`:

```yaml
- name: Gather installed package facts
  ansible.builtin.package_facts:
    manager: auto

- name: Report the installed HTTP server
  ansible.builtin.debug:
    var: ansible_facts.packages.httpd
  when: "'httpd' in ansible_facts.packages"
```

Each package name maps to a list because multiple versions or architectures can coexist. A package-changing task must run before `package_facts` when later conditions need the new state.
## Custom repository access
`ansible.builtin.yum_repository` manages DNF repository definitions, normally as `.repo` files under `/etc/yum.repos.d/`. Its `name` identifies the unique repository ID. `file` selects the file name without the `.repo` suffix. `baseurl`, `enabled`, `gpgcheck`, `gpgkey`, and `state` define access and trust.

```yaml
- name: Configure the internal repository
  ansible.builtin.yum_repository:
    name: internal-tools
    description: Internal tools for RHEL 10
    file: internal
    baseurl: https://repo.example.com/rhel/10/$basearch/
    enabled: true
    gpgcheck: true
    gpgkey: https://repo.example.com/keys/RPM-GPG-KEY-internal
    state: present
```

Administrators should use HTTPS, verify the signing-key fingerprint before import, and keep package signature checking enabled. `ansible.builtin.rpm_key` can import a key and verify its long-form fingerprint. `gpgcheck` validates package signatures. A repository that signs metadata can also enable `repo_gpgcheck`. Disabling checks converts a trust failure into a software supply-chain risk.
## Publishing a custom repository
A repository server needs RPM files, generated metadata, and a secure delivery service. The `createrepo_c` package supplies the metadata generator:

```console
createrepo_c --update /srv/www/repo
```

RHEL 10 uses Zstandard compression for non-database metadata by default, and `createrepo_c` no longer creates SQLite databases by default. Ansible can install `createrepo_c`, copy or download authorised RPMs, run the command when content changes, and publish the directory through HTTPS. The older anonymous FTP design exposes content and key retrieval to avoidable transport risks.

The package-download task still needs access to an upstream repository. For a known HTTPS URL, `ansible.builtin.get_url` downloads a file to a managed node and can verify its checksum. `ansible.builtin.fetch` performs the opposite transfer by copying a file from a managed node to the control node, so it does not replace an HTTP downloader.
## RHEL registration and entitled repositories
RHEL 10 recommends activation keys and an organisation ID for unattended registration. The `redhat.rhel_system_roles.rhc` RHEL System Role can register systems and manage entitled repositories. It avoids the obsolete workflow that attached a host manually to a subscription pool before enabling content.

```yaml
- name: Apply registration and repository settings
  ansible.builtin.include_role:
    name: redhat.rhel_system_roles.rhc
  vars:
    rhc_auth:
      activation_keys:
        keys: ["{{ rhc_activation_key }}"]
    rhc_organization: "{{ rhc_organisation_id }}"
    rhc_repositories: [{name: "{{ rhel10_appstream_repo_id }}", state: enabled}]
```

Repository IDs depend on architecture, subscription, and service configuration. Administrators should discover the available IDs instead of carrying RHEL 8 identifiers into RHEL 10 automation.

The containing play should enable privilege escalation and load vaulted values through `vars_files`. Activation keys, passwords, and tokens must not appear as clear text in playbooks, inventory, shell commands, or command-line extra variables. Ansible Vault protects stored variables, although teams must also protect the vault password and restrict output. `no_log: true` should cover tasks that could return secret values.

The `community.general.redhat_subscription` and `community.general.rhsm_repository` modules provide another registration path. They belong to the `community.general` collection rather than `ansible-core`. Current `rhsm_repository` tasks use `state: enabled` or `state: disabled`, not the removed `present` and `absent` values.
## Playbook structure and verification
Inventory variables, `group_vars`, `host_vars`, and loaded variable files can provide values across several plays. Command-line extra variables have high precedence and suit deliberate overrides, but they expose secrets through process history and logs. Vault-encrypted files or an approved secret service provide safer inputs.

Software playbooks should use `become: true`, fully qualified collection names, descriptive task names, and handlers for actions that follow a reported change. Administrators should first run syntax checks, then check mode with diff output, and finally a live test limited to a non-production host. Tags can isolate registration, repository, installation, and verification phases without duplicating plays. A final `package_facts` task can confirm the installed name, version, release, and architecture.