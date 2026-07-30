# Using Essential Tools 
> [!NOTE]
> A practical introduction to essential RHEL command-line skills, covering lab setup, remote access, text processing, file management, permissions, ownership, links, archiving, and compression.
## RHEL 8 and a safe practice environment
Red Hat Enterprise Linux 8 is an enterprise Linux distribution built for long support cycles, controlled updates, and stable application compatibility. As of July 2026, RHEL 8 is in Maintenance Support, which Red Hat currently schedules through 31 May 2029. Application Streams can have shorter life cycles than the operating system, so administrators must check the life cycle of each selected stream.

A RHEL subscription governs access to Red Hat repositories, updates, support services, and related content. It does not follow that RHEL binaries and updates carry no conditions because much of the underlying software uses open-source licences. An eligible Red Hat account and subscription can register a development or production system. Interactive registration avoids exposing a password in shell history or a process list:

```bash
$ sudo subscription-manager register
$ sudo subscription-manager identity
$ sudo dnf repolist
```

RHEL 8 uses BaseOS for core operating-system content and AppStream for additional user-space applications and runtimes. A registered system can install useful command-line packages with DNF:

```bash
$ sudo dnf install nano vim-enhanced bash-completion
```

The full installation ISO contains the main repositories. The smaller boot ISO starts the installer but requires network access to installation repositories. Image sizes and installer screens change, so current Red Hat installation documentation should guide a new build.

A virtual machine provides a disposable practice system. An administrator can install RHEL from current media on a suitable hypervisor, take a snapshot, and practise without risking a production host. Vagrant can automate supported providers through a project `Vagrantfile`, with `vagrant up`, `vagrant ssh`, and `vagrant halt`. Older third-party boxes such as `generic/rhel8` can disappear, lag security updates, or carry unsuitable terms. Teams should verify the publisher, version, licence, provenance, and update path before using any prebuilt image.

A useful lab has enough memory and storage for the selected package set, network access for updates, and a non-root administrator account. A minimal installation reduces initial software but may omit tools used in exercises. The administrator can add those tools from enabled repositories rather than selecting a larger installation profile without review. A second host-only or isolated network can support SSH practice, but it should not expose an unpatched lab directly to an untrusted network.

The installer must receive an explicit destination disk. In a disposable VM, automatic partitioning is usually adequate, but the operator still verifies the selected virtual disk before starting installation. Network installation also requires a working address, route, name service, and repository source. After installation, the administrator updates the system, confirms its release, and takes a clean snapshot:

```bash
$ sudo dnf update
$ cat /etc/redhat-release
$ uname -r
```

Vagrant commands operate against the `Vagrantfile` in the project directory. `vagrant up` creates or starts the machine, `vagrant ssh` uses connection details and keys maintained by Vagrant, and `vagrant halt` requests a clean shutdown. This convenience does not remove the need to register an entitled RHEL guest, patch it, protect credentials, and understand the resulting network exposure.
## bash access, SSH, and command help
A physical or virtual bash connects directly to a terminal such as `/dev/tty1`. An SSH session normally uses a pseudo-terminal such as `/dev/pts/0`. The `tty` command identifies the current terminal.

Before an SSH connection, an administrator checks the host identity, addresses, service, listening sockets, firewall policy, and route:

```bash
$ hostnamectl
$ ip -4 address show
$ systemctl status sshd
$ ss -ltn
$ ssh account@host.example.com
```

A listening TCP port alone does not prove that a remote client can connect. The SSH service might bind only to one address, a firewall might block the traffic, or routing might fail. On the first connection, the client must verify the server's host-key fingerprint through a trusted channel before accepting it.

Linux commands, user names, file names, options, and many configuration values are case-sensitive. A command line normally contains a command, options, and arguments:

```bash
$ ls -al /etc
```

Here, `ls` is the command, `-al` combines two options, and `/etc` is the argument. `ls -a` includes names beginning with `.`, while `ls -l` produces a long listing.

Administrators can obtain help from several local sources:

```bash
$ ip address help
$ ls --help
$ man ip
$ man man
$ man -k network
$ ls /usr/share/doc
```

Manual pages usually open in `less`. The `q` key quits, `/text` searches forward, and `n` repeats the search. Package documentation under `/usr/share/doc` can include examples, change logs, and format-specific guidance.

The shell offers several safe efficiency features. `Ctrl-L` redraws a clear screen without altering command history. The up and down arrow keys navigate history, while `history` lists previous commands. Tab completion can complete commands and paths or display alternatives. An administrator should reread a recalled command before running it because history preserves old arguments, including destructive targets.

Command substitution places a command's standard output into another command. The modern `$(command)` form nests more clearly than backticks:

```bash
$ ls -l "$(tty)"
```

Quoting the substitution keeps its result as one argument. Shell quoting, expansion, and splitting occur before a program receives its arguments, so a displayed command can behave differently when quotes disappear.
## Streams, redirection, pipelines, and `tee`
Every process starts with three conventional file descriptors:

| Descriptor | Name | Normal destination or source |
| --- | --- | --- |
| `0` | Standard input | Keyboard or upstream data |
| `1` | Standard output | Terminal |
| `2` | Standard error | Terminal |

Bash redirection changes these connections before the command runs:

| Form                  | Effect                                                |
| --------------------- | ----------------------------------------------------- |
| `command > file`      | Replace `file` with standard output                   |
| `command >> file`     | Append standard output to `file`                      |
| `command 2> file`     | Replace `file` with standard error                    |
| `command 2>> file`    | Append standard error to `file`                       |
| `command > file 2>&1` | Send both output streams to `file`                    |
| `command &> file`     | Send both output streams to `file` in Bash            |
| `command < file`      | Read standard input from `file`                       |
| first \| second       | Feed the first command's output to the second command |

The order of redirections affects the result. In `command > file 2>&1`, Bash first connects standard output to `file`, then connects standard error to the same destination. A single `>` truncates an existing file before the command writes, so administrators should inspect a command carefully before running it against important data.

A here-document supplies multiple input lines. Quoting the delimiter prevents parameter expansion, command substitution, and arithmetic expansion within the body:

```bash
$ cat > story.txt <<'EOF'
Line 1
Line 2
EOF
```

The `tee` command copies standard input to standard output and one or more files. It also solves a common privilege problem. In `sudo echo text >> /etc/hosts`, `sudo` elevates `echo`, but the unprivileged shell still opens `/etc/hosts`. A privileged `tee` opens the file instead:

```bash
$ printf '%s\n' '192.0.2.10 example-host' | sudo tee -a /etc/hosts
```

The `-a` option appends. Without it, `tee` replaces the target. A pipeline passes standard output by default, not standard error.

Redirection supports separate diagnostic and result files:

```bash
$ command >result.log 2>error.log
$ command >>result.log 2>>error.log
```

Scripts often need one ordered stream instead:

```bash
$ command >>combined.log 2>&1
```

`/dev/null` discards data, but silent error removal can hide a failed operation. Logging an expected failure with context usually serves administration better than discarding all standard error. A pipeline's exit status normally comes from its last command unless Bash enables `pipefail`, so robust scripts commonly use `set -o pipefail` and test the resulting status.
## Creating and editing text
`touch file1` creates an empty file when `file1` does not exist. When it exists, `touch` updates its timestamps unless options request another action. The `file file1` command examines content and metadata to classify the object. `cat file1` writes its content to standard output.

Nano offers direct editing and displays its principal shortcuts. An administrator opens a file with `nano path`, edits it, presses `Ctrl-X`, confirms whether to save, and confirms the file name. A privileged configuration file requires an authorised command such as `sudo nano /etc/hosts`.

Vim starts in Normal mode. The `i` and `a` commands enter Insert mode before and after the cursor. The `I` and `A` commands insert at the beginning and append at the end of the line. `Esc` returns to Normal mode, and `:` opens Command-line mode. Common commands include:

| Command | Action |
| --- | --- |
| `:w` | Write changes |
| `:q` | Quit when no unsaved changes remain |
| `:wq` | Write and quit |
| `:x` | Write if needed, then quit |
| `:q!` | Quit and discard unsaved changes |

`vimtutor` provides guided practice. An administrator should become fluent in one available editor before changing live service configuration.

A careful configuration edit begins with a backup or version-controlled copy, preserves ownership and permissions, and validates syntax before a service reload. Some commands provide dedicated validators, such as `sshd -t` for OpenSSH server configuration. A reload applies supported changes without stopping established work:

```bash
$ sudo cp -a /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
$ sudo vim /etc/ssh/sshd_config
$ sudo sshd -t
$ sudo systemctl reload sshd
```

The administrator retains the current privileged session until a second connection confirms that SSH still works. This practice prevents a syntax or access error from closing the only available management path.
## Paths, directories, files, and shell expansion
Linux presents many resources through file-like interfaces, but the phrase "everything is a file" is shorthand. Regular files, directories, symbolic links, devices, pipes, and sockets have distinct semantics. A directory maps names to filesystem objects rather than behaving like an ordinary text file.

An absolute path begins at `/`, the filesystem root. A relative path begins at the current working directory. `.` names the current directory, `..` names its parent, and `~` expands to a user's home directory. These commands control location:

```bash
$ pwd
$ cd /usr/share/doc
$ cd -
$ cd
```

`pwd` prints the current directory. `cd -` returns to the previous directory, and `cd` without an argument returns to the user's home. Tab completion reduces typing errors. Quoting protects paths that contain spaces or shell metacharacters.

Core file operations include:

| Command | Purpose |
| --- | --- |
| `mkdir directory` | Create one directory |
| `mkdir -p parent/child` | Create missing parent directories |
| `rmdir directory` | Remove an empty directory |
| `cp source destination` | Copy a file |
| `mv source destination` | Move or rename an entry |
| `rm file` | Unlink a file name |
| `rm -r directory` | Recursively remove a directory tree |

Copying normally requires access to read the source file, traverse its path, and create or replace the destination. A same-filesystem move normally renames a directory entry and requires suitable permissions on the source and destination directories. A cross-filesystem move copies data and then removes the source. Removal depends mainly on write and execute permission on the parent directory, not write permission on the file itself.

`rm` has no general undo facility. Recursive or forced removal deserves an explicit, verified target. An administrator can use `rm -i` for a small interactive operation, but prompts do not replace backups or target validation.

Bash filename expansion and brace expansion perform different jobs:

| Expression | Meaning |
| --- | --- |
| `*` | Match zero or more characters |
| `?` | Match exactly one character |
| `[0-9]` | Match one character in the range |
| `file{1..12}` | Generate twelve words before pathname matching |

Patterns do not normally match a leading `.` unless the pattern also begins with `.` or shell settings change that rule. Brace expansion generates text even when no matching file exists. For example, `touch file{1..12}` creates twelve names, while `ls file?` matches `file1` through `file9` but not `file10`.

Expansion occurs before `ls`, `cp`, `mv`, or `rm` starts. The command receives the expanded list rather than the pattern. If a pattern matches nothing, Bash can leave it unchanged unless options such as `nullglob` alter that behaviour. An administrator can preview a prospective set with `printf '%s\n' pattern` before applying a mutating command. `--` also ends option processing for many utilities, which protects a file name beginning with `-`:

```bash
$ printf '%s\n' file?
$ rm -- -unusual-name
```

Copy and move commands can overwrite an existing destination. Interactive options can request confirmation, but scripts need explicit collision handling. Preserving metadata may require `cp -a` rather than plain `cp`, especially for a directory tree containing links, timestamps, ownership, or extended attributes.
## Reading and searching text
Different tools suit different file sizes and questions:

| Command | Use |
| --- | --- |
| `cat file` | Emit a short file |
| `head -n 10 file` | Emit the first ten lines |
| `tail -n 10 file` | Emit the last ten lines |
| `less file` | Page, navigate, and search interactively |
| `wc -l file` | Count newline-terminated lines |
| `grep pattern file` | Emit matching lines |

