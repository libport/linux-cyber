# Computer Networks and Network Security

> [!NOTE]
> This guide surveys core computer networking and security concepts. It explains OSI and TCP/IP models, common protocols, ports, sockets, and wireless IEEE standards, then covers network design and hardware roles. It introduces IPv4/IPv6 addressing, subnetting, routing tables, ARP, and packet/header fields, with practical troubleshooting using tools like Wireshark, traceroute, DNS, DHCP, syslog, and flow analysis. Security sections detail packet inspection, firewall types, IDS/IPS, NAT, and layered defenses, plus FIM, DLP, NAC, UBA, and EDR/XDR practices. 
## Networking Fundamentals
### Networking models
- A networking model defines how systems communicate, including how data is packaged, addressed, transmitted, and received.
- The Open Systems Interconnection, or OSI, model is a conceptual framework with seven layers.
- The Transmission Control Protocol and Internet Protocol, or TCP/IP, model is the practical protocol suite used on the internet. It groups OSI functions into fewer layers.
### OSI layers
- Application: user-facing services such as web and email clients
- Presentation: data format handling and encryption
- Session: session control, authentication, and reconnection
- Transport: end-to-end delivery and error handling, commonly TCP or UDP
- Network: logical addressing and routing, including IP
- Data link: local delivery framing and MAC addressing, with local error handling
- Physical: raw bit transmission over cable, fibre, or radio
### Standards and standards bodies
- Standards support interoperability across vendors and industries.
- De jure standards are formal standards published by recognised bodies.
- De facto standards are widely used through market adoption without a formal ratification process.
- Key organisations include ISO, ITU, DARPA, IEEE, W3C, and IETF. IETF publishes Requests for Comments, or RFCs.
### Protocols, ports, and sockets
- TCP prioritises reliability through acknowledgements and retransmission, with higher overhead.
- UDP prioritises speed and low latency, with no delivery guarantee.
- A port is a logical endpoint for an application service. Port numbers range from 0 to 65,535.
- A socket identifies a specific conversation using source IP, source port, protocol, destination IP, and destination port.
### Common ports
- HTTP 80, HTTPS 443
- FTP 21 for control, with a separate data channel in some modes
- SSH 22, also used by SFTP
- Telnet 23, not recommended due to lack of encryption
- RDP 3389
- POP3 110, IMAP4 143, SMTP 25
- DHCP 67 and 68
- DNS 53
- SMB 445, with older NetBIOS services often using 137 to 139
- SNMP 161, with traps commonly on 162
- LDAP 389
### Wireless network types and IEEE standards
- WPAN uses IEEE 802.15 for short-range personal networks such as Bluetooth and Zigbee.
- WLAN uses IEEE 802.11 for local networks using Wi-Fi.
- WMAN commonly uses IEEE 802.16, including WiMAX, for metro-scale wireless access.
- WWAN covers regional to global wireless connectivity, including cellular networks such as 4G and 5G and low power WAN options such as LoRaWAN.
- Ad hoc networks form opportunistically, including mobile and vehicle variants such as MANETs and VANETs.
- IEEE 802.20 is no longer actively developed. IEEE 802.22 uses TV white space spectrum to extend broadband to lower-density and rural areas.
### Designing a basic network structure
- Network design aligns infrastructure to business needs and supports performance, scalability, security, and cost control.
- Skipping design commonly leads to outages, poor performance, avoidable security gaps, and wasted spend.
- Core steps include requirements gathering, current state assessment, goal definition, technology selection, physical layout and implementation planning, and documentation.
- Common tools include IBM Netcool, IBM Cloud Pak for Network Automation, IBM SevOne, IBM Spectrum Control, Cisco Network Assistant, SolarWinds Network Topology Mapper, NetBrain, Microsoft Visio, GNS3, and Wireshark.
### Networking hardware overview
- Servers host shared services and data. Clients consume those services. Nodes are any connected devices.
- Hubs broadcast traffic to all ports. Switches forward frames to the intended MAC address and are more efficient.
- Routers forward packets between networks using routing tables. Modems convert signals for ISP access.
- Bridges connect network segments. Gateways translate between networks or protocols.
- Repeaters and extenders improve signal reach. Wireless access points connect Wi-Fi devices to wired networks.
- Security controls include firewalls, proxy servers, intrusion detection systems, and intrusion prevention systems.
## IP Addressing, Routing and Switching
### IPv4 address structure
IPv4 identifies an interface using a 32-bit number written as four decimal octets separated by dots, such as 192.0.2.126. Each octet represents 8 bits and ranges from 0 to 255, so IPv4 spans 0.0.0.0 to 255.255.255.255 and provides 2^32 addresses, about 4.29 billion.

