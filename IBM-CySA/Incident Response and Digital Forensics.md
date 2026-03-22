# Incident Response and Digital Forensics

> [!NOTE]
> This document explains incident response as a structured process to prepare for, detect, contain, eradicate, and recover from cybersecurity incidents while minimising operational, financial, and reputational harm. It distinguishes security events from confirmed incidents, emphasises the value of a tested incident response plan, and summarises common lifecycles from NIST and SANS Institute. It then introduces digital forensics workflows (collection, examination, analysis, reporting), DFIR integration, evidence integrity and chain of custody, typical data sources, and supporting tools and automation.
## Incident Response
Incident response is a structured approach that helps an organisation prepare for, detect, manage, and recover from cybersecurity threats such as unauthorised access, malware, phishing, ransomware, and data breaches. The aim is to reduce harm to people, information, systems, operations, reputation, and cost.
### Security events and security incidents
A security event is any observable activity that may indicate a security issue, such as repeated login failures, system changes, or use of administrative privileges. Many events are routine and only need monitoring.

A security incident is a confirmed breach of security policy that threatens confidentiality, integrity, or availability. Incidents require urgent action to contain impact and restore safe operations.
### Why an incident response plan matters
An incident response plan (IRP) defines roles, responsibilities, decision pathways, and step by step actions for high pressure situations. A well maintained IRP supports faster triage, clearer communication, safer evidence handling, and quicker recovery. The IBM 2024 Cost of a Data Breach Report noted lower average breach costs for organisations with dedicated response teams and regularly tested plans, compared with those without them.
### Core incident response lifecycle
Many organisations align to widely used guidance such as NIST and SANS. The common lifecycle includes these phases.

- Preparation
- Detection and analysis
- Containment
- Eradication
- Recovery
- Post incident review

Preparation focuses on readiness, including policies, tooling, training, and playbooks. Detection and analysis separates real threats from false alarms using monitoring, log review, and threat intelligence. Containment limits spread and protects unaffected systems. Eradication removes malicious artefacts and closes the exploited weakness. Recovery restores services, validates integrity, and monitors for recurrence. Post incident review captures lessons learned and improves controls.
### NIST and SANS frameworks
The NIST incident response lifecycle groups activities into four stages.

- Preparation
- Detection and analysis
- Containment, eradication, and recovery
- Post incident activity

Typical actions include response policies, team training, risk assessment, communications planning, alert verification, initial triage, system isolation, evidence preservation, patching, restoration, and root cause review.

The SANS incident response framework uses six stages.

- Preparation
- Identification
- Containment
- Eradication
- Recovery
- Lessons learned

SANS separates containment, eradication, and recovery more explicitly and calls for a retrospective report and review meeting soon after resolution.
### Preparation tasks and plan components
Preparation work typically covers policies, plans, people, and enabling technology.

- Roles and responsibilities, including escalation paths and communication leads
- Incident classification to prioritise response based on severity and impact
- Communication protocols for internal leaders, stakeholders, regulators, and where required the public
- Investigation procedures for collecting, analysing, and documenting evidence
- Response strategies for containment, eradication, and recovery
- Review and continuous improvement processes
- Training and awareness schedules, including drills and simulations

An effective IRP also states the mission, objectives, management support, measures of effectiveness, and capability improvement goals. Larger organisations often maintain playbooks tailored to business units or common incident types.
### Documentation across the lifecycle
Documentation supports accountability, repeatability, and faster recovery. It also strengthens legal and regulatory defensibility.

Before an incident, common documents include the IRP, role assignments, and communication protocols.

During an incident, common records include event logs, communications records, and severity or impact assessments.

After an incident, common outputs include a post incident report, lessons learned, and improvement recommendations.
### Detection, analysis, and impact assessment
Detection and analysis are strengthened by timely recording of actions and observations. Real-time tracking helps reveal patterns and avoids missing critical details.

Common detection techniques include system and network monitoring, log analysis, threat intelligence, and endpoint telemetry. Tools frequently used include SIEM platforms and endpoint detection and response (EDR) solutions.

Impact assessments typically consider affected assets, data sensitivity, recovery effort, business continuity, financial loss, reputation, and regulatory exposure. Low impact cases may involve limited exposure of non sensitive data. Medium impact cases may involve proprietary organisational information. High impact cases may involve major service disruption, sensitive data exposure, or ransomware affecting critical systems.
### Containment, eradication, and recovery in practice
Containment can use short term isolation of affected devices, long term segmentation of critical systems, and creation of backups to preserve evidence. Eradication includes removing malware, eliminating persistence, and addressing the initial attack vector, such as a phishing email. Recovery restores systems from clean backups, applies patches, validates functionality, and introduces preventative controls such as improved email filtering and targeted staff training.
### Post incident activities
Post incident work typically includes incident documentation, technical review, and action tracking.

