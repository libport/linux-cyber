# Cybersecurity Case Studies

> [!NOTE]
> This document uses real-world incidents to turn cybersecurity theory into practice. It provides a repeatable framework and template for analysing breaches: timeline, root cause, response actions, impact, lessons learned, and measurable recommendations. Topics span phishing and business email compromise (invoice fraud, vishing), point-of-sale malware and skimming (Target 2013, Home Depot 2014), insider threats and cloud misconfiguration, AI-enabled deepfake fraud, ransomware patterns and major disruptions, plus incident response, digital forensics, penetration testing, and compliance penalties.
## Analysing Case Study Layouts and Phishing Case Studies
### Exploring cybersecurity case studies
Case studies help organisations and learners translate cybersecurity theory into practice by examining real incidents. They support critical thinking, reveal how adversaries operate, and show how controls, processes, and people influence outcomes.

Benefits for a security team include:
- Building pattern recognition for likely threats and recurring attack paths
- Learning from past breaches, including technical gaps and process failures
- Comparing response approaches across different contexts and teams
- Staying current with attacker tactics and defensive controls
- Informing policy and governance by linking incidents to business impact and legal exposure

Some breaches drive major remediation and regulatory scrutiny. The 2017 Equifax breach is one example. It did not create the EU General Data Protection Regulation, which was adopted in 2016 and became applicable in 2018.
### Framework for analysing a cybersecurity case study
A structured method reduces missed details and unsupported assumptions:
1. Read the scenario end to end and map the timeline, key actors, and affected systems or data.
2. Identify the root cause, including the initial trigger and contributing factors.
3. Document actions taken, including dates, decision owners, and resources used.
4. Evaluate effectiveness and timeliness against intended outcomes.
5. Identify successes, gaps, and failures with specific evidence.
6. Analyse impact, including operational disruption, financial loss, and longer-term strategic effects.
7. Capture lessons learned, including what to repeat and what to change.
8. Produce recommendations that are actionable, prioritised, and measurable.
### Case study analysis template sections
A consistent template supports comparison across incidents:
- Root cause
- Actions taken
- Effectiveness and timeliness
- Successes, gaps, and failures
- Impact on the organisation
- Lessons learned
- Recommendations for future actions
- Conclusion and broader implications
### Phishing scams and related attacks
Phishing is a social engineering attack that tricks people into disclosing sensitive information or taking unsafe actions, usually by impersonating trusted entities. It commonly uses email and text messages, and increasingly uses phone calls and social media. Messages often create urgency or fear, and may include malicious links or attachments.

Common variants include spear phishing, whaling, vishing, smishing, and pharming. Typical lures include suspicious login alerts, account or payment problems, urgent invoice requests, and prompts to share credentials or one-time codes.
### Using threat trend reports
Threat trend reports based on large-scale telemetry, including DNS-layer data, help teams understand which malware and campaigns are most prevalent and how they evolve. This supports risk assessment, control selection, and targeted awareness training.
### Case study: invoice fraud against Google and Facebook
A business email compromise scheme from around 2013 to 2015 induced two US-based internet companies, widely reported as Google and Facebook, to wire payments to fraudulent accounts. The scheme impersonated a legitimate hardware supplier and used realistic emails and forged business documents to exploit weak invoice and payment verification. US prosecutors reported losses of more than US$120 million. The perpetrator, Evaldas Rimasauskas, was arrested in 2017 and sentenced to five years in prison in 2019.

Key lessons include:
- Treat bank detail changes and unusual invoices as high risk
- Require out-of-band verification and dual approval for payments
- Separate duties across request, approval, and execution
- Train finance and procurement staff to recognise business email compromise patterns
- Use technical controls such as domain authentication and anomaly detection
### Case study: voice phishing and authority impersonation
Voice phishing uses phone calls to apply pressure and drive disclosure or transfers. Reported cases have involved impersonation of officials, detailed personal information to build credibility, and threats of legal consequences to bypass verification.

