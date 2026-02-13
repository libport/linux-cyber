# Using Essential Tools
## RHEL 8 essential tools
Red Hat Enterprise Linux 8 (RHEL 8) administration depends on command line work. Operators typically connect over SSH, inspect and modify configuration stored as text files, and secure those files with permissions and ownership. The same core skills apply whether the system runs on a workstation, a server rack, or a virtual machine.

Practical work relies on tools and patterns for:
- Building a reliable RHEL 8 lab
- Navigating and working efficiently in a shell
- Creating, editing, and searching text files
- Controlling access with file permissions, ownership, and special mode bits
- Archiving and compressing data for transfer and backup
## Building a lab system
A useful lab behaves like production. It must support package installation and updates, which usually means registering the machine so it can access Red Hat repositories.
### Installation-based lab
A conventional lab installs RHEL 8 to physical hardware or a virtual machine.

A typical virtual machine build uses:
- A small boot image to start the installer
- Network access so the installer can retrieve packages and updates
- A minimal software selection, then additional packages installed later

A practical network layout includes:
- A NAT interface for outbound access to registration and repositories
- A second interface, such as host-only, when the host must initiate SSH to the guest

During installation, the operator generally:
- Enables networking and sets interfaces to connect automatically on boot
- Sets a root password and creates a non-root administrative user
- Registers the system during or after installation so repositories become available
### Vagrant-based lab
Vagrant provisions virtual machines consistently and hides most SSH connection details. It is useful for a lab because it creates disposable machines and standardises access.

A typical workflow creates a project directory and initialises a box:

```bash
mkdir -p vagrant/rhel8
cd vagrant/rhel8
vagrant init generic/rhel8
vagrant up
```

Vagrant then connects by SSH using keys:

```bash
vagrant ssh
```

The guest still needs repository access. subscription-manager registers the system and attaches an entitlement:

```bash
sudo subscription-manager register --username USERNAME --password PASSWORD --auto-attach
```

Lifecycle commands manage the lab machine:

```bash
vagrant status
vagrant halt
vagrant up
vagrant destroy
```

The Vagrant project directory is the control point for these commands. Keeping one directory per lab machine reduces confusion and avoids operating on the wrong target.
### Subscription notes and CentOS context
RHEL package access and updates are tied to subscription entitlements. A no-cost developer entitlement can suit a personal lab and provides self-supported access for multiple registered systems under an individual account.

CentOS Linux 8 previously provided a community rebuild of RHEL 8, but it reached end of life on 31 December 2021. CentOS Stream continues, but it tracks ahead of RHEL. A lab intended to mirror RHEL behaviour and certification tasks benefits from running RHEL directly.
## Working at the command line
Linux administration is faster when the operator understands how commands are structured and how the shell expands input.
### Command structure and case sensitivity
Commands and paths are case sensitive. ls, LS, and Ls are different strings, and only one of them typically exists.

A command line usually includes:
- A command name
- Optional options or switches
- Optional arguments such as paths or patterns

Examples:

```bash
ls
ls -a
ls -l /etc
```

Options can often be combined:

```bash
ls -la /etc
```

Short options generally use a single hyphen. Many tools also support long options with double hyphens:

```bash
ls --help
```
### Help and documentation
Most commands provide a short help summary:

```bash
COMMAND --help
```

The manual system provides detailed references:

```bash
man COMMAND
man man
```

Manual pages open in a pager, commonly less. Exit with q.

Packages often ship additional documentation under /usr/share/doc:

```bash
ls /usr/share/doc
```
### Tab completion and keyboard shortcuts
Tab completion reduces typing errors, especially for long paths:
- Typing a unique prefix plus Tab completes a directory name
- Typing a partial path plus Tab completes file system components

Common shortcuts speed up routine work:
- Ctrl+L clears the screen while keeping the session active
- Esc then . inserts the previous command's last argument, which avoids retyping long paths
### Pipes, quoting, and command substitution
The shell can connect commands with pipes. A pipe sends standard output from one command to standard input of the next. This supports fast filtering without creating intermediate files.

Examples:

```bash
ip -4 addr show | grep inet
ss -ntl | grep ':22 '
```

Single quotes preserve the literal value of most characters, which is useful for regular expressions. Double quotes preserve spaces while still allowing variable expansion.

Command substitution runs a command first and inserts its output as an argument. The modern form uses $():

```bash
ls -l "$(tty)"
```
## Remote access with SSH
SSH provides a secure remote shell for servers and lab machines. A standard workflow identifies the target, verifies the service, and connects.
### Identify system details and network address
hostnamectl reports system identity and operating system information:

