# Cybersecurity architecture
> [!NOTE]
> Cybersecurity architecture combines governance, risk management, and technical controls across identity, endpoints, networks, applications, data, detection, response, and recovery.
## Architecture foundations
### Purpose and scope
Cybersecurity architecture translates business objectives, legal obligations, risk tolerance, and threat information into a coherent system design. It defines trust boundaries, security services, control responsibilities, and evidence requirements across the system lifecycle. Effective architecture reduces the likelihood and impact of compromise while supporting reliable delivery and operation.

Security decisions begin during requirements and design. Early decisions shape identity models, data flows, deployment patterns, recovery options, and operating costs. Retrofitted controls can address some weaknesses, but they rarely remove architectural exposure as efficiently as a sound initial design.
### Core design principles
- Defence in depth places independent and complementary controls across people, processes, and technology so that one failure does not expose the whole system.
- Least privilege limits each person, service, device, and process to the access required for an approved task and period.
- Separation of duties divides high-risk activities among independent roles and requires appropriate approval, review, or oversight.
- Secure by design integrates security into requirements, design, implementation, verification, deployment, operation, and retirement.
- Secure by default limits exposure and privilege without requiring users or administrators to discover and enable essential protections.
- Simplicity reduces configuration errors, hidden dependencies, and operational workarounds. Teams should choose the least complex design that satisfies the security and business requirements.
- Explicit trust decisions require systems to authenticate and authorise access according to identity, device or workload state, resource sensitivity, and current context.

Security by obscurity can add friction for an attacker, but secrecy of design cannot serve as the primary control. Kerckhoffs's principle expresses this limit for cryptography: a cryptosystem should remain secure when every detail except the key is public.
### Confidentiality, integrity, and availability
The confidentiality, integrity, and availability objectives provide a compact way to test a design.

| Objective | Security outcome | Common controls | Common failures |
| --- | --- | --- | --- |
| Confidentiality | Only authorised parties can access sensitive information. | Identity proofing, authentication, authorisation, encryption, key management, and data classification | Weak authentication, excessive privilege, exposed secrets, insecure key storage, and plaintext data |
| Integrity | Authorised parties can make approved changes, and systems can detect unauthorised or accidental changes. | Message authentication codes, digital signatures, protected audit logs, version control, change approval, and validated checks | Unprotected logs, unauthorised changes, missing audit trails, and unverified integrity results |
| Availability | Authorised users and services can access required capabilities within agreed service levels. | Redundancy, capacity management, rate limiting, monitoring, backups, disaster recovery, and denial-of-service protection | Resource exhaustion, single points of failure, unbounded workloads, and untested recovery plans |
### Confidentiality in practice
Access control and cryptography work together to protect confidentiality.

- Authentication establishes confidence that a claimant controls an authenticator bound to an identity.
- Authorisation decides which actions that identity may perform on a resource.
- Encryption protects content in transit and at rest. Symmetric cryptography uses a shared secret key. Public-key cryptography supports functions such as key establishment, encryption in suitable schemes, and digital signatures.
- Key management governs secure generation, storage, distribution, use, rotation or replacement, revocation, backup, recovery, and destruction.
- Data minimisation reduces the volume and sensitivity of information that systems collect, retain, and expose.

Strong algorithms cannot compensate for exposed keys, excessive access, insecure endpoints, or poorly designed recovery processes.
### Integrity in practice
Integrity controls must distinguish accidental corruption from hostile modification.

- Checksums can reveal accidental transmission or storage errors, but an attacker can often replace an unprotected checksum.
- Cryptographic hashes support integrity checking when a trusted process protects the expected hash value.
- Message authentication codes establish integrity and source authenticity when authorised parties share a secret key.
- Digital signatures establish integrity and origin when verifiers trust the signer's public key and the private key remains protected.
- Append-only or write-once-read-many storage, restricted administration, and forwarding to separate systems can protect audit evidence.
- Change control, peer review, and version history make changes attributable and reversible.
### Availability in practice
Availability engineering addresses faults, malicious activity, operational error, and dependency failure. Denial-of-service attacks exhaust a constrained resource such as bandwidth, connection state, memory, processing time, or a downstream service quota.

