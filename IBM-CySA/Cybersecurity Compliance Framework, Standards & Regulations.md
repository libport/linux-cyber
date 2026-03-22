# Cybersecurity Compliance Framework, Standards & Regulations

> [!NOTE]
> This document explains how cybersecurity compliance is managed through governance, risk, and compliance (GRC) practices that align security controls with business objectives and legal obligations. It outlines governance building blocks (policies, standards, procedures, oversight), risk management approaches, asset and change management, and the role of GRC tools. It also surveys major frameworks and standards (including NIST CSF 2.0, ISO/IEC, OWASP, ITIL, SOC reports, and ISACA), alongside key laws and privacy regimes, third-party risk, and emerging AI governance such as the EU AI Act.
## Introduction to Information Security and Compliance
### Governance, Risk and Compliance and Cybersecurity Management
Summary of how organisations coordinate governance, risk management, compliance, and core security practices to keep operations lawful, resilient, and aligned with business objectives.
### Governance, Risk and Compliance (GRC)
GRC integrates how an organisation sets direction, manages uncertainty, and proves conformance with external and internal requirements, including across information technology.

- Governance: sets policies, goals, decision rights, and accountability to support ethical and transparent operations
- Risk management: identifies and treats financial, operational, cybersecurity, and regulatory risks to protect objectives and support informed decisions
- Compliance: meets laws, regulations, standards, and internal rules through monitoring, audit evidence, and training

Benefits commonly include reduced duplication, clearer risk visibility, more consistent decision-making, stronger audit outcomes, and improved stakeholder confidence. Weak implementation, limited integration, or poor leadership support can create high cost, fragmented effort, and limited insight into threats and control gaps.
### Governance components within a GRC framework
#### Guidelines, policies, and standards
- Guidelines recommend practices to meet risk and compliance objectives
- Policies define permitted and prohibited behaviour, often covering acceptable use, information security, continuity, disaster recovery, and incident response
- Standards specify measurable criteria to make controls repeatable, such as password rules, multi-factor authentication, physical security controls like closed-circuit television and secure entry systems, and encryption requirements like TLS for data in transit and AES for data at rest
#### Procedures and oversight
Procedures turn intent into repeatable execution, such as change management, onboarding and offboarding, and operational playbooks. Governance also considers jurisdictional legal duties and industry expectations, with oversight commonly provided by boards, committees, and regulators.
### GRC tools and common use cases
GRC tools support control mapping, workflow automation, evidence capture, and reporting. Common platforms include IBM OpenPages, RSA Archer, and MetricStream.

They are commonly used for:
- Regulatory change management and gap identification
- Cybersecurity control monitoring and risk reporting
- Third-party and vendor risk assessment
- Audit planning, evidence collection, issue tracking, and reporting
### NIST Cybersecurity Framework (CSF) 2.0
The NIST CSF 2.0 organises cybersecurity outcomes into six connected functions.

- Govern: define strategy, roles, policies, oversight, and supply chain risk expectations
- Identify: understand assets, dependencies, and the risk context
- Protect: implement safeguards such as access controls, training, and data security
- Detect: monitor for events and indicators of compromise
- Respond: contain incidents, communicate impacts, and coordinate remediation
- Recover: restore services and strengthen resilience after disruption

Implementation tiers describe maturity.

- Tier 1, Partial: largely ad hoc and reactive practices
- Tier 2, Risk Informed: risk-aware decisions with uneven adoption
- Tier 3, Repeatable: approved policies and consistent processes
- Tier 4, Adaptive: continuous improvement and rapid adjustment to changing threats

Alignment typically uses a cycle of posture assessment and gap analysis, senior leadership sponsorship, tailored control design, stakeholder and supplier collaboration, and continuous improvement. A common applied pattern is to establish governance and policy, catalogue critical assets, strengthen protection with multi-factor authentication and encryption, implement detection monitoring, use an incident response plan, and practise recovery and communications.
### Elements of effective security compliance
External drivers include government regulation, legal exposure, sector standards, local and national context, and global factors such as geopolitics, cross-border operations, and supply chain risk. Organisations must adjust policies and procedures to meet the requirements of each operating jurisdiction while keeping a consistent internal control approach.

Consequences of non-compliance can include fines, sanctions, reputational harm, loss of licences, and contractual penalties.

