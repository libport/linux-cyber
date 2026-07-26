# Operating Systems: Overview, Administration, and Security
> [!NOTE]
> Windows, Linux, macOS, iOS, and Android use distinct architectures, administration models, and security controls. Virtualisation, cloud computing, and containers extend these operating system foundations.
## Windows operating systems
### Operating system fundamentals
#### Core functions
- An operating system manages processors, memory, storage, input, output, devices, security boundaries, and application execution.
- Users and administrators interact with the operating system through graphical interfaces, command-line shells, application programming interfaces, and management tools.
- Operating systems developed from batch-processing systems into time-sharing, networked, multitasking, distributed, desktop, server, mobile, and embedded platforms.
- Linux supports servers, cloud platforms, development systems, embedded devices, and desktops through many distributions.
- Windows supports client, server, and embedded environments, with editions designed for different hardware, management, and security requirements.
- macOS is Apple's Unix-based operating system for Mac computers.
- ChromeOS is a Linux-based operating system centred on web applications, Android applications, and managed device fleets.
### Windows platforms and architecture
#### Windows client editions and lifecycle
- Windows editions differ in their security, deployment, management, and hardware capabilities, so organisations should match each edition to its device role and governance requirements.
- Windows Home targets personal use and provides core security features. Compatible devices may also support Windows device encryption.
- Windows Pro adds business features such as Active Directory domain join, Group Policy, Remote Desktop hosting, and BitLocker enablement.
- Windows Enterprise adds advanced deployment, application control, security, and management capabilities for large organisations.
- Windows Pro for Workstations supports specialised, high-performance hardware and demanding workloads.
- Microsoft ended general support for Windows 10 version 22H2 on 14 October 2025. Some Long-Term Servicing Channel editions follow separate lifecycles, and eligible devices may use Extended Security Updates during migration.
- Windows 11 requires a 64-bit processor. Feature availability still depends on the edition, hardware, licensing, and management platform.
#### User mode and kernel mode
- Windows separates execution into user mode and kernel mode to protect the operating system from application faults.
- User-mode processes receive private virtual address spaces and restricted access to hardware and kernel memory. An application failure therefore usually affects that process rather than the whole system.
- Kernel-mode code includes the operating system kernel and many device drivers. It can access protected memory and hardware directly.
- A kernel-mode fault can stop or destabilise the system, so organisations should obtain drivers from trusted sources, test them, and control their deployment.
- Applications request protected services through system calls. Windows validates each request before kernel components perform the operation.
### Storage, files, and directory layout
#### File systems and storage media
- A file system defines how an operating system names, stores, retrieves, protects, and records metadata for files and directories.
- Hard disk drives store data magnetically. Solid-state drives use flash memory and usually provide lower access latency.
- FAT32 remains common where broad device compatibility outweighs security and capacity requirements. Its maximum individual file size is 4 GiB minus 1 byte.
- exFAT supports much larger files than FAT32 and commonly serves removable media that must work across several operating systems.
- NTFS is the standard file system for Windows system volumes. It supports access control lists, journalling, quotas, compression, Encrypting File System, hard links, and the Master File Table.
- A storage format does not replace backups. Hardware failure, deletion, corruption, ransomware, and administrative error can affect any file system.
#### Windows directory structure and 32-bit compatibility
- Windows normally places the system volume on drive `C:` and arranges folders in a hierarchy below it.
- `C:\Users` stores user profiles and folders such as Desktop, Documents, Downloads, and Pictures.
- `C:\Windows` stores operating system files, components, and configuration data.
- On 64-bit Windows, `C:\Program Files` normally stores 64-bit applications, while `C:\Program Files (x86)` normally stores 32-bit applications.
- On 64-bit Windows, `C:\Windows\System32` contains native 64-bit system binaries. The WoW64 compatibility layer redirects many requests from 32-bit applications to `C:\Windows\SysWOW64`, which contains 32-bit binaries.
- Windows protects or hides some operating system files, including `pagefile.sys` for paging and `hiberfil.sys` for hibernation. `C:\PerfLogs` is a directory for performance logs, not a hidden system file.
- A 32-bit process has a much smaller virtual address space than a 64-bit process. Modern 64-bit Windows supports far larger memory configurations, subject to edition and hardware limits.
#### File naming conventions
- Clear file names improve collaboration, search, backup recovery, and incident response.
- Short, descriptive, and consistent names work best across a project or records system.
- Spaces and reserved punctuation can complicate scripts and command-line work, so many organisations use hyphens and restrict characters that carry special meaning.
- Fixed-width dates in `YYYY-MM-DD` format sort chronologically when every name follows the same convention.
- Version labels should distinguish working drafts, approved releases, and superseded files without relying on ambiguous terms such as `final-final`.
- Identifiers should avoid confusable characters, such as a lower-case `l`, an upper-case `I`, and the numeral `1`, when confusion could create operational risk.
### Windows administration foundations
#### Microsoft Management Console
- Microsoft Management Console hosts administrative snap-ins within a consistent interface.
- Administrators can add, arrange, and save snap-ins in custom console files that suit particular support or delegation tasks.
- Common snap-ins include Computer Management, Event Viewer, Device Manager, Disk Management, Local Users and Groups, Services, Task Scheduler, Performance Monitor, Group Policy Management, and Certificates.
- A custom console does not grant permissions by itself. Windows still enforces the operator's account rights and each tool's access controls.
#### Command-line administration
- Command Prompt runs traditional Windows commands, while PowerShell provides object-based administration and automation. Windows Terminal can host both shells.
- Windows does not reduce local access to two privilege levels. Accounts and processes receive access tokens that reflect group membership, assigned rights, integrity level, and User Account Control elevation.
- Administrators should perform routine work with standard privileges and elevate only approved tasks.
- `gpupdate` refreshes Group Policy, while `gpresult` reports the policies that apply to a user or computer.
- `cd` or `chdir` changes directories, a drive letter followed by a colon changes the active drive, and `vol` displays a volume label and serial number.
- `shutdown`, `sfc`, `chkdsk`, `diskpart`, `winver`, and `format` support system maintenance and inspection. `diskpart` and `format` can destroy data when used incorrectly.
- `dir`, `md`, `rd`, `ren`, and `del` manage files and directories. Administrators should confirm paths and recovery options before deleting content.
- `copy`, `xcopy`, and `robocopy` copy data. `robocopy` can mirror directory trees, which can also propagate deletions when configured to mirror a source.
- `hostname`, `ipconfig`, `ping`, `tracert`, `nslookup`, and `netstat` support basic network diagnosis.
### Identity and directory services
#### User account management in Windows Server
- Windows Server can manage local accounts, while Active Directory Domain Services manages domain identities and computer accounts centrally.
- Administrators should create named accounts, assign access through groups, and avoid shared administrator credentials.
- Account lifecycle tasks include creation, role and group assignment, access review, temporary disablement, and removal after retention requirements are met.
- Disablement normally suits temporary suspension because it preserves the account identifier, group memberships, and audit history.
- Removing an identity from a local directory does not automatically remove linked accounts, licences, sessions, or data in cloud services. Administrators must coordinate each connected system.
- Current Windows Server administration commonly uses Server Manager, Windows Admin Center, PowerShell, Active Directory Administrative Center, and Active Directory Users and Computers. Older Windows Server Essentials releases used a dashboard with simplified user-management workflows.
#### Active Directory Domain Services
- Active Directory Domain Services stores identities, groups, computers, policies, and other directory objects, and it supports centralised authentication and authorisation.
- Organisational units arrange objects for delegation and targeted Group Policy application. They do not form security boundaries.
- Security groups simplify access control by assigning permissions to groups instead of individual accounts.
- Home folders and folder redirection can centralise user data, but they do not replace versioned, tested backups.
- Group Policy Objects apply configuration and security settings to sites, domains, and organisational units.
- A domain partitions directory data and provides an administrative, authentication, policy, and replication scope.
- A domain tree contains domains in a contiguous namespace. A forest contains one or more domains that share a schema, configuration, global catalogue, and transitive trust structure.
- The forest forms the principal security boundary in Active Directory. Compromise of a sufficiently privileged forest account can affect every domain in that forest.
## Windows security
### Account controls and data protection
#### User Account Control
- User Account Control reduces the risk that applications will make unauthorised system-wide changes with administrator privileges.
- Windows gives an administrator's interactive session a filtered token for routine work and requests consent before many elevated actions.
- A consent prompt asks a signed-in administrator to approve elevation.
- A credential prompt asks a standard user to provide credentials for an account authorised to elevate the action.
- User Account Control limits privilege use, but it does not replace application control, patching, endpoint protection, or least privilege.
#### Drive and file encryption
- BitLocker encrypts operating system and fixed data volumes. BitLocker To Go applies BitLocker protection to removable data drives.
- BitLocker can protect keys with a Trusted Platform Module and, where configured, a PIN, startup key, password, or recovery mechanism.
- Organisations should escrow recovery information securely, restrict access to it, and test recovery before relying on encryption.
- Encrypting File System encrypts selected files and folders on NTFS volumes and decrypts them for authorised identities that hold the required certificate and private key.
- Loss of an Encrypting File System private key can make data unrecoverable unless a valid data recovery agent or key backup exists.
- Encryption protects data at rest. It does not protect data after an authorised account or compromised process has decrypted it.
#### Accounts, groups, and least privilege
- User accounts identify people or services and provide the security context for access decisions.
- Groups simplify permission management and reduce inconsistent direct assignments.
- Least privilege gives each account, process, and administrator only the access required for an approved task.
- Administrators should use separate privileged accounts, protect them with phishing-resistant multi-factor authentication where supported, and monitor their use.
#### Windows account types
- A local account exists in one computer's local security database.
- A Microsoft account connects a personal identity to Microsoft cloud services and can synchronise supported settings.
- An Active Directory domain account receives centralised authentication and policy from domain controllers.
- A Microsoft Entra account supports cloud identity and can join or register compatible Windows devices without creating a traditional Active Directory domain account.
- Built-in accounts vary by Windows edition, role, and feature. Administrators should not treat historical accounts such as HelpAssistant as universal defaults on current systems.
#### NTFS and share permissions
- NTFS permissions apply to files and folders on NTFS volumes and support detailed inheritance and access-control entries.
- Share permissions apply when a client reaches a folder through a Windows file share.
- Remote access must pass both the share permissions and the NTFS permissions. The resulting access cannot exceed the more restrictive permission set.
- Effective access also reflects group membership, inheritance, explicit permissions, deny entries, ownership, privileges, and the path used to reach the resource.
- Administrators should grant access to security groups, review inherited permissions, and test effective access with representative accounts.
### Endpoint protection, firewalling, and user awareness
#### Microsoft Defender Antivirus
- Microsoft Defender Antivirus provides built-in real-time protection and supports quick, full, custom, and offline scans on supported Windows editions.
- Scheduled tasks can run supported scan types at defined times. A schedule is not a separate scan type.
- Current security intelligence, platform updates, cloud-delivered protection, and correctly configured exclusions strengthen detection.
- Running several real-time antivirus engines on the same endpoint can cause conflicts, duplicate scanning, and performance problems. Organisations should choose a supported endpoint-protection design.
#### Windows Defender Firewall
- Windows Defender Firewall controls inbound and outbound network traffic through rules associated with domain, private, and public profiles.
- Rules can evaluate applications, services, ports, protocols, interfaces, users, computers, and addresses.
- Administrators should keep the firewall enabled, allow only required traffic, document exceptions, and remove obsolete rules.
- A firewall reduces network exposure but cannot compensate for vulnerable applications, weak credentials, or unsafe administrative practices.
#### Social engineering and phishing
- Social engineering manipulates people through phishing email, voice phishing, text-message phishing, malicious QR codes, impersonation, and lookalike websites.
- Technical controls reduce exposure, but attackers can still abuse legitimate services, stolen sessions, and convincing pretexts.
- Effective programmes combine secure email and web controls, phishing-resistant authentication, simple verification procedures, realistic exercises, and clear reporting channels.
- Training should reinforce cautious action without blaming people who report suspected attacks.
### Patching and update management
#### Patches and updates
- A patch corrects a defect or vulnerability after release.
- An update can include security fixes, reliability changes, drivers, feature changes, or compatibility improvements.
- Organisations should deploy urgent security fixes according to exposure, exploit activity, asset criticality, and vendor guidance rather than applying one deadline to every system.
- Feature updates usually require compatibility testing, capacity planning, communications, and a supported rollback or recovery path.
#### Patch management process
- Patch management identifies assets and applicable updates, obtains content from trusted sources, tests representative systems, deploys in controlled stages, verifies results, and handles exceptions.
- A patch policy should define ownership, risk tiers, testing, maintenance windows, emergency changes, rollback, reporting, and unsupported systems.
- Automation improves coverage and consistency, but administrators still need accurate inventory, monitoring, failure handling, and exception review.
- Backups and recovery plans reduce operational risk, but they do not justify delaying critical security fixes without a documented decision.
#### Windows Update and Windows Server Update Services
- Windows Update delivers operating system, security, quality, and selected driver updates to supported Windows devices.
- Microsoft Update can also supply updates for other Microsoft products when administrators enable that option.
- Centrally managed services can stage deployments, define deadlines, monitor compliance, and pause a rollout when testing identifies a serious issue.
- Windows Server Update Services supports local approval and distribution of Microsoft updates. Microsoft has deprecated it and no longer develops new features, although supported deployments continue to receive security and quality updates under the applicable product lifecycle.
- Organisations planning new update-management architectures should assess current cloud and endpoint-management options instead of assuming that Windows Server Update Services is the default choice.
### Authentication and network access protocols
#### Kerberos
- Kerberos authenticates clients and services with time-limited tickets and symmetric cryptography.
- A Key Distribution Centre provides authentication and ticket-granting services. Active Directory domain controllers perform this role for their domains.
- After initial authentication, a ticket-granting ticket lets a client request service tickets without repeatedly sending the user's password.
- Kerberos supports mutual authentication when the client and service validate each other's identities.
- Kerberos depends on accurate time, correct service principal names, secure key handling, healthy domain services, and protected privileged accounts.
#### Privileged roles and built-in identities
- Domain Admins members hold broad domain privileges and, in a single-forest design, may obtain control that affects the wider forest.
- Enterprise Admins and Schema Admins hold forest-wide capabilities and require especially tight membership control.
- Each Active Directory domain contains a `krbtgt` account for Kerberos ticket protection. Organisations should protect its credentials and follow established procedures if compromise requires a controlled password reset.
- Local Administrator and Guest accounts exist on Windows systems, but their state and management vary by configuration. Organisations should inventory built-in identities instead of relying on an old universal list.
#### Other authentication frameworks and protocols
- RADIUS centralises authentication, authorisation, and accounting for services such as virtual private networks and enterprise Wi-Fi.
- TACACS+ commonly controls administrative access to network devices and separates authentication, authorisation, and accounting functions. It is distinct from the older TACACS protocol.
- Extensible Authentication Protocol provides a framework for network-access authentication. EAP-TLS uses certificates and commonly supports enterprise Wi-Fi and wired access.
- Protocol choice does not secure access by itself. Strong cryptography, certificate validation, protected shared secrets, resilient servers, logging, and least-privilege policy remain essential.
### Security auditing
#### Purpose and design
- Security auditing records selected events so administrators can detect misuse, investigate incidents, support accountability, and demonstrate compliance.
- Audit design starts with critical assets, threat scenarios, legal and operational requirements, and the actions that require evidence.
- Excessive auditing can obscure useful signals and increase storage demand, while insufficient auditing can leave material gaps.
- Organisations should centralise important logs, synchronise time, restrict log access, monitor collection health, define retention, and protect evidence from tampering.
#### Common advanced audit policy categories
- Account Logon records credential validation and Kerberos activity on the system that authenticates the account.
- Logon/Logoff records interactive, network, service, and other session events on the accessed system.
- Account Management records changes to users, computers, and security groups.
- Directory Service Access records selected access to Active Directory objects when administrators configure suitable system access control lists.
- Object Access records selected access to files, folders, registry keys, shares, and other securable objects.
- Policy Change, Privilege Use, Detailed Tracking, System, and other categories capture changes and activity relevant to security investigations.
### Operational priorities
- Identity controls, encryption, endpoint protection, patching, backups, firewalling, and auditing work best as a coordinated control set.
- Least privilege, separate administrative identities, and verified workflows reduce the impact of phishing, malware, and operator error.
- Centralised identity and update services simplify governance, but their reach also makes them high-value targets that require hardening, monitoring, resilience, and tested recovery.
## Linux operating systems
### Linux essentials
#### Organisational uses
- Linux supports servers, cloud workloads, software development, scientific computing, network appliances, embedded systems, and desktops.
- Open-source licensing allows eligible recipients to inspect, modify, and redistribute source code under the relevant licence terms.
- Source availability supports review and adaptation, but it does not guarantee security or code quality. Maintainer practices, configuration, testing, and timely updates remain decisive.
- Distributions combine the Linux kernel with user-space tools, libraries, installers, package repositories, support policies, and desktop environments.
#### Distributions and interfaces
- Common distributions include Debian, Ubuntu, Fedora, Red Hat Enterprise Linux, openSUSE, and Linux Mint.
- A graphical desktop can simplify interactive tasks, while a command-line environment supports remote administration, automation, and low-resource systems.
- Distribution families use different package managers, release models, defaults, and support periods. Administrators should follow documentation for the installed distribution and version.
### Accounts and system information in Ubuntu
#### Local accounts and administrative access
- Administrators should create standard user accounts for routine work and elevate only approved commands.
- Ubuntu normally uses `sudo` for authorised administrative actions rather than encouraging persistent root login.
- Group membership, home-directory permissions, shell access, and service-specific roles determine an account's effective scope.
- Account reviews should identify stale users, unexpected group memberships, unused SSH keys, and services that run with excessive privilege.
#### Device and operating system information
- Ubuntu Desktop exposes hardware, storage, operating system version, and update information through Settings and system-information tools.
- Commands such as `uname`, `hostnamectl`, `lsblk`, `lscpu`, `free`, and `df` provide targeted system details.
- System Monitor and command-line tools such as `top` report process, processor, memory, and resource activity.
### Terminal and shell fundamentals
#### Terminal and shell roles
- A terminal application accepts input and displays output.
- A shell interprets commands, expands expressions, connects programs, and requests operating system services.
- Common Linux shells include Bash, Dash, Zsh, and Fish, but their syntax and startup files differ.
#### Paths and navigation
The current working directory provides the starting point for relative paths.

