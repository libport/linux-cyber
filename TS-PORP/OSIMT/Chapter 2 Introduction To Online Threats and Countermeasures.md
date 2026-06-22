# Chapter 2: Introduction To Online Threats and Countermeasures
## Operational risk in OSINT
Open source intelligence work leaves technical, behavioural and documentary traces. Search terms, account logins, browser settings, IP addresses, device identifiers, file metadata and payment records can connect an investigation to an individual or organisation. Criminal groups, extremist networks and hostile services can monitor their web infrastructure, correlate visitors and exploit careless contact. OSINT practitioners therefore need a clear threat model before they collect, store or share information.

Investigators should also control time, language and behavioural signals. Repeated visits at unusual hours, consistent typing patterns, reused usernames, repeated browser window sizes and predictable search sequences can assist correlation. A collection plan should define when to browse, which identity to use, which network path to use and where to store results. Consistency inside a persona matters, but consistency across personas can reveal the operator.

Good operational security starts with separation. Investigators should use dedicated accounts, devices, virtual machines or live operating systems for sensitive research. They should avoid mixing personal browsing, work accounts and investigative activity. The strongest configuration still fails when an operator logs into a personal account, reuses a recognisable writing style, uploads a file with metadata, or searches from an identifiable network without authorisation.

Anonymity has limits. Tools such as VPNs, Tor, hardened browsers and live operating systems reduce attribution risk, but they do not make poor tradecraft safe. Investigators should match controls to the risk of the target, the legal authority for the work, the sensitivity of the data and the likely capability of adversaries.
## Main online threats
Malware is hostile code that damages systems, steals information, spies on users or helps an attacker maintain access. Common forms include ransomware, spyware, trojans, worms, rootkits, scareware and adware. Some adware appears inside otherwise legitimate installers, while spyware and trojans usually act covertly. Rootkits hide deeply in a system and may resist ordinary antivirus detection. Worms spread across networks and can install backdoors or consume large volumes of bandwidth.

Pharming redirects users from a legitimate destination to a fraudulent one. Attackers may change local host records, compromise routers, or poison DNS resolution. On Windows, the hosts file belongs in `C:\Windows\System32\drivers\etc`, not in a misspelt system directory. Making the hosts file read-only can reduce one local attack path, but it does not protect DNS, browsers, routers or malicious browser extensions. Secure DNS, router hardening, operating system updates and endpoint protection provide broader defence.

Phishing uses impersonation and social pressure to obtain credentials, money or sensitive information. Attackers may use email, SMS, messaging apps, phone calls, cloned login pages and malicious QR codes. Older warning signs such as spelling errors still matter, but modern phishing can look polished, personalised and technically convincing. Users should not rely on grammar alone. They should inspect the sender, domain, link destination and request. They should verify sensitive requests through a trusted channel rather than using contact details supplied in the message.

Ransomware encrypts data, locks devices, steals information, or combines these tactics with extortion. It often enters through phishing, exposed remote access, stolen credentials, malicious advertising, vulnerable internet-facing services or a compromised software supply chain. Strong prevention includes tested offline or immutable backups, fast patching, multi-factor authentication, restricted administrator privileges, application control, macro restrictions, email filtering and endpoint monitoring. When ransomware strikes, organisations should isolate affected systems, preserve evidence, seek specialist help, restore from clean backups and report the incident. Paying a ransom does not guarantee recovery or deletion of stolen data.

DDoS attacks overwhelm online services with traffic from many sources. They usually seek disruption rather than data theft. Defence depends on upstream filtering, capacity planning, content delivery networks, DDoS protection services and incident playbooks.

Juice jacking can occur when public USB charging points expose a device to data access or malware. Modern mobile operating systems reduce this risk through prompts and restricted modes, but caution remains appropriate. Travellers should use power-only cables, USB data blockers or trusted chargers.

Public Wi-Fi creates interception and impersonation risk. Attackers can run rogue access points, capture unencrypted traffic, force downgrade attempts, or exploit weak captive portals. Users should prefer mobile data or trusted networks for sensitive work. When public Wi-Fi is unavoidable, they should use a reputable VPN, HTTPS, secure DNS where appropriate and device firewalls.
## Defensive baseline
Security software helps, but it cannot carry the whole defence. Modern Windows includes Microsoft Defender Antivirus and Windows Security, which manage built-in protections such as virus and threat protection and firewall status. Third-party security suites may add useful capabilities for organisations, but multiple real-time antivirus products can conflict. A better baseline combines maintained endpoint protection, automatic security updates, least privilege, multi-factor authentication, safe configuration and user training.

Updates matter because attackers exploit known vulnerabilities quickly. Operating systems, browsers, office suites, PDF readers, email clients, VPN clients and security tools need routine patching. Critical internet-facing vulnerabilities and actively exploited flaws require urgent treatment. Unsupported operating systems and applications should be replaced or isolated.

Users should avoid administrator accounts for routine browsing and research. A standard user account limits the effect of many accidental installs and drive-by attacks. User Account Control can warn before privileged changes, but it is not a complete security boundary. Remote access features, remote assistance, file sharing and unused services should stay disabled unless a documented need exists.

