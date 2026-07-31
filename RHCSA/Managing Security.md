# Managing Security
> [!NOTE]
> A layered approach to securing RHEL systems through SSH key authentication, precise ACL permissions, SELinux enforcement, persistent policy configuration, and disciplined audit-led troubleshooting.

RHEL 8 protects systems through several complementary controls. OpenSSH authenticates remote hosts and users. POSIX access control lists extend conventional owner, group, and other permissions. SELinux applies mandatory access control after ordinary discretionary checks. Each control addresses a different risk, so one control does not replace another.

Secure administration requires more than enabling features. Administrators must verify host identities, protect private keys, calculate effective ACL permissions, keep SELinux enforcing, and test every configuration change. They should also retain a recovery path before changing authentication or policy settings.
## SSH key-based authentication
### Host authentication and user authentication
SSH performs two distinct checks:

| Check | Client-side record | Server-side record | Purpose |
| --- | --- | --- | --- |
| Host authentication | `~/.ssh/known_hosts` | Server host private key | Helps the client detect server impersonation and changed host keys |
| User authentication | User private key | User public key in `~/.ssh/authorized_keys` | Proves that the connecting user controls the matching private key |

On the first connection, the server presents its host public key. The client displays a fingerprint and, after acceptance, records the host key in `known_hosts`. Acceptance alone does not establish trust. An administrator should compare the fingerprint with a value obtained through a trusted channel. On later connections, SSH warns if the stored identity and presented identity differ.

The client keeps the user's private key. The server receives only the public key. A strong passphrase encrypts the private key at rest and reduces the damage caused by a copied key file. File permissions should prevent other local users from reading private material.

Key-based authentication resists online password guessing, but its security still depends on key custody and account management. A stolen, unlocked private key can authenticate until an administrator removes or restricts the matching public key. Each person, device, and automation service should therefore use a distinct identity. Distinct keys let an administrator revoke one credential without disrupting unrelated access and make audit records easier to interpret.

The comment at the end of a public key helps identify its owner and origin, but the server does not treat that comment as an access-control field. Administrators should maintain a separate inventory that records the responsible person or service, authorised destinations, creation date, cryptographic policy, and planned review date. They should remove obsolete entries from `authorized_keys`, protect private-key backups, and avoid copying one private key across many hosts.

An organisation should select a key algorithm and size that comply with its cryptographic policy. A typical setup generates a key pair, installs the public key for the intended remote account, and verifies public-key authentication:

```shell
ssh-keygen -t ed25519
ssh-copy-id -i ~/.ssh/id_ed25519.pub admin@server.example
ssh -o PreferredAuthentications=publickey admin@server.example
```

Ed25519 does not suit every compliance mode, including some FIPS configurations. Where policy excludes it, the administrator should generate an approved alternative. The private key must remain on the client. `ssh-copy-id` normally appends the selected public key to the remote account's `authorized_keys` file and establishes the required directory and file permissions.

Common permissions include:

```shell
chmod 700 ~/.ssh
chmod 600 ~/.ssh/config
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_ed25519
```

The exact private-key filename depends on the selected algorithm and any custom name. On an SELinux system, `restorecon -Rv ~/.ssh` can restore expected labels after files have been copied or created unusually.

`authorized_keys` can also constrain a key with options such as a forced command, a source-address restriction, or disabled forwarding. These restrictions suit narrowly defined automation accounts, especially backup and deployment services. Administrators should test them carefully because an incorrect forced command or source restriction can block the intended workflow. A key used for interactive administration should not automatically become a general automation credential.
### Using `ssh-agent`
A passphrase-protected key normally prompts for its passphrase when the client needs the key. `ssh-agent` does not cache the passphrase. It holds an unlocked private key in memory and answers signing requests on the user's behalf. `ssh-add` loads an identity into the running agent:

```shell
eval "$(ssh-agent -s)"
ssh-add -t 1h ~/.ssh/id_ed25519
ssh-add -l
```

The optional lifetime reduces exposure by removing the identity after a set period. Desktop and login environments often start an agent automatically, so starting a second agent in every shell can create abandoned processes and confusing sockets. Administrators should inspect the existing environment before adding agent startup code. They should also avoid forwarding an agent to untrusted hosts because a process that can access the forwarded socket may request signatures.
### Client configuration
The per-user file `~/.ssh/config` can define aliases, users, identity files, and connection settings. OpenSSH uses the first value it obtains for most directives, so specific host blocks should precede broad defaults:

```text
Host app-prod
    HostName 192.0.2.20
    User admin
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes

Host *
    ServerAliveInterval 300
    ServerAliveCountMax 2
```

The alias permits `ssh app-prod`. `ServerAliveInterval` asks the server for a response after a period without received data, while `ServerAliveCountMax` limits unanswered probes before disconnection. These controls detect a lost connection. They do not strengthen authentication.
### Disabling password authentication safely
An administrator should disable SSH password authentication only after every required account can log in with a key. The administrator should retain an existing privileged session, console access, or out-of-band recovery while testing. On RHEL 8, relevant server settings include:

```text
PubkeyAuthentication yes
PasswordAuthentication no
ChallengeResponseAuthentication no
```

Included configuration files and `Match` blocks can affect the effective result. The daemon can validate syntax and display resolved settings:

```shell
sshd -t
sshd -T | grep -E 'pubkeyauthentication|passwordauthentication|challengeresponseauthentication'
systemctl reload sshd
```

Editing `sshd_config` does not make a running daemon adopt the change automatically. A reload or restart must follow successful validation. The administrator should then open a separate connection and confirm key-based access before closing the recovery session. Disabling `PasswordAuthentication` alone may leave keyboard-interactive authentication available, so the effective authentication methods require explicit review.

These settings affect remote authentication through `sshd`. They do not disable local console passwords, `sudo` authentication, or passwords used by unrelated services. Other controls require separate decisions, including direct root login, permitted user groups, multi-factor authentication, connection limits, and idle-session policy. Administrators should change only the controls supported by the organisation's access and recovery design.
## POSIX access control lists
Traditional Linux mode bits define permissions for one owner, one owning group, and all other users. POSIX ACLs add named user and named group entries without forcing administrators to create a new group for every sharing pattern. They remain discretionary controls. The file owner or a sufficiently privileged process can change them, and SELinux can still deny an operation that an ACL permits.

RHEL 8 normally supports POSIX ACLs on XFS and ext4. The `acl` package supplies `getfacl` and `setfacl`. Administrators can confirm the package, file-system type, and mount options:

```shell
rpm -q acl
findmnt -no FSTYPE,OPTIONS /srv/project
getfacl /srv/project
```

An access ACL applies to an existing file or directory. A default ACL exists only on a directory and supplies the initial access ACL for newly created children. A default ACL does not alter existing children, and it does not grant access to the directory that holds it.

Linux evaluates ACL entries in a defined order. It first checks the file owner. It then checks a matching named-user entry. If neither applies, it evaluates the owning group and every matching named group. Finally, it checks the `other` entry. The mask limits named-user and group-class results. An ACL does not provide a general ordered list of allow and deny rules, so administrators should not design it like a Windows discretionary ACL.

The access decision also uses the process's effective user ID, effective group ID, and supplementary groups. A user who belongs to several matching groups can receive the combined permissions of those matching group entries, subject to the mask. Removing permissions from one matching group does not create an explicit denial if another matching group grants them.
### Managing access ACLs
`setfacl -m` adds or changes entries. `setfacl -x` removes a selected entry. The `X` permission adds execute access to directories and to files that already have an execute bit, which makes recursive directory operations safer than granting execute to every regular file.

```shell
setfacl -m u:alice:rwX,g:auditors:rX /srv/project
getfacl /srv/project
setfacl -x u:alice /srv/project
```

Directory permissions have distinct meanings. Read lists names, write creates or removes entries, and execute searches or traverses the directory. A user often needs execute on every parent directory in a path even when the final file requires only read access.
### Default ACLs and object creation
A directory can give an operations group continuing access to new content while excluding other users:

```shell
setfacl -m u::rwx,g::r-x,g:ops:rwx,m::rwx,o::--- /srv/project
setfacl -m d:u::rwx,d:g::r-x,d:g:ops:rwx,d:m::rwx,d:o::--- /srv/project
getfacl /srv/project
```

