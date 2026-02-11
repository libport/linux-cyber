# Cybersecurity Architecture

> [!NOTE]
> This document introduces cybersecurity architecture as the disciplined design of systems that remain trustworthy under attack, grounded in the CIA triad and practical frameworks. It explains core principles like defence in depth, least privilege, separation of duties, secure-by-design, and simplicity, and contrasts them with security by obscurity. It then surveys key domains of endpoint, identity and access management, network, application, and data security. Plus monitoring, SIEM/XDR, incident response, SOAR, and vulnerability assessment, linking practices to OWASP and NIST for end-to-end risk reduction.
## Cybersecurity Architecture Overview
### Cybersecurity architecture fundamentals
Cybersecurity architecture applies security principles, the CIA triad, and recognised frameworks to design systems that remain trustworthy under attack and during failures. It aims to reduce risk to acceptable levels while keeping delivery practical for product teams and operations. Security is most effective when designed into the system lifecycle rather than bolted on after implementation.
### Five security principles
- Defence in depth uses multiple, independent layers so that a single control failure does not expose the system.
- Least privilege grants only the minimum access required, for the minimum time required, and removes unnecessary services and accounts.
- Separation of duties avoids single points of control by requiring independent approval or oversight for high-risk actions.
- Secure by design integrates security from requirements through deployment and operations so that secure defaults are the starting point.
- KISS keeps designs and controls as simple as possible so that legitimate users can comply without workarounds.
### Principle to avoid
- Security by obscurity treats secrecy of design as protection. Kerckhoffs's principle is the common counterpoint, stating that a cryptosystem should remain secure even if everything about it is public except the key.
### The CIA triad
The CIA triad is a practical checklist for system design and assurance.

| Objective | Meaning | Common controls | Typical failures |
| --- | --- | --- | --- |
| Confidentiality | Only authorised parties can access sensitive information. | Authentication and authorisation, role-based access control (RBAC), encryption in transit and at rest, key management, data classification. | Weak identity proofing, excessive privileges, poor key handling, plaintext storage or transport. |
| Integrity | Information remains accurate and tampering is detectable. | Hashing and checksums, message authentication codes (MACs), digital signatures, immutable logging, version control, controlled change processes. | Unprotected logs, unauthorised changes, lack of audit trails, integrity checks that are not verified. |
| Availability | Authorised users can access services when needed. | Redundancy, capacity planning, monitoring, rate limiting, timeouts, DDoS protection, backups, disaster recovery planning. | Resource exhaustion, single points of failure, unbounded sessions, poor resilience planning. |
### Confidentiality in practice
Confidentiality is commonly enforced through access control and encryption.

- Authentication answers who the user is, for example with multi-factor authentication that combines factors such as something known, something held, and something inherent.
- Authorisation answers what the authenticated user is allowed to do, often through RBAC that maps roles to permitted actions and resources.
- Encryption protects data so that interception does not expose content, provided the attacker does not have the key. Symmetric encryption uses the same shared key to encrypt and decrypt, while asymmetric encryption uses a public and private key pair to support secure exchange and digital signatures.
- Key management is essential. Strong encryption provides little value if keys are stored insecurely, shared widely, or never rotated.
### Integrity in practice
Integrity mechanisms ensure that modification attempts can be detected and acted upon.

- System logs should be protected so attackers cannot erase or rewrite evidence of activity. Common approaches include append-only storage, write once read many media, and forwarding logs to a separate system.
- Cryptographic integrity checks, such as MACs and digital signatures, can reveal whether messages, records, or logs have been altered.
- Change control and versioning support integrity by making edits attributable and reviewable. This is relevant to both configuration and code.
- Distributed ledgers and append-only data structures support immutability by making edits and deletions evident, while allowing new entries to be added.
### Availability in practice
Availability failures commonly involve denial of service where resources are exhausted.

- A denial of service attack overwhelms a service with traffic or workload so legitimate requests cannot be served.
- A distributed denial of service attack amplifies impact by using many compromised machines under central control, often a botnet.
- A SYN flood abuses the TCP three-way handshake. The client sends SYN, the server replies with SYN-ACK, and the client should complete with ACK. Attackers send large numbers of SYNs without completing the handshake, forcing the server to hold state until timeouts or resource limits are reached.
- Reflection and amplification attacks can increase attacker impact by tricking third-party systems into sending large responses to a target.

