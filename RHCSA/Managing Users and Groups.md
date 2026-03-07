## RHEL user, group and privilege management
RHEL manages local access through four linked areas: user accounts, password policy, group membership and delegated privilege. The system stores identity data in plain text databases, applies account defaults from configuration files, and enforces access rules through ownership, group membership and `sudo`. Effective administration depends on reading those data stores accurately, choosing the right command for each task and avoiding shortcuts that weaken security.

The administrative model is simple in structure but detailed in practice. A user record points to a primary group and a login environment. Password data sits in a separate protected file. Supplementary groups extend access to shared resources. `sudo` overlays role-based control so staff can perform specific privileged tasks without a shared root password. Each layer solves a different problem, and each layer has its own files, commands and risks.
## Core account files
| Path | Purpose |
| --- | --- |
| `/etc/passwd` | Stores account names, UIDs, primary GIDs, home directories and login shells |
| `/etc/shadow` | Stores password hashes and password ageing data |
| `/etc/group` | Stores group names, GIDs and supplementary group members |
| `/etc/gshadow` | Stores protected group data, including group passwords and group administrators |
| `/etc/default/useradd` | Stores default settings used by `useradd -D` |
| `/etc/login.defs` | Stores broader login and password defaults such as home directory creation and password ageing |
| `/etc/skel` | Provides the template files copied into a new home directory |
| `/etc/sudoers` and `/etc/sudoers.d/` | Store delegated privilege rules for `sudo` |

Name lookups do not rely only on local files. `getent` queries the Name Service Switch, which may read local databases, SSSD-backed sources or external directory services, depending on `nsswitch.conf`. That distinction matters because a local `grep` only sees flat files, while `getent` can return the effective account or group view used by the system.

A practical rule follows from that design. Flat files show raw local records. `getent` shows what the system actually resolves. On a stand-alone host the results may match. On a centrally managed host they may differ significantly.
## Listing and interpreting users
Administrators inspect accounts first through `/etc/passwd`. Each entry records the login name, a placeholder in the password field, the numeric UID, the primary GID, the account comment field, the home directory and the login shell. Modern Linux systems keep password hashes out of `/etc/passwd` and store them in `/etc/shadow`, which keeps sensitive data out of a world-readable file.

A short inspection sequence covers most account checks.

```text
getent passwd
getent passwd vagrant
cut -d: -f1,3 /etc/passwd
awk -F: '/^vagrant:/ {print $1, $3}' /etc/passwd
```

`grep`, `cut` and `awk` remain useful for fast inspection. `cut -d: -f1,3 /etc/passwd` extracts usernames and UIDs. `awk -F: '/^name:/ {print $1, $3}' /etc/passwd` performs the same job in one process and scales better when filtering or reformatting larger datasets.

`getent passwd username` improves on direct file parsing because it returns the account record as the operating system resolves it. That method suits mixed environments that combine local users with LDAP or Active Directory. It also avoids guessing whether a missing result reflects an absent account or a lookup path that bypasses `/etc/passwd`.

Primary and supplementary groups need clear separation. The primary group appears in field four of `/etc/passwd` and determines the default group ownership of newly created files. Supplementary groups appear in `/etc/group` and extend access to shared resources. They are not the same thing, and they are not stored in the same place.

That separation affects troubleshooting. A user can appear to belong to a group in `id` output because the group is primary, while `/etc/group` shows no member list for that same group. The records are still correct. They simply live in different databases.
## Creating user accounts
`useradd` creates local accounts. With default RHEL settings, a new user usually receives the next available UID, a private primary group with the same name as the account, a home directory under `/home`, and `/bin/bash` as the login shell. The `id` command confirms the resulting UID, primary GID and supplementary memberships.

Several files shape that behaviour.

- `useradd -D` displays the default `useradd` settings
- `/etc/default/useradd` holds defaults such as the base home path and default shell
- `/etc/login.defs` influences broader behaviour such as `CREATE_HOME` and `USERGROUPS_ENAB`
- `/etc/skel` supplies the initial contents of a new home directory

