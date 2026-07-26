# Introduction to Cybersecurity Tools and Cyberattacks
> [!NOTE]
> Effective cyber security combines sound judgement, threat awareness, vulnerability management, incident response, identity controls, and physical safeguards. No single control can protect an organisation from every attack.
## Security Context and Analytical Judgement
### Lessons from major events
#### September 11 and US security policy
- The terrorist attacks of 11 September 2001 were not cyberattacks. The subsequent inquiry exposed failures in intelligence sharing and coordination, while digital records helped investigators reconstruct activity.
- The Homeland Security Act of 2002 established the US Department of Homeland Security by combining all or part of 22 federal departments and agencies. Cyber security became one part of a broader homeland security mission.
- The USA PATRIOT Act expanded investigative and surveillance authorities after the attacks. The USA FREEDOM Act of 2015 later prohibited bulk collection under several authorities, changed the handling of telephone metadata, and introduced additional oversight and transparency requirements.
- These policy changes affected intelligence, surveillance, information governance, and inter-agency coordination. They should not be presented as a single technical cyber security response.
#### COVID-19 and cyber risk at scale
- The COVID-19 pandemic accelerated remote work, cloud adoption, and dependence on online collaboration. Rapid deployment expanded the attack surface through remote access services, unmanaged devices, home networks, and hurried configuration changes.
- Criminal and state-linked actors used pandemic themes in phishing, malware delivery, domain impersonation, and attacks on remote access infrastructure.
- Organisations responded by strengthening remote access, multi-factor authentication, endpoint management, email security, cloud configuration, monitoring, and staff guidance.
- A ransomware attack disrupted Düsseldorf University Hospital in 2020 and forced it to divert emergency patients. Investigators later found insufficient evidence to attribute a patient's death directly to the attack. The incident still demonstrated how cyber disruption can endanger clinical operations.
#### WarGames and early US information security policy
- The 1983 film WarGames brought unauthorised access and automated military decision-making into public debate.
- National Security Decision Directive 145 followed in 1984. It assigned government-wide responsibilities for telecommunications and automated information systems security, designated the US Secretary of Defense as executive agent, and designated the Director of the National Security Agency as national manager.
- The film and the directive appeared during the same period of growing concern about computer security. The historical record does not support a simple claim that the film caused the directive.
### Critical thinking in cyber security
Critical thinking helps security teams distinguish evidence from assumption, compare competing explanations, and choose controls that reduce risk without creating unnecessary harm.

| Analytical dimension | Practical application |
| --- | --- |
| Evidence quality | Teams assess provenance, collection methods, completeness, timeliness, and corroboration before drawing conclusions. |
| Competing hypotheses | Analysts test plausible actors, motives, techniques, timelines, and benign explanations. |
| Technical knowledge | Practitioners apply knowledge of systems, networks, identity, malware, and security tools to interpret evidence accurately. |
| Organisational context | Teams relate technical findings to business services, legal duties, safety, privacy, and operational constraints. |
| Risk judgement | Decision-makers weigh likelihood, exposure, exploitability, impact, and the cost of treatment. |
| Communication | Analysts state facts, assumptions, confidence, limitations, and recommended action clearly. |

Strong analysis challenges assumptions, checks source quality, identifies the main risk drivers, and records uncertainty. It also avoids treating a vulnerability score, alert, or tool output as a conclusion on its own.
## Cybersecurity Threats
### Social engineering, phishing, and AI
#### Human and technical weaknesses
Social engineering manipulates trust, attention, authority, fear, curiosity, or reward. Attackers often combine this manipulation with technical weaknesses such as weak authentication, exposed services, insecure workflows, or unpatched software.

Common objectives include credential theft, account takeover, malware installation, data theft, fraudulent payments, and unauthorised access.
#### Generative AI and phishing
- Generative AI can increase the speed, language quality, personalisation, and volume of malicious messages.
- Public information about a target's role, employer, contacts, interests, and current activities can make generated lures more convincing.
- AI does not guarantee a successful attack. Reconnaissance, delivery infrastructure, timing, and the target's controls still influence results.
- Fluent grammar no longer provides a reliable way to distinguish legitimate messages from phishing. Training therefore focuses on context, requests, links, attachments, sender identity, and independent verification.
- Synthetic voice, video, and images can strengthen impersonation. Organisations rely on verified contact channels, transaction controls, and approval procedures rather than appearance or voice alone.
#### Open-source intelligence and targeting
Attackers can collect open-source intelligence from professional profiles, corporate websites, media releases, conference programs, procurement notices, job advertisements, social media, and public records. A convincing lure often refers to a real project, supplier, benefit, event, device refresh, or policy change.

