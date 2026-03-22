# Penetration Testing, Threat Hunting, and Cryptography

> [!NOTE]
> This document surveys three core security disciplines. It explains penetration testing as an authorised, scoped simulation of real attacks, covering planning, reconnaissance, exploitation, evidence gathering, and reporting, and clarifying how it differs from automated vulnerability scanning. It then introduces threat hunting and threat intelligence, including how telemetry, hypotheses, and structured frameworks improve detection and response. Finally, it covers cryptography fundamentals of confidentiality, integrity, hashing, digital signatures, symmetric vs. asymmetric encryption, key management pitfalls, and the shift toward post-quantum approaches.
## Penetration Testing: Planning and Discovery Phases
Penetration testing, often called pen testing, is a structured security assessment in which authorised testers simulate real attacks against systems, applications, and people to identify weaknesses and demonstrate how those weaknesses could be exploited. It is one technique within ethical hacking, which also covers activities such as malware analysis and risk assessment. Pen tests are commonly delivered by independent specialists to provide an external perspective that may surface issues missed by in house teams.

Pen testing differs from vulnerability scanning. Scanning is usually automated and repeatable, and it flags known weaknesses for review. Pen testing uses a mix of automated and manual methods, then attempts exploitation within an agreed scope to confirm what is truly reachable and what impact an attacker could achieve. This reduces false positives and produces evidence that supports practical remediation.
### Why organisations use pen testing
Organisations commission pen tests to:
- Understand how attackers could access sensitive data or disrupt operations.
- Validate whether existing security controls work as intended under realistic conditions.
- Supplement vulnerability assessments with exploitation focused findings.
- Demonstrate compliance with security obligations and standards where testing is required or recommended.

Common compliance drivers include health and privacy requirements such as HIPAA and GDPR, payments requirements such as PCI DSS, and voluntary security management standards such as ISO/IEC 27001.
### Types of pen tests
Pen tests can focus on different assets and threat paths:
- Application testing targets web, mobile, cloud, and API surfaces. Testing often starts with issues in the OWASP Top 10, then explores system specific weaknesses such as logic flaws, misconfiguration, and authentication failures.
- Network testing assesses internet facing and internal environments. External tests simulate attackers on the internet. Internal tests simulate a malicious insider or an attacker using stolen credentials.
- Hardware testing targets endpoints and connected devices, including IoT and operational technology, and considers both software flaws and physical security gaps.
- Personnel testing evaluates susceptibility to social engineering such as phishing, voice phishing, SMS phishing, and attempts to bypass physical access controls.
### Common misconceptions and the facts
- Pen testing is not the same as vulnerability scanning because it validates exploitability and impact rather than only detection.
- One-off testing is insufficient because environments change and new weaknesses appear, so testing needs to be repeated.
- Smaller organisations still face material risk, and the cost of a breach can exceed the cost of testing.
- Testers do not need to exploit every weakness, and they prioritise issues that create the highest risk.
- Well planned testing usually avoids major disruption by using agreed windows, safeguards, and stop conditions.
- Automation alone is not enough because human judgement is needed to chain weaknesses and identify context specific risks.
- Testing approaches vary by tester and scope, which can reveal different classes of weakness.
### Pen testing phases
A pen test is typically delivered in five phases, with scope and safety controls agreed in advance:
- Planning defines scope, timing, and rules of engagement, including whether the test is black box, white box, or grey box.
- Discovery gathers intelligence and identifies likely weaknesses using tools, logs, traffic inspection, and open source intelligence.
- Attack attempts exploitation within scope, including privilege escalation and lateral movement where justified. Combining multiple weaknesses to reach a higher value target is often called vulnerability chaining.
- Verification confirms impact, documents what worked, and assesses how controls responded.
- Cleanup and reporting remove any artefacts introduced during testing and provide a report describing findings, evidence, and remediation actions.

Key planning considerations include clear scope specification, resourcing, risk management, communications, and legal and compliance requirements. Rules of engagement commonly cover scope boundaries, allowed methods, testing windows, authorised personnel, communications, incident handling, and post test procedures.
### Discovery and reconnaissance
Reconnaissance is the information gathering activity that supports discovery. Passive reconnaissance uses publicly available information without directly probing the target, such as websites, domain registration data, social media, public records, job advertisements, and online discussion forums.

Active reconnaissance directly interacts with systems and is more likely to be detected, such as host and port discovery, service enumeration, traceroutes, banner collection, packet capture and analysis, vulnerability scanning, and domain name system enumeration.

