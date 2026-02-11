# Introduction to Cybersecurity Tools & Cyberattacks

> [!NOTE]
> This primer connects cyberattacks to major events and early culture like WarGames. Showing how priorities, policy, and tooling evolved. Especially in national security in the context of the U.S. Department of Homeland Security. It introduces critical-thinking techniques for triage and root-cause analysis, then surveys threats. AI-boosted phishing and OSINT targeting, malware and ransomware, insider risk, sniffing, spoofing, DoS, and injection. It also outlines controls and operations: IAM (SSO, passkeys), vulnerability scanning (Tenable Nessus; Greenbone Networks OpenVAS), and incident response/forensics aligned with NIST and SANS Institute.
## Cybersecurity Insights
### Cybersecurity lessons from major events and critical thinking
#### 9/11 and the shift in cyber priorities
- The September 11, 2001 attacks highlighted how online communications and publicly available information can support planning, coordination, recruitment, and propaganda.
- Post-incident investigation also demonstrated the value of digital forensics, including analysis of email artefacts, browsing traces, and other recoverable activity.
- In the aftermath, opportunistic online attacks increased, including website defacements, denial-of-service activity, and scam and spam campaigns that exploited public attention.
- In the United States, cybersecurity was increasingly a national security issue. This contributed to changes such as the creation of the Department of Homeland Security and programs focused on cyber defence.
- Policy responses also expanded surveillance authorities under the USA PATRIOT Act, followed by later reforms under the USA FREEDOM Act that narrowed certain bulk collection practices and added transparency measures.
- Security practice also evolved toward stronger encryption for sensitive communications, earlier integration of security into system design, and wider information sharing across agencies to reduce strategic surprise.
#### COVID-19 and cyber risk at scale
- The COVID-19 pandemic accelerated remote work and cloud adoption, increasing exposure through home networks, rapid rollout of collaboration tools, and broader reliance on personal devices.
- Attackers exploited pandemic themes to drive phishing and malware delivery, often using urgent or authoritative messaging.
- Risk factors included inconsistent patching and endpoint protection on personal devices, weaker home network security, and increased human error under stress.
- Public advisories during 2020 warned of increased malicious activity, including abuse of video conferencing and collaboration platforms.
- Ransomware activity affected critical services, including hospitals. The 2020 incident at University Hospital Düsseldorf is widely cited as an example of operational disruption and patient safety impact.
- Organisational responses commonly included expanded security awareness training, stronger authentication, improved email and cloud storage practices, and wider deployment of endpoint protection.
#### Critical thinking as a cybersecurity capability
- Critical thinking is the disciplined practice of analysing information and forming reasoned judgements based on evidence.
- In cybersecurity, it supports threat assessment, source evaluation, ethical decision-making, incident triage, and long-term improvement rather than short-term fixes.
- Practical applications include cross-checking vulnerability claims before patching, analysing suspicious emails before interacting, and conducting root cause analysis after incidents.
### A practical critical thinking model
#### Core elements
- Characteristics of thought: curiosity, scepticism, objectivity, and a focus on truth seeking.
- Technical skills: relevant security and IT capabilities, including secure configuration, network concepts, malware awareness, and proficiency with security tools.
- Interpersonal skills: clear communication, teamwork, negotiation, and project management to drive decisions and coordinated action.
- Theoretical and experimental knowledge: security principles combined with hands-on practice that tests assumptions in real environments.
- Intellectual abilities: memory, attention, logical reasoning, and problem-solving that support analysis under pressure.
#### Five critical thinking skills in practice
- Challenge assumptions by listing what is being taken for granted and testing it against evidence.
- Consider alternatives by exploring plausible actors, motives, timelines, locations, and methods.
- Evaluate data by assessing source quality, collection methods, and relevance before drawing conclusions.
- Identify key drivers such as attacker incentives, system weaknesses, and potential business impact.
- Understand context by relating a specific issue to broader threat trends, organisational constraints, and stakeholder needs.
### WarGames and early US cyber policy
- The 1983 film WarGames helped popularise concerns about unauthorised access and automation risk in military and government systems.
- In 1984, the United States issued National Security Decision Directive 145 to strengthen protection of telecommunications and automated information systems.
- The directive supported coordinated threat and vulnerability evaluation, protection of classified information handled electronically, and guidance for improving security across government and relevant private sector systems.
### Consolidated takeaways
- Major events can rapidly reshape attacker behaviour, defender priorities, and government policy.
- Technical controls are necessary, but decisions improve when analysis is evidence-led, context-aware, and ethically grounded.
## Cybersecurity Threats
### Social engineering, phishing, and the impact of AI
#### What social engineering targets
Social engineering exploits human behaviour rather than technical weaknesses. Common levers are urgency, fear, curiosity, and reward.

