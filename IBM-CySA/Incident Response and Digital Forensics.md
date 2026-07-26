# Incident Response and Digital Forensics
> [!NOTE]
> Organisations use incident response to prepare for, detect, contain, investigate, and recover from cyber incidents. Digital forensics supports that work by preserving, examining, and interpreting digital evidence through controlled and documented methods.
## Incident response
Incident response coordinates the people, processes, and technology that manage cybersecurity incidents. It addresses events such as unauthorised access, malicious code, phishing, ransomware, data breaches, service disruption, and misuse by trusted users. Effective response limits operational, financial, legal, safety, and reputational harm while restoring secure services.
### Cybersecurity events and incidents
A cybersecurity event is an observable occurrence involving systems, networks, data, identities, or services. An event may be benign, suspicious, or harmful. Repeated sign-in failures, configuration changes, and administrative activity are events that require monitoring or analysis in context.

A cybersecurity incident is an occurrence that actually or imminently jeopardises the confidentiality, integrity, or availability of information or systems without lawful authority. It may also violate, or threaten to violate, law, security policy, security procedures, or acceptable-use policy. Not every event becomes an incident, and responders should validate reports before assigning an incident category and priority.
### Why an incident response plan is essential
An incident response plan defines decision authority, roles, escalation paths, communications, and coordinated actions for high-pressure situations. A maintained and exercised plan helps teams triage reports, protect evidence, meet notification obligations, contain damage, and restore operations. For Australian Government systems covered by the Information Security Manual, the Australian Signals Directorate requires organisations to develop and maintain an incident response plan and exercise it at least annually.

An organisation should align its incident response plan with business continuity, disaster recovery, crisis management, privacy, legal, physical security, and supplier arrangements. It should also support secure communications if normal email, identity, or collaboration systems become unavailable or untrusted.
### Incident response lifecycles
Incident response rarely follows a strictly linear sequence. Teams often perform detection, containment, analysis, eradication, and recovery at the same time, then revise earlier decisions as new evidence emerges. A practical operational sequence includes:
- Preparation
- Detection and analysis
- Containment
- Eradication
- Recovery
- Lessons learned and continuous improvement

Preparation establishes governance, capabilities, access, training, and playbooks. Detection and analysis distinguish incidents from benign activity and establish an initial scope. Containment limits damage and spread. Eradication removes malicious access, persistence, and exploited weaknesses. Recovery restores trusted services and monitors them for renewed activity. Teams should share and apply lessons as soon as they identify them rather than waiting for all recovery work to finish.
### NIST and SANS approaches
NIST Special Publication 800-61 Revision 3, published in 2025, aligns incident response with all six functions in the NIST Cybersecurity Framework 2.0:
- Govern
- Identify
- Protect
- Detect
- Respond
- Recover

Govern, Identify, and Protect support preparation and reduce incident risk. Detect, Respond, and Recover cover the direct response to an incident. The Improvement category within Identify receives lessons from all six functions and feeds prioritised changes back into governance, protection, detection, response, and recovery.

Older guidance may show the four-phase lifecycle from NIST Special Publication 800-61 Revision 2:
- Preparation
- Detection and analysis
- Containment, eradication, and recovery
- Post-incident activity

Revision 3 superseded Revision 2 and replaced that lifecycle with the Cybersecurity Framework 2.0 model.

The SANS incident handling model uses six phases:
- Preparation
- Identification
- Containment
- Eradication
- Recovery
- Lessons learned

The SANS sequence remains useful for operational playbooks. The NIST model places stronger emphasis on enterprise risk management and continuous improvement across organisational functions.
### Preparation and plan components
Preparation should cover:
- Scope, objectives, management support, and authority to act
- Named roles, deputies, escalation paths, and decision thresholds
- Incident categories and priorities based on business impact and urgency
- Current asset, data, service, supplier, and dependency information
- Internal, external, and out-of-band communication methods
- Legal, privacy, contractual, regulatory, and insurance requirements
- Contacts for service providers, forensic specialists, regulators, and law enforcement
- Procedures for triage, containment, evidence preservation, eradication, and recovery
- Recovery criteria, service priorities, clean backup requirements, and business acceptance
- Exercises, staff training, capability measures, and improvement tracking

Organisations should tailor playbooks to common incident types, critical services, technologies, and business units. Each playbook should identify prerequisites, decision points, safeguards, expected evidence, and conditions for escalation.
### Documentation throughout the lifecycle
Before an incident, organisations should maintain the response plan, playbooks, contact lists, role assignments, system diagrams, data classifications, and communication templates.