- A distributed denial-of-service attack sends traffic or workload from many sources, which can include a botnet.
- A SYN flood sends Transmission Control Protocol connection requests without completing the three-way handshake, consuming connection state until controls discard or expire it.
- Reflection attacks spoof a target's address so third-party services send responses to the target.
- Amplification attacks use protocols whose responses greatly exceed the request size.

Common protections include upstream filtering, connection and request limits, rate limiting, short and risk-appropriate timeouts, caching, bounded queues, circuit breakers, resilient dependencies, and capacity reserves. Autoscaling requires cost and resource guardrails because unbounded scaling can convert an attack into a financial or dependency failure. Alerting, runbooks, tested backups, and recovery exercises complete the architecture.
### Cybersecurity architect responsibilities
A cybersecurity architect converts stakeholder needs into target, reference, and solution architectures, then guides the teams that implement and operate them. The architect examines how a system can fail and how attackers can abuse it, not only how it should function.

Core responsibilities include:
- Establishing business objectives, legal and regulatory obligations, risk tolerance, and delivery constraints
- Identifying assets, sensitive data, trust assumptions, external dependencies, and failure modes
- Modelling threats and abuse cases, including threats across organisational and technology boundaries
- Selecting controls that reduce risk and satisfy confidentiality, integrity, availability, privacy, safety, and resilience requirements
- Defining reusable security patterns, standards, and decision records
- Reviewing implementation evidence and revisiting assumptions as the system, threats, and operating environment change

STRIDE can help teams classify threats as spoofing, tampering, repudiation, information disclosure, denial of service, and elevation of privilege. It supports threat modelling, but it does not replace root-cause analysis after an incident.
### Architecture artefacts
Architecture artefacts should show enough detail to support decisions and verification.

- Business context diagrams show relationships among users, organisations, business capabilities, and external parties.
- System context diagrams map business relationships to systems, services, and integrations.
- Data-flow diagrams show processes, data stores, data flows, external entities, and trust boundaries.
- Deployment diagrams show runtime components, zones, platforms, and administrative paths.
- Sequence diagrams show identity exchanges, authorisation decisions, error paths, and sensitive transactions.
- Decision records capture assumptions, alternatives, consequences, owners, and review triggers.

Every diagram should label sensitive data, trust boundaries, protocols, authentication points, authorisation points, and security-relevant dependencies. Diagram reviews should confirm that every cross-boundary flow has an explicit purpose and control set.
### Security domains
Most architectures distribute controls across connected domains:
- Governance, risk, and compliance
- Identity and access management
- Endpoint and workload security
- Network and infrastructure security
- Application and software supply chain security
- Data security and privacy
- Monitoring, detection, and threat intelligence
- Incident response, recovery, and resilience

The architecture should collect usable telemetry from each domain, correlate evidence across boundaries, and support timely containment and recovery. Prevention alone cannot address every credible scenario.
### Cybersecurity Framework 2.0
The National Institute of Standards and Technology (NIST) Cybersecurity Framework 2.0 organises cybersecurity outcomes into six concurrent functions.

- Govern establishes and monitors cybersecurity risk strategy, policy, roles, oversight, and supply chain risk management.
- Identify develops an understanding of assets, suppliers, vulnerabilities, threats, and current risk.
- Protect applies safeguards such as access control, awareness, data security, platform security, and infrastructure resilience.
- Detect finds and analyses possible attacks, compromises, and other adverse events.
- Respond manages, analyses, contains, reports, and communicates about detected incidents.
- Recover restores affected assets and operations, verifies recovery, and communicates progress.

