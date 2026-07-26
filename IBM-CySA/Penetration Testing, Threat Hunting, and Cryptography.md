# Penetration Testing, Threat Hunting, and Cryptography
> [!NOTE]
> Authorised penetration testing validates exploitable weaknesses, threat hunting searches for adversary activity that routine controls have missed, and cryptography protects data and communications. Each capability supports a broader security program and requires clear governance, skilled practitioners, and current controls.
## Penetration testing
### Purpose and scope
Penetration testing is an authorised security assessment in which testers simulate realistic attacks against systems, applications, devices, people, or facilities. Testers identify weaknesses, attempt agreed forms of exploitation, and demonstrate the access or business impact an attacker could achieve. Internal teams, independent specialists, or a combination of both can conduct the work.

Penetration testing differs from vulnerability scanning. A scanner checks systems against known signatures and configuration rules, then reports potential weaknesses. A penetration test combines automated discovery with manual analysis and controlled exploitation. It can validate whether a weakness is reachable, show how several weaknesses form an attack path, and produce evidence for remediation. It cannot prove that an environment has no vulnerabilities.

Organisations commission penetration tests to:
- assess realistic paths to sensitive data, critical systems, or operational disruption
- verify whether preventive, detective, and responsive controls work as intended
- prioritise remediation according to demonstrated likelihood and impact
- assess major changes, new applications, cloud deployments, or acquired environments
- meet contractual, regulatory, or assurance obligations where those obligations apply

Compliance requirements vary. PCI DSS includes penetration-testing requirements for organisations within its scope. Article 32 of the General Data Protection Regulation requires a process for regularly testing, assessing, and evaluating security measures, but it does not prescribe penetration testing in every case. The HIPAA Security Rule and ISO/IEC 27001 use risk-based requirements and do not impose one universal penetration-testing schedule on every organisation. Each organisation must interpret its obligations according to its jurisdiction, scope, contracts, and risk profile.
### Types of test
The target and threat model determine the test type:
- Application testing examines web applications, mobile applications, application programming interfaces, thick clients, and business logic.
- Network and infrastructure testing examines internet-facing services, internal networks, identity systems, cloud resources, and segmentation controls.
- Wireless testing examines access points, encryption, authentication, client isolation, and unauthorised devices.
- Hardware and embedded-system testing examines endpoints, internet-connected devices, operational technology, firmware, interfaces, and physical protections.
- Social-engineering and physical testing examines phishing, voice phishing, SMS phishing, tailgating, badge controls, and other human or facility risks when the rules of engagement expressly permit them.

Black-box testing gives the tester little or no internal information. White-box testing provides detailed knowledge, such as source code, architecture, credentials, or configurations. Grey-box testing provides partial knowledge or limited access. These labels describe information available to the tester, not the quality or depth of the assessment.
### Engagement lifecycle
Methodologies divide an engagement in different ways. NIST SP 800-115 groups penetration testing into planning, discovery, attack, and reporting. The Penetration Testing Execution Standard uses seven sections:
1. Pre-engagement interactions
2. Intelligence gathering
3. Threat modelling
4. Vulnerability analysis
5. Exploitation
6. Post-exploitation
7. Reporting

These models overlap rather than conflict. Discovery and attack often repeat as new access reveals new information. Cleanup, evidence handling, and retesting also require explicit attention even when a framework includes them within broader phases.
### Planning and rules of engagement
Planning establishes authority, safety, and a shared definition of success. A written scope should identify:
- target systems, applications, networks, accounts, locations, and third parties
- excluded assets and prohibited techniques
- test dates, time zones, maintenance windows, and rate limits
- authorised testers, client contacts, and escalation paths
- data-handling, privacy, evidence, retention, and disposal requirements
- acceptable exploitation, privilege escalation, persistence, social engineering, and physical activity
- stop conditions for instability, safety risk, unexpected sensitive data, or suspected real compromise
- incident-response procedures and rules for distinguishing test activity from hostile activity
- cleanup, reporting, remediation validation, and retest expectations

