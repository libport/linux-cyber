# Networking Issues
## Troubleshooting Linux Network Connectivity
Linux network troubleshooting starts with a clear path through the stack. Applications such as HTTPD and SSH generate data. The transport layer splits that data into segments and carries it with TCP or UDP. The network layer routes packets between networks. The data link and physical layers move frames through interfaces, MAC addresses, cables, switches, and virtual network devices. The TCP/IP model compresses these functions into application, transport, internet, and network access layers.

A useful diagnosis asks where traffic fails. The administrator should first confirm basic reachability, then test the required protocol and port, then inspect the service, firewall, name resolution, routes, and interface configuration.
## Verify Connectivity
`ping` checks IP reachability with ICMP echo requests. It reports response times and packet loss, but it does not test application ports. Useful options include:
- `ping -c 5 <host>` to send five probes
- `ping -s 1024 <host>` to set the ICMP payload size
- `ping -f <host>` to flood a host where privileges and policy allow it

Port checks need tools that speak transport protocols. `telnet <host> <port>` can confirm whether a TCP service accepts a connection, although it does not validate application correctness or encryption. `ncat` or `nc` provides broader testing because it can connect over TCP or UDP and can also listen on a chosen port.

A simple TCP test can use one host as a listener and another as a client:

```bash
nc -l 443
nc <server-ip> 443
```

Traffic that passes between these sessions shows that the network path and port can work. If the test succeeds but the real application still fails, the service configuration, service state, firewall rules, or security controls need closer review.
## Diagnose and Fix Connectivity Failures
A failed connection should prompt several checks:
- Can the host reach the network at all?
- Can it reach another host on the same network?
- Can it reach the destination host?
- Can it reach the required TCP or UDP port?
- Does the target service listen on the expected address and port?
- Does a firewall or security policy block the traffic?

On Red Hat Enterprise Linux 8, `firewalld` commonly manages firewall policy. The service status shows whether it runs:

```bash
systemctl status firewalld
```

`firewall-cmd --list-all` shows the active zone, interfaces, services, and ports. If HTTPD works locally but not from another host, the firewall may allow SSH but block HTTP. The administrator can open HTTP at runtime with either a service or a port rule:

```bash
firewall-cmd --zone=public --add-service=http
firewall-cmd --zone=public --add-port=80/tcp
```

Runtime changes disappear when `firewalld` reloads or the system restarts. Use `firewall-cmd --runtime-to-permanent` to save the current runtime policy. Alternatively, add the rule with `--permanent` and reload `firewalld`.

Name resolution and routing also cause failures. `bind-utils` provides `dig` and `host`, which help test DNS lookups and identify the resolver in use. Resolver configuration normally appears in `/etc/resolv.conf`, although NetworkManager or system resolver services may manage that file.

Interface and route checks should rely on live system state, not old persistent udev naming files. Useful commands include:

```bash
ip link show
ip addr show
ip addr show dev eth0
ip route show
ip route show default
nmcli device status
nmcli connection show
nmcli device show eth0
```

`ip` can apply immediate address and route changes, such as `ip route add default via <gateway> dev <interface>`. These changes usually do not survive a restart. Persistent Red Hat Enterprise Linux network configuration should use NetworkManager connection profiles through `nmcli`, `nmtui`, or another supported NetworkManager interface.
## Inspect Network Traffic
Packet capture confirms what actually crosses an interface. `tcpdump`, Wireshark, and TShark work with packet capture files and libpcap-style capture filters. TShark is the command line form of Wireshark. Modern Wireshark tools commonly write pcapng by default, while tcpdump commonly writes pcap.

A capture can print packets directly, write them to a file, or read a saved capture:

```bash
tcpdump -i eth0
tshark -i eth0 -c 5 -w /tmp/file.pcap
tcpdump -r /tmp/file.pcap
tshark -r /tmp/file.pcap
```

The `-i` option selects the interface, `-c` limits the packet count, `-w` writes a capture file, and `-r` reads a saved capture. Capture filters reduce noise before packets enter the file. For example, `port 80` captures traffic involving TCP or UDP port 80. More precise filters, such as `tcp port 80`, avoid unrelated UDP traffic.

Traffic inspection helps distinguish network failure from service failure. If captures show incoming HTTP requests and no reply, the service or host policy needs investigation. If captures show no incoming requests, routing, firewall policy, source traffic, or an upstream network device may block the path.
## Key Commands
- `ping` tests ICMP reachability.
- `telnet` tests basic TCP connection attempts.
- `nc` or `ncat` tests TCP or UDP paths and can create listeners.
- `firewall-cmd` reads and changes `firewalld` policy.
- `dig` and `host` test DNS behaviour.
- `ip` inspects and changes live interface and route state.
- `nmcli` inspects and changes NetworkManager configuration.
- `tcpdump` and `tshark` capture and read network traffic.