Strong authentication now favours long, unique passwords or passphrases stored in a password manager, plus multi-factor authentication. Routine forced password changes can reduce security when users choose predictable replacements. Passwords should change after suspected compromise, exposure in a breach, loss of a device, or departure of an authorised user.

Firmware and boot protections also matter. UEFI passwords can slow unauthorised boot changes, but they should not replace full disk encryption. BitLocker, FileVault, LUKS or similar tools protect data at rest. Recovery keys require secure storage because a lost recovery key can permanently block access.

Biometric sign-in can improve usability and phishing resistance when properly configured. Windows Hello stores biometric templates locally and does not send them to Microsoft. For high-risk OSINT, investigators should still consider whether biometrics create legal, physical coercion, border search or device seizure risks in the relevant jurisdiction.
## Incident handling and reporting
Security controls need an incident process. Individuals and organisations should decide in advance who can disconnect systems, preserve logs, call legal advisers, contact insurers, notify regulators and speak to affected people. A rushed response can destroy evidence or spread malware. A delayed response can let an attacker move further across a network.

For suspected compromise, responders should record the time, symptoms, visible ransom notes, suspicious emails, unusual logins, changed files, network indicators and affected accounts. They should isolate affected devices from networks without wiping them unless safety requires it. They should rotate credentials from a clean device, revoke active sessions and preserve copies of relevant emails, logs and malware samples for specialists.

Phishing victims should change exposed passwords, enable multi-factor authentication, review account forwarding rules, check recovery details, notify affected organisations and report the scam through national channels. Financial fraud requires immediate contact with the bank or payment provider. Early reporting can help freeze transfers and warn other victims.
## Windows and browser privacy
Windows, macOS, Linux, Android and iOS all collect some diagnostic and account data. Investigators should review privacy settings, disable unnecessary telemetry, block location access where unneeded and restrict camera, microphone, contacts and clipboard permissions. They should avoid signing into personal cloud accounts from investigative environments.

Private browsing reduces local traces such as browsing history and session cookies after the private window closes. It does not make the user anonymous to websites, employers, network operators, VPN providers, malware, browser extensions or account services. A hardened browser should use HTTPS-only mode, tracking protection, careful extension choices and separate profiles for separate tasks.

Browser extensions can help or harm. uBlock Origin can block ads, trackers and some malicious domains in supported browsers. Privacy-focused extensions should come only from trusted stores and reputable developers. Installing many privacy extensions can increase fingerprint uniqueness and create new attack surfaces.

Browser fingerprinting identifies users through combinations of device and browser characteristics, such as fonts, canvas rendering, screen size, time zone, language, installed extensions and hardware details. Changing many settings can make a browser more unique. Investigators often gain better cover by using common, updated configurations, Tor Browser for anonymity, or a clean virtual machine profile rather than a heavily customised browser.
## Data, metadata and physical security
Digital files often carry metadata. Images may contain camera model, time, thumbnails and GPS coordinates. Office documents can contain author names, tracked changes, comments, templates and revision history. PDFs, audio and video files can carry creator, location and software data. Investigators should strip metadata before sharing files unless they intentionally preserve it as evidence. They should use tested tools and preserve original evidence separately with a documented chain of custody.

A sound metadata workflow separates source evidence from publication copies. The original file should remain unchanged, hashed and stored in a controlled evidence location. A working copy can be reviewed for hidden data, converted if needed and sanitised before disclosure. Screenshots should capture context without exposing unrelated tabs, bookmarks, usernames or system notifications. Redaction must remove the underlying content, not merely draw a black box over visible text.

Data deletion requires more than moving files to the recycle bin. On hard disk drives, deletion commonly removes file references while data remains recoverable. On solid-state drives, wear levelling and TRIM make recovery and wiping less predictable. Modern sanitisation should follow an approved standard and match the sensitivity of the data. Options include clearing, purging, cryptographic erase and physical destruction. For SSDs and encrypted devices, cryptographic erase or manufacturer-supported secure erase usually works better than repeated overwriting. Highly sensitive media may require physical destruction with validation.

Physical security remains fundamental. Laptops and removable drives should use full disk encryption and strong screen locks. Devices should not remain unattended in public places or unlocked offices. Bluetooth and Wi-Fi should stay off when not needed. Asset records should include model, serial number and other identifiers. Sensitive files should not sit on portable devices without authorisation and encryption.
## Tracking and network anonymity
Websites, advertisers, social platforms and network operators can track users through IP addresses, cookies, login identifiers, browser storage, link decoration, device fingerprinting and embedded third-party content. A public IP address can reveal an approximate location and identify an internet service provider. A private internal IP address only identifies a device inside a local network.

Cookies support legitimate sessions, shopping carts and preferences, but trackers use them across sites. Modern browsers limit some third-party tracking, and the retirement of Flash removed a major source of Flash cookies. Persistent tracking now relies more on browser storage, fingerprinting, account correlation, mobile advertising IDs, analytics scripts and server-side data sharing.

