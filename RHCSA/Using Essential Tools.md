# Using Essential Tools
> [!NOTE]
> This document is a practical introduction to essential RHEL/Linux command-line administration, covering lab setup, SSH access, shell help, redirection, text editing, file navigation and manipulation, `grep`, permissions and ownership, links and identity changes, special permission patterns, and archiving/compression workflows with `tar`, `gzip`, and `bzip2`.
## Building a lab environment
A practical RHEL lab can start in two ways. The first uses a fresh virtual machine and a minimal RHEL installation. The second uses a prebuilt image managed by Vagrant. Both approaches work. The best choice depends on whether the goal is to practise the installer itself or to start using the shell quickly.

A clean installation gives full control over language, storage, users, software selection, and networking. A minimal install keeps the footprint small and reduces unnecessary packages. For training, that approach suits a simple virtual machine on VirtualBox or another hypervisor. A lab host benefits from two network paths when possible. One interface can provide outbound connectivity for package access and registration. Another can make SSH access from the local workstation easier.

A Vagrant-based lab removes much of the setup overhead. Vagrant can download a base image, create the machine, boot it, and manage SSH access with minimal manual work. That approach is useful when the priority is command-line practice rather than installer familiarity. Vagrant also makes rebuilds predictable because the same definition can provision the same machine again.

Red Hat still provides a no-cost Developer Subscription for Individuals, which gives individual developers access to RHEL and related resources. That makes a proper RHEL lab possible without relying on an equivalent downstream clone. CentOS Stream remains a distinct platform. It sits upstream of future RHEL content and is not a drop-in substitute for learning the exact behaviour of a supported RHEL release.

Subscription registration is required to access package repositories and updates. The registration workflow uses `subscription-manager` to attach the system to an account and enable the standard software channels. On RHEL, the core repositories typically include BaseOS for the underlying operating system and AppStream for user-space components and application streams.

A basic Vagrant workflow looks like this:

```bash
mkdir -p ~/vagrant/rhel8
cd ~/vagrant/rhel8
vagrant init generic/rhel8
vagrant up
vagrant ssh
```

A basic registration workflow on a newly installed or newly provisioned system looks like this:

```bash
sudo subscription-manager register --username <username> --password <password> --auto-attach
sudo dnf repolist
```

In RHEL, `yum` remains as a compatibility interface, but `dnf` is the underlying package manager. Either command works for many common tasks. Administrators should still recognise that current RHEL documentation uses `dnf` terminology.
## Connecting to the system and reading host information
A Linux host can be accessed directly at the system console or remotely over SSH. Physical or virtual console access is useful during installation and recovery. Remote shell access is the normal method for everyday administration because it works consistently across cloud instances and local labs.

Before opening a remote session, an administrator usually checks the hostname, interface addresses, and listening network services. These commands establish the basic identity and reachability of the host:

```bash
hostnamectl
ip -4 addr show
ss -ntl
```

`hostnamectl` reports the host name and operating system details. `ip -4 addr show` lists IPv4 addresses on each interface. `ss -ntl` shows listening TCP sockets without name resolution. A listening service on port 22 normally confirms that the OpenSSH server is available.

A basic SSH login uses the form below:

```bash
ssh user@host
```

If the host uses a private lab address, the `host` field can be the IP address. If DNS resolves the host name correctly, the host name can be used instead. The first connection records the server key in the local known-hosts file. After that, the client checks the key on future connections to detect unexpected changes.

Linux commands follow a predictable structure. The command name comes first. Options usually follow. Arguments identify targets. For example, `ls` by itself lists the current directory. `ls -a` includes hidden entries that begin with a dot. `ls -l /etc` requests a long listing of `/etc`. That pattern repeats across most command-line tools.

Shell efficiency depends on a few habits:
- use tab completion to reduce typing and prevent path errors
- use clear, explicit paths when the current directory is uncertain
- use `Ctrl-L` to clear a crowded screen
- use `tty` to identify the current terminal device when needed

