# Introduction to Cybersecurity Essentials

> [!NOTE]
> This document introduces foundational cybersecurity concepts and everyday practices for protecting information assets. It explains data and privacy categories (PII, PCI, PHI), common threats (physical risks, insider threats, malware, web attacks, DoS, and social engineering), and how recurring breach patterns often trace back to weak patching, access control, and monitoring. It then covers practical controls such as MFA, SSO, RBAC, auditing, encryption basics, safe password habits, secure browsing on public networks, and VPN/IPsec fundamentals, emphasising defence in depth and least privilege.
## Security Concerns
### Security and information privacy essentials
#### Information assets and analytics
- An information asset is data or information that has value to an organisation, such as patient records, customer details, and intellectual property.
- Data is raw facts. Information summarises data. Insights are conclusions from analysis that support decisions.
- Data-driven decisions depend on:
 - data capture from sources such as logs, IoT sensors, and surveys
 - data correlation to find patterns and relationships
 - reporting that presents analysed results for interpretation
#### Intellectual property and digital products
- Intellectual property covers creations of the mind, including designs, trade secrets, industrial designs, and research discoveries. Protection can involve copyright, trade marks, patents, and non-disclosure agreements.
- Digital products include software, online courses, e-books, and website themes. Risks include piracy, licence misuse, and reverse engineering.
- Digital rights management can restrict copying, although it can be defeated. In the United States, the Digital Millennium Copyright Act generally restricts bypassing certain copy protection measures and trafficking in circumvention tools, with limited exemptions.
#### Confidential information categories
Confidential information must be kept from unauthorised access and disclosure. Typical categories include:
- personally identifiable information, which can identify an individual
- organisation confidential information, such as internal plans, designs, and financials
- customer confidential information, including account details and purchase histories
- protected health information, which is health information linked to an identifiable person

Handling practices usually include least privilege access, encryption, secure storage, logging, secure disposal, and appropriate consent and notice where required.

Terms can be mixed up. In common practice:
- PII refers to identifying personal data
- PCI usually refers to Payment Card Industry standards and cardholder data under PCI DSS
- sensitive personal information is higher-risk personal data that can cause harm if exposed
### Threats and attack techniques
#### Physical and environmental threats
- Threats include theft, tampering, and disasters such as fire, floods, storms, and power outages.
- Controls include restricted physical access, surveillance, fire suppression, backup power, HVAC maintenance, backups, and tested response plans.
#### Data exposure and insider risk
- A data leak is exposure caused by weaknesses or errors. A data breach is unauthorised access or acquisition and may be intentional or accidental.
- Stolen datasets may be sold or published. Poor disposal can expose data, so shredding and secure media destruction are standard controls.
- Insider threats may be malicious, negligent, or coerced. Monitoring, separation of duties, and training reduce risk.
#### Software threats and malware
Threats include licence theft, exploitation of vulnerabilities, and malware delivered through attachments, unsafe downloads, compromised websites, and infected removable media.