Private user groups are enabled by default in typical RHEL setups. As a result, `useradd` may report `GROUP=100` in its defaults while still creating a unique primary group for each new account when `USERGROUPS_ENAB` is enabled. The defaults and the policy file work together, so both need inspection before changing account creation behaviour.

Non-default creation options let administrators shape accounts at the point of creation.

- `-c` sets the comment field, often used for a full name
- `-g` sets a specific primary group
- `-G` assigns one or more supplementary groups
- `-M` avoids creating the home directory
- `-m` forces home directory creation
- `-N` disables private group creation and uses the configured default group instead
- `-r` creates a system account from the system UID range

A typical creation workflow looks like this:

```text
useradd user1
id user1
getent passwd user1

useradd -N -g users -G wheel -c "User Two" user2
id user2
getent passwd user2
```

The first account follows system defaults. The second account overrides them by using the shared `users` group as the primary group, joining `wheel` as a supplementary group and writing a comment field. That pattern matters in environments that separate ordinary staff accounts from service roles or operational teams.

System accounts differ from ordinary login accounts because they generally serve services rather than people. They normally draw from a lower UID range and often require different password and expiry policies. Administrators should create them intentionally and avoid treating them as interchangeable with human login accounts.

The home directory policy also deserves deliberate control. `-M` suppresses immediate home directory creation, which can help when provisioning many accounts at once or when home directories are generated on first login through PAM-based tooling. `-m` forces home directory creation when the default policy would otherwise skip it. In both cases, `/etc/skel` still defines the template content when a home directory is actually created.
## Modifying and deleting users
`usermod` changes existing accounts with options that mostly mirror `useradd`. It can update the comment field, switch the primary group, adjust the shell or move the account into new supplementary groups.

The most important group option is `-aG`. `-G` alone replaces the current supplementary group list. `-aG` appends one or more groups to the existing list. Administrators who forget `-a` often remove required access by accident.

```text
id user1
usermod -c "User One" -g users -aG wheel user1
id user1
```

That sequence confirms the original state, applies the change and verifies the result immediately. It also makes the change visible in a shell transcript or change record.

`userdel` removes accounts. Without `-r`, the command deletes the account record but leaves the home directory and mail spool behind. That can preserve data for review, but it also leaves orphaned files. `userdel -r` removes the home directory and mail spool as part of the deletion. A cautious process checks for files still owned by the deleted UID before final cleanup.

A sensible lifecycle therefore separates identity removal from data retention. Temporary retention may suit legal review, handover or incident response. Immediate cleanup may suit temporary lab accounts or service accounts with no retained business value. The command choice should match that purpose.
## Password storage and ageing
`/etc/shadow` stores password hashes and account ageing data in a protected file that ordinary users cannot read. A typical entry contains the login name, password hash, last password change date, minimum password age, maximum password age, warning period, inactivity period and optional account expiry date.

The most important shadow fields are these:

| Field | Meaning |
| --- | --- |
| 1 | Login name |
| 2 | Password hash or lock marker |
| 3 | Last password change in days since 1 January 1970 |
| 4 | Minimum days before another password change |
| 5 | Maximum days before password expiry |
| 6 | Warning days before expiry |
| 7 | Inactive days after expiry before disabling password login |
| 8 | Account expiry date in days since 1 January 1970 |

Several of those fields matter operationally.

- A password field that starts with `!` indicates a locked password
- The date fields use days since 1 January 1970
- Minimum age restricts how quickly a user can change a password again
- Maximum age defines when a password expires
- Warning age controls how many days before expiry the system starts warning the user
- Inactivity counts the days after password expiry before the account is disabled for password authentication
- Account expiry disables the account itself on a fixed date

`chage -l username` presents the same ageing data in a more readable format. It is safer and faster than manually interpreting raw shadow fields during routine administration.

Defaults for new accounts come from `/etc/login.defs`. Settings such as `PASS_MAX_DAYS`, `PASS_MIN_DAYS` and `PASS_WARN_AGE` affect accounts created after the change. They do not retroactively rewrite existing shadow entries, so current accounts need separate updates through `chage`.

A clean policy workflow uses both levels:

```text
grep '^PASS_' /etc/login.defs
chage -l user1
chage -M 45 -W 7 user1
```

