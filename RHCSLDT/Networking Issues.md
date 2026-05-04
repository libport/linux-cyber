# Networking Issues
## Network connectivity
Network diagnosis should move from reachability to service access and then to configuration. The administrator should identify the interface, address, route, DNS servers, firewall state, and listening service.

Basic verification tools include:
- `ip addr` to show addresses.
- `ip route` to show routes.
- `ping HOST` to test ICMP reachability.
- `telnet HOST PORT` to test a TCP port.
- `nc -vz HOST PORT` to test TCP more flexibly.
- `curl URL` to test HTTP services.
- `ss -lntp` to show listening TCP ports.
- `ss -antp` to show TCP connections.

DNS diagnosis uses bind-utils. `dig NAME`, `host NAME`, and related commands show whether name resolution works and which servers answer. The resolver configuration normally appears through `/etc/resolv.conf`, often managed by NetworkManager. Do not edit generated resolver files without checking NetworkManager state.

NetworkManager configuration uses `nmcli`. `nmcli device status` shows device state. `nmcli connection show` lists connection profiles. `nmcli connection modify NAME ipv4.addresses ADDRESS ipv4.gateway GATEWAY ipv4.dns DNS ipv4.method manual` updates a profile. `nmcli connection up NAME` activates it. For temporary tests, `ip addr add`, `ip route add`, and related `ip` commands may be faster, but they do not persist unless saved in the connection profile.

Firewalld commonly blocks services that work locally. `firewall-cmd --list-all` shows the active zone configuration. `firewall-cmd --add-service=http --permanent` or `firewall-cmd --add-port=80/tcp --permanent` adds persistent access. `firewall-cmd --reload` applies permanent changes. Always check the active zone and interface before assuming the rule applies.

Packet capture clarifies ambiguous network faults. `tcpdump` and `tshark` both capture packets through libpcap style filters. Common options include `-i INTERFACE` for interface, `-w FILE` to write a capture, `-c COUNT` to stop after a number of packets, and `-r FILE` to read a capture. Captures show whether packets leave, arrive, receive replies, or fail during handshake.

Network troubleshooting should distinguish local service failure from path failure. A successful `curl localhost` proves the application can answer locally. A failed remote connection may still come from firewall rules, binding to loopback only, routing, DNS, or cloud security controls. A failed `curl localhost` points back to the service, application configuration, SELinux, file permissions, or the listening socket.

Name resolution deserves a separate test. A hostname failure with a successful IP connection usually means DNS or `/etc/hosts`. An IP failure means DNS is not the main problem. `dig` output shows the server queried, the answer section, and timing. If `/etc/resolv.conf` contains the wrong server but NetworkManager owns the file, update the NetworkManager connection rather than making a temporary edit.

Routing faults often show through asymmetry. The client may reach the server, but replies leave through the wrong gateway or interface. `ip route get DESTINATION` shows the chosen route. `tcpdump` on both sides can prove whether packets arrive and whether replies leave. This evidence prevents unnecessary service changes when the real issue is path selection.

Firewall repair should use services where possible because firewalld service definitions capture standard ports and protocols. Custom ports require `--add-port`. Runtime rules disappear after reload or reboot unless made permanent. Permanent rules do not affect the current runtime until reload. The administrator should verify both the runtime state and the persistence requirement.