Social engineering and physical techniques may also be assessed when in scope, including phishing pretexts, phone based impersonation, and attempts to gain unauthorised entry. Physical information may be recovered from discarded materials, which is often dumpster diving.
### Google dorking in authorised testing
Google dorking, also called Google hacking, is an advanced search approach that uses search operators to locate exposed files, misconfigurations, and other publicly indexed information relevant to an organisation’s security posture. It is appropriate only with proper authorisation and within legal and policy constraints.

Common operators used to narrow searches include:
- site: to limit results to a domain
- filetype: or ext: to find specific file formats
- intitle: and inurl: to target page titles and URLs
- before: and after: to constrain results by date
- quotes, minus, and wildcard characters to refine matching

Used responsibly, these techniques can help teams identify unintended exposure and prioritise fixes before attackers exploit the same information.
## Penetration Testing: Attack Phase
The attack phase follows planning and discovery and involves authorised attempts to exploit identified weaknesses in a target environment. Activities are performed under an agreed scope to test controls and quantify risk. Typical outcomes include evidence of impact, notes on how access was obtained, and inputs to a vulnerability report.

Key actions often include:
- Running exploits with frameworks and tooling such as Metasploit, Browser Exploitation Framework (BeEF), and SQLMap
- Bypassing defensive controls and access barriers using approved techniques, including network evasion methods and physical security testing such as lock manipulation or RFID credential cloning
- Escalating privileges from user access to administrator or system level access where possible
- Completing post-attack activities, including identifying what data or systems were exposed, how compromise could spread through the environment, and what persistence paths exist within the authorised test

Common vulnerability types include misconfigurations, kernel flaws, weak input validation, symbolic link issues, file descriptor errors, race conditions, buffer overflows, and incorrect file or directory permissions.
### Penetration testing tools
Pen testing tools support controlled, repeatable testing across several broad categories:
- Network scanning and vulnerability discovery: Nmap, Nessus, IBM QRadar
- Exploitation and adversary simulation: Metasploit, IBM X-Force Red
- Web application testing: Burp Suite, OWASP ZAP
- Wireless security testing: Aircrack-ng, Kismet
- Password strength and recovery testing: John the Ripper, Hashcat
- Traffic capture and analysis: Wireshark, tcpdump
- Social engineering simulation and OSINT: Social-Engineer Toolkit (SET), Maltego
### Port scanning concepts
A network port is a logical endpoint used to direct traffic to a service on a host. Port scanning checks whether ports are open, closed, or filtered by controls such as firewalls. Port ranges are commonly:
- 0 to 1023: well-known ports assigned by IANA for widely used services
- 1024 to 49151: registered ports for many applications
- 49152 to 65535: dynamic or private ports, often used for ephemeral connections

Common scan types include ICMP discovery scans, TCP SYN scans, TCP connect scans, UDP scans, and low-and-slow scanning approaches intended to reduce detection.
### Network protocol analysers and packet capture
Network protocol analysers capture and decode traffic so administrators and testers can troubleshoot performance and review security controls. Wireshark is a widely used analyser that inspects many protocols, supports filtering, and can work with live captures or offline files via its GUI or TShark.

Packet captures are commonly stored as pcap or pcapng files, with pcapng providing richer metadata such as multiple interfaces. On Windows, packet capture often relies on the Npcap driver, while Unix-like systems commonly use libpcap based capture tools.
## Penetration Testing: Reporting Phase
Organisations use authorised penetration testing and secure development practices to reduce application risk. The module covers application testing methods, common weaknesses, exploitation activities, security tooling, repository scanning, reporting, and PTES.
### Application penetration testing
Application penetration testing simulates cyberattacks to identify vulnerabilities before adversaries exploit them. A security focused approach includes:
- Finding weaknesses through assessment and testing
- Reviewing architecture, code, configuration, and deployment
- Meeting required compliance controls
- Limiting unauthorised access with multifactor authentication, encryption, patching, training, and least privilege

Common testing approaches include SAST, DAST, IAST, MAPT, and WAPT.
### Common application weaknesses
Typical issues and controls include:
- Injection flaws mitigated through input validation and parameterised queries
- Cross-site scripting mitigated through validation, output encoding, and content security policy headers
- Broken authentication mitigated through secure credential and session handling plus multifactor authentication
- Sensitive data exposure mitigated through encryption in transit and at rest, plus strong data handling
- Security misconfiguration mitigated through hardened defaults and routine audits
### Attack phase
The attack phase follows planning and discovery and tests real impact within the agreed scope. Activities may include:
- Exploit execution using frameworks such as Metasploit, BeEF, and SQLMap
- Defence bypass attempts, including controlled physical testing such as lockpicking and RFID credential cloning
- Privilege escalation and post-exploitation checks for data exposure and lateral movement

