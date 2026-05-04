# Authentication Issues
## PAM and authentication faults
Pluggable Authentication Modules, or PAM, control authentication, account checks, password updates, and session setup. PAM configuration lives under `/etc/pam.d`. Each service file contains ordered rules. A rule includes a module type, control value, module path or name, and module arguments.

The four major PAM module types are:
- `auth`, which authenticates credentials.
- `account`, which checks account validity and access.
- `password`, which manages password changes.
- `session`, which sets up and tears down session state.

Control values determine how PAM reacts to success or failure. Common values include `required`, `requisite`, `sufficient`, and `optional`. More detailed bracket syntax can make decisions based on exact return codes. A missing, reordered, or damaged PAM rule can block a service even when users and passwords are correct.

RHEL 8 uses authselect rather than the old authconfig workflow. Authselect manages selected authentication profiles and generates related PAM configuration. Manual changes to generated files may be overwritten or may place the system outside a supported profile. `authselect current` shows the selected profile. `authselect list` lists available profiles. `authselect select PROFILE` changes profile. `authselect apply-changes` applies updates.

A service-specific PAM problem often shows in that service's file under `/etc/pam.d`. If a package-owned PAM file has been damaged, move the broken file aside and reinstall the owning package. For example, a damaged Samba PAM file can be restored by reinstalling the relevant Samba package after preserving the faulty file for comparison. Then restart the service and retest authentication with the client utility.

Authentication failures should be split into identity, authentication, account authorisation, and session setup. A user absent from `getent passwd` has an identity lookup problem. A user present in `getent passwd` but unable to authenticate may have a password, Kerberos, PAM auth, or SSSD issue. A user who authenticates but cannot access a service may fail account rules. A user who starts login but receives a session error may hit session modules, home directory problems, SELinux, or resource limits.

PAM order matters because modules run in sequence. A `sufficient` success can short-circuit later modules when no prior required module has failed. A `required` failure may allow later modules to run but still fail the stack. A `requisite` failure stops immediately. These control values explain why adding one misplaced line can change the whole authentication result.

Authselect should be treated as the owner of generated authentication configuration. If a task requires SSSD, select an SSSD-capable profile. If local custom PAM changes are required, use an authselect custom profile rather than editing generated files in a way that future authselect operations overwrite. In a repair scenario, comparing generated files to the active profile can reveal manual damage.