Organisations reduce exposure by limiting unnecessary publication of staff details, internal processes, travel, technology, and reporting lines. They balance this control with legitimate transparency and communication needs.
#### Common phishing forms and delivery techniques
| Form or technique | Description |
| --- | --- |
| Phishing | Broad fraudulent messaging that seeks information, money, access, or execution of malicious content. |
| Spear phishing | Targeted messaging tailored to a person, role, team, or organisation. |
| Whaling | Spear phishing directed at executives or other high-privilege decision-makers. |
| Smishing | Phishing delivered through SMS or another text messaging service. |
| Vishing | Phishing conducted through voice calls or recorded messages. |
| Domain impersonation | Lookalike domains, subdomains, or sender names imitate a trusted organisation. |
| Search engine poisoning | Attackers manipulate search visibility so that users encounter malicious websites. It is a delivery technique rather than a distinct phishing channel. |

Phishing commonly directs a target to a credential-harvesting site or prompts the target to open an attachment, install software, authorise access, disclose data, or make a payment.
#### Physical and blended social engineering
- Tailgating occurs when an unauthorised person follows an authorised person through a controlled entrance.
- Shoulder surfing exposes passwords, PINs, access codes, or sensitive screen content to an observer.
- Dumpster diving recovers information or media from insecure waste.
- Impersonation can involve IT support, an executive, a supplier, a bank, or a government agency.
- Blended attacks combine email, messaging, phone calls, video, and physical access to create credibility and pressure.
#### Controls that remain effective
- Organisations verify unexpected requests for money, credentials, access, or confidential information through an independent channel.
- Staff use known directories, official portals, or verified supplier records rather than contact details supplied in the request.
- Approval workflows separate initiation, review, and authorisation for sensitive transactions.
- Users navigate to known services directly and treat unsolicited links, attachments, and downloads as untrusted.
- Reporting processes send suspicious messages to the security team without forwarding malicious content widely.
- Email security, DNS filtering, web filtering, endpoint protection, and browser controls reduce exposure but do not replace verification.
- Phishing-resistant authentication limits the value of stolen passwords.
- Awareness programs teach manipulation patterns, provide safe reporting, and avoid blaming people who report mistakes quickly.
### Malware and ransomware
#### Main malware categories
| Category | Typical behaviour |
| --- | --- |
| Virus | Attaches to files or other host content and spreads when that content runs. |
| Worm | Propagates between systems, often by exploiting a vulnerability or weak configuration. |
| Trojan | Presents itself as legitimate or desirable software while performing malicious activity. |
| Spyware and keylogger | Collects activity, credentials, communications, or other sensitive information. |
| Fileless malware | Uses memory, scripts, or legitimate system tools to reduce reliance on conventional malicious files. It can still leave logs and other forensic artefacts. |
| Bot malware | Enrols a compromised device into a remotely controlled botnet. |
| Rootkit | Conceals malicious activity or persistence at the user, kernel, boot, hypervisor, or firmware layer. |
| Ransomware | Encrypts systems or data, disrupts operations, steals information, or combines these actions for extortion. |
#### Ransomware impact and resilience
- Ransomware can cause operational disruption, data loss, financial loss, safety consequences, privacy breaches, regulatory exposure, and reputational damage.
- Modern extortion frequently combines encryption with data theft and threats to publish stolen information.
- Tested, isolated, and appropriately immutable backups support recovery from encryption or destruction. Backups do not prevent an attacker from disclosing data already stolen.
- Encryption at rest protects stolen storage only when the attacker cannot access decrypted content or obtain the keys.
- Effective prevention combines patching, multi-factor authentication, restricted administrative privileges, application control, endpoint detection, network segmentation, secure remote access, and monitoring.
- Response planning covers containment, legal and regulatory obligations, communications, evidence preservation, restoration, and lessons learned.
#### Rootkit checking
Host-checking tools such as rkhunter can compare Linux and Unix systems with known indicators, file properties, and configuration expectations. These tools can generate false positives and cannot prove that a system is clean. Teams combine them with secure boot controls, file integrity monitoring, endpoint telemetry, vulnerability management, log review, and forensic analysis.
### Insider risk
Insider risk arises when a current or former employee, contractor, partner, or other trusted person uses authorised access in a harmful way. Harm may be intentional, negligent, accidental, or enabled by coercion or account compromise.

