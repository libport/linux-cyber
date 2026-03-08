# Managing Security
> [!NOTE]
> This file is a practical RHEL security administration guide that covers SSH key-based authentication, POSIX ACLs, and SELinux modes, contexts, and troubleshooting for securing remote access, filesystem permissions, and service configurations.
## SSH key-based authentication
### Replacing passwords with keys
SSH key-based authentication reduces exposure to guessed or reused passwords. A user generates a key pair with `ssh-keygen`. The private key stays on the client. The public key moves to the target account on the remote host, usually with `ssh-copy-id`. The remote host stores that public key in `~/.ssh/authorized_keys` for the destination account.

SSH also verifies the remote host. On the first connection, the client receives the server's host key and, after the user accepts it, stores it in `~/.ssh/known_hosts`. Later connections compare the stored key with the server's presented key. That check helps detect an unexpected or impersonated host.

The default key names in the transcript use `id_rsa` and `id_rsa.pub`. The private key should remain private, and a passphrase should protect it. A stolen encrypted private key is harder to misuse than an unprotected one.
### Creating and installing keys

- Generate a key pair with `ssh-keygen`
- Accept or specify a file path for the key pair
- Add a strong passphrase to protect the private key
- Copy the public key to the remote account with `ssh-copy-id user@host`
- Test the login and confirm that SSH now requests the key passphrase rather than the account password

A localhost session can stand in for a second machine during lab work, provided the SSH server runs locally. The public key must land in the correct destination account, because SSH matches the presented key against that account's `authorized_keys` file. Private and public key material should never be swapped, and the remote `.ssh` directory and `authorized_keys` file should remain locked down so SSH continues to trust them.
### Caching the passphrase
A passphrase improves security, but repeated prompts slow routine work. `ssh-agent` solves that problem by holding decrypted private keys in memory for the life of a session. A user starts the agent in the current shell, then loads the private key with `ssh-add`. After that, SSH and SCP can reuse the cached key without asking for the passphrase every time.

That approach keeps the private key encrypted at rest while making routine connections practical. It also suits login scripts or desktop sessions where the user authenticates once and then reuses the agent throughout the day.
### Simplifying connections with `~/.ssh/config`
The SSH client configuration file can define defaults and short host aliases. A broad `Host *` section can set common options such as the identity file and connection keepalive values. Individual host blocks can then override those defaults for specific systems.

Typical entries include these settings:
- `Host` for the nickname used at the command line
- `HostName` for the real DNS name or IP address
- `User` for the remote account
- `IdentityFile` for the private key to present
- `ServerAliveInterval` and `ServerAliveCountMax` for connection health checks

A concise host alias speeds up routine administration and also supports shell completion. The configuration file must remain private. `chmod 600 ~/.ssh/config` prevents group or world access and avoids SSH refusing the file because of insecure permissions.
### Disabling password authentication safely
Once key-based access works reliably, the SSH daemon can stop accepting passwords by setting `PasswordAuthentication no` in `/etc/ssh/sshd_config`. `sshd -T` helps verify the effective configuration.

- Confirm that at least one administrative account can log in with keys
- Keep an existing root or console session open while testing
- Change `PasswordAuthentication` to `no`
- Reload or restart `sshd` so the daemon applies the new setting
- Open a fresh session and verify key-based login before closing the original session

A syntax check with `sshd -t` before the reload adds one more layer of protection against simple mistakes in the daemon configuration. Disabling password authentication affects SSH only. It does not change local logins.
## POSIX ACLs
### Why standard mode bits are not enough
Traditional UNIX file mode grants access to one owning user, one owning group, and everyone else. That design works for simple ownership but becomes awkward in shared environments. Administrators often end up granting access through the `other` class because one group entry cannot express several distinct access patterns.

POSIX ACLs solve that limitation by attaching a list of entries to a file or directory. Those entries can grant different permissions to multiple named users and groups without changing basic ownership.
### Confirming ACL support
RHEL commonly uses XFS, which supports POSIX ACLs. Ext4 also supports them. ACL capability depends on both filesystem support and kernel support, and the userspace tools normally come from the `acl` package.