Common mitigations include aggressive timeouts, connection limits, upstream filtering, rate limiting, caching, autoscaling with guardrails, and designing capacity and redundancy for expected abuse cases. Availability also relies on operational readiness, including alerting, runbooks, and tested recovery procedures.
### Cybersecurity architect role and mindset
A cybersecurity architect translates stakeholder needs into a secure reference architecture and guides engineers who implement the design. The role differs from general IT architecture because it considers how systems fail, not only how they function. The architect typically works through:
- Understanding business goals, compliance requirements, and constraints from stakeholders.
- Producing context and overview diagrams that communicate components, trust boundaries, and critical data flows.
- Identifying threat scenarios and failure modes, then selecting mitigations that reduce risk and meet the CIA objectives.
- Defining security principles and policies that can be implemented consistently by delivery teams.
- Reviewing designs and changes over time so security assumptions remain valid as systems evolve.

The architect is most effective when involved early and continuously. Late engagement often forces expensive redesigns and leads to weaker compromises.
### Common architecture artefacts
Architects often use a progressive set of diagrams.

- Business context diagrams show high-level relationships among actors and organisations.
- System context diagrams map those relationships to systems and integrations.
- Architecture overview diagrams describe key components, interfaces, and data flows at a level suitable for threat modelling and control placement.

These artefacts provide a shared language between stakeholders, architects, and engineers.
### Security domains in a typical system
Security controls are typically designed across multiple domains that align to system components.

- Identity and access management
- Endpoint security
- Network security
- Application security
- Data security
- Monitoring and detection, often through security information and event management
- Incident response and orchestration

A secure architecture aims to collect security telemetry from each domain, detect anomalies quickly, and coordinate response actions so incidents are contained before they become business-impacting outages.
### OWASP Top 10 and application security
The OWASP Top 10 is a widely used set of web application risk categories that helps teams focus on common attack paths. The 2021 edition includes:
- Broken access control
- Cryptographic failures
- Injection
- Insecure design
- Security misconfiguration
- Vulnerable and outdated components
- Identification and authentication failures
- Software and data integrity failures
- Security logging and monitoring failures
- Server-side request forgery (SSRF)

The list is most useful when it informs threat modelling, architecture reviews, and development gates rather than being treated as a compliance checklist. For example, access control should be enforced at each layer, injection risk should be reduced through parameterised queries and input validation, and logging should be designed so investigations are possible.
### Practical tools aligned to OWASP risks
- OWASP ZAP (ZAP by Checkmarx) supports dynamic testing for common web issues and is often suited to development and smaller environments.
- Burp Suite is commonly used for manual penetration testing workflows and deeper investigation.
- SonarQube supports static analysis across many languages and can help detect patterns linked to common web risks.
- Snyk supports software composition analysis to identify vulnerable dependencies and recommend remediation.

Tooling provides value when it is integrated into delivery workflows, such as pull request checks, pipeline scans, and regular dependency review.
### NIST Cybersecurity Framework
The NIST Cybersecurity Framework provides a common structure for managing cyber risk through five functions.

- Identify focuses on understanding assets, governance, and risk context.
- Protect implements safeguards such as access control, training, and data protection.
- Detect enables timely discovery through monitoring, alerting, and analysis.
- Respond coordinates containment, eradication, and communications.
- Recover restores services and incorporates lessons learned into improved resilience.

These functions operate continuously, with outputs from one informing the others. The framework can be used as a coverage check to confirm that prevention, detection, response, and recovery are all addressed.
### Implementing least privilege
Least privilege is implemented through a mix of policy, process, and technology.

- Identity and access management tools enforce consistent authentication and authorisation decisions.
- RBAC simplifies administration by assigning permissions to roles that map to job functions.
- Privileged access management reduces risk by controlling, rotating, and monitoring elevated accounts and by recording privileged activity.
- System hardening reduces attack surface by removing unused services, disabling default accounts, and changing default credentials.

Regular access reviews help prevent privilege creep, remove just-in-case access, and ensure accounts and services that are not required are disabled or removed.
### Architecture checklist for delivery teams
A project can be assessed against the following questions.