Practical controls include:
- Independent call-back procedures using trusted numbers
- Transaction holds for unusual or high-value requests
- Regular scenario-based training for social engineering
- Clear reporting pathways and rapid escalation to incident response
- Monitoring for suspicious authentication and account activity
## Analysing PoS and Insider Breach Case Studies
Point of sale (POS) systems have moved from standalone cash registers to networked platforms that combine checkout, inventory, customer management, workforce reporting, and analytics. Industry standardisation such as barcode scanning reduced manual entry and errors. The Universal Product Code (UPC) standard was adopted in 1973 and first used in live retail scanning in 1974, supported by vendors including IBM.

Modern POS environments often include cloud services and mobile devices so staff can transact anywhere in a store. This convenience increases the need to treat POS as a critical part of the organisation’s security perimeter.

POS systems commonly store and process:
- Transaction data such as item, price, quantity, and time
- Inventory data such as stock levels, supplier details, and reorder points
- Customer data such as contact details, purchase history, and loyalty rewards
- Workforce data such as user activity, shifts, and sales performance
- Payment data such as card details and billing information
- Access control data such as user IDs, roles, and passwords

Common weaknesses include malware, card skimming, interception of poorly protected network traffic, phishing, insider misuse, weak authentication, and inadequate network security. Controls are often aligned with PCI DSS and broader security governance.
### POS malware and skimming
POS malware is malicious software designed to steal payment data from POS systems, often by capturing card data from memory before it is encrypted or transmitted. Skimming is the use of an unauthorised device on a payment terminal, ATM, or fuel pump to capture card data, and sometimes PINs.

Typical paths into a POS environment include:
- Stolen credentials, including third-party vendor access
- Social engineering and phishing that lead to malware installation
- Poorly secured remote access tools and weak password practices
- Misconfigured systems and delayed patching

A common malware pattern is RAM scraping, followed by data staging on internal systems and exfiltration to attacker controlled servers. Indicators can include unexpected outbound connections, unusual process names, and security alerts that go untriaged.

Response and prevention measures include:
- Disconnecting affected terminals from the network to limit spread
- Preserving logs and images for investigation
- Cleaning or reimaging terminals and rotating credentials
- Applying patches and hardening configurations
- Monitoring POS networks for suspicious activity
- Training staff to recognise phishing and handling procedures

Skimming controls focus on the physical layer:
- Regular inspection of terminals for overlays and tampering
- Tamper evident seals and locked enclosures
- CCTV coverage of checkout areas
- Restricting physical access and separating duties
- Staff training to spot distraction tactics
### Case study: Target POS breach in 2013
Public reporting on the 2013 Target incident describes attackers gaining initial access through a third-party vendor, moving into Target’s network, and deploying memory scraping malware on POS terminals. Reporting also indicates that stolen data was staged on internal servers and exfiltrated to external infrastructure over a period of time.

Mapped to a cyber kill chain, the incident is commonly:
- Reconnaissance of vendor facing systems and processes
- Delivery of phishing or malware to the vendor
- Credential theft and initial entry into the retailer environment
- Lateral movement to reach POS assets
- Installation of RAM scraping malware and collection of card data
- Command and control that maintained access for weeks
- Exfiltration of card data and customer records

Key response challenges included detecting and acting on alerts, constraining vendor access, and preventing movement from less trusted zones into cardholder environments.
### Case study: Home Depot POS breach in 2014
Reporting on the 2014 Home Depot breach describes attackers using stolen third-party credentials, exploiting a Windows vulnerability to increase access, and deploying memory scraping malware across thousands of self checkout terminals. The incident was not detected for months, increasing the scale of exposure.

Contributing factors commonly cited in incident write ups include:
- Weak segmentation between corporate systems and POS networks
- Legacy endpoints and insecure configurations
- Gaps in monitoring and alert response
- Missing or disabled security controls, including endpoint protections
- Limited use of strong encryption such as point to point encryption