Monitoring and assurance commonly rely on due diligence, attestation, incident acknowledgment, internal audits and training, independent external reviews, and automation for tasks such as vulnerability scanning and patch management.
### Standardised processes, automation, and change control
Standardised cybersecurity processes are documented and applied consistently across teams to improve reliability and compliance.

- Network configuration baselines to reduce misconfiguration risk and speed troubleshooting
- Identity and access management using centralised accounts, single sign-on, and multi-factor authentication
- Vulnerability and patch management using regular scanning and prompt patching
- Incident response and recovery using tested procedures and post-incident review
- Continuous improvement through audits, feedback loops, and training

Automation performs tasks with minimal human effort. Orchestration coordinates automated steps across systems. In practice this supports user provisioning, role-based access control, access changes during onboarding and offboarding, ticketing and escalation workflows, and integration through application programming interfaces. Benefits include faster delivery, fewer manual errors, consistent baselines, secure scaling, improved service reliability, and better use of limited staffing.

Change management controls system modifications through approval, impact analysis, testing, backout planning, maintenance windows, and strong documentation. Technical impacts commonly considered include allow and deny rules, restricted activities during rollout, downtime, service restarts, legacy compatibility, and dependencies across systems.
### Asset management
Asset management tracks assets from acquisition to disposal to support cost control, security, and compliance.

- Acquisition and procurement: define need, evaluate options, confirm compatibility, and align to budgets and strategy
- Monitoring and tracking: maintain inventory accuracy, deter loss, optimise use, and support audits through tagging, inventory systems, reconciliation, and lifecycle tracking
- Disposal and decommissioning: protect the environment and data by securely erasing or destroying storage media and meeting electronic waste obligations
## Foundations of IT Service Management and Risk Governance
### ITIL and IT service management
ITIL, originally known as the Information Technology Infrastructure Library, is a set of guidance and practices for planning, delivering, supporting and continually improving IT services so they meet business needs. It began in the 1980s in the United Kingdom public sector and has since been adopted internationally as a service management framework.

IT service management (ITSM) is the broader discipline covering the policies, processes and activities used to design, run and improve IT services. ITIL is one commonly used set of ITSM practices.
#### Configuration management database
A configuration management database (CMDB) is a managed data store that records key information about configuration items (CIs) and the relationships between them. It supports a shared view of the IT environment, including:
- Physical assets such as servers, end user devices, network equipment and peripherals
- Non physical assets such as software versions and licences, documentation, service agreements and configuration baselines
- Relationships and dependencies, including which components support which services and how changes affect upstream and downstream systems

Using a CMDB improves incident response and problem resolution by accelerating identification of affected components. It also supports change and release planning, asset lifecycle management, compliance reporting and security risk identification.
### ITIL processes and practices
In ITIL v3, service management is commonly described through five lifecycle stages:
1. Service strategy, which defines value, target customers, service portfolios, funding and high level risk management
2. Service design, which designs new or changed services and the controls that make them reliable, scalable and cost effective, including service levels, capacity, availability, continuity, information security and supplier arrangements
3. Service transition, which moves changes into operation through controlled change management, service asset and configuration management, release and deployment management and knowledge management
4. Service operation, which delivers and supports services day to day through the service desk and operational processes such as incident, problem, event, request, and access management
5. Continual service improvement, which uses measurement and feedback to prioritise and deliver ongoing improvements, often applying the Plan-Do-Check-Act cycle

ITIL 4 reframes this into a Service Value System and a set of practices rather than a strict lifecycle, but many organisations still use the lifecycle language to organise work.
### Implementing ITIL aligned processes
Successful implementation is treated as organisational change rather than a documentation exercise. Common practice includes:
- Assessing current process maturity to identify gaps and priorities
- Securing executive sponsorship to enable resourcing and decision making
- Defining scope, objectives and measures that align to business outcomes
- Building a phased roadmap, piloting before wider rollout, then iterating based on evidence
- Training staff and stakeholders so roles, responsibilities and expected behaviours are understood
- Selecting supporting ITSM tools that fit the operating model and reporting needs
- Establishing key performance indicators and regular reviews to drive continual improvement
- Maintaining clear, consistent communication with stakeholders
### Cybersecurity risk management
Risk management identifies, assesses and controls risks that could affect an organisation’s objectives, including financial, legal, operational and security risks. In cybersecurity, the focus is on protecting information assets and maintaining business continuity.