| Insider scenario | Example |
| --- | --- |
| Accidental | A staff member sends sensitive information to the wrong recipient. |
| Negligent | A user bypasses a required control for convenience or ignores repeated security guidance. |
| Malicious | A trusted person steals data, commits fraud, sabotages systems, or assists an external actor. |
| Compromised | An external actor takes control of a trusted account or device and uses its legitimate access. |

Organisations reduce insider risk through least privilege, separation of duties, strong authentication, role-based training, data handling controls, behavioural and technical monitoring, prompt offboarding, and fair investigation procedures. Monitoring also respects privacy, employment law, proportionality, and due process.
### Threat actors and motives
| Actor category | Common motives and behaviour |
| --- | --- |
| Organised cybercrime | Financial gain through ransomware, extortion, fraud, credential theft, and illicit markets. |
| Nation-state or state-linked actor | Espionage, strategic influence, intellectual property theft, disruption, or preparation for conflict. |
| Hacktivist | Political or social influence through disruption, data release, defacement, or publicity. |
| Opportunistic or low-skill actor | Curiosity, status, grievance, or profit using readily available tools and exposed systems. |
| Insider | Personal gain, grievance, ideology, coercion, negligence, or error from within a trusted relationship. |

Actor analysis helps teams prioritise controls and response plans, but motive alone does not identify an attacker. Attribution requires corroborated technical, behavioural, operational, and intelligence evidence.
## Cybersecurity Operations, Controls, and Tools
### Vulnerability management
#### Continuous workflow
- Organisations maintain an accurate inventory of hardware, software, cloud services, identities, data stores, and externally exposed assets.
- Asset owners classify business criticality, data sensitivity, safety implications, exposure, and recovery requirements.
- Authorised scanning and testing cover the defined environment at a frequency suited to risk and operational constraints.
- Credentialed scans usually provide deeper visibility into configuration and patch state than unauthenticated scans.
- Teams validate findings, assign ownership, set treatment deadlines, and record accepted or transferred risk.
- Prioritisation considers active exploitation, internet exposure, exploitability, asset criticality, likely impact, and existing controls rather than severity scores alone.
- Remediation removes or patches the weakness, changes configuration, restricts exposure, or applies a compensating control.
- Verification confirms that treatment worked and did not introduce harmful side effects.

Scans require defined authorisation, scope, safety controls, and communication with system owners. High-availability, safety-critical, legacy, and operational technology environments may require specialised methods.
#### Common tools
| Tool or capability | Appropriate use and limitation |
| --- | --- |
| Nessus | A commercial vulnerability scanner that uses plugin-based checks. Results still require validation and risk context. |
| Greenbone Community Edition | An open-source vulnerability management stack. OpenVAS Scanner is one component of the stack, not the current name for the entire product. |
| Nmap | Discovers hosts, ports, services, and selected operating system characteristics within an authorised scope. |
| Wireshark | Captures and analyses network packets for troubleshooting, protocol analysis, and investigation on networks where capture is authorised. |
| rkhunter | Checks Linux and Unix hosts for selected rootkit indicators and anomalies. It supplements, rather than replaces, forensic and endpoint analysis. |
| SIEM | Centralises and correlates logs to support detection, investigation, reporting, and retention. |
| SOAR | Automates defined security workflows and response actions. Automation requires safeguards, approvals, and testing. |

Tool choice depends on coverage, accuracy, update frequency, scalability, deployment model, reporting, integrations, staff capability, and the environment's tolerance for active testing.
### Application security testing
Security teams integrate testing throughout planning, design, development, build, deployment, operation, and retirement.

| Method | Main purpose |
| --- | --- |
| Threat modelling | Identifies assets, trust boundaries, misuse cases, threats, and design controls before implementation. |
| Static application security testing | Analyses source code, bytecode, or binaries without exercising the running application. |
| Software composition analysis | Identifies third-party components, licences, known vulnerabilities, and dependency risk. |
| Dynamic application security testing | Tests a running application through exposed interfaces. |
| Interactive application security testing | Observes application behaviour from inside the running test environment while tests exercise the application. |
| Manual review and penetration testing | Examines business logic, chained weaknesses, and context that automated tools may miss. |

