# Operating Systems: Overview, Administration, and Security

> [!NOTE]
> This document surveys operating system fundamentals and secure administration across Windows, Linux, and Apple platforms. It explains OS roles, storage and filesystems, Windows architecture, and essential management tools (Microsoft Management Console, command line, Group Policy, WSUS). Security coverage includes UAC, encryption (BitLocker/EFS, FileVault), endpoint protection, firewalling, patching, auditing, and directory-based authentication (Active Directory, Kerberos). It then introduces Linux distributions, shell navigation, core commands, permissions, and hardening. Finally, it reviews mobile OS basics plus virtualization, cloud service models, and containers such as Docker.
## Windows Operating Systems
### Operating system fundamentals
#### What an operating system provides
- An operating system coordinates input, output, processing, storage, and access to hardware resources while supporting user interfaces and application execution.
- Interfaces commonly include a command line environment and a graphical user interface.
- Operating system evolution can be a progression from early batch processing, through time-sharing and networked systems, to modern multitasking desktop and mobile platforms.
- Linux is widely used for servers and embedded systems and supports many distributions.
- Windows supports consumer and enterprise environments and includes separate server editions for network and infrastructure roles.
- macOS is a Unix-based platform for Apple devices.
- ChromeOS is a lightweight operating system built around the Chrome browser and is commonly used in managed education and fleet environments.
### Windows platforms and architecture
#### Windows 10 and Windows 11 editions
- Windows editions differ in management and security features, so organisations align edition choice to device role and governance needs.
- Home editions focus on consumer use and include baseline security features.
- Pro editions add business features such as domain join, BitLocker support on many configurations, and remote management options.
- Enterprise editions add advanced security and management capabilities designed for large environments, including stronger deployment and policy control.
- Pro for Workstations targets high performance workloads and supports hardware and storage features suited to specialised workstations.
#### User mode and kernel mode
- Windows separates execution into user mode and kernel mode to improve stability and limit damage from application faults.
- User mode processes run with restricted privileges and isolated address spaces, so a crash is more likely to impact only the affected application.
- Kernel mode components, including most drivers, share a broader execution context and can interact directly with hardware.
- Kernel faults can destabilise the operating system, so drivers and privileged code require stronger testing and control.
- User mode requests that require hardware access are mediated through system calls, which provide controlled entry into kernel services.
### Storage, files, and layout
#### File systems and their purpose
- A file system organises how files are stored, named, and retrieved on storage devices and maintains metadata such as size, timestamps, and permissions.
- Hard drives use magnetic storage. Solid state drives use flash memory and are typically faster for access patterns seen in modern systems.
- FAT variants remain common for removable media due to broad compatibility.
- FAT32 has practical limits, including a typical maximum file size of 4 GB, and volume limits that depend on formatting tools and partition style.
- NTFS is the standard Windows file system and supports access control, journaling for recovery, quotas, encryption features, hard links, and a Master File Table for efficient tracking of files and directories.
#### Windows directory structure and 32-bit and 64-bit behaviour
- Windows commonly uses a hierarchical structure rooted at the system drive, often C.
- C:\Users stores per-user profiles and subfolders such as Documents, Downloads, Desktop, and Pictures.
- C:\Windows stores core operating system files and settings.
- Program Files stores 64-bit applications on 64-bit Windows, and Program Files (x86) stores 32-bit applications.
- On 64-bit Windows, System32 contains 64-bit binaries, and SysWOW64 contains 32-bit binaries used by the WoW64 compatibility layer.
- Windows keeps some system files hidden to reduce accidental damage, including pagefile.sys for virtual memory, hiberfil.sys for hibernation, and PerfLogs for performance logging.
- 32-bit systems have a smaller address space and practical memory limits. 64-bit systems support far larger memory ranges and are the modern baseline for most environments.
#### File naming conventions
- Clear naming improves collaboration, searching, and recovery during incident response and routine operations.
- Names are most usable when they are short, descriptive, and consistent across a project.
- Spaces and special characters can break scripts and tooling, so many organisations prefer hyphens and avoid punctuation that has system meaning.
- ISO date format supports predictable sorting, and version indicators distinguish drafts and releases.
- Confusable characters, such as lower case L and the numeral one, are avoided in identifiers where mistakes create operational risk.
### Windows administration foundations
#### Microsoft Management Console
- Microsoft Management Console is a framework for running administrative snap-ins through a consistent Windows interface.
- Administrators can build a console that matches the organisation’s workflow by adding, arranging, and saving snap-ins.
- Centralising tools reduces context switching and supports safer delegation when access is limited to specific snap-ins.
- Common built-in snap-ins include Computer Management, Event Viewer, Device Manager, Disk Management, Local Users and Groups, Services, Task Scheduler, Performance Monitor, Group Policy Management, and Certificates.
#### Command line administration in Windows
- The Command Prompt provides a text interface for running administrative commands, troubleshooting, and automating routine tasks.
- Windows commonly uses two privilege levels for local work.
- Standard accounts support everyday activity with reduced risk.
- Administrator accounts support system-wide changes and should be used only when required, with User Account Control left enabled to prompt for elevation.
- Group Policy troubleshooting commonly uses gpupdate to refresh policy and gpresult to report applied settings.
- Drive and directory navigation commonly uses vol to show volume details, cd or chdir to change directories, and drive letters to switch context.
- System maintenance commands include shutdown, sfc for integrity checks, chkdsk for disk checks, diskpart for partition management, winver for version details, and format for reinitialising storage.
- File and directory management commonly uses dir, md, rd, ren, and del, with caution around deletion.
- Copy and backup tooling includes copy, xcopy, and robocopy, which can mirror large directory structures when configured carefully.
- Network diagnostics commonly start with hostname, ipconfig, and ping.
### Identity and directory services
#### User account management in Windows Server Essentials
- User accounts control access to shared folders, remote access features, and other network resources.
- Windows Server Essentials typically distinguishes between standard accounts and administrator accounts.
- Administrative roles can add, modify, copy, deactivate, or remove user accounts, with deactivation preferred when access needs to be suspended temporarily.
- The user creation process usually sets a logon name and password, assigns an access level, sets group membership, configures access to shared folders, and enables or blocks remote access.
- Where accounts are linked to cloud services, administrators need to understand what is deactivated or removed in both local and online systems.
- Dashboards commonly show a user list with access level, remote access status, account status, and related management indicators that support routine administration.
#### Active Directory overview
- Active Directory is a Microsoft directory service that stores identities and resources and supports centralised authentication and access control.
- Organisational Units group objects to support delegation and targeted policy application.
- Security Groups simplify access management by applying permissions to groups rather than individual accounts.
- Home folders and folder redirection support consistent user storage and reduce data loss risk when devices fail, provided backups are in place.
- Group Policy Objects enforce configuration at scale, including security settings, software deployment, and password policy.
- The hierarchy includes domains as administrative boundaries, domain trees as related domains, and forests as collections that share a schema and trust relationships.
## Windows Security
### Windows security settings and account controls
#### User Account Control
- User Account Control reduces the chance of unauthorised system changes by prompting before elevated actions.
- A consent prompt asks an already signed-in administrator to approve an action that needs elevation.
- A credential prompt asks a standard user to provide administrator credentials before the action proceeds.
#### Drive and file encryption
- BitLocker encrypts internal drives, including the operating system volume and fixed data volumes, so data is unreadable without the decryption key.
- BitLocker can use a Trusted Platform Module, a PIN, a password, or a startup key, and recovery keys should be backed up securely.
- BitLocker To Go applies similar encryption to removable media such as USB drives and external disks.
- Encrypting File System encrypts selected files and folders on NTFS volumes and decrypts them transparently for authorised users with the correct keys.
#### Users, groups, and least privilege
- User accounts identify individuals and control access to local and network resources.
- Groups simplify permission management by assigning access to a group rather than to each user.
- Least privilege limits accounts to the minimum access needed for the task and reduces damage from mistakes and malware.
- Day-to-day work is safer under a standard account, with administrator accounts used only when required.
#### Windows account types
- Local accounts exist only on a single device.
- Microsoft accounts link a user to Microsoft cloud services and can synchronise settings across devices.
- Domain accounts are managed by a domain controller and provide centralised authentication, policy enforcement, and access to shared resources.
#### NTFS permissions and share permissions
- NTFS permissions apply to files and folders stored on NTFS volumes and can be set at a fine-grained level.
- Share permissions apply only when accessing a shared folder over the network and are less granular.
- Effective access is governed by the combination of NTFS and share permissions, so both need to be designed and reviewed.
### Endpoint protection, firewalling, and user awareness
#### Microsoft Defender Antivirus and antimalware practice
- Microsoft Defender Antivirus provides built-in, real-time protection in Windows and supports scheduled, quick, full, custom, and offline scans.
- Threat intelligence and signatures must be kept current to improve detection of new malware.
- Multiple real-time antivirus products on the same endpoint can conflict and reduce stability, so organisations typically choose one primary endpoint protection platform.
#### Windows Firewall fundamentals
- A software firewall controls inbound and outbound connections based on rules.
- Rules can be tuned with allowed apps, ports, protocols, and addresses, and alerts can notify when an application is blocked.
- Exceptions should be limited to what is necessary to reduce exposure.
#### Social engineering and phishing
- Social engineering targets human decision-making through techniques such as phishing emails, vishing calls, smishing messages, and lookalike websites.
- Technical controls reduce risk, but awareness and verification procedures remain critical because phishing can bypass malware defences.
- Ongoing anti-phishing training is most effective when paired with simulated phishing tests and clear reporting pathways.
### Patching and update management
#### Patches, updates, and why timing matters
- A patch fixes specific defects or vulnerabilities after release.
- An update may include patches and may also introduce new features or performance improvements.
- Security patches for critical systems are applied promptly to close known vulnerabilities, while less urgent feature updates can be scheduled to reduce disruption.
#### Patch management as a process
- Patch management includes identifying relevant updates, acquiring them from trusted sources, testing, deploying, and verifying results.
- A workable policy defines priorities, testing requirements, maintenance windows, rollback plans, and reporting.
- Automation reduces human error and helps maintain consistent coverage across many endpoints.
#### Microsoft Update and WSUS
- Microsoft Update delivers updates for Windows and other Microsoft products and can include drivers and security fixes.
- Automatic Updates suits many endpoints because it downloads and installs updates on a schedule or with user prompts.
- Administrator-managed updates provide tighter control, including staged rollout across many devices.
- Windows Server Update Services supports centralised approval, testing, and distribution of updates inside an organisation.
### Kerberos, directory services, and authentication methods
#### Kerberos in plain terms
- Kerberos is a network authentication protocol that uses time-limited tickets and shared-secret cryptography to authenticate clients and services.
- The Key Distribution Centre issues tickets after the user authenticates, typically backed by Active Directory in Windows domains.
- Kerberos supports single sign-on by allowing an authenticated user to access approved services without re-entering credentials during the ticket lifetime.
- Kerberos provides mutual authentication, so both client and service can verify each other.
#### Active Directory purpose and privileged roles
- Active Directory is the central identity and resource directory for many Windows enterprise networks.
- A domain groups users, devices, and policies into an administrative boundary managed by domain controllers.
- Domain Admin accounts have broad control and require strong protection, limited use, and careful monitoring.
- Common default accounts include Administrator, Guest, KRBTGT for Kerberos ticketing, and HelpAssistant for remote assistance scenarios.
#### Other authentication methods
- RADIUS supports centralised authentication, authorisation, and accounting and is common for VPN and Wi-Fi access control.
- TACACS and TACACS+ are common in network device administration because they can separate authentication, authorisation, and accounting and support granular command authorisation.
- EAP is a framework used in network access authentication, including EAP-TLS in enterprise Wi-Fi.
### Security auditing in Windows
#### What auditing is for
- Security auditing records selected events so administrators can detect misuse, investigate incidents, and demonstrate accountability.
- Audit design starts by identifying critical assets, deciding which actions need monitoring, enabling the right policies, and protecting logs from tampering.
#### Common audit policy categories
- Account logon and logon events.
- Account management events such as password resets and group membership changes.
- Directory service access events in Active Directory environments.
- Object access events for sensitive files, folders, and registry keys.
- Policy change, privilege use, process tracking, and system events such as startup and shutdown.
### Consolidated operational takeaways
- Security improves when identity controls, encryption, endpoint protection, patching, and auditing reinforce each other.
- Least privilege and verified workflows reduce the impact of phishing, malware, and administrative mistakes.
- Centralised update and identity services simplify governance, but they also become high-value targets, so they need strong hardening and monitoring.
## Linux Operating Systems
### Linux operating system essentials
#### Why organisations adopt Linux
- Linux is widely used for servers and developer workloads because it is stable, efficient, and flexible across many hardware platforms.
- Open source licensing allows communities and vendors to inspect, modify, and distribute code, which supports rapid improvement and transparent review.
- Organisations often benefit from broad vendor support and large software repositories delivered through package managers.
#### Distributions and interfaces
- Linux versions are called distributions. Common examples include Debian, Ubuntu, Fedora, Red Hat Enterprise Linux, openSUSE, and Linux Mint.
- Linux can be operated through a graphical user interface or through a command line interface.
- The command line interface supports automation and low resource usage, while the graphical user interface can be easier for new users.
### Accounts and system information in Ubuntu
#### Local accounts and administrative access
- Administrators commonly create standard user accounts for everyday use and reserve administrative access for tasks that require elevated privileges.
- In Ubuntu, elevated actions are typically performed with sudo rather than by logging in permanently as the root user.
- Account settings are reviewed after creation to confirm group membership and access scope.
#### Viewing device and operating system details
- System information such as hardware, storage, OS version, and update status is available through the system Settings About panel.
- Performance and resource use can be reviewed with a system monitor tool that reports CPU, memory, disk, and process activity.
### Linux terminal and shell fundamentals
#### Terminal and shell roles
- A terminal is the application used to enter commands and view output.
- A shell interprets those commands and coordinates with the operating system and kernel to run programs and return results.
#### Paths and navigation concepts
The current working directory is the location the shell uses when resolving relative paths.

