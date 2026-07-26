# Cybersecurity Compliance Frameworks, Standards, and Regulations
> [!NOTE]
> Effective cybersecurity compliance integrates governance, risk management, service management, technical standards, legal obligations, and independent assurance into a coherent operating model.
## Governance, risk, and compliance
### Governance, risk, and compliance operating model
Governance, risk, and compliance (GRC) integrates the way an organisation sets direction, manages uncertainty, and demonstrates compliance with internal and external requirements.

- Governance establishes objectives, decision rights, accountability, oversight, and ethical expectations
- Risk management identifies, analyses, evaluates, treats, monitors, and communicates risks that could affect organisational objectives
- Compliance identifies applicable obligations, implements controls, monitors performance, and retains evidence of conformance

An effective GRC model reduces duplicated work, improves risk visibility, supports consistent decisions, strengthens assurance, and increases stakeholder confidence. Fragmented ownership, weak leadership support, poor data quality, and disconnected tools can increase cost and obscure control gaps.
### Governance instruments and oversight
Organisations use several governance instruments for different purposes.

- Policies state mandatory principles, responsibilities, and rules
- Standards translate policies into specific, measurable, and mandatory requirements
- Procedures define the steps, roles, records, and escalation paths needed to perform work consistently
- Guidelines recommend flexible practices where circumstances may require judgement

Policies commonly address acceptable use, information security, business continuity, disaster recovery, incident response, data handling, and third-party access. Supporting standards may specify authentication, secure configuration, physical access, logging, encryption, cryptographic key management, and retention requirements.

Boards and executives set risk appetite and accountability. Management committees coordinate implementation. Control owners operate and monitor controls. Internal audit provides independent assurance, while regulators and external practitioners exercise authority within their mandates.
### GRC platforms
GRC platforms support obligation registers, control mapping, workflow automation, evidence collection, issue management, and reporting. Examples include IBM OpenPages, Archer, and MetricStream.

Organisations commonly use these platforms for:
- Regulatory change management and compliance gap analysis
- Cybersecurity control monitoring and risk reporting
- Third-party and supplier risk management
- Audit planning, evidence collection, finding management, and remediation tracking

A platform supports governance but does not replace accountable decisions, reliable evidence, or effective controls.
## NIST Cybersecurity Framework 2.0
The NIST Cybersecurity Framework (CSF) 2.0 organises cybersecurity outcomes into six connected Functions.

- Govern: establish strategy, policy, roles, oversight, and cybersecurity supply chain expectations
- Identify: understand assets, dependencies, vulnerabilities, threats, and the risk context
- Protect: apply safeguards such as identity management, access control, awareness, platform security, and data security
- Detect: monitor for anomalies, indicators of compromise, and other adverse events
- Respond: manage incidents, analyse effects, communicate, contain activity, and coordinate mitigation
- Recover: restore assets and operations, communicate recovery status, and incorporate lessons learned

Organisations use a Current Profile to describe present outcomes and a Target Profile to define selected outcomes. The gap between the profiles helps leaders prioritise actions. The CSF describes outcomes rather than prescribing technologies, controls, or a fixed implementation sequence.

Implementation Tiers characterise the rigour of an organisation's cybersecurity risk governance and management practices. They are not maturity levels, and the highest Tier is not automatically the appropriate target.

- Tier 1 - Partial: practices are largely informal, ad hoc, and reactive
- Tier 2 - Risk Informed: leaders understand risk, but practices remain inconsistent across the organisation
- Tier 3 - Repeatable: approved policies and organisation-wide practices support consistent risk management
- Tier 4 - Adaptive: the organisation uses lessons, indicators, and changing conditions to improve and adjust its practices

Leaders select an appropriate Tier by considering objectives, legal duties, supply chain requirements, acceptable risk, and resources. They can map CSF outcomes to detailed controls through Informative References.
## Compliance operations
### Obligations, monitoring, and assurance
Laws, regulations, licences, contracts, industry rules, internal policies, and operating jurisdictions can all create compliance obligations. Cross-border operations, geopolitical conditions, and supply chain dependencies can change the applicable duties and risk profile.

