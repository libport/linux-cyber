# Computer Networks and Network Security
> [!NOTE]
> Computer networks connect devices and services through agreed protocols, addressing, routing, and physical media. Network security combines architecture, monitoring, access control, and endpoint protection to reduce the likelihood and impact of compromise.
## Networking fundamentals
### Networking models
A networking model groups communication functions into layers. Each layer provides services to the layer above it and uses services from the layer below it. Layering helps engineers design interoperable systems, isolate faults, and compare protocols.

The Open Systems Interconnection (OSI) model provides a seven-layer conceptual framework. The TCP/IP protocol suite underpins the internet and usually uses four broad layers: application, transport, internet, and link. Real implementations do not always map cleanly between the two models.

| OSI layer | Primary function | Examples |
| --- | --- | --- |
| 7. Application | Provides network services to application processes | HTTP, SMTP, and DNS |
| 6. Presentation | Represents, translates, compresses, and encrypts data | Character encoding and data formats |
| 5. Session | Establishes, manages, synchronises, and closes dialogues | Session checkpoints and remote procedure calls |
| 4. Transport | Supports process-to-process delivery and, depending on the protocol, segmentation, flow control, and reliable delivery | TCP and UDP |
| 3. Network | Provides logical addressing and routing between networks | IPv4, IPv6, and ICMP |
| 2. Data link | Frames data and controls delivery across a local link | Ethernet and IEEE 802.11 MAC functions |
| 1. Physical | Transmits signals and bits through a medium | Copper, fibre, and radio |
### Standards and standards bodies
Standards allow equipment and software from different suppliers to interoperate. Formal, or de jure, standards follow an established approval process. De facto standards gain broad adoption without formal ratification.

ISO and ITU-T publish international standards and recommendations. IEEE develops standards for technologies such as Ethernet and wireless local area networks. The IETF develops internet protocols through an open process and publishes technical documents in the Request for Comments (RFC) series. Not every RFC defines an internet standard. W3C develops web standards. DARPA funded research that contributed to ARPANET and TCP/IP, but it is not a general networking standards body.
### Protocols, ports, and sockets
TCP establishes connections and provides reliable, ordered byte-stream delivery through sequencing, acknowledgements, retransmission, flow control, and congestion control. UDP sends independent datagrams without connection establishment, delivery guarantees, ordering, retransmission, flow control, or congestion control. Its smaller header and limited transport functions reduce overhead.

A transport port is a 16-bit endpoint identifier within a transport protocol, with values from 0 to 65,535. IANA divides the range into system ports from 0 to 1,023, user ports from 1,024 to 49,151, and dynamic or private ports from 49,152 to 65,535.

Socket terminology varies by operating system and programming interface. A local transport endpoint normally combines a protocol, local IP address, and local port. Flow tools commonly identify one direction of TCP or UDP traffic with a five-tuple: source IP address, source port, destination IP address, destination port, and transport protocol. Reverse traffic exchanges the source and destination fields unless the tool explicitly combines both directions.
### Common service ports
Port registrations describe defaults. They do not prove that traffic contains the registered application or is safe.