Common path symbols include:
- / for the filesystem root
- ~ for the current user home directory
-. for the current directory
-.. for the parent directory
#### Shell features that improve speed and accuracy
- Tab completion expands file and directory names to reduce typing errors.
- Command history allows reuse of earlier commands using the arrow keys.
- Pipes and redirection support automation by chaining commands and sending output to files.
### Common Linux commands
#### Navigation and file operations
- ls lists directory contents.
- cd changes directory.
- pwd prints the current directory path.
- cp copies files or directories.
- mv moves or renames files or directories.
- rm removes files or directories and should be used with care.
- mkdir creates a directory.
#### Viewing and editing text
- cat prints file contents.
- head and tail show the first or last lines of a file.
- nano provides a lightweight terminal text editor.
#### Permissions and administration
- chmod changes file permissions.
- chown changes file ownership.
- sudo runs a command with elevated privileges when authorised.
- apt or apt-get installs, updates, and removes packages on Debian based systems.
#### Monitoring, networking, and transfer
- ps and top display running processes and resource usage.
- df reports filesystem space usage.
- ip displays and manages network configuration, with ifconfig considered legacy on many systems.
- ping tests reachability, while hostname prints the system name.
- curl displays content from a URL and wget downloads files from a URL.
- rsync supports efficient synchronisation between systems.
#### Archiving and scheduling
- tar creates archives and can compress them when configured for gzip compression.
- Cron schedules recurring jobs using a time and date pattern in a crontab.
### Linux filesystem layout
#### Files and directories
- In long listing output, a regular file is shown with a leading dash and a directory is shown with a leading d.
- Linux uses a single root tree that starts at /, unlike Windows drive letters.
#### Key directories and typical purposes
- /bin contains essential user commands.
- /sbin contains administrative commands.
- /etc stores system and application configuration.
- /var stores variable data such as logs.
- /tmp stores temporary files, often cleared on reboot.
- /home stores user home directories.
- /root is the home directory for the root user.
- /boot stores boot loader and kernel related files.
### Run levels and operational states
#### Traditional run levels and modern equivalents
- Legacy SysV run levels describe operating states from 0 for halt through 6 for reboot, with multi user modes commonly mapped to 3 and 5.
- Many modern distributions use systemd targets that correspond to similar states, managed with systemctl rather than init.
### Securing Linux systems
#### Essential controls
- Keeping the operating system updated reduces exposure to known vulnerabilities, commonly using apt update and apt upgrade on Debian based systems.
- Unused software, services, and open network ports are removed or disabled to reduce attack surface.
- A firewall is enabled and configured to block unauthorised access.
- Regular backups are created and stored securely in more than one location.
- Secure remote access practices include disabling SSH root login, using strong keys, and adding multi-factor authentication where feasible.
- Logs in locations such as /var/log are monitored to detect unusual activity early.
## macOS and Virtualisation
### macOS productivity and security features
#### Workflow and navigation
- Spaces and multiple desktops help separate tasks into distinct workspaces, reduce clutter, and support focus switching.
- Mission Control provides a single view of open windows and desktops to speed up navigation and window management.
- Spotlight supports fast search for files and apps, quick calculations, and basic lookups that reduce manual browsing.
#### Credential and data protection
- Keychain stores passwords and cryptographic items so users do not need to memorise multiple credentials.
- iCloud synchronises files and settings across Apple devices to support continuity, sharing, and real-time collaboration.
- FileVault encrypts the startup disk so data is unreadable without a user password or recovery key if the device is lost or stolen.
#### Storage and advanced tools
- Disk Utility supports formatting, partitioning, creating disk images, and basic disk repair checks.
- Terminal provides command line access to the Unix based system for scripting, automation, and advanced troubleshooting.
- Remote Disc can access an optical drive on another Mac over the network, which can help when a device has no physical drive.
- The Dock provides quick access to frequently used apps, files, and folders and can be customised to match work patterns.
### Maintaining macOS
#### Upgrades and application updates
- Before upgrading macOS, a full backup reduces the impact of upgrade failures and rollback scenarios.
- Updates are typically installed through the App Store or Software Update, with on screen prompts guiding restarts and progress.
- Applications also need updates after an OS upgrade to maintain compatibility and to receive security fixes.
- Automatic updates can reduce admin overhead, but organisations often stage rollouts to manage risk and disruption.
#### System settings evolution
- Apple renamed System Preferences to System Settings and redesigned it with a left sidebar list and a main detail panel.
- Some settings moved into new groupings, with several items now under General and others under Desktop and Dock.
- Toggle switches replaced many checkboxes to align the design with Apple mobile platforms.
- Search within settings was streamlined so results remain visible and easier to revisit.
### iOS device setup and essentials
#### Why organisations support iOS
- Organisations often value iOS for security features, privacy controls, managed device capabilities, and integration with Apple platforms and services.
#### Setup flow and navigation
- Common setup steps include language and region selection, Wi Fi connection, account sign in, device security setup, and optional data migration.
- Core screen areas include the Lock Screen, Home Screen, Today View, and App Library.
#### Settings and device information
- Settings is the central location for configuration, including Wi Fi, cellular plans, notifications, and accessibility.
- The About screen lists device identifiers and status details such as model, serial number, storage, network addresses, and cellular identifiers including IMEI and SIM identifiers where present.
- iOS can be configured to download and install updates automatically to maintain security patching cadence.
### Identity and backup concepts in Apple environments
#### Apple IDs and enterprise management
- A personal Apple ID supports individual services such as iCloud and the App Store with user controlled data.
- A managed Apple ID supports organisational control through mobile device management, enabling policy enforcement, app distribution, and configuration management.
#### Backups with Time Machine
- Time Machine is the built in macOS backup feature that can back up files, apps, and settings to an external destination.
- Regular backups support recovery from device loss, upgrade issues, and accidental deletion.
### Android device setup and essentials
#### Why organisations support Android
- Android supports devices from multiple vendors at many price points and offers enterprise management options through Google and device makers.
- Android has the largest global share of smartphone operating systems, which can simplify fleet sourcing.
#### Setup flow and navigation
- Setup commonly includes Wi Fi connection, Google account sign in, date and time, security configuration, and optional app migration.
- The Settings app provides access to network, battery, storage, accessibility, security, and app specific configuration.
#### Device information and updates
- About phone screens provide device identifiers such as model and IMEI, plus OS information such as build and kernel details used for troubleshooting.
- Android security updates are commonly delivered on a regular cadence, but timing varies by manufacturer and corporate management policy.
### Virtualisation foundations
#### Definition and benefits
- Virtualisation divides physical compute resources into multiple isolated virtual machines, improving utilisation, agility, and resilience.
- Virtual machines run as guests on a host and can be created and rebuilt faster than provisioning new physical servers.
#### Hypervisors and common types
- A hypervisor manages virtual machines and prevents interference between them.
- Type 1 hypervisors run directly on hardware and are common in data centres.
- Type 2 hypervisors run on top of an existing operating system and are common on endpoints and lab devices.
#### Virtualisation categories
- Desktop virtualisation includes centrally hosted virtual desktops and local desktop virtual machines.
- Network virtualisation includes software defined networking and network function virtualisation for virtualised appliances.
- Storage virtualisation pools storage into a managed resource for simpler provisioning and improved utilisation.
- Application virtualisation runs an app in an isolated layer without virtualising a full desktop.
- Data virtualisation provides unified access across multiple data sources to reduce data silos.
### Cloud computing essentials
#### Definition, components, and benefits
- Cloud computing provides computing resources over the internet, delivered by a provider or hosted internally, and scaled to demand.
- Core building blocks include data centres, networking, and virtualisation, often supported by load balancing and content delivery services.
- Benefits include reduced infrastructure overhead, rapid scaling, remote work support, and faster adoption of advanced services.
#### Service models and deployment models
| Model | What it provides | Typical responsibility split |
| --- | --- | --- |
| IaaS | compute, storage, and networking | customer manages OS and apps |
| PaaS | managed runtime and development platform | provider manages OS and platform |
| SaaS | complete application delivered as a service | provider manages app and platform |
| Serverless | event driven execution environment | provider manages infrastructure |