- Are confidentiality requirements defined and met for sensitive data?
- Are authentication and authorisation decisions consistent across interfaces and layers?
- Are integrity controls in place for logs, code, configurations, and critical records?
- Are availability targets defined, and are resilience measures tested against realistic failure and attack scenarios?
- Are monitoring, incident response, and recovery procedures documented and exercised?
## Access Management and Endpoint Security
### Endpoint security
Endpoint security focuses on the trustworthiness of the devices that connect to an organisation’s systems. Strong identity and access management controls, including multi-factor authentication, rely on a device that has not been compromised. A jailbroken, malware infected, or unmanaged device can bypass otherwise strong identity checks.
#### What counts as an endpoint
An endpoint is any computing device that connects to a network and exchanges information. This includes:
- Servers, including virtual and cloud hosted instances
- Desktops and laptops
- Mobile phones and tablets
- Internet of Things devices, including cameras and embedded systems in appliances and equipment
- Home devices used for work, especially where home networks connect to corporate services

Each additional endpoint increases the attack surface. Diversity of hardware and operating systems increases complexity, and complexity increases security risk.
#### Endpoint management approach
A common but weaker approach is to manage servers, user endpoints, and mobile devices with separate tools and separate administrative teams, with limited coverage for IoT. A stronger approach is unified endpoint management, where one logical console enforces consistent security baselines, deploys updates, and collects alerts across device types. Visibility and control enable faster response and better policy consistency.
#### Core endpoint controls
Endpoint security controls typically include:
- Inventory and discovery to identify known and unknown devices
- Approved hardware and software baselines, including minimum supported versions such as current release and one prior release
- Password and device lock policies, including minimum length, strength, and expiry rules where passwords are used
- Patch management for operating systems and applications
- Encryption requirements for data at rest to reduce impact of loss or theft
- Remote wipe capability, ideally selective wipe for corporate data on personal devices
- Endpoint protection and endpoint detection and response to reduce malware and support investigation
- Location tracking where appropriate, noting privacy and consent implications
- Secure disposal processes to prevent data exposure when devices are retired
#### BYOD and bring your own IT
Bring your own device programs extend to broader bring your own IT behaviours, including unsanctioned apps and cloud services. Organisations generally end up with either a well defined program or a poorly defined program. A blanket prohibition often results in unmanaged use.

A well defined program sets expectations and reduces risk:
- User consent that explains what the organisation will manage, monitor, and remediate
- Clear monitoring scope, including whether monitoring applies only to corporate data and activity
- Selective wipe rules for loss, theft, or employment exit
- Minimum supported operating system and application versions
- Required security applications where feasible, such as mobile device management or endpoint security agents
- Prohibited applications that materially increase risk
- Supported hardware models to limit unmanageable variability
- Approved services for functions such as file sharing, with controls to reduce shadow IT

Security outcomes improve when the approved path is easier than the unauthorised path.
### Identity and access management
Identity and access management is the set of processes and technologies that establish who a user is and what they are allowed to do. It is often described using four capabilities.
#### The four As
| Capability | Purpose |
|---|---|
| Administration | Creates, updates, and removes accounts and entitlements |
| Authentication | Verifies identity, answering who the user is |
| Authorisation | Determines what actions are allowed, answering what the user can do |
| Audit | Verifies that the first three were applied correctly and detects misuse |
#### Directories, store and synchronise
Identity data is stored in directories, meaning systems that hold user account information. A directory generally includes:

| Directory element | Meaning |
|---|---|
| Database | Where identity attributes are stored |
| Schema | How identity attributes are structured |
| Protocol | How systems query and update the directory, commonly LDAP, which stands for Lightweight Directory Access Protocol |

Large organisations often have multiple directories because some applications require specific identity stores. When identities cannot be held in a single enterprise directory, synchronisation becomes important.

Two common synchronisation patterns are:
- Virtual directory, which provides an index and retrieves identity attributes on demand, with optional caching
- Meta directory, which aggregates selected attributes into an enterprise store ahead of time
#### Identity governance and provisioning
Administration is often implemented through identity governance and administration, supported by role management. Role definitions map business roles to typical access needs, enabling repeatable access patterns.

Common lifecycle use cases are:
- Joiner, a new employee record triggers creation of standard accounts through workflow and approvals
- Mover, an existing user requests additional access or changes role and entitlements are updated through the same workflow
- Leaver, employment exit triggers rapid removal of access across systems, prioritising deprovisioning to reduce residual risk