Non-compliance may lead to enforcement action, fines, litigation, contractual remedies, loss of a licence or market access, operational disruption, and reputational harm. Compliance with a requirement does not establish that every relevant security risk is controlled, and strong security does not establish compliance with every applicable obligation.

Organisations maintain compliance through:
- An obligation register linked to responsible owners, controls, evidence, and review dates
- Due diligence, control self-assessments, and management attestations
- Internal audit and independent external assurance
- Incident reporting and regulatory or contractual notification
- Workforce education, exercises, and role-specific training
- Continuous control monitoring and periodic control testing
- Issue management, corrective action, and remediation validation

Automation can collect evidence, test configurations, scan for vulnerabilities, and track patch status. These activities support monitoring and control operation, but they do not provide independent assurance by themselves.
### Standardised processes, automation, and change control
Organisations document and apply standardised processes to improve reliability, traceability, and compliance.

- Network and cloud configuration baselines reduce misconfiguration and accelerate investigation
- Identity and access management centralises accounts, single sign-on, multi-factor authentication, and access review
- Vulnerability and patch management identifies exposures, prioritises remediation by risk, and verifies completion
- Incident response and recovery use tested procedures, defined communications, exercises, and post-incident review
- Continual improvement uses metrics, audit results, lessons learned, feedback, and training

Automation performs defined tasks with limited human intervention. Orchestration coordinates automated tasks across systems. Organisations use both for provisioning, role-based access, onboarding and offboarding, ticketing, escalation, evidence collection, and integration through application programming interfaces.

Change control governs modifications through authorisation, impact and security analysis, testing, implementation planning, rollback planning, maintenance windows, segregation of duties, and documentation. Reviewers consider service interruption, rule changes, restricted activities, restarts, legacy compatibility, dependencies, monitoring, and recovery requirements before approval.
### Asset and configuration management
Asset management identifies and governs assets throughout their lifecycle. Cybersecurity asset management covers data, hardware, software, systems, facilities, services, people, and suppliers according to their importance and risk.

- Acquisition: the organisation defines the need, evaluates security and compatibility, assigns ownership, and records the asset
- Operation and maintenance: owners classify the asset, manage access, monitor status, maintain licences and warranties, patch components, and reconcile inventories
- Reassignment: custodians update ownership, access, configuration, location, and records when the asset changes use
- Disposal: owners remove access, retain required records, sanitise media, dispose of electronic waste responsibly, and confirm completion

For media reuse or disposal, organisations select clear, purge, or destroy methods according to information sensitivity, media type, and intended destination. Deleting files or performing an ordinary erase does not necessarily sanitise media.

A configuration management database (CMDB) records selected configuration items and their relationships. Configuration items can include physical devices, software, cloud resources, documentation, service agreements, and configuration baselines. A CMDB supports incident, problem, change, release, risk, and service-impact analysis.

A CMDB does not replace an asset register, financial record, software licence repository, or document management system. An asset register tracks ownership, financial, contractual, and lifecycle information, while a CMDB focuses on the configuration and relationships needed to deliver products and services. Integrated service management tools may link these records.
## IT service and cybersecurity risk management
### ITIL and IT service management
IT service management (ITSM) covers the policies, practices, roles, and activities used to plan, deliver, support, and improve IT services. ITIL is one framework within that broader discipline.

ITIL began as the Information Technology Infrastructure Library in the United Kingdom public sector in the late 1980s. ITIL now provides guidance for digital product and service management. ITIL Version 5 entered a phased release in 2026. It retains established value-system concepts and guiding principles, and adds integrated product and service lifecycle guidance, digital experience, and support for AI-enabled environments.

The ITIL Value System connects guiding principles, governance, value-chain activities, management practices, and continual improvement. Four dimensions support a balanced operating model:
- Organisations and people
- Information and technology
- Partners and suppliers
- Value streams and processes