The shell also provides access to built-in help and full manuals. Short option help often appears with `--help`. Longer reference material appears in the manual pages. Package-specific documentation often lives under `/usr/share/doc`.

```bash
ip --help
man ip
man man
ls /usr/share/doc
```

Manual pages use a pager, usually `less`. Common navigation keys include the arrow keys, Page Up, Page Down, `/pattern` for searching, and `q` to quit.
## Redirecting shell input and output
Redirection is central to Linux administration because almost every tool can read from standard input and write to standard output or standard error. The shell connects those streams to the terminal by default, but it can also send them to files, pipes, or other commands.

Standard output carries normal results. Standard error carries errors and diagnostic messages. The shell usually treats them as separate streams:
- `>` writes standard output to a file and overwrites the file
- `>>` appends standard output to a file
- `2>` writes standard error to a file
- `2>>` appends standard error to a file
- `&>` sends both standard output and standard error to the same file in shells that support that syntax

Simple file creation often starts with `touch`, which creates an empty file if it does not already exist. Content can be written with `echo` and redirection:

```bash
touch file1
echo "hello" > file1
echo "again" >> file1
```

That sequence creates a file, writes one line, then appends another line. Overwrite and append differ in an important way. `>` replaces the existing contents. `>>` preserves them and adds more text at the end.

Administrators often test the difference between successful output and error output with a command such as `ls`. Listing a real file produces standard output. Listing a missing file produces standard error. Separate redirection makes that difference visible and useful.

A pipeline sends the standard output of one command to the standard input of the next command. That makes simple text-processing chains possible:

```bash
ls /etc | less
```

The shell can also feed multi-line input directly into a command through a here-document. That is useful in scripts or one-off configuration tasks because it allows a block of literal text to be created without opening an editor.

```bash
cat > story.txt <<'EOF'
Line 1
Line 2
EOF
```

The marker after `<<` defines the end of the text block. Everything up to that marker becomes the command input.

The `tee` command solves a common privilege problem. Redirection happens in the current shell. That means `sudo echo "entry" >> /etc/hosts` fails for a regular user because the shell, not `echo`, attempts the write to `/etc/hosts`. The correct pattern pipes the text to `tee`, which runs with elevated privileges:

```bash
echo "1.0.0.1 cf" | sudo tee -a /etc/hosts
```

`tee` writes the content to the screen and to the target file. The `-a` option appends instead of overwriting. This pattern is safer and clearer than trying to force privilege escalation into the shell redirection itself.
## Creating, editing, and navigating text files
Text files drive much of Linux administration. Configuration files, logs, service definitions, and shell scripts all depend on accurate text editing.

`touch` creates empty files. Redirection creates or updates files while commands run. Those methods work well for short changes. For sustained editing, a text editor is faster and less error-prone.

Two common editors in RHEL environments are `nano` and `vim`. `nano` is simple and menu-driven. It displays key prompts at the bottom of the screen, which makes it a practical choice for new administrators or quick edits. `vim` is more powerful and more efficient once learned, but it requires an understanding of modes.

A typical package installation sequence is:

```bash
sudo dnf repolist
sudo dnf install -y nano vim bash-completion
```

`bash-completion` is not an editor, but it improves shell completion and makes command-line work faster.

A typical `nano` session is direct:

```bash
nano story.txt
```

Within `nano`, the administrator types normally, uses the arrow keys to move, presses `Ctrl-X` to exit, confirms saving when prompted, and writes the file name if needed. When editing protected files, `sudo nano /etc/hosts` or a similar command opens the file with the required privileges.

`vim` starts in normal mode. In that mode, keystrokes act as commands rather than text input. Common entry points into insert mode include:
- `i` to insert before the cursor
- `a` to append after the cursor
- `I` to insert at the start of the line
- `A` to append at the end of the line

After editing, `Esc` returns to normal mode. `:wq` writes and quits. `:q!` quits without saving. `vimtutor` provides an interactive introduction and is worth using before any serious editing session.

```bash
vimtutor
vim story.txt
```

Short files suit `cat`. Large files suit `less`. The beginning and end of files can be inspected quickly with `head` and `tail`.

