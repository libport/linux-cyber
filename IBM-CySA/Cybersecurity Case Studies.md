# Cybersecurity Case Studies
Cybersecurity case studies connect technical evidence, organisational decisions, business effects, legal duties, and control improvements. Effective analysis distinguishes confirmed facts from inference, explains how an incident unfolded, and turns evidence into proportionate safeguards.
## Analysing Cybersecurity Case Studies and Phishing Incidents
### Purpose and limits
Case studies help organisations understand how attackers combine technical weaknesses with human behaviour and operational gaps. They also show how detection, escalation, communication, and recovery decisions affect the scale of an incident.

Useful case studies can:
- reveal recurring attack paths and control failures
- connect security events with financial, legal, operational, and human consequences
- test whether policies work under pressure
- support training with realistic decisions and trade-offs
- identify controls that can prevent recurrence or reduce harm

Each case study remains specific to its time, technology, threat actor, and organisation. Analysts should avoid treating one incident as a universal model or claiming a causal link that the evidence does not support. For example, the 2017 Equifax breach intensified public scrutiny of data protection, but it did not cause the European Union's General Data Protection Regulation. The European Union adopted the regulation in 2016, and it applied from 25 May 2018.
### Evidence-led analysis
An analyst can structure a case study around eight questions:
1. The analyst defines the incident, affected organisation, relevant dates, and confirmed scope.
2. The analyst separates primary evidence, credible reporting, disputed claims, and unresolved questions.
3. The analyst identifies the assets, identities, data, and business processes at risk.
4. The analyst reconstructs the attack path from initial access to discovery, containment, and recovery.
5. The analyst maps technical weaknesses and organisational conditions without assuming that a single failure caused the event.
6. The analyst assesses operational, financial, legal, regulatory, and human effects.
7. The analyst evaluates which controls failed, which controls worked, and which safeguards would reduce similar risk.
8. The analyst records uncertainty and updates conclusions when stronger evidence becomes available.

Timelines, attack-path diagrams, evidence tables, and decision logs can improve clarity. They should identify their sources and distinguish observed events from analytical reconstruction.
### Phishing and business email compromise
Phishing uses deceptive messages or interactions to persuade a target to disclose information, approve a payment, open a malicious file, follow a harmful link, or grant access. Attackers often impersonate trusted people or organisations and create urgency, authority, fear, or curiosity.

| Form | Common method | Main risk |
|---|---|---|
| Bulk phishing | High-volume messages sent to many recipients | Credential theft, malware delivery, or payment fraud |
| Spear phishing | Tailored messages aimed at a person or team | Access to sensitive systems or information |
| Whaling | Targeting executives or other high-value decision-makers | Strategic access, confidential data, or large payments |
| Business email compromise | Impersonating or taking over a business account | Fraudulent transfers, invoice changes, or payroll diversion |
| Smishing | Deceptive text messages | Credential theft, malicious application installation, or payment fraud |
| Vishing | Deceptive voice calls or voice messages | Disclosure of credentials, security codes, or financial approval |

Attackers strengthen these approaches with information from corporate websites, social media, breached databases, and compromised mailboxes. A convincing message may quote a real invoice, continue an existing email thread, or imitate an established approval process.
#### Rimasauskas business email compromise
Evaldas Rimasauskas and his associates impersonated an Asian hardware manufacturer that conducted business with two US internet companies. They created lookalike corporate infrastructure, sent fraudulent invoices and contracts, and directed payments to bank accounts under their control. The scheme obtained more than US$120 million before investigators traced the transactions. The US Department of Justice did not name the victim companies in its public case record.

The case demonstrates that a technically simple impersonation can produce a major loss when payment processes rely on email and familiar branding. Effective controls include independent verification of bank-detail changes, dual approval for high-value transfers, payment limits, domain monitoring, mailbox security, and rapid recall procedures with financial institutions.
#### Voice phishing and authority impersonation
Voice phishing can combine spoofed caller identification, personal information, scripted urgency, and synthetic audio. An attacker may claim to represent an executive, a bank, a supplier, or a government agency. Caller identification and a familiar voice do not establish identity.