ITIL v3 used five service lifecycle stages: service strategy, service design, service transition, service operation, and continual service improvement. ITIL 4 replaced the strict lifecycle with a Service Value System and management practices. ITIL Version 5 retains a value-system approach and adds unified lifecycle guidance for digital products and services. Legacy operating models may still use v3 terminology, but that terminology does not describe the current structure.

Organisations implement ITIL-aligned practices by:
- Assessing current capability, pain points, and priorities
- Securing executive sponsorship, resources, and decision authority
- Defining scope, outcomes, responsibilities, measures, and interfaces
- Building a phased roadmap, piloting changes, and improving them from evidence
- Training staff and stakeholders on expected roles and behaviours
- Selecting tools that fit the operating model, integrations, and reporting needs
- Reviewing performance and communicating results to stakeholders
### Cybersecurity risk management
Cybersecurity risk management identifies, assesses, responds to, monitors, and communicates risks that could affect objectives, operations, assets, individuals, other organisations, and stakeholders.

A practical process includes:
1. The organisation establishes context, scope, criteria, ownership, and decision authority.
2. Risk owners identify assets, dependencies, threat sources, threat events, vulnerabilities, predisposing conditions, existing controls, and legal requirements.
3. Analysts estimate likelihood and impact, determine risk, record assumptions and uncertainty, and prioritise the results.
4. Leaders select a response such as acceptance, avoidance, mitigation, sharing, or transfer, then approve actions and residual risk.
5. Control owners implement the response and monitor risk indicators, control performance, environmental changes, and emerging threats.
6. Risk owners communicate results, review decisions at defined intervals and after significant change, and update records.

Risk appetite describes the types and amount of risk an organisation broadly accepts while pursuing value. Risk tolerance sets acceptable boundaries for a specific objective or outcome.

Qualitative analysis uses defined categories for likelihood, impact, and severity. Quantitative analysis uses numerical estimates and should expose assumptions and uncertainty. One loss-estimation method uses these measures:
- Exposure factor: the estimated proportion of asset value lost in one event
- Single loss expectancy: asset value x exposure factor
- Annualised rate of occurrence: the estimated number of events per year
- Annualised loss expectancy: single loss expectancy x annualised rate of occurrence
### Third-party and supply chain risk
Third-party due diligence examines the service, data access, dependencies, security practices, incident history, contract terms, vulnerability management, recovery capability, and relevant independent assurance. An organisation reviews penetration-test reports or conducts testing only when the supplier explicitly authorises the scope, timing, method, and rules of engagement.

Cybersecurity supply chain risk management addresses products and services throughout design, development, integration, deployment, operation, and disposal. It considers malicious functionality, counterfeit components, vulnerable products, poor development or manufacturing practices, concentration risk, subprocessor risk, fourth-party dependencies, and limited visibility into upstream suppliers.

Contracts allocate responsibilities and establish enforceable requirements. Common instruments include master services agreements, service-level agreements, statements of work, work orders, non-disclosure agreements, data processing agreements, business associate agreements where HIPAA applies, memoranda of agreement, and memoranda of understanding.

Ongoing management includes performance monitoring, control reviews, assurance reports, incident notification, vulnerability disclosure, change notification, resilience testing, remediation tracking, exit planning, and secure data return or destruction.
## AI governance and the EU AI Act
AI systems can improve productivity and decision-making, but they can also create risks involving privacy, security, discrimination, surveillance, opaque decisions, human autonomy, safety, and employment.

Trustworthy AI practices address human agency and oversight, technical robustness and safety, privacy and data governance, transparency, diversity and fairness, social and environmental wellbeing, and accountability. Organisations assign responsibility, assess risks and effects before deployment, maintain oversight proportionate to risk, document data provenance and system limitations, test performance across relevant groups and operating conditions, monitor systems after deployment, and provide routes for review and redress.

Representative, fit-for-purpose data can reduce some risks, but dataset composition alone cannot establish fairness. Organisations also test outcomes, controls, security, robustness, and human oversight.

