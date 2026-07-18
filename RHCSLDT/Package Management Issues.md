# Package Management Issues
> [!NOTE]
> Explains how to diagnose and recover RHEL 8 package-management problems by resolving dependency and version conflicts, repairing corrupted RPM databases, and verifying or restoring altered package files safely.

## Resolving RHEL package management issues
Red Hat Enterprise Linux 8 systems use RPM as the low-level package manager and DNF as the main dependency-aware front end. RPM stores installed package metadata in the RPM database and uses that metadata to install, query, verify, update and remove packages. DNF reads repository metadata, resolves dependencies, selects suitable package versions and then uses RPM to carry out the transaction.
### Diagnose dependency and version problems
Administrators should inspect package versions, dependencies and file ownership before changing packages. Useful commands include:
- `dnf list <package> --showduplicates` to display available versions from enabled repositories.
- `dnf deplist <package>` or `dnf repoquery --deplist <package>` to inspect dependency requirements and providers.
- `dnf provides <path-or-command>` to identify which package supplies a file, command or capability.
- `dnf downgrade <package>` to move to the highest installable lower version when a downgrade is necessary and supported.
- `dnf check --dependencies --duplicates --provides` to identify local package database problems.

A version lock can prevent an expected upgrade. Install the plugin with `dnf install 'dnf-command(versionlock)'`, then use `dnf versionlock list`, `dnf versionlock add <package>`, `dnf versionlock delete <package>` or `dnf versionlock clear`. Version locks affect transactional operations, not informational commands such as `list`, `info` or `repoquery`.
### Recover a corrupted RPM database
A corrupted RPM database often appears when `rpm` or `dnf` commands fail, hang or report errors such as `cannot open Packages database in /var/lib/rpm`. On RHEL 8 and earlier, the database uses Berkeley DB files under `/var/lib/rpm`.

Use a cautious recovery sequence:
1. Stop package-management activity and check for open RPM files with `lsof /var/lib/rpm/*`.
2. Remove stale lock files only after confirming no package process still needs them: `rm -f /var/lib/rpm/__db*`.
3. Back up the database directory: `cp -a /var/lib/rpm /var/lib/rpm.old`.
4. Verify the Packages file: `cd /var/lib/rpm && /usr/lib/rpm/rpmdb_verify Packages`.
5. If verification fails, rebuild the file from a dump: `/usr/lib/rpm/rpmdb_dump Packages > Packages.dump`, then `mv Packages Packages.bak`, then `/usr/lib/rpm/rpmdb_load Packages < Packages.dump`.
6. Verify the rebuilt Packages file with `rpmdb_verify`.
7. Rebuild the database indexes with `rpm -vv --rebuilddb`.

This process protects the original files before repair and confirms each recovery step before the administrator resumes package work.
### Track and restore changed package files
RPM verification compares installed files with the package metadata recorded at installation. `rpm -V <package>` checks one package, `rpm -Va` checks all installed packages and `rpm -qf <path>` identifies the package that owns a file.

Verification output uses letters to show changed attributes. Important indicators include:
- `S` for file size.
- `M` for mode, including permissions and file type.
- `5` for checksum or content changes.
- `L` for symbolic link changes.
- `U` for owner changes.
- `G` for group changes.
- `T` for modification time.
- `?` when RPM cannot read the file.

A `c` marker identifies a configuration file. Changed configuration content may be intentional, so administrators should assess it before overwriting it.

Use `rpm --setperms <package>` to restore packaged permissions and `rpm --setugids <package>` to restore packaged ownership. Use `dnf reinstall <package>` when package files need a clean replacement from the repository. DNF reinstall does not blindly replace modified configuration files, so administrators should review `.rpmnew` or `.rpmsave` files and merge configuration changes deliberately.