A central system of record improves both provisioning and deprovisioning because it tracks which entitlements were granted and where.
#### Access management
Authentication typically uses one or more factors:

| Factor category | Examples |
|---|---|
| Something known | Password, passphrase, PIN |
| Something held | Phone, hardware token, smart card |
| Something inherent | Fingerprint, face recognition, other biometrics |

Multi-factor authentication combines two or more categories, reducing reliance on passwords. Passwordless approaches often combine a registered device with biometrics, with an emphasis on securing the endpoint itself.

Single sign on improves usability by authenticating once and then brokering access to multiple services. Security depends on strong authentication at the sign on point, especially multi-factor authentication.

Authorisation determines whether a user may perform a specific action. Modern approaches often include risk based or adaptive controls, for example:
- Additional verification for high value transactions or sensitive actions
- Restrictions based on unusual location, device state, or behaviour
- Limits based on amount or frequency that trigger step up controls
#### Privileged access management
Privileged access management focuses on administrator and root level accounts. Common failure modes include shared passwords, infrequent rotation, and weak attribution. A privileged access management system reduces these risks by:
- Enforcing individual authentication for privileged users, ideally with multi-factor authentication
- Brokering access to privileged accounts via controlled checkout and check in
- Rotating passwords automatically, often after each use
- Recording privileged sessions to support investigation and deterrence
- Improving attribution so actions can be linked to a specific administrator
#### Audit and behaviour analytics
Audit validates that administration, authentication, and authorisation controls operate as intended. Effective audit combines logging, correlation, and analytics to detect anomalies, such as rapid creation of an account, data extraction, and account deletion. User and entity behaviour analytics can assist by identifying activity patterns that deviate from normal behaviour.
#### Federation across identity domains
Federation allows one organisation to act as an identity provider while another acts as a service provider, enabling controlled access across organisational boundaries and cloud services. Federation reduces password sprawl and supports consistent policy enforcement.
### Multi-factor authentication overview
Multi-factor authentication requires two or more independent verification factors to access a system, application, or network. It increases resilience because an attacker must compromise multiple controls rather than a single credential.

Single factor authentication relies on one factor, typically a password. Two factor authentication uses two different factor categories, and multi-factor authentication uses two or more categories.

Selection of an appropriate method depends on sensitivity, threat level, access context, and user experience. Lower risk use cases may rely on two factors, while high risk environments may require additional controls, stronger device assurance, and stricter authorisation conditions.
### Key messages
- Endpoint security underpins trust in identity controls, including multi-factor authentication
- Unified endpoint management improves visibility, control, and policy consistency across device types
- Identity and access management aligns administration, authentication, authorisation, and audit
- Directories and directory synchronisation enable identity reuse across many systems
- Privileged access management reduces the highest impact risks through brokering, rotation, and monitoring
- Risk based controls and effective auditing strengthen detection and response
## Network, Application, and Data Security
### Application security and secure delivery
Application security architecture reduces the likelihood and impact of software defects that become exploitable vulnerabilities. Because software of meaningful complexity contains bugs, and some bugs create security weaknesses, application security is treated as a lifecycle concern rather than a one-off review.
#### Why application security matters
- Vulnerabilities are commonly introduced during design and coding
- Defects discovered after release usually cost more to remediate because they trigger operational work, patch rollout, customer impact, and reputational risk
- A program that detects issues earlier reduces both cost and disruption
#### From SDLC to DevSecOps
Traditional software delivery is often a linear software development life cycle, moving from design to build, to test, to release, then operate. This model encourages siloed handovers between development and operations.

DevOps replaces the linear flow with a feedback loop across build, release, deploy, operate, and improve. DevSecOps extends the DevOps loop by embedding security into each phase. Security becomes a built-in quality attribute and not a bolt-on control.

Common DevSecOps elements include:
- Security by design, including threat-aware architecture decisions before coding starts
- Shift-left controls, where security checks run as early as practical in the delivery pipeline
- Shared ownership across development, security, and operations teams
- Automation so that security keeps pace with delivery velocity
#### Secure coding foundations
Secure coding relies on repeatable practices and clear standards that developers can apply consistently, including:
- Input validation to reduce injection, buffer overflow, and parsing faults
- Authentication and authorisation patterns that avoid ad-hoc implementations
- Cryptography usage guidelines, including safe defaults and key handling
- Error handling that avoids information leakage through debug output and stack traces
- Secure code review, supported by automated checks and targeted manual review