A practical flow includes:
1. Risk identification, using expert input, historical incidents, tooling and structured methods such as SWOT analysis
2. Risk assessment, estimating likelihood and impact using ad hoc, recurring, one time or continuous approaches
3. Risk prioritisation, ranking risks to direct effort to the most consequential exposures
4. Risk tolerance and risk appetite, defining the level of risk the organisation will accept to achieve its goals
5. Risk treatment, selecting an approach such as transfer, acceptance, avoidance or mitigation

Risk analysis is commonly:
- Qualitative, which categorises risks by severity and likelihood without relying on precise numbers
- Quantitative, which estimates financial exposure using measures such as exposure factor, single loss expectancy, annualised rate of occurrence and annualised loss expectancy
### Third-party and supply chain risk
Vendor assessment evaluates third parties against security and compliance expectations. Typical components include penetration testing, contractual rights to audit, evidence of internal audits and independent security assessments.

Supply chain analysis focuses on resilience and integrity through vendor selection, due diligence on financial and governance stability, and the management of conflicts of interest.

Common agreement types used to formalise expectations include service level agreements, master service agreements, statements of work or work orders, non disclosure agreements, memoranda of agreement, memoranda of understanding and business partner agreements.

Ongoing vendor management relies on performance monitoring, structured questionnaires and clear rules of engagement for communication, review cadence and responses to non compliance.
### AI ethics and the EU AI Act
AI is widely embedded in everyday systems such as predictive text, recommendation engines, biometric identification, navigation and smart home devices. Key ethical concerns include privacy and data security, algorithmic bias and unfair outcomes, surveillance and consent, limited transparency, and workforce disruption from automation.

Principles used in practice often emphasise AI that augments human decision making, clear data governance and transparent, explainable systems. Trustworthiness can be assessed through pillars such as explainability, fairness, robustness, transparency and privacy. Implementation typically includes governance rules, technical guardrails, diverse training data and bias evaluation tooling.

The European Union’s Artificial Intelligence Act, adopted in 2024, uses a risk based approach. It prohibits certain harmful uses, sets transparency requirements for some AI outputs, and imposes stronger obligations on high risk systems, including human oversight, record keeping, conformity assessments and registration in an EU database. Application dates are phased, so organisations operating in the EU usually plan compliance work across the transition period.
## Understanding Cybersecurity Laws and Regulations
### Cybersecurity law and service management summary
Cybersecurity laws and standards set expectations for preventing cybercrime, protecting personal information, and operating digital services.
#### Core United States laws
- Computer Fraud and Abuse Act 1986: targets unauthorised access to computers and misuse of access
- Electronic Communications Privacy Act 1986: regulates interception and access to electronic communications, including stored data
- Health Insurance Portability and Accountability Act 1996: privacy and security obligations for protected health information
- Children’s Online Privacy Protection Act 1998: parental consent and notice for collection of data from children under 13
- Identity Theft and Assumption Deterrence Act 1998: criminalises identity theft
- Gramm-Leach-Bliley Act 1999: safeguards and transparency for customer information in financial services
- Federal Information Security Management Act (FISMA) 2002: information security program requirements for federal agencies and key contractors
- Homeland Security Act 2002 and Cybersecurity Information Sharing Act of 2015: coordination and voluntary sharing of threat indicators
- USA PATRIOT Act 2001 and USA FREEDOM Act 2015: surveillance authorities and limits on bulk collection
- Sarbanes-Oxley Act 2002 and consumer protection laws: controls for records and deceptive online conduct
#### HIPAA Security Rule essentials
- Privacy Rule: governs permitted use and disclosure of protected health information
- Security Rule: protects electronic protected health information using administrative, physical, and technical safeguards
- Covered entities and business associates assess risk, control access, and respond to incidents
- HHS provides a Security Risk Assessment Tool and NIST publishes guidance and checklists for implementation
#### Practical HIPAA controls
- Conduct risk assessments routinely and after major change
- Use role based access, logging, and encryption where feasible
- Train staff and maintain an incident response plan
- Patch systems and review alerts and logs
### Global privacy and cybersecurity frameworks
- General Data Protection Regulation: EU and EEA rules for lawful processing, data subject rights, privacy by design, and breach notification
- Network and Information Systems Directive and NIS2: EU cybersecurity duties for critical and important sectors
- California Consumer Privacy Act: consumer privacy rights and transparency requirements
- Data Protection Act 2018 and UK GDPR: United Kingdom privacy framework
- Personal Information Protection and Electronic Documents Act: Canadian private sector privacy law
- Privacy Act 1988: Australian privacy framework based on the Australian Privacy Principles
- Cybersecurity Law of the People’s Republic of China and Information Technology Act 2000: examples of national regimes covering security and cyber offences
- PCI DSS: payment card security standard for organisations that handle card data
#### Cross jurisdiction compliance practices
- Map data flows and apply the strictest applicable requirements where practical
- Build a common baseline, then add local controls and supplier contractual requirements
- Audit and update controls as threats and laws change
### ITIL, risk management, and third parties
ITIL is a set of IT service management practices used to align IT services with business needs. A configuration management database records configuration items and their relationships, supporting faster incident diagnosis, change planning, asset lifecycle management, and compliance.