The organisation must obtain permission from every relevant system owner and service provider. Authorisation for one asset does not extend to connected assets, shared cloud services, suppliers, or personal accounts. Testers should use the least intrusive technique that can establish the required evidence and stop when further action would create unnecessary risk.
### Reconnaissance and discovery
Passive reconnaissance gathers information without directly probing the target's systems. Sources can include corporate websites, public records, certificate-transparency data, domain-registration data, code repositories, job advertisements, social media, and public breach reporting. Passive work can still create privacy, legal, and ethical risks, so the engagement scope must govern collection and use.

Active reconnaissance interacts with the target environment. Common activities include host discovery, port scanning, service enumeration, banner collection, DNS enumeration, authenticated configuration review, and vulnerability scanning. Active techniques can trigger alerts, affect fragile systems, or expose the tester to data outside the intended scope.

Search operators can help authorised testers locate publicly indexed exposure. Common operators include:
- `site:` to restrict results to a domain
- `filetype:` or `ext:` to locate particular file types
- `intitle:` and `inurl:` to target page titles or addresses
- `before:` and `after:` to limit results by date
- quotation marks and the minus sign to refine terms

Search-engine indexing changes over time, and an indexed result does not prove that access is authorised. Testers should record the minimum evidence needed, avoid downloading unnecessary sensitive data, and report exposed content through the agreed channel.
### Exploitation and post-exploitation
The attack phase tests whether identified weaknesses can produce an agreed security impact. Activities can include exploit execution, authentication bypass, privilege escalation, lateral movement, segmentation testing, and controlled access to representative data. Testers sometimes combine several weaknesses into an attack chain that reaches a higher-value objective.

Post-exploitation establishes the significance of the access without causing avoidable harm. Testers may identify affected trust relationships, reachable systems, exposed records, available privileges, and control failures. They should not copy large data sets, establish long-lived persistence, alter production data, or disrupt services unless the scope specifically requires and safeguards those actions.

Physical techniques, credential cloning, lock manipulation, phishing, and defence-evasion tests require explicit approval. The rules of engagement should set close supervision, narrow targets, immediate escalation, and clear safety limits for these higher-risk activities.
### Tools and their roles
Tools support testing, but tester judgement determines scope, validation, and impact. Common categories include:
- Network discovery and scanning: Nmap, Nessus, and Greenbone/OpenVAS
- Web application testing: Burp Suite and OWASP ZAP
- Exploitation and adversary simulation: Metasploit and purpose-built test scripts
- Wireless testing: Aircrack-ng and Kismet
- Password auditing: Hashcat and John the Ripper
- Traffic capture and analysis: Wireshark, TShark, and tcpdump
- Social-engineering simulation and open-source intelligence: Social-Engineer Toolkit and Maltego

Security information and event management platforms such as IBM QRadar can supply logs, correlation, and defensive evidence during a test, but they do not replace a vulnerability scanner. IBM X-Force Red is a security-testing service, not an individual exploitation tool.
### Port scanning and packet analysis
A transport-layer port identifies a service endpoint for protocols such as TCP and UDP. Port scanning determines whether a target responds as open, closed, or filtered. Host-discovery techniques, including some uses of ICMP, identify reachable systems but do not scan ports by themselves.

IANA divides port numbers into three ranges:
- 0-1023: System Ports, with port 0 reserved
- 1024-49151: User Ports
- 49152-65535: Dynamic or Private Ports

Common techniques include TCP SYN scans, TCP connect scans, UDP scans, version detection, and carefully rate-limited scans. A registered port suggests an expected service, but it does not prove which application generated the traffic or whether that traffic is benign.

Network protocol analysers capture and decode traffic for troubleshooting, control validation, and investigation. Wireshark provides a graphical interface, while TShark provides a command-line interface. Wireshark supports both pcap and pcapng files. Pcapng can preserve richer metadata and captures from multiple interfaces. Wireshark commonly uses Npcap on Windows and libpcap on Unix-compatible systems. Packet capture requires appropriate authority because captures can contain credentials, personal information, session tokens, and business data.
### Application and repository security
Application penetration testing examines deployed behaviour and attack paths. It complements, but does not replace, other application-security techniques:
- Static application security testing analyses source code or compiled artefacts without running the application.
- Dynamic application security testing examines a running application from the outside.
- Interactive application security testing observes a running application through instrumentation while tests exercise it.
- Software composition analysis identifies known risks in third-party components and dependencies.
- Manual code review and threat modelling identify context-specific design, trust, and logic weaknesses.