An IPv4 address is interpreted as:
- Network portion: identifies the network that a host belongs to
- Host portion: identifies a specific host within that network
- Subnetting: splits a larger network into smaller networks by extending the network portion

Modern networks express the boundary using CIDR notation, for example /24, and often assign addresses automatically using DHCP, Dynamic Host Configuration Protocol.
### Classful networks and key corrections
Classful addressing is a legacy scheme that divided IPv4 into ranges with default subnet masks. Most real networks now use CIDR instead, but the classes are still useful for historical context.

Class ranges and intent:
- Class A: 0.0.0.0 to 127.255.255.255, default mask 255.0.0.0
- Class B: 128.0.0.0 to 191.255.255.255, default mask 255.255.0.0
- Class C: 192.0.0.0 to 223.255.255.255, default mask 255.255.255.0
- Class D: 224.0.0.0 to 239.255.255.255, multicast groups
- Class E: 240.0.0.0 to 255.255.255.255, experimental and reserved

Important reservations:
- 0.0.0.0 is used for special meanings, including a default route
- 127.0.0.0/8 is reserved for loopback, such as 127.0.0.1

Not all IPv4 addresses are routable on the public internet. Private ranges such as 10.0.0.0/8, 172.16.0.0/12, and 192.168.0.0/16 are intended for internal networks and typically rely on network address translation at the edge.
### IP protocol, packets, and routing
Internet Protocol is a Layer 3 protocol that carries traffic as packets. Routers forward packets by inspecting the destination IP address, while stateful firewalls commonly inspect both source and destination addresses as part of session tracking and policy enforcement.

Key routing concepts:
- Routable traffic can be forwarded beyond the originating network, subject to addressing policy and routing rules
- The default gateway is the local router that forwards packets to other networks when no more specific route exists
- The subnet mask or CIDR prefix tells a host which addresses are local and which must be sent to the gateway

IPv4 also uses a broadcast address per subnet. For a /24 network such as 192.168.52.0/24, the broadcast address is 192.168.52.255, which sets all host bits to 1.
### IP header essentials
The IP header includes fields that support forwarding and delivery. Commonly referenced fields are:
- Version: IPv4 or IPv6
- Time to live: decremented by one at each router hop, then dropped at zero to prevent endless loops
- Protocol number: identifies the next layer, for example ICMP 1, TCP 6, UDP 17
- Source address and destination address
- Payload: the encapsulated higher layer data

Tools such as Wireshark can capture packets and display Layer 2 and Layer 3 headers for troubleshooting.
### Layer 2 addressing and ARP
A network interface is identified at Layer 2 by a MAC address, typically 48 bits. The first 24 bits are the organisationally unique identifier that indicates the vendor, and the remaining bits identify the interface.

MAC addresses are factory assigned, but many operating systems can be configured to present a different MAC address, which is known as MAC spoofing.

Common ways to display MAC addressing include:
- Windows systems typically use ipconfig /all
- Linux systems commonly use ip link or ip addr
- macOS commonly uses ifconfig or networksetup

Address Resolution Protocol, ARP, maps an IPv4 address to a MAC address on the local broadcast domain. ARP does not resolve MAC addresses for remote networks. When a host sends traffic to an off subnet destination, it resolves and uses the MAC address of the default gateway, not the remote host.

Common ways to display neighbour caches include:
- Windows: arp -a
- Linux: ip neigh show
### Routing tables and route types
Routing decisions come from a routing table that exists on routers and also on endpoints. The table contains destination prefixes and the next hop interface or gateway.

Common route types:
- Connected routes: networks directly attached to an interface
- Default route: the route used when no more specific match exists
- Static routes: manually configured
- Dynamic routes: learned and updated via routing protocols such as OSPF, RIP, or EIGRP