| Service | Transport and port | Operational note |
| --- | --- | --- |
| HTTP | TCP 80 | Carries unencrypted web traffic by default |
| HTTPS | TCP 443 and UDP 443 | HTTP/1.1 and HTTP/2 commonly use TLS over TCP. HTTP/3 uses QUIC over UDP |
| FTP | TCP 21 for control | Active mode commonly uses TCP 20 for server data. Passive mode negotiates another data port |
| SSH and SFTP | TCP 22 | SFTP runs over SSH and is not FTP over TLS |
| Telnet | TCP 23 | Sends data without transport encryption and is unsuitable for untrusted networks |
| SMTP | TCP 25, 587, and 465 | Port 25 supports server relay, 587 supports message submission, and 465 supports submission over implicit TLS |
| POP3 | TCP 110 and 995 | Port 995 wraps POP3 in TLS |
| IMAP | TCP 143 and 993 | Port 993 wraps IMAP in TLS |
| DHCPv4 | UDP 67 and 68 | Servers use 67 and clients use 68 |
| DNS | UDP and TCP 53 | Most ordinary queries use UDP. TCP supports larger responses, retries after truncation, and zone transfers |
| SMB | TCP 445 | Legacy NetBIOS commonly uses UDP 137, UDP 138, and TCP 139 |
| RDP | TCP and UDP 3389 | Microsoft Remote Desktop can use both transports |
| SNMP | UDP 161 and 162 | Managers commonly query agents on 161. Traps and informs commonly use 162 |
| LDAP | TCP 389 and 636 | Port 636 commonly carries LDAP over TLS |
### Wireless network types and standards
| Network type | Typical scope | Common technologies |
| --- | --- | --- |
| Wireless personal area network (WPAN) | A person, room, or short-range device cluster | Bluetooth standards from the Bluetooth SIG and IEEE 802.15.4 technologies such as Zigbee and Thread |
| Wireless local area network (WLAN) | A home, office, campus area, or venue | Wi-Fi based on IEEE 802.11 |
| Wireless metropolitan area network (WMAN) | A town or metropolitan area | IEEE 802.16 technologies, including WiMAX |
| Wireless wide area network (WWAN) | Regional, national, or international coverage | Cellular systems such as 4G and 5G |
| Low-power wide area network (LPWAN) | Long-range links for low-data-rate devices | LoRaWAN and other low-power technologies |
| Wireless regional area network (WRAN) | Regional broadband using available television spectrum | IEEE 802.22 |

Ad hoc networks allow nodes to communicate without fixed infrastructure. Mobile ad hoc networks (MANETs) change topology as nodes move, while vehicular ad hoc networks (VANETs) apply similar principles to vehicles and roadside systems.
### Basic network design
Effective design aligns technical choices with business services, performance targets, growth, resilience, security, operational capacity, and cost.

A sound design process includes:
- documenting business, application, user, compliance, and availability requirements
- assessing the current topology, traffic, addressing, dependencies, and failure points
- defining logical segmentation, IP addressing, routing, naming, and access policy
- planning physical and wireless coverage, capacity, redundancy, and power
- selecting technologies that meet support, interoperability, security, and lifecycle requirements
- testing assumptions through modelling, simulation, prototypes, or pilot deployments
- preparing implementation, validation, rollback, monitoring, and incident response procedures
- maintaining diagrams, configurations, inventories, and decision records

Network teams use diagramming, simulation, packet capture, topology discovery, configuration management, automation, and performance monitoring tools. Tool selection depends on scale, platform support, integration requirements, staff capability, and budget.
### Networking hardware
- Servers provide shared applications, data, identity, and infrastructure services. Clients consume services, and nodes include any addressable devices connected to the network.
- Hubs repeat incoming signals to every other port. Ethernet switches learn source MAC addresses, forward known unicast frames towards the appropriate port, and flood broadcasts and unknown unicasts within the relevant VLAN.
- Routers forward packets between IP networks by consulting routing tables. Broadband modems adapt signals for a carrier medium, while optical network terminals terminate fibre access services.
- Bridges join data-link segments. A switch is a multiport bridge. Gateways connect systems that use different protocols, data formats, or application conventions.
- Repeaters regenerate signals, and wireless access points bridge wireless clients to a distribution network.
- Firewalls, proxies, intrusion detection systems, and intrusion prevention systems enforce or monitor security policy at different points in the architecture.
## IP addressing, routing, and switching
### IPv4 address structure
IPv4 assigns a 32-bit address to an interface. Dotted-decimal notation writes the address as four 8-bit octets from 0 to 255, such as `192.0.2.126`. The full space contains `2^32`, or 4,294,967,296, values, although protocol reservations and operational policy prevent hosts from using every value as an ordinary unicast address.

An IPv4 prefix separates network bits from host bits. Classless Inter-Domain Routing (CIDR) notation records the prefix length, such as `/24`. Subnetting extends the network portion to divide an address block into smaller prefixes. DHCP can lease IPv4 addresses and supply other configuration to hosts, but networks can also use static configuration or other provisioning systems.
### Classful addressing and special ranges
Classful addressing historically assigned fixed prefix lengths according to the leading bits of an IPv4 address. CIDR replaced classful routing and allocation, but the old classes still appear in legacy material.