Application tests should cover relevant OWASP guidance and system-specific risks. Common weaknesses and controls include:
- Broken access control, addressed through server-side authorisation, least privilege, and deny-by-default rules
- Injection, addressed through parameterised interfaces, safe APIs, validation, and context-aware output handling
- Cross-site scripting, addressed through output encoding, safe templating, sanitisation where required, and content security policy
- Authentication and session failures, addressed through strong authentication, secure recovery, protected session handling, and multi-factor authentication
- Cryptographic failures, addressed through approved algorithms, sound key management, and protection for data in transit and at rest
- Security misconfiguration, addressed through hardened baselines, minimal services, secret management, and regular review

Repository scanning moves feedback earlier in the development lifecycle. Secret scanning, static analysis, dependency analysis, infrastructure-as-code checks, and review gates can identify issues before deployment. Pull requests support peer review, while protected branches and required checks help enforce controls. A fork creates a separate repository lineage and does not change the upstream repository unless maintainers accept a contribution.
### Reporting, cleanup, and retesting
A useful report gives decision-makers and technical teams the evidence they need. It normally contains:
- an executive summary of objectives, major risks, and business impact
- scope, dates, assumptions, exclusions, and rules of engagement
- methodology, limitations, and test coverage
- prioritised findings with evidence, affected assets, attack paths, and risk rationale
- practical remediation actions and compensating controls
- a record of cleanup, unresolved artefacts, and required follow-up
- appendices for supporting technical detail

The testing team should remove accounts, files, payloads, keys, scheduled tasks, and other artefacts that it introduced, then confirm cleanup with the system owner. The organisation should protect the report as sensitive information through access controls, encryption, retention rules, and secure disposal. A focused retest should verify that remediation closes the demonstrated attack path without introducing a new weakness.
## Threat hunting and threat intelligence
### Threat hunting
Threat hunting is a proactive, evidence-led search for adversary behaviour that existing controls have not detected or resolved. It uses telemetry from endpoints, identities, networks, cloud services, email systems, and applications. Skilled analysts form a testable hypothesis, determine the required data, examine the evidence, and distinguish malicious activity from benign anomalies.

Hunting complements alert triage, compromise assessment, detection engineering, and incident response. A search for a known indicator can seed a hunt, but a narrow indicator check alone may miss related behaviour or variants. A confirmed finding should enter the incident-response process rather than remain inside a hunt.

Common starting points include:
- threat intelligence about an adversary, campaign, technique, or targeted technology
- an indicator of compromise, such as a domain, address, certificate, file hash, or account artefact
- an anomaly, such as unusual authentication, unexpected process execution, or atypical data movement
- a local risk, such as a new cloud service, acquisition, critical vulnerability, or high-value business process
- a known telemetry or detection gap
### Hunt lifecycle
A disciplined hunt follows an iterative lifecycle:
1. Define the objective, scope, assumptions, and success criteria.
2. Form a falsifiable hypothesis about adversary behaviour.
3. Map the behaviour to relevant tactics, techniques, and procedures, then identify the required telemetry.
4. Assess data quality, coverage, time synchronisation, retention, and known blind spots.
5. Query and analyse the data, then validate suspicious results with additional evidence.
6. Determine the affected identities, hosts, applications, time period, and potential impact.
7. Escalate confirmed activity, preserve evidence, and support containment when required.
8. Improve detections, telemetry, controls, playbooks, and future hypotheses.