The case reinforces the value of least privilege for vendors, rapid patching, and continuous monitoring of POS environments.
### Insider threats and detection
An insider is a person who has, or previously had, authorised access to an organisation’s systems, sites, data, or equipment. Insider threats can be malicious, negligent, or accidental, and can involve theft, sabotage, unauthorised disclosure, or workplace violence. Consequences can include financial loss, operational disruption, legal exposure, reputational damage, and harm to staff and customers.

An insider threat incident response plan typically defines roles, reporting paths, investigation steps, and legal and privacy requirements. Actions often include detection, triage, containment, evidence preservation, documentation, and notification where required.

A common behavioural progression model describes six stages:
- Grievance and ideation
- Preparation
- Exploration
- Experimentation
- Execution
- Escape

Understanding these stages supports earlier intervention and monitoring of risky behaviour.
### Mitigation program themes
Effective programs typically combine people, process, and technology:
- Ongoing staff engagement, screening, and security awareness training
- Asset identification and risk prioritisation, including who has access to what
- A repeatable operating model such as detect, identify, assess, and manage
- Cross functional governance across security, IT, HR, legal, and leadership
- Strong access control, privileged account monitoring, and zero trust principles
### Insider case studies
The Vault 7 leak refers to the public release of CIA cyber tool documentation by WikiLeaks in March 2017. US authorities later prosecuted former CIA employee Joshua Schulte, who was convicted of offences related to unauthorised disclosure. The case highlights the need to monitor privileged access, protect sensitive repositories, and investigate anomalies quickly.

Pegasus Airlines reported a 2022 incident involving an exposed Amazon S3 bucket that was publicly accessible due to misconfiguration. Researchers reported that the bucket contained airline operational data, software components, and credentials. The incident demonstrates how configuration errors and unsecured cloud storage can create large scale exposure, and why cloud access controls, logging, and staff training are essential.
### Summary
POS evolution links with modern breach risk. It emphasises that POS environments process high value payment and identity data, making them attractive targets for malware, skimming, credential theft, and insider misuse. Strong security relies on layered controls across technology, physical safeguards, vendor governance, monitoring, and consistent incident response practices.
## Analysing AI-Related Breaches and Ransomware Case Studies
Artificial intelligence can strengthen cyber security while also amplifying cybercrime. Below list common AI-enabled attack patterns, how defenders apply AI in security operations, and the risks that come with deploying AI.
### How attackers use AI
AI can increase the speed, scale and believability of attacks by automating decisions and tailoring content to targets.

- Phishing and social engineering, including personalised messages and synthetic audio or video impersonations
- Password guessing and credential abuse, where models learn likely patterns from leaked datasets
- Malware that changes code or behaviour to evade signature-based tools
- Faster vulnerability discovery and exploitation through automated code and traffic analysis
- Reconnaissance at scale, including public-data harvesting to select targets, timing and techniques
- Ransomware optimisation, including selective encryption of high-value data to maximise leverage
### How defenders use AI
AI can improve detection, triage and response when it is integrated into security processes and monitored.

- Threat detection and response using real-time analysis of high-volume logs and alerts
- Behavioural analytics that flags unusual user and system activity
- Continuous monitoring and automated auditing to identify indicators of compromise
- Vulnerability management that helps prioritise patching based on risk and likely impact
- Spam and phishing filtering that adapts to evolving attacker techniques
- Fraud detection through anomaly detection in transactions and account activity
- Predictive analytics that uses prior incidents and precursor signals to forecast likely attack paths
### Governance and safety considerations
AI outcomes depend on data quality and system design. Biased, incomplete or poisoned training data can drive false positives, missed attacks, or unsafe decisions. Security programmes also need to manage privacy and misuse risks through access controls, model and prompt security, and clear policies. Ongoing monitoring, human oversight and workforce training remain essential.
### Industry context on AI adoption in security
Splunk’s State of Security 2024 research reports that 91% of security teams use generative AI in security operations, yet 65% say they do not fully understand its implications. The same research reports broad public generative AI use across organisations and notes that 34% do not have a generative AI policy.
### Case study: deepfake video conference fraud
Hong Kong police reported a fraud in which a finance employee transferred HK$200 million, about US$25 million, after seeing and hearing convincing synthetic versions of senior staff on a video call. The case shows how deepfakes can defeat informal verification habits in finance workflows.

