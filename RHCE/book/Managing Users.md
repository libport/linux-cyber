# Managing Users
Ansible manages Linux accounts with a small set of modules. `user` controls local account properties. `group` manages group membership. `authorized_key` installs SSH public keys. `known_hosts` manages trusted host keys. `lineinfile` and `template` handle supporting configuration such as sudoers content.

The `user` module distinguishes between a primary group and supplementary groups.

- `group` sets the primary group
- `groups` adds supplementary groups
- `append: true` preserves existing supplementary groups for an existing user

When a new user is created without an explicit primary group, the system commonly creates a private group with the same name as the user. Explicit group settings make the result predictable across hosts and easier to audit.

A compact pattern for creating a group and a user is:

```yaml
- name: Create application group
  group:
    name: students
    state: present

- name: Create user
  user:
    name: anna
    group: students
    groups: wheel
    append: true
    create_home: true
    generate_ssh_key: true
    ssh_key_bits: 2048
    ssh_key_file: .ssh/id_rsa
```

This creates the account, assigns group membership, creates a home directory, and generates an SSH key pair. When several users share the same structure, loop over a variable file instead of repeating tasks.
## Managing sudo access
Sudo access is best managed through a dedicated file in `/etc/sudoers.d` rather than by editing `/etc/sudoers` directly. This keeps the change isolated and easier to validate. A template works well when only some groups should receive sudo access.

Example variables:

```yaml
sudo_groups:
  - { name: developers, sudo: false }
  - { name: admins, sudo: true }
  - { name: sales, sudo: true }

users:
  - { username: linda, groups: sales }
  - { username: lori, groups: sales }
  - { username: lisa, groups: account }
```

Template:

```jinja2
{% for item in sudo_groups %}
{% if item.sudo %}
%{{ item.name }} ALL=(ALL:ALL) NOPASSWD:ALL
{% endif %}
{% endfor %}
```

Playbook tasks:

```yaml
- name: Create groups
  group:
    name: "{{ item.name }}"
    state: present
  loop: "{{ sudo_groups }}"

- name: Create users
  user:
    name: "{{ item.username }}"
    groups: "{{ item.groups }}"
  loop: "{{ users }}"

- name: Install sudoers fragment
  template:
    src: sudo_groups.j2
    dest: /etc/sudoers.d/sudogroups
    mode: "0440"
    validate: "visudo -cf %s"
```

This approach grants sudo by group, not by individual account, which keeps the policy easier to maintain. Validation with `visudo` prevents broken sudoers syntax from being deployed.
## Managing SSH access
SSH setup involves two separate trust relationships.

- The client verifies the server's host key, usually through `known_hosts`
- The server verifies the user's public key in `~/.ssh/authorized_keys`

The host does not simply send a token encrypted with its private key for the client to trust blindly. SSH host identity is established during key exchange, and the client checks the presented host key against the stored key. User login with public keys then depends on the matching public key being present in `authorized_keys`.

To manage trusted host keys:

```yaml
- name: Trust managed host key
  known_hosts:
    name: ansible2
    key: "{{ lookup('file', 'files/ansible2_hostkey.pub') }}"
```

To install a user's public key on a managed host:

```yaml
- name: Install public key for ansible user
  authorized_key:
    user: ansible
    state: present
    key: "{{ lookup('file', '/home/ansible/.ssh/id_rsa.pub') }}"
```

For multiple users, loop over a variable and use one key file per user.

```yaml
- name: Install user public keys
  authorized_key:
    user: "{{ item.username }}"
    key: "{{ lookup('file', 'files/' + item.username + '/id_rsa.pub') }}"
  loop: "{{ users }}"
```

`lookup('file', ...)` runs on the control node, not on the managed host. It can read an absolute path if the file is readable to the controller process. Copying keys into the project directory is a workflow choice that can simplify access and versioning, not a rule caused by hidden directories themselves. When Ansible generates user keys, `ssh_key_comment` helps produce a clearer comment in the public key.
## Managing passwords
The `user` module expects a hashed password, not plain text. On Linux, the value stored in `/etc/shadow` contains the algorithm identifier, a salt, and the resulting hash. A common approach uses the `password_hash` filter to generate a SHA-512 hash and then stores that value in a protected variable, ideally with Ansible Vault.

```yaml
vars:
  password_hash_value: "{{ 'password' | password_hash('sha512', 'mysalt') }}"

tasks:
  - name: Create user with hashed password
    user:
      name: anna
      password: "{{ password_hash_value }}"
```

A safer production workflow stores the hash or the underlying secret in Vault. Printing password hashes with `debug` can help during learning, but it should not be part of a normal secure workflow.

Using `echo ... | passwd --stdin` may work on some Linux distributions, including older Red Hat based systems, but it is less portable and can expose secrets in process listings or logs. A hashed password passed to the `user` module is the cleaner default.
## A practical pattern
A complete workflow for local and remote account creation usually follows this order.

1. Define users and their groups in a variable file
2. Create groups
3. Create users with home directories, SSH keys, and hashed passwords where needed
4. Install each user's public key on the managed host with `authorized_key`
5. Install a validated sudoers fragment for privileged groups

This pattern scales well because the same user list can drive both local account creation and remote access configuration. It also avoids common mistakes such as using the wrong module name, overwriting supplementary groups by omitting `append: true`, writing unvalidated content into the sudoers configuration, or confusing key installation with password management.
## Verification and common mistakes
A sound verification pass checks four outcomes.

- Groups exist and users belong to the expected primary and supplementary groups
- Each user can log in with the intended SSH key
- Only the intended groups receive sudo access
- Password hashes are set without exposing secrets in playbooks or logs

Common mistakes include using `groups` where `group` is required, forgetting `append: true` for existing users, installing malformed sudoers content without validation, and assuming `authorized_key` sets a password. It installs public keys only.