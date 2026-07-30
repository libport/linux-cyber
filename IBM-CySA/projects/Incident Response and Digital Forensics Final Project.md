# *Incident Response and Digital Forensics* Final Project
## Part 1 - Case Study: Incident response at SecureSync Corporation
SecureSync Corporation is a mid-sized technology company that develops cloud-based solutions for businesses. With a team of over 500 employees, this organization prides itself on innovation and reliability. The company is headquartered in San Francisco with additional offices in Austin and Boston. The company's IT infrastructure supports a client base of approximately 200,000 users. Although the organization feels confident in its incident response capabilities, the company does not have a formalized incident response plan.
### Technical details
The company's IT infrastructure follows a hybrid model that combines on-premises servers with cloud-based services, specifically using Azure and Amazon Web Services (AWS) for scalability and redundancy. These services host the company's cloud-based solutions, which are segmented into virtual private networks (VPNs) for enhanced security. The organization's network architecture supports a multilayered security approach, incorporating firewalls, intrusion detection systems (IDS), and data encryption protocols.
### Current security tools and protocols
SecureSync Corporation uses the following security tools and protocols:
- **Firewalls:** SecureSync Corp. uses hardware and software firewalls to control incoming and outgoing network traffic based on predetermined security rules. These firewalls are critical in defending against unauthorized access.
- **Intrusion detection and prevention systems (IDPS):** The company uses real-time IDPS to monitor network traffic for suspicious activities and potential threats. These systems alert the IT team when anomalies are detected.
- **Endpoint security:** All employee devices are equipped with endpoint protection software, such as antivirus, threat detection, and response capabilities, to safeguard against malware infection and data breaches.
- **Encryption and data security:** Company data at rest and in transit is encrypted using advanced encryption standards (AES).
- **Access controls:** Role-based access control (RBAC) ensures employees can onlhy access the resources necessary for their roles, minimizing the risk of internal threats.
- **Logging and monitoring:** Comprehensive logging configurations capture detailed records across all systems and applications. These logs play a crucial role in incident detection and retrospective analysis.

Although SecureSync Corp. understands the importance of incident response, the company does not have a formal incident response plan documented. The Chief Information Officer (CIO) recognizes the need to document the preparedness plan, which will now include:
- Updated staff training programs focused on recognizing phishing attempts,
- Task reporting of suspicious activity
- Regular security drills.

The organization has established key personnel roles, which include an incident response team leader, a communication liaison, and technical specialists.

The organization also invested in advanced detection software systems to enhance their defenses against cyberthreats.
### Incident response in action
On a busy Monday morning, the IT security team at SecureSync Corp. detected unusual network activity. The IT security team identified multiple unauthorized access attempts on the company's email server, which stores sensitive client data and internal communications. This attempt raised concerns about a potential data breach. Initial scans revealed malware deployed to siphon sensitive data to an unknown offsite server.

During the incident investigation, the following information was discovered:
- The IT security team received alerts about anomalous behavior indicative of a cyberattack using the company's Incident Detection System (IDS).
- Log files revealed remote logins occurring at odd hours and from unusual locations.
- Automated scripts captured packet details, suggesting potential data exfiltration.

The company activated the incident response team upon confirmation of the breach and began a detailed investigation. Analysts discovered that the attack vector was a sophisticated spear-phishing email that compromised user email accounts and gave attackers backdoor access. The incident response team constructed a timeline to trace the incident's development and identify affected systems.

The team's rapid response initially focused on short-term containment to limit further damage. The team sealed off critical access points by deactivating compromised accounts and isolating affected network segments. The company performaed an extensive threat hunt to identify the malware. Threat intelligence research helped the team understand the malware's typical behavior. The team used these insights to devise an eradication plan that included removing malicious code from all infected systems. After confirming malware eradication, the team shifted their efforts to recovery. The team restored systems from secure backups and conducted system-wide checks to ensure no residual damage or vulnerabilities remained undetected. The team then established continuous monitoring to detect any attempts at further breaches or compromises.

With the incident under control, SecureSync Corp. initiated a comprehensive post-mortem analysis to evaluate its incident response strategy. The analysis focused on response times, communication efficiency, security protocol effectiveness, and areas needing improvement. Team debriefings highlighted necessary enhancements in the incident response plan, particularly regarding threat intelligence sharing and expedited cross-departmental communication. New preventive measures were discussed and documented, and actions were taken to patch identified vulnerabilities.