Organisations can reduce the risk through callback procedures using trusted contact details, separate approval channels, transaction limits, staff training, and a culture that allows employees to pause an urgent request. Authentication should rely on verified information or cryptographic controls rather than biographical facts that an attacker can obtain.
## Point-of-Sale Systems and Insider Risk
### Point-of-sale exposure
Point-of-sale systems process transactions and often connect payment terminals, registers, store networks, inventory platforms, identity services, and external service providers. They may handle payment-card data, customer information, employee credentials, product records, and transaction logs. That combination makes them attractive to criminals and difficult to secure across large retail estates.

Common attack paths include:
- stolen vendor or employee credentials
- weak remote-access controls
- poor network segmentation
- unpatched operating systems and applications
- malicious software that captures card data in memory
- altered terminals or attached skimming devices
- insecure integrations with suppliers and payment processors
- inadequate monitoring and delayed response to alerts

Payment environments need layered safeguards. Point-to-point encryption protects card data from the point of interaction to a secure decryption environment. Tokenisation reduces the amount of reusable card data stored in business systems. Network segmentation limits movement between corporate, vendor, and payment systems. Multifactor authentication, application allowlisting, secure configuration, device inspection, central logging, and prompt incident response provide further protection. Compliance with the Payment Card Industry Data Security Standard supports these controls but does not guarantee that an environment is secure.
### Target, 2013
Attackers compromised Target during the 2013 holiday period and stole payment-card data from about 40 million accounts. Target also reported that personal information relating to as many as 70 million people had been taken. A US Senate staff analysis reconstructed the incident from public evidence and reported that attackers used credentials associated with a third-party vendor, moved through Target's network, installed card-stealing malware, staged captured data internally, and removed it through external systems.

Some details in early public reconstructions remained unconfirmed. The evidence nevertheless supports several control lessons:
- Third-party access should use least privilege, multifactor authentication, time limits, and monitoring.
- Payment networks should remain isolated from vendor and general corporate networks.
- Security teams need clear ownership for high-confidence alerts and rapid escalation paths.
- Organisations should test whether monitoring tools produce action, not only whether the tools generate alerts.
- Incident exercises should include payment systems, suppliers, executives, legal advisers, and communications teams.
### Home Depot, 2014
Home Depot reported that criminals used a third-party vendor's credentials to enter the company's network. They then obtained elevated rights and deployed custom malware on self-checkout systems in US and Canadian stores. The malware captured payment-card information from April to September 2014 and affected as many as 56 million cards.

Home Depot's public filing supports the need for strong vendor authentication, privilege control, segmentation, endpoint hardening, encryption, and continuous monitoring. It also shows why a large retailer must deploy controls consistently across every store and terminal rather than treating the corporate network as the only critical environment.
### Insider risk
Insider risk arises when a person with authorised access or organisational knowledge causes harm, whether intentionally or unintentionally. Relevant people can include employees, contractors, suppliers, partners, former staff, and anyone whose trusted account remains active.

Insider incidents can involve:
- theft or disclosure of sensitive information
- fraud, sabotage, or unauthorised system changes
- accidental exposure through error or insecure configuration
- misuse of privileged accounts
- compromised credentials used by an external attacker
- concealment of activity through deleted logs or altered records

No universal behavioural sequence identifies a harmful insider. Personal characteristics alone do not establish risk. Organisations should focus on documented activity, access patterns, conflicts with policy, and corroborated indicators while respecting privacy, employment law, and procedural fairness.

Effective safeguards include least privilege, separation of duties, access reviews, prompt offboarding, privileged-access management, data-loss prevention, tamper-resistant logging, secure reporting channels, and proportionate investigation. Security, human resources, legal, privacy, and management teams should define responsibilities before an incident occurs.
### Joshua Schulte and the Vault 7 disclosures
Joshua Schulte, a former Central Intelligence Agency software developer, abused his access to steal classified cyber tools and transmit them to WikiLeaks. The disclosures became known as Vault 7 and Vault 8. A US court sentenced him in 2024 to 40 years in prison for offences that included espionage and computer hacking.

