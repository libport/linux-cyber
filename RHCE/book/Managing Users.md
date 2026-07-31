# Managing Users
RHEL 10 includes `ansible-core` 2.16 for managing RHEL 9 and RHEL 10 nodes. Account automation should define identities, group membership, SSH trust, credentials, and delegated privileges as explicit state. It should also preserve central identity services, avoid secret disclosure, and apply least privilege.

| Component | Responsibility |
|---|---|
| `ansible.builtin.user` | Local account state and core properties |
| `ansible.builtin.group` | Local group state and identifiers |
| `ansible.builtin.known_hosts` | Trusted SSH host keys |
| `ansible.posix.authorized_key` | Public login keys for user accounts |
| `ansible.builtin.file` lookup | Controller-side key and data retrieval |
| `redhat.rhel_system_roles.sudo` | Structured sudo policy on RHEL |
## Users and groups
`ansible.builtin.group` manages local groups. `ansible.builtin.user` manages local accounts, home directories, shells, identifiers, password hashes, account locks, and optional SSH key generation.

The `group` argument assigns the primary group. The `groups` argument assigns supplementary groups. With the default `append: false`, Ansible replaces the existing supplementary set with the declared set. `append: true` adds declared groups while retaining other memberships. Administrators should choose enforcement or preservation deliberately because unconditional appending can leave obsolete access in place.

```yaml
- name: Enforce the account definition
  ansible.builtin.user:
    name: "{{ item.name }}"
    group: "{{ item.primary_group }}"
    groups: "{{ item.supplementary_groups }}"
    append: false
    create_home: true
    shell: /bin/bash
    state: present
  loop: "{{ accounts }}"
```

A structured `accounts` variable lets one task manage several users consistently. Group tasks should create each required group once rather than repeat the same group for every member. Stable UIDs and GIDs help shared storage, backups, and cross-host ownership, but an organisation should allocate them centrally to prevent collisions.

`state: absent` removes an account. `remove: true` can also delete its home directory and mail spool, so removal needs an explicit data-retention decision. Local account automation does not replace Identity Management, LDAP, or Active Directory for centrally governed identities.
## Account lifecycle and verification
Account data should include the login name, UID policy, primary group, supplementary groups, shell, lock state, expiry policy, and authorised keys. `ansible.builtin.assert` can reject duplicate names, duplicate identifiers, missing groups, and invalid shells before any host changes. Inventory and `group_vars` suit non-secret policy data, while Vault or a secret service should hold hashes and credentials.

Service accounts that do not need an interactive session should use an appropriate non-login shell. `password_lock: true` disables password authentication without deleting the account or its files. Password ageing parameters can set minimum, maximum, and warning periods where the platform supports them. These controls should follow the organisation's identity policy rather than use one lifetime for every account.

Verification should query effective state instead of trusting task status alone. `ansible.builtin.getent` can collect passwd and group records, while `ansible.builtin.stat` can check home-directory ownership and permissions. SSH and sudo tests should run under a test identity with the intended command scope.
## Sudo policy
RHEL 10 provides the `redhat.rhel_system_roles.sudo` System Role for consistent, granular sudo policy. It can restrict named users or groups to defined hosts and command paths. Broad rules such as `%admins ALL=(ALL:ALL) NOPASSWD: ALL` grant unrestricted password-free root access and should not serve as a routine default.

Password-free sudo should apply only when non-interactive automation requires it, and each rule should name the smallest practical command set. Wildcards and writable scripts can expand privilege beyond the apparent rule. Teams should test both permitted and denied commands after every policy change.

When a small drop-in file is sufficient, `ansible.builtin.template` or `ansible.builtin.copy` can manage a file under `/etc/sudoers.d/`. The task should set owner and group to `root`, quote mode `"0440"`, and validate the candidate with `/usr/sbin/visudo -cf %s` before installation. Managing the main `/etc/sudoers` file with `lineinfile` creates unnecessary conflict and recovery risk.

RHEL 10 requires `authselect`, which owns selected PAM and NSS configuration. Administrators should use supported `authselect` profiles or appropriate RHEL System Roles instead of editing generated PAM files directly with a generic module.
## SSH trust and authentication
SSH uses separate host and user key pairs. During key exchange, the server proves possession of its private host key with a signature. The client checks the corresponding public host key against a trusted `known_hosts` entry. For public-key login, the client then proves possession of the user's private key by signing session data, and the server checks the public key in `authorized_keys`. Neither private key crosses the network.

`ansible.builtin.known_hosts` manages client trust records. Automation should obtain host keys through an authenticated inventory, provisioning system, SSH certificate authority, or independently verified fingerprint. Blind acceptance or an unverified `ssh-keyscan` result cannot establish server identity.

A changed host key can indicate a legitimate rebuild, a reassigned address, or an attack. Automation should not overwrite the trusted entry until an independent channel confirms the new fingerprint. For a non-standard port, both `name` and `key` use the `[host]:port` form. A template can manage large system-wide trust stores more efficiently than many individual module calls.

`ansible.posix.authorized_key` manages login keys on a managed node and belongs to the `ansible.posix` collection, not `ansible-core`:

```yaml
- name: Authorise the deployment key
  ansible.posix.authorized_key:
    user: deploy
    state: present
    key: "{{ lookup('ansible.builtin.file', 'files/deploy.pub') }}"
    key_options: 'from="192.0.2.0/24",no-agent-forwarding,no-port-forwarding'
```

The file lookup runs on the control node. It can read an absolute path inside a hidden `.ssh` directory when the Ansible process has permission, so no hidden-directory restriction exists. A public-key comment supplies a human-readable label and does not determine key validity.

The module normally creates and secures the user's `.ssh` directory. With an alternative `path`, `manage_dir: false` can prevent it from changing the parent directory. `exclusive: true` removes every key not supplied in the same call. Because exclusivity applies to each loop iteration, all retained keys must be passed together.

Private keys should remain with their owner or an approved secret system. Central automation should distribute public keys rather than create and collect human private keys. If `ansible.builtin.user` generates a key, it creates the pair on the managed node and leaves the private key without a passphrase unless `ssh_key_passphrase` is set. The `ssh_key_comment` parameter remains optional.
## Password hashes
Linux stores a salted, one-way password hash in `/etc/shadow`, not an encrypted password that can be decrypted. A modular crypt value records an algorithm identifier, parameters, a salt, and a hash. The username occupies a separate field. A salt prevents identical passwords from producing identical values, but it does not protect a weak password from offline guessing after hash disclosure.

The `password` parameter of `ansible.builtin.user` expects a hash on Linux. The safest repeatable workflow generates each hash once with a random salt, stores the result in Ansible Vault or an approved secret service, and passes only the stored hash:

```yaml
- name: Create the service account
  ansible.builtin.user:
    name: reportsvc
    password: "{{ vault_reportsvc_password_hash }}"
    update_password: on_create
    state: present
  no_log: true
```

`ansible.builtin.password_hash` can generate SHA-512 crypt and other supported formats. A fixed salt reused for every account weakens the design. Recomputing a randomly salted hash on every run also reports continuous changes, so the generated value should be retained. `update_password: on_create` prevents later replacement, while an intentional rotation can provide a new stored hash and use `update_password: always`.

Shell pipelines such as `echo password | passwd --stdin user` expose clear text to playbook output, shell processing, process inspection, and logs. They also obscure change detection. Debug tasks and command-line extra variables can disclose credentials or hashes. Vault encryption, restricted output, `no_log`, and controlled password rotation provide a safer path.