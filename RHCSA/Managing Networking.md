# Managing Networking
## Networking essentials
Networking administration centers on four linked tasks. Administrators inspect and change live network state with the `ip` command. They persist interface settings with NetworkManager and `nmcli`. They filter traffic with `firewalld`, which manages the underlying `nftables` framework. They add tighter controls with direct `nft` rules or with Fail2Ban for repeated login abuse.

The `ip` command replaces the older pattern of using separate tools such as `ifconfig`, `route` and `arp`. It provides a consistent syntax for addresses, routes, links and neighbour entries. In practical terms, that means one command family handles most routine TCP/IP work on a RHEL host. Common inspections include these forms:

```bash
ip a
ip -4 a show dev eth1
ip r
ip neigh
ip link
```

Short forms matter because they make day to day administration faster without changing meaning. `ip a` shows addresses because `show` is the default action. `ip r` shows the route table. `ip neigh` shows the neighbour cache, which replaces the older ARP display tools for IPv4 and also covers IPv6 neighbour discovery. The command family stays readable even when abbreviated.

Runtime changes apply immediately and disappear when the interface resets or the system restarts, unless another service persists them. A live address can be added like this:

```bash
sudo ip addr add 172.16.1.1/16 dev eth1
```

That command adds a secondary address to `eth1` in the running kernel state. It does not survive reboot. This distinction between runtime and persistent state is fundamental throughout RHEL networking. Administrators use `ip` for fast testing and temporary adjustments, then use NetworkManager to keep changes.
## Runtime addressing, neighbour state and route control
A host needs more than an IP address to move packets on a local network. The kernel must also resolve the destination IP address to a layer 2 hardware address. On IPv4 networks, the neighbour cache stores mappings between IP addresses and MAC addresses. `ip neigh` displays those mappings and the state of each entry.

Entries usually appear as `REACHABLE` when the host has communicated with the neighbour recently, or `STALE` when the mapping still exists but has not been refreshed within the current ageing window. This behaviour matters during troubleshooting. A failed ping to a local host may point to a missing or stale neighbour entry, while successful access to a remote internet address usually uses the default gateway rather than a direct cache entry for the remote server.

Administrators can delete a neighbour entry and force fresh resolution:

```bash
sudo ip neigh del 192.168.33.12 dev eth1
```

The next packet to that local host triggers a new address resolution step. This is useful when a host has changed NICs, a virtual machine has moved, or a stale mapping is suspected.

Neighbour ageing is controlled by kernel parameters. The key value in this material is `gc_stale_time`, not the mistranscribed `gc_scale_time`. The default is commonly 60 seconds. A running system can inspect the value through `/proc/sys` or `sysctl`:

```bash
cat /proc/sys/net/ipv4/neigh/eth0/gc_stale_time
sysctl -a | grep gc_stale_time
```

A temporary change can be written with `sysctl -w`, and a persistent change can be stored in `/etc/sysctl.conf` or a drop in file under `/etc/sysctl.d/`. Raising the value keeps entries marked as reachable for longer. That can reduce neighbour discovery traffic on stable networks, but it also delays refresh if an incorrect mapping appears.

Route inspection uses the same `ip` tool family. `ip r` shows directly connected networks and the default route. RHEL learns routes automatically for local interfaces, but additional subnets often need explicit static routes. A simple example is a host with a private lab network behind a virtual link. The host may know its local `10.0.0.0/24` segment and its default gateway, yet still lack a route to `192.168.100.0/24` behind a peer interface. A static route fixes that:

```bash
sudo ip route add 192.168.100.0/24 via 10.0.0.1
```

In the course example, the private network sits behind a virtual Ethernet pair, so the route is added to reach a subnet inside a separate namespace. The important concept is broader. Private address ranges are not advertised on the public internet, so internal networks must be reached through explicit internal routing. When the kernel does not know a path to a private subnet, the default route is usually wrong.
## Network namespaces and virtual links
Network namespaces let one RHEL system run isolated network stacks side by side. Each namespace has its own interfaces, routes, neighbour table and local services. This is useful for labs, containers, testing, service isolation and software defined networking.

A new namespace can be created with:

```bash
sudo ip netns add net1
ip netns show
```

The host still has its default namespace even when `ip netns show` lists only custom ones. A new namespace starts almost empty. It usually contains only a loopback interface, which may need to be brought up if local testing inside that namespace matters.

Namespaces become useful when a link connects them. A virtual Ethernet pair, or veth pair, acts like a patch cable with two ends. One end stays in the default namespace and the other moves into the new namespace:

```bash
sudo ip link add veth0 type veth peer name veth1 netns net1
```

After that, `veth0` exists in the default namespace and `veth1` exists inside `net1`. Each end can receive an address and be brought up independently:

```bash
sudo ip addr add 10.0.0.2/24 dev veth0
sudo ip link set veth0 up

sudo ip netns exec net1 ip addr add 10.0.0.1/24 dev veth1
sudo ip netns exec net1 ip link set veth1 up
```

With matching addresses on the same subnet, the default namespace can reach `10.0.0.1` and the namespace can reach `10.0.0.2`. Without those addresses and active links, the virtual cable exists but the stacks still cannot communicate.

The power of namespaces appears when they model a separate routed network. The namespace can hold extra addresses and local services that the default namespace does not know yet. For example, adding `192.168.100.1/24` to `veth1` creates a new subnet inside `net1`. The default namespace still cannot reach that subnet until a static route points traffic through the peer on the shared `10.0.0.0/24` link. That is why namespaces make route concepts clearer. They create a small, self contained routing problem on a single host.

Administrators can inspect namespace state with standard commands prefixed by `ip netns exec`. That keeps the toolset consistent:

```bash
ip netns exec net1 ip a
ip netns exec net1 ip r
ip netns exec net1 ping -c 1 10.0.0.2
```

This approach makes namespaces valuable for learning and for production style segmentation. Each namespace behaves like an independent host from the network stack's point of view.
## Persistent network configuration with NetworkManager
Runtime testing is useful, but RHEL needs a persistent mechanism to rebuild network settings on boot. NetworkManager provides that layer. It manages connection profiles, activates them on devices and writes the active resolver configuration.

In RHEL, connection profiles commonly appear as `ifcfg-*` files under `/etc/sysconfig/network-scripts/`. Those files remain usable and NetworkManager can manage them. The source material treats them as the main persistent form, which fits RHEL. A more future facing note is that keyfile profiles support all NetworkManager settings, while the older `ifcfg` format has limits. For this RHEL workflow, however, the key idea is simple. Profiles define persistent settings and NetworkManager applies them.

A profile is not the same thing as a device. A device is the physical or virtual NIC such as `eth0` or `eth1`. A connection is the software profile that binds address settings, routes, DNS behaviour and startup options to a device. One device can have different profiles for different environments.

Typical profile files include values such as these:
- device name
- static or DHCP method
- IP address and prefix
- gateway
- DNS behaviour
- whether the profile starts on boot
- whether NetworkManager controls it

This model explains why a host can switch from one network context to another by changing the active profile on the same NIC.
## Managing connections with nmcli
`nmcli` is the command line interface to NetworkManager. It can list devices, list connections, create new profiles, modify existing ones and activate or deactivate them. It also writes the persistent configuration, so manual file editing is often unnecessary.

Useful discovery commands include:

```bash
nmcli device
nmcli connection
systemctl status NetworkManager
```

`nmcli device` shows physical and logical interfaces. `nmcli connection` shows saved profiles and indicates which one is active. `NetworkManager` must be both running and enabled for profile based startup to work consistently.

A new static profile can be created for a specific interface with a command like this:

```bash
sudo nmcli connection add \
  type ethernet \
  ifname eth1 \
  ipv4.method manual \
  ipv4.addresses 192.168.100.1/24 \
  connection.id cafe
```

This example deliberately uses a named profile to show that the administrator controls profile identity. Once created, the profile can be activated with:

```bash
sudo nmcli connection up cafe
```

Activation matters because a saved profile does nothing until NetworkManager applies it. After activation, the generated `ifcfg-cafe` file records the persistent settings, and the interface starts using them immediately.

One of `nmcli`'s main advantages is tab completion. The command exposes many properties, but shell completion removes most of the memorisation burden. That makes it practical for administrators to explore valid subcommands, property names and connection identifiers without constant reference checks.

A generated `ifcfg-*` profile also shows why manual editing can feel deceptively simple. The file may only list a handful of familiar keys, but `nmcli` keeps the running state, the on disk profile and the NetworkManager view aligned. That reduces the risk of a mismatch between a hand edited file and the active connection state. For routine administration, `nmcli` is therefore the safer default, while direct file edits remain useful for review, templating and recovery.
## DNS control and resolver priority
A persistent connection includes more than an address. It can also define DNS servers, search domains, gateways and route preferences. In RHEL, NetworkManager writes `/etc/resolv.conf`, so that file reflects the currently active connection set rather than serving as the primary source of truth.

A DNS server can be added to an existing connection with a command like this:

```bash
sudo nmcli connection modify cafe ipv4.dns 8.8.8.8
sudo nmcli connection up cafe
```

After the profile comes up again, `/etc/resolv.conf` includes the new server. If another active profile already contributes DNS servers, the new server might not appear first. Resolver order is controlled by DNS priority.

The important rule is straightforward. Lower values have higher priority. A standard profile defaults to `100`. VPN profiles often use lower values. Setting a profile's `ipv4.dns-priority` to `10` places its DNS server ahead of a profile that still uses `100`:

```bash
sudo nmcli connection modify cafe ipv4.dns-priority 10
sudo nmcli connection up cafe
```

Negative priorities are more restrictive. A value such as `-1` does not merely push the server to the top. It excludes DNS servers from profiles that do not also use negative priority. That makes negative values useful on networks where a host must consult only a nominated resolver and no fallback should occur.

This behaviour is significant for security and split network design. A machine on a trusted corporate VPN may need to resolve names only through internal servers. A laptop on a public network may need to favour a local resolver or a privacy preserving external service. NetworkManager can express those rules in the profile rather than relying on manual edits to `resolv.conf`, which would be overwritten.
## firewalld, zones and runtime versus permanent rules
RHEL secures host traffic with `firewalld` as the management layer over `nftables`. `firewalld` introduces a zone model that groups trust levels and allowed traffic. Zones can be attached to interfaces or to source addresses, and each zone can allow services, ports and richer rules.

The first operational checks are simple:

```bash
sudo firewall-cmd --state
sudo firewall-cmd --get-default-zone
sudo firewall-cmd --list-all
```

A default installation commonly uses the `public` zone. The listing shows associated interfaces and services. Services are more readable than raw ports because each service definition can bundle ports, protocols and helper information. For example, the `http` service normally maps to TCP port 80.

A major `firewalld` concept is the separation between runtime and permanent configuration. Runtime rules take effect immediately and vanish after reload or restart unless they are saved. Permanent rules are stored on disk and repopulate runtime state when `firewalld` starts. This supports safer administration. An administrator can test a live change, verify connectivity and only then save it.

Allowing a service temporarily is as direct as:

```bash
sudo firewall-cmd --add-service=http
```

That opens the service in the active zone at runtime. A remote system can then test access. If the change works, it can be copied into the permanent configuration:

```bash
sudo firewall-cmd --runtime-to-permanent
```

The corrected idea here is "copy runtime to permanent", not "flush" in the storage sense. A restart of `firewalld` then reproduces the rule.

Service and zone definitions come from XML files. Vendor supplied defaults live under `/usr/lib/firewalld`. Local changes belong under `/etc/firewalld`, which overrides the vendor copy. That split helps package updates deliver new defaults without overwriting local policy.
## Source based access, temporary rules and custom services
An interface based zone works when trust is tied to where traffic enters the host. That is often enough for simple servers. It becomes less useful when the same interface receives both trusted and untrusted traffic. Source based rules solve that problem.

A service can be added to a more trusted zone such as `internal`, then that zone can be activated for a source subnet or host:

```bash
sudo firewall-cmd --zone=internal --add-service=http
sudo firewall-cmd --zone=internal --add-source=192.168.33.0/24
```

This pattern allows HTTP access only for clients from the nominated source range. The service is not simply open to every host that reaches the interface. That is a stronger design for branch offices, management subnets and lab networks.

`firewalld` can also manage raw ports directly. That is useful when no suitable service definition exists, or when a quick test is needed:

```bash
sudo firewall-cmd --zone=internal --add-port=443/tcp
```

A temporary opening can include a timeout:

```bash
sudo firewall-cmd --zone=internal --add-port=443/tcp --timeout=10s
```

After the timeout expires, the port disappears from runtime configuration automatically. This is a practical safeguard when remote administrators need to test access without risking a long lived mistake.

Custom service definitions provide a cleaner long term answer than scattering individual port rules. The common method is to copy the vendor XML file into `/etc/firewalld/services/`, edit it and reload `firewalld`. For example, a local copy of `http.xml` can be expanded to include both TCP 80 and TCP 443. Once reloaded, any zone that allows `http` now permits both ports according to the custom local definition.

These mechanisms show the strength of `firewalld`. It offers a high level interface for common policy work while still preserving the distinction between tested runtime state and durable on disk policy.
## Fail2Ban for repeated login abuse
Port based filtering limits exposure, but it does not stop repeated authentication attempts against an allowed service such as SSH. Fail2Ban addresses that gap. It watches logs for repeated failures and inserts blocking rules against abusive source addresses.

On RHEL based systems, Fail2Ban is commonly installed from EPEL. The exact package repository arrangement varies by environment, but the workflow stays the same. Install the repository package, install Fail2Ban, create a jail definition, enable the service and inspect jail status.

The SSH jail in this material lives under `/etc/fail2ban/jail.d/sshd.conf`. The edited transcript contains an inconsistency between seven failures and three failures. The demonstrated configuration uses three failed logins within ten minutes and a 72 hour ban, so that is the corrected working example. A concise configuration pattern looks like this:

```ini
[DEFAULT]
bantime = 72h
findtime = 10m
maxretry = 3
backend = systemd

[sshd]
enabled = true
```

With the service running, status can be checked through:

```bash
sudo fail2ban-client status sshd
```

That shows current failures, totals and banned addresses. The source example uses a public AWS host to illustrate how quickly exposed SSH services attract automated probing. Even a newly started instance can collect failed login attempts within minutes or hours. Fail2Ban is effective because it reacts to behaviour, not just to open ports. It complements the firewall rather than replacing it.

Operationally, Fail2Ban suits internet facing systems that must allow password authentication, training environments, jump hosts and public test servers. It can also be tuned more tightly or more leniently depending on the risk profile and the likelihood of genuine user error.
## Working directly with nftables
`firewalld` is convenient, but RHEL also allows direct control of `nftables` with the `nft` command. `nftables` is the successor to `iptables`, `ip6tables`, `arptables`, `ebtables` and `ipset`. Its most important practical advantage here is that one framework can manage IPv4 and IPv6 together through the `inet` family.

Basic inspection commands include:

```bash
sudo nft list ruleset
sudo nft list tables
sudo nft list table inet filter
```

When `firewalld` is running, many tables and chains may already exist. Stopping `firewalld` stops the daemon, but existing `nftables` rules remain in the kernel until they are removed. A clean start therefore requires an explicit flush:

```bash
sudo systemctl disable --now firewalld
sudo nft flush ruleset
```

A direct `nftables` configuration begins with no default tables. Administrators define only the structures they need. That is a major difference from older expectations based on `iptables`.

A compact inbound only policy can use one table and one chain:

```bash
sudo nft add table inet filter
# create an input chain with type filter, hook input, priority 0 and policy accept
sudo nft add rule inet filter input iif lo accept
sudo nft add rule inet filter input ct state established,related accept
sudo nft add rule inet filter input tcp dport 22 accept
sudo nft add rule inet filter input counter drop
```

This ruleset does four things. It accepts loopback traffic. It allows packets that belong to established or related connections. It permits new inbound SSH traffic on TCP 22. It counts and drops everything else. Using `inet` means the same table can apply to IPv4 and IPv6.

The design also corrects a common misunderstanding. Chain names such as `input` are conventional, but the real function comes from the hook and chain type. If the host never forwards packets and does not filter outbound traffic, it does not need `forward` or `output` chains at all. Lean rulesets are easier to read and have lower overhead.

The example chooses a default chain policy of `accept` and ends with an explicit drop rule. That keeps remote administration safer during live editing because a temporary flush does not instantly lock the operator out. A stricter policy of `drop` is also valid, but it requires more care during staged changes.

This also explains why `firewalld` and direct `nftables` editing should not usually be mixed casually on the same host. `firewalld` expects to own the rule model it creates. Direct low level changes can be overwritten on reload, or they can leave a host with a policy that no longer matches the higher level zone definition. Administrators normally choose one control path for a given policy area and stay consistent.
## Persisting nftables rules
A direct `nftables` ruleset is still only runtime state until a service loads it from disk. On RHEL, `nftables.service` commonly reads `/etc/sysconfig/nftables.conf`. A persistent workflow therefore has three parts.

First, save a clean ruleset to the configuration file. Second, ensure the file begins by clearing the active ruleset so that reloads do not append a duplicate copy. Third, enable the service.

A typical start of file includes:

```nft
flush ruleset
```

After that line, the defined tables, chains and rules follow. Once the file is ready:

```bash
sudo systemctl enable --now nftables.service
```

The service then restores the rules at boot. A useful caution appears in the example. If the current ruleset is exported after traffic has already hit a `counter` rule, the saved file may include packet and byte counts. Reloading that file preserves those counts as the starting value, which can confuse later monitoring. Administrators who want fresh counters on each restart should remove those saved values from the file.

Direct `nftables` control makes sense when a host needs an exact low level ruleset, when `firewalld` abstractions do not fit the use case, or when the smallest possible firewall footprint matters. For common service based rules, `firewalld` is usually faster and safer. For specialised packet handling, direct `nft` work provides full control.
## Consolidated operational model
The material resolves into a clear administrative sequence.

- inspect and test live state with `ip`
- build isolated lab topologies with namespaces and veth pairs
- persist interface and DNS settings through NetworkManager and `nmcli`
- apply service and source based firewall policy with `firewalld`
- use Fail2Ban when allowed services attract repeated authentication abuse
- drop to direct `nftables` only when a host needs precise low level control

That sequence keeps temporary testing, persistent configuration and security policy separate without making them disconnected. `ip` answers "what is true right now". NetworkManager answers "what should return on boot". `firewalld` answers "what traffic should the host allow". `nftables` answers "how the kernel enforces that policy". Fail2Ban answers "how the host reacts when an allowed service is abused".

Taken together, these tools give RHEL a layered networking model that supports both fast troubleshooting and disciplined long term configuration.