The case shows how broad privileged access, weak separation of duties, and gaps in monitoring can allow one trusted person to remove highly sensitive material. High-risk environments should restrict bulk access, record administrative actions, separate development from release authority, monitor unusual collection activity, and test whether controls can detect misuse by skilled insiders.
### PegasusEFB cloud exposure
Security researchers found an openly accessible Amazon S3 bucket linked to PegasusEFB, an electronic flight-bag service. The bucket contained about 6.5 terabytes across more than 23 million files, including flight information, crew personal data, source code, and plain-text secrets. The researchers reported no evidence that malicious actors had accessed the material.

The exposure should not be described as a confirmed breach by Pegasus Airlines or as proof of malicious insider activity. It demonstrates how an authorised user, developer, or administrator can create serious risk through insecure cloud configuration. Cloud safeguards should include private-by-default storage, automated configuration checks, secret management, access logging, data classification, and prompt credential rotation after exposure.
## AI-Enabled Threats and Ransomware
### Artificial intelligence in cyber operations
Artificial intelligence can help attackers and defenders process information, generate content, and automate parts of a workflow. Current evidence supports an increase in the speed, scale, and quality of established techniques more strongly than it supports claims of fully autonomous, novel cyberattacks.

Threat actors can use generative systems to:
- draft persuasive phishing messages in several languages
- gather and summarise public information for targeting
- generate or modify scripts and code
- create synthetic voices, images, and video for impersonation
- accelerate vulnerability research and basic reconnaissance
- adapt social-engineering content for different recipients

These uses do not remove the need for infrastructure, access, operational decisions, and human oversight. Analysts should label an incident AI-enabled only when evidence connects an AI capability to the attack. Automation, data scraping, or a large data set does not establish that connection.

Defenders can use artificial intelligence to prioritise alerts, identify anomalous activity, classify malicious messages, summarise investigations, assist malware analysis, and search large collections of security data. These systems can also produce false results, expose sensitive inputs, inherit bias, or encourage excessive trust. Organisations need access controls, approved-use policies, data handling rules, human review, testing, monitoring, and an ability to disable unsafe functions.
### Adoption and governance
Splunk's 2024 survey of 1,650 security leaders found extensive use of generative artificial intelligence in security operations. It also found governance gaps. Thirty-four per cent of respondents said their organisations had no generative AI policy, and 65 per cent said they did not fully understand the technology's implications.

These results describe the surveyed population, not every organisation. They support a practical governance priority: an organisation should understand which tools staff use, what data those tools receive, how outputs influence decisions, and who accepts the associated risk.
### Deepfake payment fraud in Hong Kong
An official Hong Kong Police return recorded three deepfake fraud cases in 2024. In the largest case, fraudsters used a pre-recorded video conference to impersonate company officers and induced an employee to transfer HK$240 million. A second pre-recorded conference case caused a loss of HK$4 million.

The cases show why visual presence and voice similarity cannot replace transaction controls. High-value payments should require independent verification, multiple approvers, trusted contact channels, and anomaly checks. Staff should be able to challenge an unusual instruction regardless of the apparent seniority of the requester.
### Classification discipline
Public incidents sometimes receive an AI label without supporting evidence. Facebook's large-scale data-scraping case concerned the automated collection of public profile data and weaknesses in platform design. LinkedIn likewise stated that a 2021 data set offered for sale combined scraped public data from LinkedIn with information from other websites and did not expose private member data through a breach of LinkedIn.

Scraping can create privacy and security risks, but it is not itself evidence of artificial intelligence. Accurate classification helps organisations select the right controls and avoids overstating the role of emerging technology.
### Ransomware
Ransomware restricts access to systems or data and demands payment. Many operators also steal information and threaten disclosure, a practice commonly called double extortion. The ransomware-as-a-service model allows affiliates to conduct intrusions with tools and infrastructure supplied by a specialist operator.

