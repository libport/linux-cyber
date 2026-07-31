# Installing Ansible
## Environment and versions
An Ansible environment contains a control node, an inventory, and managed nodes. The control node runs commands and playbooks. Managed nodes receive work through a suitable connection, but they do not require an Ansible agent.

RHEL 10 provides Python 3.12 as its default implementation and includes `ansible-core` 2.16. This RHEL package supports the management of RHEL 9 and RHEL 10 nodes. A RHEL subscription supports `ansible-core` for Red Hat-provided automation content, including RHEL system roles. Red Hat supports broader enterprise automation through Red Hat Ansible Automation Platform.

The former RHEL 8 Ansible repository, CentOS 8 EPEL procedure, Python 2 discussion, `yum install ansible` command, and fixed Ansible 2.x guidance no longer apply. Hardware requirements depend on inventory size, parallelism, collections, and workload. The former one CPU, 1 GB of RAM, and 20 GB of disk specification described a small lab, not a product requirement.
## Installing the control node
A registered RHEL 10 host can install the supported RHEL system roles and their runtime with:

```shell
sudo dnf install rhel-system-roles
python3 --version
ansible --version
ansible-galaxy collection list
```

The package installs `ansible-core` as a dependency and places the `redhat.rhel_system_roles` collection under `/usr/share/ansible/collections/ansible_collections/redhat/rhel_system_roles/`. It does not require the obsolete `ansible-2-for-rhel-8-x86_64-rpms` repository.

Administrators can use `pipx` for an isolated upstream community installation of `ansible` or `ansible-core`. They should not run `pip` as root against the RHEL system Python because this can replace supported libraries. A virtual environment offers another isolated option. RHEL 10 already maps the unversioned `python` command to Python 3.12, so the old `alternatives --set python` step is unnecessary.

Ansible Automation Platform runs automation in containerised execution environments and manages jobs through automation controller. These names replace the former Ansible Engine and Ansible Tower terminology.

A project-level `ansible.cfg` can define the default inventory, remote account, connection options, and privilege-escalation behaviour. Teams should keep configuration and inventory under version control, exclude secrets, and use fully qualified collection names to avoid ambiguity. Installing `ansible-core` alone does not install every community collection. Collections and roles can require extra Python libraries or system packages, so teams should validate each automation dependency before deployment.
## Preparing managed nodes
Each managed RHEL node needs:
- Network reachability from the control node
- A resolvable inventory name or an explicit address
- A running SSH service and a firewall rule for the configured SSH port
- A login account with an interactive POSIX shell
- Python for standard Python-based modules

RHEL 10 normally provides Python 3.12. Administrators can install it with `dnf install python3` when an image omits it. The `raw` module can bootstrap a host without Python. Network appliances often use device APIs or connection plug-ins and do not need Python. Windows nodes use PSRP, WinRM, or supported SSH configurations, along with Windows-specific PowerShell modules.

SSH does not always use TCP port 22, so the firewall must allow the actual configured port. Installation profiles and cloud images can also disable `sshd` or omit packages, so administrators should verify services instead of assuming their state.
## Accounts, keys, and privilege escalation
A dedicated automation account improves auditability, but Ansible does not require the username `ansible`. Direct root login is unnecessary and increases risk.

Key-based SSH authentication suits unattended automation. `ssh-keygen` creates a key pair on the control node, and `ssh-copy-id` installs only the public key on a managed node. Administrators must protect the private key on the control node. A passphrase and `ssh-agent` strengthen key security without prompting for every connection.

SSH password authentication remains possible. The RHEL 10 `ansible-core` package currently omits `sshpass` as a dependency, so password-based SSH requires that additional package. Key-based authentication or centrally managed credentials usually provide a stronger operational design.

Ansible uses `become` for tasks that need elevated privileges. The remote account needs only the permissions required by those tasks, and the control node needs local `sudo` access only for local privileged work. Administrators should edit drop-in policies with `visudo -f /etc/sudoers.d/<account>`. A password prompt can work with `--ask-become-pass`, while automation controller can manage credentials centrally. `NOPASSWD: ALL` grants broad root access and should not serve as the default.
## Verification sequence
1. Administrators install and verify the control-node packages.
2. They add managed hosts to an INI or YAML inventory.
3. They confirm name resolution, host keys, SSH authentication, and privilege escalation.
4. They run `ansible-inventory --graph` and `ansible all -m ping`.
5. They test a playbook in check mode, where its modules support it, before applying changes.