The configuration file defines the baseline for future accounts. `chage` adjusts the actual record for an existing account. Confusing those two layers leads to inconsistent password policy across the host.
## Setting passwords and understanding authentication
`passwd` remains the standard interactive password tool. Root can set another user’s password without knowing the existing one. Ordinary users can change only their own password and normally need to enter the current password first.

RHEL-based systems also support `passwd --stdin`, which reads a password from standard input. That is convenient in controlled scripts, though it still requires careful handling to avoid exposing secrets in shell history or process listings. `chpasswd` provides a broader non-interactive method and accepts `user:password` pairs from standard input. That approach suits batch resets across multiple accounts.

```text
echo 'Password1' | passwd --stdin user1
echo 'user1:Password1' | chpasswd
printf 'user1:Password1\nuser2:Password2\n' | chpasswd
```

`passwd -S username` reports password status. `passwd -l username` locks password authentication for an account, and `passwd -u username` unlocks it. Those actions affect password use, not every possible authentication method. An expired or locked password does not necessarily disable SSH keys or Kerberos if those methods are configured separately.

Linux does not store recoverable passwords. It stores one-way hashes. The hash field embeds the algorithm identifier, the salt and the resulting hash value. During authentication, the system combines the entered password with the stored salt, applies the same hashing algorithm and compares the result with the stored hash. Matching hashes confirm the password without revealing the original text.

That model explains two important security properties. First, administrators cannot read a user’s original password from `/etc/shadow`. Second, two users with the same password can still have different stored hashes because each account uses a different salt.

A simplified hash breakdown looks like this:

```text
$6$salt$hash
```

In that structure, `6` identifies SHA-512, the middle field stores the salt and the last field stores the resulting hash. The exact format can vary by algorithm, but the principle remains the same. Authentication compares computed hashes rather than decrypting stored passwords.

System accounts may need separate treatment. A service account created with `useradd -r` may not need forced password rotation, interactive login or any password at all. Applying human password policy to service identities without review can break applications or create unnecessary maintenance work.
## Managing groups
`/etc/group` stores group names, GIDs and supplementary members. `groupadd` creates new groups, usually with the next available GID unless an explicit value is required. `usermod -aG group user` adds an existing user to a supplementary group. The change affects the account immediately at the database level, though a current login session may need a new login to pick up the new group set.

A short group workflow illustrates the pattern:

```text
groupadd marketing
usermod -aG marketing vagrant
id vagrant
```

If the current shell still shows the old group list, that does not mean the change failed. The account database is updated immediately, while the session’s supplementary groups are refreshed at login.

Primary group membership is less obvious than supplementary membership. A shared primary group may have no member list in `/etc/group` even when several users rely on it. To list those users, administrators match the group’s GID from `/etc/group` against field four in `/etc/passwd`. That method matters on systems that use a shared primary group such as `users` instead of private user groups.

The logic looks like this:

```text
gid=$(awk -F: '/^users:/ {print $3}' /etc/group)
awk -F: -v gid="$gid" '$4 == gid {print $1}' /etc/passwd
```

That sequence pulls the GID for the `users` group and then prints the accounts whose primary GID matches it. This is cleaner than guessing from names and more reliable than reading only the member list in `/etc/group`.

`gpasswd` complements `usermod` by focusing on the group side of administration.

- `gpasswd -a user group` adds a member to a group
- `gpasswd -d user group` removes a member
- `gpasswd -A user group` assigns a group administrator

Group administrators can manage membership for designated groups without receiving full root access. That model can suit departmental resources where a manager needs limited control over access.

`/etc/gshadow` stores protected group data, including group passwords, group administrators and the member list used for secured group operations. It is the group equivalent of `/etc/shadow`.

Group passwords deserve caution. A password on a group creates a self-service path into that group for anyone who knows the shared secret. That weakens accountability and usually weakens security. In most environments, direct membership management through `usermod` or `gpasswd` is safer than sharing a group password.
## Privilege escalation with `su`
`su` changes identity to another account, most commonly root. Without an explicit target, `su` switches to root. `su -` and `su -l` start a full login shell, while plain `su` starts a non-login shell.