A typical intrusion may involve stolen credentials, exploitation of an internet-facing service, malicious email, or access purchased from another criminal. Attackers often escalate privileges, disable security tools, discover backups, move laterally, steal data, and deploy encryption. The exact sequence varies, and encryption may never occur when data theft alone creates sufficient leverage.
### Atlanta and Colonial Pipeline
The SamSam ransomware incident disrupted many City of Atlanta services in March 2018 and required an extensive recovery effort. The incident reinforced the need for resilient backups, current asset information, network segmentation, central logging, and practised recovery procedures.

Colonial Pipeline detected ransomware on 7 May 2021 and shut down its pipeline operations to contain uncertainty around the intrusion. The shutdown affected fuel supply across parts of the eastern United States. The event showed how an intrusion into business systems can create operational consequences when an organisation cannot quickly establish the boundary between corporate and industrial environments.

Ransomware resilience depends on controls that work together:
- phishing-resistant multifactor authentication and secure remote access
- rapid remediation of exposed vulnerabilities
- network segmentation and restricted administrative pathways
- protected, tested, and recoverable backups
- endpoint detection, central logging, and active monitoring
- rehearsed containment, restoration, communication, and decision processes
- predefined engagement with law enforcement, regulators, insurers, and specialist advisers
## Incident Response and Digital Forensics
### Incident-response scope
Incident response helps an organisation detect, contain, eradicate, and recover from cybersecurity incidents while preserving evidence and learning from the event. Vulnerability management, patching, identity security, and resilience sit within the broader cybersecurity risk program. They support preparation and reduce incident likelihood, but they do not become incident-response activities solely because they improve security.

NIST Special Publication 800-61 Revision 3 integrates incident response with the six Cybersecurity Framework 2.0 functions: Govern, Identify, Protect, Detect, Respond, and Recover. Preparation depends on governance, asset knowledge, protective controls, roles, communication paths, suppliers, and exercises. During an incident, the organisation should establish authority, prioritise safety and business needs, preserve reliable records, communicate consistently, and adapt as evidence develops.
### Google Home rollout incident
During a 2017 rollout, a Google Assistant software defect caused devices to retrieve speaker-recognition files about 50 times more often than expected. The requests exhausted a service quota. Google paused the rollout at 25 per cent, but the team misdiagnosed the problem, increased the quota, and resumed deployment without identifying the root cause. When the rollout reached all users over a weekend, many requests failed and the response escalated late.

The review identified several operational lessons:
- Teams should declare an incident early when user impact or uncertainty exceeds normal operations.
- Responders should assign incident command, operations, and communications roles.
- Teams should stabilise service before pursuing a complete diagnosis.
- A paused rollout should not resume until the team understands the failure or can control the risk.
- High-risk changes should occur when the required responders can monitor and intervene.
- A shared incident record should capture decisions, evidence, actions, and ownership.
### Google data-centre lightning incident
In 2015, four lightning strikes near a Google data centre in Belgium disrupted power to disk equipment. Servers remained powered, but some disk trays lost power when parts of the backup system failed to transfer correctly. Google declared a major incident, restored power, migrated virtual machines, and repaired affected storage.

About five per cent of standard persistent disks in the affected zone experienced at least one input or output error. Google reported permanent data loss affecting about 0.000001 per cent of the space allocated to running persistent disks in the zone. Snapshots remained unaffected.

The event demonstrates the value of layered power protection, geographic resilience, tested backups, clear incident roles, and precise communication about impact. A low percentage of permanent data loss can still affect customers whose data falls within that fraction.
### Digital forensics
Digital forensics identifies, collects, preserves, examines, and reports digital evidence. Investigators need lawful authority, technical competence, reproducible methods, and records that explain who handled evidence and what changed.