`/etc/passwd` stores account attributes such as the user ID, group ID, home directory, and login shell. It does not store usable password hashes. RHEL normally stores password hashes in the access-restricted `/etc/shadow`.

Regular expressions make `grep` precise:

```bash
$ grep '^root:' /etc/passwd
$ grep '/bin/bash$' /etc/passwd
$ grep -n 'search text' file
$ sudo grep -Ev '^[[:space:]]*(#|$)' /etc/ssh/sshd_config
```

`^` anchors a match at the start of a line, and `$` anchors it at the end. `-E` enables extended regular expressions, `-v` selects non-matching lines, and `-n` displays line numbers. The last command removes blank lines and comments, including lines with leading whitespace.

Filtering a configuration file does not always reveal the effective setting. OpenSSH can apply defaults, `Match` blocks, command-line options, and included files. `sshd -T` evaluates the effective server configuration:

```bash
$ sudo sshd -T | grep '^passwordauthentication '
```

Pipelines combine selection and measurement. The following command counts active-looking lines after removing blank lines and full-line comments:

```bash
$ sudo grep -Ev '^[[:space:]]*(#|$)' /etc/ssh/sshd_config | wc -l
```

The count describes the filtered text, not the number of effective configuration directives. Duplicate keys, included files, context-specific blocks, and built-in defaults still require a configuration-aware validator. `grep` reads data without changing the source file, which makes it suitable for inspection before an edit.
## Metadata and file types
`ls -l` shows the file type and mode, hard-link count, owner, group, size, modification time, and name. `ls -ld directory` reports on the directory entry instead of listing its contents. Colour depends on aliases and environment settings, so it cannot establish type, security, or access.

`stat` supplies fuller metadata and supports selected output:

```bash
$ stat file
$ stat -c '%a %A %U %G %n' file
$ ls -lZ file
```

For GNU `stat`, `%a` displays the permission bits in octal, not decimal. `%A` displays symbolic permissions. `%U`, `%G`, and `%n` display the owner, group, and name. `ls -Z` shows the SELinux security context.

The first character in an `ls -l` mode string identifies the broad object type:

| Character | Type |
| --- | --- |
| `-` | Regular file |
| `d` | Directory |
| `l` | Symbolic link |
| `b` | Block device |
| `c` | Character device |
| `p` | Named pipe |
| `s` | Socket |

A socket can represent local interprocess communication or a network endpoint. It does not always represent an active network connection.
## Permissions, `umask`, and mode changes
Discretionary access-control mode bits divide permissions among the owning user, owning group, and all other users. The kernel selects one applicable class. It uses owner bits when the effective user ID matches the owner, group bits when a group matches, and other bits otherwise. It does not combine a matching owner's permissions with group or other permissions.

The symbols `r`, `w`, and `x` change meaning for directories:

| Object | `r` | `w` | `x` |
| --- | --- | --- | --- |
| Regular file | Read content | Modify content | Execute a suitable program or script |
| Directory | List names | Create, remove, or rename entries when traversal also succeeds | Traverse the directory and access known entries |

Deleting a name depends on the parent directory. A user can sometimes remove a read-only file while lacking permission to change its content. Access-control lists can grant named users and groups more specific access. SELinux policy can still deny an operation that mode bits allow. The root account commonly bypasses discretionary checks through capabilities, but it does not bypass every kernel, SELinux, filesystem, or immutable-attribute control.

Octal mode values add `4` for read, `2` for write, and `1` for execute. Therefore, `7` represents `rwx`, `6` represents `rw-`, and `5` represents `r-x`. Three digits normally specify user, group, and other permissions.

For a typical new object, a program requests a creation mode and the process `umask` clears selected bits. Many tools request `0666` for regular files and `0777` for directories. They do not automatically make regular files executable. RHEL commonly assigns `0002` to a standard interactive user and `0022` to root, although service settings and local policy can differ.