Release gates reflect risk and include a process for exceptions, ownership, expiry, and compensating controls. A passed scan does not establish that an application is secure.
### Incident response and digital forensics
#### Incident response as risk management
Current NIST guidance integrates incident response across cyber risk management rather than confining it to an emergency phase. Operational teams still organise work around preparation, detection, analysis, containment, eradication, recovery, communication, and improvement.

- Preparation defines roles, authority, escalation paths, communications, legal support, suppliers, tools, evidence procedures, backups, and rehearsed playbooks.
- Detection and analysis confirm whether an event is an incident, establish scope, assess impact, and set priorities.
- Containment limits harm while preserving critical services and evidence.
- Eradication removes malicious access, persistence, compromised credentials, and exploited weaknesses.
- Recovery restores services safely, increases monitoring, and verifies integrity.
- Post-incident work records lessons, assigns improvements, and tracks them to completion.
#### Digital forensics
Digital forensics identifies, collects, preserves, examines, analyses, and reports digital evidence. Sources can include storage, memory, network traffic, cloud services, identity systems, applications, mobile devices, and logs.

Teams preserve integrity through documented procedures, appropriate acquisition methods, cryptographic hashes, secure storage, access records, and chronological chain-of-custody documentation. Legal advice and applicable rules determine when evidence must meet judicial or regulatory requirements.
### Network threats and attacker techniques
#### Packet capture and sniffing
- Network packets contain protocol headers and data, although encryption may conceal application content.
- Authorised packet capture supports troubleshooting, performance analysis, protocol validation, and security investigation.
- Passive capture observes available traffic without changing its path.
- Active interception techniques, such as ARP spoofing on a local network, manipulate traffic flow to expose or alter communications.
- Encryption in transit, authenticated network access, secure Wi-Fi configuration, segmentation, switch protections, and monitoring reduce risk.
- HTTPS protects content in transit but does not hide every item of metadata or protect a compromised endpoint.
#### IP spoofing and denial of service
- IP spoofing forges a packet's source address. It is especially useful in one-way attacks and UDP reflection or amplification attacks.
- Source spoofing alone does not normally allow an attacker to receive replies or complete an ordinary two-way connection.
- Ingress and egress filtering reduce forged-source traffic. BCP 38 provides established guidance for network ingress filtering.
- A denial-of-service attack degrades availability through resource exhaustion, protocol abuse, application load, or a software flaw.
- A distributed denial-of-service attack uses multiple sources, often compromised devices, to increase scale and complicate filtering.
- Defences combine resilient architecture, capacity, rate limiting, traffic scrubbing, upstream coordination, filtering, hardening, monitoring, and rehearsed response.
#### Injection and browser-side attacks
- Injection occurs when a system interprets untrusted input as part of a command, expression, or query.
- SQL injection can expose, alter, or delete data, bypass application controls, and execute database operations with the application's privileges.
- Parameterised queries provide the primary defence against SQL injection. Input validation and least-privilege database accounts add protection.
- Cross-site scripting occurs when a browser executes attacker-controlled content in a trusted site context.
- Reflected, stored, and DOM-based cross-site scripting differ in how attacker-controlled content reaches an execution sink.
- Context-sensitive output encoding, safe templating, HTML sanitisation where required, and safe browser APIs provide primary cross-site scripting defences. A carefully configured Content Security Policy adds defence in depth.
### Control categories and functions
Administrative, physical, and technical describe broad ways to implement controls. They are not the same as the formal control families in frameworks such as NIST SP 800-53.

| Implementation category | Examples |
| --- | --- |
| Administrative | Governance, policy, risk assessment, training, supplier management, and separation of duties. |
| Physical | Locks, barriers, guards, surveillance, environmental monitoring, and secure disposal. |
| Technical | Access controls, encryption, firewalls, endpoint protection, logging, and automated response. |

