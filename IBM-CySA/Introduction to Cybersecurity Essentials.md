# Introduction to Cybersecurity Essentials
> [!NOTE]
> Effective cybersecurity combines governance, trained people, secure processes, and layered technical controls to protect data, devices, accounts, networks, browsing activity, and applications.
## Security concerns
### Information assets, privacy, and intellectual property
#### Information assets and analytics
An information asset is data or information that provides value to an organisation. Examples include patient records, customer details, financial records, operational logs, source code, research data, and intellectual property.

Data consists of recorded facts or observations. Information gives those data context and meaning. Analysis can produce insights that support decisions, but the quality of those insights depends on the relevance, accuracy, completeness, and timeliness of the underlying data.

Data-informed decisions usually involve three activities:
- Systems capture data from sources such as transaction records, security logs, sensors, and surveys.
- Analysts correlate and interpret the data to identify patterns, relationships, and anomalies.
- Reports communicate the results, limitations, and implications to decision-makers.
#### Intellectual property and digital products
Intellectual property can include inventions, literary and artistic works, software, designs, trade marks, and confidential commercial know-how. Different rights protect different types of creation or information. Patents can protect qualifying inventions, copyright can protect original expression such as software code, registered designs can protect a product's visual appearance, and trade marks can distinguish goods or services. Trade secret protection depends on keeping commercially valuable information confidential. Non-disclosure agreements can support that protection but do not create every form of intellectual property right.

Digital products include software, online courses, e-books, templates, and website themes. Common risks include unauthorised copying, licence breaches, account sharing, piracy, and the extraction of protected code or confidential information. The legality of reverse engineering depends on the jurisdiction, licence terms, purpose, and applicable exceptions.

Digital rights management can control access to or use of digital content, although attackers may bypass it. In the United States, section 1201 of the Digital Millennium Copyright Act restricts circumvention of technological measures that control access to copyrighted works and restricts trafficking in certain circumvention technologies. Permanent exceptions and periodically reviewed exemptions apply in defined circumstances.
#### Confidential and regulated information
Organisations classify information according to its sensitivity, business value, legal requirements, and potential impact if exposed or altered. Terminology varies across jurisdictions and industries.

| Term | Meaning |
| --- | --- |
| Personal information | Information or an opinion that identifies a person or enables reasonable identification. Some jurisdictions use personally identifiable information, or PII, for a related concept. |
| Sensitive information | A higher-risk subset of personal information. Under Australian privacy law, it includes categories such as health, genetic, certain biometric, racial or ethnic origin, political opinion, religious belief, sexual orientation, and criminal record information. |
| Protected health information | A term defined by the United States HIPAA rules for individually identifiable health information that covered entities or business associates create, receive, maintain, or transmit, subject to specified exclusions. |
| Cardholder data | Payment account data covered by the Payment Card Industry Data Security Standard, or PCI DSS. PCI names the industry standards framework, not a category of personal information. |
| Organisational confidential information | Non-public business information such as strategies, security designs, pricing, source code, financial results, and trade secrets. |

Organisations protect sensitive information through data minimisation, least-privilege access, encryption, secure storage, monitoring, retention controls, secure disposal, and appropriate consent and notice. Applicable law, contracts, and industry standards determine the exact obligations.
### Threats and attack techniques
#### Physical and environmental threats
Theft, tampering, fire, flood, storms, extreme temperatures, and power failures can disrupt systems or expose information. Organisations reduce these risks through controlled physical access, surveillance where lawful, fire detection and suppression, backup power, environmental monitoring, off-site backups, and tested continuity and recovery plans.
#### Data breaches and insider risk
A data breach occurs when an unauthorised party accesses or receives information, or when an organisation loses information in circumstances that expose it to unauthorised access or disclosure. Malicious activity, human error, and control failures can all cause a breach. "Data leak" is an informal term and does not reliably distinguish accidental exposure from a data breach.