During an incident, the response team should keep a time-stamped record of observations, decisions, approvals, commands, communications, evidence identifiers, containment actions, and changes to systems. The record should use a consistent time zone and distinguish confirmed facts from assumptions and hypotheses.

After an incident, the organisation should preserve the final timeline, impact assessment, technical findings, response evaluation, notification record, evidence inventory, lessons, and remediation plan. Each remediation action should have an owner, priority, due date, and verification method.
### Detection, analysis, and impact assessment
Security information and event management systems, endpoint telemetry, identity logs, network monitoring, application logs, cloud audit records, threat intelligence, and reports from users or third parties can reveal suspicious activity. Analysts should correlate multiple sources because a single alert rarely establishes the full scope or cause of an incident.

Triage should establish what occurred, whether activity continues, which assets and identities are affected, how the attacker gained or retained access, and what evidence requires urgent preservation. Incident priority should reflect business impact and urgency, not technical severity alone. Relevant factors include:
- Safety and critical-service disruption
- Scope and duration
- Data sensitivity and likely exposure
- Privilege level and attacker persistence
- Legal, regulatory, contractual, and privacy obligations
- Recovery complexity and available workarounds
- Financial and reputational consequences
### Containment, eradication, and recovery
Containment may isolate hosts, segment networks, disable accounts, revoke sessions and tokens, block malicious infrastructure, restrict vulnerable services, or apply temporary controls. Responders should preserve volatile evidence when safe and feasible, but they should not delay action needed to protect people or critical operations. They should record any action that changes potential evidence.

Eradication removes malware and persistence, closes exploited weaknesses, resets exposed credentials, fixes insecure configurations, and rebuilds systems from trusted sources when necessary. Teams should identify the initial access path and broader control failures rather than treating a visible artefact as the entire cause.

Recovery restores data and services in stages. Teams should verify system integrity, apply security updates, test business functions, monitor for renewed activity, and obtain approval from service owners before returning critical systems to normal operation. Recovery from a clean backup does not remove the need to address the access path that enabled the incident.
### Lessons learned and improvement
A review should address:
- How the organisation detected and declared the incident
- What happened, when it happened, and which assets or data were affected
- Which actions responders took, who authorised them, and how well they worked
- Which technical and organisational controls failed or succeeded
- Which facts remain uncertain and why
- Whether communications and notifications met requirements
- Which improvements require new controls, resources, training, or supplier action

Attribution should appear only when evidence supports it and the organisation needs it for a defined decision. The response team should prioritise containment, recovery, and risk reduction over unsupported claims about an actor's identity.
### Tools and automation
Incident response commonly relies on:
- Security information and event management platforms
- Endpoint detection and response tools
- Network detection and traffic analysis tools
- Identity, cloud, application, and email security telemetry
- Threat intelligence platforms and indicator-sharing services
- Case management systems for decisions, evidence, tasks, and communications
- Security orchestration, automation, and response platforms
- Forensic acquisition and analysis tools
- Backup, restoration, and integrity-validation systems
- Secure and out-of-band communication channels

Automation can enrich alerts, collect volatile data, isolate endpoints, block indicators, and open cases. Organisations should require suitable approval and safeguards for high-impact actions, particularly those that could interrupt services, destroy evidence, or affect many systems.
## Digital forensics
Digital forensics applies scientific and technical methods to identify, collect, preserve, examine, analyse, and report digital evidence. Organisations use it to investigate cyber incidents, policy violations, fraud, operational failures, litigation, and regulatory issues. Computer forensics focuses on computers and associated storage, while digital forensics also covers networks, mobile devices, cloud services, applications, embedded systems, and other digital sources.

Forensic reliability depends on documented and repeatable methods, competent practitioners, validated tools, and integrity controls. No tool, format, or hash value automatically makes evidence admissible. Legal requirements vary by jurisdiction and purpose.
### Core forensic workflow
NIST describes four broad phases:
- Collection: Investigators identify, label, record, acquire, and preserve relevant data. They should consider legal authority, scope, data volatility, collection order, and the risk that acquisition will change the source.
- Examination: Investigators process collected data and extract relevant artefacts through automated and manual methods. They may recover deleted content, decode formats, filter known files, search text, and organise metadata while protecting evidence integrity.
- Analysis: Investigators correlate artefacts, test competing explanations, reconstruct timelines, and determine what the evidence supports. They should separate direct observations from inference and account for clock error, missing data, and alternative causes.
- Reporting: Investigators document the request, scope, evidence, methods, tool versions, integrity checks, findings, limitations, and conclusions. Reports should use language suited to their technical, executive, legal, or regulatory audience without overstating certainty.
### Digital forensics and incident response
Digital forensics and incident response, often shortened to DFIR, combines urgent operational response with evidence-led investigation. Responders limit harm while forensic practitioners preserve volatile and persistent data, reconstruct activity, and test the incident scope.