Useful checks include these:
- `df -T` to identify the filesystem type
- `uname -r` and the corresponding kernel config file to inspect ACL support if needed
- `rpm -qf /usr/bin/getfacl` to confirm which package provides the ACL tools

The core commands are `getfacl` to read entries and `setfacl` to create, change, or remove them.
### Default ACLs on directories
A default ACL on a directory affects new files and subdirectories created under that directory. It does not retroactively relabel or re-permission existing content. That distinction is essential.

A practical web example shows the value of defaults. Apache content under `/var/www/html` may end up readable through the `other` class if administrators rely only on mode bits. A default ACL can instead grant the Apache service account read access while removing all access for `other`. New files then inherit the intended rule automatically.

- The web server receives only the access it needs
- New content inherits that access automatically
- `other` no longer needs broad read permissions

Directories deserve special care. Read permission lets a user list names. Execute permission lets that user traverse the directory and access entries by name. A directory ACL that omits execute can still prevent useful access even when read appears present. New regular files also differ from new directories. A default ACL may include execute in its template, but regular files still depend on the creation mode and do not normally appear with execute permission unless a workflow explicitly sets it.

`ls -l` marks ACL presence with a trailing `+`. `getfacl` displays the actual entries.
### Managing named users and groups
ACLs can apply directly to a directory, to its default entries, or to both.

- `setfacl -m` adds or changes an entry
- `setfacl -x` removes one entry
- `setfacl -b` removes all ACL entries
- `getfacl -d` shows default ACL entries only

A private directory illustrates the difference between direct and default permissions. An administrator can create a directory with mode `700`, then grant a named user `rwx` access through an ACL without changing ownership. That change allows entry to the directory itself. A separate default ACL is still required if new files created inside that directory must inherit access for the same user.

The transcript describes the group permission bits as a conduit. More precisely, the ACL mask limits the effective permissions of named user entries, named group entries, and the owning group entry. Because of that mask, the group field shown by `ls -l` may appear broader than the owning group's actual business need. Administrators should read the full ACL rather than rely only on the traditional mode string. `getfacl` also reports effective permissions where the mask narrows an entry. That output matters when a named user seems to have `rwx` on paper but the mask quietly reduces the result to `rw-` or `r-x`.
### Backing up and restoring ACLs
ACLs can be exported and restored. Redirecting `getfacl` output to a text file captures the entries in a reusable form. `setfacl --restore` can then reapply them.

That workflow helps during migrations, testing, or bulk rollback.

- Export ACLs with `getfacl`
- Remove them if required with `setfacl -b`
- Restore them later with `setfacl --restore`

The restore file records paths relative to the point where `getfacl` ran, so restore operations should run from a matching location or use carefully adjusted paths.
## SELinux modes and controls
### SELinux as mandatory access control
SELinux is a mandatory access control system, not an access control list. It applies policy decisions even to privileged processes and limits what services can do after a compromise. Standard mode bits and POSIX ACLs remain important, but SELinux adds another layer that can block access even when discretionary permissions would otherwise allow it.

RHEL normally uses the targeted policy. That policy focuses on common services and confines them by type. This matters because many network daemons run with broad filesystem visibility or with elevated privileges. SELinux narrows their reach so a compromised service does not automatically inherit unrestricted access to every readable path or bindable port on the system.
### Operating modes
SELinux has three main modes.

- Enforcing applies policy and logs denials
- Permissive allows actions that policy would block, but still logs denials
- Disabled turns SELinux off

Enforcing is the normal and preferred state. Permissive is a diagnostic state. It allows administrators to see what policy would deny without breaking the workload. Disabled removes both enforcement and the audit trail that would help fix a policy problem. For that reason, disabling SELinux is usually the least useful option.
### Reading and changing the mode
`getenforce` reports the current runtime mode. `sestatus` shows additional detail, including whether SELinux is enabled, the current mode, the configured boot mode, and the loaded policy type. `/etc/selinux/config` holds the persistent mode and policy choice.

`setenforce` switches the runtime state between enforcing and permissive. It does not disable SELinux. Disabling requires a configuration change and a reboot.

The distinction between runtime and persistent state matters.

