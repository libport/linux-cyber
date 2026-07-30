# Using Essential Tools 
> [!NOTE]
> A practical introduction to essential RHEL command-line skills, covering lab setup, remote access, text processing, file management, permissions, ownership, links, archiving, and compression.

Red Hat Enterprise Linux 8 administration centres on command-line work, text configuration, file management, access control, remote access, and archiving. RHEL combines open source software with Red Hat's tested builds, update channels, lifecycle, and support services. Access to official repositories follows Red Hat's subscription terms. A qualifying Red Hat Developer Subscription can provide no-cost access for individual development and learning, subject to the current conditions.

A safe learning environment uses a dedicated virtual machine rather than a production host. The learner can install RHEL from current installation media or use a trusted, appropriately licensed virtual machine image. A minimal installation provides enough software for command-line practice while limiting resource use and unnecessary services.
## Build and access a lab system
A virtual machine normally needs a virtual disk, sufficient memory for the selected RHEL 8 release and workload, installation media, and network access. The RHEL Boot ISO retrieves packages from a configured network source, while the full installation ISO contains more local content. Current Red Hat requirements should replace old fixed examples for ISO size, memory, and disk capacity.

Network address translation gives the guest outbound access, but it does not automatically make the guest reachable from the host. Host-only networking, bridged networking, or explicit port forwarding can provide an SSH path. The chosen design must also account for the guest firewall and the exposure of services to other networks.

The installer can configure language, time, storage, network interfaces, software selection, and user accounts. It can also register the system. Registration can instead occur after installation:

```bash
sudo subscription-manager register
sudo subscription-manager identity
sudo dnf repolist
```

Interactive registration keeps credentials out of shell history. Simple Content Access is now the default subscription model, so a registered system usually does not need `subscription-manager attach --auto`. An organisation that still uses legacy entitlement attachment must follow its own subscription configuration.

Vagrant can manage a repeatable lab when the host has a supported provider and Vagrant installation. The operator should select a maintained image from a trusted publisher and verify its licence, architecture, version, and registration requirements. A community box name does not establish Red Hat provenance.

```bash
mkdir -p vagrant/rhel8
cd vagrant/rhel8
vagrant init <trusted-rhel8-box>
vagrant up
vagrant ssh
vagrant halt
```

`vagrant init` creates a `Vagrantfile`, `vagrant up` creates or starts the guest, `vagrant ssh` opens its configured shell, and `vagrant halt` requests a graceful shutdown. The project directory retains the machine configuration and allows later starts.

A console login opens a physical or virtual terminal such as `/dev/tty1`. An SSH login opens a pseudo-terminal such as `/dev/pts/0`. These commands identify the host, terminal, addresses, service state, and listening TCP sockets:

```bash
hostnamectl
tty
ip -4 address show
systemctl is-active sshd
ss -lnt
```

`ip -4 address show` restricts output to IPv4. The shorter `ip address` or `ip a` also displays IPv6, so the forms do not produce identical results. A listener on TCP port 22 shows that a process accepts local connections on that socket. It does not prove that `sshd` owns the socket, that the firewall permits access, or that a remote route works. `ss -lntp` adds process details when the caller has sufficient privilege.

An SSH client connects with an account and a resolvable host name or address:

```bash
ssh account@server.example.test
```

The client asks the user to verify an unseen server host key before storing it. That check protects against connecting to the wrong host. Current Windows, macOS, and Linux systems can provide OpenSSH clients, while other compatible clients remain available.

Linux commands, account names, paths, and file names are case-sensitive. A command generally has this structure:

```text
command [options] [arguments]
```

`ls` lists the current directory, `ls -a` includes names beginning with a dot, and `ls -l /etc` produces a long listing for `/etc`. A command can provide concise help through `--help`, a subcommand such as `ip address help`, or a manual page such as `man ip`. Installed packages often place additional material under `/usr/share/doc`. Tab completion reduces typing errors, `Ctrl-L` clears the display, and `Alt-.` or Escape followed by a full stop recalls the previous command's last argument in Bash.