The framework defines outcomes rather than prescribing a product or implementation. Organisations can use current and target profiles to expose gaps, set priorities, and communicate risk.
### Architecture delivery checks
- Do business owners define confidentiality, integrity, availability, privacy, and recovery requirements for critical services and data?
- Do diagrams show assets, dependencies, administrative paths, sensitive data flows, and trust boundaries?
- Do all interfaces enforce consistent authentication and authorisation?
- Do services validate input, protect output, handle errors safely, and restrict secrets?
- Do teams protect code, configuration, logs, and critical records against unauthorised change?
- Do designs remove single points of failure and bound resource consumption?
- Do monitoring and logging support detection, investigation, attribution, and legal obligations?
- Do incident response and recovery plans assign owners, define decision authority, and receive regular exercises?
- Do assurance activities test control operation and record residual risk?
## Identity and endpoint security
### Endpoint trust and scope
Endpoints include user devices, servers, virtual machines, cloud workloads, mobile devices, operational technology, and internet-connected equipment. A compromised or unmanaged endpoint can steal authenticators, alter transactions, evade local controls, or provide an attacker with a foothold. Strong server-side identity controls limit the damage, but they do not make endpoint condition irrelevant.

Endpoint inventories should cover:
- Physical and virtual servers
- Desktops, laptops, mobile phones, and tablets
- Cloud instances, containers, and managed workloads
- Operational technology and Internet of Things devices
- Personally owned devices authorised for work
- Administrative workstations and recovery devices

Each endpoint type needs an owner, an approved configuration, a support status, a risk classification, and a retirement process.
### Endpoint management
An organisation may use one management platform or several specialised platforms. Security depends on complete coverage, consistent policy, reliable telemetry, and coordinated response rather than the number of consoles.

Core controls include:
- Asset discovery that identifies authorised, unknown, unmanaged, and retired devices
- Vendor-supported operating systems, firmware, and applications
- Secure configuration baselines with controlled exceptions and drift detection
- Risk-based patching that accounts for exposure, exploitation, asset criticality, and compensating controls
- Secure boot, hardware-backed key protection, and application control where supported
- Device locking and authentication settings aligned with the organisation's identity policy
- Full-disk or device encryption with recoverable, protected keys
- Endpoint protection, endpoint detection and response, and central alerting
- Restricted local administration and dedicated controls for privileged workstations
- Remote lock or wipe capabilities for managed devices that remain reachable, with controls that respect ownership, safety, and privacy obligations
- Secure media sanitisation and disposal at retirement

Password policies should emphasise length, compromised-password blocklists, rate limiting, and secure password storage. Systems should not impose routine password changes without evidence of compromise, and they should not require arbitrary character-composition rules. Higher-risk access should use phishing-resistant authentication where practical.
### Personally owned devices and shadow IT
Personally owned device arrangements can improve flexibility, but unclear rules encourage unmanaged applications, accounts, and cloud services. An enforceable program should make the approved path easy to use and should define:
- Eligibility, enrolment, support, and removal conditions
- The data and activity the organisation may manage or monitor
- Transparent notices, privacy boundaries, and an applicable legal basis
- Minimum device, operating system, encryption, and security-agent requirements
- Separation of corporate and personal data where the platform supports it
- Selective wipe and account-revocation procedures for loss, theft, or employment exit
- Approved applications and services for email, messaging, file sharing, and collaboration
- Exceptions for accessibility, safety, operational, and legal needs

A prohibition without effective enforcement can displace activity into unsanctioned services. Governance, usable approved services, and proportionate technical controls reduce that risk.
### Identity and access management
Identity and access management governs digital identities and their access throughout the lifecycle.

| Capability | Purpose |
| --- | --- |
| Administration | Creates, updates, suspends, and removes identities, accounts, credentials, roles, and entitlements. |
| Authentication | Establishes confidence that a claimant controls an authenticator bound to an identity. |
| Authorisation | Determines whether an authenticated identity may perform an action on a resource. |
| Audit | Records and evaluates identity events, access decisions, administrative changes, and control performance. |
### Directories and identity data
Directories store and expose identity attributes through defined schemas and interfaces. Lightweight Directory Access Protocol is one common access protocol, but modern identity systems also use databases, application programming interfaces, and cloud directory services.