SecureSync Corp. also planned educational workshops to reinforce cybersecurity practices among employees to mitigate potential future threats. The management also recognized the importance of public transparency and issued a client-facing report assuring clients of improved security measures, reinforcing trust, and retaining brand credibility.

---
### Tasks
**Task 1: List the roles and responsibilities of four key team members in the incident response team.**

1. Incident response team leader: Leads the response, confirms the incident's severity and scope, assigns tasks, coordinates decisions, records approvals, and reports status to the Chief Information Officer.

2. Incident handler and forensic analyst: Validates alerts, preserves and examines evidence, builds the incident timeline, identifies affected accounts and systems, and reports findings with appropriate confidence.

3. IT and cloud operations specialist: Applies approved containment, eradication, and recovery actions across on-premises and cloud systems. This person also validates backups, configurations, and restored services before production use.

4. Communications liaison: Coordinates accurate and approved updates for executives, employees, clients, cloud providers, legal and privacy advisers, and other authorised stakeholders. The liaison records notifications and prevents unverified information from being released.

**Task 2: List the four methods, or tasks, you can use to monitor this company's internal systems for unusual activity.**

1. Centralised log monitoring and correlation: Collect email, authentication, firewall, IDS, endpoint, application, and cloud logs in a security information and event management platform. Correlate events to detect unusual login times, unexpected locations, repeated failures, new privileges, and abnormal data transfers.

2. Network and egress monitoring: Review IDPS alerts, firewall logs, DNS activity, proxy logs, VPN logs, and packet metadata for command-and-control traffic, lateral movement, unusual protocols, and transfers to unapproved external destinations.

3. Endpoint detection and response monitoring: Use endpoint detection and response telemetry to identify suspicious processes, scripts, persistence mechanisms, credential access, and unusual parent-child process relationships on employee devices and servers.

4. Identity, email, and cloud activity monitoring: Monitor sign-ins, multifactor authentication changes, session and token use, mailbox forwarding rules, OAuth application grants, bulk mailbox access, and cloud administrative actions. Compare activity with an appropriate baseline and corroborate anomalies.

**Task 3 : Using what you know about the NIST framework and this company, list four details to include when documenting detected incidents.**

1. Incident identity and timing: Record a unique incident identifier, the date and time of detection, the source timestamp and timezone, the reporting person or system, and the current incident owner.

2. Detection and evidence basis: Record the alert source, observed events, relevant indicators, affected security controls, and the reason the activity meets the organisation's incident criteria.

3. Scope and impact: Record known and potentially affected accounts, hosts, email services, network segments, cloud resources, client data, business services, and external parties.

4. Assessment and response status: Record the incident category, priority, confidence, current status, containment and recovery actions, approvals, communications, unresolved questions, and the next scheduled review.

**Task 4: List at least two containment strategies and explain how these strategies will help the company with containment.**

Compromised account and session containment: Disable confirmed compromised accounts, revoke active sessions and tokens, remove malicious forwarding rules or OAuth grants, and require secure credential reset and multifactor authentication re-enrolment after necessary evidence is preserved. This strategy removes the attacker's current access and reduces the risk of renewed access through stolen sessions or credentials.

Network isolation: Quarantine infected endpoints and isolate affected network segments or workloads while retaining approved forensic access. This strategy limits lateral movement and malware communication without shutting down unaffected services.

Malicious egress blocking: Block confirmed command-and-control IP addresses, domains, and URLs at perimeter controls, such as firewalls, proxies, DNS servers, email gateways, and cloud gateways. This strategy interrupts malware communication and suspected exfiltration while analysts continue to assess the full scope.

**Task 5: List the four-steps the company would use to conduct post-incident reviews based on the NIST framework.**

Step 1: Compile and validate the incident record. Confirm the timeline, affected assets and accounts, business impact, response actions, decisions, communications, and remaining uncertainties.

Step 2: Conduct a blameless lessons learned meeting. Ask responders, system owners, leadership, communications staff, and relevant service providers what occurred, which actions were effective, which actions caused delays, and where controls or coordination failed.

Step 3: Complete an after-action report. Document the supported root cause, incident scope, response and recovery performance, evidence gaps, successful controls, control failures, lessons learned, and recommended improvements.

Step 4: Prioritise and track corrective actions. Assign each approved improvement an owner, due date, priority, validation method, and residual risk, then update relevant plans, procedures, detections, training, and exercises.