```bash
$ umask
$ umask 0077
$ touch private-file
$ mkdir private-directory
```

With the usual requested modes, `0077` yields `0600` for a file and `0700` for a directory. A `umask` removes permissions from the requested mode and cannot add permissions that the creating program did not request.

`chmod` changes an existing mode:

```bash
$ chmod 0640 report.txt
$ chmod u=rw,g=r,o= report.txt
$ chmod o+w shared.txt
$ chmod -R a+X tree
```

Numeric notation replaces the selected mode bits. Symbolic `+`, `-`, and `=` add, remove, and assign permissions. When a symbolic clause omits the class, the current `umask` affects which classes change. An explicit class such as `a` avoids that ambiguity. Uppercase `X` adds execute only to directories or to files that already have execute set for at least one class. Recursive changes can expose or disable a large tree, so administrators should inspect the scope before using `-R`.

Permission diagnosis follows the complete path rather than the final file alone. An administrator checks the process identity, supplementary groups, each parent directory, the target mode, any ACL, and the SELinux context:

```bash
$ id
$ namei -l /srv/project/report.txt
$ stat -c '%A %a %U %G %n' /srv/project/report.txt
$ getfacl /srv/project/report.txt
$ ls -lZ /srv/project/report.txt
```

`namei -l` displays path components and their modes. `getfacl` reveals named user, named group, and mask entries that `ls -l` compresses into a `+` marker. An ACL mask limits the effective permissions of named users, named groups, and the owning group. `chmod` can change that mask, so administrators should recheck ACLs after a mode change.

SELinux labels add type-based controls independently of discretionary permissions. Broadening a mode to `0777` cannot solve a denial caused by an unsuitable SELinux type, and it can create an unrelated exposure. Administrators should examine audit records and repair the label or policy instead of disabling SELinux or opening every mode bit.
## Ownership, identities, privilege, and links
RHEL commonly creates a private group with the same name as each ordinary user. A new file normally takes the process's effective user ID and effective group ID. A directory with the set-group-ID bit can instead make new entries inherit the directory's group.

`chown` changes ownership, and `chgrp` changes group ownership:

```bash
$ sudo chown alice report.txt
$ sudo chown alice:editors report.txt
$ sudo chgrp editors report.txt
```

An unprivileged owner can usually change a file's group to one of that user's groups. Changing the owning user normally requires suitable privilege.

`id` displays the current user, primary group, and supplementary groups. Adding an account to a supplementary group does not update its existing processes:

```bash
$ sudo usermod -aG wheel alice
$ id alice
```

A fresh login obtains the new supplementary-group list. `newgrp wheel` starts a new shell with a different effective group, and `exit` returns to the preceding shell.

Group ownership supports collaborative directories when the administrator combines membership, the directory's set-group-ID bit, and an appropriate `umask` or default ACL. Without set-group-ID, new entries commonly use the creating process's effective group. Without group write permission, other project members still cannot modify the entry. Existing sessions retain their old supplementary groups until they start a new login context.

Administrators should use `sudo` for authorised administrative commands because policy can limit and log access. The `wheel` group commonly receives broad `sudo` rights on RHEL. `sudo -i` opens a root login shell when sustained administrative work genuinely requires one. `su - account` opens a login shell as the target account and normally requires the target account's password. Setting a root password solely to use `su` expands credential risk without improving a task that `sudo` already authorises.

A hard link creates another directory entry for the same inode:

```bash
$ ln source-file second-name
```

Hard links normally cannot cross filesystems, and ordinary users cannot create them for directories. Removing one name decreases the link count, but the data remains while another hard link or open file description refers to it. Directory link-count behaviour varies across filesystems, so administrators should not infer a universal subdirectory count from it.

A symbolic link stores a path to another name:

```bash
$ ln -s /etc/services ports
```