Trace tools can reveal the hop by hop path:
- Windows: tracert
- Linux and macOS: traceroute
### IPv6 addressing and operation
IPv6 expands addressing to 128 bits and expresses addresses as eight groups of four hexadecimal digits, separated by colons. Each group is 16 bits.

IPv6 supports:
- Unicast: one to one
- Multicast: one to many
- Anycast: one to nearest of several interfaces sharing an address

IPv6 has no broadcast addressing. Multicast and anycast cover the common broadcast use cases.

Address representation rules:
- Leading zeros in a group can be omitted
- A single double colon can compress one contiguous run of all zero groups
- IPv4 mapped IPv6 addresses commonly use the form::ffff:192.0.2.128

IPv6 headers are generally simpler than IPv4 headers and rely on extension headers for optional features. IPv6 advantages include a vastly larger address space, improved autoconfiguration support, more efficient multicast, and better support for mobility and modern network design. IPsec is widely supported in IPv6 stacks, though its use still depends on configuration and policy.
### Subnetting and host calculations
IPv4 examples:
- 192.168.1.0/24 corresponds to subnet mask 255.255.255.0
- Splitting 10.0.0.0/24 into 4 subnets borrows 2 host bits, giving /26 and mask 255.255.255.192
- /26 subnets increment by 64 in the last octet and have 62 usable host addresses each

Resulting /26 subnets:
- 10.0.0.0/26, usable 10.0.0.1 to 10.0.0.62, broadcast 10.0.0.63
- 10.0.0.64/26, usable 10.0.0.65 to 10.0.0.126, broadcast 10.0.0.127
- 10.0.0.128/26, usable 10.0.0.129 to 10.0.0.190, broadcast 10.0.0.191
- 10.0.0.192/26, usable 10.0.0.193 to 10.0.0.254, broadcast 10.0.0.255

Usable host count for /26:
- 2^(32-26) - 2 equals 64 - 2 equals 62 usable hosts

IPv6 examples:
- Creating 16 subnets from 2001:db8:85a3::/48 borrows 4 bits, producing /52
- A /52 prefix mask is ffff:ffff:ffff:f000:: in shorthand
- The /52 subnets step by 0x1000 in the fourth group, for example 2001:db8:85a3:0000::/52, 2001:db8:85a3:1000::/52, 2001:db8:85a3:2000::/52

For a /64 IPv6 subnet, the interface identifier space contains 2^64 possible addresses, about 1.84 × 10^19. Some addresses are reserved for specific functions, but the space is still effectively enormous for typical network design.
### Binary, decimal, octal, and hexadecimal
IPv4 troubleshooting often requires converting between decimal and binary.

Key points:
- Decimal is base 10 and uses digits 0 to 9
- Binary is base 2 and uses digits 0 and 1, with place values 1, 2, 4, 8, 16, 32, 64, 128
- Octal is base 8 and uses digits 0 to 7
- Hexadecimal is base 16 and uses digits 0 to 9 and letters A to F for values 10 to 15

Corrected conversion examples:
- 11011010 in binary equals 128 + 64 + 16 + 8 + 2 equals 218 in decimal
- 235 in decimal equals 11101011 in binary

