# Managing Users and Groups
## Managing Linux Users
### User account data
Linux stores basic account information in /etc/passwd. The file contains usernames, user IDs, group IDs, home directories and default shells. Modern systems do not store passwords in this file. Instead, an x in the password field indicates that the encrypted password resides in /etc/shadow.

Administrators commonly inspect account data using:
- cat /etc/passwd to view entries
- grep to locate specific users
- getent passwd to query all configured identity sources

The getent command is preferred because it honours the Name Service Switch configuration and can include directory services such as LDAP or Active Directory.
### Filtering account information
Field extraction tools simplify analysis of passwd data.

- cut selects specific colon-delimited fields
- awk performs pattern matching and formatted output

Awk is more flexible and can replace multi-command pipelines when complex filtering is required.
## Creating Users
### Basic user creation
The useradd command creates new accounts. Running useradd username applies system defaults automatically. After creation, the id command confirms:
- assigned user ID
- primary group
- supplementary groups

RHEL 8 uses user private groups by default, meaning each new user receives a matching group.
### Default settings
User defaults originate from several locations:
- useradd -D displays command defaults
- /etc/default/useradd stores configurable values
- /etc/login.defs controls broader account behaviour

Important login.defs settings include:
- CREATE_HOME to automatically create home directories
- USERGROUPS_ENAB to enable private groups

Understanding these files allows administrators to standardise account creation across systems.
### Creating users with custom options
Useradd supports extensive overrides, for example:
- -N disables private group creation
- -g sets the primary group
- -G adds supplementary groups
- -c sets the GECOS comment field
- -M prevents home directory creation

These options allow bulk provisioning workflows and specialised account setups.
## Modifying and Deleting Users
### Modifying accounts
The usermod command adjusts existing users. Most useradd options also apply here. A critical flag is -aG, which appends supplementary groups rather than replacing them. Omitting -a can unintentionally remove existing memberships.

Typical modifications include:
- changing the primary group
- updating the comment field
- adding administrative group membership
### Removing accounts
The userdel command deletes users.

- userdel username removes the account only
- userdel -r username also removes the home directory, mail spool and crontab

Administrators often inspect orphaned files after deletion to ensure no residual ownership issues remain.
## Understanding Linux Passwords
### Shadow file purpose
Passwords and ageing data reside in /etc/shadow. This design improves security because the shadow file is readable only by privileged users. It also allows multiple ageing fields that the passwd file cannot support.

Key shadow fields include:
- encrypted password
- last password change
- minimum age
- maximum age
- warning period
- inactivity period
- account expiry

If the password field begins with an exclamation mark, the account is locked.
### Viewing ageing information
Administrators can read shadow data directly as root or use:
- chage -l username

The chage output presents human-readable ageing details and is safer than parsing raw shadow entries.
## Password Ageing Policies
### Default ageing controls
Password policy defaults come from /etc/login.defs. Important variables include:
- PASS_MAX_DAYS
- PASS_MIN_DAYS
- PASS_WARN_AGE

Changes affect only newly created users. Existing accounts retain previous settings unless modified manually.
### Adjusting ageing for users
The chage command modifies per-user policy. Common options include:
- -M maximum days between changes
- -m minimum days
- -W warning period
- -E account expiry
- -I inactivity period

These controls enforce organisational password standards.
## Password Authentication Mechanics
### Hash structure
Linux passwords are one-way hashes rather than reversible encryption. The shadow password field contains:
- hashing algorithm identifier
- salt value
- resulting hash

RHEL 8 typically uses SHA-512, represented by the value 6.
### Role of the salt
The salt is random data combined with the plaintext password before hashing. Its purpose is to ensure identical passwords produce different hashes. This prevents attackers from identifying shared passwords across accounts.

Authentication works by hashing the supplied password with the stored salt and comparing the result with the saved hash.
### Verifying hashes
Administrators can demonstrate the process using openssl passwd with the same algorithm and salt. Matching hashes confirm correct authentication behaviour.
## Managing User Passwords
### Using passwd
The passwd command is the standard tool for setting passwords. Behaviour differs by privilege level:
- normal users change only their own password
- root can reset any account

Useful passwd options include:
- -S to display account status
- -l to lock an account
- -u to unlock an account
### Non-interactive password changes
For automation, administrators avoid manual prompts.

On Red Hat systems:
- passwd --stdin reads the password from standard input

Cross-platform method:
- chpasswd reads username:password pairs from standard input or a file

This approach supports bulk updates and scripting.
### Locking and unlocking accounts
Account locking prepends an exclamation mark to the shadow password field. The user cannot authenticate with a password until the account is unlocked. This is useful for temporary suspensions.
## System Accounts
### Creating system users
System services often require non-login accounts. Use:
- useradd -r username

System accounts receive lower user IDs and different default ageing behaviour.
### Adjusting ageing for system accounts
If required, administrators can normalise settings using chage. Many environments disable ageing for service accounts to prevent unexpected lockouts.
## Managing Groups in Linux
### Group database
Group information resides in /etc/group. Each entry contains:
- group name
- group password placeholder
- group ID
- member list

Primary group membership is recorded in /etc/passwd, while supplementary memberships appear in /etc/group.
### Primary vs secondary groups
The primary group determines default group ownership of new files created by the user. Secondary groups grant additional access permissions without affecting file ownership defaults.

Correct group design is essential for secure file sharing.
### Adding users to groups
Administrators commonly use:

```Bash
usermod -aG group username
```

The append flag is critical to avoid overwriting existing memberships.
### Using gpasswd
The gpasswd command manages group administration. It can:
- assign group administrators
- add or remove members
- set group passwords

Group passwords generally reduce security and are rarely recommended in modern environments.
## Best Practice Summary
Effective RHEL 8 identity management relies on consistent use of core tools and configuration files. Administrators should:
- understand the separation between passwd and shadow
- use getent for reliable identity queries
- manage defaults through login.defs and useradd configuration
- enforce password ageing with chage
- prefer append operations when modifying group membership
- automate password changes with chpasswd where appropriate
- review orphaned files after user removal

Applying these practices improves security, auditability and operational reliability across Linux systems.