Cybersecurity risk management typically includes identification, assessment, prioritisation, appetite and tolerance setting, and treatment through avoidance, mitigation, transfer, or acceptance. Analysis may be qualitative or quantitative, using measures such as exposure factor, single loss expectancy, and annualised loss expectancy.

Third-party risk management includes vendor due diligence, security testing where appropriate, contractual audit rights, evidence of independent assessments, and ongoing monitoring. Common agreements include service level agreements, master service agreements, statements of work, and non disclosure agreements.
### AI ethics and the EU AI Act
AI can improve productivity, but it also creates risks including privacy loss, bias, opaque decisions, surveillance, and job displacement. Controls include human oversight, transparent data handling, diverse training data, bias testing, and guardrails that prevent harmful use.

The European Union Artificial Intelligence Act uses a risk based model. It bans certain harmful uses, requires transparency for some AI generated content, and imposes stronger obligations on high risk systems, including documentation, oversight, conformity assessment, and registration where required.
## Understanding Cybersecurity Standards and Audits
### Industry standards in IT
Information technology standards support interoperability, security and consistent performance across systems. Standards are typically developed by recognised bodies and industry groups and are applied across hardware, networks, software and security controls.
#### Hardware standards
Hardware standards help devices work together and meet safety and performance expectations.

- Physical dimensions and form factors for components and connectors
- Power requirements such as voltage and current ranges
- Data interfaces and connectors such as USB, SATA and HDMI
- Electromagnetic compatibility to limit interference and improve resilience
- Safety requirements to reduce electrical, thermal and mechanical risks
#### Communication standards
Communication standards enable reliable data exchange across diverse systems and networks.

- Network and application protocols such as TCP/IP, HTTP and FTP
- Bandwidth and speed definitions to manage throughput and avoid bottlenecks
- Security requirements such as TLS for encrypted sessions and WPA2 or WPA3 for Wi-Fi
- Wireless specifications such as IEEE 802.11 and Bluetooth profiles
- Quality of service rules to prioritise time critical traffic such as voice and video
#### Security standards
Security standards establish baseline controls for protecting confidentiality, integrity and availability.

- Cryptography for data at rest and in transit such as AES and TLS
- Authentication and authorisation patterns such as OAuth, OpenID Connect and SAML
- Integrity controls such as hashing with SHA-256 and SHA-3
- Security testing approaches including vulnerability scanning and penetration testing
- Incident handling guidance covering detection, analysis, containment, eradication and recovery
- Governance and assurance frameworks such as ISO/IEC 27001 and PCI DSS, plus privacy regulation such as the EU GDPR
#### Software standards
Software standards improve quality, security and compatibility across development and deployment.

- Coding conventions that promote readability and maintainability
- API specifications that enable service interoperability
- Secure development practices for encryption, authentication and safe data handling
- Quality assurance methods for testing, benchmarking and defect management
- Standard data formats such as JSON, XML and HTML
- Compatibility expectations across platforms and versions
- Accessibility guidance to support people with disability and legal obligations
### OWASP resources
The Open Web Application Security Project is a global non profit community that publishes free application security resources.