OWASP guidance is commonly used for secure coding checklists and recurring weakness categories. The OWASP Top 10 is often used as a teaching tool and as a high-level risk communication device.
#### Third-party dependencies, Log4j, and SBOM
Modern applications depend on third-party packages and shared libraries. Even widely used components can carry high-impact vulnerabilities, as shown by the Log4j incident. Supply chain security therefore benefits from a software bill of materials (SBOM) that records:
- Component inventory and versions
- Dependency relationships
- Source and provenance where possible
- Known vulnerability exposure and patch status

An accurate SBOM supports faster identification of affected systems and faster remediation after new vulnerabilities are disclosed.
#### Vulnerability testing in the pipeline
Automated testing complements review by continuously scanning for patterns associated with weaknesses.

- Static application security testing examines source code and configuration to identify risky patterns earlier
- Dynamic application security testing probes a running service from the outside, helping find runtime and integration flaws

Both approaches are complementary because they discover different classes of issues and provide different evidence.
#### Generative AI considerations
Large language models can generate and debug code rapidly. Risk increases when generated code is not reviewed, when prompts include sensitive source code, or when outputs introduce insecure patterns. Many organisations reduce risk by treating generated code like any other contribution:
- Apply the same security standards and review expectations
- Scan and test outputs in the normal pipeline
- Use approved tools and data handling rules
- Restrict sharing of confidential source code with external services
### Layered application security architecture
Application security architecture is most effective when controls are layered across build-time, test-time, and run-time. No single tool or control provides adequate protection for modern services.
#### Build-time controls
Build-time controls aim to prevent known weaknesses from being merged or released:
- Automated code quality and security scanning in CI/CD pipelines
- Dependency scanning and policy gates to block known vulnerable versions
- Infrastructure as code scanning to detect unsafe cloud and platform configurations
- Secrets detection to reduce credential leakage in repositories
- Standard patterns for authentication, authorisation, logging, and cryptography
#### Test-time controls
Test-time controls validate behaviour in realistic environments:
- Automated dynamic scanning of web applications and APIs, including common injection and authentication faults
- Security regression tests for issues that were previously fixed
- Abuse case testing for critical workflows, such as login, account recovery, and payment flows
- Validation of security configuration, including headers, TLS settings, and session handling
#### Run-time controls
Run-time controls focus on detection and containment:
- Runtime monitoring and alerting for anomalous behaviour
- Web application firewall controls where they fit the threat model
- Container and workload hardening, including vulnerability monitoring
- Policy enforcement for workloads and service interactions, including least privilege access
- Logging that supports investigation, attribution, and accountability

Architectures that perform well at scale keep security checks largely automatic and provide fast feedback to engineering teams.
### Application security architecture principles
#### Zero trust between services
Zero trust design removes implicit trust. Service-to-service calls are authenticated and authorised, including internal calls. Mutual TLS and service identity controls can reduce the risk of lateral movement.
#### Defence in depth
Defence in depth spreads controls across layers so that a single failure does not result in compromise. Practical examples include:
- Authentication and authorisation at the API gateway
- Input validation in services
- Segmentation between tiers
- Monitoring and alerting at critical junctions
#### Observability for security
Security outcomes improve when the architecture makes risk visible. Logging, metrics, and traces support faster detection and response. Observability becomes more important as services become more distributed.
### Security architecture diagrams
Security architecture diagrams communicate trust boundaries, security zones, and data flows. They help teams identify architectural weaknesses before implementation and provide a shared view for review.
#### Typical zone model for web services
- Internet zone, untrusted traffic sources
- DMZ, public-facing services with restricted exposure
- Internal zone, sensitive services and data stores
#### Core elements in a simple web application diagram
- External users and internal administrators
- External and internal firewalls or equivalent controls
- Public web tier in the DMZ
- Application tier in the internal zone
- Database tier in the internal zone, protected from direct internet access
#### Common traffic flows and control points
- External users to web tier over HTTPS
- Web tier to application tier over restricted internal interfaces
- Application tier to database tier over a narrowly scoped database connection
- Administrative access from internal networks through controlled paths
#### Diagram annotations that add security value
- Allowed ports and protocols, such as HTTPS on 443
- Deny-by-default policies with explicit allow lists
- Where encryption is applied in transit and at rest
- Where logging occurs and where logs are stored
- Authentication and authorisation enforcement points
#### Diagram review checks
- Every cross-zone flow passes through an explicit security control
- Sensitive data stores are furthest from untrusted zones
- The purpose and protocol of each flow is labelled
- Key controls are documented and traceable to requirements
### Data security
Data security protects sensitive information across its lifecycle, including creation, storage, use, sharing, archiving, and disposal. It combines governance, discovery, protection, compliance, detection, and response.
#### Governance and classification
Governance clarifies what is sensitive, who owns it, and how it must be handled. A practical governance model includes:
- Classification levels with clear criteria and examples
- Minimum protection requirements by classification
- Data catalogues so sensitive repositories are known and maintained
- Resilience planning for recovery after loss, corruption, or ransomware