Core practices include:
- defining the legal and investigative scope before collection
- preserving original evidence and using verified working copies where appropriate
- recording chain of custody and evidence-handling decisions
- using validated tools and documenting their versions and settings
- collecting volatile evidence when delay would destroy it
- normalising time zones and checking clock accuracy
- separating observed facts from interpretation
- protecting irrelevant, personal, privileged, and sensitive information
- retaining and disposing of evidence under lawful, documented rules
### Bernard Madoff
Bernard Madoff pleaded guilty in March 2009 to 11 federal offences connected with his investment fraud and received a 150-year prison sentence in June 2009. Public court records establish the plea and sentence, but they do not support attributing the prosecution to one specific forensic-imaging or metadata technique. The case should therefore support only claims grounded in the available investigative or court record.
### United States v Ganias
Federal agents created forensic images of Ganias's computer hard drives in 2003 while executing a warrant in an investigation of other people. They retained non-responsive data and obtained a second warrant in 2006 to search it for evidence against Ganias. A Second Circuit panel initially found a Fourth Amendment violation and vacated the conviction. The court later reheard the case en banc, assumed without deciding that a constitutional violation had occurred, applied the good-faith exception, and affirmed the conviction.

The final decision did not rule that investigators may retain and search unrelated data without limit. It shows why investigators need particularised authority, defensible retention rules, minimisation procedures, and legal review when a later investigation seeks data collected under an earlier warrant.
## Penetration Testing and Compliance
### Penetration testing
Penetration testing uses authorised, controlled attempts to identify and validate exploitable weaknesses. It differs from a vulnerability scan because testers may demonstrate how an attacker could combine weaknesses, gain access, escalate privileges, or reach a defined objective. Written authorisation and agreed rules of engagement set the scope, timing, methods, data handling, safety limits, contacts, and stop conditions.

A sound engagement includes:
1. The organisation defines objectives, critical assets, exclusions, and acceptable risk.
2. Testers gather information within the approved scope.
3. Testers identify vulnerabilities and validate them with the least harmful method that answers the test objective.
4. Testers document attack paths, affected assets, evidence, and business consequences.
5. The organisation prioritises remediation according to exploitability, exposure, impact, and existing controls.
6. Testers confirm whether remediation closes the attack path without creating new weaknesses.

Automated tools can improve coverage but cannot replace judgement. They may miss chained weaknesses, produce false positives, or cause disruption. A test should protect personal information, credentials, production data, and operational availability throughout the engagement.
### Authorised phishing simulation
An authorised phishing simulation can test reporting behaviour, technical controls, and business processes. A realistic engagement may use a plausible login page or business request, but it should minimise data collection and avoid unnecessary distress. The organisation should establish approval, scope, privacy protections, escalation paths, and a plan for any real compromise discovered during the exercise.

Useful measures include report rate, time to first report, security-team response time, control effectiveness, and completion of follow-up actions. Click rate alone provides an incomplete view and can encourage blame rather than improvement.
### Physical and social-engineering assessments
An authorised assessment may test reception procedures, visitor controls, identity verification, clean-desk practices, removable-media handling, and secure disposal. Testers can use agreed pretexts to assess whether staff verify unusual requests and report suspicious behaviour.

Rules of engagement should prohibit unsafe conduct, uncontrolled access to personal data, coercion, and entry into excluded areas. The test should protect staff dignity and focus on systems, training, and processes rather than public identification of individuals.
### Equifax, 2017
Equifax failed to patch a known Apache Struts vulnerability promptly after a fix became available in March 2017. Attackers exploited the vulnerability from May to July and accessed personal information relating to about 147 million people. Investigations also identified weaknesses in asset management, network segmentation, data protection, and monitoring. An expired digital certificate prevented a network-monitoring tool from inspecting encrypted traffic for months.

The incident demonstrates how basic control failures can combine:
- incomplete asset information can prevent a patching process from reaching an exposed system
- weak verification can allow a failed remediation task to remain undetected
- poor segmentation can increase the data available after initial access
- inadequate monitoring can extend an attacker's dwell time
- concentrated personal data can amplify long-term harm to individuals

The 2019 settlement with the US Federal Trade Commission, Consumer Financial Protection Bureau, states, and territories required Equifax to pay at least US$575 million and potentially up to US$700 million. The settlement included consumer relief and security obligations.
### Compliance and security
Compliance means satisfying applicable legal, regulatory, contractual, industry, and internal requirements. Security means managing risk through suitable governance, people, processes, and technology. The two overlap, but one does not establish the other.