The number of values representable with n digits in base b is b^n, giving a range from 0 to b^n - 1.
## Network Protocols
### Transport protocols
- Transmission Control Protocol, TCP, is connection oriented and provides reliable, ordered delivery using acknowledgements, retransmission, and sequencing.
- User Datagram Protocol, UDP, is connectionless and provides best effort delivery with low overhead and low latency.
- Neither TCP nor UDP provides encryption. Confidentiality and integrity are provided by higher layer security such as TLS or VPN technologies.
### TCP operation and the three-way handshake
- A TCP session begins with a three-way handshake to confirm both endpoints can communicate and to establish initial sequence numbers.
- Step 1: the client sends a SYN segment to request a connection.
- Step 2: the server replies with SYN-ACK to acknowledge and synchronise.
- Step 3: the client replies with ACK to complete session establishment.
- After establishment, TCP delivers data as segments and tracks progress with sequence numbers and acknowledgement numbers.
- TCP can transmit multiple segments before receiving acknowledgements. The receiver uses sequence numbers to detect gaps and the sender retransmits missing segments.
### TCP header essentials
- Source port and destination port identify the sending and receiving application endpoints.
- Sequence number identifies the position of a segment in the byte stream.
- Acknowledgement number confirms receipt and indicates the next expected byte.
- Flags indicate control state, including SYN, ACK, FIN, and RST.
- Window size supports flow control by signalling how much data can be received without further acknowledgement.
### UDP characteristics and common services
- UDP is often selected when timeliness matters more than perfect delivery, such as voice, video, and some real-time telemetry.
- Applications that need reliability on top of UDP typically add their own controls, or use a higher layer protocol such as QUIC.
- Common UDP based services include DNS queries on port 53, DHCP on ports 67 and 68, SNMP on ports 161 and 162, and TFTP on port 69.
- DNS can also use TCP for cases such as zone transfers and responses that do not fit within a single UDP message.
### Common TCP applications
- HTTP and HTTPS for web traffic, with HTTPS using TLS to encrypt application data.
- SMTP for sending email and POP3 or IMAP for retrieving email.
- FTP for file transfer, with SFTP running over SSH rather than FTP.
- SSH for secure remote administration.
### Wireshark lab workflow
- Wireshark is installed with a capture driver, commonly Npcap on Windows.
- Wireshark is started with administrative rights where required so the capture driver can access network interfaces.
- The active interface is selected, capture is started, and traffic is generated using a browser or command line tools.
- Display filters commonly include tcp, udp, tcp port 443, and udp port 53.
- A TCP handshake is identified by locating SYN, SYN-ACK, and ACK, then reviewing header fields for ports, sequence numbers, acknowledgements, and flags.
- UDP traffic is captured with DNS queries, then the UDP header is reviewed for source port, destination port, length, and checksum.
- TCP and UDP captures are compared to confirm that TCP includes session and reliability fields that are absent from UDP.
### DNS and DHCP
- Domain Name System, DNS, translates domain names into IP addresses so clients can locate services.
- Dynamic Host Configuration Protocol, DHCP, automatically leases IP configuration to endpoints, including address, subnet mask, default gateway, and DNS server information.
- DHCP uses a four message exchange: Discover, Offer, Request, Acknowledgement.
- DHCP messages are often broadcast at Layer 2 and Layer 3 when the client does not yet know local addressing details.
### DNS filtering
- DNS filtering enforces policy at the name resolution layer by allowing, blocking, or redirecting DNS queries based on rules.
- Security filtering blocks domains associated with phishing, malware distribution, and command and control infrastructure.
- Content filtering blocks categories such as social media or gambling to support acceptable use and compliance requirements.
- DNS filtering can reduce risk and bandwidth use, and many services provide logs for auditing and incident response.
- Implementations may be network level using a recursive resolver, firewall, appliance, or cloud service, and device level using endpoint agents.
- Policies require ongoing review so controls keep pace with new threats and organisational changes.
### Syslog logging
- Syslog provides a structured way to generate, transport, store, and analyse event messages across networked systems.
- The content layer defines message content. The application layer handles generation, routing, and storage. The transport layer carries messages across the network.
- Common roles include originator, relay, collector, transport sender, and transport receiver.
- Messages include a facility code to indicate the source subsystem and a severity level from 0, emergency, to 7, debug.
- Local facilities are often used by network devices, while operating systems may map facilities to system services and daemons.
### Network flows and flow analysis
- A network flow is a set of packets that share common properties such as source and destination IPs, ports, protocol, and timing.
- Flow exporters on routers and switches can generate flow records using formats such as NetFlow, sFlow, or IPFIX.
- Flow records commonly include packet and byte counts, start and end times, input and output interfaces, protocol, and QoS markings.
- Flow analysis supports traffic profiling, capacity planning, and threat detection, including DDoS patterns and large outbound transfers consistent with exfiltration.
### Port mirroring and promiscuous mode
- Port mirroring copies traffic from one or more switch ports or VLANs to a monitoring port, often called SPAN on Cisco platforms.
- Remote mirroring, often called RSPAN, can send mirrored traffic to a port on another switch.
- Intrusion detection systems can analyse mirrored traffic without sitting inline, so monitoring does not block or delay production traffic.
- A capture host uses promiscuous mode to accept frames not addressed to its MAC address.
- Port mirroring requires strong access controls because it can expose sensitive traffic if misused.
### User behaviour analysis
- User behaviour analysis, UBA, monitors activity to establish baselines and detect anomalies that may indicate insider threats or compromised accounts.
- Core concepts include baselining normal behaviour, anomaly detection, and contextual awareness such as role, location, and time of access.
- Data sources include authentication logs, application logs, endpoint telemetry, and network flow records.
- Techniques include statistical methods, rule based logic, and machine learning models that adapt over time.
- Common use cases include detection of unusual logins, privilege misuse, atypical access to sensitive data, and data exfiltration patterns.
- Key challenges include false positives, privacy obligations, and scalability for high volume logs and flows.
### Integrating flow analysis and UBA
- Correlating flow anomalies with user context improves confidence and triage quality.
- Example: an abnormal outbound transfer combined with an unusual login location strongly suggests account compromise.
- Effective programmes place collection points at critical network segments, update baselines routinely, secure telemetry in transit and at rest, and restrict access using role based access controls.
### Endpoint protection with Xcitium OpenEDR
- Xcitium OpenEDR is an endpoint detection and response platform that uses a client server design. A central cloud manager collects telemetry and an agent runs on each endpoint.
- Initial setup includes creating an account, enabling multi-factor authentication using an authenticator app, and enrolling devices from the Device Management area.
- Enrolment typically provides an installation link for an endpoint agent, followed by a device restart so the agent can register and report.
- The console can be used to review endpoint inventory and performance data, examine audit logs, and triage alerts by severity.
- Alert policies can be customised to match organisational requirements, then removed or reverted if they create unnecessary noise.
- Patch management supports operating system and third party updates, including on demand rechecks and remote install or uninstall actions.
- On demand scans, including quick scans, support malware detection and quarantine review. Results are surfaced in the cloud manager for investigation and follow up.
## Network Security Techniques
### Packet inspection and firewalls
- Packet inspection evaluates network traffic against security policy.
- Stateless inspection checks each packet in isolation, typically using source and destination addresses, ports, and protocol. It does not maintain a session table, so it cannot confirm whether a packet belongs to an existing connection.
- Stateful inspection, also called dynamic packet filtering, tracks active sessions and evaluates each packet with context from earlier packets in the same session. Session tracking is commonly based on source IP, destination IP, source port, destination port, and protocol.
- Stateful firewalls maintain session records that include policy, interfaces, counters, and timeouts so sessions do not persist indefinitely. Return traffic that matches an approved session can be allowed with less processing than new flows.
- Many platforms apply basic packet screening before stateful processing. If a packet does not match an existing session, the firewall typically applies protection screens, network address translation rules if configured, routing decisions, zone mapping, and then policy evaluation. Additional services can be applied depending on platform and licensing, such as application tracking, denial of service controls, quality of service, application layer controls, or intrusion prevention.
### Firewall filter types
- Stateless firewalls filter based on packet headers without connection context.
- Stateful firewalls filter using both packet headers and connection state.
- Application level firewalls inspect application data to enforce more granular controls, such as HTTP method restrictions or application specific policy.
### Intrusion detection and intrusion prevention
- An intrusion detection system, IDS, monitors traffic and system activity and raises alerts when patterns indicate misuse, anomalies, or policy violations. IDS is typically passive and depends on administrators to respond.
- Network based IDS monitors traffic on network segments, while host based IDS monitors logs, file integrity, and processes on an individual host.
- An intrusion prevention system, IPS, detects threats and takes action automatically, such as dropping packets, blocking sources, or resetting connections. IPS is commonly deployed in line, so it can affect latency and throughput if not sized and tuned correctly.
- Network based IPS protects network flows near the perimeter or internal chokepoints, while host based IPS protects a specific endpoint.
- IDS and IPS can generate false positives. IDS can cause alert fatigue, while IPS can block legitimate traffic if policies are too aggressive. Both require ongoing tuning.
### Network address translation
- Network address translation, NAT, rewrites addressing information in IP packet headers as traffic crosses a boundary device such as a router or firewall.
- NAT supports IPv4 address conservation by allowing multiple internal hosts to share fewer public addresses.
- NAT hides internal addressing from external networks, which can reduce exposure but does not replace proper security controls.
- Static NAT maps one private address to one public address and is used for services that must be reachable from the internet.
- Dynamic NAT maps private addresses to a pool of public addresses, assigning an address per session or period of use.
- Port address translation, PAT, also called NAT overload, maps many private addresses to one public address by using distinct source ports per session.
- Common NAT issues include incorrect port forwarding, address range overlap between interconnected private networks, added processing overhead under heavy load, and complications for protocols that embed addressing information in payloads, such as SIP used for voice over IP.
### Limitations of firewalls and IDS
- Firewalls primarily control traffic crossing a boundary, so they provide limited protection against insider threats and activity that originates within the trusted network.
- Traditional firewall rules do not reliably detect advanced persistent threats that blend with normal traffic and use valid credentials.
- Encrypted traffic reduces visibility. Decryption features can restore inspection capability but add complexity, performance cost, and privacy considerations.
- Misconfiguration can introduce openings or block needed services, so change control and validation are critical.
- Firewalls are weak against zero day exploits when policy is based on known signatures or simple port based rules.
- IDS limitations include high false positive rates, limited ability to stop attacks without integration to other controls, evasion techniques such as fragmentation and obfuscation, reduced visibility into encrypted traffic, and resource pressure in high traffic environments.
- Effective network defence typically combines multiple layers such as firewalls, IDS and IPS, web application firewalls, endpoint controls, threat intelligence, and centralised logging and correlation through a SIEM.
### File integrity monitoring
- File integrity monitoring, FIM, detects unauthorised changes by comparing current files to a trusted baseline.
- Baselines capture the expected state of critical system, configuration, application, and data files.
- FIM commonly uses cryptographic hashes as file identifiers. Modern deployments prefer SHA-256 or stronger algorithms rather than MD5 or SHA-1.
- Monitoring can run continuously or on a schedule, with alerting and reporting to support investigation and compliance.
- Responding to integrity violations includes verifying whether a change is authorised, investigating suspicious changes, restoring known good files from backups when needed, and fixing the underlying cause such as vulnerable services or excessive permissions.
### Data loss prevention
- Data loss prevention, DLP, reduces the risk of sensitive data leaving approved boundaries across data in use, data in transit, and data at rest.
- DLP typically includes data classification, policy enforcement, content inspection, contextual analysis of users and devices, and protective controls such as encryption or tokenisation.
- Implementation involves identifying sensitive data and risk scenarios, deploying monitoring and enforcement points, defining policies aligned to regulatory requirements, and training staff to reduce accidental leakage.
- Incident handling includes reviewing alerts and logs, containing affected systems or accounts, investigating root cause, and updating policies and training based on lessons learned.
### Network access control
- Network access control, NAC, restricts access based on identity, role, and device security posture.
- Core functions include authentication, authorisation through role based access control, and compliance checks such as patch level and endpoint protection status.
- Non compliant devices can be quarantined for remediation so they do not endanger the main network.
- NAC strengthens defence in depth when integrated with firewalls, IDS and IPS, and endpoint protection, and it supports compliance by enforcing consistent access policy.
### Endpoint detection and response and XDR
- Endpoint detection and response, EDR, monitors endpoints such as desktops, laptops, servers, and mobile devices to detect, investigate, and respond to suspicious activity.
- EDR agents collect logs and telemetry about processes, files, network connections, and user actions, then correlate events to detect threats using signature, heuristic, and behavioural techniques.
- Investigation features include timeline reconstruction and root cause analysis. Response actions include isolating a host, terminating malicious processes, and applying patches or configuration changes.
- Extended detection and response, XDR, expands EDR by combining data from multiple security sources such as network telemetry, cloud controls, and email security, then correlating alerts in a single platform.
- XDR can improve detection of multi vector attacks and supports faster response through automated containment and richer context. It also supports compliance through consistent logging, reporting, and audit trails.