Large organisations often operate several authoritative and application-specific identity stores. Integration patterns include:
- A virtual directory that presents a unified view and retrieves attributes from source systems on demand
- A metadirectory that copies and reconciles selected attributes into a consolidated store
- Standards-based provisioning that creates, updates, and disables accounts across connected services

Synchronisation creates security and integrity risks when systems disagree. The architecture should identify authoritative sources, attribute ownership, conflict rules, update latency, and failure handling.
### Identity lifecycle and governance
Identity governance and administration connects business roles, approval policy, provisioning, access review, and deprovisioning.

- Joiner processes create approved baseline access after the authoritative workforce or customer event.
- Mover processes remove obsolete access before or while granting access for a new role.
- Leaver processes disable interactive and programmatic access promptly, recover assets, and preserve required records.
- Periodic and event-driven reviews remove dormant accounts, toxic combinations, and accumulated privilege.

A central record of entitlements helps teams discover where access exists and confirm that revocation has reached each target system.
### Authentication factors and assurance
| Factor category | Examples |
| --- | --- |
| Knowledge | A password, passphrase, or activation PIN |
| Possession | A device or token that holds an authentication secret or cryptographic key |
| Inherence | A biometric characteristic used through an approved biometric system |

Multi-factor authentication requires proof of two distinct factor categories. Two passwords do not create multi-factor authentication. A phone number alone is not a possession factor, while control of an authenticator bound to a phone can provide one.

Biometrics are not secret and should normally activate an authenticator rather than serve as a remotely stored password substitute. Passwordless authentication does not automatically provide multi-factor or phishing-resistant assurance. The protocol, authenticator, activation method, key protection, and recovery process determine the assurance level.

Authentication design should match risk. High-value or administrative access should favour phishing-resistant cryptographic authenticators, strong device assurance, protected recovery, and reauthentication for sensitive actions. Manually entered one-time codes can support multi-factor authentication, but they remain vulnerable to phishing and relay attacks.
### Single sign-on, federation, and adaptive access
Single sign-on reduces password sprawl and centralises policy, but it also increases the impact of identity-provider failure or compromise. The design should protect the sign-on point, restrict token scope and lifetime, validate token audience and issuer, secure signing keys, and provide resilient recovery.

Federation allows an identity provider to assert identity information to a service provider across organisational or platform boundaries. Contracts and technical controls should define assurance, attribute release, account linking, revocation, logging, key rollover, and incident communication.

Adaptive access can consider resource sensitivity, requested action, device condition, location signals, network context, session history, and anomalous behaviour. Risk signals should strengthen an access decision, not replace sound authentication or authorisation.
### Privileged access management
Privileged access management protects administrator, root, service, and emergency accounts. Controls commonly include:
- Named administrator identities with multi-factor authentication
- Separate standard and privileged accounts
- Just-in-time or time-bounded privilege elevation
- Approval for high-risk actions and emergency access
- Credential vaulting and rotation where shared or reusable credentials remain necessary
- Session brokering, command control, or recording where risk and privacy obligations justify it
- Closely monitored emergency accounts with tested access procedures
- Restricted administrative paths and hardened privileged workstations

Automation should use managed workload identities or short-lived credentials instead of embedded long-lived secrets wherever supported.
### Identity audit and behaviour analytics
Audit evidence should connect lifecycle changes, authentication events, authorisation decisions, privilege use, and administrative activity. Correlation can expose suspicious sequences such as account creation, privilege assignment, bulk data access, and rapid deletion. User and entity behaviour analytics can help identify deviations, but analysts must validate context because unusual activity does not establish malicious intent by itself.
## Network, application, and data security
### Network security and segmentation
Network architecture controls connectivity, limits implicit trust, and provides evidence about flows. Segmentation reduces the paths available after compromise, but effective segmentation requires enforced policy and monitored exceptions.

Common patterns include:
- An internet-facing zone for public services with tightly restricted inbound and outbound access
- Internal service zones separated by sensitivity, function, environment, or administrative ownership
- Dedicated management paths that do not share ordinary user access
- Isolated backup, recovery, security, and identity infrastructure
- Explicit allow lists for required flows and deny-by-default handling for other traffic