- OWASP Top 10 outlines common web application risks and mitigation priorities
- Application Security Verification Standard defines security requirements by assurance level
- Mobile Application Security Verification Standard defines a baseline for mobile app security
- Software Assurance Maturity Model guides incremental improvement of secure development practices
- OWASP Testing Guide provides a structured approach to web application security testing
- OWASP Cheat Sheet Series offers short, practical secure coding guidance
### NIST, ISO/IEC and IEEE in cybersecurity
#### NIST
The National Institute of Standards and Technology is a US government agency that grew from the National Bureau of Standards established in 1901 and was renamed in 1988. It publishes widely used cybersecurity guidance, including the NIST Cybersecurity Framework for managing security risk.

Key NIST publications commonly used in practice include:
- FIPS 140-3 for cryptographic module security requirements
- SP 800-53 for security and privacy controls
- SP 800-63 for digital identity guidelines
- SP 800-37 for applying the Risk Management Framework
- SP 800-30 for conducting risk assessments
- SP 800-61 for incident handling
- SP 800-88 for media sanitisation
#### ISO/IEC
The International Organization for Standardization, working with the International Electrotechnical Commission, publishes international standards. The ISO/IEC 27000 family supports an information security management system that addresses people, process and technology.

Commonly referenced standards include:
- ISO/IEC 27001 for establishing and improving an ISMS
- ISO/IEC 27002 for control selection guidance
- ISO/IEC 27005 for information security risk management
- ISO/IEC 27017 and 27018 for cloud security and cloud privacy controls
- ISO/IEC 27035 for incident management
- ISO/IEC 27701 for privacy information management
#### IEEE
The Institute of Electrical and Electronics Engineers develops technical standards used across networking, computing and critical infrastructure.

Examples with cybersecurity relevance include:
- IEEE 802.11 for Wi-Fi
- IEEE 802.1X for network access control and device authentication
- IEEE 802.3 for Ethernet
- IEEE 1619 for encryption on block storage devices
- IEEE 1686 for security capabilities of intelligent electronic devices in critical infrastructure contexts
### Security control audits and assurance
Auditing and assurance activities provide evidence that controls are designed appropriately and operating effectively.

- Internal audits review internal controls, methods and procedures to identify weaknesses and recommend improvements
- Compliance audits assess adherence to laws, regulations and contractual requirements
- Audit committees oversee audit independence, financial reporting integrity and control effectiveness
- Self assessments identify gaps before formal audits and support continual improvement
- Attestation provides third-party validation that stated controls and practices align with defined criteria
- External assessments by independent auditors or regulators provide objective assurance to stakeholders

Penetration testing simulates attacks to identify exploitable weaknesses.

- Physical testing examines controls that prevent unauthorised physical access
- Red teaming tests detection and response under realistic attack scenarios
- Blue teaming strengthens defensive controls in a known environment
- Purple teaming combines red and blue efforts to accelerate learning and remediation
- Passive reconnaissance collects information without direct interaction
- Active reconnaissance interacts with systems to identify hosts, services and exposures
### Performing a security audit
A security audit is a structured review of an organisation’s security architecture, controls and compliance position.

- Define scope covering systems, sites, processes and data
- Review policies, access controls, incident response and legal obligations
- Collect evidence such as configurations, network diagrams, access control records and documentation
- Identify vulnerabilities using scans, configuration reviews and penetration tests
- Analyse risk by estimating likelihood, impact and business consequences
- Evaluate control operation for access management, data protection and incident response
- Report findings and recommendations, then follow up to verify remediation
### ISACA frameworks
ISACA is a global association focused on IT governance and assurance. Its frameworks support auditing and governance decisions.

- COBIT aligns IT governance and management with enterprise objectives and stakeholder needs
- Risk IT describes structured IT risk identification, evaluation, response and monitoring
- Val IT supports value realisation from IT investments across the investment lifecycle
### SOC reports
System and Organisation Controls reports are issued under the AICPA SOC reporting framework to provide assurance over controls at service organisations.

- SOC 1 covers controls relevant to internal control over financial reporting
- SOC 2 reports on Trust Services Criteria covering security, availability, processing integrity, confidentiality and privacy
- SOC 3 provides a high level summary of SOC 2 style controls for public distribution

Typical SOC uses include cloud services assurance, vendor due diligence, regulated industry requirements and evidence of control maturity for customers and partners.