Symbolic links can cross filesystems and can refer to directories. They have their own inode and can dangle when the target disappears. Relative link targets resolve from the directory containing the link, not from the process's current directory.
## Restricted directories and reliable logging
Execute permission without read permission on a directory allows traversal and access to known names but blocks an ordinary listing. This arrangement does not provide strong confidentiality because users can guess names, learn them elsewhere, or observe them through another channel. Write and execute permission can let a user create entries and remove or rename known entries.

The sticky bit protects a shared writable directory by restricting removal and renaming to an entry's owner, the directory owner, or a privileged process:

```bash
$ sudo chmod 1777 /srv/drop
```

A set-group-ID project directory can preserve group ownership:

```bash
$ sudo chgrp editors /srv/project
$ sudo chmod 2770 /srv/project
```

A write-only regular file does not by itself form a secure append-only log. Write permission can allow truncation or replacement of content, `>>` requests append behaviour only for that open operation, and a file owner can normally change the mode. Reliable logging should send records to a controlled service such as `systemd-journald` or `rsyslog`, keep log files under a privileged owner, apply suitable SELinux policy, and restrict log-management privileges.

```bash
$ logger -t training 'event text'
$ journalctl -t training
```

Access-control lists support a named user or group without changing the owning group:

```bash
$ sudo setfacl -m u:alice:rw /srv/project/report.txt
$ getfacl /srv/project/report.txt
```

The ACL mask can reduce the displayed named entry's effective access. Administrators should use `getfacl` to verify the result and should remember that copying or archiving may require explicit options to preserve ACLs and extended attributes.
## Archives and compression
An archive combines files and metadata into one stream. Compression reduces repeated data patterns within that stream. An uncompressed tar archive does not guarantee space savings. Tar adds headers and padding, and comparisons between a directory's allocated blocks and an archive's apparent size can mislead.

GNU tar uses `-c` to create, `-t` to list, `-x` to extract, and `-f` to identify the archive:

```bash
$ tar -cf files.tar file1 file2 directory/
$ tar -tf files.tar
$ mkdir restore
$ tar -xf files.tar -C restore
```

Relative member names make restoration location explicit. Before extraction, an administrator should list an unfamiliar archive, inspect its paths, and extract it into a new restricted directory. Archives from untrusted sources can contain traversal paths, links, device entries, or ownership metadata that becomes dangerous under privilege. Administrators should not test recovery by deleting a live system file. They should verify the archive, perform a separate test extraction, and compare the result before removing any original.

`star` provides another tape-archive implementation when its package is installed, but scripts should not assume that every RHEL system includes it. GNU tar remains the common baseline for these operations.

Standalone `gzip` and `bzip2` compress one file and normally replace the input after successful compression. Their corresponding decompression commands are `gunzip` and `bunzip2`. The `-k` option retains the input where the installed version supports it. Tar can invoke either compressor:

```bash
$ tar -czf files.tar.gz directory/
$ tar -cjf files.tar.bz2 directory/
$ tar -tf files.tar.gz
$ tar -xf files.tar.gz -C restore
```

GNU tar can detect common compression formats while reading. Gzip often favours speed, while bzip2 can produce a smaller result for some data. Neither outcome is universal. Data content, compressor settings, processor speed, memory, and storage determine the actual time and size.

An archive becomes a useful backup only when another failure domain stores it, access controls protect it, retention policy preserves it, and a tested restoration procedure can recover the required files.

A verification exercise creates the archive, lists it, extracts it to a temporary location, and compares the restored tree with the source. File-content checks do not by themselves prove that ownership, permissions, ACLs, extended attributes, sparse-file layout, hard links, and SELinux contexts survived. The required restore outcome determines which metadata the archive must preserve and which verification commands apply.

Archive names should state the scope and creation time without relying on local ambiguity. Checksums can detect later byte changes:

```bash
$ sha256sum files.tar.gz > files.tar.gz.sha256
$ sha256sum -c files.tar.gz.sha256
```

A matching checksum establishes byte integrity against the recorded value. It does not establish that the archive was trustworthy when created, that it contains the required files, or that a restoration succeeds. Protected signing or an authenticated storage service provides stronger provenance when an attacker could replace both the archive and its checksum.