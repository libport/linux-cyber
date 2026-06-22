# Chapter 4: Operational Security
Operational security (OPSEC) protects critical information by identifying what an adversary can observe, how those observations could reveal intent or capability, and which countermeasures can reduce that risk. For open-source intelligence (OSINT), good OPSEC protects the analyst, the investigation, the organisation, and the integrity of the case.

OPSEC starts with threat modelling. Analysts define the investigation's scope, identify likely adversaries, identify people and systems that could be targeted, select protective measures, and test whether those measures address the risk. This process should remain active throughout the investigation because adversary behaviour, collection opportunities, and operational requirements can change.
## Threat modelling methods
Persona-based modelling helps analysts view the investigation from an adversary's perspective. A persona should describe a plausible adversary's motives, goals, resources, skill level, likely methods, and tolerance for risk. Security cards use a similar approach by prompting analysts to consider threat categories such as human impact, motivation, resources, and methods. Attack trees map a specific adversary goal into smaller steps, such as identifying an analyst's location through geolocation, social media disclosures, website photos, canary tokens, IP addresses, or metadata.

These methods help analysts move beyond generic risk statements. A useful model explains who may act, what information they need, how they may collect it, and how the investigation can make collection harder.
## The OPSEC process
Effective OPSEC follows five linked steps:
1. Define critical information.
2. Analyse the threat.
3. Determine vulnerabilities.
4. Assess risk.
5. Apply countermeasures.

Critical information includes any detail that could help an adversary identify the analyst, compromise the investigation, anticipate collection activity, or disrupt operations. Threat analysis identifies adversary motives, goals, capabilities, likely methods, and existing knowledge. Vulnerability analysis asks whether the adversary can observe analyst activity, identify collection points, or combine small indicators into a useful picture. Risk assessment weighs the likelihood and impact of exposure against the cost of protection. Countermeasures then reduce, disguise, or remove the exposed indicators.

Cost should match risk. A team investigating a state-sponsored advanced persistent threat may justify research-only laptops, dedicated phones, virtual private network (VPN) accounts, and virtual private servers. The same equipment may be excessive for a low-risk fraud inquiry. OPSEC is not a contest to use every privacy tool. It is a disciplined choice of controls that fit the threat, the mission, and the budget.
## OPSEC failures
OPSEC failures often come from small, repeated disclosures. The Silk Road investigation linked online aliases, forum posts, email addresses, code, and timing to Ross Ulbricht. Operation Purple Dragon showed that predictable operational patterns and notices to airmen could disclose strike timing and locations. The 2021 discovery of UK Ministry of Defence documents at a bus stop showed how poor document handling can expose sensitive planning. Leaked Conti ransomware chats also showed that attackers often exploit simple password patterns rather than advanced techniques.

The common lesson is that adversaries can use low-cost collection. They may not need exploits, malware, or privileged access if public data, metadata, account behaviour, leaked credentials, or routine procedures reveal enough.
## Practical countermeasures
Analysts should minimise identity exposure before an investigation begins. They should reduce unnecessary public personal information, keep devices and browsers patched, use password managers, enable multifactor authentication, lock idle screens, block unnecessary tracking, review browser and IP fingerprinting risk, and avoid using personal accounts or personal devices for investigative activity.

Countermeasures need regular review. A control that works for one case may fail in another. Investigating a corporate fraud matter does not require the same exposure controls as investigating a violent extremist network, a capable cybercrime group, or a state-linked target.
## OPSEC technology
VPNs encrypt traffic between the device and the VPN provider and make visited sites see the VPN server's IP address rather than the user's direct public IP address. They improve privacy on untrusted networks and can help analysts view region-specific content. They do not create full anonymity. Apps, logged-in accounts, cookies, browser fingerprints, payment records, DNS leaks, WebRTC leaks, or provider logs can still expose activity. A VPN can also signal that the user is attempting to hide their ordinary IP address. Analysts should assess provider reputation, jurisdiction, payment methods, audit history, logging claims, server locations, simultaneous connection limits, data caps, and dedicated IP options.

Tor routes traffic through multiple relays so that local observers do not see destination sites and destination sites see a Tor exit rather than the user's direct IP address. Tor can support censorship resistance and privacy, but it is slower than ordinary browsing and does not protect users who identify themselves through logins, downloads, unsafe documents, or repeated behaviour. Tor Browser includes fingerprinting protections, but analysts should still avoid unnecessary plugins, unsafe file handling, and personal account use.

Hyphanet, originally Freenet, provides peer-to-peer publishing and communication with privacy protections. It supports decentralised content sharing and friend-to-friend networks. I2P provides a private network layer that uses tunnels and garlic routing concepts, including layered encryption and message bundling. These tools serve different purposes from ordinary web browsing and require careful configuration and training.

Virtual machines (VMs) give analysts disposable or separated workspaces. They can isolate research environments, separate personas, and reduce contamination between cases. They are not a complete safety boundary. Browser fingerprinting, device identifiers, webcam compromise, malware escape, poor file handling, and analyst behaviour can still expose the user. Mobile emulators can support testing or access to mobile-only services, but analysts should use reputable software and avoid linking emulator activity to personal accounts or numbers.
## Research accounts
Research accounts can separate authorised investigative work from personal identity, but their use must comply with law, organisational policy, client authority, and platform rules. Analysts should not impersonate real people, misuse another person's image, bypass safeguards, or collect more information than the investigation permits.

A credible and ethical research account requires a defined purpose, approval, access controls, a persona record, and an exit plan if the account is exposed. Analysts should avoid personal contact details, personal devices, reused usernames, and behavioural patterns that connect the account to them. Account maintenance should match the approved purpose and should not become deceptive engagement beyond the scope of authority.

Strong OPSEC depends less on one tool than on disciplined habits. Analysts should know the adversary, define what must stay hidden, understand how small indicators combine, and choose proportionate controls before collection begins.