| Deployment | Core idea | Common drivers |
| --- | --- | --- |
| Public cloud | shared provider environment | elasticity and cost efficiency |
| Private cloud | single customer environment | control and compliance needs |
| Hybrid cloud | mix of public, private, and on premises | flexibility and phased migration |
### Containers and Docker
#### Containers compared with virtual machines
- Containers package an application and its dependencies while sharing the host kernel, which supports fast start up and high density.
- Virtual machines emulate full machines with separate operating systems, which increases isolation but also increases overhead.

| Feature | Containers | Virtual machines |
| --- | --- | --- |
| Start up | near instant | slower boot |
| Overhead | lower | higher |
| Isolation | process level | stronger hardware style isolation |
| Best fit | microservices and portability | full OS control and strong isolation |
#### Container benefits and challenges
- Benefits include portability across environments, improved utilisation, rapid deployment, and support for modern delivery practices such as CI and CD.
- Challenges include operational complexity at scale, right sizing, and the need to secure the host and supply chain.
#### Common tools and vendors
- Docker is a popular platform for building and running containers and uses Linux kernel features such as namespaces for isolation.
- Other common options include Podman, LXC, and Vagrant, each with different trade offs in architecture and workflow.
### Key comparisons and practical takeaways
| Topic | Reliable takeaway |
| ------------------------ | ------------------------------------------------------------------------------------------------------------- |
| Settings and usability | Strong defaults reduce risk, but consistent updates and backups matter more than interface changes |
| Mobile device management | Fleet control depends on clear enrolment, policy, and update processes rather than brand alone |
| Virtualisation and cloud | Virtualisation enables efficient resource sharing, while cloud adds on demand delivery and service models |
| Containers | Containers speed delivery and portability, but governance and monitoring are required to keep risk manageable |