- `setenforce 0` changes the live system to permissive
- `setenforce 1` returns the live system to enforcing
- `/etc/selinux/config` controls the next boot unless a kernel parameter overrides it
### Preventing runtime mode changes
SELinux booleans provide on or off feature switches inside policy. `getsebool -a` lists them. `semanage boolean -l` provides a fuller view of current and persistent values.

The boolean `secure_mode_policyload` can prevent administrators from changing SELinux mode at runtime. With that control enabled persistently, mode changes must come from the configuration and a reboot rather than an ad hoc `setenforce` command. That can support environments with stricter change control.

The transcript also refers to installing additional SELinux tooling. The practical goal is clear even where package details drift. RHEL benefits from tools that list booleans, explain AVC denials, and manage persistent policy customisations.
### Boot-time overrides
A broken label on a critical file can stop services from behaving correctly at boot. In that situation, a temporary kernel argument such as `enforcing=0` can force permissive mode for one boot. That gives the administrator a chance to repair contexts or policy while preserving audit messages.

This override does not replace the configuration file. It temporarily supersedes it for the current boot.
## SELinux contexts
### Understanding labels
SELinux decisions rely heavily on context labels. A context typically contains user, role, type, and MLS level. Under the targeted policy, the type is usually the most important field.

Commands that expose contexts include these:
- `ls -Z` for files and directories
- `ps -Z` for processes
- `semanage port -l` for port labels

A file such as `/etc/shadow` carries a specialised type like `shadow_t`. A shell process might run as `unconfined_t`. A web server process runs under a more restricted service type, and ports also carry labels that determine which service types may bind them. SELinux then decides whether the source type may access the target type. This is type enforcement. The model is consistent across objects. A process type must be authorised for a file type, a directory type, or a port type before the action succeeds.
### Temporary changes and safe recovery
`chcon` changes a file or directory label directly. That is useful for testing, but it is not the preferred long-term solution because it does not update the policy's default mapping. A later relabel can remove the manual change.

`restorecon` works the other way around. It reads the policy's file context rules and restores the expected label. That makes it the right tool when a file or directory has drifted away from policy.

The transcript demonstrates a dangerous mistake by altering the label of a sensitive authentication file and then recovering with `restorecon`. The broader lesson is practical.

- Use `chcon` sparingly
- Treat it as a temporary override unless policy already defines the mapping
- Use `restorecon` to return files to their expected state
- Test security-sensitive paths carefully before logging out of a recovery shell
### Persistent mappings with `semanage fcontext`
Persistent custom paths require policy changes, not just direct label edits. `semanage fcontext` adds or clones file context mappings in SELinux policy. `restorecon` can then apply those mappings to current files and to future content.

- `chcon` changes the current label on the object
- `semanage fcontext` changes the policy mapping
- `restorecon` applies the policy mapping to objects on disk
### Alternate home directory roots
A site may want staff home directories under `/staff` instead of `/home`. A new directory such as `/staff` usually starts with a generic label like `default_t`, which does not carry the expected semantics of a home directory root.

`semanage fcontext` can clone the mapping from `/home` to `/staff` or define a suitable home root type explicitly. After that, `restorecon` relabels the existing path, and newly created user directories under `/staff` inherit the correct SELinux expectations.

That approach is better than repeatedly applying `chcon` by hand. The policy holds the rule, and the system can reapply it consistently.
## Custom service configurations with SELinux
### Changing Apache to a non-standard port
SELinux also labels ports. An Apache daemon may listen on port 80 by default and on a small set of other permitted HTTP ports. Merely editing Apache to listen on port 1000 does not mean SELinux will permit the bind.

- Change Apache's `Listen` directive to the new port
- Restart or reload Apache
- Test the service
- Inspect AVC denials if the bind fails
- Add a persistent SELinux rule for the new port with `semanage port`

In practice, administrators can use `ausearch` for recent AVC denials and `sealert` for human-readable suggestions. A denial that mentions the Apache process and the new port confirms that policy, not Apache syntax, blocks the bind. Once the port is mapped to the correct HTTP type, Apache can bind successfully under SELinux. This approach keeps the change narrow. It authorises one service class for one additional port instead of weakening SELinux globally.

