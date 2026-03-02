# Deploying, Configuring and Maintaining Systems
## Managing Software Packages
### DNF and YUM Relationship
RHEL 8 standardises on DNF 3 as the package management engine. The yum and dnf commands are symbolic links to the same backend executable. Administrators may continue using yum for familiarity, but both commands provide identical functionality.

The system relies primarily on two repositories:
- BaseOS provides core operating system packages in RPM format
- AppStream supplies additional software delivered as RPMs and modular content

Traditional RPM packages install a single version of software with separate dependency handling. AppStream modules extend this model by grouping applications and dependencies into versioned streams. Profiles within a module allow selective installation of components such as client only or full server stacks.
### Verifying Subscriptions and Repositories
Before installing software, administrators must confirm subscription status using subscription-manager. A valid subscription enables access to Red Hat repositories. Repository availability can be checked with yum repolist or dnf repolist.

The course environment demonstrates that symbolic links connect yum and dnf to dnf-3, confirming that command choice does not affect functionality.
### Core Package Management Tasks
DNF provides consistent subcommands for common operations:
- list to query available or installed packages
- install to add software
- remove or purge to uninstall packages
- update to apply upgrades
- history to review and undo transactions

Standard users can query repositories but require elevated privileges to install or remove packages. The provides subcommand identifies which package supplies a specific file or command.

A key behaviour is dependency management. Installing software may pull additional packages, and simple removal does not always remove dependencies. Transaction history allows administrators to undo entire operations safely.
### Practical Package Operations
Typical workflows include:
- searching for a package such as tree
- installing software with yum install
- automating confirmation with the -y option
- removing packages and reversing actions through yum history undo

Complex packages like httpd demonstrate how multiple dependencies install automatically. Transaction rollback provides a reliable recovery method when changes cause issues.
## Understanding Package Repositories
### Repository Structure
Repository definitions reside in /etc/yum.repos.d as .repo files. The primary configuration redhat.repo contains both enabled and disabled repositories tied to the subscription.

Administrators can display all repositories, including disabled ones, with yum repolist --all. Understanding repository state is essential when troubleshooting package availability.
### Creating Local File Repositories
Local repositories allow systems to install software without internet access. The process involves:
- attaching the RHEL installation ISO
- mounting it under /mnt
- confirming the presence of BaseOS and AppStream directories
- installing dnf-plugins-core if required
- adding repositories with yum config-manager --add-repo

This method provides package access but not ongoing updates because the DVD content is static. It is useful for isolated environments or testing.

Repository files created by config-manager appear automatically in the yum.repos.d directory. Administrators can enable or disable repositories to control package sources and verify behaviour by installing packages such as zsh from the local media.
### Building Web Based Repositories
For multi system environments, repositories can be published through an HTTP server. The process includes:
- installing and starting the httpd service
- mounting the installation media under /var/www/html
- adding repository definitions that reference the web server
- validating access with yum makecache

This approach reduces external bandwidth use and centralises package distribution. However, like the local DVD method, it does not provide vendor updates unless synchronised with official mirrors.
## Working with Security Updates
RHEL 8 enhances update management through metadata tagging. Updates are categorised as security fixes, bug fixes or enhancements. This capability is not available in CentOS in the same form.

Administrators can inspect update information using yum updateinfo. Filters allow focus on security advisories or specific severity levels such as Important. The system also exposes Red Hat Security Advisory identifiers, enabling highly targeted patching.

Typical security workflows include:
- listing all updates and categories
- filtering for security notices only
- restricting to high severity updates
- applying a single advisory with yum update --advisory
- performing severity based updates

Kernel updates install new versions rather than replacing the existing kernel, preserving rollback capability. Transaction history again provides a safety mechanism to undo problematic updates.
## Working with Modules
### Concept of Module Streams
Modules introduce flexible version control within AppStream. A stream represents a supported version of an application stack. Multiple streams can coexist in the repository, allowing administrators to select versions that match application requirements.

For example, the Ruby module may offer several streams such as 2.5, 2.6 and 2.7. The default stream installs automatically unless another version is specified.
### Module Profiles
Profiles define which components of a module are installed. Common profiles include:
- server
- client
- minimal
- common

This granularity avoids installing unnecessary packages. For instance, MariaDB can install only the client tools without the full database server.
### Module Management Commands
Key commands include:
- yum module list to display available modules
- yum module install to deploy a chosen stream and profile
- specifying streams with module:version syntax
- specifying profiles with module/profile syntax

Modules provide controlled software lifecycles and simplify managing applications that require specific versions.
## Configuring Time Services
### Chrony and NTP
RHEL 8 replaces the legacy ntpd daemon with chronyd as the default time synchronisation service. Both use the Network Time Protocol, but chrony offers faster convergence and better performance on systems that sleep or experience high CPU load.

The chrony configuration file is similar to the traditional ntp.conf, easing migration. Administrators should confirm the chrony package is installed and the chronyd service is active and enabled.
### Using timedatectl
timedatectl provides a central interface for time configuration. It displays:
- local time
- universal time
- time zone
- NTP synchronisation status

Best practice is to keep the hardware clock in UTC and allow the system time zone offset to provide local time. This avoids daylight saving complications.

Time zones can be changed with timedatectl set-timezone. If the automatic link update fails, administrators can manually link the appropriate zoneinfo file to /etc/localtime.
## Managing Services with systemctl
systemctl controls services managed by systemd. Important operations include:
- status to view service state
- start and stop to control runtime behaviour
- enable and disable to manage boot behaviour
- enable --now to start and persist a service in one command
- restart after configuration changes

The tool can also display unit files with systemctl cat and create editable copies with systemctl edit --full. Once modified, unit files move from the vendor directory to /etc/systemd/system, overriding defaults.

Chronyd conflicts with other time synchronisation services such as ntpd and systemd-timesyncd, so only one should run at a time.
## Editing chrony Configuration with sed
### Configuration Location
The chrony configuration resides in /etc/chrony.conf. The file includes extensive comments that can obscure operational settings. Because full documentation exists in the manual pages, administrators may streamline the configuration for clarity.
### Using sed for Consistent Edits
The stream editor sed enables repeatable, automated configuration changes. A scripted approach ensures consistent results across multiple systems.

Typical automation tasks include:
- removing comment and blank lines
- inserting geographically appropriate NTP pools
- deleting default pool entries
- performing in place edits with the -i option

Using a separate sed script file simplifies command syntax and supports scalable configuration management. After modifying the configuration, the chronyd service must be restarted to apply changes.
## Chronyc Monitoring Tools
Chrony includes the chronyc command line client for monitoring synchronisation status and performance. The daemon is designed for rapid convergence and resilience under load, making it suitable for both servers and mobile systems.
## Summary
The course demonstrates practical administration techniques for RHEL 8. Core skills include managing packages with DNF, controlling repositories, applying security updates, using modular software streams and maintaining accurate system time with chrony. systemctl provides unified service management, while sed enables automated configuration changes.

Together, these capabilities allow administrators to deploy, secure and maintain RHEL 8 systems efficiently in both standalone and enterprise environments.