Structured hunts often map a hypothesis to known adversary behaviours, including MITRE ATT&CK techniques. Unstructured hunts can begin with an indicator or anomaly and expand through time, systems, identities, and related activity. Situational hunts respond to local business change, architecture, vulnerabilities, or emerging campaigns. These labels describe useful approaches rather than rigid categories.
### Threat intelligence
Cyber threat intelligence is analysed, contextualised information that supports security decisions. It connects observations to questions about adversaries, intent, capability, infrastructure, victims, campaigns, vulnerabilities, and likely impact. Raw indicators become intelligence only after analysts assess their relevance, reliability, context, and timing.

Threat intelligence commonly operates at four levels:
- Strategic intelligence explains longer-term trends, risk, and potential business impact for senior decision-makers.
- Operational intelligence describes campaigns, adversaries, objectives, timing, targeting, and infrastructure.
- Tactical intelligence describes adversary tactics, techniques, and procedures for defenders and detection engineers.
- Technical intelligence supplies short-lived machine-readable observations, such as malicious domains, network addresses, URLs, certificates, and file hashes.

A common intelligence lifecycle includes:
1. Direction and requirements
2. Collection
3. Processing
4. Analysis and production
5. Dissemination and integration
6. Feedback and evaluation

Security teams act on the resulting intelligence through hunting, detection, hardening, incident response, and risk decisions. Action is an outcome of the lifecycle rather than a universally recognised seventh phase.
### Intelligence sources and sharing
Useful sources include:
- public reporting, research, repositories, and security communities
- commercial feeds, research services, and intelligence platforms
- trusted sharing groups, government bodies, and law-enforcement channels
- internal incidents, logs, case records, malware analysis, and lessons learned
- information obtained from human sources, subject to legal, ethical, and reliability controls

Analysts should grade source reliability, separate observation from assessment, record confidence, respect handling restrictions, and remove personal or sensitive information that recipients do not need.

Structured Threat Information Expression, or STIX, provides a language and data model for representing cyber threat and observable information. Trusted Automated Exchange of Intelligence Information, or TAXII, provides an application-layer protocol and API for exchanging that information. STIX and TAXII can improve interoperability, but they do not establish the accuracy or relevance of the content.
### SIEM, behaviour analytics, and automation
Security information and event management platforms centralise logs and support normalisation, correlation, alerting, dashboards, investigation, and reporting. They often integrate with endpoint detection and response, network security controls, identity systems, cloud platforms, threat-intelligence services, and security orchestration, automation, and response tools.

User behaviour analytics focuses on activity associated with users. User and entity behaviour analytics extends that scope to devices, servers, service accounts, applications, and other entities. These systems often baseline activity and assign risk scores to support triage. Analysts must validate results because unusual behaviour can be legitimate, and harmful behaviour can resemble routine work.

Machine learning and generative AI can cluster events, identify anomalies, enrich records, summarise evidence, and help analysts write queries. Their performance depends on data quality, representative training, secure integration, and human review. Security teams should test for false positives, false negatives, bias, data leakage, prompt injection, model drift, and adversarial manipulation before relying on automated output.
### Analytical frameworks
Frameworks organise evidence and guide investigation, but no single framework represents every intrusion.

- MITRE ATT&CK records adversary tactics, techniques, and procedures based on observed behaviour. Its Enterprise, Mobile, and ICS knowledge bases support threat modelling, hunting, detection engineering, adversary emulation, and control assessment.
- The Cyber Kill Chain describes reconnaissance, weaponisation, delivery, exploitation, installation, command and control, and actions on objectives. Its linear sequence can clarify defensive opportunities, but analysts should account for attacks that skip, repeat, or reorder stages.
- The Diamond Model represents an intrusion event through four connected features: adversary, infrastructure, capability, and victim. Analysts pivot among those features, add context, and link events into activity threads.

ATT&CK can describe how an adversary behaves, while the Diamond Model can organise relationships among the actor, infrastructure, capability, and victim. Used together, they can turn intelligence into hunt hypotheses, detection requirements, and defensive priorities.
## Cryptography
### Security properties
Cryptography uses mathematical techniques to protect information and communications. Correctly designed and implemented cryptographic systems can support:
- confidentiality by preventing unauthorised disclosure
- integrity by detecting unauthorised change
- authenticity by verifying data origin or a communicating party
- non-repudiation support by producing evidence that links a valid signature to a controlled signing key