| Historical class | First-octet range | Default prefix or purpose |
| --- | --- | --- |
| A | 0 to 127 | `/8`, subject to reserved ranges |
| B | 128 to 191 | `/16` |
| C | 192 to 223 | `/24` |
| D | 224 to 239 | Multicast |
| E | 240 to 255 | Reserved and special use |

Several ranges have special purposes:
- `0.0.0.0` represents the unspecified address in relevant contexts, while `0.0.0.0/0` represents the IPv4 default route.
- `127.0.0.0/8` supports loopback traffic.
- `169.254.0.0/16` supports IPv4 link-local addressing.
- `10.0.0.0/8`, `172.16.0.0/12`, and `192.168.0.0/16` support private internets and should not appear as public internet routes.

Networks often combine private addressing with network address translation at an internet boundary, but private addressing does not require NAT inside an organisation.
### IP packets and routing
IPv4 and IPv6 provide connectionless, best-effort packet delivery at the network layer. Routers inspect a destination address, select the most specific matching prefix in the routing table, and forward the packet towards the next hop. A stateful firewall can also evaluate source and destination addresses, transport ports, protocol, connection state, interfaces, zones, and security policy.

A host uses its routing and on-link information to determine whether a destination is directly reachable. In IPv4, this commonly follows the assigned address and subnet mask. In IPv6, address assignment does not by itself make the corresponding prefix on-link. The host sends off-link traffic to a suitable router, often its default gateway. A route with a longer matching prefix takes precedence over a less specific route.

IPv4 subnets commonly reserve an all-ones host value as the directed broadcast address. In `192.168.52.0/24`, the directed broadcast address is `192.168.52.255`. Routers normally do not forward directed broadcasts unless an administrator explicitly enables that behaviour.
### IP header essentials
The IPv4 header includes the version, header length, differentiated services field, total length, identification and fragmentation fields, time to live (TTL), protocol number, header checksum, source address, and destination address. The payload follows the header and is not itself an IP header field.

Each router that forwards an IPv4 packet reduces its TTL by at least one and discards the packet if the value reaches zero. IPv6 performs the same loop-limiting function with the Hop Limit field. The IPv4 Protocol field and IPv6 Next Header field identify the following header. Common protocol numbers include ICMPv4 1, TCP 6, UDP 17, and ICMPv6 58.

The fixed IPv6 header contains fewer fields than the IPv4 header. It includes the version, traffic class, flow label, payload length, next header, hop limit, source address, and destination address. IPv6 carries optional internet-layer information in extension headers.

Packet analysers such as Wireshark display data-link, network, transport, and application headers when the capture point and encryption state expose them.
### Layer 2 addressing and ARP
Ethernet commonly uses 48-bit MAC addresses. IEEE assigns 24-bit organisationally unique identifiers to organisations, and many globally administered EUI-48 addresses begin with one. Locally administered and randomised MAC addresses do not reliably identify the hardware manufacturer. Operating systems and virtualisation platforms can present an address that differs from hardware defaults. The purpose determines whether practitioners describe this as MAC spoofing or MAC randomisation.

Common commands include:
- Windows: `getmac` or `ipconfig /all`
- Linux: `ip link` or `ip addr`
- macOS: `ifconfig` or `networksetup`

Address Resolution Protocol (ARP) maps an IPv4 address to a link-layer address on the local broadcast domain. ARP does not resolve a remote host's MAC address. For an off-link IPv4 destination, a host normally resolves the next-hop router's MAC address and sends the frame to that router. IPv6 uses the Neighbour Discovery Protocol rather than ARP.

Neighbour information commonly appears through:
- Windows: `arp -a` or `Get-NetNeighbor`
- Linux: `ip neigh show`
- macOS: `arp -a` or `ndp -a` for IPv6 neighbours
### Routing tables and route types
Routers and endpoints maintain routing tables. Each relevant entry associates a destination prefix with an outgoing interface, a next hop, or both. Metrics and administrative preferences help select between otherwise eligible routes.