Common weakness classes referenced include misconfigurations, kernel flaws, insufficient input validation, symbolic link flaws, file descriptor errors, race conditions, buffer overflows, and incorrect file or directory permissions.
### Tools, scanning, and traffic analysis
Pen testing tools are grouped by function, including network scanning, exploitation, web testing, wireless testing, password testing, traffic analysis, and social engineering.

Port scanning checks whether services are reachable and whether ports are open, closed, or filtered by firewalls. Common ranges include 0 to 1023 for well known services, 1024 to 49151 for registered ports, and 49152 to 65535 for dynamic ports. Common scan types include ICMP discovery, TCP SYN scans, TCP connect scans, UDP scans, and low and slow scanning to reduce detection.

Network protocol analysers capture and decode packets to troubleshoot performance and assess security. Wireshark and TShark support filtering and offline analysis. Captures are commonly stored as pcap or pcapng. Npcap is often used on Windows, while libpcap underpins capture tooling on Unix-like systems.
### Repository scanning and GitHub workflow
A code repository is a managed store for source code and related artefacts used for version control and collaboration. Scanning repositories supports a shift left approach by providing fast feedback and automation inside developer workflows. Common options include Snyk, IDE scanning plugins, and OWASP Dependency-Check, supported by regular review of scan results.

Workflow concepts covered include creating repositories, committing changes, using pull requests for review, and forking public repositories to modify a copy without changing the upstream project. Git is a distributed version control system, while GitHub provides a hosted platform and web interface.
### Reporting and PTES
A penetration testing report communicates findings, evidence, and remediation to stakeholders with different needs. A typical structure includes an executive summary, methodology and scope, findings, recommendations, conclusion, and appendix. Reports are handled as sensitive material through encryption, restricted access, defined retention, and secure disposal.

PTES is a framework to standardise engagement quality and communication. It describes pre-engagement interactions, intelligence gathering, threat modelling, vulnerability analysis, exploitation, post-exploitation, and reporting.
## Threat Hunting and Threat Intelligence
This summary describes how organisations use threat hunting, threat intelligence, SIEM, behaviour analytics, and common frameworks to reduce cyber risk.
### Threat hunting
Threat hunting is a proactive search for malicious activity that has bypassed routine controls. It is conducted within an agreed scope and produces evidence, impact assessment, and improvements to detections and controls.

Threat hunting relies on:
- Security telemetry from endpoints, identities, networks, cloud services, and applications
- Analysts who form and test hypotheses and validate results

Hunts commonly start from triggers such as unusual logins, unexpected data movement, or anomalous traffic.

Hunting types
- Structured hunting maps activity to indicators of attack and adversary tactics, techniques, and procedures (TTPs), often using MITRE ATT&CK as a reference.
- Unstructured hunting begins from at least one indicator of compromise and expands to determine entry point, scope, and follow on activity.
- Situational hunting is driven by local risk assessment, business change, or emerging threat trends.

Operating models
- Intelligence led hunting searches for activity linked to known indicators and verifies whether compromise occurred.
- Hypothesis led hunting uses playbooks aligned to adversary behaviour to uncover stealthy activity.
- Custom hunting blends both approaches to match business context and current risk.
### Threat intelligence
Threat intelligence is analysed information about threat actors, their methods, and exploitable weaknesses. It provides context for security decisions and helps prioritise what to defend and monitor.

Threat intelligence is often upstream of threat hunting:
- Threat intelligence gathers and assesses information about the wider threat landscape
- Threat hunting applies that context inside an environment through targeted investigation

Common benefits include better prioritisation, faster detection and response, more effective control hardening, and stronger risk management.

Threat intelligence categories
- Tactical: near term attacker tactics and common attack patterns
- Operational: campaign and TTP detail
- Strategic: long term trends and business impact drivers
- Technical: machine usable indicators such as malicious domains, IPs, and file hashes
### Threat intelligence lifecycle and sources
A common lifecycle converts raw data into action:
1. Planning and direction
2. Collection
3. Processing
4. Analysis
5. Dissemination and integration
6. Feedback
7. Action