```bash
cat /etc/hosts
head /etc/passwd
tail -n 20 /var/log/messages
less /etc/services
wc -l /etc/services
```

`wc -l` counts lines, which helps when judging file size or verifying that a generated file has the expected number of records.

Directory navigation underpins all file work. The shell always has a current working directory. `pwd` prints it. `cd` changes it. `cd` with no argument returns to the user’s home directory. `cd -` returns to the previous working directory, which is one of the fastest ways to switch between two locations.

```bash
pwd
cd /usr/share/doc
cd
cd -
```

`mkdir` creates directories. `mkdir -p` creates parent directories as needed. `rmdir` removes empty directories only. `rm -r` removes a directory tree recursively, and `rm -rf` suppresses prompts and errors for many cases.

```bash
mkdir -p ~/work/dir1/dir2
rmdir emptydir
rm -r olddir
```

File operations rely on `cp`, `mv`, and `rm`. Globbing expands patterns before the command runs. `*` matches any string of characters. `?` matches a single character. Braces such as `{1..12}` perform shell expansion rather than filename matching.

```bash
cp /etc/hosts ~/hosts.copy
mv hosts.copy hosts.test
rm hosts.test
touch files{1..12}
ls files*
ls files?
ls files??
```

Permission requirements matter during file operations. To copy a file, the process needs read access to the source file and write plus execute access to the destination directory. To move or rename a file within a directory tree, the operation depends mainly on permissions on the directories involved, not on write access to the file itself. To delete a file, the process needs write and execute access to the parent directory. That distinction explains why a user can sometimes delete a read-only file if the directory permissions allow it.
## Searching text with grep and regular expressions
Text search becomes powerful when it combines plain strings with simple regular expressions. `grep` scans lines for matches and prints the matching lines. That makes it ideal for checking configuration files, looking for users or services, and isolating effective settings from commented defaults.

A plain search finds any line that contains the target text:

```bash
grep root /etc/passwd
```

That result can include unintended matches. For example, a search for `root` also matches a line that ends in `/root`. Anchors solve that problem. `^` anchors the pattern to the start of the line. `$` anchors it to the end.

```bash
grep '^root' /etc/passwd
grep 'bash$' /etc/passwd
```

Configuration files often contain comments. A plain search can therefore match both comments and real settings. Anchoring the pattern helps identify active directives. For example, a search for lines that start with `PasswordAuthentication` in the SSH daemon configuration shows the effective directive more clearly than a broader search for `password`.

```bash
sudo grep '^PasswordAuthentication' /etc/ssh/sshd_config
```

The search is case-sensitive unless options such as `-i` request case-insensitive matching. That matters because many configuration directives use precise capitalisation. Administrators should search for the exact setting name when possible.

`grep` also works well with pipelines. One command can generate a list and another can narrow it. That workflow is often faster than opening a large file in an editor when the goal is only to confirm one value.
## Understanding file metadata and permissions
Linux stores metadata with each file and directory. Long listings reveal the key fields:

```bash
ls -l file1
stat file1
```

The first character in the permissions string identifies the file type. A dash represents a regular file. `d` represents a directory. `l` represents a symbolic link. The remaining nine permission bits are arranged in three groups of three:
- user permissions for the file owner
- group permissions for the file’s group owner
- other permissions for everyone else

Each triplet uses the same symbols:
- `r` for read
- `w` for write
- `x` for execute

Those permissions apply differently to files and directories. On regular files, read allows viewing contents, write allows modifying contents, and execute allows running a program or script when other requirements are met. On directories, read allows listing entries, write allows creating, deleting, or renaming entries, and execute allows entering the directory and accessing entries by name. Directory write without execute is rarely useful. Directory execute without read can be useful when a path should be traversable but not listable.

Numeric permissions translate the same bits into octal values. `r` equals 4, `w` equals 2, and `x` equals 1. The values add within each triplet. Common examples include:
- `7` for `rwx`
- `6` for `rw-`
- `5` for `r-x`
- `4` for `r--`