Firewalls can apply packet, stateful, application-aware, and identity-aware policy. Anti-spoofing controls reject traffic with invalid or inappropriate source addresses. Egress controls restrict command-and-control traffic, data exfiltration, and unapproved dependencies.

Remote access can use virtual private networks, zero trust network access, secure access service edge capabilities, or controlled administrative gateways. The choice should enforce strong authentication, device checks, least privilege, session protection, and useful logging.

Zero trust does not remove networks or grant universal access after authentication. It removes implicit trust based only on network location, affiliation, or asset ownership. Each access decision should consider the subject, device or workload, resource, requested action, and relevant risk signals.
### Security architecture diagrams
A web service diagram should identify external users, administrators, edge controls, public services, internal services, data stores, identity services, management systems, and external dependencies. It should also show:
- Trust boundaries and security zones
- Flow direction, purpose, protocol, and port where relevant
- Authentication and authorisation enforcement points
- Encryption in transit and at rest
- Secret and key locations without exposing secret values
- Logging sources, collection paths, and protected stores
- Failure paths, recovery dependencies, and administrative access

Sensitive data stores should have no direct path from untrusted zones. Every cross-zone flow should pass through an explicit enforcement point, and every exception should have an owner, rationale, expiry or review date, and monitoring requirement.
### Secure software delivery
Application security treats software risk as a lifecycle responsibility. Teams introduce weaknesses through requirements, design, code, dependencies, configuration, deployment, and operation. Early controls reduce rework, but production monitoring and response remain essential.

The Secure Software Development Framework can integrate secure practices into iterative, sequential, and hybrid delivery models. DevSecOps applies the same principle through shared responsibility, automated feedback, and security controls within delivery and operating workflows.

Secure coding practices include:
- Validating input against expected type, length, range, format, and business rules
- Using parameterised queries and safe application programming interfaces to reduce injection risk
- Applying context-appropriate output encoding to reduce cross-site scripting risk
- Using memory-safe languages or robust bounds and lifetime controls for memory safety
- Reusing reviewed authentication, authorisation, session, and cryptography components
- Handling errors without exposing secrets, internal paths, stack traces, or sensitive records
- Keeping secrets out of source code, build logs, images, and unprotected configuration
- Combining automated checks with targeted peer and specialist review
### Application security resources
The Open Worldwide Application Security Project (OWASP) Top 10 is an awareness resource, not a complete verification standard. The 2025 edition identifies these web application risk categories:
1. Broken access control
2. Security misconfiguration
3. Software supply chain failures
4. Cryptographic failures
5. Injection
6. Insecure design
7. Authentication failures
8. Software or data integrity failures
9. Security logging and alerting failures
10. Mishandling of exceptional conditions

The OWASP Application Security Verification Standard provides detailed, testable security requirements. The OWASP Cheat Sheet Series provides implementation guidance. Teams can use the Top 10 for awareness, the verification standard for requirements and assurance, and threat modelling for system-specific risks.
### Security testing across the lifecycle
No single tool finds every weakness. A layered testing strategy combines complementary evidence.

| Stage | Representative controls |
| --- | --- |
| Design | Threat modelling, abuse cases, architecture review, data-flow review, and security requirements |
| Build | Static analysis, dependency analysis, secret scanning, infrastructure-as-code scanning, and policy gates |
| Test | Dynamic application testing, application programming interface testing, fuzzing, security regression tests, and manual review |
| Release | Artefact signing, provenance checks, environment approval, configuration verification, and rollback readiness |
| Runtime | Workload hardening, runtime monitoring, web application firewall controls where justified, anomaly detection, and response automation |

Automated results need triage. Teams should tune rules, record accepted risk, track recurring causes, and give engineers fast feedback with clear remediation guidance.
### Third-party components and supply chain records
Modern applications depend on packages, libraries, build services, containers, and external platforms. The Log4Shell vulnerability in Apache Log4j 2 demonstrated how a flaw in a widely deployed component can create urgent, portfolio-wide exposure.