Manual pages contain named sections, synopsis notation, options, files, examples, and related commands. `man man` explains the interface, `man 5 passwd` requests the file-format page rather than a command page, and `apropos keyword` searches manual-page descriptions. The shell's `type` command distinguishes an executable from an alias, function, keyword, or built-in:

```bash
type ls
type cd
command -V ip
```

This distinction helps when two command names appear to behave differently. Aliases can add options to interactive commands, while scripts often receive the underlying command without the alias. A minimal installation can also lack a package that a fuller installation includes, so an absent command does not establish that the operating system lacks the capability.

Remote access requires several independent conditions. The guest needs a usable address and route, `sshd` must run and listen on an intended interface, the firewall must admit the connection, and the client must reach the host. These checks separate the layers:

```bash
ip route
systemctl status sshd
sudo sshd -t
sudo firewall-cmd --list-services
ssh -v account@server.example.test
```

`sshd -t` validates server configuration syntax without starting a new daemon. The client option `-v` adds diagnostic output. A failed connection therefore calls for evidence from each layer rather than an assumption based only on port 22.
### Redirect shell input and output
Each process starts with standard input, standard output, and standard error, represented by file descriptors 0, 1, and 2. The shell connects them to the terminal unless redirection or a pipeline changes the connection.

| Bash form | Effect |
| --- | --- |
| `command > file` | Replaces `file` with standard output |
| `command >> file` | Appends standard output to `file` |
| `command 2> file` | Replaces `file` with standard error |
| `command 2>> file` | Appends standard error to `file` |
| `command > file 2>&1` | Sends both output streams to `file` |
| `command &> file` | Sends both output streams to `file` in Bash |
| `producer | consumer` | Sends standard output from one process to standard input of another |

The single greater-than operator truncates an existing destination before the command runs. The double form appends. Redirections run from left to right, so their order can change the result. Bash supports `&>` as a convenience, while `> file 2>&1` works in a wider range of Bourne-style shells.

A command can produce both streams during one invocation:

```bash
ls /etc/hosts /etc/Hosts > output.txt 2> error.txt
ls /etc/hosts /etc/Hosts > combined.txt 2>&1
ls /etc/hosts /etc/Hosts >> combined.txt 2>&1
```

On a normally configured host, the lowercase path enters standard output and the mixed-case path enters standard error. The first command separates them, the second replaces one combined log, and the third appends to that log. Redirection captures bytes, not their meaning, so a caller must also inspect the command's exit status when success or failure controls later work.

A here-document supplies several lines to a command. The closing delimiter must appear alone, without extra characters:

```bash
cat > story.txt <<'END'
Line 1
Line 2
END
```

Quoting `END` prevents parameter, command, and arithmetic expansion inside the body. An unquoted delimiter allows those expansions. Scripts commonly use here-documents to generate short configuration fragments.

Shell redirection occurs under the shell user's identity. Elevating only `echo` therefore does not grant permission to the shell that opens a protected destination:

```bash
sudo echo "192.0.2.10 app.example.test" >> /etc/hosts
```

The `tee` process can open the file with elevated privilege instead:

```bash
printf '%s\n' '192.0.2.10 app.example.test' |
  sudo tee -a /etc/hosts > /dev/null
```

`tee` copies input to both standard output and one or more files. Its `-a` option appends. An administrator should inspect the destination first because an unintended duplicate or an omitted `-a` can damage a configuration file.
### Edit text at the command line
RHEL 8 uses YUM v4, which builds on DNF technology. The `yum` and `dnf` command names support the same package-management workflow on RHEL 8. A registered system with enabled BaseOS and AppStream repositories can install common editors and shell completion:

```bash
sudo dnf install nano vim-enhanced bash-completion
```

Nano provides a direct editing model. `nano story.txt` opens an existing file or starts a new buffer. `Ctrl-X` exits, and Nano then asks whether to save changed content. An editor that opens a protected file needs appropriate privilege, such as `sudo nano /etc/hosts`, although administrators should use elevated editors only for files that require them.