- Treat urgent and confidential payment requests as high risk
- Require multi-person approval for high-value transfers
- Use out-of-band verification, such as direct calls to known numbers, before releasing funds
- Improve staff awareness of deepfake tactics and social engineering pressure
### Examples of AI-adjacent incidents and techniques
Several public incidents where automation and large-scale data extraction feature prominently:
- TaskRabbit, April 2018: the service took its platform offline to investigate a cyber security incident and advised users to change passwords where reused
- Facebook contact data exposure publicised in 2021: a dataset affecting 533 million users was linked to scraping and contact importer features, with Ireland’s regulator later issuing a €265 million fine in 2022
- LinkedIn data for sale reports, 2021: LinkedIn stated the data was scraped from public profiles and was not the result of a breach of its systems
- Yum Brands, January 2023: a ransomware incident disrupted certain IT systems and temporarily closed nearly 300 UK restaurants for one day
- Activision, December 2022: reporting described a successful phishing incident that led to unauthorised access to internal data, including employee information
### Ransomware fundamentals and trends
Ransomware encrypts systems or data to coerce payment. Modern campaigns commonly add extortion by stealing data and threatening to publish it.

- Double extortion: encryption plus a threat to leak stolen data
- Triple extortion: added pressure on customers or partners through additional threats or contact
- Ransomware as a Service: packaged tooling and infrastructure sold to affiliates

AI can further increase efficiency by improving target selection, refining phishing, and accelerating vulnerability discovery. Many law enforcement agencies discourage ransom payments because payment can fund further crime, does not guarantee recovery, and can create legal risks in sanctioned jurisdictions.
### Case studies: City of Atlanta and Colonial Pipeline
- City of Atlanta, March 2018: SamSam ransomware disrupted municipal services, while emergency response continued through manual workarounds. The incident highlights the cost of deferred IT maintenance and weak credential controls.
- Colonial Pipeline, May 2021: attackers entered via a compromised VPN password without multi-factor authentication, stole data and deployed ransomware, prompting a shutdown and fuel shortages. The case reinforces the need for MFA, credential hygiene and coordinated response.
### Practical safeguards
- Harden identity controls, including MFA and strong password policies
- Apply least privilege and monitor privileged access
- Patch and reduce known vulnerabilities with risk-based prioritisation
- Maintain tested backups and restoration procedures
- Train staff on phishing, impersonation and deepfake risks
- Build and rehearse incident response and business continuity plans
## Analysing Incident Response and Digital Forensics Case Studies
### Incident response in cybersecurity
Incident response is a coordinated set of proactive and reactive activities that reduce the likelihood and impact of cyber attacks and restore services quickly. It includes technical work plus legal, human resources, and communications coordination.

Proactive controls commonly include monitoring, vulnerability assessment, and patch management. Security information and event management systems can centralise logs and flag anomalies. Automated response tools may isolate compromised assets to contain malware.

Many organisations document this in an incident response plan that defines detection, containment, recovery, and reporting steps. Plans should be updated using lessons learned and changing threats.
### Learning from incident response case studies
Case study reviews help teams understand why an incident happened, whether controls and procedures were effective, and what should change. They can also surface attacker tactics, techniques, and procedures and indicators of compromise, such as suspicious logins, abnormal traffic, and malware signatures.

Improvements often include:
- Stronger phishing resistance through training and email security
- Faster remediation through disciplined patching
- Strong password policies and multi-factor authentication
- Secure coding and code review to reduce injection risks
- Tighter access controls, monitoring, and network resilience
### Case study: Google Home rollout and quota incident
During a Google Assistant rollout affecting Google Home, an on call engineer saw an unexpected rise in queries per second. The rollout was halted and limited to a partial deployment, but the team did not declare an incident and initially pursued an incorrect suspected cause. Miscommunication between client and server teams slowed diagnosis.