Common path symbols include:
- `/` for the filesystem root
- `~` for the current user's home directory in shell expansion
- `.` for the current directory
- `..` for the parent directory

Linux paths are case-sensitive on common native file systems, so `Report.txt` and `report.txt` can name different files.
#### Shell features
- Tab completion expands commands and paths, reducing typing and some errors.
- Command history recalls earlier commands, but administrators should review a recalled command before executing it in a different context.
- Pipes send one program's standard output to another program's standard input.
- Redirection sends input and output between commands, files, and devices. Incorrect output redirection can overwrite data.
- Quoting controls expansion and word splitting. Administrators should quote paths and variables when spaces or special characters may appear.
### Common Linux commands
#### Navigation and file operations
- `ls` lists directory entries.
- `cd` changes the current directory.
- `pwd` prints the current directory path.
- `cp` copies files or directories.
- `mv` moves or renames files and directories.
- `rm` removes directory entries and usually bypasses a desktop rubbish bin.
- `mkdir` creates directories.
#### Viewing and editing text
- `cat` writes file content to standard output and can concatenate files.
- `less` provides paged viewing for longer text.
- `head` and `tail` display the beginning or end of input.
- `nano`, `vim`, and other editors modify text from a terminal.
- `grep` searches text, while `find` searches directory trees by criteria such as name, type, owner, and time.
#### Permissions, ownership, and packages
- `chmod` changes permission bits and, where supported, special mode bits.
- `chown` changes ownership and can also change group ownership.
- `sudo` runs an authorised command under another security context, usually as root.
- `apt` and `apt-get` manage packages on Debian-based distributions. `apt update` refreshes package metadata, while `apt upgrade` installs available upgrades within its dependency rules.
#### Monitoring, networking, and transfer
- `ps` reports processes, while `top` provides an updating view of activity.
- `df` reports filesystem space, and `du` estimates space used by files and directories.
- `ip` displays and configures network interfaces, addresses, and routes. `ifconfig` is legacy on many current systems.
- `ping` tests reachability through Internet Control Message Protocol when networks permit it. A failed response does not always prove that a host is offline.
- `hostname` reports or changes the system name, depending on its arguments and privileges.
- `curl` transfers data using supported protocols, while `wget` retrieves network resources non-interactively.
- `rsync` synchronises files efficiently, but its deletion and path options require careful review.
#### Archiving and scheduling
- `tar` creates or extracts archives and can work with compression tools such as gzip, bzip2, and xz.
- `cron` schedules recurring commands from a crontab, subject to the cron implementation and environment.
- `systemd` timers provide another scheduling mechanism on systemd-based distributions and integrate with service management and logging.
### Linux filesystem layout
#### File types and the root tree
- In long `ls` output, the first character identifies the entry type. A hyphen identifies a regular file, while `d` identifies a directory.
- Linux places mounted file systems within one directory tree rooted at `/`, rather than assigning a drive letter to each volume.
- Mount points connect a device, remote share, or virtual file system to a directory in that tree.
#### Important directories
- `/bin` contains essential user commands under the Filesystem Hierarchy Standard, although many current distributions link it to `/usr/bin` through a merged `/usr` layout.
- `/sbin` contains essential system-administration commands, although many current distributions link it to `/usr/sbin`.
- `/etc` stores host-specific system configuration.
- `/var` stores variable data such as logs, caches, queues, and application state.
- `/tmp` stores temporary files. Retention and cleanup behaviour depend on the distribution and configuration.
- `/home` commonly stores user home directories.
- `/root` is the root account's home directory.
- `/boot` stores boot-loader files, kernels, and related boot data.
- `/run` stores volatile runtime data created since boot.
- `/proc` and `/sys` expose process, kernel, device, and system information through virtual file systems.
### Runlevels and systemd targets
- Traditional System V init uses numbered runlevels. Common conventions assign `0` to halt and `6` to reboot, but distributions can define other runlevels differently.
- Historical Linux configurations often associated runlevel `3` with multi-user text operation and runlevel `5` with a graphical login.
- Most major current distributions use systemd targets instead of native System V runlevels.
- `multi-user.target`, `graphical.target`, `rescue.target`, and `emergency.target` express common system states.
- `systemctl get-default` displays the default target, while `systemctl set-default` changes it with suitable privileges.
### Linux security controls
- Timely operating system and application updates reduce exposure to known vulnerabilities. Debian-based systems commonly use `apt update` followed by an appropriate upgrade command.
- Administrators should remove or disable unused packages, accounts, services, and listening ports to reduce the attack surface.
- A host firewall should permit required traffic and reject unauthorised connections. Ubuntu commonly manages netfilter rules through Uncomplicated Firewall or other supported tools.
- Backups should use separate failure domains, protected credentials, encryption where required, retention controls, and routine restoration tests.
- Secure remote administration commonly disables direct SSH root login, restricts source networks, uses strong cryptographic keys, and adds multi-factor authentication where the environment supports it.
- Administrators should validate SSH configuration before restarting a remote service to avoid accidental lockout.
- Logs may reside under `/var/log` or the systemd journal. Central collection and alerts help detect suspicious activity and preserve evidence.
- Mandatory access-control systems such as AppArmor or SELinux limit processes beyond traditional user and group permissions when administrators configure and maintain them correctly.
## macOS, mobile platforms, and virtualisation
### macOS productivity and security
#### Workflow and navigation
- Spaces and multiple desktops separate activities into workspaces and reduce window clutter.
- Mission Control displays open windows and desktops to support navigation and window management.
- Spotlight searches files, applications, settings, and other indexed content. Current macOS versions can also expose actions and calculations through search.
- The Dock provides configurable access to applications, files, folders, and minimised windows.
#### Credentials and data protection
- Keychain stores passwords, passkeys, certificates, cryptographic keys, and other secrets for authorised applications and services.
- iCloud can synchronise supported files, settings, passwords, photos, and application data across devices signed in to the same Apple Account.
- FileVault encrypts the Mac startup volume and requires authorised credentials or a valid recovery method to unlock protected data.
- Organisations should escrow FileVault recovery keys through an approved management service and restrict access to recovery records.
#### Storage and advanced tools
- Disk Utility formats and partitions storage, creates disk images, and runs supported file-system checks and repairs.
- Terminal provides command-line access to the Unix-based operating system for administration, automation, development, and diagnosis.
- Current Mac computers use an external optical drive when they need to read or write supported CDs or DVDs. Older Remote Disc guidance does not describe the standard current workflow.
- Activity Monitor reports process, processor, memory, energy, disk, and network activity.
### Maintaining macOS
#### Operating system and application updates
- A current, tested backup reduces the impact of a failed upgrade, incompatible software, corruption, or accidental deletion.
- System Settings provides macOS updates under General and Software Update on current releases. The App Store updates applications obtained through the store.
- Applications, security tools, drivers, and management agents may also require updates after a macOS upgrade.
- Automatic updates reduce delay for many devices, while organisations may stage major upgrades to test compatibility and manage disruption.
- Administrators should confirm hardware support, storage capacity, recovery options, and critical application compatibility before a major upgrade.
#### System Settings
- Apple renamed System Preferences to System Settings in macOS Ventura 13 and introduced a sidebar-based layout.
- Current releases place many operating system controls under General, Privacy and Security, Desktop and Dock, and other categories.
- Apple can reorganise controls between releases, so administrators should document settings by purpose and supported management payload rather than relying only on an old screen path.
- Search helps locate settings when their position changes.
### iOS setup and administration
#### Organisational use
- Organisations adopt iPhone and iPad devices for their security architecture, privacy controls, accessibility, application ecosystem, device-management capabilities, and integration with other Apple platforms.
- Organisational suitability depends on supported hardware, update eligibility, management controls, application requirements, data handling, and lifecycle cost.
#### Setup and navigation
- Setup Assistant guides language and region selection, network connection, Apple Account sign-in, passcode and biometric configuration, device transfer, privacy choices, and supported management enrolment.
- Core navigation areas include the Lock Screen, Home Screen, Today View, App Library, Control Centre, and Notification Centre.
- Organisations can use automated device enrolment and a mobile device management service to supervise eligible corporate devices and apply policy before routine use.
#### Settings, identifiers, and updates
- Settings controls Wi-Fi, mobile service, notifications, privacy, security, accessibility, applications, and device management.
- The About screen reports details such as model, serial number, storage, network addresses, and cellular identifiers when the hardware provides them.
- Automatic update settings can download and install eligible iOS updates, security responses, and system files.
- Administrators should still monitor update compliance because power, storage, connectivity, deferrals, hardware support, and user action can delay installation.
### Apple identities and backup
#### Personal and managed accounts
- Apple now uses the term Apple Account for the identity previously called Apple ID.
- A personal Apple Account gives an individual control over supported Apple services, purchases, subscriptions, and iCloud data.
- A Managed Apple Account belongs to an organisation and provides controlled access to selected Apple services. Its available services differ from those of a personal account.
- Managed Apple Accounts can support account-driven enrolment and organisational identity workflows, but the device-management service applies configuration profiles, restrictions, and application assignments.
- Administrators should separate personal and organisational data according to device ownership, enrolment method, policy, and legal requirements.
#### Time Machine
- Time Machine backs up applications, documents, photos, and other files that do not form part of the macOS installation.
- Time Machine can create automatic hourly, daily, and weekly backups and can retain local snapshots on supported Apple File System volumes.
- A backup destination outside the internal disk provides protection when the Mac or its internal storage fails.
- Organisations should encrypt backup media where required, control retention, monitor completion, and test restoration.
### Android setup and administration
#### Organisational use
- Android supports hardware from many manufacturers and offers enterprise controls for work profiles, corporate-owned devices, dedicated devices, and managed applications.
- Device suitability varies by vendor support period, update delivery, hardware security, management compatibility, application requirements, and regional availability.
- Fleet standards should specify approved models and minimum patch levels instead of relying on the Android brand alone.
#### Setup and navigation
- Setup commonly covers Wi-Fi, Google Account sign-in, date and time, screen lock, privacy choices, application restoration, and optional enterprise enrolment.
- The Settings application manages network, battery, storage, accessibility, security, privacy, accounts, and application-specific controls, although menu names vary by Android version and manufacturer.
- Android Enterprise can separate work data in a work profile or apply stronger control to a fully managed corporate device.
#### Device information and updates
- About phone and Android version screens report the device model, Android version, security update level, Google Play system update, and build number. Available fields vary by device.
- The Android Security Bulletin publishes platform fixes regularly, while device and chipset manufacturers may publish additional bulletins.
- Update availability and installation timing depend on the device model, manufacturer, mobile operator, region, support period, and organisational policy.
- Administrators should monitor both the Android security patch level and Google Play system update status where supported.
### Virtualisation foundations
#### Definition and benefits
- Virtualisation abstracts physical processors, memory, storage, and networking so several isolated virtual machines can share a host.
- Each virtual machine runs a guest operating system and virtual hardware presented by a hypervisor.
- Virtualisation can improve utilisation, provisioning speed, workload mobility, testing, recovery, and isolation.
- Consolidation also concentrates risk. Host failure, management-plane compromise, capacity exhaustion, and shared storage faults can affect many guests.
#### Hypervisor types
- A type 1 hypervisor runs directly on hardware or as part of a purpose-built host platform and commonly supports data-centre workloads.
- A type 2 hypervisor runs as an application on a general-purpose host operating system and commonly supports development, testing, and desktop use.
- Product architectures can blur this simple classification, so security and operations teams should assess the actual trust boundaries and management components.
#### Virtualisation categories
- Desktop virtualisation includes centrally hosted virtual desktops and local desktop virtual machines.
- Network virtualisation includes software-defined networking and network-functions virtualisation.
- Storage virtualisation pools or abstracts storage resources for central management and allocation.
- Application virtualisation isolates or streams an application without presenting a complete virtual desktop to the user.
- Data virtualisation presents a unified access layer across multiple data sources without necessarily copying every source into one store.
### Cloud computing
#### Characteristics and benefits
- Cloud computing provides on-demand network access to a shared pool of configurable resources that can be provisioned and released rapidly.
- Core characteristics include on-demand self-service, broad network access, resource pooling, rapid elasticity, and measured service.
- Cloud services depend on data centres, compute, storage, networking, identity, management, automation, and often virtualisation or containers.
- Potential benefits include faster provisioning, elastic capacity, global reach, managed services, and reduced ownership of some physical infrastructure.
- Cloud adoption does not remove customer responsibility. Identity, data, configuration, applications, resilience, and governance remain shared responsibilities that vary by service.
#### Service models
| Model | Provider supplies | Customer focus |
| --- | --- | --- |
| IaaS | Virtualised compute, storage, and networking | Guest operating systems, applications, identities, data, and configuration |
| PaaS | Managed infrastructure, operating system, and application platform | Application code, identities, data, and service configuration |
| SaaS | Complete provider-operated application | Accounts, access policy, data, configuration, and authorised use |
| Serverless computing | Managed execution environment triggered by events or requests | Code, identities, data, dependencies, limits, and service configuration |

