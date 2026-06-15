# Chapter 9: Critical Infrastructure and Industrial Intelligence
Critical infrastructure keeps communities supplied, connected, powered, governed, transported and protected. It includes energy, water, transport, communications, food, health, finance, emergency services, government facilities, dams, nuclear materials, information technology, commercial facilities, critical manufacturing, the defence industrial base and chemical facilities. A serious failure in any of these sectors can damage public safety, national security and economic stability.

Threats to these systems come from natural hazards, ageing equipment, poor maintenance, exposed services, unpatched software, default credentials, third-party dependencies, insider risk, supply-chain concentration, physical sabotage and cyber intrusion. A resilient assessment treats digital and physical exposure as connected. A gunfire attack against substations can cause disruption as effectively as malware when attackers understand which assets matter most.

Industrial intelligence uses lawful open-source collection to identify exposure before an adversary exploits it. Analysts assess what public information reveals about industrial sites, control systems, suppliers, employees, contracts, wireless networks and Internet-facing devices. The aim is defensive: reduce attack surface, protect proprietary information and help asset owners understand how their operations appear from outside the organisation.

Reconnaissance sits at the start of many intrusion models. The ICS Cyber Kill Chain places reconnaissance in the planning phase because adversaries need to understand targets before they can deliver malware, steal data or disrupt physical processes. Defensive analysts therefore study the same public clues that attackers seek, then convert those clues into risk findings and practical mitigations.

Stuxnet showed how cyber operations can damage industrial processes. The malware, discovered in 2010, targeted Siemens control systems associated with centrifuges at Iran's Natanz uranium enrichment facility. It spread through infected media and trusted relationships, then manipulated control logic to degrade centrifuge operation. The case remains important because it joined cyber intrusion, industrial process knowledge and supply-chain access in one operation.

Critical infrastructure also faces wartime and geopolitical risk. In Ukraine, the major nuclear safety incident in March 2022 involved a projectile hitting a training building at the Zaporizhzhia nuclear power plant, not a reactor fire at Chernobyl. The event did not damage reactor safety systems, but it showed how military action around nuclear facilities can create global safety concerns. Attacks on cell towers and communications networks can also isolate populations during conflict.
## Operational technology and industrial systems
Operational technology covers programmable systems and devices that monitor or control the physical environment. It includes industrial control systems, building automation, transportation systems, access control, measurement systems and environmental monitoring.

Key industrial terms include:
- Industrial control systems, which control industrial processes and include SCADA, DCS and PLC environments.
- Distributed control systems, which coordinate complex plant processes across distributed controllers.
- Supervisory control and data acquisition systems, which supervise equipment, collect data and manage remote operations.
- Programmable logic controllers, which control machinery, assembly lines, pumps, valves, robots and other process equipment.
- Remote terminal units, which connect field equipment to control and monitoring systems.

The convergence of OT with information technology and Internet-connected devices increases efficiency, but it also creates pathways between business systems and physical processes. Poor segmentation, weak authentication and exposed management interfaces can let an attacker move from a seemingly minor device into a more important environment.

The Internet of Things connects sensors, appliances, vehicles, cameras, meters and industrial devices to networks. Industrial IoT applies the same idea to production, logistics, utilities, transport and smart infrastructure. Smart sensors can monitor temperature, water levels, train movement, grid performance, port operations and maintenance conditions. These devices can improve safety and reliability, but insecure deployment can expose industrial environments to espionage, ransomware, disruption or destructive action.

PIPEDREAM, also known through Dragos reporting on CHERNOVITE, illustrates the risk from ICS-focused malware. The framework can target industrial software and programmable controllers, which gives defenders a strong reason to maintain asset inventories, test incident response plans and monitor for abnormal controller activity.

The Colonial Pipeline ransomware incident showed that business-system compromise can still disrupt critical services. DarkSide actors attacked Colonial Pipeline in 2021, and the company shut down pipeline operations while responding. The incident reinforced basic defensive priorities: multifactor authentication for remote access, monitoring, backup resilience, network segmentation, incident response planning and exposure management.
## Analytical method
Effective analysis starts with clear requirements. Stakeholders may ask about a city, a company, a sector, a technology, a supplier or an adversary group. The analyst then narrows the problem through a funnel approach.

- Baseline the environment with historical information, known providers, technologies, geography and normal operating patterns.
- Start wide by mapping major assets, owners, suppliers, access points and dependencies.
- Narrow the focus to selectors such as domains, IP ranges, employee roles, physical access points, wireless identifiers, exposed services and specific technologies.
- Layer further checks for leaked credentials, known vulnerabilities, public contracts, job advertisements, social media disclosures, satellite imagery and Internet-facing infrastructure.

