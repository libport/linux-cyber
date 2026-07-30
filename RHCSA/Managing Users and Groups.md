# Managing Users and Groups
> [!NOTE]
> A practical guide to securely managing RHEL users, groups, passwords, account lifecycles, and delegated privileges through accurate identity inspection and least-privilege administration.

Red Hat Enterprise Linux 8 controls access through user identities, group identities, authentication data, file permissions, and delegated administrative privileges. Sound administration keeps local identity records consistent, applies account policy deliberately, and grants each person or service only the access it requires.
## Identity databases and lookups
RHEL stores local identity data in four colon-delimited files.

| File | Purpose |
| --- | --- |
| `/etc/passwd` | Stores the login name, password placeholder, UID, primary GID, comment field, home directory, and login shell |
| `/etc/shadow` | Stores the password hash or lock marker, password-age values, and account expiry |
| `/etc/group` | Stores the group name, password placeholder, GID, and explicit supplementary members |
| `/etc/gshadow` | Stores group-password data, group administrators, and group members |

Regular users can read `/etc/passwd` and `/etc/group` because programs need to translate numeric IDs into names. Access to `/etc/shadow` and `/etc/gshadow` remains restricted because their hashes support offline password attacks if disclosed.

The Name Service Switch configuration in `/etc/nsswitch.conf` determines where applications obtain user and group records. A RHEL host can consult local files, the System Security Services Daemon (SSSD), systemd, and other configured sources. SSSD can connect the host to LDAP, Red Hat Identity Management, Active Directory, or Kerberos services.

`getent` queries the configured NSS databases and should take precedence over direct file searches when an account might come from a central service:

```console
$ getent passwd alice
$ getent group developers
$ id alice
$ groups alice
```

`getent passwd` without a key enumerates only sources that support and permit enumeration. SSSD commonly resolves a named remote user even when it does not enumerate every remote account. Administrators therefore should not treat an empty full listing as proof that no remote account exists.

The seven `/etc/passwd` fields are the login name, `x` password placeholder, UID, primary GID, GECOS comment, home directory, and login shell. The `x` directs password-aware tools to `/etc/shadow`. The primary GID controls the user's initial group and often influences the group ownership of files that the user creates. Directory setgid permissions can override that ownership behaviour for shared directories.

Administrators can filter resolved records without bypassing NSS. `cut` selects known fields, while `awk` can apply numeric or textual conditions:

```console
$ getent passwd alice | cut -d: -f1,3,4,6,7
$ getent passwd | awk -F: '$3 >= 1000 && $3 <= 60000 {print $1, $3}'
```

The first command prints Alice's login name, UID, primary GID, home directory, and shell. The second prints accounts in RHEL's default regular UID range from sources that allow enumeration. A direct command such as `grep '^alice:' /etc/passwd` examines only the local file and can miss a central identity. It can also find a different result from `getent` when NSS source order or account overrides apply.

Account-management commands lock and update related files consistently. Administrators should not edit `/etc/passwd`, `/etc/shadow`, `/etc/group`, or `/etc/gshadow` with an ordinary text editor. `vipw` and `vigr` provide locking when exceptional manual repair becomes necessary. `pwck` and `grpck` check structural consistency, although a clean structural check does not prove that account policy or ownership remains correct.
## Creating and maintaining users
`useradd` creates a local account. On a standard RHEL 8 host, it assigns the next available regular UID, creates a user private group with the same name, creates a home directory, copies initial files from `/etc/skel`, and assigns the configured shell.

```console
# useradd alice
# passwd alice
# id alice
# getent passwd alice
```

RHEL normally reserves UIDs and GIDs below 1000 for system users and groups. `/etc/login.defs` defines ranges such as `UID_MIN`, `UID_MAX`, `SYS_UID_MIN`, and `SYS_UID_MAX`. Administrators should avoid hard-coding IDs unless shared storage, containers, or cross-system consistency requires stable values.

Several sources control account defaults:
- `useradd -D` displays and changes selected defaults.
- `/etc/default/useradd` defines values such as the base home path, default shell, skeleton directory, and password-inactivity default.
- `/etc/login.defs` defines UID and GID ranges, password-age defaults, home creation, and user private group behaviour.
- `/etc/skel` supplies files and directories for a newly created home.

The `INACTIVE` value does not measure time since the user's last login. It specifies the grace period after password expiry before the account becomes inactive. `CREATE_HOME` controls default home creation, and `USERGROUPS_ENAB` helps control user private groups.

Command-line options override defaults for one account:

```console
# useradd -c "Carla Nguyen" -G developers,qa carla
# useradd -M batchrunner
# useradd -N -g users shareduser
# useradd -r -s /sbin/nologin serviceagent
```

`-c` writes the descriptive GECOS field. `-G` sets supplementary groups. `-M` suppresses home creation. `-N` suppresses creation of a user private group, while `-g` chooses an existing primary group. `-r` selects the system UID range. A system account does not receive a home directory unless the administrator also requests one. Service identities should normally use a locked password and a non-interactive shell unless the service design requires interactive authentication.

Other useful creation options include `-u` for an explicit UID, `-d` for a non-default home path, `-m` to require home creation, `-s` for a login shell, and `-e` for an account-expiry date. An administrator should confirm that an explicit UID or GID does not collide with a local, remote, container, or storage identity. Reusing an old numeric ID can silently give a new account access to files that still carry that ID.

The user private group's name commonly matches the login name, and its GID commonly matches the user's UID. Linux does not require the numeric values to match. The private group reduces accidental sharing in ordinary home directories while allowing administrators to create collaborative directories with group ownership, setgid inheritance, and an appropriate umask.

`usermod` changes an existing local account. The distinction between the primary group and supplementary groups is essential:

```console
# usermod -g developers alice
# usermod --append -G wheel,qa alice
# usermod -G developers,qa alice
```

`-g` changes the single primary group. `--append -G` adds groups without removing existing supplementary memberships. `-G` without `--append` replaces the entire supplementary group list. An omitted group therefore disappears from the user's supplementary memberships.

Running processes retain the credentials and group list that they received at session creation. A user normally needs to log out and back in before a changed membership affects a new session. An administrator can inspect the database immediately with `id alice`, but that output does not alter credentials in an existing shell.

Account renaming and home relocation require coordinated options. `usermod -l newname oldname` changes the login name but does not automatically rename the home directory. `usermod -d /home/newname -m newname` changes the configured path and moves the existing home content. Administrators must also review mail aliases, scheduled jobs, application configuration, SSH authorisation, SELinux contexts, and external identity references.

`userdel` removes an account record. `userdel -r` also removes the user's home directory and mail spool when the tool can identify them. It does not guarantee removal of every file, scheduled task, process, or resource owned by the UID. Before deletion, an administrator should record the UID, stop or reassign processes, locate files on relevant file systems, preserve business records, and decide whether another owner should receive them. Deleting the account before recording its UID can leave numeric ownership that is harder to interpret.

A staged departure process often locks access first, preserves the account while records transfer, terminates active sessions, and deletes the identity only after approval. This sequence gives administrators time to distinguish personal files from organisational records and to prevent a later UID reuse from inheriting abandoned ownership.
## Passwords, ageing, and account state
RHEL stores password verifiers as salted, one-way hashes rather than reversible passwords. A modular crypt string commonly has the form `$id$salt$hash`. In RHEL 8, `$6$` identifies SHA-512 crypt where that algorithm remains configured. The salt causes identical passwords to produce different stored strings, which prevents a simple comparison from revealing shared passwords. During authentication, the system applies the recorded algorithm and salt to the supplied password, then compares the resulting hash with the stored hash.

Some hash formats include parameters such as work factors, so scripts should not assume that every valid field has exactly three components. PAM and account-management configuration choose the format used for newly set local passwords. Changing that configuration does not automatically rehash existing passwords. The system normally upgrades a stored verifier only when a user changes the password or another authorised process resets it.

The nine `/etc/shadow` fields have distinct roles.

| Field | Meaning |
| --- | --- |
| 1 | Login name |
| 2 | Password hash, empty value, or lock marker |
| 3 | Date of the last password change, counted in days from 1970-01-01 |
| 4 | Minimum password age |
| 5 | Maximum password age |
| 6 | Warning period before password expiry |
| 7 | Grace period after password expiry before account inactivity |
| 8 | Account expiry date, counted in days from 1970-01-01 |
| 9 | Reserved field |

An exclamation mark at the start of field 2 locks the Unix password. It does not necessarily disable the account because SSH keys, Kerberos, or another authentication method can still succeed. Field 7 concerns time after password expiry, not time after the last login. Field 8 expires the account itself and blocks login independently of the password's age.

Root can inspect shadow records through NSS. A user can normally list that user's own ageing information:

```console
# getent shadow alice
$ chage -l "$USER"
```

`chage` applies per-account ageing and expiry values:

```console
# chage -m 0 -M 90 -W 14 alice
# chage -I 7 alice
# chage -E 2027-12-31 contractor
# chage -d 0 alice
```

