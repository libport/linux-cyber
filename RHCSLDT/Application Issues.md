# Application Issues
> [!NOTE]
> Presents a systematic approach to diagnosing Linux application failures through shared-library inspection, memory analysis, runtime tracing, and SELinux troubleshooting while distinguishing administrator-remediable issues from code defects requiring developer action.
## Diagnosing Linux application issues
Linux administrators diagnose application failures by checking shared libraries, memory behaviour, trace output and SELinux policy. The work usually separates system faults from application faults. Administrators can repair missing packages, incorrect labels, port mappings and SELinux booleans. They usually report application memory leaks or code defects to the application owner with clear diagnostic evidence.
### Shared library dependencies
Applications often reuse shared libraries instead of carrying common functions in each binary. The runtime linker-loader, such as `ld-linux.so`, loads the required shared objects into a process when the program starts. `ldconfig` maintains the runtime linker cache, and `ldconfig -p` prints the candidate libraries stored in that cache. This cache is not a list of libraries currently running in memory.

Administrators inspect a trusted binary with `ldd` to print its shared object dependency tree:

```bash
ldd $(which httpd)
```

Missing libraries appear without a resolved path. After identifying the library name, `dnf whatprovides` can find the owning package. Quoting the wildcard prevents the shell from expanding it too early:

```bash
dnf whatprovides '*/liblzma.so*'
dnf reinstall xz-libs
```

`ldd` must not be run against untrusted executables because some implementations can execute code while resolving dependency information. Safer static inspection can start with `objdump -p /path/to/program | grep NEEDED`, although that shows only direct dependencies.
### Memory leak reports
Valgrind provides a suite of debugging and profiling tools. Its default tool, Memcheck, detects memory errors and leaks in native executables. A basic run identifies leaks when the program exits:

```bash
valgrind ./memleak_test_app
```

More useful reports add a full leak check and include all leak kinds:

```bash
valgrind --leak-check=full --show-leak-kinds=all ./memleak_test_app
```

Memcheck classifies leak results by severity. `definitely lost` and `possibly lost` need the closest attention. `still reachable` usually means the program retained allocated memory until exit, which may or may not indicate a real defect. Administrators who need to send evidence to developers can write the output to a file:

```bash
valgrind --leak-check=full --show-leak-kinds=all --log-file=memcheck ./memleak_test_app
```
### Runtime tracing
`ltrace` and `strace` help diagnose applications when source code is unavailable. `ltrace` runs a command and records the dynamic library calls and signals associated with that process. It can also print system calls. `strace` focuses on system calls and signals at the user-kernel boundary, so it often produces more detail than `ltrace`.

Both tools support practical filters:
- `-o file` writes trace output to a file.
- `-p pid` attaches to an existing process.
- `-e expression` limits the trace to selected calls.

For example, tracing `cat` against a test file can show calls that open, read and close the file. Filtering with `-e read` reduces noise:

```bash
ltrace -e read $(which cat) empty
strace -e read $(which cat) empty
```

Administrators often start with `ltrace` when library interactions seem likely, then move to `strace` when the failure involves files, permissions, sockets, signals or other kernel-facing operations. `strace` is a system-call trace, not a stack trace.
### SELinux context and policy checks
SELinux enforces mandatory access control through labels and policy. File and process labels contain four parts: SELinux user, role, type and level. In `system_u:object_r:httpd_log_t:s0`, the type `httpd_log_t` usually drives the access decision, while `s0` represents the MLS or MCS level in the targeted policy.

Administrators view labels with `ls -Z`, which comes from SELinux-aware core utilities rather than SETools. SETools supplies policy query tools such as `seinfo`, while `policycoreutils-python-utils` supplies `semanage` on RHEL 8. Useful queries include:

```bash
semanage login -l
semanage user -l
semanage fcontext -l
semanage port -l
seinfo -u
seinfo -r
seinfo -t
seinfo --portcon
```

`semanage port -l` shows SELinux port type mappings. For example, `http_port_t` covers the standard HTTP and HTTPS ports that policy already permits for Apache. Non-standard ports need an added mapping, such as `semanage port -a -t http_port_t -p tcp 3131`.

File labels also matter when services use non-standard paths. `chcon` can change a label immediately, but it does not change the persistent file context rules. Durable fixes use `semanage fcontext` followed by `restorecon`. `restorecon -Rv /var/log/httpd` restores labels under that path from policy and local file context mappings. Apache content outside `/var/www` should receive an appropriate HTTP content type, such as `httpd_sys_content_t`, before the service can read it under enforcing policy.
### SELinux troubleshooting sequence
Administrators first confirm whether SELinux causes the failure. Temporarily switching to permissive mode with `setenforce 0`, repeating the failed action, and returning to enforcing mode with `setenforce 1` can separate SELinux denials from ordinary Linux permissions, service configuration and application errors.

SELinux denials usually appear in `/var/log/audit/audit.log` as AVC records. Focused searches reduce noise:

```bash
ausearch -m AVC,USER_AVC,SELINUX_ERR,USER_SELINUX_ERR -ts recent
sealert -a /var/log/audit/audit.log
```

`sealert` often explains the source context, target context, denied operation and recommended fix. `ausearch` gives raw audit evidence, which can be matched to an inode with `find / -inum <number>` when the path is unclear.

SELinux booleans enable defined policy variations without writing a custom module. Administrators list and verify booleans with `getsebool -a` or `semanage boolean -l`, then test a change with `setsebool name on`. The `-P` option makes the change persistent:

```bash
getsebool httpd_enable_homedirs
setsebool httpd_enable_homedirs on
setsebool -P httpd_enable_homedirs on
```

The strongest fixes keep SELinux enforcing and adjust labels, ports or booleans to match the intended service design. Disabling SELinux hides the problem and weakens the host.
### Repair and reporting priorities
A reliable investigation follows the observable failure. Missing shared objects point to package ownership and reinstall checks. Valgrind output points to application code, so administrators preserve the report rather than guessing at a fix. Trace output shows whether the program reaches the expected file, socket, library or permission boundary. SELinux evidence shows whether policy, labels or booleans block an otherwise valid service action.

Each command should answer a narrow question. `ldd` asks which libraries a trusted executable needs. `valgrind` asks how the process handles memory during execution and at exit. `ltrace` asks which library calls the program makes. `strace` asks which system calls reach the kernel and how the kernel responds. SELinux tools ask whether policy allows the source context to access the target context for the requested operation.