Cryptography does not provide availability, correct weak access control, or protect a compromised endpoint by itself. Operational controls, software assurance, identity management, physical security, and incident response remain necessary.
### Core mechanisms
Encryption transforms plaintext into ciphertext under a key. An authorised recipient uses the corresponding key and algorithm to recover the plaintext. Secure encryption requires a suitable mode or construction, correct nonce handling, protected keys, and authentication where tampering is possible.

A cryptographic hash function maps an input to a fixed-length digest for that algorithm. A secure hash should make it computationally infeasible to recover the input from the digest or find two inputs with the same digest. A digest can detect change when a trusted value exists, but an unkeyed hash does not authenticate the source.

A message authentication code uses a shared secret to provide integrity and data-origin authentication. A digital-signature algorithm uses a private key to generate a signature and the corresponding public key to verify it. Signature schemes normally hash messages as part of defined generation and verification procedures. They do not generically encrypt a hash with a private key.

Password storage requires a purpose-designed, salted password-hashing or key-derivation scheme with a configurable work factor. General-purpose fast hash functions are unsuitable because they let attackers test guesses too quickly after a credential database leak.
### Symmetric, asymmetric, and hybrid systems
Symmetric encryption uses a shared secret key and performs efficiently on bulk data. The parties must protect and distribute that key securely.

Asymmetric cryptography uses related public and private keys. Depending on the algorithm, it supports encryption, digital signatures, or key establishment. Not every asymmetric algorithm supports all three functions.

Hybrid protocols combine these strengths. A handshake authenticates one or both parties and establishes shared keying material. Symmetric authenticated encryption then protects application data. TLS 1.3, for example, uses ephemeral Diffie-Hellman or pre-shared-key modes for key establishment and can use certificates with digital signatures for authentication. It removed static RSA key exchange.
### Algorithms and protocols
- AES is a symmetric block cipher with a 128-bit block size and 128-bit, 192-bit, or 256-bit keys. Security also depends on the selected mode, nonce management, key protection, and implementation.
- RSA relies on the difficulty of factoring large composite integers. Modern systems use it chiefly for signatures and some key-transport applications. Implementations require approved padding, adequate key sizes, and side-channel protections.
- Elliptic-curve cryptography supports efficient signatures and key agreement with smaller keys than traditional finite-field systems at comparable classical security levels.
- TLS protects a connection against eavesdropping, tampering, and message forgery when implementations validate peer identity, negotiate secure parameters, and protect private keys. TLS does not automatically provide end-to-end protection beyond the connection endpoints.
### Key management and implementation risk
Key management covers generation, registration, distribution, storage, use, rotation, backup, recovery, revocation, archival, and destruction. Weak randomness, exposed keys, uncontrolled copies, missed rotation, or failed revocation can defeat a sound algorithm.

Organisations should:
- generate keys and nonces with approved cryptographically secure random sources
- separate keys by purpose and environment
- store high-value private keys in appropriately protected cryptographic modules or managed key services
- restrict and audit key access
- rotate or replace keys according to risk, usage limits, compromise, and policy
- maintain certificate inventories and automate renewal where possible
- test recovery and revocation procedures
- remove obsolete protocols, cipher suites, algorithms, and certificates