A mode such as `644` therefore means read and write for the owner, and read-only for group and others. A mode such as `755` means full access for the owner and read plus execute for group and others.

New files and directories start from default creation modes and then lose permissions through the shell’s `umask`. In practice, it is safer to inspect the current `umask` than to assume a universal default, because distributions, login profiles, and site policies can change it. A common collaborative setting removes write permission from others. More restrictive settings remove group write as well.

```bash
umask
touch file2
mkdir dir2
ls -ld file2 dir2
```

The key rule is that `umask` removes permissions from the creation defaults. It does not add permissions beyond those defaults. Regular files normally start without execute bits. Directories normally include execute because directory traversal requires it.

`chmod` changes permissions. It accepts numeric modes and symbolic expressions. Numeric modes replace the full permission set explicitly. Symbolic modes add, remove, or assign bits more selectively.

```bash
chmod 644 file1
chmod u=rw,go=r file1
chmod o+w file1
chmod g-w file1
```

The symbolic forms use `u`, `g`, `o`, and `a` for user, group, other, and all. They combine with `+`, `-`, or `=` and the permission letters. An uppercase `X` has a specialised meaning. It adds execute only to directories and to files that already have at least one execute bit set. That is useful when fixing directory trees without accidentally marking ordinary text files as executable.

```bash
chmod -R a+X projectdir
```
## Ownership, links, and identity changes
Permissions depend on ownership as well as mode bits. Every file has a user owner and a group owner. Long listings display both. The `id` command shows the current user ID, primary group ID, and supplementary groups.

```bash
id
ls -l file1
```

RHEL commonly creates a private primary group for each user, where the user name and the default group name match. New files usually inherit the current effective primary group of the process that created them.

`chown` changes ownership. `chgrp` changes group ownership. Root privileges are normally required to change the user owner. Group changes may be possible when the user belongs to the target group and local policy permits it.

```bash
sudo chown alice file1
sudo chown alice:wheel file1
sudo chgrp wheel file1
```

Links provide alternate names for files. Hard links point directly to the same inode as the original file. Symbolic links store a path to another file or directory. Hard links generally stay on the same filesystem and are normally restricted to regular files. Symbolic links can cross filesystems and can point to directories, which makes them the normal choice for shortcuts and compatibility paths.

```bash
ln original.txt hardlink.txt
ln -s /etc/services services
```

A symbolic link shows `l` as the file type in `ls -l`, along with an arrow to the target path. Removing a symbolic link does not remove the target. Removing one hard link only removes one name. The underlying file content remains until all hard links are removed and no process still holds the file open.

Administrators also need to understand user and group identity changes inside a shell session. `newgrp` starts a new shell with a different primary group. That affects the group ownership of newly created files. `su` switches user identity. `su -` starts a full login shell for the target account and loads the target user’s environment. That form is usually safer for administrative work than plain `su`.

```bash
newgrp wheel
su -
```

One correction matters here. Adding a user to a supplementary group does not automatically update existing processes in every current session. New logins pick up the new group set. Existing shells usually need a fresh login or a new shell with the required group context before they reflect the change.

For single administrative tasks, `sudo` is usually clearer than switching fully to root because it limits privilege elevation to one command and preserves an audit trail.
## Special permission patterns for controlled access
Standard read, write, and execute bits can support some useful access patterns without introducing more advanced controls. Two examples stand out.

A write-only file can accept appended data from a user while blocking that same user from reading the existing contents. That pattern suits simple logging or drop-box style input where many users must add records but should not read each other’s entries. The owner or root can still inspect the file as needed.

An execute-only directory allows traversal without directory listing. If a user knows the full file name and the file itself grants appropriate access, that user can open the file through the directory path. Without read permission on the directory, the user cannot list the directory contents. If the directory grants write and execute, the user can create and remove entries even without directory read permission. That pattern is useful when a directory must accept known-path access but should not expose its full contents casually.

These patterns rely on exact directory semantics. For directories:
- read controls listing
- write controls entry creation, removal, and renaming
- execute controls traversal and named access