Vim separates actions into modes. It starts in Normal mode. `i` inserts before the cursor, `a` appends after it, `I` inserts at the start of the line, and `A` appends at the end. Escape returns to Normal mode. A colon opens the command-line mode, where `:x` writes changes when necessary and exits, while `:q!` abandons changes. `vimtutor` provides an interactive introduction.
### Manage directories and files
An absolute path begins at `/`, the root of the file system. A relative path begins at the current directory. `~` denotes the current user's home directory, `.` denotes the current directory, and `..` denotes its parent. These commands navigate and create a small hierarchy:

```bash
pwd
cd /usr/share/doc
cd -
cd
mkdir -p project/input project/output
```

`pwd` prints the current directory. `cd -` returns to the previous directory, and `cd` without an argument returns to the home directory. `mkdir -p` creates missing parent directories and does not fail when an existing parent already satisfies the request.

`touch name` creates an empty regular file when `name` does not exist. When the file already exists, `touch` updates timestamps and preserves its content. The `file` command examines content and selected metadata to classify an object:

```bash
touch file1
file file1
printf '%s\n' 'hello' > file1
file file1
```

The first classification reports an empty file. After redirection, `file` normally identifies text. This inspection gives stronger evidence than a filename extension, which Linux does not require for file typing. Names beginning with a dot remain ordinary directory entries. Shell tools hide them from default listings by convention, not through a separate hidden-file attribute.

`rmdir` removes an empty directory. `rm -r` recursively removes a directory tree, and `rm -f` suppresses selected prompts and errors. Combining them as `rm -rf` can erase extensive data without confirmation. An administrator should resolve and inspect the exact target before using recursive deletion and should not use a broad wildcard in a home, system, or configuration directory.

The principal file operations are `cp`, `mv`, and `rm`:

```bash
cp /etc/hosts .
mv hosts hosts.old
rm -i hosts.old
```

Copying a regular file requires search permission on path components, read access to the source, and suitable access to the destination. Creating a new destination entry normally requires write and search permissions on its directory. Moving or renaming within one file system normally requires write and search permissions on both affected directories. A move across file systems behaves more like a copy followed by deletion and can need additional access. Removing a file depends mainly on write and search permissions for its directory, not write permission on the file itself. Sticky-bit rules, ACLs, SELinux, immutable attributes, and read-only mounts can impose further limits.

Bash expands patterns before it starts a command. `*` matches any string within one path component, `?` matches one character, and bracket expressions match selected characters. Brace expansion generates literal alternatives or sequences and differs from pathname matching:

```bash
touch file{1..12}
printf '%s\n' file*
printf '%s\n' file?
printf '%s\n' file??
```

The first command generates twelve names. `file?` matches `file1` through `file9`, while `file??` matches names with two characters after `file`, including `file10`, `file11`, and `file12`. Quoting a pattern prevents pathname expansion. Before passing a pattern to `rm`, an operator can print the expansion with `printf` or `ls` and confirm the targets.

Short text files suit `cat`. `head` and `tail` read the beginning or end, while `less` provides paging and interactive search. `wc -l` counts newline-terminated lines:

```bash
cat /etc/hosts
head /etc/passwd
tail -n 2 /etc/passwd
less /etc/services
wc -l /etc/services
```

Within `less`, `/text` searches forward, `n` finds the next match, and `q` quits. The `man` command commonly uses a pager such as `less`.

`grep` selects lines that match text or a regular expression. Matching remains case-sensitive unless an option changes it. A caret anchors a match at the beginning of a line, and a dollar sign anchors it at the end:

```bash
grep '^root:' /etc/passwd
grep '/bin/bash$' /etc/passwd
sudo grep -E '^PasswordAuthentication[[:space:]]+' /etc/ssh/sshd_config
sudo grep -vE '^[[:space:]]*(#|$)' /etc/ssh/sshd_config
```

The last command filters blank lines and lines whose first non-space character is `#`. It does not modify the file. Configuration includes, defaults, command-line options, and service policy can affect the final SSH behaviour, so a textual match alone does not always establish the effective setting.