The first command sets access on the directory itself. The second defines the default ACL inherited by new children. During creation, the requested file mode still limits inherited permissions. A program that creates a regular file without execute permission does not gain execute permission solely because the directory's default ACL includes it.
### The ACL mask
An extended ACL contains a mask that limits the effective rights of the owning group, named users, and named groups. The mask does not limit the file owner or the `other` entry. `getfacl` reports an `effective` comment when an entry requests permissions that the mask removes.

When an extended ACL exists, the group triplet displayed by `ls -l` represents the ACL mask, not necessarily the owning group's entry. A mode such as `rwxrwx---` can therefore coexist with a more restrictive owning-group ACL. The display does not mean that every member of the owning group receives all three group-class permissions.

`setfacl` normally recalculates the mask from the union of affected entries unless an administrator supplies a mask or uses `-n`. A later `chmod` can also change the mask and unexpectedly reduce or expand effective ACL rights. Administrators should inspect `getfacl` after either command instead of inferring access from `ls -l` alone.
### Removal, backup, and restoration
An administrator can remove a named entry with `-x`, remove extended access entries with `-b`, and remove a directory's default ACL with `-k`. A recursive backup preserves ACL text for restoration:

```shell
getfacl -R -p /srv/project > /root/project.acl
setfacl --restore=/root/project.acl
```

The `-p` option retains absolute pathnames. Without it, `getfacl` normally strips leading slashes, so restoration depends on the working directory recorded by the backup. Backups should receive protection appropriate to the ownership and access information they contain.

Copy and archive workflows need explicit testing. A move within one file system normally preserves an inode and its ACL, while a copy creates a new object and may inherit the destination directory's default ACL. Tools and options differ in whether they preserve ACL metadata. Administrators should verify restored content with `getfacl` instead of assuming that matching file data also means matching access controls.
## SELinux states, modes, and policy
SELinux adds mandatory access control to the kernel. Conventional mode bits and POSIX ACLs run first. If they permit an operation, SELinux policy can still deny it. This design confines services even when a process has elevated discretionary privileges.

Policy evaluates subjects and objects, not pathnames alone. A process domain identifies the subject. File types, port types, device types, object classes, and requested operations describe the target interaction. A confined process running with user ID 0 can bypass many discretionary checks through Linux capabilities, yet SELinux can still deny an operation outside the process domain's authorised policy.

SELinux can be enabled or disabled. When enabled, it runs in one of two modes:

| Mode or state | Policy loaded | Denials enforced | AVC denials logged |
| --- | --- | --- | --- |
| Enforcing | Yes | Yes | Yes |
| Permissive | Yes | No | Yes |
| Disabled state | No | No | No |

RHEL 8 installs with the targeted policy in enforcing mode by default. Targeted policy confines selected service domains while leaving many interactive users unconfined. Permissive mode supports diagnosis because it records operations that policy would deny. Disabled SELinux provides neither enforcement nor useful AVC evidence and can leave new objects unlabelled.

`getenforce` reports the current condition. `sestatus` also reports the loaded policy, current mode, and configured boot mode:

```shell
getenforce
sestatus
```

`setenforce 0` changes an enabled system to permissive mode for the current boot. `setenforce 1` returns it to enforcing mode. The setting does not survive a reboot. `/etc/selinux/config` controls the persistent mode, and a reboot applies the change. The `enforcing=0` kernel parameter can provide a one-boot permissive recovery path when enforcing policy prevents a successful boot.

Administrators should prefer a short, controlled permissive interval to disabling SELinux. They should reproduce the failure, collect AVC records, correct the configuration, and return to enforcing mode. Re-enabling SELinux after a disabled period requires careful relabelling because objects created while policy was absent may lack valid labels. A controlled recovery commonly starts in permissive mode, schedules relabelling with `fixfiles -F onboot`, reboots, reviews denials, and then restores enforcing mode.

System-wide permissive mode removes enforcement from every domain. Where diagnosis requires a longer observation period, an administrator can place one domain into permissive operation with `semanage permissive -a <domain>` while the rest of the system remains enforcing. This exception still weakens protection for that domain, so it needs a defined scope, monitoring, and prompt removal with `semanage permissive -d <domain>`.
### Policy tools and booleans
The base packages provide commands such as `getenforce`, `setenforce`, and `restorecon`. RHEL 8 supplies `semanage` through `policycoreutils-python-utils`, while `setroubleshoot-server` provides higher-level denial analysis:

```shell
dnf install policycoreutils-python-utils setroubleshoot-server
```

SELinux booleans expose supported policy choices without requiring a new policy module. Administrators can inspect available settings and descriptions, then enable only the capability required by the service:

```shell
getsebool -a
semanage boolean -l
setsebool -P use_nfs_home_dirs on
```

The `-P` option writes the value to persistent policy configuration. A boolean broadens or narrows an existing policy path, so an administrator should read its description and assess its scope before changing it.
## SELinux contexts and persistent labelling
SELinux associates a context with files, processes, ports, and other objects. A context normally contains an SELinux user, role, type, and level:

```text
system_u:object_r:shadow_t:s0
```

Under targeted policy, type enforcement drives most service decisions. A process runs in a source domain such as `httpd_t`, while a target file carries a type such as `httpd_sys_content_t`. Policy rules authorise a domain to perform specific operations on a target type and object class. Source and target types do not need to match.

For example, the password-changing program can transition into a domain such as `passwd_t`, while `/etc/shadow` carries `shadow_t`. Policy allows the password domain to perform the required operations on that file type. A general web-service domain receives no equivalent permission. The relationship comes from an allow rule for a domain, type, class, and permission, not from identical labels.

Administrators can display contexts with:

```shell
ls -Z /etc/shadow
ps -eZ
semanage port -l
```

`chcon` changes the label stored on an object, but it does not update the persistent file-context mapping. A later `restorecon` or full relabel can discard that change. `semanage fcontext` records a persistent pathname rule, and `restorecon` applies the expected label to existing objects:

```shell
semanage fcontext -a -t httpd_sys_content_t '/web(/.*)?'
restorecon -Rv /web
matchpathcon -V /web/index.html
```

Adding an `fcontext` rule does not relabel existing files by itself. The explicit `restorecon` step remains essential. `semanage fcontext -l -C` lists local file-context customisations.

File-context expressions apply to full paths and use regular-expression syntax. The pattern `'/web(/.*)?'` covers the directory and every descendant. An overly broad pattern can assign a service type to unrelated data, while an overly narrow pattern leaves descendants with an unsuitable type. `matchpathcon` shows the label that policy expects, and `restorecon -nRv` can preview proposed changes without writing them.

An equivalence rule can make a new tree follow an established tree's context mappings. A separate staff home hierarchy can mirror `/home`:

```shell
mkdir /staff
semanage fcontext -a -e /home /staff
restorecon -Rv /staff
```

The equivalence rule records expected labels for the new path. File-creation behaviour, service tools, or a later `restorecon` determines when objects receive those labels. Administrators should verify the result rather than assume that the mapping changed every inode immediately.

Deliberately relabelling `/etc/shadow` provides a hazardous demonstration because it can disable authentication and `sudo`. Training should use disposable systems and non-critical test objects. If an accidental label change affects a system file, a retained root session, rescue environment, or console can run `restorecon` on the affected path.
## Diagnosing SELinux denials
An access failure does not automatically identify SELinux as the cause. Administrators should first check service configuration, process state, file ownership, mode bits, ACLs, parent-directory traversal, firewall rules, and network listeners. SELinux evaluates access alongside these controls rather than replacing them.

A disciplined diagnosis follows a short sequence:
1. `getenforce` confirms whether SELinux can enforce a denial.
2. The administrator reproduces the failure once and records the time.
3. `ausearch` retrieves relevant audit events.
4. The administrator reads the source context, target context, object class, requested permission, path, and process.
5. The administrator selects an existing label, port type, boolean, or documented configuration that expresses the intended access.
6. The service runs again under enforcing mode, and a functional test confirms the result.

RHEL 8 can search recent audit records with:

```shell
ausearch -m AVC,USER_AVC,SELINUX_ERR,USER_SELINUX_ERR -ts recent
```

When `setroubleshoot-server` is installed, `sealert -l "*"` can explain recorded alerts and suggest fixes. Suggestions require review. Automatically generating and installing an `audit2allow` module can authorise behaviour caused by a wrong label, unsafe file permission, or incorrect service configuration. Existing policy mechanisms should take priority over a new local allow rule.

An AVC record commonly identifies `scontext`, `tcontext`, `tclass`, the denied permission, the process name, and a target path or port. The source context shows the process domain. The target context shows the object's current type. The class distinguishes files, directories, sockets, and other object kinds. These fields let the administrator test a specific hypothesis instead of treating every denial as a request for broader policy.