TLS deployments should prefer supported modern protocol versions and authenticated-encryption suites. They should validate certificate names and chains, protect private keys, monitor expiry, and prevent downgrade. Certificate-transparency monitoring can reveal unexpected public certificates, but it does not replace certificate validation or revocation controls.
### Attacks and mitigations
Cryptographic attacks target algorithms, protocols, implementations, keys, passwords, or endpoints:
- Brute-force attacks search a key or password space. Adequate key strength, long user-chosen passwords, rate limits, multi-factor authentication, and expensive password hashing increase attacker cost.
- Dictionary and credential-stuffing attacks use common or previously exposed credentials. Blocklists for compromised passwords, unique passwords, password managers, rate limits, and phishing-resistant multi-factor authentication reduce risk.
- Machine-in-the-middle attacks intercept or alter communications. Authenticated TLS, correct certificate validation, secure name resolution where available, and trusted network controls reduce exposure.
- Downgrade attacks force weaker protocol choices. Version floors, secure negotiation, current libraries, and removal of legacy options limit the opportunity.
- Nonce or key reuse can break otherwise strong authenticated-encryption schemes. Implementations must follow each algorithm's uniqueness and usage limits.
- Known-plaintext and chosen-plaintext attacks give an adversary relationships between plaintext and ciphertext. Standard, well-reviewed authenticated-encryption schemes should remain secure under their defined attack models when implemented correctly.
- Side-channel attacks infer secrets from timing, cache behaviour, power use, electromagnetic emissions, or faults. Constant-time software, masking, blinding, hardware protections, and testing can reduce leakage.
- Differential and linear cryptanalysis examine statistical structure in block ciphers. Modern standardised algorithms use designs and parameters intended to resist these techniques.

Cryptanalysis strengthens security by testing assumptions, finding design and implementation flaws, and informing standards. Organisations should rely on open, well-reviewed algorithms rather than proprietary designs that depend on secrecy of the method.
### Evolution of cryptography
Cryptography developed from manual substitution systems into mathematically analysed algorithms and protocols:
- The Caesar cipher shifted letters by a fixed amount.
- Al-Kindi described frequency analysis, which enabled systematic attacks on monoalphabetic substitution ciphers.
- Polyalphabetic systems, including the Vigenere cipher, varied substitutions to obscure simple frequency patterns.
- The Enigma machine mechanised polyalphabetic substitution during the Second World War. Work by Polish cryptanalysts and the Allied teams at Bletchley Park, including Alan Turing and many others, helped decrypt significant German traffic.
- Whitfield Diffie and Martin Hellman published a landmark public-key agreement method in 1976.
- Ron Rivest, Adi Shamir, and Leonard Adleman published RSA in 1977.
- Modern protocols combine symmetric encryption, public-key techniques, hashing, authentication, and careful key management.
### Post-quantum cryptography
A cryptographically relevant quantum computer could use Shor's algorithm to break widely deployed RSA, finite-field Diffie-Hellman, and elliptic-curve systems. This creates a harvest-now-decrypt-later risk for information that must remain confidential for many years. Grover's algorithm gives a smaller, quadratic advantage against symmetric search, so larger symmetric keys can retain substantial security margins.

NIST finalised three principal post-quantum standards in 2024:
- FIPS 203 specifies ML-KEM, a key-encapsulation mechanism derived from CRYSTALS-Kyber.
- FIPS 204 specifies ML-DSA, a digital-signature algorithm derived from CRYSTALS-Dilithium.
- FIPS 205 specifies SLH-DSA, a stateless hash-based digital-signature algorithm derived from SPHINCS+.

NIST selected Falcon for an additional signature standard, FIPS 206, under the name FN-DSA, but that standard remains in development. NIST also selected HQC for future key-encapsulation standardisation, which remains under development. Organisations should refer to final standard names rather than describing Kyber, Dilithium, Falcon, and SPHINCS+ collectively as current leading candidates.

A post-quantum migration starts with an inventory of cryptographic assets, protocols, certificates, dependencies, and data-retention periods. Organisations can then prioritise long-lived secrets and exposed trust services, test interoperable implementations, build cryptographic agility, and follow relevant regulator, vendor, and standards guidance. New algorithms still require sound implementation, key management, and side-channel protection.
### Key points
- Penetration testing demonstrates realistic attack paths within an authorised scope, while vulnerability scanning identifies potential weaknesses at scale.
- Threat hunting tests hypotheses against organisational telemetry and converts findings into incident response, improved detections, and stronger controls.
- Threat intelligence adds context and decision value to observations, indicators, and adversary behaviour.
- Cryptography supports confidentiality, integrity, authenticity, and non-repudiation evidence, but secure outcomes depend on protocols, key management, implementation, and endpoint security.
- Security programs should retest changing environments, improve telemetry, and prepare for post-quantum migration.