Typical attacker outcomes include:
- credential theft and account takeover
- installation of malware, including remote access tools
- extraction of intellectual property or other confidential data
- fraud, including unauthorised payments and gift card scams
#### Generative AI and phishing capability
Experiments and industry reporting show that generative AI can draft persuasive phishing content quickly, especially when prompts include:
- the target industry and role
- likely staff concerns and incentives
- instructions to use persuasion and marketing language
- guidance on who the message should appear to come from

Human-crafted lures can still outperform automated drafts when attackers invest time in reconnaissance, but AI reduces cost and raises scale.

AI also weakens traditional training cues such as spotting poor grammar, because fluency is no longer a reliable signal.
#### OSINT and targeting accuracy
Open source intelligence can improve the credibility of phishing by using public sources such as:
 - professional profiles and job titles
 - corporate blogs and media releases
 - conference agendas and travel announcements
 - review sites and public discussions of workplace issues
- Targeting improves when the lure aligns with a realistic event, such as a training invitation, a benefits update, a device refresh, or a policy change.
### How phishing works in practice
#### Common digital vectors
Phishing often delivers one of two outcomes:
 - a link to a lookalike site that harvests credentials
 - an attachment or download that installs malware
- Variants include:
 - spear phishing, aimed at a specific person or team
 - whaling, aimed at executives and other high privilege roles
 - smishing, delivered via SMS
 - vishing, delivered via phone calls
 - search engine manipulation, where malicious sites are pushed higher in results
 - lookalike domains, where minor spelling changes are used to trick the eye
#### Physical vectors and blended attacks
- Tailgating, where an unauthorised person follows an authorised person through a controlled entry point.
- Shoulder surfing, where passwords, PINs, and screens are observed in public.
- Dumpster diving, where confidential information is recovered from unsecured waste.
- Many incidents combine digital and physical methods, such as a phone call that pressures a staff member to approve access, followed by email instructions and a link.
#### Impersonation and deepfakes
Impersonation may involve pretending to be:
 - IT support
 - a supplier or partner
 - a manager or executive
 - a government agency or bank
- Voice and video deepfakes can increase realism. The defensive response is procedural, not visual recognition.
- Strong procedures include call-backs to known numbers, approval workflows for sensitive actions, and rules that prevent sending corporate data to personal accounts.
### Defences that remain effective
#### A verification-first habit
- Unexpected requests for money, credentials, access, or confidential information are verified through an independent channel.
- Staff do not use contact details provided in the message. They use known directories, internal portals, or verified vendor contact lists.
- Requests that include urgency or consequences, such as job loss, account deactivation, or missed payments, are treated as high risk and slowed down.
#### Link and attachment discipline
- Users avoid clicking links in unsolicited messages. They navigate to known sites directly or use bookmarks.
- Attachments are treated as untrusted until the sender and context are confirmed.
- Suspicious messages are reported, not forwarded broadly, so security teams can contain and block related lures.
#### DNS and web filtering
- DNS security and web filtering can block known malicious domains and reduce exposure to lookalike sites.
- Public DNS security services exist, but effectiveness depends on current threat intelligence and correct configuration.
- DNS filtering supports user awareness training but does not replace it.
### Organisational prevention controls
#### Identity and access management
- Multi-factor authentication reduces account takeover risk when passwords are stolen through phishing or credential reuse.
- Passwordless approaches such as passkeys based on FIDO standards reduce exposure to credential theft because there is no reusable password to hand over.
- Access control follows least privilege and role alignment, so accounts cannot reach systems and data they do not need.
- Privileged access is separated from everyday work. Administrator accounts are used only when required.
#### Email, endpoint, and network protection
- Email filtering and secure gateways reduce delivery of malicious links and attachments.
- Endpoint protection helps detect and block malware at execution time, including suspicious script behaviour.
- Patch management reduces exposure to known vulnerabilities that attackers use after a successful click.
- Monitoring focuses on authentication events, unusual data access, and atypical outbound connections.
#### Testing and asset focus
- Penetration testing and security assessments identify weaknesses before attackers do, followed by prompt remediation.
- Critical asset identification helps prioritise controls around systems and data that drive business continuity and competitive advantage.
#### Awareness and a strong reporting culture
- Training focuses on manipulation patterns, not cosmetic cues.
- Programmes promote reporting without blame, so near misses become learning opportunities.
- Organisations limit unnecessary public exposure of staff roles, internal processes, and travel details that can increase targeting accuracy.
### Preventing physical social engineering
- Awareness training covers shoulder surfing, visitor escort rules, and how to challenge unknown individuals politely.
- Safe disposal practices include shredding sensitive paper and securely destroying media before disposal.
- Access control controls tailgating through individual authentication at entry points and clear policy enforcement.
- Security staff and surveillance can deter unauthorised entry, but staff behaviour remains essential.
### Malware and ransomware essentials
#### Key malware categories
- Malware is software designed to disrupt, damage, or gain unauthorised access. Common categories include:
 - viruses, which infect files and commonly require execution
 - worms, which can spread by exploiting weaknesses
 - trojans, which disguise themselves as legitimate software
 - spyware and keyloggers, which collect credentials and activity
 - fileless malware, which runs in memory and leaves fewer file traces
 - bot malware, which forms botnets controlled by command and control servers
 - rootkits, which hide malicious activity and support privileged access
 - ransomware, which locks or encrypts data for extortion