The port rule belongs in policy. Ad hoc workarounds or broad disables do not.
### Moving the document root
A custom document root causes a similar problem. Apache may have ordinary filesystem permissions to read `/web`, but SELinux can still deny access if that path carries a generic type such as `default_t` rather than an HTTP content type.

The sequence for a safe custom document root:
- Create the new directory, for example `/web`
- Place content there
- Update Apache configuration so `DocumentRoot` and the matching `<Directory>` block reference the new path
- Restart Apache
- Test access and inspect denials
- Add a persistent file context rule for the new path with `semanage fcontext`
- Apply the new mapping to existing content with `restorecon -Rv /web`

After relabelling, Apache can read the content because the service process type and the file type now align with policy.
### Diagnosing denials instead of disabling SELinux
AVC denials are diagnostic signals, not evidence that SELinux should be turned off. In enforcing mode they block the action and log the reason. In permissive mode they allow the action and still log the reason. That makes denials useful for troubleshooting both port changes and document-root changes.

Three tools stand out:
- `ausearch` for recent AVC events from the audit log
- `sealert` for explanatory summaries and suggested fixes
- `man semanage-fcontext` and related manual pages for canonical examples

This method keeps the policy tight while still accommodating real deployment requirements.
## Practical security principles
Several consistent principles run through the course material.
### Prefer precise controls
Security improves when controls match the workload closely.

- SSH keys beat shared or guessable passwords
- ACLs beat broad `other` permissions
- SELinux policy beats service-wide exceptions
- Persistent mappings beat repeated manual fixes
### Separate temporary fixes from permanent ones
RHEL provides tools for both emergency repair and durable policy.

- `setenforce 0` is temporary
- `enforcing=0` is temporary for one boot
- `chcon` is temporary unless policy already agrees
- `semanage fcontext` and `semanage port` are persistent
- `restorecon` reapplies persistent mappings

That distinction reduces drift and makes future relabels predictable.
### Treat audit output as part of the workflow
The most effective troubleshooting pattern is simple:
- make the intended change
- test it
- inspect AVC denials or effective settings
- add the smallest persistent rule that solves the problem
- retest in enforcing mode

That pattern avoids both guesswork and over-correction.
### Protect administrative access while hardening the system
Any change to SSH authentication or SELinux around login paths can cut off remote administration if it is applied carelessly. A console session, an existing root shell, or a verified second session protects against lockout. Hardening should reduce exposure, not remove recovery options.
## Abridged operational checklist
### SSH
- generate a key pair with `ssh-keygen`
- protect the private key with a passphrase
- install the public key with `ssh-copy-id`
- verify host keys through `known_hosts`
- cache the passphrase with `ssh-agent` and `ssh-add`
- define repeatable client settings in `~/.ssh/config`
- set `PasswordAuthentication no` only after verified key logins
- reload or restart `sshd` after configuration changes
### ACLs
- confirm filesystem and package support
- use `getfacl` to inspect entries
- use `setfacl -m` to add or modify entries
- use default ACLs on directories for inherited access on new content
- use `setfacl -x` to remove one entry
- use `setfacl -b` to remove all entries
- export and restore ACLs with `getfacl` and `setfacl --restore`
### SELinux
- keep SELinux in enforcing mode unless troubleshooting
- use permissive mode to collect denials safely
- avoid disabling SELinux
- inspect state with `getenforce` and `sestatus`
- use booleans for targeted policy changes
- use kernel overrides only for recovery
- label custom paths and ports persistently with `semanage`
- apply policy mappings to existing content with `restorecon`
- diagnose AVC denials with `ausearch` and `sealert`
## Security posture in practice
RHEL does not rely on one control to do every job. Strong administration comes from layering small, precise decisions. Keys replace passwords for remote entry. ACLs express shared access without opening paths to everyone. SELinux keeps services inside defined boundaries and records the denials that matter. The safest workflow is repetitive and disciplined. Make one change, test it, read the logs, apply the narrowest permanent fix, and test again in enforcing mode. That approach produces systems that are both harder to break and easier to understand.