DNS also creates exposure. A device usually asks a resolver to translate domain names into IP addresses. If those queries bypass a VPN or go to an untrusted resolver, they can reveal browsing destinations even when web content uses HTTPS. Encrypted DNS can reduce local interception, but it shifts trust to the chosen resolver. OSINT practitioners should document DNS settings in their operating environment and test them before sensitive browsing.

A VPN encrypts traffic between the device and VPN provider and hides the user's IP address from destination websites. It does not make a person anonymous. The VPN provider can see connection metadata, and poor VPN configuration can leak DNS, IPv6 or WebRTC traffic. Investigators should test for leaks, choose reputable providers, avoid free proxies for sensitive work and understand the provider's jurisdiction, logging claims and audit history.

Proxies can change the apparent source IP address for specific traffic, but many free proxies inject ads, log traffic or expose credentials. A proxy should not carry sensitive investigative traffic unless the operator controls it or has assessed it properly.

Tor provides stronger anonymity for web browsing by routing traffic through multiple relays. Destination sites see the exit relay rather than the user's real IP address. Tor Browser only protects traffic inside Tor Browser. Other applications on the same device do not automatically use Tor. Users should not install extra browser extensions, resize the browser unnecessarily, download and open risky files outside the protected environment, or log into personal accounts during anonymous sessions.

Bridges and pluggable transports help users reach Tor where ordinary Tor connections face blocking or monitoring. They reduce the visibility of Tor use, but they do not guarantee undetectability against capable network observers. Tails and similar live operating systems offer stronger compartmentalisation by running from removable media, reducing local traces and routing supported traffic through Tor or blocking it. They still require disciplined behaviour.

OnionShare provides a practical way to share files through Tor without a third-party storage provider. The sender must protect the onion address and keep the sharing session available until transfer finishes. Sensitive sharing still needs encryption, recipient verification and secure handling of the downloaded material.
## Payments, accounts and legal boundaries
Payments can reveal investigative activity. Card payments expose names, banks, merchants, times and sometimes billing details. Prepaid cards can reduce attribution in some situations, but availability and legal rules vary. Cryptocurrency does not provide automatic anonymity. Bitcoin and many public blockchains are pseudonymous and permanently record transactions. Investigators should assume blockchain analytics can connect wallets, exchanges, IP data and identity records.

OSINT work sometimes requires platform accounts or personas. These should be authorised, documented and legally defensible. Investigators should not create deceptive accounts unless policy, law and the platform context permit it. Personal accounts should never mix with investigative accounts. Dedicated email addresses, phone numbers, devices and payment methods support compartmentalisation, but they do not remove legal obligations.
## Encryption and secure communications
Encryption protects data in transit and at rest. It does not protect compromised endpoints, weak passwords, exposed recovery keys or careless sharing. Password managers help users create unique passwords for each account and reduce reuse. Multi-factor authentication should protect email, cloud storage, password managers, VPN accounts, administrative consoles and financial accounts.

Full disk encryption protects lost or seized devices when they are powered off or properly locked. File or container encryption helps when sharing selected material. Cloud storage should not hold sensitive material unless files are encrypted before upload or the provider supplies suitable end-to-end encryption for the purpose. Backup design should include tested restoration, separate credentials and resistance to ransomware deletion.

Email is difficult to secure perfectly. OpenPGP and S/MIME can protect message content when both parties manage keys correctly, but metadata such as sender, recipient, subject timing and server logs may remain exposed. Modern Thunderbird includes OpenPGP support, so older instructions that rely on the Enigmail add-on need updating. For routine secure messaging, Signal provides end-to-end encrypted messages and calls, but users must still verify contacts, control group membership, protect devices and manage backups carefully.
## Virtualisation and working methods
Virtual machines let investigators isolate tools, websites and accounts from the host environment. Snapshots can return a system to a known state, and disposable virtual machines can reduce residue after a task. Virtualisation does not stop all malware, and hostile code can sometimes detect or escape weakly configured sandboxes. Host patching, network controls and isolation remain essential.

Live USB systems and dedicated laptops provide stronger separation for high-risk work. Kali Linux and similar distributions contain many security and investigation tools, but tools alone do not create good evidence. Investigators need repeatable processes, notes, source preservation and clear reporting.

OSINT teams should organise findings with local notes, timelines, diagrams, bookmarks and evidence registers. Cloud-based note and diagram tools can leak sensitive searches or case details if used without approval. Translation tools help triage foreign-language sources, but they may send text to third-party servers. Sensitive content should be translated through approved channels.
## Practical operating rules
- Define the threat model before collection starts.
- Separate personal, work and investigative identities.
- Use updated operating systems, browsers and security tools.
- Prefer standard user accounts and restrict administrator rights.
- Enable multi-factor authentication and use a password manager.
- Keep tested offline or immutable backups.
- Strip metadata before sharing non-evidentiary files.
- Use full disk encryption on laptops and removable media.
- Use Tor Browser or Tails for anonymity needs, not an ordinary browser with random extensions.
- Test VPN configurations for DNS, IPv6 and WebRTC leaks.
- Avoid free proxies and pirated software.
- Preserve evidence separately from working copies.
- Report cybercrime and ransomware through appropriate national channels.