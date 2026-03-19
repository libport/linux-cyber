# Managing Software with Ansible
Ansible manages software on Linux with package, yum, dnf, apt, yum_repository, package_facts, rpm_key, redhat_subscription and rhsm_repository. The generic package module suits mixed environments. Platform-specific modules suit distribution-specific behaviour. On RHEL 8 and later, dnf is the package manager underneath even when older tooling still uses yum terminology.
## Repository access
Managed nodes need a reachable repository before they can install or update packages. The yum_repository module writes a .repo file under /etc/yum.repos.d/ and defines the repository name, description, file name, base URL and GPG behaviour.

```yaml
- name: Configure repository access
  hosts: all
  tasks:
    - name: Add example repository
      yum_repository:
        name: example-repo
        description: Example repository
        file: example-repo
        baseurl: ftp://control.example.com/repo/
        enabled: yes
        gpgcheck: yes
```

GPG validation should stay enabled whenever a trusted key is available. Clients can import that key with rpm_key.

```yaml
- name: Import repository signing key
  hosts: all
  tasks:
    - name: Install GPG key
      rpm_key:
        key: ftp://control.example.com/repo/RPM-GPG-KEY
        state: present
```
## Installing and updating packages
The yum module installs, removes and updates individual packages, package groups and AppStream content. A full system update uses the wildcard package name and state: latest.

```yaml
- name: Update all packages
  hosts: all
  tasks:
    - name: Bring installed packages to the latest version
      yum:
        name: '*'
        state: latest
```

Package groups start with @. AppStream content uses the module stream and, if needed, a profile.

```yaml
- name: Install grouped software
  hosts: all
  tasks:
    - name: Install a package group
      yum:
        name: '@Virtualization Host'
        state: latest
    - name: Install a module stream profile
      yum:
        name: '@php:7.3/devel'
        state: present
```

When several packages are required, passing a list to name is more efficient than looping because the package manager resolves them in one transaction.
## Package facts
Standard fact gathering does not include installed package data. The package_facts module adds that information to ansible_facts.packages so tasks can test whether software is present and inspect version details.

```yaml
- name: Gather package details
  hosts: all
  vars:
    my_package: nmap
  tasks:
    - name: Ensure package is installed
      yum:
        name: "{{ my_package }}"
        state: present

    - name: Refresh package facts
      package_facts:
        manager: auto

    - name: Show package details
      debug:
        var: ansible_facts.packages[my_package]
      when: my_package in ansible_facts.packages
```
## Building a local repository
Ansible does not have a single module that creates a repository server from scratch. It combines standard modules to install the server, start the service, open the firewall, create the repository directory, stage packages and generate metadata.

A simple FTP-based repository needs an FTP daemon, a directory such as /var/ftp/repo, downloaded RPMs and repository metadata. On RHEL 8, createrepo_c is the package and createrepo_c is the correct command to generate metadata. Using createrepo may work on some systems through compatibility links, but createrepo_c is the accurate choice.

When packages must be downloaded from an existing repository without installation, yum can use download_only and download_dir. When a file must be downloaded from a URL, get_url is the correct module. The fetch module copies files from managed nodes back to the control node and does not download from arbitrary URLs.
## Managing RHEL subscriptions
RHEL systems often need both repository configuration and a valid subscription. The redhat_subscription module registers the host and supports entitlement attachment. The rhsm_repository module enables repository IDs exposed through Subscription Manager. Credentials should never be stored in plain text inside a playbook. Ansible Vault should hold subscription secrets.

```yaml
- name: Register a RHEL host and enable repositories
  hosts: all
  vars_files:
    - vault.yml
  tasks:
    - name: Register host
      redhat_subscription:
        username: "{{ rhsm_user }}"
        password: "{{ rhsm_password }}"
        state: present

    - name: Enable required repositories
      rhsm_repository:
        name:
          - rh-gluster-3-client-for-rhel-8-x86_64-rpms
          - rhel-8-for-x86_64-appstream-debug-rpms
        state: present
```
## Practical playbook design
A larger bootstrap playbook benefits from tags so operators can rerun only the required phase, such as inventory changes, host preparation or subscription registration. The playbook should add the host to inventory, make name resolution work, create an ansible administrative user, grant controlled sudo access and install an SSH public key.

Some older examples use shell commands to set passwords and copy SSH keys manually. The safer approach is to use the user module with a hashed password and the authorized_key module to install SSH keys idempotently. Changes to sudoers should always use visudo validation.
## Key points
Software automation on RHEL relies on three layers. Repository access exposes packages. Package modules install, update and inspect software. Subscription modules unlock Red Hat content where entitlement is required. The most reliable playbooks keep repository trust explicit, prefer idempotent modules over shell commands, avoid clear-text secrets and organise complex provisioning work with tags.