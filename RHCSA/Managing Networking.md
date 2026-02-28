# Managing Networking
## Managing the TCP/IP Stack
Red Hat Enterprise Linux 8 replaces legacy networking tools such as ifconfig, route, and arp with the unified ip command. This single utility manages addresses, routes, ARP cache entries, multicast settings, and network namespaces. Administrators use ip address show or the abbreviated ip a to view interface configuration, and they add runtime addresses with elevated privileges using ip address add.

Runtime changes modify the active configuration only. Rebooting or restarting an interface removes these settings unless they are persisted through NetworkManager or configuration files. The ip command improves consistency because its syntax remains similar across networking tasks.
## Working with Runtime IP Addresses
The ip command displays both IPv4 and IPv6 information and allows filtering by protocol or interface. Administrators typically verify connectivity by examining interface status, assigned addresses, and routing information. Adding a temporary address requires sudo privileges and takes effect immediately, but it does not survive a reboot. This behaviour is intentional and useful for testing.
## Understanding the ARP Cache
Address Resolution Protocol maps IP addresses to MAC addresses so systems can deliver frames on the local network. The ip neighbor command replaces the legacy arp utility. It displays neighbour states such as REACHABLE and STALE, indicating whether the mapping is current.

Administrators can remove entries with ip neighbor delete and refresh them by generating traffic such as ping. Systems create ARP entries only for hosts on the same subnet. Traffic destined for remote networks resolves only the default gateway MAC address.
## Controlling ARP Cache Timeout
Linux manages neighbour ageing through sysctl parameters such as gc_stale_time. The default value is typically 60 seconds. Administrators can inspect current values under /proc/sys/net and modify them with sysctl -w. Changing the default affects only new interfaces, so each active interface may also require adjustment.

To make changes persistent, administrators add the required keys to /etc/sysctl.conf so the system applies them at boot. Careful tuning prevents stale or incorrect mappings from remaining in memory for too long.
## Network Namespaces
Network namespaces provide isolated networking stacks on a single host. Each namespace maintains its own interfaces, routing tables, firewall rules, and ARP cache. Administrators create namespaces with ip netns add and manage them using ip netns exec.

Virtual ethernet pairs connect namespaces. Creating a veth pair places one interface in the default namespace and the peer in another namespace. Each interface requires an IP address and must be brought up before communication can occur. Namespaces are widely used in container and virtualisation environments because they provide strong network isolation.
## Routing Between Namespaces
Systems know routes to directly connected networks and to the default gateway. When additional subnets exist inside namespaces, administrators add static routes with ip route add. These routes specify the destination network and the next hop interface.

After defining routes and assigning addresses on both ends of the virtual link, hosts can communicate across namespaces. This process demonstrates how Linux routing decisions depend on the routing table rather than simple interface connectivity.
## Persisting Network Configuration
Runtime configuration is temporary. Persistent configuration ensures that interfaces and services start automatically after reboot. Red Hat systems traditionally store interface profiles in /etc/sysconfig/network-scripts using ifcfg files. Although these files still exist, NetworkManager now manages them dynamically.

The NetworkManager service must be active and enabled. When the service starts, it reads connection profiles marked with ONBOOT=yes and brings those interfaces online. Administrators can still view or edit the files directly, but using NetworkManager tools is preferred.
## Managing Connections with nmcli
The nmcli command line utility provides full control of NetworkManager. Devices represent physical interfaces, while connections represent configuration profiles applied to those devices. Administrators list devices with nmcli device status and view profiles with nmcli connection show.

Creating a connection requires specifying the interface name, IP method, addressing information, and a connection identifier. Tab completion simplifies the process. After creation, the connection becomes active only when brought up with nmcli connection up. The command automatically writes persistent configuration to the appropriate ifcfg file.
## DNS Configuration and Priority
NetworkManager writes DNS settings to /etc/resolv.conf based on active connections. Administrators modify DNS servers using nmcli connection modify and set values such as ipv4.dns. Multiple connections can contribute DNS entries, and the dns-priority parameter controls ordering.

Lower numeric values represent higher priority. Setting a negative priority forces exclusive use of that DNS server, which is useful in controlled or security-sensitive environments. Reactivating the connection applies the changes and updates both the runtime configuration and the persistent profile.
## Key Practices
Administrators should rely on the ip command for modern networking tasks and avoid deprecated tools. They should distinguish between runtime and persistent configuration, ensure NetworkManager is enabled, and verify routing when troubleshooting connectivity. Network namespaces and virtual interfaces provide powerful isolation features, while nmcli offers consistent management of persistent settings.
## Conclusion
Effective networking in Red Hat Enterprise Linux 8 depends on understanding the unified ip toolset, NetworkManager integration, and proper routing design. By combining runtime testing with persistent configuration, administrators can build reliable and secure Linux network environments.