Common source categories
- Open source intelligence from public reporting and communities, requiring validation for accuracy and recency
- Commercial intelligence from vendors and platforms, often curated and timely
- Closed source intelligence shared within trusted groups or via government and law enforcement channels, with handling constraints
- Internal intelligence from organisational logs, incidents, and lessons learned
- Human intelligence from analysts and subject matter experts, providing interpretation and judgement
### SIEM and intelligence sharing
Security information and event management (SIEM) platforms centralise security logging and event analysis. Core functions include log collection and normalisation, event correlation, alerting and dashboards, investigation workflows, and compliance reporting.

SIEM commonly integrates with:
- Endpoint detection and response and other security tooling
- SOAR for automated response workflows
- Threat intelligence feeds and sharing standards, including STIX (Structured Threat Information Expression) and TAXII (Trusted Automated Exchange of Intelligence Information)
### AI and behaviour analytics
AI and machine learning can help identify anomalies, correlate weak signals, and summarise large volumes of structured and unstructured data. Effective use depends on strong data governance, quality inputs, and human oversight, particularly due to bias, false positives, and adversary adaptation.

User behaviour analytics (UBA) focuses on end user interactions. User and entity behaviour analytics (UEBA) extends analysis to entities such as servers, network devices, and IoT systems. UEBA typically assigns risk scores to unusual activity to support triage.

Tactical UEBA use cases
- Malicious or compromised insiders
- Compromised devices, including IoT systems
- Potential data exfiltration

Strategic UEBA use cases
- Support for zero trust programs through continuous verification signals
- Evidence for data access governance and privacy compliance obligations
### CTI frameworks and applied models
A cyber threat intelligence framework sets a structure for data collection, analysis, sharing, response planning, and continuous improvement.

Cyber Kill Chain
1. Reconnaissance
2. Weaponisation
3. Delivery
4. Exploitation
5. Installation
6. Command and control
7. Actions on objectives

Diamond Model of Intrusion Analysis elements
- Adversary
- Infrastructure
- Capability
- Victim

Relationships between elements can guide investigation and mitigation:
- Adversary to victim
- Adversary to infrastructure
- Victim to infrastructure
- Victim to capability

Using MITRE ATT&CK and the Diamond Model
- ATT&CK provides matrices for Enterprise, Mobile, and ICS and supports mapping behaviours to detection and response improvements.
- For phishing, mapping techniques across stages supports controls such as email filtering, endpoint monitoring, and credential protection.
- For ransomware, the Diamond Model supports analysis of the threat actor, delivery and control infrastructure, ransomware capability, and victim weaknesses, informing actions such as stronger authentication, patching and configuration hygiene, segmentation, backups, and detection aligned to observed techniques.
## Cryptography: Principles and Techniques
### Cryptography in cybersecurity
Cryptography is a set of techniques that protects information by transforming it into a form that only authorised parties can use. It supports secure storage and secure communication across networks by combining encryption, decryption, hashing, and digital signatures.
#### Security goals
Cryptography mainly supports two goals.

- Data confidentiality keeps information private so only authorised users can access it.
- Data integrity keeps information accurate and detects unauthorised change during storage or transmission.

Many sectors treat confidentiality and integrity as legal, ethical, and operational requirements. Common compliance drivers include GDPR, HIPAA, and PCI DSS.
#### Core mechanisms
Encryption converts readable plaintext into ciphertext so that intercepted data remains unintelligible without the correct key. Decryption reverses that process for authorised recipients.

Hashing converts data into a fixed-size hash value. Hashing is one-way, so the original data cannot be recovered from the hash. Small input changes create a different hash, which makes hashing useful for integrity checks and password storage.

Digital signatures combine hashing with public key cryptography to provide integrity and authenticity. A sender signs a message by hashing it and then applying the sender’s private key to the hash. A recipient checks the signature with the sender’s public key and confirms the message has not changed.
#### Symmetric and asymmetric encryption
Encryption systems are often described by how keys are used.

- Symmetric encryption uses one shared secret key for encryption and decryption. It is typically fast and well suited to bulk data encryption.
- Asymmetric encryption uses a key pair. A public key encrypts or verifies, and a private key decrypts or signs. It is typically slower and is commonly used for key exchange and digital signatures.

In practice, modern systems use hybrid designs. Asymmetric cryptography secures the exchange of a short-lived symmetric session key, and symmetric cryptography encrypts the bulk data.
#### Common algorithms and protocols
Several widely used components appear throughout modern security systems.

