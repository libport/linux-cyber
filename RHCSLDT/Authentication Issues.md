# Authentication Issues
> [!NOTE]
> Explains how to troubleshoot RHEL 8 authentication by validating authselect profiles, PAM service rules, NSS sources, packaged configurations, and SSSD connections to LDAP and Kerberos providers.
## PAM and identity management troubleshooting in RHEL 8
Red Hat Enterprise Linux 8 manages most host authentication through authselect, Pluggable Authentication Modules (PAM), Name Service Switch (NSS), and the System Security Services Daemon (SSSD). Administrators should treat older authconfig habits as legacy practice. Authselect selects a tested profile and generates the PAM and NSS configuration that supports that profile.
### Authentication configuration
Authselect profiles describe how PAM and NSS should behave. The selected profile appears with `authselect current`. A new profile can be selected with `authselect select sssd`, and profile changes can be applied with `authselect apply-changes`.

Administrators should inspect `/etc/nsswitch.conf` to confirm which sources resolve users, groups, services, and hosts. When authselect manages the system, administrators should not edit `/etc/nsswitch.conf` directly. They should change the authselect profile or the appropriate authselect NSS template, then apply the profile changes.
### PAM service files
PAM stores per-service configuration in `/etc/pam.d/`. Each rule normally contains a PAM type, a control value, a module, and optional module arguments.

The four PAM types have distinct purposes:
- `auth` verifies identity and credentials.
- `account` checks whether the account may access the service.
- `password` manages password changes.
- `session` creates and closes the login session, including related limits, logging, and environment setup.

The control value tells PAM how to handle a module result. `required` records failure but continues through the stack. `requisite` stops immediately on failure. `sufficient` can allow success when earlier required modules have not failed. `optional` usually affects the result only when no stronger rule decides it. `include` and `substack` both pull in rules from another PAM file, but `substack` confines `done` and `die` actions to the included stack.
### Troubleshooting PAM faults
A broken PAM file can prevent a service from authenticating or authorising users. Logs such as `/var/log/secure` may help, but they can contain noisy or misleading authentication messages. Package verification often gives a clearer starting point when a packaged PAM file has changed.

For Samba, `obey pam restrictions = yes` tells Samba to honour PAM account and session management. It does not make PAM handle SMB password authentication when encrypted passwords are enabled. If `/etc/pam.d/samba` has lost required rules, administrators can preserve the damaged file for comparison, verify the package, and reinstall the package that owns the file. After restoration, they should retest with a known Samba user and remove the broken backup only after confirming the service works.
### SSSD, LDAP, and Kerberos
RHEL 8 commonly uses SSSD to connect a local client to remote identity and authentication providers, including LDAP directories and Kerberos realms. The main SSSD file is `/etc/sssd/sssd.conf`. Domain sections define providers and connection details such as LDAP URI, search base, TLS settings, Kerberos realm, and server names.

When LDAP or Kerberos authentication fails, administrators should verify that authselect uses the expected profile, that SSSD is active, that `/etc/sssd/sssd.conf` contains the correct domain values, and that NSS lists `sss` where remote identity lookup is required.