Malware categories include:
- viruses, which usually require execution and can infect files
- worms, which can self-propagate by exploiting weaknesses
- trojans, which appear legitimate but deliver malicious payloads
- spyware, which collects credentials and activity
- adware, which can track activity and may be bundled with unwanted programs
- ransomware, which encrypts or locks data to extort payment, with no guarantee of recovery
- Many malware families use packing and polymorphism to evade detection, so fixed percentage claims are unreliable.
- Defences include patching, tested backups, security awareness training, strong authentication, and endpoint protection.
#### Snooping and web application attacks
- Snooping intercepts data in transit, including packet sniffing, on-path attacks, and replay attacks that reuse captured session tokens.
- Controls include encryption in transit, certificate validation, and careful use of untrusted networks.
- Web application threats include cross-site scripting and SQL injection. Prevention relies on input validation and sanitisation, parameterised queries, least privilege database access, and web application firewalls. Error messages may hint at weak input handling but do not prove a vulnerability.
#### Denial of service
- Denial of service attacks exhaust resources so services cannot respond. Distributed variants often use botnets.
- Mitigation uses monitoring, rate limiting, redundancy, upstream filtering, and rehearsed incident response.
#### Social engineering and phishing
- Impersonation can involve evil twin Wi-Fi, lookalike websites, or attackers posing as staff or authorities.
- Techniques include shoulder surfing, baiting, pretexting, and phishing. Variants include spear phishing, whaling, and vishing.
- Controls include training, verification procedures, multi-factor authentication, secure password reset processes, and reporting pathways.
### Controls, tooling, and governance
#### Breach examples and recurring lessons
- Publicly reported incidents at Yahoo, Equifax, Court Ventures, and T-Mobile highlight common failures such as weak patching, poor access control, limited monitoring, and staff susceptibility to social engineering.
- Effective programmes combine technical controls with process controls, including asset inventory, vulnerability management, and incident response practice.
#### Microsoft Defender Antivirus overview
- Microsoft Defender Antivirus provides real-time protection and scan types including quick, full, custom, offline, and scheduled scans.
- Response actions include quarantine and removal. Exclusions should be tightly controlled to avoid blind spots.
- Tamper protection, automatic sample submission, and regular definition updates strengthen detection.
#### CIA triad and regulatory context
- Confidentiality prevents unauthorised disclosure. Integrity protects accuracy and prevents unauthorised change. Availability keeps systems accessible for authorised users.
- Integrity controls can include validation checks, file integrity monitoring, and database audit logging. Availability controls often include redundancy and failover.
- Security and privacy duties are shaped by contracts and law. Examples include HIPAA in the United States for health information and the GDPR in the European Union for personal data, with penalties for non-compliance.
## Security Best Practices
### Authentication, access control, and SSO
#### Core concepts
- Access control sets the boundaries of what a person or system can reach.
- Authorisation grants permission to perform an action or access a resource.
- Authentication verifies identity, often through a login plus an additional check.
#### Authentication factors and methods
Authentication factors are categories of evidence:
- something known, such as a password or PIN
- something possessed, such as a security key, phone, or smart card
- something inherent, such as a fingerprint or face scan
- sometimes context, such as location or device posture

Single-factor authentication relies on one factor, usually a password, and is vulnerable to theft through phishing, malware, credential stuffing, and data breaches.

Two-factor authentication uses two different factors. Multi-factor authentication uses two or more factors and is a preferred baseline where feasible.