Classification is ineffective when definitions are vague or inconsistent. It improves when the organisation defines what qualifies as sensitive, and maps each level to controls.
#### Discovery and data loss prevention
Discovery identifies sensitive content across both structured and unstructured stores:
- Databases and application stores that hold structured records
- Files, email, and collaboration tools that hold unstructured content

Data loss prevention capabilities support monitoring and control of sensitive content as it moves between systems and outside the organisation. DLP tends to be most effective when paired with clear governance rules and low-noise detection patterns.
#### Protection and resilience
Core protections commonly include:
- Encryption for data at rest and in transit
- Key management with secure generation, storage, rotation, and access controls
- Strong access control integrated with identity and access management
- Backups that support recovery objectives and resilience against ransomware
- Testing of backup restore processes to confirm recoverability

Encryption without key management is fragile. Key lifecycle management matters because loss of keys can mean loss of data, and weak key protection can negate encryption.
#### Compliance and retention
Regulatory obligations vary by industry and geography. Programs usually require demonstrable controls, reporting capability, and retention policies. Effective retention keeps records for required periods and disposes of data that no longer has a valid purpose, reducing exposure and storage burden.
#### Detection and response
Prevention is not sufficient. Detection and response typically rely on:
- Monitoring and alerting on data access and movement
- Behaviour analytics to flag unusual access patterns and suspicious downloads
- Incident playbooks that guide investigation and containment
- Automation and orchestration to scale response while retaining human oversight for novel incidents

Organisations that rehearse response, maintain clear ownership, and collect usable logs generally recover faster.
### Quantum-safe cryptography
Quantum computing may weaken widely used public-key cryptography in future. Some threats are long-tail, including harvest now decrypt later scenarios, where encrypted data is collected today and decrypted later if cryptography becomes weak.

A planning approach typically includes:
- Inventory of where public-key cryptography is used, including TLS, key exchange, and certificates
- Identification of data with long confidentiality lifetimes
- Migration planning for cryptographic libraries, protocols, and key infrastructure
- Testing pathways that support staged rollout and rollback

Post-quantum approaches are often discussed in several families:
- Lattice-based schemes
- Hash-based signatures
- Code-based schemes
- Multivariate schemes
- Symmetric cryptography with suitable key sizes
### Architects’ role in incidents and failures
Systems architects support incident work by connecting events to design decisions and by improving resilience.
#### During an incident
Architectural support during response commonly includes:
- Mapping the attack path across diagrams and trust boundaries within the first day
- Identifying choke points that can limit spread and reduce blast radius
- Reviewing adjacent systems that may be exposed through lateral movement
- Recommending containment options that change connectivity and trust, not only blocklists
#### After an incident
Post-incident activities commonly include:
- Root cause analysis using structured methods such as STRIDE
- Documentation of design gaps, such as missing authentication boundaries, weak segmentation, or insufficient logging
- Architectural remediation plans that implement zero trust principles and explicit trust boundaries
- Continuous threat modelling integrated into change processes so similar gaps do not recur
#### Resilience patterns
Architectural patterns that reduce impact include:
- Default deny controls and explicit allow lists for network and service access
- Rate limiting, throttling, and circuit breaker controls for abuse resistance
- Immutable infrastructure patterns that favour rapid replacement over in-place repair
- Strong identity controls for administrators, including privileged access controls
### Security audits
Security audits assess whether controls are present, effective, and aligned with obligations. Audits also provide a baseline for continuous improvement.