| Control function | Purpose | Examples |
| --- | --- | --- |
| Preventive | Stops or reduces the likelihood of an incident. | Multi-factor authentication, patching, segmentation, and application control. |
| Detective | Identifies events or control failures. | Auditing, intrusion detection, SIEM alerts, and video surveillance. |
| Deterrent | Discourages unsafe or malicious behaviour. | Warning notices, visible guards, lighting, and sanctions. |
| Corrective | Limits damage and restores an acceptable state. | Quarantine, credential reset, patching, restoration, and incident response. |
### Layered system, network, and application security
#### System security
- Strong authentication and least privilege limit access to systems and data.
- Encryption protects data at rest and in transit when organisations secure the endpoints and keys.
- Patch and configuration management reduce exposure across operating systems, applications, firmware, and virtual infrastructure.
- Tested backups support recovery objectives and protect against accidental loss, hardware failure, and ransomware.
- Host firewalls, endpoint protection, application control, and central logging provide protection when perimeter controls fail.
#### Network security
- Firewalls, secure routing, network access control, segmentation, intrusion detection and prevention, and protected remote access regulate connectivity.
- Segmentation limits lateral movement and separates environments with different trust, safety, or compliance requirements.
- SIEM improves visibility by centralising events, while SOAR can automate approved enrichment, containment, and notification workflows.
- Defence in depth assumes that any individual control can fail and places independent safeguards around critical services.
#### Application security
- Secure design defines trust boundaries, access rules, abuse cases, data protection, failure behaviour, and logging before coding begins.
- Developers use parameterised queries, safe output handling, reviewed cryptographic libraries, secure secret storage, and explicit authorisation checks.
- Build and deployment pipelines protect source code, dependencies, artefacts, credentials, and signing processes.
- Testing combines automated analysis with manual review and targeted testing based on risk.
#### Network mapping
- Network mapping records physical and logical connections, assets, interfaces, routes, services, trust boundaries, and data flows.
- Defenders use accurate maps for troubleshooting, change control, capacity planning, vulnerability management, segmentation, and incident response.
- Attackers conduct reconnaissance to identify exposed services, high-value assets, and possible paths through an environment.
- Nmap supports authorised discovery and service identification. Wireshark supports packet-level inspection and protocol analysis.
## Identity and Physical Controls
### Identity, authentication, and authorisation
#### The four As and AAA
A practical four-A model uses administration, authentication, authorisation, and audit. It differs from the established AAA model of authentication, authorisation, and accounting.

| Function | Purpose |
| --- | --- |
| Administration | Creates, changes, reviews, suspends, and removes identities and access. |
| Authentication | Verifies control of one or more authenticators associated with an identity. |
| Authorisation | Determines which actions an authenticated identity may perform. |
| Audit | Records and reviews identity, access, and administrative activity. |
#### Authentication protocols
| Protocol or framework | Role and limitation |
| --- | --- |
| RADIUS | Carries authentication and authorisation information between network access equipment and a shared server. Companion specifications provide accounting. Deployments commonly support Wi-Fi, VPN, and other network access. |
| CHAP | Uses a challenge-response exchange instead of sending a reusable password in cleartext. Its legacy design does not provide modern session protection and should not serve as a strong standalone control. |
| EAP | Provides a framework for multiple authentication methods. Security depends on the selected method, server validation, certificate handling, and configuration. |
| EAP-TLS | Uses certificates within EAP and can provide strong mutual authentication when organisations validate and protect certificates correctly. |
| Kerberos | Uses a trusted key distribution centre and ticket-based authentication. It supports centralised authentication but depends on protected keys, accurate time, secure configuration, and resilient infrastructure. |
#### Identity systems and credential stores
- Microsoft Active Directory provides directory, domain, policy, and authentication services. It is not simply an authentication server.
- LDAP is a protocol for accessing and managing directory information. It is not a credential store by itself.
- A RADIUS server may validate credentials directly or delegate authentication to a directory, identity provider, certificate service, or another system.
- Centralised identity services simplify administration and visibility but create high-value infrastructure that requires hardening, tiered administration, monitoring, backup, and recovery.
### Authentication methods and strength
- Passwords remain vulnerable to phishing, reuse, guessing, malware, and offline cracking after credential database theft.
- Multi-factor authentication requires authenticators from different factor categories. Repeating two checks from the same category does not automatically create multi-factor authentication.
- SMS and voice codes reduce password-only risk but remain exposed to phishing, number transfer fraud, interception, and recovery process weaknesses.
- Time-based one-time passwords from authenticator apps resist password reuse and some replay attacks, but users can still enter them into phishing sites. They are not phishing-resistant.
- Push approval can improve convenience but needs number matching or equivalent context, rate limits, anomaly detection, and protection against approval fatigue.
- FIDO security keys and correctly implemented passkeys use public-key cryptography and origin binding to provide phishing-resistant authentication.
- Biometrics usually unlock or activate a device-bound authenticator. Organisations protect biometric data because a person cannot replace a compromised biometric characteristic as easily as a password.
- Account recovery receives the same scrutiny as sign-in because a weak recovery path can bypass strong authentication.
### Access control models and permissions
| Model | How it assigns access |
| --- | --- |
| Role-based access control | Assigns permissions to roles aligned with job functions. |
| Attribute-based access control | Evaluates attributes of the identity, resource, action, and context. |
| Rule-based access control | Applies system-defined rules or access control lists, such as network filtering rules. |
| Mandatory access control | Enforces centrally defined labels and rules that ordinary users cannot change. |
| Discretionary access control | Allows an object owner to grant or revoke access within system policy. |