A software bill of materials (SBOM) records components and dependency relationships. It can include version, supplier, identifier, and provenance data, but an SBOM does not by itself prove that a component is vulnerable or that a product is exploitable. Vulnerability Exploitability eXchange (VEX) information can state whether a known vulnerability affects a particular product and explain the status.

Effective supply chain controls include trusted repositories, dependency pinning, update processes, build isolation, artefact signing, provenance verification, and rapid inventory queries after disclosure.
### Generative artificial intelligence in software delivery
Generative artificial intelligence can accelerate coding, review, and troubleshooting, but it can also produce insecure code, disclose sensitive inputs, invent dependencies, or reproduce unsafe patterns. Organisations should:
- Apply the same security requirements and review standards to generated and human-written code
- Use approved services, accounts, and data-handling configurations
- Keep confidential code, credentials, personal information, and restricted data out of unapproved prompts
- Verify packages, application programming interfaces, licences, and technical claims
- Scan, test, and review generated changes through the normal delivery workflow
- Record significant use where traceability, assurance, or legal obligations require it
### Service-to-service security and observability
Internal network location should not establish trust. Services should authenticate workloads, authorise each requested action, restrict credentials, and encrypt sensitive traffic. Mutual Transport Layer Security can authenticate both ends of a connection, but the application or policy layer must still authorise access.

Logs, metrics, and traces support detection and diagnosis across distributed systems. Telemetry should use consistent time, service identity, request correlation, and event schemas. Systems should prevent secrets and unnecessary personal information from entering logs, protect log integrity, and retain evidence according to risk and legal requirements.
### Data governance and classification
Data security protects information during creation, collection, storage, use, transfer, sharing, archival, and disposal. Governance assigns ownership and connects classification to concrete handling rules.

An effective classification model defines:
- Clear levels with practical examples
- Owners and decision authority
- Minimum access, encryption, logging, sharing, retention, and disposal controls for each level
- Labelling and discovery requirements
- Exception and declassification processes
- Recovery priorities and data-integrity requirements

Classification fails when labels are vague, inconsistent, or disconnected from controls.
### Discovery and data loss prevention
Discovery identifies sensitive data across databases, object stores, file systems, email, collaboration platforms, endpoints, backups, and logs. A catalogue should record ownership, purpose, location, lineage, classification, retention, and approved use.

Data loss prevention can inspect and control sensitive content at endpoints, in networks, and within cloud services. Effective policies use accurate data definitions, contextual signals, staged enforcement, and exception handling. Poorly tuned controls create excessive alerts or disrupt legitimate work.
### Data protection and resilience
Core protections include:
- Encryption in transit and at rest with approved algorithms and configurations
- Managed key lifecycles with separation between keys and protected data
- Least-privilege access linked to identity governance
- Integrity controls for critical records and transactions
- Backups aligned with recovery point and recovery time objectives
- Immutable or offline recovery copies where ransomware risk justifies them
- Separate backup administration and protected recovery credentials
- Restore tests that verify data, applications, dependencies, and operating procedures

Retention schedules should satisfy legal, regulatory, contractual, operational, and historical needs. Disposal should remove data when no valid purpose remains, subject to legal holds and authorised exceptions.
### Data detection and response
Monitoring should cover sensitive data access, permission changes, bulk exports, unusual queries, new sharing links, and transfers to unapproved destinations. Playbooks should define investigation, containment, evidence preservation, legal review, communication, and recovery. Automation can enrich and contain known scenarios, while human decision-makers should control novel or high-impact actions.
### Post-quantum cryptography
A sufficiently capable cryptographically relevant quantum computer could break widely used public-key algorithms. Data with a long confidentiality lifetime faces harvest-now-decrypt-later risk when an adversary stores encrypted traffic for future decryption.

Organisations should begin migration planning with:
- A cryptographic inventory covering protocols, certificates, libraries, hardware, code signing, identity systems, and external dependencies
- Identification of data and signatures that require long-term protection
- Crypto-agile interfaces that support algorithm and parameter changes
- Supplier and standards roadmaps
- Interoperability, performance, rollback, and failure testing