Attackers may sell, publish, exploit, or use stolen data for extortion. Poor disposal can also expose information, so organisations should shred sensitive paper records and sanitise or physically destroy storage media according to risk and policy.

Insider threats can involve malicious, negligent, compromised, or coerced people. Least privilege, separation of duties, monitoring, secure offboarding, and regular training reduce the likelihood and impact of insider activity.
#### Software threats and malware
Attackers exploit software vulnerabilities and deliver malware through phishing, compromised websites, unsafe downloads, supply-chain compromises, and infected removable media.

Common malware categories include:
- A virus attaches to a file or program and usually spreads when someone executes the infected host.
- A worm spreads between systems without relying on a user to run an infected file, often by exploiting a vulnerability or weak configuration.
- A trojan presents itself as legitimate software while performing an undisclosed malicious function.
- Spyware collects information such as credentials, communications, and browsing activity.
- Adware displays intrusive advertising and may track activity or install other unwanted components.
- Ransomware can encrypt data, lock systems, steal information, and pressure victims through threats of disclosure or disruption. Paying a ransom does not guarantee recovery or deletion of stolen data.

Malware developers use techniques such as packing, obfuscation, and polymorphism to hinder detection. Organisations should combine prompt patching, application control, tested offline backups, phishing-resistant authentication, staff training, network segmentation, monitoring, and endpoint protection.
#### Network interception and web application attacks
Network attackers may capture unencrypted traffic, position themselves between communicating systems, or hijack sessions. Replay attacks resend captured authentication messages or protocol data. Secure protocols use encryption, integrity protection, fresh challenges, nonces, or sequence numbers to resist interception and replay.

Cross-site scripting allows untrusted content to execute in a user's browser. Developers should use framework protections, context-appropriate output encoding, safe browser interfaces, and HTML sanitisation where an application accepts HTML. A content security policy can limit impact, while a web application firewall provides an additional control rather than a primary fix.

SQL injection allows untrusted input to alter a database query. Developers should use prepared statements with parameterised queries, validate inputs, and give application database accounts only the permissions they need. Generic input sanitisation alone does not reliably prevent either cross-site scripting or SQL injection.
#### Denial of service
Denial-of-service attacks consume bandwidth, processing capacity, memory, connections, or another limited resource until legitimate users cannot access a service. Distributed denial-of-service attacks coordinate traffic from many systems, which may form part of a botnet.

Organisations reduce the impact through monitoring, rate limiting, capacity planning, resilient architecture, content delivery services, upstream filtering, and rehearsed incident response.
#### Social engineering and phishing
Social engineering manipulates people into disclosing information, transferring money, granting access, or running malicious content. Attackers may create an evil-twin wireless network, copy a legitimate website, or impersonate a colleague, supplier, executive, or public authority.

Common techniques include shoulder surfing, baiting, pretexting, and phishing. Spear phishing targets a specific person or group, whaling targets senior or influential people, and vishing uses voice calls or messages.

Organisations reduce these risks through practical training, independent verification for sensitive requests, phishing-resistant multi-factor authentication, secure account recovery, payment controls, and simple reporting pathways.
### Controls, security tools, and governance
#### Lessons from security incidents
Public incident reports repeatedly identify unpatched systems, weak authentication, excessive privileges, exposed services, inadequate monitoring, insecure third parties, and poor response preparation. An effective security program therefore combines asset inventory, secure configuration, vulnerability management, access governance, logging, backups, supplier assurance, and incident response exercises.
#### Microsoft Defender Antivirus
Microsoft Defender Antivirus provides real-time, cloud-delivered, and behaviour-based protection on supported Windows systems. Its on-demand scan types are quick, full, and custom. Administrators can schedule quick or full scans, while Microsoft Defender Offline runs in a trusted environment outside the normal Windows session to investigate persistent threats.

