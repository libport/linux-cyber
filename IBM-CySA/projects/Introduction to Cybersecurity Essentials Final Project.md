# *Introduction to Cybersecurity Essentials* Final Project

During this final project, you will use a Windows Server based lab environment to resolve three service tickets.

This project is divided into three different assignment scenarios. Each scenario has one or more tasks in it. You will have to complete all 6 tasks across the three scenarios and take screenshots as per instructions.
## Scenario - Ticket 1
You are a service desk employee for XYZ Corporation. You have received a ticket indicating that a recent update caused a conflict with an existing application. You are working on a fix for this problem, but in the meantime, you must undo the most recent update and temporarily pause automatic updates.
### Task 1
Uninstall the most recent security update for Microsoft Server.
#### Answer
- Open PowerShell as an administrator: sign in to Server Core with an administrator account -> if Command Prompt opens, enter `powershell`
- Identify the correct update: run the following command -> find and record the newest applicable Windows security update’s **HotFixID** and **InstalledOn** date:
```powershell
Get-HotFix |
    Sort-Object InstalledOn -Descending |
    Format-Table HotFixID, Description, InstalledOn -AutoSize
```
- Find the update’s package name: replace `<KB_NUMBER>` with the recorded KB number -> run the following command -> record the exact **PackageName**:
```powershell
Get-WindowsPackage -Online -PackageName "*<KB_NUMBER>*"
```
- Remove the update: replace `<PACKAGE_NAME>` with the recorded package name -> confirm that it identifies the intended security or cumulative update -> run:   
```powershell
Remove-WindowsPackage -Online -PackageName "<PACKAGE_NAME>" -NoRestart
```
- Restart Windows Server after removal completes:
```powershell
Restart-Computer -Force
```
- Verify the rollback after restarting: sign back in -> reopen PowerShell -> replace `<PACKAGE_NAME>` with the recorded package name -> run the following command:
```powershell
Get-WindowsPackage -Online -PackageName "<PACKAGE_NAME>" |
    Select-Object PackageName, PackageState
```
- Confirm the package is no longer in the **Installed** state. You can also replace `<KB_NUMBER>` below and confirm that the command returns no installed update:
```powershell
Get-HotFix -Id <KB_NUMBER> -ErrorAction SilentlyContinue
```
### Task 2
Temporarily pause automatic updates.
#### Answer
- Open PowerShell as an administrator: sign in to Server Core with an administrator account -> if Command Prompt opens, enter `powershell`
- Choose the pause duration: set `$PauseDays` to a value from `1` through `35`:
```powershell
$PauseDays = 35

if ($PauseDays -notin 1..35) {
    throw 'PauseDays must be between 1 and 35.'
}

$ResumeDate = (Get-Date).Date.AddDays($PauseDays)
$PauseStart = $ResumeDate.AddDays(-35).ToString('yyyy-MM-dd')
```
- Configure the quality-update pause policy:
```powershell
$PolicyPath = 'HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate'

New-Item -Path $PolicyPath -Force | Out-Null

New-ItemProperty -Path $PolicyPath `
    -Name 'DeferQualityUpdates' `
    -PropertyType DWord `
    -Value 1 -Force | Out-Null

New-ItemProperty -Path $PolicyPath `
    -Name 'DeferQualityUpdatesPeriodInDays' `
    -PropertyType DWord `
    -Value 0 -Force | Out-Null

New-ItemProperty -Path $PolicyPath `
    -Name 'PauseQualityUpdatesStartTime' `
    -PropertyType String `
    -Value $PauseStart -Force | Out-Null

gpupdate /target:computer /force
```

- Verify the configured pause and expected resume date:
```powershell
$Policy = Get-ItemProperty -Path $PolicyPath