Operational and forensic priorities can conflict. Disconnecting a system may stop an attack but destroy a live network connection or prevent memory capture. Leaving it connected may preserve evidence but allow further harm. The incident lead and forensic lead should assess safety, business impact, attacker activity, data volatility, and legal needs, then document the decision and any resulting changes.
### Data sources for event reconstruction
- File systems provide contents, allocation status, metadata, permissions, and timestamps.
- Memory can reveal running processes, active sessions, decrypted content, encryption keys, injected code, and live connections.
- Network sources include packet captures, flow records, domain name system logs, proxy logs, firewall records, and virtual private network logs.
- Identity systems record authentication, multifactor challenges, privilege changes, directory activity, and token use.
- Applications and databases record user actions, queries, errors, transactions, and security events.
- Email systems provide headers, routing information, message content, attachments, and authentication results.
- Cloud and software-as-a-service platforms provide audit logs, object histories, configuration changes, snapshots, and provider-held records.
- Mobile and embedded devices can provide messages, calls, application data, sensor data, and location records, subject to lawful authority and technical limitations.
### Handling digital evidence
Investigators should obtain proper authority, define scope, minimise changes to source data, and use write blocking where the source and acquisition method support it. They should create protected originals or master acquisitions, conduct analysis on verified working copies, restrict access, and preserve an audit trail.

A chain-of-custody record should identify the evidence, its source, the collector, the acquisition method, relevant dates and times, the time zone, cryptographic hashes, storage locations, access, and every transfer. Cryptographic hashes help detect change, but they do not establish provenance or replace a complete chain of custody.

Investigators should verify hashes after acquisition and significant transfers, protect evidence and hash records from unauthorised change, and document any failed comparison. Courts and regulators apply jurisdiction-specific requirements, and gaps in handling records can weaken evidential weight or invite challenges. Legal advisers should determine the applicable requirements.
### Acquisition types and storage artefacts
- A bit-stream image copies the addressable sectors that the device and interface expose, including allocated space, unallocated space, and file slack. Damaged sectors, hidden regions, controller behaviour, encryption, and solid-state storage processes may limit the acquisition.
- A logical acquisition collects selected files, directories, accounts, or service data through the operating system, an application, or an application programming interface. It can be faster and less disruptive than physical imaging but usually omits unallocated space, slack space, and some deleted content.
- A live acquisition captures volatile state such as memory, processes, sessions, and network connections. Collection changes the running system, so investigators must document the method and resulting effects.
- Deleted data may remain recoverable until overwriting, trimming, garbage collection, encryption-key loss, or physical damage makes recovery impracticable.
- Unallocated space may retain remnants of deleted files. File slack contains unused bytes within the last allocated block of a file and may retain residual data.

File systems record timestamps with different meanings. NTFS commonly stores created, content-modified, accessed, and metadata-entry-modified times. Unix-like systems commonly use change time for inode metadata rather than creation time, although some file systems also store a birth time. Clock drift, time-zone conversion, copying, system activity, and deliberate alteration can affect timestamps, so analysts should corroborate them with independent sources.
### Common forensic tool categories
- Disk imaging and acquisition tools, such as FTK Imager, Magnet Acquire, and Guymager
- File-system and artefact analysis suites, such as Autopsy, The Sleuth Kit, Magnet AXIOM, and OpenText EnCase Forensic
- Memory analysis tools, such as Volatility 3
- Network analysis tools, such as Wireshark, Zeek, and NetWitness
- Mobile device tools, such as Cellebrite UFED and Physical Analyzer, Oxygen Forensic Detective, and Magnet AXIOM
- Windows artefact tools, such as RegRipper and Registry Explorer

Teams should select tools for the data source, platform, question, and legal context. They should test tools against known data, record versions and settings, understand known limitations, and corroborate important findings with another method when practicable.
### Data recovery and evidence protection
Business recovery restores operations, while forensic recovery seeks data that can answer investigative questions. The two activities require coordination because restoration, repair, or reuse can alter or destroy evidence.

When storage hardware fails, repeated power cycles, repair utilities, or unplanned recovery attempts can worsen damage or overwrite recoverable data. Trained specialists should stabilise the source, acquire it with suitable methods, and work from verified copies where possible. Encryption, physical damage, overwriting, solid-state drive trimming, and incomplete backups can reduce recovery prospects. Tested backups support operational resilience, but a backup does not replace a forensic acquisition or complete evidence records.