Defender can quarantine, remove, or allow detected items according to configuration and policy. Administrators should restrict exclusions because every exclusion creates a monitoring gap. Tamper protection, automatic sample submission, cloud protection, and current security intelligence improve its ability to prevent and detect malicious activity.
#### The CIA triad and legal duties
The confidentiality, integrity, and availability triad describes three central security objectives.

| Objective | Purpose | Example controls |
| --- | --- | --- |
| Confidentiality | Prevent unauthorised access or disclosure. | Access control, encryption, data classification, and secure disposal. |
| Integrity | Prevent or detect unauthorised or accidental alteration. | Digital signatures, validation, file integrity monitoring, and audit logging. |
| Availability | Keep systems and information accessible to authorised users when needed. | Redundancy, failover, backups, capacity management, and recovery testing. |

Legal and contractual duties depend on the organisation, information, location, and activity. In Australia, the Privacy Act 1988 and the Australian Privacy Principles regulate covered entities' handling of personal information. The United States HIPAA rules apply to specified covered entities and business associates rather than to every holder of health information. The European Union's General Data Protection Regulation governs the processing of personal data within its scope. Non-compliance can trigger regulatory action, contractual consequences, and financial penalties.
## Security best practices
### Authentication, access control, and single sign-on
#### Core concepts
- Authentication verifies a claimed identity.
- Authorisation determines which actions or resources an authenticated identity may access.
- Access control enforces authorisation decisions through policies and technical mechanisms.

A secure system aligns all three functions. Strong authentication cannot correct excessive permissions, and carefully designed roles cannot protect an account that an attacker can easily take over.
#### Authentication factors and methods
Authentication uses three recognised factor types.

| Factor | Description | Examples |
| --- | --- | --- |
| Knowledge | Something the person knows. | A password or PIN. |
| Possession | Something the person has. | A security key, smart card, or device containing a cryptographic key. |
| Inherence | Something the person is. | A fingerprint, face characteristic, or other biometric trait. |

Location, network, device posture, and behaviour can inform adaptive access decisions, but they do not form a fourth authentication factor under widely used digital identity standards.

Single-factor authentication relies on one factor. Attackers can steal passwords through phishing, malware, credential stuffing, and data breaches. Multi-factor authentication requires evidence from at least two distinct factors and limits the value of a stolen password.

FIDO2 and WebAuthn security keys and passkeys can provide phishing-resistant authentication. Authenticator-app one-time codes avoid some telecommunications risks but remain vulnerable to real-time phishing. SMS codes face additional risks from SIM swapping, number reassignment, and message interception.
#### Single sign-on
Single sign-on allows a trusted identity provider to authenticate a user for multiple connected services. It can reduce password reuse and simplify account provisioning, deprovisioning, and policy enforcement. It also concentrates risk in the identity provider and its sessions. Organisations should protect single sign-on with strong multi-factor authentication, short-lived and well-controlled sessions, conditional access, monitoring, and tested recovery procedures.
### Access control design and accountability
#### Least privilege and role-based access control
Least privilege gives each person, service, and process only the access required for an authorised task and only for as long as required. Role-based access control assigns permissions to defined job roles, then assigns people to those roles. This approach supports consistent onboarding, job changes, access reviews, and offboarding when organisations maintain accurate roles and group membership.

Privileged access requires stronger controls. Organisations should separate administrator accounts from everyday accounts, approve and record elevated access, and review permissions regularly.
#### Audit logs and activity records
Audit logs record events such as authentication attempts, permission changes, administrative actions, configuration changes, and access to sensitive data. Well-designed logging supports operations, threat detection, incident response, and forensic analysis. Organisations should protect logs from alteration, synchronise system time, define appropriate retention periods, and restrict access to authorised personnel.