That distinction matters. A non-login shell may leave environment variables and the current working directory largely unchanged. A login shell resets the environment more completely and changes to the target account’s home directory. Administrative work is generally more predictable with `su -`.

A simple comparison shows the effect:

```text
su
pwd
echo $USER

su -
pwd
echo $USER
```

The first form can leave the original user’s environment partly intact. The second form behaves like a proper login for the target account. For shell initialisation, mail paths and other environment-dependent behaviour, the login form is the safer choice.

`su` also has a structural drawback. It relies on the target account’s password. If multiple administrators need root access through `su`, they all need the root password. Shared root passwords are hard to audit, hard to rotate and risky when staff change. That weakness makes `sudo` the preferred model for most managed environments.

Root can also use `su - username` to test a new account without setting the user’s password first. That is useful for verifying home directory creation, shell selection and environment setup.
## Delegating privilege with `sudo`
`sudo` grants controlled privilege without handing out the root password. The main policy file is `/etc/sudoers`, and the preferred place for local custom rules is `/etc/sudoers.d/`. Drop-in files reduce the risk of local changes being lost during package updates and keep policy changes easier to review.

All edits should go through `visudo`. It opens the policy for editing and performs syntax checks before saving. A malformed `sudoers` file can break delegated access system-wide, so syntax validation is essential. Keeping an existing root shell open while editing provides a safe recovery path if a policy change fails.

`sudoers` rules can be narrow or broad. A rule can target an individual user or a group, restrict access by host, require or skip password prompts, and limit the allowed commands. Group names begin with `%`. A helpdesk rule, for example, can permit members of a helpdesk group to run `/usr/bin/passwd` for ordinary accounts while explicitly denying `/usr/bin/passwd root`.

A compact rule looks like this:

```text
%helpdesk ALL=(root) /usr/bin/passwd, !/usr/bin/passwd root
```

That policy grants one administrative task rather than broad root-equivalent power. It is easier to audit, easier to justify and much safer than a blanket `ALL=(ALL) ALL` rule for staff who only need password reset duties.

`sudo -l` shows the commands available to the current user under the active policy. `sudo` also caches credentials for a short period by default, which reduces repeated password prompts during a maintenance task. That convenience should not be confused with unrestricted access. The cache expires, and the command policy still applies on each invocation.

A practical privilege model therefore follows three principles.

- Give the minimum command set needed for the role
- Prefer group-based rules for teams that share the same duties
- Deny especially sensitive actions explicitly when a broader command pattern might otherwise allow them
## Environment handling and editor selection
`sudo` resets much of the calling environment by default. That behaviour protects the privileged session from untrusted variables, but it also means user-defined values such as `EDITOR` do not automatically carry across.

Administrators can work around that in two ways. `sudo -i` starts an interactive root shell, after which the root environment can be set directly. A cleaner long-term option is a `sudoers` default such as `Defaults env_keep += "EDITOR"`, which preserves the editor choice across `sudo` for authorised users. That allows `visudo` to open in the preferred editor without weakening the rest of the environment policy.

This pattern keeps the policy clear:

```text
Defaults env_keep += "EDITOR"
export EDITOR=nano
sudo visudo
```

The preserved variable changes only what the policy explicitly allows. It does not disable the broader environment reset, which remains valuable for security.
## Operational guidance
A reliable RHEL account management workflow follows a consistent order.

- Inspect identity data with `getent`, `id` and the account files before making changes
- Review `useradd -D`, `/etc/default/useradd` and `/etc/login.defs` before altering account creation policy
- Use `usermod -aG` when adding supplementary groups to an existing account
- Use `chage` to manage existing password ageing rather than assuming new defaults will apply retrospectively
- Avoid shared group passwords except in narrow legacy cases
- Prefer `sudo` over shared root passwords and maintain local policy in `/etc/sudoers.d/`
- Edit privilege policy only with `visudo`
- Leave a recovery root shell open while changing `sudo` rules

That workflow keeps the administrative model coherent. It starts with observation, uses the system’s own data structures rather than guesswork, applies the smallest effective change and verifies the result. On a busy RHEL host, that discipline prevents most avoidable account and privilege mistakes.