- AES is a symmetric block cipher commonly used for files, disks, Wi-Fi, and VPN traffic. It operates on 128-bit blocks and commonly uses 128, 192, or 256-bit keys.
- RSA is an asymmetric algorithm based on the difficulty of factoring large composite numbers. It is widely used for digital signatures and for establishing secure channels, usually with key sizes such as 2048 bits or larger.
- TLS is a protocol that uses cryptography to protect data in transit, such as web browsing and email. It relies on certificates and strong cipher suites, and it is designed to prevent eavesdropping and tampering.
### Key management and implementation risks
Strong algorithms do not compensate for weak operational practices. Key management includes generating, storing, distributing, rotating, and revoking keys. Weak randomness, insecure storage, or poor distribution can undermine otherwise strong encryption.

SSL and TLS deployments can fail due to outdated protocol versions, weak cipher suites, and compromised certificate authorities. Practical mitigations include disabling legacy SSL, preferring modern TLS versions, selecting strong ciphers such as AES-GCM, and monitoring certificate ecosystems through transparency logs and robust revocation processes.
### Common attacks and practical mitigations
Cryptographic systems face attacks that target algorithms, protocols, and real-world implementations.

- Brute force attacks attempt every key or password combination. Strong, long keys and strong passwords increase the search space and reduce feasibility.
- Dictionary attacks try common passwords first. Password complexity, rate limiting, salts, and multi-factor authentication reduce risk.
- Man in the middle attacks intercept or modify communications, often via insecure Wi-Fi or downgrade tricks. Proper TLS usage, certificate validation, and VPN use on untrusted networks are common safeguards.
- Known plaintext and chosen plaintext techniques use relationships between chosen or known plaintext and observed ciphertext to infer weaknesses. Randomisation, padding schemes, and robust protocols reduce exposure.
- Side-channel attacks infer secrets from timing, power use, or emissions. Hardware hardening and implementation countermeasures reduce leakage.
- Differential and linear cryptanalysis exploit structural weaknesses in some ciphers. Modern, well-reviewed algorithms help limit exposure.

Cryptanalysis also plays a positive role. It is used to test, validate, and improve algorithms and standards, and to uncover implementation flaws that can undermine secure designs.
### Evolution of cryptography
Cryptography has moved from simple substitution methods to complex, mathematically grounded systems.

- Ancient approaches used basic encodings and ciphers to conceal meaning.
- The Caesar cipher used fixed letter shifts for military messaging.
- Frequency analysis, associated with the scholar Al-Kindi, enabled systematic code breaking by analysing letter patterns.
- Polyalphabetic ciphers, including the Vigenère cipher, reduced obvious patterns by varying shifts with a keyword.
- During World War II, the Enigma machine mechanised complex encryption. Allied code breaking at Bletchley Park, including work by Alan Turing and others, produced major intelligence advantages.
- In the 1970s, public key cryptography, introduced by Whitfield Diffie and Martin Hellman, enabled secure communication without sharing a secret key in advance.
- RSA, developed by Ron Rivest, Adi Shamir, and Leonard Adleman, became one of the first practical public key systems.
- Modern internet security relies on combinations of symmetric encryption, public key cryptography, and authenticated protocols.
### Quantum-safe cryptography
Large-scale quantum computing could threaten common public key systems by solving certain mathematical problems far faster than classical computers. This creates a risk where adversaries collect encrypted data now and decrypt it later.

Quantum-safe, also called post-quantum, cryptography aims to replace vulnerable public key methods with alternatives designed to resist quantum attacks while retaining practical performance.

Quantum-safe approaches commonly discussed include lattice-based, hash-based, code-based, and multivariate schemes. Symmetric encryption such as AES-256 is generally considered more resilient, though operational security still matters.

NIST has run standardisation efforts for post-quantum algorithms, with leading candidates including CRYSTALS-Kyber for key establishment and CRYSTALS-Dilithium, Falcon, and SPHINCS+ for digital signatures.
### Practical learning and tools
Hands-on exercises can help learners connect theory to practice.

- Cryptographic tooling can generate key pairs, encrypt and decrypt messages, and demonstrate how key size influences security and performance.
- Lab workflows often include RSA key generation, encryption using the public key, and decryption using the private key.
- Demonstrations of legacy algorithms, such as DES, can illustrate why older designs are deprecated and how cryptanalysis techniques can recover plaintext under certain conditions.
### Key concepts summary
- Cryptography supports confidentiality, integrity, and authenticity for data at rest and data in transit.
- Encryption protects confidentiality, hashing supports integrity checks, and digital signatures provide integrity and authenticity.
- Symmetric encryption is efficient for bulk data, asymmetric encryption is useful for key exchange and signatures, and hybrid systems combine both.
- Secure outcomes depend on key management, modern protocol choices, and careful implementation.
- Security threats evolve, so standards, audits, and updates are essential, including planning for post-quantum transitions.