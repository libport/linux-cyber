# Aiding Third-Party Investigations
## SSSD, LDAP, and Kerberos client configuration
LDAP and Kerberos identity problems on RHEL 8 normally involve SSSD and authselect. OpenLDAP server setup is not the focus. The system acts as a client that uses SSSD to reach identity and authentication services.

SSSD configuration lives in `/etc/sssd/sssd.conf`. The file must use correct permissions, normally root ownership and mode 600, because it can contain sensitive configuration. It defines services, domains, providers, LDAP URIs, TLS certificate paths, Kerberos realms, and related identity settings.

Common checks include:
- `systemctl status sssd` to confirm the daemon runs.
- `journalctl -u sssd` to inspect service errors.
- `sssctl config-check` where available to validate configuration.
- `getent passwd USER` to test identity lookup.
- `id USER` to test user resolution.
- `kinit USER` to test Kerberos where applicable.
- `authselect current` to confirm the selected profile matches SSSD use.

LDAP faults may involve DNS, TLS certificates, bind credentials, base DN, firewall access, or wrong provider settings. Kerberos faults may involve realm names, time synchronisation, KDC reachability, DNS records, or keytab issues. The administrator should validate identity lookup and authentication separately because one can work while the other fails.

SSSD caches identity information, which can confuse testing. A cached user may appear after the directory service becomes unavailable. Clearing cache is sometimes useful, but it should not replace fixing the connection. Logs under SSSD and the journal show provider, domain, TLS, and Kerberos errors. Debug levels can increase detail when ordinary logs do not explain the failure.

Time synchronisation is critical for Kerberos. A client with significant clock drift may resolve users through LDAP but fail Kerberos authentication. DNS also matters because realms, KDCs, and service discovery often rely on names. Therefore, an identity management fault may require checking `chronyd`, DNS, routes, certificates, and SSSD together.

Permissions on `/etc/sssd/sssd.conf` are a frequent source of failure. SSSD refuses unsafe configuration files. The file should be owned by root and readable only by root. After repair, restart SSSD and test identity lookup with commands that do not require a full login first. Then test authentication and service access.
## Kernel crash dumps
Kdump captures kernel crash dumps for later analysis. It reserves memory for a crash kernel, boots that kernel after a panic, and writes a vmcore. The administrator enables it with `systemctl enable --now kdump`.

The main configuration file is `/etc/kdump.conf`. The default dump location is usually under `/var/crash`. Configuration can direct dumps elsewhere, including local paths or remote targets. After changing configuration, restart kdump and confirm status.

Crash dumps support third-party investigation because they preserve kernel state at failure time. For safe testing, use a controlled virtual machine rather than a production host. Forced crashes can damage work and interrupt services.
## SystemTap modules
SystemTap instruments a running Linux system by compiling scripts into kernel modules or running probes through supported mechanisms. It helps gather diagnostic information for issues that ordinary logs and metrics do not expose.

SystemTap setup may require enabling debug repositories and installing packages that match the running kernel, including kernel debuginfo or kernel-devel packages where required. Version mismatch is a common source of failure.

Default tapsets live under `/usr/share/systemtap/tapset`. Not every `.stp` file is a standalone script. Some files provide reusable probe definitions for other scripts. The administrator must distinguish runnable scripts from library-style tapsets.

Important commands include:
- `stap SCRIPT.stp` to compile and run a script.
- `stap -v SCRIPT.stp` to show verbose compilation and run information.
- `stap -p4 -m NAME SCRIPT.stp` to stop after module generation and name the module.
- `staprun NAME.ko` or `staprun NAME` to run a compiled SystemTap module where appropriate.

A generated `.ko` file is a kernel module. Running it requires suitable permissions and compatible kernel support. If SystemTap reports missing kernel packages, install the exact package variant suggested for the running kernel. If a probe hangs or produces no output, confirm that the probed event actually occurs during the test.

Crash dump collection has two distinct goals. The first goal is readiness before a crash. The second goal is evidence after a crash. Readiness means kdump is enabled, has reserved memory, has a valid target, and starts cleanly. Evidence means the vmcore exists after a crash and can be handed to the team that will analyse it.

SystemTap should be used with the same restraint as any kernel-level diagnostic tool. A probe can generate overhead, produce large output, or fail when kernel symbols do not match. The administrator should run a narrow script, reproduce the event, capture the data, and stop the probe. This keeps investigation focused and reduces the risk of creating a new performance issue.

For EX342, the important SystemTap skill is operational. The candidate should know how to install the needed packages, recognise missing kernel support, locate scripts, run a script with `stap`, compile a module with `-p4 -m`, and run a compiled module with `staprun`. The candidate does not need to become a full SystemTap developer to gather useful diagnostic evidence.