Common audit stages include:
- Scope definition, including systems, processes, and regulatory requirements
- Evidence collection, including configurations, diagrams, access control records, and policies
- Vulnerability assessment, including scanning and targeted testing
- Risk analysis based on likelihood and impact
- Control evaluation for identity, data protection, and incident readiness
- Reporting with prioritised recommendations and follow-up verification

Audit value increases when recommendations are mapped to owners, timelines, and measurable outcomes.
### Network security essentials
Network security reduces exposure by controlling connectivity, limiting trust, and monitoring flows. The architecture often uses segmentation to ensure that compromise of a public-facing tier does not create direct access to sensitive systems.
#### Firewalls and filtering
Core firewall concepts include:
- Packet filtering that evaluates source, destination, and port information
- Anti-spoofing rules that block traffic claiming internal origins from external sources
- Stateful inspection that considers session context and sequence
- Application-aware inspection for common malicious patterns
#### Segmentation and DMZ patterns
Common segmentation patterns include:
- A DMZ that hosts internet-facing services
- An internal zone that hosts application services and data stores
- Restrictive rules that allow only required flows between tiers
- Separate controls for administrative access paths
#### Remote access and modern perimeter patterns
Remote access is commonly implemented through:
- VPN solutions with strong authentication and device posture checks
- Secure access service edge approaches that apply policy based on identity, device, and context
#### Monitoring and response readiness
Network monitoring supports detection of policy violations and unusual traffic patterns. Logging and alerting become more important as environments become more distributed and dynamic.
## Detection and Response
### Detection and response in security operations
Security outcomes are commonly prevention, detection, and response. Prevention aims to stop compromise. Detection aims to identify abnormal or malicious activity. Response aims to contain impact, restore services, and reduce recurrence.
#### Security Operations Centre responsibilities
A Security Operations Centre centralises monitoring, analysis, and response. It consolidates telemetry from identity, endpoint, network, application, and data controls so analysts can detect anomalies, investigate alerts, and coordinate remediation.
#### SIEM overview
A security information and event management system aggregates security-relevant data such as logs, alerts, events, and network flow information. Typical capabilities include:
- Central collection and normalisation of telemetry
- Correlation of related events across multiple sources
- Rules and analytics to prioritise alerts and assign severity
- Anomaly detection, including user behaviour analytics
- Reporting to track trends such as alert volume, resolution time, and recurring causes
#### XDR overview
Extended detection and response builds on endpoint detection and response by combining telemetry across endpoints and other controls. Common characteristics include:
- Agents or sensors close to assets for faster detection and response actions
- Central investigation workflows spanning multiple signal sources
- Federated search in some architectures so data can be queried in place rather than fully preloaded into a central store

In many environments, SIEM and XDR are complementary. SIEM often provides broad ingestion and correlation. XDR often provides deep endpoint context and response actions, and may use SIEM alerts as triggers for investigation.
### Indicators of compromise and threat intelligence
An indicator of compromise is evidence suggesting a system or network may have been breached. Common examples include:
- Suspicious network traffic such as unexpected spikes or connections to suspicious IP addresses
- Unusual system behaviour such as unexplained slowdowns, crashes, or unexpected processes
- Abnormal log activity such as repeated failed logins or logins from unusual locations
- Phishing indicators such as suspicious emails, links, or attachments
- Unrecognised or unexpected user accounts

Threat intelligence is the collection and analysis of information about threats, attacker behaviour, and indicators that can help organisations anticipate and defend against attacks. It is commonly grouped into:
- Behavioural intelligence about tactics, techniques, and patterns
- Reputational intelligence about risky IP addresses, domains, and URLs
- Raw threat data used as inputs for deeper analysis

