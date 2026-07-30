# *Operating Systems: Overview Administration and Security* Final Project
Cyber Secure Inc. recently hired you as a junior cybersecurity analyst. Other businesses contract Cyber Secure Inc. to handle their system administration and security.

Your supervisor assigned you six tickets; the first three require you to work with Windows OS, and the other three require you to work with Linux.
> [!NOTE]
> Command-line tools replaced GUI tools in this project, as command-line tools offer more stable interfaces and often expose more detailed OS or application APIs.
## Windows tasks
---
### Ticket 1: Create a new user
In this task, you will create a new user and group and then add the user to the group.
### Answer
1. Open the Start menu, search for `PowerShell`, right-click Windows PowerShell, and select `Run as administrator`.
2. Confirm that the LocalAccounts module and the required commands are available.
```powershell
Get-Command New-LocalUser, New-LocalGroup, Add-LocalGroupMember
```
3. Set variables for the account and group.
```powershell
$UserName = "Velma"
$GroupName = "Accounting"
```
4. Check whether the user or group already exists.
```powershell
Get-LocalUser -Name $UserName -ErrorAction SilentlyContinue
Get-LocalGroup -Name $GroupName -ErrorAction SilentlyContinue
```
5. Prompt for a password.
```powershell
$Password = Read-Host "Enter a strong, unique password for $UserName" -AsSecureString
```
6. Create local user.
```powershell
New-LocalUser -Name $UserName -Password $Password -FullName "Velma" -Description "Accounting user"
```
7. Create Accounting group.
```powershell
New-LocalGroup -Name $GroupName -Description "Accounting department"
```
8. Add the new user to the Accounting group.
```powershell
Add-LocalGroupMember -Group $GroupName -Member $UserName
```
#### Verification
Run both commands and confirm that the account is enabled and that `Velma` appears in the Accounting group.

```powershell
Get-LocalUser -Name $UserName | Select-Object Name, Enabled, PasswordRequired
Get-LocalGroupMember -Group $GroupName
```
---
### Ticket 2: Check for virus and threat protection updates
In this task, you will use the Windows operating system to check for virus and threat protection updates and run a "Quick Scan".
### Answer
1. Continue in an elevated PowerShell window.
2. Confirm that the Microsoft Defender Antivirus service exists and review its current state.
```powershell
Get-Service -Name WinDefend
Get-MpComputerStatus | Select-Object AntivirusEnabled, RealTimeProtectionEnabled, AntivirusSignatureVersion, AntivirusSignatureLastUpdated
```
3. Update Microsoft Defender security intelligence.
```powershell
Update-MpSignature
```
4. Run a Quick Scan.
```powershell
Start-MpScan -ScanType QuickScan
```
#### Verification
Confirm that the signature timestamp is current for the system and that the Quick Scan has start and end times.
```powershell
Get-MpComputerStatus | Select-Object AntivirusSignatureVersion, AntivirusSignatureLastUpdated, QuickScanStartTime, QuickScanEndTime
```
---
### Ticket 3: Configure firewall and network protection
Access Windows Defender Firewall with Advanced Security to create a new rule. The new rule should have the following properties:
1. A port rule that controls connections for a TCP port on port 80
2. Allow the connection on Domain, Private, and Public
3. Includes the rule name: "Port 80 Permitted"
### Answer
1. Continue in an elevated PowerShell window.
2. Create an enabled inbound rule that allows TCP port 80 on all three required profiles.
```powershell
New-NetFirewallRule -DisplayName "Port 80 Permitted" -Direction Inbound -Action Allow -Protocol TCP -LocalPort 80 -Profile Domain,Private,Public -Enabled True
```
#### Verification
The first command verifies the rule action, direction, profiles, and enabled state. The second command verifies the protocol and local port.
```powershell
Get-NetFirewallRule -DisplayName "Port 80 Permitted" | Format-List DisplayName, Enabled, Direction, Action, Profile
Get-NetFirewallRule -DisplayName "Port 80 Permitted" | Get-NetFirewallPortFilter | Format-List Protocol, LocalPort
```
If service validation is within the ticket's scope, use the following read-only check.
```powershell
Get-NetTCPConnection -State Listen -LocalPort 80 -ErrorAction SilentlyContinue
```
## Linux tasks
---
### Ticket 4: Create a new user
1. Create a new user with the `adduser` command and name the user after a favorite cartoon character.
2. Set a secure password and complete other information prompted for.
3. Display the newly created account using the command `cat /etc/passwd | grep {username}`.
### Answer
1. Confirm that the proposed account does not already exist. No output is expected when `velma` is unused.
```bash
getent passwd velma
```
2. Create the user with `adduser`.
```bash
sudo adduser velma
```
3. Enter a secure password when prompted. Complete the requested name and contact fields, or press Enter to leave optional fields blank. Review the summary, and enter `Y` if the information is correct.
4. Display newly created account with the assignment's required command. Anchoring the search to `velma:` avoids matching a different account that contains the same text.
```bash
cat /etc/passwd | grep '^velma:'
```
#### Verification
Confirm the account identifier, primary group, supplementary groups, home directory, and login shell.
```bash
id velma
getent passwd velma
sudo passwd -S velma
```
---
### Ticket 5: Manage files and folders
1. Create folders named Accounting and Payroll.
2. Create a file named Best Practices inside the Accounting folder.
3. Display the contents of the Accounting folder.
### Answer
1. Change to your home directory and confirm the location.
```bash
cd "$HOME"
pwd
```
2. Check whether the required names already exist.
```bash
ls -ld Accounting Payroll 2>/dev/null
```
3. If neither name exists, create both folders.
```bash
mkdir Accounting Payroll
```
4. Display new folders.
```bash
ls -ld Accounting Payroll
```
5. Create a file named `Best Practices` inside the Accounting folder. Quotation marks are required because the filename contains a space.
```bash
touch "Accounting/Best Practices"
```
8. Display contents of the Accounting folder.
```bash
ls -la Accounting
```
#### Verification
Confirm both paths are directories and `Best Practices` is a regular file.
```bash
test -d Accounting && echo "Accounting directory exists"
test -d Payroll && echo "Payroll directory exists"
test -f "Accounting/Best Practices" && echo "Best Practices file exists"
```
---
### Ticket 6: Apply system updates
1. Check for updates using the `sudo apt update` command.
2. Use the `sudo apt upgrade` command to install updates.
3. Use the `less /var/log/dpkg.log` command to display a list of recent updates.
### Answer
1. Record operating system release and check available disk space.
```bash
cat /etc/os-release
df -h /
```
2. Refresh local package index.
```bash
sudo apt update
```
3. Review packages that have upgrades available.
```bash
apt list --upgradable
```
4. Start upgrade.
```bash
sudo apt upgrade
```
5. Read the proposed changes before confirming.
6. Display recent package-management log with the required command.
```bash
less /var/log/dpkg.log
```
#### Verification
Check for incomplete package operations, remaining upgrades, failed services, and a required reboot.
```bash
sudo dpkg --audit
apt list --upgradable
systemctl --failed
test -f /run/reboot-required && echo "Reboot required" || echo "No reboot marker present"
```
No output from `sudo dpkg --audit` indicates that it did not identify a partially installed or inconsistent package. If `apt list --upgradable` lists no packages, that result supports, but does not by itself prove, that the intended updates were installed. Review the APT output and `/var/log/dpkg.log` together.