Online services may also record device attributes, browser characteristics, IP addresses, and session identifiers. Cookies can maintain sessions and preferences, while browser history and saved form data can reveal activity to anyone who gains access to the device or profile.
#### Evidence and non-repudiation
Digital signatures can provide strong evidence that a particular private key approved data and that the data has not changed. Trusted timestamps, protected audit logs, and delivery records can strengthen attribution. CCTV footage, badge records, and ordinary receipts can support an investigation, but they do not provide the same cryptographic assurance. Legal weight also depends on key custody, system design, identity proofing, and evidentiary rules.
### Device hardening and safe use
#### Patching and firmware security
Hardening reduces attack surface. Organisations should remove or disable unnecessary software, accounts, services, and interfaces, apply secure configurations, and install security updates within risk-based time frames.

Patches correct known weaknesses but cannot prevent every new or unknown exploit. Monitoring, network segmentation, application control, least privilege, and resilient backups provide additional protection.

UEFI Secure Boot checks authorised signatures before the system loads boot components. A Trusted Platform Module can protect cryptographic keys and record integrity measurements. These technologies strengthen startup security when manufacturers and administrators configure and maintain them correctly.
#### Encryption, hashing, and keys
Encryption uses an algorithm and a key to transform plaintext into ciphertext. Data at rest resides on storage media, while data in transit moves between systems or across networks.

- Symmetric encryption uses a shared secret key to encrypt and decrypt data. It suits high-volume data protection.
- Asymmetric cryptography uses a public and private key pair. It supports digital signatures, key establishment, and certificates within a public key infrastructure.
- Cryptographic hashing maps input to a fixed-length digest. Hashing supports integrity checks but does not encrypt data and does not use a decryption key.

Systems should store passwords with a purpose-built, salted password-hashing scheme and an appropriate cost factor. They should never store passwords with fast general-purpose hashes alone. Cryptographic controls also require secure key generation, storage, rotation, backup, recovery, and destruction.
#### Network protections
Firewalls allow or block traffic according to policy. Host firewalls protect individual endpoints, while network firewalls control traffic between networks or security zones. Administrators should permit required traffic, deny unnecessary traffic, document exceptions, review rules, and remove obsolete entries. Large or inconsistent rulesets increase complexity and the chance of configuration errors.

Users should treat public wireless networks as untrusted. HTTPS protects the content and integrity of properly secured web connections, while a reputable virtual private network can protect traffic between a device and the VPN gateway. Neither control protects a user who approves a fraudulent login, installs malware, or ignores a certificate warning.
#### Trusted sources and default credentials
Users should obtain operating-system updates, applications, firmware, and drivers from the vendor, an official app store, an original equipment manufacturer, or another source the organisation has approved. Pirated software, deceptive advertisements, and untrusted download sites often distribute unwanted or malicious programs.

Administrators should replace or disable default credentials during setup. They should also prevent internet access to management interfaces unless a documented need and suitable protections exist.
### Email, phishing, and password hygiene
#### Spam and phishing
Spam consists of unsolicited bulk messages. It can advertise legitimate products, but it also carries scams, malware, and credential-harvesting links. Phishing uses impersonation, urgency, authority, fear, or opportunity to push a target towards an unsafe action.

People should verify sensitive requests through a known, independent channel. They should open a service through a saved bookmark, trusted application, or manually entered address instead of following an unexpected link.
#### Password practices
Strong password practice prioritises length and uniqueness. Each account should use a different password or passphrase so that a breach of one service does not expose other accounts. Services should reject common and compromised passwords, permit password managers and autofill, and avoid arbitrary periodic changes. They should require a change when evidence indicates compromise.

Password managers can generate and store unique credentials, reduce reuse, and help users recognise a website for which no matching credential exists. Users should protect the manager with a strong master credential and multi-factor authentication where available.