Hardware security keys and authenticator apps are typically stronger than SMS one-time codes, which can be exposed through SIM swap and interception risks.
#### Single sign-on
- Single sign-on allows one authentication event to unlock multiple connected applications.
- SSO reduces password fatigue but raises the impact of one compromised account, so it is commonly paired with MFA, strong session controls, and monitoring.
### Access control design and accountability
#### Least privilege and RBAC
- Least privilege limits access to what is required for a role and for the shortest practical time.
- Role-based access control groups users by job function and assigns permissions to the group, which supports consistent provisioning and deprovisioning.
- Effective security requires alignment across access control, authorisation, and authentication. Strong passwords do not compensate for over-permissive roles, and well-defined roles do not help if users are placed in the wrong groups.
#### Digital auditing and activity records
- Audit logs support troubleshooting, security analysis, and forensics by recording events such as logins, configuration changes, and data access.
- Online services may also record device and browser attributes and session identifiers. Cookies can store session state and preferences, and browsing history can reveal visited sites to anyone with device access.
#### Non-repudiation
- Non-repudiation reduces the ability to deny an action, commonly using digital signatures, time-stamped logs, and delivery receipts. In physical settings, controls such as CCTV and badge records can support investigations but are not a substitute for cryptographic proof.
### Device hardening and safe use
#### Patching and firmware security
- Hardening reduces attack surface by disabling unnecessary features and services, applying updates, and using layered controls.
- Patches address known weaknesses. They do not prevent all unknown exploits, so organisations also rely on monitoring, segmentation, and secure configuration.
- Firmware and boot protections include UEFI Secure Boot, which validates trusted boot components, and a Trusted Platform Module, which can protect keys and support integrity checks.
#### Encryption and key concepts
- Encryption converts plaintext into ciphertext using an algorithm and a key.
- Data at rest includes information stored on disks and devices. Data in transit includes information moving across networks.
- Symmetric encryption uses one shared key and is efficient for bulk data.
- Asymmetric encryption uses a public and private key pair and supports key exchange, digital signatures, and certificates within a public key infrastructure.
- Cryptographic hashing produces a fixed-length digest that supports integrity checks and password storage. Hashes are one-way and should be combined with salting and strong password policies.
#### Network protections
- Firewalls filter inbound and outbound traffic based on rules. Host firewalls run on endpoints, and network firewalls sit at network boundaries.
- Rules are typically built around least privilege, allowing required traffic and denying everything else. Excessive rules can reduce performance and increase complexity.
- Public Wi-Fi should be treated as untrusted. Encryption such as HTTPS reduces exposure, and a VPN can add protection, but risky activities should be avoided on unknown networks.
#### Safe sources and default credentials
- Updates and drivers should come from vendor app stores, original equipment manufacturers, or trusted resellers.
- Pirated software and untrusted driver sites are common malware delivery paths.
- Default usernames and passwords should be changed or disabled during setup, including for routers, IoT devices, and administrative interfaces.
### Email, spam, phishing, and password hygiene
#### Spam and phishing
- Spam is unsolicited bulk messaging. It can deliver scams, malware, or credential-harvesting links.
- Phishing uses impersonation and urgency to trigger unsafe actions. Verification is safer when performed by navigating to the legitimate site directly rather than clicking links.
#### Password management practices
- Strong passwords prioritise length, uniqueness, and unpredictability. Reuse increases impact when one site is breached.
- Password changes are most effective when triggered by compromise signals or policy requirements rather than frequent forced rotation.
- Password managers help generate and store unique passwords, reduce typing errors, and can support breach alerts.
- Account privileges should follow least privilege, keeping administrator rights separate from everyday user activity.
## Safe Browsing Practices
### Safe browsing and public network risk
#### What makes public networks risky
- Public Wi-Fi in airports, cafes, hotels, and similar venues is often open or uses a shared password. That makes it easier for attackers on the same network to observe traffic, interfere with connections, or attempt account takeover.
- Some network operators may log browsing activity and monetise it, especially where access is offered in exchange for terms and conditions.
#### Common public browsing attacks
- Eavesdropping and traffic interception on poorly protected connections.
- Session hijacking, where an attacker takes over an authenticated session if weak session controls exist.
- Shoulder surfing, where credentials and PINs are observed in public spaces.
- Impersonation through lookalike sites and links that harvest logins.
#### Safer behaviours
A mobile hotspot or cellular data is usually safer than unknown Wi-Fi.