Common route types include:
- connected routes cover prefixes attached directly to an interface
- local routes identify addresses that belong to the device itself
- administrators or automation systems install static routes
- protocols such as OSPF, IS-IS, BGP, or RIP provide dynamic routes
- a default route handles a destination when no more specific route matches

Windows provides `tracert`, while Linux and macOS commonly provide `traceroute`. These tools infer a hop-by-hop path from ICMP responses to packets with increasing TTL or Hop Limit values. Filtering, load balancing, asymmetric routing, and rate limits can hide hops or produce a path that differs from other traffic.
### IPv6 addressing and operation
IPv6 uses 128-bit addresses. Its preferred text form contains eight 16-bit hexadecimal groups separated by colons.

IPv6 supports three delivery types:
- Unicast delivers a packet to one interface.
- Multicast delivers a packet to all interfaces in a selected group.
- Anycast delivers a packet to one interface in a group, normally the nearest according to routing policy.

IPv6 does not define broadcast addresses. Multicast replaces the functions that broadcasts perform in IPv4.

IPv6 text follows two main compression rules:
- Each group can omit leading zeroes.
- One double colon can replace one contiguous sequence of all-zero groups in an address.

IPv4-mapped IPv6 addresses use the prefix `::ffff:0:0/96` and can appear in mixed notation, such as `::ffff:192.0.2.128`.

IPv6 expands the address space, simplifies the base header, supports stateless address autoconfiguration, and uses scoped multicast. IPv6 standards define IPsec-related extension headers, but IPv6 does not automatically encrypt ordinary traffic. Security still depends on protocols, configuration, key management, and policy.
### Subnetting and host calculations
For IPv4, `192.168.1.0/24` corresponds to the subnet mask `255.255.255.0`. Splitting `10.0.0.0/24` into four equal subnets borrows two host bits and produces four `/26` prefixes with the mask `255.255.255.192`.

| Subnet | Conventional usable host range | Directed broadcast |
| --- | --- | --- |
| `10.0.0.0/26` | `10.0.0.1` to `10.0.0.62` | `10.0.0.63` |
| `10.0.0.64/26` | `10.0.0.65` to `10.0.0.126` | `10.0.0.127` |
| `10.0.0.128/26` | `10.0.0.129` to `10.0.0.190` | `10.0.0.191` |
| `10.0.0.192/26` | `10.0.0.193` to `10.0.0.254` | `10.0.0.255` |

Each `/26` contains `2^(32 - 26) = 64` addresses. Conventional multi-access IPv4 subnetting excludes the network and directed broadcast addresses, leaving 62 host addresses. Point-to-point `/31` links follow different rules, and `/32` identifies a single address.

Creating 16 equal prefixes from `2001:db8:85a3::/48` borrows four bits and produces `/52` prefixes. The fourth hexadecimal group advances by `0x1000`:
- `2001:db8:85a3:0000::/52`
- `2001:db8:85a3:1000::/52`
- `2001:db8:85a3:2000::/52`

A `/64` contains `2^64` interface identifier values. Network designers normally allocate IPv6 subnets by prefix policy and protocol requirements rather than by trying to conserve individual interface identifiers.
### Binary, decimal, octal, and hexadecimal
- Decimal uses base 10 and digits 0 to 9.
- Binary uses base 2 and digits 0 and 1. An octet uses place values 128, 64, 32, 16, 8, 4, 2, and 1.
- Octal uses base 8 and digits 0 to 7.
- Hexadecimal uses base 16, digits 0 to 9, and letters A to F for values 10 to 15.