| Requirement | Primary focus | Important distinction |
|---|---|---|
| General Data Protection Regulation | Personal-data processing in the European Economic Area and related territorial scope | It is European Union law with security, accountability, transparency, and data-subject obligations. |
| California Consumer Privacy Act, as amended | Privacy rights and duties for covered businesses handling California residents' personal information | It is a state privacy law, not a complete cybersecurity framework. |
| Health Insurance Portability and Accountability Act rules | Protected health information handled by covered US entities and business associates | The Privacy, Security, and Breach Notification Rules apply according to role and data. |
| Payment Card Industry Data Security Standard | Cardholder-data environments | It is an industry standard enforced mainly through contractual relationships, not a general privacy law. |
| Sarbanes-Oxley Act | Corporate governance, financial reporting, and internal control for covered US issuers | It does not prescribe a complete cybersecurity control set. |

An organisation should identify the requirements that apply to its location, role, data, contracts, and services. It should then map those requirements to controls, evidence, ownership, monitoring, and review. A checklist may confirm the presence of a control without establishing that the control works in practice.
### Marriott and Starwood
Attackers compromised Starwood's reservation environment before Marriott acquired Starwood in 2016. Marriott discovered the activity in 2018. The Information Commissioner's Office estimated that the incident affected about 339 million guest records worldwide, including about 30.1 million records associated with residents of the European Economic Area at the time of notification.

The Information Commissioner's Office imposed a final penalty of £18.4 million in 2020 for infringements during the period in which the General Data Protection Regulation applied. The regulator identified inadequate technical and organisational measures, including failures in monitoring and protection. The case highlights the need for security due diligence before an acquisition, verified remediation after completion, secure integration, data minimisation, and continuing oversight of legacy systems.
### Equiniti Trust Company
The US Securities and Exchange Commission described two incidents involving Equiniti Trust Company, formerly American Stock Transfer and Trust Company.

In 2022, an attacker hijacked an existing email conversation, impersonated a US issuer, and fraudulently instructed Equiniti to issue and liquidate shares. Equiniti transferred about US$4.78 million to overseas accounts and recovered about US$1 million.

In 2023, attackers used stolen Social Security numbers to create fake accounts that Equiniti's systems automatically linked to real shareholder accounts. The names did not match, but the matching Social Security numbers allowed the linkage. The attackers liquidated about US$1.9 million in securities, and Equiniti recovered about US$1.6 million.

Equiniti reimbursed affected clients and shareholders. In 2024, the Securities and Exchange Commission censured the company, ordered it to cease and desist, and imposed a US$850,000 civil penalty. The incidents support stronger out-of-band verification, identity matching, anomaly detection, manual review of high-risk changes, and tested fraud-response procedures.
### Selected regulatory outcomes
The legal basis, calculation method, appeal status, and finality of each outcome differ. Headline amounts should not be compared as if regulators applied one common formula.

| Year | Organisation | Outcome | Principal issue or current status |
|---|---|---|---|
| 2023 | Meta Platforms Ireland | €1.2 billion | The Irish Data Protection Commission addressed transfers of personal data from the European Union and European Economic Area to the United States. |
| 2021 | Amazon Europe Core | €746 million decision | A Luxembourg court annulled the fine in March 2026 and returned the case for reassessment. The 2021 amount is not a final current penalty. |
| 2021 | WhatsApp Ireland | €225 million | The Irish Data Protection Commission addressed transparency obligations under the General Data Protection Regulation. |
| 2020 | British Airways | £20 million | The Information Commissioner's Office addressed security failures connected with a 2018 personal-data breach. |
| 2020 | Marriott International | £18.4 million | The Information Commissioner's Office addressed inadequate security during the period covered by the General Data Protection Regulation. |
| 2022 | Didi Global | 8.026 billion yuan | China's cyberspace regulator found violations of the Cybersecurity Law, Data Security Law, and Personal Information Protection Law. |
| 2019 | Equifax | At least US$575 million, potentially up to US$700 million | The settlement resolved claims arising from the 2017 breach and included consumer relief and security obligations. |