When public Wi-Fi is unavoidable, risk is reduced by:
- keeping the operating system, browser, and apps updated
- using a host firewall and endpoint protection
- enabling multi-factor authentication on key accounts
- using a reputable VPN where policy permits
- avoiding sensitive transactions on untrusted networks
### Website trust and browser configuration
#### HTTP, HTTPS, and what the lock means
- HTTP sends data without transport encryption, so it is unsuitable for logins and sensitive browsing on untrusted networks.
- HTTPS uses TLS encryption in transit and helps verify the server identity through a certificate, often still called an SSL certificate.
- HTTPS reduces interception risk, but it does not prove a site is legitimate or safe. A convincing scam site can also use HTTPS.
#### Simple checks for suspicious sites
- Inspect the URL carefully for lookalikes, unusual domains, or small character swaps.
- Check certificate details in the browser for the site name and issuer when something looks wrong.
- Use WHOIS and independent reviews to assess unfamiliar sites, while treating forum reports as indicators rather than proof.
- If credentials were entered into a suspected scam site, the recommended response is to exit, run endpoint scans, change the password, and contact the bank if payment details were exposed.
#### Plug-ins, extensions, and toolbars
- Modern browsers largely removed legacy plug-ins. Older web apps may still depend on features such as ActiveX or Java, which increases risk and should be controlled tightly.
- Extensions run with elevated browser permissions. Only reputable extensions should be installed, permissions should be reviewed, and unused extensions should be removed.
- Toolbars were historically a major source of nuisance software, tracking, and performance issues. Modern security baselines discourage them.
### Privacy data in browsers
#### Cookies, cache, and history
- Cookies store small pieces of data used for sessions and preferences. Tracking can occur through third-party cookies and similar mechanisms.
- Session cookies are typically short-lived. Persistent cookies may remain until expiry or deletion.
- Large caches can slow performance over time. History and saved form data can expose browsing behaviour to anyone with device access.
#### Private browsing limits
- Private browsing generally avoids saving local history and persistent cookies on that device.
- It does not hide activity from the network, an employer or school network, or the internet service provider.
- Downloads and bookmarks can still remain after the private session ends.
### Messaging, social platforms, and adware
#### Social engineering and unsafe links
- Social networks and messaging apps are frequent targets for phishing and impersonation. Attackers often rely on urgency, familiarity, or threats to push unsafe clicks.
- Security varies by platform. Some services support end-to-end encryption for message content, while others do not. Attachments and links can still carry risk regardless of encryption.
#### Safer handling of chat and files
- Sensitive data is not shared over chat on untrusted networks or unmanaged devices.
- Unexpected links and attachments are treated as malicious until verified, even when they appear to come from a known contact.
#### Adware and redirects
- Adware is unwanted software that delivers intrusive advertising and may track activity or redirect to harmful sites.
- Common signs include new extensions, changed home pages, aggressive popups, sluggish performance, and unexpected redirects.
- Prevention relies on trusted software sources, careful installs, and regular updates and scans.
### VPN fundamentals and IPsec
#### What a VPN provides
- A virtual private network creates an encrypted tunnel between endpoints so that traffic is unreadable to most intermediaries. It helps reduce eavesdropping risks on untrusted networks.
- VPNs do not eliminate all risk. Malware, phishing, and unsafe sites still compromise users who approve harmful actions.
#### Common VPN connection patterns
- Site-to-site VPN connects two networks over the internet using VPN devices at each site.
- Host-to-site VPN, also called remote access VPN, connects a user device to a corporate network.
- Host-to-host VPN connects one device to another device, often used for limited peer connectivity.
#### Hardware and software options
- VPN functions can be delivered by routers, firewalls, and VPN concentrators, or through software clients on endpoints.
- Centralised devices simplify management for many connections, while software clients support mobile and remote users.
#### IPsec building blocks
IPsec is a set of standards that can provide confidentiality and integrity for IP traffic.

Key components include:
- Encapsulating Security Payload for encryption and integrity
- Authentication Header for integrity and source authentication, used less often in practice
- Internet Key Exchange to negotiate keys and security parameters
- anti-replay protection to reduce token and packet reuse attacks

Tunnel mode typically encrypts the full original packet inside a new packet and is common for site-to-site use. Transport mode encrypts the payload while leaving the outer IP header visible and is often used for some remote access designs.
### Application ecosystem security
#### Mobile apps, desktop apps, and compromise paths
- Apps can be compromised through weak credentials, malware, unsafe downloads, and design flaws. Risks include access to contacts, messages, files, and business data.
- Installing only from official app stores and keeping devices updated reduces risk.
- Rooting and jailbreaking can bypass platform safeguards and increase exposure, so they are discouraged in most organisational environments.
#### Organisational controls and shared data
- Businesses reduce risk through patching, security tooling, staff education, and layered controls.
- File sharing needs to remain usable. Excessive friction can drive staff to unapproved consumer tools.
- Access is typically limited to a genuine need, supported by multi-factor authentication, monitoring, and clear sharing policies.