For example, binary `11011010` equals `128 + 64 + 16 + 8 + 2`, or decimal 218. Decimal 235 equals binary `11101011`. With `n` digits in base `b`, a numbering system represents `b^n` values from 0 to `b^n - 1`.
## Network protocols and monitoring
### Transport protocols
| Characteristic | TCP | UDP |
| --- | --- | --- |
| Communication model | Connection-oriented byte stream | Connectionless datagrams |
| Delivery | Reliable and ordered while the connection remains viable | Best-effort delivery with no transport-level guarantee |
| State | Maintains connection and sequence state | Maintains no protocol connection state |
| Main header size | At least 20 bytes | 8 bytes |
| Typical uses | Web traffic, email, file transfer, and remote administration | Name resolution, voice, video, telemetry, and discovery |

Neither TCP nor UDP encrypts application data. Protocols such as TLS, SSH, IPsec, and secure VPN technologies provide confidentiality and integrity at other layers.
### TCP connection establishment
TCP normally establishes a connection through a three-way handshake that synchronises initial sequence numbers and confirms bidirectional communication:
1. The initiating peer sends a segment with the SYN flag.
2. The responding peer acknowledges that SYN and sends its own SYN in a SYN-ACK segment.
3. The initiating peer acknowledges the responding SYN with an ACK.

TCP then tracks bytes with sequence and acknowledgement numbers. A sender can transmit several segments before receiving an acknowledgement, subject to the receive window and congestion control. The receiver detects gaps, and the sender retransmits data that loss-recovery logic identifies as missing.
### TCP header essentials
- Source and destination ports identify the application endpoints.
- The sequence number identifies a position in the byte stream.
- The acknowledgement number identifies the next byte that the sender of the acknowledgement expects.
- Control flags include SYN, ACK, FIN, and RST.
- The receive window supports flow control.
- The checksum detects corruption across the TCP header, data, and IP pseudo-header.
- Options can support features such as maximum segment size, window scaling, selective acknowledgements, and timestamps.
### UDP characteristics and services
Applications often select UDP when they favour low latency and low overhead over transport-level loss recovery and ordered delivery. Voice and video applications can often tolerate some loss better than the delay caused by retransmission. Applications that need additional reliability can implement it above UDP.

QUIC builds encrypted, reliable, multiplexed streams over UDP and includes loss recovery and congestion control. It does not inherit those functions from UDP.

Common UDP services include DNS queries on port 53, DHCPv4 on ports 67 and 68, SNMP on ports 161 and 162, and TFTP on port 69. DNS also uses TCP for zone transfers, large responses, and retries after a truncated UDP response.
### Common application protocols
- HTTP/1.1 and HTTP/2 commonly run over TCP, with TLS providing HTTPS protection. HTTP/3 runs over QUIC and UDP and requires encryption.
- SMTP transfers email, while POP3 and IMAP retrieve or synchronise mailboxes.
- FTP transfers files over separate control and data connections.
- SFTP transfers files through SSH and is a different protocol from FTP and FTPS.
- SSH supports encrypted remote administration, tunnelling, and file transfer.
### Wireshark capture workflow
An authorised packet-capture exercise follows a controlled sequence:
1. The operator installs Wireshark and a supported capture driver where the operating system requires one. Windows installations commonly use Npcap.
2. The operator selects the relevant interface and obtains the permissions needed to capture from it.
3. The operator starts a bounded capture and generates known test traffic with a browser, name-resolution tool, or other approved client.
4. The operator applies display filters such as `tcp`, `udp`, `tcp.port == 443`, and `udp.port == 53`.
5. The operator distinguishes display filters from capture filters. For example, `tcp.port == 443` is a display filter, while `tcp port 443` is a capture filter.
6. The operator locates SYN, SYN-ACK, and ACK segments, then reviews ports, sequence numbers, acknowledgement numbers, flags, windows, and options.
7. The operator reviews UDP source port, destination port, length, and checksum fields in a DNS exchange.
8. The operator protects capture files because they can contain credentials, identifiers, session data, and other sensitive information.

Encryption can hide application content even when the capture exposes network and transport headers.
### DNS and DHCP
The Domain Name System (DNS) stores distributed resource records. It maps names to IPv4 and IPv6 addresses, identifies mail exchangers and name servers, supports aliases, and publishes other service information.

