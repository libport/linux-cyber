## Secure Remote Access with SSH Keys
SSH key-based authentication replaces password login with a
cryptographic key pair. The administrator generates a private and public
key using ssh-keygen. The private key remains on the local system while
the public key is copied to the remote host using ssh-copy-id. The
remote host stores the public key in the user's authorized_keys file
inside the .ssh directory.

During the first connection the client records the server public key in
the known_hosts file. This step establishes trust between the client and
the server. Future connections validate the server identity against this
stored key.

Key authentication improves security because attackers cannot simply
guess passwords. Access requires possession of the private key.
Administrators often protect private keys with a passphrase that
encrypts the key file.
### Managing SSH Authentication
Common SSH configuration practices include the following.

-   Generate a key pair with ssh-keygen
-   Store the public key on the remote host in authorized_keys
-   Protect the private key with a passphrase
-   Verify host authenticity through the known_hosts file
### Improving SSH Usability
Repeated passphrase prompts can interrupt workflow. The ssh-agent
service resolves this issue by caching passphrases in memory. The user
starts the agent and loads the key with ssh-add. The cached key allows
repeated SSH connections without repeated passphrase entry during the
session.

Administrators can also define client behaviour in the \~/.ssh/config
file. This file stores host entries, identity files, and connection
options. It reduces typing and supports command line tab completion for
hostnames.

Example configuration options include identity file paths and connection
keepalive settings. These settings simplify remote administration while
maintaining security.
### Disabling Password Authentication
Systems can further strengthen SSH security by disabling password
authentication on the server. The sshd daemon configuration file
controls this behaviour at /etc/ssh/sshd_config.

Administrators verify current settings using sshd -T. To enforce key
authentication they change the parameter PasswordAuthentication to no.
Once applied the server accepts only key-based logins through SSH. Local
console authentication remains unaffected.

This approach prevents brute force password attacks and ensures that
only users with valid keys can access the system remotely.
## POSIX Access Control Lists
Traditional Linux permissions assign access to one owner, one group, and
others. This structure works well for simple systems but becomes
restrictive in enterprise environments where multiple users require
different levels of access to the same resources.

POSIX Access Control Lists extend the permission model by allowing
multiple users and groups to receive specific permissions on a single
file or directory. ACL entries can grant read, write, or execute access
without restructuring group membership.
### ACL Capabilities
ACLs provide several key capabilities.

-   Assign permissions to multiple users
-   Assign permissions to multiple groups
-   Define default permissions for new files within a directory
-   Provide more precise control than standard file modes

The tools used to manage ACLs are provided by the acl package. The
primary commands include setfacl for modifying ACL entries and getfacl
for displaying them.
### Default ACLs
Default ACLs apply to directories. When set on a directory they
automatically apply to newly created files and subdirectories within
that location.

This feature simplifies administration in environments such as web
servers. For example the Apache service user can receive read access to
all new content in the web root directory without manual permission
changes.

Administrators configure default rules with setfacl using the -d option.
Multiple rules can be defined in a single command separated by commas.
### Managing ACL Entries
Administrators use setfacl to add or modify entries and getfacl to
review them.

Common tasks include the following.

-   Add user permissions with setfacl -m
-   Add group permissions with setfacl -m
-   Remove entries with setfacl -x
-   Remove all ACLs with setfacl -b
-   Back up ACL definitions using getfacl output
-   Restore ACL definitions with setfacl --restore

A plus sign in file permission listings indicates that an ACL exists on
that file or directory.

ACL backups can be created by redirecting getfacl output to a text file.
Administrators later restore the configuration using the restore option
of setfacl. This method simplifies recovery after configuration changes.
## SELinux Mandatory Access Control
Security Enhanced Linux provides a mandatory access control system that
enforces policy restrictions beyond standard permissions and ACLs. Even
privileged users cannot override SELinux policy rules.

SELinux was developed by the United States National Security Agency and
integrated into enterprise Linux distributions including Red Hat
Enterprise Linux.
### SELinux Concepts
SELinux policies control how processes interact with files, devices, and
network resources. Each process and file receives a security context
that defines its allowed interactions. The policy engine evaluates these
contexts and determines whether access is permitted.

This system protects the operating environment from compromised
applications. Even if a service runs with elevated privileges SELinux
can restrict its behaviour.
### SELinux Operating Modes
SELinux supports three operational modes.

-   Enforcing
-   Permissive
-   Disabled

Enforcing mode actively applies policy rules and logs violations. This
is the default configuration in Red Hat Enterprise Linux.

Permissive mode does not block actions but records policy violations in
logs. Administrators often use this mode when diagnosing application
behaviour or adjusting security policies.

Disabled mode turns off SELinux completely. This removes enforcement and
logging and is not recommended because administrators lose visibility
into security violations.
### Viewing SELinux Status
Administrators check the current runtime mode using the getenforce
command. The sestatus command provides additional information including
policy type and configuration state.

The persistent configuration resides in the file /etc/selinux/config.
This file defines both the SELinux mode and the policy type used by the
system.
### Changing SELinux Modes
Administrators temporarily change modes using setenforce.

-   setenforce 1 sets enforcing mode
-   setenforce 0 sets permissive mode

These changes affect only the current runtime session. Persistent
changes require editing the configuration file followed by a system
reboot.

SELinux cannot be disabled through the runtime command. Full disablement
requires editing the configuration file and restarting the system.
### Policy Types
The default SELinux policy in most systems is targeted. This policy
protects key system services while leaving most user processes
unrestricted.

Other policy types include minimum and multi level security
configurations. The targeted policy remains the most common choice in
enterprise deployments because it balances protection and compatibility.
## Integrated Security Strategy
Secure Linux administration relies on multiple layers of protection.

SSH key authentication prevents password attacks and secures remote
access. ACLs extend file permissions to support complex user
environments. SELinux enforces strict policy rules that limit
application behaviour and contain potential compromises.

Together these mechanisms form a layered security model that protects
systems, data, and services in enterprise Linux environments.