Regulation (EU) 2024/1689, the Artificial Intelligence Act, uses a risk-based framework. It prohibits specified AI practices, regulates high-risk AI systems, imposes transparency duties on specified systems and synthetic content, and sets separate obligations for providers of general-purpose AI models.

Requirements for high-risk systems include risk management, data governance, technical documentation, logging, information for deployers, human oversight, accuracy, robustness, and cybersecurity. Providers complete the applicable conformity assessment and register systems when the Act requires it. Deployers have separate obligations for use, oversight, monitoring, information, and impact assessment in specified cases.

The Act applies in phases:
- 1 August 2024: the Act entered into force
- 2 February 2025: prohibitions, definitions, and AI literacy duties began to apply
- 2 August 2025: governance and general-purpose AI obligations began to apply
- 2 August 2026: Article 50 transparency duties and most other applicable rules begin to apply
- 2 December 2027: high-risk rules for Annex III systems begin to apply under the amended timetable
- 2 August 2028: high-risk rules for systems embedded in products covered by Annex I begin to apply under the amended timetable
## Cybersecurity laws and regulations
Laws impose binding duties within their scope. Their application depends on jurisdiction, entity type, activity, sector, data, system, and contractual role. Regulations and implementing rules provide more detailed obligations. Standards and frameworks remain voluntary unless legislation, regulation, contract, market rules, certification, or internal policy incorporates them.
### Core United States laws
- Computer Fraud and Abuse Act 1986: criminalises specified conduct involving access to protected computers without authorisation or beyond authorised areas, together with related fraud, damage, extortion, and trafficking offences
- Electronic Communications Privacy Act 1986: amended federal wiretap law and added the Stored Communications Act, which regulate interception of electronic communications and access to stored communications and subscriber records
- Health Insurance Portability and Accountability Act 1996 and its implementing rules: the Privacy Rule governs protected health information, while the Security Rule requires safeguards for electronic protected health information
- Children's Online Privacy Protection Act 1998 and the COPPA Rule: apply to operators of websites and online services directed to children under 13, and to other operators that know they collect personal information online from a child under 13
- Identity Theft and Assumption Deterrence Act 1998: establishes federal offences involving the knowing transfer, possession, or use of another person's means of identification without lawful authority and with specified unlawful intent
- Gramm-Leach-Bliley Act 1999: requires covered financial institutions to provide privacy notices and protect customer information under applicable privacy and safeguards rules
- Federal Information Security Modernization Act 2014: updated the 2002 FISMA framework and requires federal agencies to maintain agency-wide information security programs for federal information and systems, including systems operated on an agency's behalf
- Homeland Security Act 2002: established the Department of Homeland Security and assigned federal homeland security and critical infrastructure coordination functions
- Cybersecurity Information Sharing Act 2015: provides a voluntary, conditional framework for sharing cyber threat indicators and defensive measures. Its current authority ends on 30 September 2026 unless Congress extends it
- USA PATRIOT Act 2001 and USA FREEDOM Act 2015: changed national security surveillance, records, transparency, and court procedures. The temporary section 215 authority and related provisions expired on 15 March 2020
- Sarbanes-Oxley Act 2002: requires covered issuers to assess and report on internal control over financial reporting, and includes record preservation and anti-tampering provisions
- Federal Trade Commission Act: prohibits unfair or deceptive acts or practices in or affecting commerce and supports federal privacy and data security enforcement
### HIPAA Security Rule
The HIPAA Privacy Rule governs uses and disclosures of protected health information in any form by regulated entities and establishes individual rights. The Security Rule applies to electronic protected health information and requires administrative, physical, and technical safeguards.

Covered entities and business associates must:
- Conduct an accurate and thorough risk analysis
- Apply reasonable and appropriate measures to manage identified risks
- Control access, review system activity, and respond to security incidents
- Evaluate safeguards periodically and after environmental or operational changes
- Maintain workforce training, incident procedures, contingency plans, patch management, records review, and remediation processes proportionate to risk