The Dynamic Host Configuration Protocol for IPv4 (DHCPv4) leases IPv4 addresses and supplies options such as a subnet mask, router, DNS servers, and lease time. A new client commonly uses the four-message Discover, Offer, Request, and Acknowledgement sequence. Renewals, rebindings, releases, declines, and information requests use other exchanges.

DHCPv4 clients initially use broadcasts when they lack usable addressing or server information. Servers and clients can later use unicast in defined states. A DHCP relay carries client and server messages between subnets so each subnet does not require a local server.
### DNS filtering
DNS filtering applies policy during name resolution. A managed resolver or security service can allow a query, refuse it, return a policy response, or direct the client to a controlled sinkhole.

Security policies can block domains associated with phishing, malware distribution, and command-and-control infrastructure. Acceptable-use policies can also restrict selected content categories. Resolver, firewall, cloud, and endpoint implementations can generate logs for investigation and audit.

DNS filtering cannot inspect every connection or stop access by direct IP address. Encrypted DNS can bypass network-level policy unless administrators manage client configuration and egress paths. Effective programmes review blocking decisions, exceptions, privacy requirements, false positives, and threat intelligence quality.
### Syslog
Syslog transports event messages between originators, relays, and collectors. RFC 5424 defines three conceptual layers: syslog content, syslog applications, and syslog transport. It defines the message format independently of a single transport, but conforming implementations must support the TLS transport mapping and should support the UDP transport mapping. Deployments should select a mapping that meets their reliability and security requirements.

A syslog priority combines a facility value, which identifies a broad source category, with a severity value:

| Severity | Name |
| --- | --- |
| 0 | Emergency |
| 1 | Alert |
| 2 | Critical |
| 3 | Error |
| 4 | Warning |
| 5 | Notice |
| 6 | Informational |
| 7 | Debug |

Central collection supports search, correlation, retention, alerting, and investigation. Administrators should synchronise time, protect logs in transit and at rest, restrict access, monitor collection failures, and define retention according to operational and legal needs.
### Network flows and flow analysis
A network flow groups packets that share selected properties during an observation interval. Common flow keys include source and destination IP addresses, source and destination ports, and protocol. Flow records can also contain byte and packet counts, start and end times, interfaces, TCP flags, and quality-of-service markings.

NetFlow names a family of Cisco-originated flow technologies, while IPFIX defines an IETF standard for exporting flow information. sFlow uses statistical packet sampling and interface counters rather than reproducing the same flow-record model.

Flow analysis supports traffic profiling, capacity planning, troubleshooting, and threat hunting. Analysts can investigate denial-of-service patterns, scanning, unexpected protocols, lateral movement, and unusual outbound transfers. Flow records summarise communications and normally do not contain full packet payloads.
### Port mirroring and capture modes
Port mirroring copies traffic from selected switch ports or VLANs to a monitoring destination. Cisco platforms commonly call local mirroring SPAN and remote mirroring RSPAN. Other suppliers use different terms, and network taps provide another way to observe traffic.

A wired capture interface in promiscuous mode accepts frames that the interface would otherwise discard because of the destination MAC address. Promiscuous mode alone does not make a switch send unrelated traffic to the capture host, so the design still needs a mirror, tap, hub, or suitable network position. Wireless analysis can require monitor mode to capture 802.11 management and control frames.

Passive intrusion detection systems can analyse mirrored traffic without forwarding production packets. Mirroring can oversubscribe the destination and drop copies, so engineers must account for aggregate bandwidth and switch capabilities. Strong access controls and retention rules protect the sensitive traffic that monitoring systems can expose.
### User and entity behaviour analytics
User and entity behaviour analytics (UEBA) combines activity data, context, and baselines to identify unusual behaviour associated with users, accounts, hosts, or applications.

Data sources include authentication records, application logs, endpoint telemetry, cloud audit logs, and network flows. Statistical models, rules, and machine-learning techniques can identify unusual login patterns, privilege use, access to sensitive data, and outbound transfer behaviour.

Correlation improves triage. For example, an unusual login location and an abnormal outbound transfer raise the risk score more than either signal alone, but they do not prove account compromise. Analysts still need to validate identity, device, travel, application, and business context.