NIST has standardised the Module-Lattice-Based Key-Encapsulation Mechanism (ML-KEM) for key establishment, the Module-Lattice-Based Digital Signature Algorithm (ML-DSA), and the Stateless Hash-Based Digital Signature Algorithm (SLH-DSA). NIST selected HQC as a backup key-encapsulation algorithm, but standardisation remains in progress. Migration should follow applicable standards and protocol guidance rather than replace algorithms without system-level testing.
### Architectural support during incidents
Architects support incident teams by relating evidence to trust boundaries, dependencies, and design assumptions.

During an incident, architects can:
- Map likely attack paths and affected data flows
- Identify containment points, dependencies, and operational consequences
- Assess exposure in adjacent systems and shared services
- Propose changes to connectivity, identity, trust, or deployment patterns
- Record temporary risk decisions and conditions for reversal

After an incident, architects can:
- Participate in root-cause and contributing-factor analysis
- Update threat models, diagrams, assumptions, and risk records
- Design remediation that addresses control and architecture gaps
- Validate that changes do not introduce new single points of failure
- Add regression tests, monitoring, and review triggers

Useful resilience patterns include deny-by-default policy, bounded resource use, circuit breakers, isolated recovery systems, immutable deployment artefacts, rapid workload replacement, strong administrative identity, and tested degraded modes.
### Security audits
Security audits assess whether required controls exist, operate effectively, and produce sufficient evidence. A risk-based audit usually covers:
- Scope, criteria, systems, processes, suppliers, and obligations
- Architecture, configuration, policy, access, change, and incident evidence
- Control design and operating effectiveness
- Authorised vulnerability assessment or testing where relevant
- Risk analysis that considers likelihood, impact, exposure, and control strength
- Findings with owners, priorities, due dates, and verification requirements

Audits provide assurance at a point in time. Continuous monitoring, management review, and follow-up verification sustain improvement between audits.
## Detection and response
### Security operations
A security operations centre coordinates monitoring, analysis, investigation, and response. It combines telemetry from identity, endpoints, workloads, networks, applications, cloud platforms, and data controls. Clear ownership and escalation paths are as important as collection volume.
### Security information and event management and extended detection and response
A security information and event management system collects, parses, normalises, searches, and correlates security-relevant data. It can support alerting, investigation, compliance reporting, trend analysis, and case workflows.

Extended detection and response combines detection and response capabilities across endpoints and other security domains. Product boundaries vary. In many environments, security information and event management provides broad data retention and correlation, while extended detection and response provides deeper endpoint or workload context and direct response actions. Architecture should define the source of truth, data ownership, hand-off points, and action authority.
### Indicators of compromise and threat intelligence
An indicator of compromise is an observable that may support a conclusion that compromise occurred. Examples include malicious file hashes, command-and-control domains, unauthorised accounts, suspicious process chains, anomalous authentication, and unexpected data transfers. A single weak indicator, such as a slow device, rarely confirms compromise without context.

Threat intelligence converts threat information into analysis that supports a decision. It can address:
- Strategic questions about business exposure, investment, and risk
- Operational questions about campaigns, adversaries, and likely targeting
- Tactical questions about attacker techniques and defensive opportunities
- Technical questions about infrastructure, malware, vulnerabilities, and indicators

Sources include internal incidents, government advisories, industry communities, open sources, commercial services, research reports, honeypots, and controlled sharing relationships. Collection and sharing should protect privacy, confidential information, legal privilege, and source restrictions.
### Vulnerability assessment
A vulnerability assessment identifies and analyses weaknesses across assets, configurations, software, identities, and cloud services. It differs from a penetration test, which attempts to demonstrate attack paths within an explicitly authorised scope.