Quoting protects a regular expression from shell expansion before `grep` receives it. Basic `grep` recognises anchors and character classes, while `grep -E` enables extended operators such as grouping, alternation, and repetition without many backslashes. `grep -F` treats the search as a fixed string, which suits punctuation that should have no regular-expression meaning.

```bash
grep -F 'root:' /etc/passwd
grep -i 'worldwide http' /etc/services
grep -n '^root:' /etc/passwd
grep -c '/bin/bash$' /etc/passwd
grep -rF 'setting=value' project
```

The `-i` option ignores case, `-n` prints line numbers, `-c` reports a count of matching lines, and `-r` searches below a directory. Recursive searches should use a narrow starting path because pseudo-file systems, large trees, unreadable files, and binary content can produce slow or noisy results.

A pipeline can combine selection and measurement:

```bash
sudo grep -vE '^[[:space:]]*(#|$)' /etc/ssh/sshd_config |
  wc -l
```

`wc -l` counts the filtered lines that reach it. It does not validate the configuration and, without additional shell settings, the pipeline's final status normally comes from `wc`. `grep -c` avoids that pipeline when only a match count is needed. When a privileged file requires reading, `sudo` belongs on the process that opens the file. Giving privilege to a later `wc` process cannot repair a read failure in `grep`.
## Interpret metadata and file types
`ls -l` displays the file type and mode, hard-link count, owner, group, size, modification time, and name. `ls -ld directory` reports the directory entry itself rather than its contents. `stat` provides fuller metadata and supports selected output:

```bash
ls -ld /etc
stat /etc/hosts
stat -c '%A %a %U %G %n' /etc/hosts
```

`stat` prints modes such as `644` in octal, not decimal. The first character in an `ls -l` mode identifies the file type.

| Character | File type |
| --- | --- |
| `-` | Regular file |
| `d` | Directory |
| `l` | Symbolic link |
| `p` | Named pipe |
| `b` | Block device |
| `c` | Character device |
| `s` | Unix domain socket |

A Unix domain socket supports local interprocess communication. It does not represent every network connection. Commands such as `ss` inspect active sockets more directly.

The next nine mode characters form three classes: owner, group, and other. Each class receives read, write, and execute bits. Access checks select the owner class when the process's effective user ID matches the owner. Otherwise, they select the group class when an effective group matches. If neither matches, they select the other class. The kernel does not combine a matching owner's bits with group or other bits.

| Bit | Octal value | Regular file | Directory |
| --- | ---: | --- | --- |
| `r` | 4 | Reads file content | Lists entry names |
| `w` | 2 | Changes file content | Creates, removes, or renames entries when search access also exists |
| `x` | 1 | Executes a suitable program or script | Searches or traverses the directory |

The values combine within each class. Read and write form 6, read and execute form 5, and all three form 7. A mode such as `0640` gives the owner read and write, the group read, and other users no access.

New regular files usually begin from a base mode no broader than `0666`, and new directories begin from `0777`. The process applies its `umask` as a bit mask:

```text
effective mode = base mode & ~umask
```

The operation removes masked bits rather than performing ordinary subtraction. On a standard RHEL 8 configuration, a regular user's `umask` commonly starts at `0002`, producing `0664` for a new regular file and `0775` for a new directory. Root commonly starts at `0022`, producing `0644` and `0755`. Shell startup files, PAM configuration, services, containers, and applications can set different values.

```bash
umask
umask 0077
touch private.txt
mkdir private-dir
stat -c '%A %a %n' private.txt private-dir
```

`umask 0077` removes all group and other bits from newly created objects in that shell. It does not retroactively change existing objects, and an application can request a narrower base mode.

`chmod` changes mode bits. Numeric notation sets a complete mode, while symbolic notation changes selected classes:

```bash
chmod 0640 report.txt
chmod u=rw,g=r,o= report.txt
chmod g+w shared.txt
chmod -R a+X project
```

Uppercase `X` adds execute permission to directories and to regular files that already have at least one execute bit. This makes `a+X` safer than `a+x` for many recursive directory operations. When symbolic notation omits the class, as in `chmod +w`, the current `umask` limits the affected classes. An explicit class such as `a+w` does not rely on that omission rule.