The Security Rule treats encryption as an addressable implementation specification. A regulated entity implements it when reasonable and appropriate. If it is not reasonable and appropriate, the entity documents its assessment and implements an equivalent alternative when reasonable and appropriate. Addressable does not mean optional without analysis and documentation.

The United States Department of Health and Human Services and the Office of the National Coordinator for Health Information Technology provide a Security Risk Assessment Tool. NIST SP 800-66 Rev. 2 provides voluntary implementation guidance. The Security Rule amendments proposed in December 2024 do not form part of the current rule unless the rulemaking process finalises them.
### Global privacy and cybersecurity regimes
- General Data Protection Regulation: governs lawful personal data processing, individual rights, data protection by design and by default, security, and qualified breach notification across the EU and EEA, with specified extraterritorial reach
- NIS2 Directive: required EU Member States to transpose cybersecurity risk-management, governance, and incident-reporting duties for covered essential and important entities. It repealed the original NIS Directive from 18 October 2024
- California Consumer Privacy Act, as amended by the California Privacy Rights Act: gives California consumers rights to know, delete, correct, opt out of sale or sharing, and limit specified uses of sensitive personal information, while imposing duties on covered businesses
- UK GDPR and Data Protection Act 2018, as amended by the Data (Use and Access) Act 2025: govern personal data processing in the United Kingdom
- Personal Information Protection and Electronic Documents Act: governs private-sector handling of personal information in Canadian commercial activities within federal scope, subject to exemptions for substantially similar provincial laws
- Privacy Act 1988: regulates personal information handling by Australian Privacy Principle entities and includes the Notifiable Data Breaches scheme
- Security of Critical Infrastructure Act 2018: imposes risk-management, registration, incident-reporting, and other obligations on specified Australian critical infrastructure entities and assets
- Cybersecurity Law of the People's Republic of China: regulates network operations, critical information infrastructure, network information security, monitoring, and incident response. Amendments effective from 1 January 2026 strengthen penalties and add provisions concerning artificial intelligence
- Information Technology Act 2000: provides India's core electronic transactions, intermediary, cybersecurity, and cyber-offence framework
- Digital Personal Data Protection Act 2023 and Digital Personal Data Protection Rules 2025: establish India's digital personal data regime through phased commencement
### Cross-jurisdictional compliance
Organisations operating across jurisdictions:
- Identify each legal entity, establishment, regulated activity, service, system, data category, individual, and supplier relationship
- Map data flows, storage locations, transfers, subprocessors, and access paths
- Record each obligation, applicability test, regulator, owner, control, evidence source, and deadline
- Build a compatible common control baseline and add jurisdiction-specific overlays
- Analyse conflicts, localisation rules, transfer restrictions, secrecy duties, and competing notification requirements
- Maintain incident decision trees for regulatory, contractual, insurer, customer, and law-enforcement notifications
- Review legal developments, contracts, control performance, and operating changes regularly

Applying the strictest requirement as a universal rule can create conflicts or unnecessary processing. A common baseline can reduce variation, but it does not replace an analysis of each applicable regime.
## Cybersecurity standards and guidance
Recognised bodies such as ISO, IEC, IEEE, IETF, W3C, and industry councils develop or maintain technical and management standards. Standards support interoperability, security, safety, quality, and consistent performance.
### Hardware and communications standards
Hardware standards address:
- Physical dimensions, form factors, and connector specifications
- Electrical ratings, power requirements, and energy management
- Data interfaces such as USB, SATA, and HDMI
- Electromagnetic compatibility and resilience
- Electrical, thermal, mechanical, and functional safety

Communications standards and specifications address:
- Network and application protocols such as IP, TCP, HTTP, DNS, and secure file-transfer options
- Link rates, media, signalling, and interoperability
- Secure transport through supported, securely configured protocols such as TLS
- Wireless LAN specifications such as IEEE 802.11 and Wi-Fi security programs such as WPA2 and WPA3
- Quality-of-service mechanisms for time-critical traffic such as voice and video