The first command permits immediate password changes, requires a change after 90 days, and starts warnings 14 days before expiry. The second command allows a seven-day grace period after password expiry. The third expires a contractor account on a stated date. The fourth forces a password change at the next login.

Those numbers serve as syntax examples, not universal security settings. An organisation should follow its current risk, regulatory, contractual, and identity-provider requirements. A minimum age only delays rapid changes. It does not implement password history. PAM modules or a central identity service enforce password quality and reuse controls. On RHEL 8, PAM and `pam_pwquality` govern password strength through files such as `/etc/security/pwquality.conf`, rather than the obsolete `PASS_MIN_LEN` setting in `/etc/login.defs`.

Values such as `PASS_MIN_DAYS`, `PASS_MAX_DAYS`, and `PASS_WARN_AGE` in `/etc/login.defs` affect newly created local accounts. Changing them does not rewrite existing shadow records. Administrators use `chage` when an existing account needs revised values.

`passwd` changes a password and performs other password-state operations:

```console
# passwd alice
# passwd -S alice
# passwd -l alice
# passwd -u alice
```

`passwd -S` reports whether the account has a usable, locked, or absent Unix password and displays ageing values. `passwd -l` locks only the Unix password. `passwd -u` restores the previous hash when it can do so safely.

An ordinary user changes only that user's password and normally supplies the current password first. Root can reset another local user's password without knowing the old value. That reset can affect encrypted home directories, key rings, or applications that derived secrets from the previous login password, so an administrator should assess those dependencies before forcing a reset.

An administrator who must disable all login should use account expiry or another complete access-control procedure, then terminate active sessions and revoke other credentials where policy requires it:

```console
# usermod --expiredate 1 alice
# usermod --expiredate -1 alice
```

The first command places the account expiry in the past. The second removes that expiry. A complete offboarding process also addresses SSH keys, Kerberos tickets, API credentials, running jobs, delegated access, and centrally managed identities.

`chpasswd` can set passwords for several local users from `user:password` input, while RHEL's `passwd --stdin` can read one password from standard input. Automation must keep cleartext passwords out of command histories, process arguments, logs, source repositories, and long-lived files. A secret-management system or a controlled provisioning mechanism provides safer input than an exposed `echo` pipeline.

System accounts usually should not authenticate interactively. A locked password, a shell such as `/sbin/nologin`, restricted file permissions, and service-specific confinement reduce exposure. Password ageing provides little benefit to an identity that has no usable password. Administrators should rotate the actual credentials used by the service, such as keys, certificates, or tokens, under the controls designed for those credential types.
## Managing groups
A user has one primary group and can have several supplementary groups. `/etc/passwd` records the primary GID in field 4. `/etc/group` records explicit supplementary members in field 4 of the group entry. Consequently, an empty member list in `/etc/group` does not prove that no user has the group as a primary group.

RHEL's user private group model creates a group for each regular user. The user receives that group as the primary group, although the user's name need not appear in the member list in `/etc/group`. Administrators should describe this relationship as primary membership, not supplementary membership.

The following commands create a group and maintain memberships:

```console
# groupadd marketing
# usermod --append -G marketing alice
# gpasswd -a bob marketing
# gpasswd -d bob marketing
```

`groupadd` creates the local group. `usermod --append -G` adds the group to a user's existing supplementary list. `gpasswd -a` and `gpasswd -d` add or remove one user from one local group. These tools update local files and do not modify LDAP, Active Directory, or other remote groups.

`getent group marketing` shows the explicit supplementary member list returned through NSS. For a local or enumerable database, an administrator can identify users whose primary GID matches the group:

```console
$ gid=$(getent group marketing | cut -d: -f3)
$ getent passwd | awk -F: -v gid="$gid" '$4 == gid {print $1}'
```

The result depends on NSS enumeration. A central directory that disables enumeration requires a directory-specific query or a known user lookup.

`gpasswd -A alice marketing` assigns local group administration to Alice. A group administrator can manage that group's local membership without general root access. `/etc/gshadow` stores the administrator list, group-password data, and members. Delegation should match the sensitivity of the resources controlled by the group.

`gpasswd -M alice,bob marketing` replaces the complete local member list. As with `usermod -G`, replacement can remove access unintentionally. Add and delete operations provide safer changes when an administrator intends to alter one membership. After any change, `getent group marketing`, `id alice`, and a fresh login session provide complementary checks.