#### Ransomware impact patterns
- Ransomware pressure commonly takes two forms:
 - data loss and operational disruption, where encryption blocks access to files and systems
 - data breach and extortion, where stolen data is threatened with public release
- Backups are a primary control for recovery from encryption and destruction, but they do not prevent data exposure.
- Strong access control and encryption reduce the usefulness of stolen data, particularly when encryption keys are protected.
#### Prevention priorities
- Maintain regular, tested backups that support restoration to specific points in time.
- Keep operating systems and applications updated to close known entry points.
- Use reputable anti-malware tools with real-time protection and current detection updates.
- Limit the ability for malware to run by controlling administrative privileges and using application controls where practical.
- Reinforce safe handling of attachments and downloads, including verification and sandboxing where available.
#### Rootkits and server scanning
- Rootkits aim to provide stealthy administrator-level control and can conceal other malware.
- On Linux and Unix systems, tools such as rkhunter can scan for known indicators, anomalies, and misconfigurations.
- Rootkit hunting tools work best as part of a layered approach that includes intrusion detection, patching, secure configuration, and log review.
### Insider threats
#### What makes insider risk distinct
- Insider incidents originate from people with authorised access, so they can bypass some perimeter controls.
- Insider harm may be deliberate or accidental.
#### Common types
- Oblivious insider, who acts without understanding the risk, such as responding to a convincing phishing lure.
- Negligent insider, who ignores policy for convenience or overconfidence.
- Malicious insider, who abuses access to steal data or cause disruption.
- Professional insider, who is recruited to conduct espionage for an external party.
#### Risk reduction
- Use least privilege, strong authentication, and logging across sensitive systems.
- Apply separation of duties for high impact actions, such as payments, access approvals, and data exports.
- Provide role-specific training and reinforce secure handling of data.
- Offboarding processes remove access quickly and verify that devices and credentials are recovered or disabled.
### Threat actors and motives
#### Major categories
- Hacktivists pursue political or social goals and often use defacements and denial-of-service to attract attention.
- Organised crime pursues financial gain through ransomware, fraud, and identity theft.
- Nation state actors pursue espionage and geopolitical advantage, including targeting critical infrastructure and intellectual property.
- Script kiddies use existing tools for curiosity or notoriety and can still cause serious disruption.
- Insider threat actors act from within an organisation, either deliberately or through mistakes.
#### Why actor analysis matters
- Motives and capability influence which attacks are most likely.
- Actor analysis helps security teams prioritise controls, anticipate likely attack paths, and prepare response plans that match business risk.
## Cybersecurity Controls
### Cybersecurity operations essentials
#### Vulnerability management workflow
- Vulnerability management is a continuous process to identify, assess, and reduce weaknesses across systems and applications.
- Effective programs start with an accurate asset inventory so scan targets include all servers, endpoints, network devices, and applications that connect to or support the network.
- Assets are prioritised by business criticality, exposure, and the sensitivity of stored or processed data, which drives scan frequency and remediation urgency.
- Routine scanning is scheduled and automated where possible, with tools kept current so detections reflect recent vulnerability research.
- Scans are commonly run in off-peak windows to reduce disruption, while still providing timely coverage.
#### Scanning tools and selection criteria
- Scanners are chosen for coverage, reporting quality, update cadence, scalability, and integration with existing workflows.
- Reports should be actionable and support triage, ownership assignment, and verification after remediation.
- A mixed toolset can be appropriate when it improves coverage or reduces blind spots.
- Nessus is a widely used commercial scanner known for a large plug-in library and frequent updates.
- OpenVAS is a widely used open source scanner that can provide strong baseline coverage, with updates that may vary by community release cycles.
#### Application vulnerability scanning in the SDLC
- Application security testing is often embedded into the software development lifecycle.
- Static application security testing inspects source code without executing it to identify risky patterns and defects.
- Dynamic application security testing evaluates a running application by probing user-facing interfaces.
- Interactive application security testing combines aspects of static and dynamic techniques by analysing behaviour while the application runs under test.
- Many organisations require a defined set of tests to pass before deployment to production.
#### Scan configuration and quality
- Scan settings balance depth against system load and false positives, and are tuned for each asset class.
- Credentialed scans usually improve accuracy because they provide deeper visibility into configuration and patch state.
- Findings are prioritised by exploitability and impact, not only by severity score.
### Incident response and digital forensics
#### Why incident response is treated as core capability
- Security incidents are expected over time, including malware, ransomware, insider misuse, and human error.
- Incident response is most effective when treated as an ongoing program rather than an ad hoc emergency activity.
#### Incident response life cycle
- A common model includes preparation, detection and analysis, containment and eradication with recovery, and post-incident improvement.
- Preparation covers roles, communication pathways, decision authority, tooling, and rehearsed procedures.
- Post-incident activities capture lessons learned and drive control improvements, including patching and process changes.
#### Digital forensics in incident response
- Digital forensics supports incident response by identifying, preserving, extracting, and documenting evidence from drives, memory, logs, and other artefacts.
- Chain of custody is maintained through chronological documentation of who handled evidence, when it was handled, and what was done, to support reliability and legal defensibility.
- Timelines and event correlation help explain what occurred, how it occurred, and what was affected.
### Network threats and common attacker techniques
#### Packet sniffing basics
- Network packets typically include a header for routing information and a payload containing the data.
- Packet sniffing captures and analyses packets to diagnose performance and detect anomalies on networks under an organisation’s control.
- Unauthorised sniffing can expose credentials and personal or financial data, especially when traffic is unencrypted.
- Passive sniffing observes traffic without altering it and is common on shared networks such as public Wi-Fi.
- Active sniffing manipulates traffic flows on switched networks, often through techniques such as ARP spoofing, to increase visibility of other hosts’ traffic.
- Risk reduction relies on encryption in transit, strong authentication, careful Wi-Fi use, and secure network design.
#### IP spoofing and its uses
- IP spoofing forges the source address in packet headers to conceal origin or impersonate another host.
- Spoofing is commonly used in reflection and amplification DDoS attacks, where third-party systems are tricked into sending large responses to a victim address.
- Spoofing can also complicate attribution for botnet activity and can support on-path attacks when combined with other weaknesses.
- Mitigation focuses on filtering rather than elimination.
- Ingress filtering and egress filtering reduce the flow of spoofed traffic and align with best practice guidance such as BCP 38.
#### Denial of service patterns and defences
- Denial of service aims to reduce availability through crashes, protocol abuse, or resource exhaustion.
- Resource exhaustion attacks include SYN floods, where large volumes of incomplete connection attempts consume server capacity.
- Distributed denial of service uses many compromised systems in a botnet to increase scale and reduce simple filtering effectiveness.
- Strong defensive posture combines redundancy, rate limiting, upstream filtering, patching, hardening, monitoring, and rehearsed incident response playbooks.
#### Injection attacks in web applications
- Injection attacks occur when untrusted input is interpreted as commands or queries by an application or backend system.
- SQL injection can expose confidential data, bypass authentication, alter authorisation, and damage integrity through unauthorised changes.
- Cross-site scripting occurs when a browser executes attacker-supplied script in a trusted site context.
- Common XSS categories include reflected, stored, and DOM-based variants, and impact often includes session theft and content manipulation.
- Prevention relies on input validation, parameterised queries, output encoding, safe framework defaults, and least privilege database access.
### Security controls and control functions
#### Control families
- Administrative controls set expectations through policy, training, and governance.
- Physical controls protect facilities and equipment through barriers, monitoring, and environmental safeguards.
- Technical controls use hardware, software, or firmware to enforce security outcomes and automate detection and response.
#### Control functions with examples
| Function | Typical focus | Examples |
| --- | --- | --- |
| Preventive | Stop incidents before they occur | access policies, MFA, locks, firewalls |
| Detective | Identify incidents quickly | audits, CCTV, IDS, SIEM |
| Deterrent | Discourage unsafe behaviour | awareness programs, warning banners, lighting |
| Corrective | Restore and improve after issues | patching, quarantine, incident response, repairs |
### System, network, and application security practices
#### System security essentials
- Access control limits who can use resources, supported by strong authentication and appropriate privilege management.
- Encryption protects confidentiality for data at rest and in transit, including full-disk encryption for mobile devices.
- Patch management reduces exposure to known vulnerabilities across operating systems, applications, and firmware.
- Regular backups support recovery from faults and ransomware, with strategies selected to fit recovery objectives.
- Host firewalls and endpoint protection reduce exposure when perimeter controls fail.
#### Network security essentials
- Network security aims to block unauthorised access, detect and stop active threats, and enable safe access for legitimate users.
- Defence in depth layers controls at the perimeter and internally to reduce lateral movement.
- Common capabilities include firewalls, network access control, intrusion detection and prevention, VPN for remote access, segmentation, endpoint security, SIEM for visibility, and SOAR for response automation.
#### Application security by design
- Application security is applied across planning, design, build, test, deploy, and maintain stages.
- Secure coding practice includes input validation, safe error handling, secure logging, removal of hardcoded secrets, strong access control, and use of reviewed cryptography libraries.
- Security testing aims to identify weaknesses before release through code review, automated scanning, and targeted testing.
#### Network mapping and how it is used
- Network mapping discovers and visualises physical and logical connections, devices, open ports, services, and data paths.
- Nmap supports discovery and security auditing through host identification, port scanning, service detection, and scripting.
- Wireshark is a protocol analyser that captures traffic and supports deep inspection for troubleshooting and anomaly investigation.
- Accurate maps support troubleshooting, capacity planning, documentation, and rapid detection of unauthorised changes.
- Attackers use mapping for reconnaissance to identify exposed services, infer high-value assets, and plan lateral movement for persistent access.
## Identity and Physical Controls
### Identity, authentication, and authorisation
#### Purpose and the four As
- Identity and access management reduces risk by ensuring that only the right identities can access the right resources at the right time.
- A common framing is the four As.
- Administration provisions, updates, and removes accounts.
- Authentication verifies that an entity is who it claims to be.
- Authorisation decides what an authenticated identity is allowed to do.
- Audit provides evidence that the first three steps occurred as intended through logging and review.
#### Authentication protocols and how they are used
- Authentication protocols define how systems prove identity and establish trust for access decisions.
- RADIUS is a client server protocol used for centralised authentication, authorisation, and accounting, and it is commonly used for Wi-Fi and VPN access.
- CHAP is a legacy challenge response protocol. It improves on sending cleartext passwords but is not regarded as a modern control on its own.
- EAP is a framework that supports multiple authentication methods for network access, including EAP-TLS and other EAP types used in enterprise Wi-Fi.
- Kerberos is a network authentication protocol developed at MIT that uses a trusted Key Distribution Centre and ticket based authentication, and it is widely used in Microsoft Active Directory environments.
#### Authentication servers and credential stores
- Authentication servers store and validate credentials and issue decisions or tokens used by relying systems.
- Common examples include Active Directory, LDAP directory services, and RADIUS servers.
- Centralised identity stores reduce duplication but increase the need for strong hardening, monitoring, and privileged access controls.
#### Authentication methods and factor strength
- Passwords remain common but are exposed through phishing, credential reuse, malware, and offline cracking after password database breaches.
- multi-factor authentication reduces takeover risk by requiring evidence from different factor categories.
- One time passwords provide a single use code but are weaker when delivered by SMS due to SIM swap and interception risks.
- Authenticator apps and hardware security keys typically provide stronger resistance to phishing and replay.
- Biometrics can improve usability, but biometric data needs strong privacy protections and secure storage.
- Smart cards and security tokens can store keys or generate codes to support stronger authentication.
- Push approvals can reduce user effort, but they require anti fatigue controls and careful verification of the login context.
### Access control models and permissions
#### Access control schemes
- Role based access control assigns permissions to roles aligned to job functions, which simplifies administration and supports least privilege.
- Attribute based access control uses attributes about the user, resource, action, and context to make decisions.
- Rule based access control applies rules, policies, or access control lists to resources, with firewall rules as a common example.
- Mandatory access control enforces centrally defined rules that users cannot change, and it is common in high assurance environments.
- Discretionary access control allows owners to grant access to their objects, and it is common in desktop operating systems.
#### File access controls
- File permissions limit read, write, and execute actions, with different implementations across operating systems.
- Linux represents permissions with read, write, and execute flags on files and directories.
- Windows commonly manages permissions through access control lists and security descriptors, and executable behaviour also depends on file type and policy.
#### Least privilege and accountability
- Least privilege reduces the blast radius by limiting access to what is necessary and removing access promptly when roles change.
- Access activity is monitored through logs that record authentication events, permission changes, and sensitive data access.
- Non repudiation aims to reduce denial of actions and is supported by mechanisms such as digital signatures, trusted timestamps, and tamper resistant logging.
### Single sign on, password managers, and passkeys
#### What SSO is and what it is not
- Single sign on allows one sign in with an identity provider to provide access to multiple services through federation or central session management.
- A password manager stores and auto fills credentials. It improves password hygiene but does not provide SSO by itself.
- SSO can reduce password fatigue, but it increases the impact of one compromised identity, so it is commonly paired with MFA, strong session controls, and monitoring.
#### Passkeys and passwordless authentication
- Passkeys are based on public key cryptography under FIDO standards and authenticate by signing a server challenge without revealing a reusable secret.
- The private key remains on the user device or in a protected synchronised credential store, and authentication does not require sending a password over the network.
- Account recovery remains necessary, and it relies on the organisation’s recovery process rather than the ability to reconstruct a private key.
#### Password risks and mitigations
- Poor practices include reusing passwords, writing them down in unsecured locations, and storing them in unprotected files.
- Password managers can generate and store unique passwords and reduce reset costs, but they do not prevent phishing when users approve sign in on a lookalike site.
- Strong security relies on layered controls, including MFA, phishing resistant methods, and monitoring for abnormal authentication and access patterns.
### Physical threats and controls
#### Common physical threats
- Physical threats include unauthorised access, theft of devices and media, tailgating, dumpster diving, surveillance, and vandalism.
- Physical compromise can enable direct malware introduction, data theft, and loss of service.
#### Deterrence and access controls from outside to inside
- Outdoor measures include fencing, lighting, cameras, locks, alarms, signage, and controlled entry points.
- Reception and guard presence can deter intrusion and improve verification at access points.
- Sensitive areas such as server rooms use stricter controls, including restricted access, logging of entry, and surveillance.
#### Environmental and disaster resilience
- Temperature extremes, humidity, and electrostatic discharge can damage equipment and data.
- Controls include HVAC management, grounding and anti static protection, routine maintenance, and environmental monitoring.
- Disaster planning uses geographically separated redundancy, backup power such as UPS and generators, and secure off site storage for critical data.
#### Physical control examples
| Area | Controls and tools |
| --- | --- |
| On premises | smart locks, access control systems, security cameras, security personnel, fencing, barriers, mantrap doors |
| Server rooms | biometric access controls, rack locks, video surveillance, tamper sensors, fire suppression, environmental monitoring |
| Environmental safety | climate control, ESD protection, surge protection, backup power, water detection |
| Device protection | cable locks, port locks, screen filters, full disk encryption, asset tracking |
#### Advanced options for high risk sites
- Drone detection systems can detect unauthorised aerial surveillance attempts.
- Robotic patrol systems can extend monitoring coverage in large facilities.
- AI assisted security can detect anomalies across sensor feeds and alert staff.
- Intelligent perimeter systems can distinguish likely intrusions from benign movement and reduce false alarms.