[PSCustomObject]@{
    UpdatesPaused = $Policy.DeferQualityUpdates -eq 1
    PauseStart    = $Policy.PauseQualityUpdatesStartTime
    ResumeDate    = $ResumeDate.ToString('yyyy-MM-dd')
}
```
- Confirm that **UpdatesPaused** displays `True` and **ResumeDate** displays the expected date.
- Verify the Windows Update status after the policy is processed:
```powershell
Get-ItemProperty `
    -Path 'HKLM:\SOFTWARE\Microsoft\WindowsUpdate\UpdatePolicy\Settings' `
    -Name PausedQualityStatus, PausedQualityDate `
    -ErrorAction SilentlyContinue
```
- Confirm that **PausedQualityStatus** is `1`, meaning quality updates are paused.

---
## Scenario - Ticket 2
A user is unable to access the most current version of a website. The issue may be related to the browser cache. Clear Chrome browsing data without losing the passwords and other sign-in data.
### Task 3
Clear Chrome browsing history, cookies, site data, cached images, and files without losing the passwords and other sign-in data.
#### Answer
- Open browsing data settings: open **Google Chrome** -> press **Ctrl + Shift + Delete** -> select the **Advanced** tab
- Select the required data: set **Time range** to **All time** -> select **Browsing history**, **Cookies and other site data**, and **Cached images and files** -> ensure **Passwords and other sign-in data** is not selected
- Clear and verify: select **Delete data** -> reopen the affected website -> press **Ctrl + F5** -> confirm the current version of the website loads
---
## Scenario - Ticket 3
Because of personalized security controls, a user has requested only to permit the Chrome browser access to the internet. To accomplish this task, you will ensure that the workstation enables Windows Defender Firewall, and that Google Chrome can communicate through the firewall. You will also ensure that Firefox is blocked.
### Task 4:
- Enable Windows Defender Firewall.
#### Answer
- Open PowerShell as an administrator: sign in to Server Core with an administrator account -> if SConfig opens, select option **15** to exit to PowerShell
- Enable the firewall for all network profiles:
```powershell
Set-NetFirewallProfile `
    -Profile Domain, Private, Public `
    -Enabled True
```
- Verify the effective firewall status:
```powershell
Get-NetFirewallProfile -PolicyStore ActiveStore |
    Select-Object Name, Enabled
```
- Confirm that **Enabled** displays `True` for the **Domain**, **Private**, and **Public** profiles.
---
### Task 5:
- Verify that Google Chrome is allowed to communicate through Windows Defender Firewall.
> [!NOTE]
> Windows Defender Firewall GUI is only installed on Server Core with Desktop Experience
> 
#### Answer
- Open the allowed-app list: select **Allow an app or feature through Windows Defender Firewall**
- Enable editing: select **Change settings** -> approve the administrator prompt
- Allow Chrome: find **Google Chrome** -> select its checkbox -> select the appropriate **Private** and **Public** network checkboxes
- Add Chrome if it is not listed: select **Allow another app** -> **Browse** -> select `chrome.exe` -> select **Add**
- Select **OK** -> open Chrome -> load a webpage to verify internet access
### Task 6:
- Block Firefox from communicating through Windows Defender Firewall.
#### Answer
- Identify the Firefox executable: right-click the Firefox shortcut -> select **Open file location** -> confirm the location of `firefox.exe`
- Open advanced firewall settings: press **Windows + R** -> enter `wf.msc` -> press **Enter**
- Create an outbound rule: select **Outbound Rules** -> **New Rule** -> **Program** -> **Next**
- Select Firefox: choose **This program path** -> browse to and select `firefox.exe` -> select **Next**
- Block Firefox: select **Block the connection** -> **Next** -> select **Domain**, **Private**, and **Public** -> **Next**
- Name the rule: enter **Block Mozilla Firefox - Outbound** -> select **Finish**
- Verify the rule: confirm that it displays **Enabled: Yes** and **Action: Block**
- Open Firefox -> attempt to load a webpage -> confirm that Firefox cannot access the internet