# Application Issues
## Application library and memory faults
Third-party and first-party programs depend on shared libraries. When a binary fails to start, the administrator should check whether required libraries exist, whether the dynamic linker can find them, and whether a package must be installed or restored.

`ldconfig -p` lists libraries in the linker cache. `ldd /path/to/program` lists shared library dependencies for a binary and reports missing libraries. A missing library may require installing the package that owns it, refreshing the linker cache with `ldconfig`, correcting a library path, or replacing a damaged file.

Memory leaks require observation over time. Valgrind provides a direct way to analyse memory use for a process during a test run. `valgrind PROGRAM` runs the default memcheck tool. `valgrind --tool=memcheck PROGRAM` makes the tool explicit. `valgrind --leak-check=full PROGRAM` performs fuller leak analysis. `valgrind --log-file=FILE PROGRAM` writes results to a file. Valgrind slows execution, so it suits reproduction and analysis more than normal production operation.

Tracing tools reveal what an application does before it fails. `ltrace` traces library calls. `strace` traces system calls. Both support similar workflows: `-o FILE` writes output, `-p PID` attaches to a running process, and `-e EXPRESSION` filters events. Use `ltrace` when library interaction seems relevant, and use `strace` when file access, permissions, signals, sockets, or kernel calls seem relevant.

Application debugging should start with the smallest reproducible command. If a program fails only under systemd, compare the systemd environment, working directory, user, SELinux context, limits, and permissions with a manual run. If it fails manually as the same user, tracing and library checks become simpler.

Shared library errors may also come from architecture mismatch or wrong library versions. A binary built for one architecture or linked against a missing soname will not run merely because a similarly named package exists. `file PROGRAM` identifies the binary type. `rpm -qf LIBRARY` identifies the owning package for installed libraries. `dnf provides '*/LIBRARY'` locates a package for a missing library path or soname.

Tracing output can be large. Filters make it useful. For file and permission problems, `strace -e trace=file PROGRAM` can reveal missing paths and denied access. For network problems, socket-related traces may show connection attempts. For library behaviour, `ltrace` can show repeated calls or failed library functions. The trace should answer the hypothesis rather than collect every possible event.
## SELinux context and policy faults
SELinux enforces access through labels, policy, booleans, and port mappings. A service can have correct Unix permissions and still fail when SELinux blocks access. Diagnosis should prove SELinux involvement before changing policy.

The `ls -Z` option shows SELinux context. A context contains user, role, type, and level. The type field drives many access decisions. For example, Apache configuration files, log files, and web content use different expected types. Copying or moving files into service paths can leave them with unsuitable labels.

Useful SELinux discovery commands include:
- `semanage login -l` to list SELinux user mappings.
- `semanage user -l` to list SELinux users and roles.
- `semanage fcontext -l` to list file context rules.
- `semanage port -l` to list labelled ports.
- `seinfo -u`, `seinfo -r`, `seinfo -t`, and `seinfo -p` for policy information where setools is installed.

The `chcon` command changes a label immediately, but it does not change the persistent labelling rule. It suits tests. The `restorecon -Rv PATH` command restores labels from policy and persistent file context rules. Persistent custom file labels use `semanage fcontext -a -t TYPE 'PATH_REGEX'` followed by `restorecon`.

SELinux troubleshooting starts with proof. Temporarily setting permissive mode with `setenforce 0` and repeating the failing action can show whether SELinux caused the block. Restore enforcing mode with `setenforce 1` after the test. Do not leave the system permissive as a fix.

SELinux denial evidence appears in `/var/log/audit/audit.log` when auditd is active. `ausearch -m avc -ts recent` finds recent Access Vector Cache denials. `sealert -a /var/log/audit/audit.log` gives interpreted messages and often suggests commands. Apply suggestions only after confirming they match the service and path.

SELinux booleans enable supported policy behaviour without custom policy. `getsebool -a` lists booleans. `getsebool httpd_enable_homedirs` checks one boolean. `setsebool httpd_enable_homedirs on` changes it at runtime. `setsebool -P httpd_enable_homedirs on` makes it persistent. Booleans are the right fix when the policy already supports the behaviour, such as allowing httpd to serve content from home directories.

A common SELinux pattern involves web content outside the default document root. The service may need the correct file type, suitable Unix permissions, and an enabled boolean. Changing only one layer can leave the service broken. For Apache serving content from a home directory, the administrator must consider the content label, directory execute permissions, and the `httpd_enable_homedirs` boolean.

Another common pattern involves ports. A service configured to listen on a non-standard port may fail under SELinux even when the firewall allows the port. `semanage port -l` shows which types own which ports. If policy supports the service on that port type, `semanage port -a -t TYPE -p tcp PORT` adds a persistent mapping. If the port already exists under another type, use `-m` to modify it only when that change is correct.

SELinux denials can include suggestions that are technically valid but conceptually too broad. Generating a custom policy from every denial may grant more access than needed. Prefer the smallest supported fix: restore labels, set a documented boolean, or add a proper file context or port mapping. Custom policy belongs after those options fail and after the denial clearly matches the intended behaviour.