| Assessment type | Primary focus |
| --- | --- |
| Network | Exposed services, protocols, segmentation, filtering, and unmanaged devices |
| Host and endpoint | Missing updates, insecure configuration, local privilege, software inventory, and unnecessary services |
| Application and application programming interface | Design weaknesses, authentication, access control, input handling, session management, and business logic |
| Cloud and platform | Identity policy, public exposure, storage permissions, workload configuration, logging, and control-plane settings |
| Container and supply chain | Base images, packages, secrets, provenance, runtime privilege, and orchestrator policy |

Tools can include asset discovery, authenticated infrastructure scanners, static and dynamic application testing, software composition analysis, cloud security posture management, container scanning, and manual validation. Active testing requires authorisation, safety controls, and coordination with service owners.
### Risk-based prioritisation
Severity alone does not establish remediation priority. Prioritisation should combine:
- Asset criticality and data sensitivity
- Internet exposure and reachable attack paths
- Evidence of active exploitation, including the CISA Known Exploited Vulnerabilities Catalog
- Exploitation likelihood signals such as the Exploit Prediction Scoring System
- Vulnerability severity and environmental context, including the Common Vulnerability Scoring System
- Existing controls, detection coverage, and recovery capability
- Remediation complexity, service risk, and available mitigations

Scanning frequency should reflect asset exposure, change rate, threat activity, and detection capability. Dynamic internet-facing environments usually require more frequent assessment than stable, isolated assets. Metrics should track risk reduction, overdue high-priority findings, recurrence, exception age, and time to verified remediation.
### Incident response and orchestration
Incident response coordinates triage, analysis, containment, eradication, recovery, communication, and improvement. Triage validates the signal, assesses potential impact, preserves evidence, assigns severity, and establishes decision authority.

Security orchestration, automation, and response platforms can provide:
- Case management for ownership, priority, evidence, decisions, and status
- Automated enrichment of indicators, assets, identities, and vulnerabilities
- Playbooks for repeatable investigation and response
- Integration with security, identity, endpoint, network, cloud, ticketing, and communication systems
- Approval gates for disruptive or irreversible actions

Automation suits bounded and well-tested actions. Analysts should retain control when evidence is incomplete, business impact is high, or an action could destroy evidence or disrupt critical services.
### Breach notification
Response teams should promptly determine the affected data, individuals, systems, jurisdictions, likely harm, containment status, and notification deadlines. Legal and privacy specialists should interpret applicable requirements because thresholds and timeframes differ.

Under Australia's Notifiable Data Breaches scheme, covered entities must notify affected individuals and the Office of the Australian Information Commissioner when an eligible breach is likely to result in serious harm and remedial action has not removed that likelihood. Under the European Union General Data Protection Regulation, a controller generally must notify the competent supervisory authority without undue delay and, where feasible, within 72 hours after becoming aware of a personal data breach unless the breach is unlikely to result in a risk to people's rights and freedoms.

The incident plan should preserve the facts, decisions, timing, and approvals that support any notification or decision not to notify.
### Incident response frameworks
NIST Special Publication 800-61 Revision 3 integrates incident response across all six Cybersecurity Framework 2.0 functions. Govern, Identify, and Protect support preparation and risk reduction. Detect, Respond, and Recover drive operational incident handling, while lessons inform every function.

The SANS incident handling model uses six stages:
1. Preparation
2. Identification
3. Containment
4. Eradication
5. Recovery
6. Lessons learned

Framework labels should not delay action. Teams may conduct containment, analysis, recovery planning, communication, and evidence preservation in parallel as facts develop.
### Illustrative scenario: malware in an isolated operational environment
An isolated operational network can still receive malware through removable media, maintenance equipment, supply chain updates, portable administrator devices, or an incorrectly controlled connection. Limited telemetry and low tolerance for downtime can delay discovery and constrain containment.

Endpoint telemetry, protected logs, and network evidence can reconstruct persistence, execution, collection, and lateral movement. A proportionate response can isolate affected segments, preserve evidence, remove persistence, rotate exposed credentials, rebuild compromised devices, scan removable media, and verify recovery through targeted threat hunting. Architectural improvements can strengthen media control, administrative paths, internal segmentation, monitoring, and separation between public and operational services.