The rollout later resumed, and once it reached full deployment, traffic again exceeded server limits. Users experienced errors and complaints increased. Escalation to support and site reliability engineering occurred late. Analysis confirmed a quota issue, and a quota increase reduced impact and stabilised the service.

Key lessons include declaring incidents early, involving the right responders immediately, and completing root cause analysis before resuming a rollout.
### Case study: Lightning strike power event at a Google data centre
Power events are a common contributor to data centre disruptions. In a widely discussed 2015 incident, lightning strikes disrupted a local utility supplying a Google data centre in Belgium. Generators started and batteries bridged the start up period, but repeated strikes contributed to a brief loss of power to some disk trays. Servers stayed powered but temporarily lost access to affected disks, affecting virtual machine read and write operations.

The storage site reliability engineering team declared a major incident and set clear roles, including an incident commander, operations lead, and communications lead. Response priorities included restoring stable power, recovering infrastructure, mitigating impact, and communicating accurate status. Post incident analysis reported minimal data loss and commitments to reliability improvements.
### When incident response goes wrong
Common contributors include:
- Backup systems and plans that are outdated, fragile, or untested
- Unavailable key staff and unclear role definitions
- Missed detection, misdiagnosis, and slow escalation
- Weak vendor and third-party risk controls
- Poor communication that delays action and erodes trust
### Digital forensics overview
Digital forensics uses disciplined methods to collect, preserve, and analyse digital evidence for cybercrime and incident investigations. Handling follows a chain of custody so integrity and admissibility can be demonstrated. Computer forensics is narrower, focusing on computers, while digital forensics spans many device types.
### Case studies: Bernard Madoff and United States v. Ganias
The Bernard Madoff investigation illustrates effective forensic practice. Investigators created forensic images to avoid altering originals and analysed financial records, emails, and metadata to trace money flows and establish timelines. Findings supported prosecution outcomes, including a guilty plea in March 2009 and a 150 year prison sentence in June 2009.

United States v. Ganias illustrates legal risk. Investigators imaged drives under a warrant for specific targets, which also captured unrelated client data. Years later, retained images were searched for a different purpose. Courts found the approach unconstitutional, and key evidence was excluded.
### When digital forensics goes wrong
Examples include:
- The Casey Anthony trial, where expertise and interpretation were challenged
- Connecticut v. Amero, where malware driven pop ups were initially overlooked
- Mass v. Michael Fiola, where malware may have downloaded illegal content without the user’s knowledge
- Porcha Woodruff, where a facial recognition match allegedly contributed to arrest without adequate corroboration

Better outcomes rely on skilled analysts, validated tools, malware aware interpretation, strict legal compliance, and corroboration beyond a single signal.
## Analysing Penetration Testing and Compliance Case Studies
### Penetration testing
Penetration testing is a controlled security assessment that simulates attacker behaviour to identify and validate weaknesses in systems, applications, and networks. It is performed by penetration testers, also known as ethical hackers, under an agreed scope and written authorisation.
#### Why organisations commission pen tests
- They demonstrate real impact by attempting exploitation, not only detection.
- They confirm which findings are truly exploitable and reduce false positives.
- They support assurance and compliance by testing whether controls work in practice.
#### Common tool categories
- Security-focused operating systems that bundle assessment utilities.
- Password and credential testing tools.
- Port scanners for reachable services.
- Vulnerability scanners for known issues and misconfigurations.
- Packet analysers for network traffic inspection.
### Why case studies matter
- They show how attacks are executed and what consequences follow.
- They expose recurring weaknesses that can be prioritised for prevention.
- They illustrate remediation approaches and what effective reporting looks like.
- They demonstrate gaps that automated scanning can miss.
- They provide reference patterns for scoping, ethics, and follow-up.
### Case study summary: Equifax breach (2017)
The Equifax breach exposed sensitive identity data for about 147 million people. Reviews highlighted an unpatched Apache Struts remote code execution flaw (CVE-2017-5638), inconsistent patching, weak segmentation and access controls, and gaps in monitoring and logging. Delayed remediation of critical findings enabled exploitation. The incident triggered major legal and financial consequences, including a global settlement totalling at least US$575 million, and potentially up to US$700 million, with up to US$425 million allocated for consumer support.
### Case study summary: IBM X-Force Red phishing engagement
IBM X-Force Red provides offensive security services such as penetration testing and adversary simulation. In a phishing engagement, work commonly follows:
- Reconnaissance using open-source intelligence such as websites, public records, and staff social profiles.
- Email design that mimics trusted communications, with a spoofed domain and a credential-harvesting login page.
- Controlled distribution and measurement of opens, clicks, and credential submissions.
- Analysis to quantify success rates and identify filtering and awareness gaps.
- Reporting with prioritised recommendations and remediation actions.
### Pen testing tales: physical and social engineering
Physical and social routes can bypass strong technical controls:
- Internet-accessible cameras and poor placement can expose credentials.
- Perimeter and asset controls can fail when unattended devices are accessible.
- Tailgating and timing can enable unauthorised entry and data or device theft.
- Legal risk rises when scope, documentation, and escalation paths are unclear.