| Evidence | Likely correction |
| --- | --- |
| Standard path has an unexpected type | Apply `restorecon` and investigate how the label changed |
| Custom path needs the same use as a standard path | Add an `fcontext` type or equivalence rule, then apply `restorecon` |
| Service binds to an approved alternate port | Assign the documented SELinux port type |
| Service needs a supported optional capability | Evaluate and set the relevant SELinux boolean |
| Application behaviour has no suitable policy path | Correct the application or develop a narrowly scoped local policy |

Permissive mode can produce denials for operations that an enforcing system would have stopped earlier, so not every permissive AVC represents an independently reachable failure. Administrators should reproduce the intended transaction under enforcing mode after correction and confirm that the service still follows least privilege.
## Apache with non-standard settings
Changing an HTTP service from port 80 to an uncommon port does not provide meaningful security. Network scans can still discover the service, and the same application remains exposed. An alternate port should serve an operational requirement, while firewalls, TLS, authentication, patching, and least privilege provide substantive protection.

SELinux permits `httpd_t` to bind only to ports labelled with an allowed type. An administrator can inspect the current `http_port_t` assignments before choosing a port:

```shell
semanage port -l | grep -w http_port_t
```

If TCP port 3131 has no conflicting SELinux assignment and Apache must use it, the administrator can add the port label:

```shell
semanage port -a -t http_port_t -p tcp 3131
```

If the port already belongs to another SELinux type, `-a` fails. The administrator should investigate the existing assignment and service ownership before considering `semanage port -m`. The Apache `Listen` directive, the host firewall, and any upstream network controls must also permit the selected port. After configuration, `apachectl configtest`, `systemctl restart httpd`, `systemctl status httpd`, and `ss -ltnp` can validate syntax, service state, and the listening socket.

A minimal Apache change includes a listening port and an explicit authorisation block for the document root:

```apache
Listen 3131
DocumentRoot "/var/test_www/html"

<Directory "/var/test_www/html">
    Require all granted
</Directory>
```

`Require all granted` permits HTTP access through Apache's authorisation layer. File mode, ACL, SELinux, and network controls still apply. A remotely reachable service also needs a deliberate firewalld rule, such as a custom service definition or an approved port rule. Opening the firewall without configuring Apache and SELinux does not complete the change.

Apache also needs suitable discretionary permissions and SELinux labels for a custom document root. A new tree under `/var/test_www` can reuse the standard `/var/www` mappings:

```shell
mkdir -p /var/test_www/html
semanage fcontext -a -e /var/www /var/test_www
restorecon -Rv /var/test_www
```

Apache configuration must set the new `DocumentRoot` and an appropriate `<Directory>` block. The directory path must allow traversal, and content files must allow the Apache worker to read them. Static content normally receives `httpd_sys_content_t`. Only directories that the application must modify should receive a writable type such as `httpd_sys_rw_content_t`.

After the administrator changes the port, document root, and labels, the service should remain in enforcing mode during final verification. A local request can test both HTTP and content access:

```shell
curl -I http://localhost:3131/
```

A bind failure points towards configuration, port ownership, privileges, or a port-label denial. An HTTP 403 response points towards Apache authorisation, path permissions, ACLs, or file labels. Audit evidence distinguishes these cases and supports the narrowest effective correction.
## Layered operational validation
Security changes should pass functional, negative, persistence, and recovery tests. A functional test confirms that the intended account or service can complete its approved action. A negative test confirms that other identities remain blocked. A persistence test reloads the service or reboots the host, then verifies effective settings. A recovery test proves that console access, a retained session, or a documented rollback can restore service after an error.

Administrators should record the configuration change, expected security effect, validation evidence, rollback command, and responsible owner. Configuration management should apply SSH settings, ACLs, SELinux customisations, and service configuration consistently across equivalent hosts. Monitoring should detect changed host keys, failed authentication, unexpected ACL expansion, permissive SELinux operation, repeated AVC denials, and unapproved listening ports.

Backups must preserve the metadata required by the recovery design. File data alone does not preserve ACLs, local SELinux policy customisations, ownership, or service configuration. Restoration testing should confirm effective access after recovery, not only the presence of files.