- How the incident was detected
- A timeline from detection to resolution
- Actions taken, including who did what and when
- Technical findings, such as logs, indicators, and system analysis results
- Identified threat actors where possible
- Assigned actions for responsible parties
- A formal report with key evidence, lessons learned, and process improvements

Post incident outcomes often feed into updated procedures, tooling, resourcing, and training so the organisation is better prepared for the next event.
### Tools, automation, and resources
Incident response capability commonly uses these tool categories.

- Incident detection and alerting tools
- Threat intelligence platforms for collecting and sharing indicators
- Incident response case management platforms for workflow and documentation
- SOAR platforms to automate repeatable tasks and orchestrate integrations
- EDR solutions for endpoint visibility, containment, and remediation
- Network traffic analysis tools to identify anomalies and validate scope
- Compliance and reporting tools to support audit and regulatory needs

Common external resources include government cybersecurity guidance, threat intelligence sharing communities, and organisational frameworks and starter kits.
## Digital Forensics
Digital forensics is the disciplined collection, preservation and analysis of data from digital devices and services so the results can be relied on in investigations and legal proceedings. Evidence handling relies on integrity controls and a documented chain of custody so that the origin, access and changes to evidence are accountable.

Digital forensics covers evidence from any digital device or service. Computer forensics is a narrower subset focused on computers and related storage.
### Core workflow
NIST describes a four phase forensic workflow.

- Collection and preservation: Relevant systems and data sources are identified, then data is acquired and protected. Investigators typically create a forensic image so analysis occurs on a verified copy rather than the original media.
- Examination: Data is processed and searched for relevant artefacts, including metadata, logs and recoverable deleted material.
- Analysis: Artefacts are correlated to infer what occurred, when it occurred and how it occurred. This phase may include timeline building, attribution of actions to accounts or devices, and interpretation of attacker techniques.
- Reporting: Findings, methods, tools used, limitations and conclusions are documented in a form suitable for technical and non technical stakeholders, and for potential court use. Reports may also include prevention recommendations.
### DFIR
Digital forensics and incident response integrate forensic preservation with real-time containment. Response teams reduce immediate harm while forensic staff capture volatile and persistent evidence so the incident can be reconstructed without losing admissibility or reliability.
### Data sources used to reconstruct events
- File systems: file contents, timestamps, permissions, metadata and recoverable deleted files.
- Memory: running processes, active user sessions, encryption keys and live network connections.
- Network activity: IP addresses, protocols, timing and data flows that indicate command and control, lateral movement or exfiltration.
- Application data: audit trails and application logs that show user actions, errors and security events.
- Email: headers, content and delivery patterns that support phishing, malware distribution or unauthorised access investigations.
- Mobile and IoT devices: location data, calls, messages and installed applications that may link a person or device to an event.
- Cloud services: access logs, storage contents and virtual machine snapshots that show authentication, configuration changes and data movement.
### Handling digital evidence
Chain of custody records who handled an asset, when it was handled and why it was transferred. Breaks in custody can weaken credibility and may affect admissibility. Evidence protection is strengthened by access controls, monitoring and documented response actions, supported by practices aligned to the NIST Cybersecurity Framework functions of identify, protect, detect, respond and recover. Consistent use of tracking forms and audit logs supports accountability across the evidence lifecycle.
### Forensic data files
- Imaging creates a bit for bit copy of media, including free space and slack space, and is preferred when legal defensibility is required.
- Logical backups copy selected files and directories and are faster but generally miss deleted data and slack space.
- Deleted files often remain recoverable until overwritten.
- Free space can contain remnants of deleted data until reused.
- Slack space is unused space within allocated blocks that can retain residual data.
- MAC times record modification, access and creation information and support timeline reconstruction.
### Common tool categories
- Imaging and triage: FTK Imager, EnCase Imager, Cyber Triage.
- File system and artefact analysis: Autopsy, The Sleuth Kit, Magnet AXIOM.
- Network forensics: Wireshark, RSA NetWitness.
- Mobile device forensics: Cellebrite UFED, Oxygen Forensic Detective.
- Registry analysis: RegRipper, Registry Recon.
### Data recovery considerations
Data recovery success depends on the failure type and the extent of damage. Recovery efforts can be risky because attempts may overwrite evidence or worsen device failure. Common mistakes include delaying action, mishandling failing drives, using unverified tools and overlooking encryption. Key risks include data corruption, compliance exposure, security gaps and physical security breaches, which reinforces the need for tested backups and recovery plans.