Threat intelligence sources can include open-source feeds, commercial services, industry reports, government advisories, incident response learnings, and controlled collection methods such as honeypots.
### Security vulnerability assessment
A security vulnerability assessment is a structured, preventive review of an organisation’s technology environment. It identifies security weaknesses in networks, hosts, applications, databases, and cloud services before they are exploited. Findings are analysed and prioritised so remediation focuses on issues that present the highest business risk.
#### Core assessment approaches
1. Network-based assessment: scans network infrastructure to identify exposed services, open ports, weak protocols, misconfigured firewalls, and unmanaged or rogue devices.
2. Host-based assessment: reviews servers, workstations, and endpoints for missing patches, insecure configurations, and unnecessary services.
3. Application-based assessment: tests web and desktop applications for weaknesses such as SQL injection, cross-site scripting, and insecure API use. Static testing analyses source code. Dynamic testing evaluates running applications.
4. Cloud environment assessment: evaluates cloud configuration, storage permissions, identity and access management controls, and container security across platforms such as Amazon Web Services, Microsoft Azure, and Google Cloud Platform.
#### Common tool categories
Comprehensive vulnerability management platforms:
- Qualys VMDR
- Tenable.io
- Rapid7 InsightVM

Specialised scanners:
- Nessus
- Acunetix
- OpenVAS

Developer-focused tools:
- Snyk
- StackHawk
- Burp Suite

Some organisations also use platforms that prioritise findings using observed exploitability and continuous automated testing approaches.
#### Implementation principles
1. Asset baselining: asset discovery and inventory ensure assessments cover endpoints, virtual machines, containers, and services.
2. Risk-based prioritisation: severity scores are supplemented with exploit likelihood signals, evidence of active exploitation, asset criticality, and advisories such as the CISA Known Exploited Vulnerabilities catalogue.
3. Continuous scanning: schedules align to change rates, such as weekly for stable environments and more frequent scanning for dynamic cloud workloads.
4. Development integration: security testing is embedded into build and delivery workflows so issues are found earlier in the software lifecycle.
5. Response tiers: remediation targets are defined so high-risk issues on critical systems are addressed first and progress is measurable.
6. Measurement and optimisation: metrics such as mean time to remediate, closure rates for high-risk issues, and recurring root causes guide improvements.
### Incident response and SOAR
Incident response typically includes triage, investigation, containment, eradication, and recovery. Triage distinguishes false positives from real incidents and establishes urgency. Remediation focuses on stopping the attack, removing adversary access, patching or reconfiguring affected systems, and restoring normal operations.

Security orchestration, automation and response platforms aim to make incident handling more consistent and scalable. Typical capabilities include:
- Case management to track incidents, ownership, priority, and status
- Automated enrichment of cases with artefacts such as indicators of compromise
- Playbooks that guide investigations and response actions
- Integration with security tooling, ticketing, and monitoring systems

Automation can reduce effort for recurring scenarios. Orchestration supports partially automated workflows where analysts decide when to execute actions. Full automation is not always feasible because novel or first-of-a-kind events may require judgement before response actions are safe.
#### Breach notification considerations
When a data breach is confirmed, response teams commonly determine:
- The categories of data affected, such as names, government identifiers, or payment details
- The jurisdictions relevant to affected individuals, as notification obligations vary by country and can vary by state or territory
- Which regulators and affected parties must be notified, and within what timeframes

For example, the European Union General Data Protection Regulation provides for significant penalties for non-compliance with breach reporting obligations. Organisations can also be subject to overseas requirements if they hold data about people covered by those rules.
### Incident response frameworks
Two widely used frameworks are those published by the National Institute of Standards and Technology and by the SANS Institute.
#### NIST framework stages
- Preparation
- Detection and analysis
- Containment, eradication, and recovery
- Post-incident activity, including lessons learned and improvements
#### SANS framework stages
- Preparation
- Identification
- Containment
- Eradication
- Recovery
- Lessons learned

Both frameworks emphasise readiness, disciplined investigation, measured containment and recovery, and continuous improvement after incidents.
### Case study summary: malware in an air-gapped airport environment
A major international airport operating an air-gapped network identified multiple devices compromised by malware designed to capture and store data locally. The environment had limited visibility, mixed devices with different security levels, and minimal tolerance for downtime.

Endpoint detection and response tooling enabled reconstruction of attacker behaviour, including persistence mechanisms and attempts to collect sensitive information. The analysis indicated infection via removable media and lateral movement enabled by weak controls. Remediation included clearing infected devices and storage locations, verifying eradication through threat hunting, strengthening internal traffic control rules, separating public and operational networks, and adopting continuous endpoint monitoring with regular hunting campaigns.