UEBA programmes require representative data, controlled access, documented retention, privacy review, regular tuning, and measurement of false positives and missed detections. Attackers can also imitate normal behaviour or poison weak baselines.
## Network security techniques
### Packet inspection and firewalls
Packet inspection compares observed traffic with security policy. Different firewall functions provide different context:

| Function | Inspection basis | Main limitation |
| --- | --- | --- |
| Stateless filtering | Addresses, ports, protocol, direction, and other packet-header fields | It cannot determine whether a packet belongs to an established connection |
| Stateful inspection | Packet headers plus tracked connection state | It provides limited understanding of application content without additional inspection |
| Application-aware inspection | Protocol parsing, application identity, commands, methods, or content | Encryption, evasion techniques, unsupported protocols, and processing load can reduce visibility |
| Proxying | Terminates one connection and creates another on behalf of a client or server | It requires protocol support, capacity, certificate management where relevant, and carefully designed policy |

A stateful firewall commonly keys sessions with source and destination addresses, source and destination ports, and protocol. It can also record TCP state, interfaces, zones, policy identifiers, translation details, counters, and timeouts. Return traffic must match valid state and policy before the firewall allows it.

Platforms apply filtering, routing, network address translation, decryption, application inspection, quality of service, and threat prevention in different orders. Engineers must follow the relevant platform documentation and validate the effective policy rather than assume one universal packet-processing sequence.
### Intrusion detection and prevention
An intrusion detection system (IDS) analyses network or host activity and reports suspected misuse, anomalies, or policy violations. An intrusion prevention system (IPS) can also block traffic, reset connections, quarantine a file, isolate a host, or invoke another response.

A network-based IDS or IPS analyses traffic from selected network segments. A host-based system analyses activity on an endpoint, including processes, files, system calls, and logs. Network sensors can operate passively from a mirror or tap, while prevention sensors often operate inline or integrate with another enforcement point.

Detection techniques include signatures, protocol analysis, heuristics, reputation, and anomaly detection. Each technique can produce false positives or miss attacks. An inline IPS can also block legitimate traffic, add latency, or reduce availability, so teams must size, test, monitor, and tune it carefully.
### Network address translation
Network address translation (NAT) changes IP addressing information as packets cross a translation device. Port address translation also changes transport ports. NAT conserves public IPv4 addresses and can connect networks with incompatible address plans, but it does not authenticate traffic or replace a firewall.

- Static NAT maintains a fixed mapping between addresses. Publishing a service also requires appropriate destination translation, routing, and security policy.
- Dynamic NAT allocates mappings from an address pool as traffic requires them.
- Port address translation (PAT), or NAT overload, lets many internal flows share one public IPv4 address by assigning distinct transport-port mappings.

NAT can complicate peer-to-peer applications, inbound services, IPsec modes, geolocation, attribution, and protocols that embed addresses in application data. Session Initiation Protocol can require NAT traversal techniques or carefully managed application-layer gateways. Overlapping private ranges also complicate mergers, partner links, and virtual private networks.
### Limits of firewalls and IDS
A firewall controls traffic that crosses its enforcement point. It cannot govern traffic that bypasses that point or activity that remains entirely within a host. Internal segmentation firewalls can reduce this gap, but architecture and routing must force relevant traffic through them.

Valid credentials, permitted applications, novel exploits, and encrypted channels can make malicious activity resemble authorised use. Decryption can restore some visibility, but it introduces privacy, legal, certificate, compatibility, and performance risks. Configuration errors can also expose services or disrupt business traffic.

An IDS usually alerts rather than blocks. It can lose visibility through encryption, packet loss, asymmetric paths, unsupported protocols, fragmentation, or obfuscation. High event volumes and poor tuning can overwhelm analysts.

Layered defence combines segmentation, firewalls, IDS or IPS, secure configuration, identity controls, endpoint security, application security, vulnerability management, backups, threat intelligence, and central logging. Each control should address a defined risk and produce evidence that operators can test.
### File integrity monitoring
File integrity monitoring (FIM) detects changes to selected files, directories, registry data, or configuration objects. A trusted baseline records content fingerprints and relevant metadata such as path, owner, permissions, and timestamps.