```bash
hostnamectl
```

ip displays interface addresses:

```bash
ip -4 addr show
ip a
```

The output typically includes:
- lo for loopback
- One or more interfaces such as enp0s3 and enp0s8
- IPv4 addresses with their prefixes
### Confirm the SSH service is listening
ss lists sockets. The following view shows listening TCP ports without name resolution:

```bash
ss -ntl
```

If the system listens on port 22, an SSH client can connect:

```bash
ssh user@192.0.2.10
```

The first connection usually prompts to trust the host key. After acceptance, the key is recorded in the client known hosts file, and later connections validate the key to detect impersonation or man-in-the-middle risks.

End the session cleanly:

```bash
exit
```

tty identifies the terminal device. A local console may show /dev/tty1, while an SSH session usually shows a pseudo terminal such as /dev/pts/0:

```bash
tty
```
## Software management with yum and dnf
RHEL 8 uses the YUM stack based on DNF technology. The yum command remains for compatibility and behaves as an alias to dnf.

A lab should confirm access to online repositories before attempting installs:

```bash
sudo yum repolist
```

Common operations include installing, removing, and updating packages:

```bash
sudo yum install PACKAGE
sudo yum remove PACKAGE
sudo yum update
```

After applying updates, a reboot may be required when the kernel or core libraries change. uname -r reports the running kernel version and can confirm that the system booted into the expected release. Reboot planning should consider maintenance windows and service dependencies.

```bash
uname -r
```

The -y option assumes yes for prompts, which suits automation and labs:

```bash
sudo yum install -y nano vim bash-completion
```

bash-completion improves tab completion for many commands, subcommands, and options. It complements path completion and reduces errors in long or unfamiliar command names.

Package queries confirm whether a tool is already installed:

```bash
rpm -q nano
rpm -q vim-enhanced
```

Repository searches help locate package names and descriptions:

```bash
sudo yum search nano
sudo yum info vim
sudo yum list installed | grep vim
```
## Creating and editing text files
Text manipulation sits at the centre of Linux administration. The core workflow creates files, writes content, edits safely, and validates results.
### Creating files and identifying content
touch creates an empty file when it does not exist:

```bash
touch file1
```

file identifies file content type and encoding:

```bash
file file1
```

echo prints text to standard output, which normally appears in the terminal:

```bash
echo hello
```

Shell redirection can send that output to a file.
## Shell redirection and standard streams
Commands write normal output to standard output and error messages to standard error. Both streams display on the terminal by default.


The shell identifies these streams with file descriptors:
- 0 standard input
- 1 standard output
- 2 standard error
### Standard output and standard error in practice
A successful listing produces standard output:

```bash
ls /etc/hosts
```

A failing listing produces standard error:

```bash
ls /etc/Hosts
```

A single command can produce both streams when it mixes valid and invalid arguments.
### Redirecting standard output and standard error
Redirection captures output into files:

```bash
command > output.txt
command >> output.txt
command 2> error.txt
command 2>> error.txt
command &> combined.txt
```

Core behaviour:
- `>` overwrites the destination file
- `>>` appends to the destination file
- `2>` targets standard error, file descriptor 2
- `&>` sends both streams to one destination

A common operational pattern discards errors:

```bash
command 2> /dev/null
```
### Here-documents for input
A here-document feeds multi-line input to a command:

```bash
cat <<'END' > story.txt
Line 1
Line 2
END
```

The quoted delimiter prevents variable expansion inside the block, which reduces surprises when the content includes characters the shell would otherwise interpret.
### tee for logging and privileged writes
tee duplicates standard input to standard output and to a file. It writes the file while still showing the content on screen:

```bash
echo "hello" | tee file1
echo "hello" | tee -a file1
```

tee also solves a privilege issue with redirection. The shell processes redirection before sudo applies, so this pattern fails for protected targets:

```bash
sudo echo "1.0.0.1 cf" >> /etc/hosts
```

This pattern succeeds because tee performs the file write while running under sudo:

```bash
echo "1.0.0.1 cf" | sudo tee -a /etc/hosts
```
## Text editors
Editors must support safe changes to configuration and predictable file encoding.
### nano
nano provides a straightforward interface with a visible shortcut menu:

```bash
nano story.txt
sudo nano /etc/hosts
```

Key controls:
- Ctrl+X exits
- Y confirms saving when prompted
- Enter accepts the file name
### vim
vim is modal. Normal mode issues commands, insert mode edits text, and command mode saves and quits.

Open a file:

```bash
vim story.txt
```

Learn interactively:

```bash
vimtutor
```

Common actions:
- `i` inserts at the cursor
- `a` appends after the cursor
- `I` inserts at the start of the line
- `A` appends at the end of the line
- `Esc` returns to normal mode
- `:x` writes and exits (so does `:wq`)
- `:q!` exits without saving (`:q` will prompt you about unsaved changed)

In normal mode, vim also supports fast navigation and recovery:
- `/pattern` searches forward in the file
- `n` moves to the next match
- `u` undoes the last change
- `dd` deletes the current line
- `yy` copies the current line
- `p` pastes after the cursor
## Directories, paths, and file operations
Accurate path handling prevents mistakes, especially around destructive commands.
### Working directory and path basics
pwd prints the current working directory:

```bash
pwd
```

cd changes directories:

```bash
cd /usr/share/doc
cd
cd -
```

Key behaviour:
- cd with no arguments returns to the user home directory
- cd - returns to the previous working directory
- Absolute paths begin with /
- Relative paths start from the current working directory

The tilde expands to the current user home directory:

```bash
cd ~
mkdir ~/work
```
### Creating and removing directories
mkdir creates directories:

```bash
mkdir dir1
```

mkdir -p creates parent directories when required and avoids errors if they already exist:

```bash
mkdir -p dir1/dir2
```

rmdir removes empty directories:

```bash
rmdir emptydir
```

rm removes files, and rm -r removes directory trees. rm -rf forces removal and suppresses prompts:

```bash
rm file.txt
rm -rf dir1
```

A safer workflow confirms the target before running destructive commands, especially when the prompt does not clearly show the working directory.
### Copy, move, rename, and delete
cp copies files:

```bash
cp source.txt dest.txt
cp source.txt /path/to/dir/
```

mv moves or renames:

```bash
mv oldname newname
mv file.txt /path/to/dir/
```

rm deletes a name from a directory:

```bash
rm file.txt
```

Interactive and verbose options reduce mistakes during manual administration:

```bash
cp -v source.txt dest.txt
mv -i oldname newname
rm -i file.txt
```

Permissions required for these operations often surprise new administrators. Deleting a file depends on write permission on the containing directory, not on the file itself.
### Globbing and quoting
The shell expands glob patterns before running a command:
- * matches any number of characters
- ? matches a single character

Examples:

```bash
ls *
ls file?
ls files*
```

Quoting prevents expansion and protects special characters. It is essential when a pattern must reach a tool such as grep without the shell interpreting it:

```bash
grep 'bash$' /etc/passwd
```
## Reading and searching text
Linux stores most configuration as text, so quick reading and focused search reduce troubleshooting time.
### Reading tools
cat prints the entire file, which suits short files:

```bash
cat /etc/hosts
```

head shows the first lines, and tail shows the last lines. Both default to 10 lines and accept -n:

```bash
head /etc/passwd
tail -n 2 /etc/passwd
```

less pages through long files:

```bash
less /etc/services
```

less navigation keys used frequently:
- Space moves forward one page
- b moves back one page
- /pattern searches forward
- n repeats the last search
- q exits

wc counts lines, words, and bytes. -l counts lines:

```bash
wc -l /etc/services
```
### grep and regular expressions
grep finds lines that match a pattern. It matches case sensitively by default.

A broad match can return unexpected lines:

```bash
grep root /etc/passwd
```

Anchors narrow matches:
- ^root matches lines that start with root
- bash$ matches lines that end with bash

Examples:

```bash
grep '^root:' /etc/passwd
grep 'bash$' /etc/passwd
```

/etc/passwd stores account metadata such as user name, user ID, primary group ID, home directory, and login shell. It is readable by all users on most systems. Password hashes live in /etc/shadow, which restricts access to privileged users.

Protected configuration files often require sudo for reading:

```bash
sudo grep '^Password' /etc/ssh/sshd_config
```

Useful grep options include:

```bash
grep -n '^root:' /etc/passwd
grep -i 'password' /etc/ssh/sshd_config
grep -c 'bash$' /etc/passwd
```

- -n prints line numbers
- -i ignores case
- -c prints a count of matching lines

Extended regular expressions use -E. A common administrative filter removes comments and blank lines to reveal active configuration settings:

```bash
sudo grep -vE '^(#|$)' /etc/ssh/sshd_config
```
## File metadata, permissions, and ownership
Linux uses discretionary access controls based on file mode bits and ownership. Understanding how the kernel evaluates access is essential for securing configuration and diagnosing failures.