That behaviour often surprises administrators who learned file permissions first and then assumed directories follow the same rules.
## Creating archives with tar and star
Archiving groups files into a single file. Compression reduces size. The two jobs often appear together, but they are not the same thing. A tar archive can exist without compression. That distinction matters because an archive is often useful even when storage space is not the main concern.

`tar` remains the standard archiving tool on Linux. It creates a single archive from one or more files or directories, preserves paths relative to the invocation point, and can later list or extract the stored content.

Common `tar` operations include:
- `-c` to create an archive
- `-t` to list archive contents
- `-x` to extract an archive
- `-f` to specify the archive file name
- `-v` for verbose output when needed

Typical examples are:

```bash
tar -cf myetc.tar etc
tar -tf myetc.tar
tar -xf myetc.tar
```

Because `tar` stores many files inside one larger file, it can sometimes appear slightly smaller than the original directory tree even before compression. That effect comes from block allocation and metadata overhead in the filesystem rather than from true data compression.

Extraction deserves care. By default, `tar` can overwrite files depending on path, timestamps, options, and local conditions. Administrators should extract into a controlled destination or inspect the archive first when working with valuable data.

The course material also mentions `star`, another archiver. `star` offers similar create, list, and extract operations and may provide different defaults or performance characteristics. It is not always installed by default, so `tar` remains the practical baseline command for most RHEL systems.
## Compressing archives with gzip and bzip2
Compression utilities reduce file size by encoding repeated or predictable data more efficiently. Two classic tools are `gzip` and `bzip2`. Their matching decompression tools are `gunzip` and `bunzip2`.

Used directly, they compress an existing file:

```bash
gzip etc.tar
gunzip etc.tar.gz

bzip2 etc.tar
bunzip2 etc.tar.bz2
```

The choice involves a trade-off. `gzip` usually runs faster. `bzip2` often produces a smaller file, but it tends to use more CPU time. The best option depends on whether speed or storage reduction matters more for the task at hand.

`tar` can call these tools during archive creation. That combines packaging and compression into one step:

```bash
tar -czf etc.tar.gz etc
tar -xzf etc.tar.gz

tar -cjf etc.tar.bz2 etc
tar -xjf etc.tar.bz2
```

With modern `tar`, extraction often detects the compression format automatically, so the explicit `z` or `j` is sometimes unnecessary on extraction. Even so, explicit options improve clarity in training and scripts.

Compression is especially effective on text-heavy content such as configuration files, logs, source code, and many plain-text datasets. It is less effective on data that is already compressed, such as many images, videos, or archives created with strong compression beforehand.
## Practical habits for reliable command-line work
Reliable shell work depends less on memorising every option and more on disciplined habits:
- check the current directory before destructive commands
- use tab completion to reduce typing errors
- inspect permissions and ownership before assuming a fault in the application
- distinguish clearly between file permissions and directory permissions
- prefer `sudo` for single privileged actions
- inspect an archive before extracting it when the destination matters
- treat overwrite operations such as `>` and recursive removal operations such as `rm -rf` with extra care
- verify effective configuration values with focused `grep` searches instead of scanning files manually

Those habits reduce mistakes and make troubleshooting faster. They also scale well from small labs to production systems.
## Summary
RHEL administration rests on a small set of repeatable shell skills. A good lab starts with a proper RHEL system, ideally registered so that standard repositories and updates are available. Remote access over SSH, accurate inspection of host identity, and efficient use of manual pages establish a solid working environment.

From there, shell redirection, `tee`, and here-documents make text flow manageable. `nano` and `vim` provide practical editing paths. Directory navigation, file operations, and globbing support daily filesystem work. `grep` and related text tools turn large configuration files into searchable data.

Permissions, ownership, and links explain who can access what and why. `chmod`, `chown`, `chgrp`, `newgrp`, `su`, and `sudo` control identity and access with precision when they are used with a clear understanding of file and directory semantics. Finally, `tar`, `gzip`, and `bzip2` package and compress files for transfer, backup, and recovery.