# Package Management Issues
## Package management faults
Package dependency failures often come from disabled repositories, incompatible versions, modular streams, partial transactions, or strict dependency resolution. DNF and Yum provide options that help diagnose these failures without forcing unsafe changes.

Useful package commands include:
- `dnf repolist` to view enabled repositories.
- `dnf repolist all` to view disabled repositories too.
- `dnf clean all` and `dnf makecache` to refresh metadata.
- `dnf install PACKAGE --nobest` to allow a non-latest candidate where policy permits.
- `dnf install PACKAGE --skip-broken` to skip packages with unsatisfied dependencies.
- `dnf downgrade PACKAGE` to return to an earlier version.
- `dnf versionlock` where the plugin is installed and version pinning matters.

An RPM database fault can break queries and installs. The RPM database lives under `/var/lib/rpm`. Repair should start with caution: stop package activity, check for locks and open RPM files, back up the database files, verify the database, dump and reload when required, and rebuild. Tools under `/usr/lib/rpm` include `rpmdb_verify`, `rpmdb_dump`, and `rpmdb_load`. The rebuild command is `rpm -vv --rebuilddb`.

RPM verification detects changed files from installed packages. `rpm -V PACKAGE` reports altered size, permissions, ownership, group, checksum, modification time, capabilities, or link targets. `rpm --setperms PACKAGE` restores packaged permissions. `rpm --setugids PACKAGE` restores packaged ownership and group. `dnf reinstall PACKAGE` can replace damaged package files when manual restoration is slower or less reliable.

Dependency repair should avoid forcing broken packages into place. A forced RPM install can satisfy neither the package manager nor the service that needs the dependency. DNF's resolver output usually names the conflict, missing dependency, excluded architecture, disabled repository, or modular stream. The administrator should read that output before adding flags.

Repository faults often explain sudden dependency failures. The configured repository may be disabled, unreachable, mismatched with the RHEL release, or missing metadata. `dnf repolist -v` can reveal base URLs and metadata expiry. Network faults, proxy settings, subscription state, and wrong release versions can all surface as package problems. A package task may therefore require DNS, routing, certificate, or repository repair before DNF succeeds.

RPM verification symbols need careful interpretation. A changed configuration file is common and not automatically wrong. A changed binary, library, PAM file, unit file, or permission bit may be more suspicious. Restoring ownership and permissions is lower risk than replacing content. Reinstalling a package is appropriate when package-owned files are missing or damaged and local customisations are not required.

A corrupted RPM database repair should always start with a backup. Even when the exam objective expects a repair, a copy of `/var/lib/rpm` preserves a rollback point. After rebuilding, basic package queries such as `rpm -qa`, `rpm -q PACKAGE`, and `dnf history` should work again. Only then should the administrator continue with installs or reinstalls.