The permission model described here applies to native Linux filesystems. Non-native filesystems such as FAT variants do not support the same mode bits. RHEL filesystems such as XFS can also extend access control with access control lists (ACLs), which grant permissions to additional users and groups beyond the single owner and group. Standard mode bits provide the baseline covered here.
### Inspecting metadata
ls -l shows permissions, link count, owner, group, size, and timestamps:

```bash
ls -l /etc/hosts
```

ls -ld shows directory metadata rather than listing its contents:

```bash
ls -ld /etc
```

stat provides detailed metadata and can format output. Octal permissions can be extracted with %a and symbolic permissions with %A:

```bash
stat /etc/hosts
stat -c '%a %A %U:%G %n' /etc/hosts
```

The shell can evaluate a command first and substitute its output as an argument. This supports dynamic inspection, such as listing metadata for the current terminal device:

```bash
ls -l "$(tty)"
```
### File types
The first character of the permission field indicates file type:
- - regular file
- d directory
- l symbolic link
- c character device
- b block device

File type influences how execute and directory traversal work, and it shapes which operations are valid.

The remaining permission string contains three blocks of three characters:
- The first block applies to the owner
- The second block applies to the group
- The third block applies to others

An example long listing shows the structure:

```bash
ls -l /etc/hosts
```

The second field in the long listing is the link count. For regular files it reflects the number of hard links. For directories it reflects the number of directory entries pointing at the inode, which increases as subdirectories are created.

Inode numbers help confirm identity when working with links:

```bash
ls -i /etc/hosts
```
### Permission evaluation order
When a process accesses a file, the kernel applies permissions in order:
- If the process user ID equals the file owner, user permissions apply
- Otherwise, if any process group ID equals the file group, group permissions apply
- Otherwise, other permissions apply

This explains why a user in the correct group can access a file even when other permissions deny access.


A quick troubleshooting approach checks access in this order:
- Confirm the path exists and check its owner and group
- Confirm execute permission exists on every directory in the path
- Confirm the effective user and group IDs for the process
- Confirm the mode bits for the relevant class, then test again

root can bypass most discretionary access controls, but hardened environments can add additional controls. Practical troubleshooting still starts with mode bits, ownership, and group membership.
### Permission bits and octal mapping
Each of user, group, and others has three bits:
- r is 4
- w is 2
- x is 1

Common combinations:
- 7 is 4+2+1 and maps to 111 in binary
- 6 is 4+2 and maps to 110 in binary
- 5 is 4+1 and maps to 101 in binary

Examples and meaning:
- 644 gives owner read and write, group read, others read
- 600 gives owner read and write, no access for group or others
- 755 gives owner full access, group and others read and execute

Execute permission differs by type:
- Files require x to run a program or script
- Directories require x to enter and traverse
- Directories require r to list names
- Directories require w and x together to create and delete entries

A directory can allow traversal without listing by granting x without r. This supports controlled sharing where a user can access a known file name but cannot discover other names in the directory.
### Default permissions and umask
New files start from 666 and new directories start from 777. The umask removes bits from those defaults.

View the current umask:

```bash
umask
```

The shell often prints umask with a leading zero, and some tools display four digits where the first digit relates to special mode bits.

A common user umask is 002. It removes write for others, producing typical defaults:
- Files: 666 minus 002 equals 664
- Directories: 777 minus 002 equals 775

A restrictive umask such as 077 removes all group and other access:
- Files: 666 minus 077 equals 600
- Directories: 777 minus 077 equals 700

Changing umask affects only newly created objects:

```bash
umask 077
touch private.txt
mkdir private.d
ls -l private.txt private.d
```
### Changing permissions with chmod
chmod changes mode bits. Numeric mode sets the entire permission block. Symbolic mode adjusts selected bits.

Numeric examples:

```bash
chmod 644 file.txt
chmod 600 ~/.ssh/id_rsa
chmod 755 ~/bin/tool.sh
```

Symbolic mode supports targets and operators:
- Targets include u, g, o, and a
- Operators include +, -, and =
- Permissions include r, w, and x

Examples:

```bash
chmod o+w file.txt
chmod go-rw file.txt
chmod u=rw,go=r file.txt
chmod a+x tool.sh
```

Recursive changes apply to a directory tree:

```bash
chmod -R go-rw dir1
```

Conditional execute helps avoid accidental execution bits on data files:

```bash
chmod +X -R dir1
```

+X adds execute only where execute is already set or where the target is a directory.
### Ownership and groups
Ownership includes a user owner and a group owner.

Inspect identity and group membership:

```bash
id
groups
```

RHEL typically uses a private group scheme where a new user has a same-named primary group. Newly created files normally inherit that primary group, unless a directory enforces different group behaviour.