Plain FTP does not encrypt credentials or content. Secure deployments use a suitable protected alternative, such as SFTP or FTPS, when file transfer requires confidentiality and integrity.
### Security and software standards
Security standards and specifications address:
- Cryptographic algorithms, protocols, modes, key management, and module assurance
- Authentication, authorisation, and federation through mechanisms such as OAuth 2.0, OpenID Connect, and SAML
- Authenticated integrity through message authentication codes or digital signatures
- Secure configuration, vulnerability management, testing, and incident handling
- Information security management, privacy management, and payment-account data protection

AES is a block cipher rather than a complete data-at-rest control. Secure use also requires an approved mode, sound key management, access control, and integrity protection where needed. SHA-256 and SHA-3 produce cryptographic hashes, but an unkeyed hash alone cannot provide authenticated integrity against an attacker who can replace both the data and its hash.

Software standards and guidance address:
- Coding conventions and secure development practices
- API contracts, interoperability, and versioning
- Data formats such as JSON, XML, and HTML
- Testing, benchmarking, quality assurance, and defect management
- Platform compatibility and lifecycle support
- Accessibility guidance for people with disability
### OWASP resources
The Open Worldwide Application Security Project is a not-for-profit foundation that develops free, community-led software security resources.

- OWASP Top 10:2025 is an awareness document covering critical web application security risks, not a complete verification standard
- Application Security Verification Standard provides technical security requirements and a basis for web application verification
- Mobile Application Security Verification Standard defines control groups for mobile application security
- Software Assurance Maturity Model supports assessment and improvement of secure software development practices
- Web Security Testing Guide provides structured guidance for testing web applications and web services
- Cheat Sheet Series provides focused implementation guidance
### NIST publications
The National Institute of Standards and Technology develops cybersecurity standards and guidance for the United States Government and wider use. Common publications include:
- FIPS 140-3 - Security Requirements for Cryptographic Modules
- SP 800-53 Rev. 5 - Security and Privacy Controls for Information Systems and Organizations
- SP 800-63-4 - Digital Identity Guidelines
- SP 800-37 Rev. 2 - Risk Management Framework for Information Systems and Organizations
- SP 800-30 Rev. 1 - Guide for Conducting Risk Assessments
- SP 800-61 Rev. 3 - Incident Response Recommendations and Considerations for Cybersecurity Risk Management
- SP 800-88 Rev. 2 - Guidelines for Media Sanitization
### PCI DSS
PCI DSS v4.0.1 defines baseline technical and operational requirements for protecting payment account data. It applies to environments where entities store, process, or transmit payment account data, and to systems that can affect the security of those environments.

PCI DSS is an industry standard rather than legislation. Payment card brands, acquiring institutions, contracts, and card-scheme rules determine applicability, validation, and enforcement. Compliance validation reflects a defined scope and date or period, and does not establish compliance with privacy law or protection from every security incident.
### ISO/IEC 27000 family
ISO and IEC jointly publish the ISO/IEC 27000 family. ISO develops standards but does not certify organisations. Independent certification bodies conduct certification.

- ISO/IEC 27001:2022, with Amendment 1:2024, specifies requirements for establishing, implementing, maintaining, and continually improving an information security management system
- ISO/IEC 27002:2022 provides information security control guidance and is not a certifiable management system standard
- ISO/IEC 27005:2022 provides guidance on information security risk management
- ISO/IEC 27017 provides cloud security control guidance
- ISO/IEC 27018:2025 provides guidance for protecting personally identifiable information in public clouds when the cloud provider acts as a processor
- ISO/IEC 27035 is a multipart series for information security incident management
- ISO/IEC 27701:2025 specifies requirements and guidance for a privacy information management system
### IEEE standards
IEEE develops technical standards for networking, computing, storage, and critical infrastructure.