Group passwords weaken accountability because several people share one secret. When a non-member supplies a valid group password to `newgrp` or `sg`, the command changes group credentials for a new shell or command. It does not add a persistent member entry to `/etc/group`. Administrators should prefer explicit, attributable membership and should normally restrict or remove group passwords.
## Privilege elevation
UID 0 carries root authority. Administrators can change identity with `su` or execute authorised commands with `sudo`.

`su` without a target selects root and starts a non-login shell. It preserves much of the caller's environment and current directory. `su -` or `su -l` starts a login shell with the target user's login environment and home directory. An unprivileged caller normally supplies the target account's password. Root can use `su - alice` to test an account without Alice's password.

Sharing the root password weakens individual accountability and complicates offboarding. `sudo` normally authenticates the caller with the caller's own password, applies policy to the requested command, and records the caller's identity. It therefore supports granular delegation and stronger audit trails.

RHEL commonly grants broad administrative access through the `wheel` group:

```console
# usermod --append -G wheel alice
```

The default sudoers rule usually resembles `%wheel ALL=(ALL) ALL`. The percent sign identifies a group. `ALL` in the host position refers to the host on which `sudo` executes, not the workstation from which a person connected.

Administrators should place custom policy in files under `/etc/sudoers.d` and edit each file with `visudo`:

```console
# visudo -f /etc/sudoers.d/webops
# visudo -cf /etc/sudoers.d/webops
```

`visudo` checks syntax before installing an edit, and `-c` validates existing policy. Fragment names should not contain a period or end with a tilde because the default include rule can ignore such names.

A safe policy change keeps a verified administrative recovery path available until validation and delegated-user testing succeed. The administrator should check the whole policy with `visudo -c`, inspect the target account's view with `sudo -l -U alice`, and test each permitted and denied operation. Syntax validation catches parse errors but cannot establish that a rule grants the intended scope.

A sudoers rule identifies the subject, execution host, run-as identity, and permitted command:

```sudoers
%webops ALL=(root) /usr/bin/systemctl status httpd, /usr/bin/systemctl restart httpd
```

This rule lets members of `webops` run two exact operations as root on hosts that contain the rule. Administrators should specify absolute command paths and required arguments, then test the policy with `sudo -l` from the delegated account.

Rules should grant the smallest practical command set. Broad access to shells, editors, interpreters, package managers, file-copy tools, or commands with escape features can become equivalent to full root access. A broad allow rule followed by a negated exception can often be bypassed through another path or argument form. A positive allow list or a tightly controlled root-owned wrapper provides a safer design.

`NOPASSWD` removes an authentication barrier and requires a documented operational reason. Sudo's credential cache also affects reauthentication and uses a configurable timeout. A user can run `sudo -k` to invalidate the cached credential. `sudo -i` starts a root login shell and should remain limited to administrators who genuinely require unrestricted access.

Sudo resets much of the caller's environment by default. This behaviour reduces the risk that variables such as `PATH`, library paths, or language-specific settings alter a privileged program. `visudo` uses the editor allowed by sudoers policy. Administrators who require a standard editor can configure a fixed value such as `Defaults editor=/usr/bin/vim`. Retaining a user-controlled `EDITOR` variable through `env_keep` expands trust in the caller's environment and should follow an explicit policy decision.
## Verification and audit
Identity administration requires checks at both the database and session levels. `getent` confirms the record selected by NSS, `id` resolves current group assignments for a named account, and a fresh login proves that a new session receives the intended credentials. File ownership checks confirm that account changes did not strand important data under an obsolete UID or GID.

Periodic reviews should identify duplicate numeric IDs, unexpected interactive shells, missing or incorrectly owned home directories, expired temporary accounts, dormant privileged accounts, unnecessary supplementary groups, and sudo rules that exceed current duties. Central and local records also require comparison because a local account can shadow a remote name according to NSS order.

An administrator should retain evidence of the requested change, approval, commands, verification results, and rollback or recovery action. The record should distinguish a password lock from account expiry, a database membership from credentials held by an existing process, and a sudo syntax check from a successful authorisation test. These distinctions prevent a technically successful command from creating an incomplete security outcome.
## Administrative controls
- Administrators should resolve identities with `getent`, `id`, and `groups` before changing them.
- Account changes should preserve required files, UID and GID consistency, active workloads, and audit records.
- Password locks, password expiry, account expiry, and session termination should remain separate controls.
- Group changes should distinguish primary membership from supplementary membership.
- Sudo policy should use `visudo`, positive allow rules, absolute paths, exact arguments, and independent recovery access.
- Central identity systems should remain the system of record for remote users and groups.