Least privilege limits access by role, task, time, and context. Organisations review entitlements, remove access when roles change, separate privileged accounts from routine activity, and monitor sensitive actions.

Linux and Unix systems commonly express file permissions through owner, group, and other rights, supplemented by access control lists and security modules. Windows commonly uses security descriptors and access control lists. On both platforms, effective access can also depend on inheritance, privileges, application policy, and the execution context.

Logs, digital signatures, trusted timestamps, protected audit trails, and sound identity proofing support accountability. Technology can strengthen evidence that an action occurred, but it does not guarantee legal non-repudiation in every context.
### Single sign-on, password managers, and passkeys
- Single sign-on allows an identity provider or central session service to provide access to multiple relying services.
- Single sign-on reduces repeated password entry and can centralise policy, but compromise of the central identity can affect many services. Strong authentication, conditional access, session controls, monitoring, and resilient recovery reduce this concentration risk.
- A password manager generates, stores, and fills unique passwords. It improves password hygiene and can reduce exposure to lookalike sites when autofill checks the origin, but it does not eliminate endpoint malware, social engineering, or recovery risk.
- Passkeys use a cryptographic key pair. The service stores a public key, while an authenticator protects the private key locally or through a protected synchronisation service.
- Passkey authentication signs a challenge for the legitimate service without sending a reusable password. Implementation quality, device security, synchronisation, lifecycle management, and account recovery still require control.
### Physical threats and controls
#### Common threats
Physical threats include unauthorised entry, theft, device tampering, tailgating, shoulder surfing, insecure disposal, surveillance, vandalism, fire, water, power loss, temperature extremes, humidity, and electrostatic discharge.

Physical access can enable data theft, credential capture, malicious device connection, firmware or hardware tampering, service disruption, and bypass of remote security controls.
#### Layered physical protection
| Area | Controls |
| --- | --- |
| Site perimeter | Fencing, lighting, cameras, alarms, barriers, signage, and controlled entrances. |
| Building entry | Reception, guards, identity checks, visitor records, badges, and access control vestibules. |
| Sensitive rooms | Restricted access, entry logging, surveillance, rack locks, tamper detection, and appropriate fire suppression. |
| Equipment | Cable locks, port controls, full-disk encryption, secure boot, asset tracking, and protected maintenance interfaces. |
| Environment | Climate control, water detection, surge protection, grounding, anti-static measures, backup power, and maintenance. |
| Media and waste | Inventory, locked storage, approved sanitisation, shredding, and documented destruction. |

Business continuity planning aligns redundant systems, geographically separated recovery capability, uninterruptible power supplies, generators, backups, communications, and supplier arrangements with recovery objectives.

High-risk sites may add drone detection, intelligent perimeter sensors, or robotic patrol systems where law, safety, privacy, and operating conditions permit. AI-assisted monitoring can help staff identify anomalies, but trained personnel validate alerts and retain decision authority.
## Integrated Security Priorities
- Organisations identify critical services, assets, identities, data, dependencies, and recovery requirements.
- Leaders assign ownership and fund controls according to risk, legal duties, safety, and business impact.
- Security teams reduce common attack paths through patching, secure configuration, application control, restricted privileges, multi-factor authentication, and resilient backups.
- Identity teams prefer phishing-resistant authentication for high-risk and privileged access.
- Detection teams centralise useful telemetry, tune alerts, test response actions, and preserve evidence appropriately.
- Incident responders rehearse technical, legal, executive, supplier, and communication decisions before a crisis.
- Physical and cyber teams coordinate access control, monitoring, asset protection, and continuity planning.
- Governance processes test control effectiveness, track exceptions, learn from incidents, and adapt to changes in technology and threats.