These scenarios reinforce the need for clear rules of engagement and basic physical controls, including access management, device security, and monitoring.
### Cybersecurity compliance
Cybersecurity compliance is the handling of personal and sensitive data in line with law, industry standards, and organisational policy. Common objectives include transparency, accuracy, enforceable individual rights, and secure handling across the data lifecycle. Regulations and frameworks commonly referenced include:
- GDPR for EU data protection and privacy.
- HIPAA for protection of health information in the United States.
- PCI DSS for safeguarding payment card data.
- SOX for governance and financial reporting controls in public companies.
- CCPA for consumer privacy rights in California.

Noncompliance can increase cyber risk and lead to penalties, litigation, and reputational harm. Case studies and benchmarking help organisations compare practices with peers, justify investment, and align programs with regulator expectations.
### Compliance case study summary: Marriott and the Starwood breach
The breach began in Starwood systems before Marriott’s 2016 acquisition and was detected in 2018. The UK Information Commissioner’s Office linked the incident to exposure of about 339 million guest records globally and issued a final fine of £18.4 million for GDPR security failings. The case highlights weaknesses such as insufficient monitoring of privileged activity, missing defence-in-depth, and inadequate protection of sensitive fields, including a lack of encryption for some passport data.
### Compliance case study summary: Equiniti Trust Company
Equiniti Trust Company LLC, formerly American Stock Transfer and Trust Company LLC, suffered two cyber incidents involving client funds and securities.

- In 2022, an attacker compromised an email exchange and sent fraudulent instructions that led to the release and sale of newly issued shares, with proceeds of about US$4.78 million sent to an external bank account.
- In 2023, an attacker used synthetic identity fraud and weaknesses in account linking to access customer holdings and move about US$1.9 million to external accounts.

The incidents prompted remediation such as stronger verification of payment instructions and multilayer authentication that does not rely on a single identifier. US regulators later announced enforcement action, including a civil penalty of US$850,000 and obligations to improve controls protecting funds and securities.
### Selected large cybersecurity and privacy penalties
Amounts below refer to publicly reported final penalties or settlements, not initial notices of intent.

| Year | Organisation | Amount | Regulator or basis | Typical driver |
| ---- | --------------- | -------------------: | ------------------------ | --------------------------------- |
| 2023 | Meta | €1.2 billion | Irish DPC and EU process | EU to US data transfers |
| 2021 | Amazon | €746 million | Luxembourg authority | GDPR advertising consent findings |
| 2021 | WhatsApp | €225 million | Irish DPC | GDPR transparency findings |
| 2020 | British Airways | £20 million | UK ICO | 2018 payment data breach |
| 2020 | Marriott | £18.4 million | UK ICO | Starwood breach security failings |
| 2022 | Didi | 8.026 billion yuan | China regulator | data security violations |
| 2017 | Equifax | up to US$700 million | US settlement | identity data breach |