Change owner and group together:

```bash
sudo chown root:root file.txt
```

Change owner only:

```bash
sudo chown root file.txt
```

Change group only:

```bash
sudo chgrp root file.txt
```

A convenient pattern sets the group to the owner default group by omitting the group after the colon:

```bash
sudo chown vagrant: file.txt
```

Ordinary users can change a file group only to a group they belong to. Changing the user owner generally requires root privileges.
## Links and identity
Links provide additional names or pointers in the filesystem.
### Hard links
A hard link creates an additional directory entry for the same inode. Both names share the same data and metadata.

Create and inspect:

```bash
ln original.txt hardlink.txt
ls -li original.txt hardlink.txt
```

Key properties:
- Both names show the same inode number
- The link count increases
- The data remains until the link count reaches zero

Hard links cannot cross file systems and are not normally used for directories.
### Symbolic links
A symbolic link creates a separate inode that points to a path. It can cross file systems and can point to directories, but it can break if the target path changes.

Create and inspect:

```bash
ln -s /etc/hosts hosts.link
ls -l hosts.link
readlink hosts.link
```
## Switching identities to test access
Testing permissions is more reliable when the operator becomes the affected identity.

Basic identity checks:

```bash
whoami
id
```

Switch to another user with a login shell:

```bash
su - otheruser
```

Start an interactive root shell when sudo permits:

```bash
sudo -i
```

Change the active primary group in the current session:

```bash
newgrp somegroup
```
## Special mode bits and shared directories
Special mode bits modify execution and shared write behaviour. They appear in permission strings as s or t and map to an additional octal digit in front of the usual three.

The special bits also appear as an extra octal digit:
- 4xxx sets setuid
- 2xxx sets setgid
- 1xxx sets the sticky bit

Examples:

```bash
chmod 4755 /path/to/program
chmod 2775 /shared/project
chmod 1777 /shared/dropbox
```
### setuid and setgid
setuid runs an executable with the file owner identity. setgid runs an executable with the file group identity.

Set bits with chmod:

```bash
chmod u+s /path/to/program
chmod g+s /path/to/program
```

In ls -l output, s replaces x in the relevant position when set.
### setgid on directories
setgid on a directory can force group inheritance for new files. This supports shared project directories that must keep all content owned by a team group:

```bash
chmod g+s /shared/project
```
### Sticky bit
The sticky bit on a directory restricts deletion and renaming so users can remove only their own files, even when the directory is writable by multiple users. A common pattern uses 1777:

```bash
chmod 1777 /shared/dropbox
```

In ls -l output, t replaces x in the other execute position when set.
### Write-only and traverse-only patterns
A user can write to a file without reading it if permissions allow w but not r for that identity. A user can enter a directory without listing it if permissions allow x but not r. These patterns can support simple logging or controlled sharing, but they require careful ownership design and, for shared directories, the sticky bit to prevent deletion by other users.
## Archiving and compression
Archives bundle files and directories into a single stream. Compression reduces size for storage and transport.
### tar
tar is the standard Linux archiver. It can create, list, and extract archives.

Create an archive:

```bash
tar -cf archive.tar dir1
```

List contents:

```bash
tar -tf archive.tar
```

Extract into the current directory:

```bash
tar -xf archive.tar
```

Extract to a target directory:

```bash
tar -C /target -xf archive.tar
```

Verbose output helps during learning and troubleshooting:

```bash
tar -cvf archive.tar dir1
tar -xvf archive.tar
```

Operators generally avoid archiving with leading / paths unless an exact restore to the same location is intended. Excluding paths during creation helps produce a shareable archive:

```bash
tar --exclude='*.log' -cf archive.tar dir1
```
### star
star provides similar archive functionality to tar and may appear in some environments. Operators should follow local standards when exchanging archives across teams.
### Compression tools
Common compressors include gzip, bzip2, and xz. tar can invoke them directly:

```bash
tar -czf archive.tar.gz dir1
tar -cjf archive.tar.bz2 dir1
tar -cJf archive.tar.xz dir1
```

Extracting compressed tar archives uses matching flags:

```bash
tar -xzf archive.tar.gz
tar -xjf archive.tar.bz2
tar -xJf archive.tar.xz
```

Standalone compressors operate on individual files and typically replace the original with the compressed form:

```bash
gzip file.txt
bzip2 file.txt
xz file.txt
```

The matching decompression tools restore the original name:

```bash
gunzip file.txt.gz
bunzip2 file.txt.bz2
unxz file.txt.xz
```