Serverless computing is an execution and operating model rather than one of the three original NIST service models.
#### Deployment models
| Deployment | Core idea | Common drivers |
| --- | --- | --- |
| Public cloud | A provider offers pooled services to many customers with logical separation | Elasticity, reach, service breadth, and consumption-based pricing |
| Private cloud | One organisation receives exclusive use of a cloud environment | Control, integration, locality, and regulatory requirements |
| Hybrid cloud | Distinct cloud environments support data and application portability | Phased migration, workload placement, resilience, and integration |
| Community cloud | Organisations with shared requirements use a common environment | Sector-specific policy, mission, security, or compliance needs |
### Containers and Docker
#### Containers and virtual machines
- A container packages an application with its required files and runs it as an isolated process through operating system features.
- Containers on one host normally share the host kernel, while virtual machines run separate guest kernels on virtual hardware.
- Containers often start faster and use fewer resources than full virtual machines, but start time and density depend on the workload, image, storage, runtime, and host.
- Virtual machines usually provide a stronger default isolation boundary because the hypervisor separates guest kernels. Container security still depends on kernel security, runtime configuration, privileges, images, and host protection.

| Feature | Containers | Virtual machines |
| --- | --- | --- |
| Operating system | Share the host kernel | Run a separate guest kernel |
| Start-up | Usually faster | Usually slower because the guest operating system boots |
| Overhead | Usually lower | Usually higher |
| Isolation | Process and namespace boundary | Virtual hardware and guest-kernel boundary |
| Common use | Portable applications, services, and build pipelines | Full operating systems, legacy workloads, and stronger workload separation |
#### Benefits and challenges
- Containers support portability, repeatable deployment, higher workload density, rapid scaling, and continuous integration and delivery workflows.
- Container images can also carry vulnerable libraries, embedded secrets, excessive privileges, or untrusted content.
- Secure operations require trusted registries, signed or verified artefacts, minimal images, vulnerability management, least privilege, runtime controls, secrets management, network policy, logging, and protected orchestration systems.
- At scale, organisations must also manage image provenance, resource limits, service discovery, persistent data, upgrades, observability, and incident response.
#### Tools and platforms
- Docker provides tools for building, distributing, and running container images and containers.
- On Linux, Docker uses kernel features such as namespaces and control groups for isolation and resource control.
- Podman is a daemonless, Open Container Initiative-compatible container engine that can run containers as root or as an unprivileged user.
- LXC provides tools and libraries for Linux system and application containers that share the host kernel.
- Vagrant manages reproducible virtual machine environments. It can work alongside container workflows, but it is not a container engine.
### Key comparisons
| Topic | Operational takeaway |
| --- | --- |
| Settings and usability | Strong defaults help, but supported updates, tested backups, and controlled access provide broader protection |
| Mobile device management | Fleet control depends on approved hardware, enrolment, policy, application governance, and update compliance |
| Virtualisation and cloud | Virtualisation abstracts resources, while cloud adds on-demand service delivery, elasticity, and measured use |
| Containers | Containers accelerate delivery and portability, while secure operation requires image governance, host protection, least privilege, and monitoring |