FIM tools commonly use cryptographic hashes to detect content changes. SHA-256 or a stronger approved algorithm provides greater collision resistance than MD5 or SHA-1. Where practical, administrators should store and protect the baseline and monitoring configuration separately from the systems they assess.

A useful FIM alert identifies the affected object, change type, time, available account or process details, and expected change window. Responders verify authorisation, examine related activity, restore a known-good state when required, and correct the underlying weakness. FIM detects change but does not determine intent by itself.
### Data loss prevention
Data loss prevention (DLP) identifies and controls sensitive information across endpoints, networks, applications, and storage. Programmes commonly address data at rest, in transit, and in use.

DLP combines data discovery and classification with content inspection, labels, user and device context, destination, transfer method, and policy. Controls can alert, block, quarantine, encrypt, redact, or require justification. Tokenisation and rights-management systems can complement DLP.

Implementation starts with defined data classes, owners, authorised uses, and prioritised data-loss scenarios. Teams then deploy monitoring and enforcement in stages, tune false positives, protect collected content, train users, and establish an exception process. Incident handling validates the alert, contains exposure, preserves evidence, determines scope and cause, and improves controls.
### Network access control
Network access control (NAC) grants, limits, or denies connectivity according to identity, role, device ownership, and security posture. IEEE 802.1X commonly supplies port-based authentication on wired and wireless networks, although NAC products can also use other discovery and enforcement methods.

NAC can check certificate status, operating-system version, patch level, endpoint protection, and device management state. Policy can place a non-compliant or unknown device in a restricted segment for remediation instead of granting normal access.

NAC strengthens segmentation and admission control when it integrates with identity services, switches, wireless infrastructure, firewalls, and endpoint management. Safe deployment requires high availability, staged enforcement, exception handling, guest and unmanaged-device policy, and tested failure behaviour.
### EDR and XDR
Endpoint detection and response (EDR) collects and analyses endpoint telemetry to detect, investigate, and respond to suspicious activity. Coverage commonly includes workstations and servers, while support for mobile, embedded, and specialised systems varies by product.

EDR telemetry can include process creation, command lines, file activity, registry or configuration changes, user sessions, network connections, and security events. Detection combines signatures, reputation, heuristics, behavioural analytics, and threat intelligence. Investigation features can reconstruct timelines and relate parent and child processes, files, users, and connections.

Response actions can isolate a host, terminate a process, quarantine a file, block an indicator, collect forensic artefacts, or run an approved remediation action. Teams should test each action, protect the management plane, monitor agent health, and require human approval for high-impact automation when the risk warrants it.

Extended detection and response (XDR) correlates telemetry and detections across several security domains, such as endpoints, identity, email, networks, cloud services, and applications. Product scope and integration depth vary. XDR can reduce duplicate alerts and add cross-domain context, but it does not replace sound controls, complete telemetry, skilled investigation, or rehearsed response procedures.
### Xcitium OpenEDR and Xcitium Enterprise
Xcitium OpenEDR is an open-source endpoint detection and response project. Its endpoint agent records telemetry locally, which a separate log shipper can send to a self-hosted or cloud-hosted Elasticsearch deployment for analysis. OpenEDR does not include every endpoint-management feature sold through the broader Xcitium platform.

Xcitium Enterprise provides a central endpoint-management platform with capabilities that can include device enrolment, EDR, configuration profiles, antivirus controls, software inventory, and operating-system or third-party patch management. Available features, operating-system support, licensing, and console paths vary by product version and subscription.

A controlled Xcitium Enterprise deployment includes:
- confirming supported operating systems, network requirements, licensing, and data-handling obligations
- protecting administrative accounts with strong authentication and least privilege
- piloting the applicable communication, security, and EDR agents on representative endpoints
- applying profiles to defined device groups and validating telemetry, alerts, and policy effects
- testing containment, scanning, quarantine, patching, rollback, and agent removal procedures
- tuning alerts and recording changes through formal change control