**Task 6: Using the NIST framework, write a checklist of three tasks an organization can use to structure an approach for updating the response plan based on findings.**

- [ ] Convert verified findings into specific plan changes. Update escalation criteria, roles, contact details, evidence-preservation steps, cloud-provider coordination, containment playbooks, and recovery criteria.

- [ ] Assign governance and communicate the revision. Obtain approval, designate an owner and due date for each action, version-control the plan, and provide the updated procedures to employees and relevant third parties.

- [ ] Test and maintain the revised plan. Run a tabletop exercise or authorised simulation, record results and deficiencies, correct failed steps, and schedule periodic review after major incidents or significant changes to threats, technology, or business operations.

**Task 7: Based on the case study and the NIST framework, list four sources of digital evidence necessary for incident investigation.**

Source 1: Email server, secure email gateway, and mailbox evidence, including the spear-phishing message, full headers, body content, attachments, URLs, delivery logs, access logs, forwarding rules, and deletion records.

Source 2: Identity and access evidence, including authentication logs, multifactor authentication events, session and token records, IP addresses, device details, account changes, and cloud identity audit logs.

Source 3: Network and security-control evidence, including IDS alerts, firewall logs, DNS and proxy logs, VPN logs, egress records, and preserved packet captures or packet metadata.

Source 4: Endpoint and server evidence, including endpoint detection and response telemetry, volatile memory where safely acquired, forensic disk images, process and service records, malware artefacts, and system logs.

**Task 8: List the three steps required to assess the collected digital evidence and verify its integrity.**

Step 1: Establish relevance, provenance, and custody. Give each item a unique evidence identifier and record its source, owner, collector, collection method, original timestamp and timezone, collection time, tool version, storage location, and every custody transfer.

Step 2: Verify and preserve integrity. Calculate an approved cryptographic hash, such as SHA-256, as close to collection as possible. Store the original securely as read-only evidence, create a forensic working copy, and compare hashes after copying or transfer.

Step 3: Examine and analyse verified copies. Use validated tools to extract relevant artefacts, correlate independent sources into a normalised timeline, test malicious and benign explanations, and record every query, filter, transformation, tool, and result. Recheck integrity where required and document any collection gap, clock difference, or action that may have changed the evidence.

---
## Part 2 - Forensic investigations: Creating an Incident Response Plan

You will document steps and reason for an incident response plan that outlines the steps an organization would follow for any digital forensics investigation.

Your company will use this document as its standardized process and checklist to ensure consistency, thoroughness, and integrity in future investigations. You will divide the plan into four phases, each with specific objectives.

---
## Tasks
**Task 9: List the three types of digital evidence that the organization should review as part of a forensic investigation to determine the breach's origin and method. Then explain the purpose for that digital evidence.**

Evidence Type 1: Network evidence, including packet captures, flow records, firewall logs, DNS logs, proxy logs, and cloud network logs. These help trace malicious connections, lateral movement, command-and-control activity, and suspected transfers to the unknown offsite server.

Evidence Type 2: Endpoint and server evidence, including volatile memory, forensic disk images, endpoint detection and response telemetry, process records, malware artefacts, and operating-system logs. These help determine how the malware executed, what it changed, and which systems it affected.

Evidence Type 3: Email, identity, and cloud audit evidence, including the original spear-phishing message, headers, attachments, URLs, mailbox logs, sign-in events, session and token records, account changes, and cloud audit logs. These helps reconstruct initial access and subsequent use of compromised accounts.

**Task 10 :Using what you know about digital forensics, list four key components to include in structured reports following each incident and describe each component.**

First key component: The executive summary and incident overview state what occurred, when it was detected, the current status, the affected business services, the confirmed or suspected impact, the priority, and the decisions required from the intended audience.

Second key component: The scope, evidence, and methodology section defines the authorised systems, accounts, data, time range, collection methods, chain-of-custody controls, tools, and limitations so the work can be assessed and reproduced.

Third key component: The findings and timeline section presents the source-linked sequence of events, affected assets and identities, indicators, supported root cause, data exposure assessment, and confidence for each finding. It clearly separates observations from inferences, hypotheses, and unknowns.

Fourth key component: The response, recovery, and improvement actions section records containment, eradication, recovery, validation, communications, approvals, outcomes, residual risk, and lessons learned. It assigns follow-up actions to named owners with priorities, due dates, and verification methods.