Organisations should enable phishing-resistant authentication for important and privileged accounts where feasible. They should also keep administrator privileges separate from routine email, browsing, and office work.
## Safe browsing practices
### Public networks and safer connections
#### Risks on public networks
Public wireless networks in airports, cafes, hotels, and similar venues may be open, share one password among many people, or imitate a legitimate network. Properly configured HTTPS prevents nearby observers from reading or changing protected web content, but attackers can still target unencrypted traffic, spoof network names, exploit exposed device services, redirect insecure requests, or present fraudulent sites.

Network operators can observe connection metadata and any unencrypted traffic. Their privacy practices depend on their policies, technical controls, and applicable law.
#### Common attacks
- Eavesdropping captures traffic that lacks effective encryption.
- On-path interference alters or redirects insecure communications.
- Session hijacking takes over an authenticated session after an attacker steals or fixes a valid session identifier.
- Shoulder surfing captures information displayed or entered in a public place.
- Impersonation uses a lookalike network, website, account, or message to collect credentials or payments.
#### Safer behaviour
A personal mobile hotspot or cellular connection generally reduces exposure to unknown local network operators and users. When a public network is unavoidable, users should:
- confirm the network name with the venue
- disable automatic connection and local sharing
- keep the operating system, browser, and applications current
- use a host firewall and supported endpoint protection
- use HTTPS and heed browser certificate warnings
- enable multi-factor authentication on important accounts
- use an organisation-approved or reputable VPN where appropriate
- postpone high-risk transactions when the network or device appears unsafe
### Website trust and browser configuration
#### HTTP, HTTPS, and browser security indicators
HTTP does not encrypt web traffic. It cannot protect credentials or content against an observer or active attacker on the network. HTTPS uses Transport Layer Security to encrypt traffic in transit, detect alteration, and authenticate the server for the requested domain through a certificate.

HTTPS protects the connection, not the honesty of the website operator. Attackers can obtain valid certificates for deceptive domains, so a lock icon does not prove that a site is legitimate or safe.
#### Checks for suspicious websites
- Inspect the full domain name for misspellings, substituted characters, misleading subdomains, and unexpected top-level domains.
- Navigate through a trusted application, bookmark, or independently located official page when a message requests a login or payment.
- Stop when the browser reports a certificate or deceptive-site warning unless authorised technical staff confirm a legitimate cause.
- Treat domain-registration details, website design, reviews, and search ranking as supporting indicators rather than proof of trust.

If a person enters credentials into a suspected phishing site, the person should close the site, open the legitimate service independently, change the exposed password, sign out other sessions, review recovery details, enable multi-factor authentication, and report the incident. The person should contact the financial institution immediately if payment information or money is at risk. A device scan becomes especially important if the person downloaded a file, installed software, granted remote access, or observed signs of compromise.
#### Browser extensions and legacy plug-ins
Modern browsers have removed or restricted many legacy plug-in technologies. Organisations should isolate any application that still depends on obsolete components and retire the dependency as soon as practical.

Browser extensions can read or change sensitive browsing data when their permissions allow it. Users and administrators should install only approved extensions, review permissions, keep extensions current, and remove unused items. Unwanted toolbars and extensions can track activity, redirect searches, inject advertising, and degrade browser performance.
### Browser privacy data
#### Cookies, cache, and history
Cookies store small values that support sessions, preferences, security features, and measurement. Session cookies usually expire when the browsing session ends, while persistent cookies remain until their expiry date or deletion. Sites and advertising systems can also use cookies and related technologies for cross-site tracking, subject to browser controls and legal requirements.

A browser cache stores reusable copies of web resources to reduce loading time and network traffic. It usually improves performance, although stale or corrupted entries can occasionally cause display or storage problems. Cached resources, browsing history, download records, and saved form data can reveal activity to a person who gains access to the device or browser profile.
#### Limits of private browsing
Private browsing limits the records that the browser retains after the private session closes. Depending on the browser, it generally removes local history, form entries, and cookies created during the session.