`chown` changes ownership, and `chgrp` changes group ownership:

```bash
sudo chown root:root report.txt
sudo chown account: report.txt
chgrp project report.txt
```

Only root can normally change a file's user owner. The file owner can normally change its group to a group of which that owner is a member. In `chown account:`, the empty group field requests the named account's login group.

Traditional mode bits implement discretionary access control. Root processes often hold capabilities that bypass these checks, but root does not override every protection. SELinux policy, read-only mounts, immutable attributes, encrypted storage, and other kernel controls can still deny an operation. RHEL enables SELinux by default, so administrators should diagnose both mode bits and security context rather than treating root as universally unrestricted.

Access control lists extend the owner-group-other model. `getfacl` shows entries, and `setfacl` can grant named users or groups specific rights. The ACL mask limits effective permissions for named users, named groups, and the owning group. A plus sign after the mode in `ls -l` can indicate an ACL, while a full stop can indicate an SELinux context.

An access failure requires inspection of the process identity and every relevant path component. A reliable diagnosis gathers current state before changing permissions:

```bash
id
namei -l /srv/project/report.txt
ls -ld /srv /srv/project /srv/project/report.txt
getfacl /srv/project/report.txt
ls -lZ /srv/project/report.txt
```

`id` reports the credentials that the current process actually holds. `namei -l` walks the path and shows ownership and modes for each component. `ls -ld` avoids accidentally listing a directory's contents when the directory's own metadata is required. `getfacl` exposes ACL entries and the effective mask, while `ls -lZ` adds the SELinux context.

The investigation first identifies whether the operation needs content access, directory traversal, directory modification, or execution. It then determines which owner, group, or other class the kernel selects. Next, it checks ACLs, the mount state, file attributes, and SELinux denials. Changing a mode to `0777` before this analysis can expose data without resolving the real cause. The smallest permission change that supports the intended actor and operation provides the safer correction.
## Manage links, users, and groups
A hard link adds another directory entry for the same inode. Each name reaches the same file content and metadata. Removing one name leaves the inode available while another hard link or open reference remains. Hard links cannot cross file systems, and ordinary users generally cannot create hard links to directories.

```bash
ln source.txt second-name.txt
ls -li source.txt second-name.txt
```

Both names display the same inode number. A user does not need administrator status to create a permitted hard link, although directory access, ownership protections, and kernel settings still apply.

A symbolic link stores a path to another name. It has its own inode, can cross file systems, can target a directory, and can remain after its target disappears:

```bash
ln -s /etc/services ports
ls -l ports
```

Relative symbolic targets resolve from the link's containing directory. Absolute targets always begin at `/`, which can make a copied directory tree less portable.

The `id` command reports the current process's user and group identities. RHEL commonly creates a private primary group for each local user. The primary group usually supplies the group owner for a new file, although a parent directory's set-group-ID bit can instead make the file inherit that directory's group.

```bash
id
sudo usermod -aG wheel account
id account
```

`usermod -aG` appends supplementary membership in the account database. Existing login processes do not acquire the new group automatically. The user must start a new login session or deliberately create a process with the new credentials. `newgrp group` starts a new shell with a changed effective group, while `sg group command` runs a command with that group context. Each new shell should end with `exit` when its work finishes.

`sudo` runs an authorised command under another identity, usually root, according to policy. It supports more focused privilege than sharing the root password. `su - account` starts a login-style shell for another account and loads that account's login environment. `su account` changes identity without a complete login environment and normally retains the current directory. Administrators should use the form that matches the required environment and should avoid unnecessary long-lived root shells.
## Apply unusual permissions safely
A write-only mode can let a process change a regular file without reading it:

```bash
chmod u=w log
printf '%s\n' 'event' >> log
```

This arrangement does not create a secure logging service by itself. A writer may still truncate, overwrite, corrupt, or flood the file according to the open mode and other controls. System logging through `systemd-journald`, `rsyslog`, or a dedicated service provides stronger separation, concurrency handling, rotation, and policy.