A useful critical infrastructure assessment links assets to consequences. It asks which company powers a region, which supplier provides a unique component, which plant supports defence production, which communications provider covers emergency operations and which public disclosures reveal sensitive technology. A single supplier, component or facility can become a national security concern when no alternative exists.
## Visualisation and public disclosure
Visualisation helps analysts understand geography, clustering, routes and dependencies. GIS tools, Google Earth Pro and mapping platforms can plot coordinates, service territories, pipelines, substations, plants, ports, towers and device locations. Analysts can import coordinates from spreadsheets, convert them to KML or KMZ files and compare patterns across time.

Premade maps from governments, infrastructure providers, research groups and watchdogs can accelerate baselining. Examples include energy atlases, infrastructure datasets, nuclear operator maps, defence industry maps and provider service maps. Analysts should verify important findings independently because public maps may be incomplete, outdated, simplified or influenced by the publisher's purpose.

Public disclosures often reveal operational detail. Contracts can identify suppliers, service arrangements, technology versions and maintenance relationships. Job advertisements can disclose platforms, tools, certifications and plant functions. Company reports can show structure, risk concerns, key dependencies and facility locations. Social media and professional profiles can reveal equipment, control rooms, panels, badges, processes and staff responsibilities. Analysts should handle this material carefully and report exposure without amplifying sensitive details unnecessarily.

Internet infrastructure search tools can help defenders find exposed systems when used lawfully and with authorisation. Shodan, Censys and similar tools can identify Internet-facing services, banners, certificates and device metadata. Findings should feed into validation, asset ownership checks and remediation, not unauthorised interaction.

Analysts must separate observation from interaction. Passive review of public disclosures and historical databases can support defensive work, while probing, logging in, exploiting, credential testing or active collection against third-party systems can create legal and safety risk. Industrial environments also carry higher consequences than ordinary enterprise networks because a mistaken action can affect equipment, production, utilities or public safety. Reports should therefore prioritise evidence, confidence levels, business impact and safe remediation steps.
## Wireless and location intelligence
Wireless networks create identifiers that can support infrastructure analysis. Wi-Fi uses service set identifiers, basic service set identifiers and MAC addresses. SSIDs name networks, BSSIDs usually identify access points and MAC prefixes can identify vendors. These clues can suggest a device type, organisation, technology family or possible vulnerability class, although they rarely prove ownership by themselves.

Mobile networks use cell towers rather than Wi-Fi routers. Cellular IP geolocation can be unreliable because mobile carriers reallocate addresses, share addresses across many users and keep devices attached while users move between cells. Strong analysis treats cellular IP location as a weak lead unless corroborated by lawful tower records, device data or other evidence.

Bluetooth, fitness trackers, printers, vehicles and other short-range devices can also broadcast useful identifiers. Names such as a company printer or a personal device can reveal location clues, but analysts must avoid overclaiming. A visible signal shows collection at or near a place and time, not necessarily that the owner was present.

Low-power wide-area networks, including LoRaWAN, support smart city and industrial sensors over long range with low power use. They often support water sensors, environmental monitoring, metering and industrial telemetry. Analysts should understand the technology because insecure keys, exposed gateways or public imagery of deployments can reveal risk in critical sensor networks.

Wardriving collects wireless signals while moving through an area and can populate public databases such as WiGLE. Defensive analysts may use public historical data, but they should treat it as incomplete and approximate. A missing signal does not prove absence, and a plotted point does not prove exact device location. Buildings, terrain, antenna strength, collection bias and upload history can all distort results.

Cell tower databases such as OpenCellID and CellMapper can support regional resilience analysis. They can help identify provider coverage, tower density and possible communications dependencies. They should not replace authoritative carrier or emergency-management data, but they can guide further questions.
## Defensive priorities
Strong industrial intelligence produces clear, actionable findings. It identifies exposed assets, leaked credentials, sensitive disclosures, public-facing services, insecure wireless identifiers, supplier concentration, weak segmentation and critical single points of failure. It also explains likely consequences in plain language.

Physical context matters throughout this work. A publicly visible gate, access road, badge reader, radio mast or backup generator can explain how a digital dependency connects to a real-world consequence. The strongest assessments combine maps, records, technical indicators and process knowledge, then distinguish exposed information from exploitable risk.

Asset owners should maintain accurate OT and IoT inventories, segment business and control networks, enforce multifactor authentication, remove default credentials, patch or isolate vulnerable systems, control remote access, review public disclosures, train staff on operational security and rehearse incident response for physical and cyber scenarios. Analysts should report risk in a way that helps stakeholders fix exposure without publishing a road map for attackers.