Private browsing does not hide activity from websites, an internet service provider, an employer, a school, a network administrator, or malware on the device. Downloaded files and saved bookmarks can remain after the session ends.
### Messaging, social platforms, and unwanted software
#### Social engineering and unsafe links
Attackers use social networks and messaging services for phishing, impersonation, fraud, and malware delivery. They exploit familiarity, urgency, authority, threats, and compromised accounts to make harmful requests appear credible.

End-to-end encryption can protect message content between supported endpoints, but it cannot make a malicious attachment safe or confirm that the person controlling an account is trustworthy.
#### Safer handling of messages and files
People should avoid sharing sensitive information through unapproved messaging services or unmanaged devices. They should verify unexpected links, attachments, payment requests, and account-recovery messages through a separate, trusted channel, even when a familiar account sends them.
#### Adware and redirects
Adware displays unwanted advertising and may track activity, change browser settings, or redirect traffic. Warning signs include unknown extensions, a changed home page or search engine, persistent pop-ups, degraded performance, and unexplained redirects. Trusted software sources, careful installation choices, current software, browser protections, and endpoint scans reduce the risk.
### VPN fundamentals and IPsec
#### What a VPN protects
A virtual private network creates a protected tunnel between defined endpoints, such as a user device and an organisation's VPN gateway. It can prevent local network observers from reading traffic inside that tunnel. Protection ends at the VPN endpoint, so applications still need end-to-end protections such as HTTPS. The VPN operator can also gain visibility into connection metadata and traffic that lacks encryption beyond the gateway.

A VPN does not stop phishing, malicious websites, unsafe downloads, compromised endpoints, or abuse of an authenticated session.
#### Common VPN architectures
- A gateway-to-gateway, or site-to-site, VPN connects two networks through security gateways.
- A host-to-gateway, or remote-access, VPN connects an endpoint to an organisation's network or security service.
- A host-to-host VPN protects traffic directly between two endpoints.

Routers, firewalls, and dedicated VPN gateways can terminate many tunnels centrally. Endpoint clients support mobile and remote users. Organisations should choose an architecture according to trust boundaries, routing requirements, identity controls, device management, capacity, and recovery needs.
#### IPsec components and modes
IPsec protects IP traffic through a suite of protocols and services.

- Encapsulating Security Payload, or ESP, can provide confidentiality, integrity, data-origin authentication, and anti-replay protection.
- Authentication Header, or AH, provides integrity, data-origin authentication, and anti-replay protection but not confidentiality. Many modern deployments prefer ESP, and AH can conflict with network address translation.
- Internet Key Exchange, or IKE, authenticates peers and negotiates keys, algorithms, and security associations.

Tunnel mode encapsulates the complete original IP packet inside a new packet and commonly supports gateway-to-gateway and remote-access VPNs. Transport mode protects the original packet's payload while retaining its IP header and supports host-to-host deployments. The selected protocol, mode, algorithms, key management, and configuration determine the actual protection.
### Application ecosystem security
#### Mobile and desktop applications
Attackers compromise applications and their data through stolen credentials, malicious software, unsafe downloads, vulnerable dependencies, excessive permissions, insecure data storage, and design or implementation flaws. A compromised application may expose contacts, messages, files, location data, authentication tokens, and business information.

Official app stores reduce some distribution risks but do not guarantee safety. Users should review the developer, permissions, update history, and organisational approval before installation, then keep the application and operating system current.

Rooting or jailbreaking can bypass platform security controls and complicate reliable updates and device management. Most organisations prohibit these changes on devices that access organisational data.
#### Organisational controls and shared data
Organisations reduce application risk through secure design, patching, application control, endpoint and mobile-device management, staff education, monitoring, and tested response plans. They should give applications and users only the data and permissions required for authorised work.

Secure file sharing also needs usable, approved workflows. Excessive friction can encourage staff to move information into unapproved consumer services. Organisations should combine clear sharing rules, appropriate access, multi-factor authentication, data-loss controls, monitoring, and simple approved tools.