- IEEE 802.11-2024 defines wireless LAN medium access control and physical-layer specifications
- IEEE 802.1X-2020 defines port-based network access control
- IEEE 802.3-2022 defines Ethernet
- IEEE 1619-2025 specifies cryptographic protection for data on block-oriented storage devices
- IEEE 1686-2022, with Corrigendum 1-2025, defines cybersecurity capabilities for power-system intelligent electronic devices
## Audits, assurance, and security testing
Audits and assurance engagements evaluate defined criteria within an agreed scope. Their conclusions depend on the engagement type, evidence, date or period, and level of assurance. A design assessment does not establish that controls operated effectively over time.
### Assurance activities
- Internal audit provides independent and objective assurance and advice designed to strengthen governance, risk management, and control
- Compliance audits assess conformance with defined legal, regulatory, contractual, standard, or policy criteria
- An audit committee oversees internal audit independence and performance, external audit, financial reporting, and the risk and control responsibilities assigned by its charter
- Control self-assessments can identify gaps but do not provide independent assurance
- An attestation engagement uses an independent practitioner to evaluate information against suitable criteria and report a conclusion or findings under professional standards
- Regulatory inspections establish findings within a regulator's mandate and do not automatically constitute independent assurance engagements
### Security testing and team exercises
Security testing can provide evidence for an audit or assessment, but it remains a distinct activity. Penetration testing attempts to identify and exploit vulnerabilities within authorised rules of engagement.

- Physical penetration testing evaluates controls against unauthorised physical access
- Red teams conduct objective-based adversarial exercises against people, processes, and technology
- Blue teams defend systems, investigate activity, and respond to incidents
- Purple teaming coordinates offensive and defensive specialists to improve prevention, detection, and response
- Passive reconnaissance gathers information without direct interaction with the target
- Active reconnaissance interacts with authorised systems to identify hosts, services, and exposures

Passive and active reconnaissance are information-gathering techniques, not separate forms of assurance. Blue teaming and purple teaming are not categories of penetration testing.
### Security audit method
1. Auditors establish the objectives, criteria, scope, period, exclusions, responsibilities, and required independence.
2. Auditors understand the environment, obligations, processes, systems, data, risks, and relevant controls.
3. Auditors develop procedures and sampling methods that can produce sufficient, appropriate evidence.
4. Auditors examine policies, records, configurations, access, logs, diagrams, training, incidents, and control operation.
5. Auditors use interviews, observation, sampling, reperformance, configuration review, and other authorised tests.
6. Auditors assess findings against criteria, evaluate risk and root causes, and confirm factual accuracy with management.
7. Auditors report scope, methods, limitations, findings, conclusions, management responses, owners, and target dates.
8. Auditors follow up and validate remediation according to the risk and agreed assurance process.

Auditors use vulnerability scanning or penetration testing only when it supports the audit objectives, falls within authorised scope, and can proceed safely. A security audit does not require exploitation.
### ISACA guidance
ISACA develops professional guidance for audit and assurance, governance, risk, cybersecurity, privacy, and digital trust.

- COBIT 2019 supports governance and management of enterprise information and technology
- ITAF, 5th Edition, establishes standards and guidance for planning, performing, and reporting IT audit and assurance work
- Risk IT and Val IT were separate frameworks in the COBIT 4.1 era. ISACA integrated their concepts into COBIT 5 and carried the approach into COBIT 2019
### SOC reports
System and Organization Controls is the AICPA's suite of examination services for controls at service organisations and other entities. Independent licensed CPA practitioners issue SOC reports. A SOC report is not a certification.

- SOC 1 reports address controls at a service organisation that are likely to be relevant to user entities' internal control over financial reporting. They do not provide general cybersecurity assurance
- SOC 2 reports examine controls against selected Trust Services Criteria relating to security, availability, processing integrity, confidentiality, or privacy. A report does not necessarily cover all five categories
- SOC 3 reports address Trust Services Criteria but contain less detail than SOC 2 reports. Organisations may distribute these general-use reports freely
- Type 1 SOC 1 and SOC 2 reports address control design at a specified date. Type 2 reports also address operating effectiveness over a specified period

Customers and partners use relevant SOC reports as one input to third-party risk assessment. They review the scope, period, opinion, exceptions, subservice organisations, and complementary user-entity controls before relying on a report. A SOC report does not establish overall legal compliance, cybersecurity maturity, or the absence of control failures.