Directory access has different semantics from file access. Execute without read lets a user traverse the directory and access a known name when that name's own permissions allow it. Write plus execute can allow creation, deletion, and renaming of known entries even when the user cannot list the directory. Hidden names therefore provide weak protection. A shared writable directory often needs the sticky bit, as in mode `1777`, so users cannot remove or rename entries owned by others. ACLs and service-mediated access usually express multi-user requirements more safely.
## Archive and compress files
`tar` groups files and their metadata into one archive. Archiving does not inherently compress data. A tar archive includes headers and block padding, so it can be larger or smaller than a casual `du` comparison suggests. `du` commonly reports allocated disk blocks, while an archive listing reports file length. Neither result proves compression.

| Operation | Command pattern |
| --- | --- |
| Create | `tar -cf archive.tar paths` |
| List | `tar -tf archive.tar` |
| Extract all | `tar -xf archive.tar` |
| Extract selected members | `tar -xf archive.tar member` |

Using `-C` controls the directory from which tar reads or into which it writes. This approach records relative member names and avoids embedding an absolute leading slash:

```bash
sudo tar -cf etc-backup.tar -C / etc
tar -tf etc-backup.tar
mkdir -p restore-test
tar -xf etc-backup.tar -C restore-test
```

The test extraction creates `restore-test/etc` and leaves the live `/etc` tree untouched. A recovery operation can extract a selected member to `/` only after the operator validates the archive, target path, permissions, ownership, and effect on the running service. Deleting a live configuration file to demonstrate restoration creates needless risk.

GNU tar normally preserves basic mode, time, ownership, and link information in the archive, although extraction behaviour depends on privilege and options. Backups that require ACLs, extended attributes, or SELinux labels need suitable options and a tested restoration procedure. An archive should never substitute for restore testing.

A dependable archive workflow separates creation, integrity checking, content inspection, test restoration, and production recovery. The operator records the command and its exit status, stores the archive away from the source failure domain, and protects it according to the sensitivity of its contents. A checksum can detect later byte changes:

```bash
sha256sum etc-backup.tar > etc-backup.tar.sha256
sha256sum -c etc-backup.tar.sha256
tar -tvf etc-backup.tar
```

`tar -tvf` displays member types, modes, ownership, sizes, times, and names. A successful checksum confirms that the archive matches the recorded bytes, but it does not show that the selected files were complete or usable. Test extraction and application-level validation provide that evidence. A recovery plan also accounts for free space, existing destinations, service shutdown requirements, ownership mapping, SELinux relabelling, and confidential data.

GNU tar strips leading slashes from ordinary archived names by default, but an operator should not rely on that behaviour as the only safeguard. Before extracting an untrusted archive, the operator should inspect its member names for absolute paths, parent traversal, unexpected links, devices, and overwrites, then extract it inside an isolated directory.

Compression utilities trade processor time for storage and transfer savings. Results depend on the data, utility, level, and hardware, so one example cannot establish a general ratio or speed:

```bash
gzip archive.tar
gunzip archive.tar.gz
bzip2 archive.tar
bunzip2 archive.tar.bz2
```

These utilities normally replace the input with the transformed output. `gzip` often runs faster, while `bzip2` can produce a smaller result for some inputs. Actual workloads require measurement.

Tar can invoke a compressor while creating an archive:

```bash
sudo tar -czf etc-backup.tar.gz -C / etc
sudo tar -cjf etc-backup.tar.bz2 -C / etc
tar -tf etc-backup.tar.gz
tar -xf etc-backup.tar.gz -C restore-test
```

The `z` option selects gzip, and `j` selects bzip2. GNU tar can normally detect these formats during listing and extraction, so `-tf` and `-xf` can work without repeating the compression option. The archive name is an argument to `-f`, so placing `f` immediately before the archive name keeps the relationship clear.

The `star` utility provides another tar-compatible archiver with a different option set and overwrite behaviour. Package availability depends on enabled RHEL repositories. An administrator should consult the installed `star` manual rather than assume that every GNU tar option or extraction rule carries across.