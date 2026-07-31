# Managing Networking
> [!NOTE]
> A practical framework for managing and securing RHEL networking through live diagnostics, persistent configuration, network isolation, firewall policies, intrusion prevention, and low-level packet filtering.

Red Hat Enterprise Linux 8 separates runtime network state from persistent configuration. The `ip` utility changes the live kernel state, NetworkManager stores and activates connection profiles, and either `firewalld` or `nftables` enforces a host firewall. Administrators should test disruptive changes through a console or an out-of-band path because an address, route, profile, or firewall error can terminate a remote session.
## Runtime networking with `ip`
The `iproute2` suite replaces older tools such as `ifconfig`, `route`, and `arp`. Its `ip` command manages links, addresses, neighbours, routes, multicast settings, and network namespaces through a consistent object-and-action syntax.

Most `ip` objects and actions accept unambiguous abbreviations. For example, `ip a` displays addresses and `ip r` displays routes because `show` is the default action. Full object names remain safer in scripts and operational records. The `-4` and `-6` options select an address family, while `-brief` presents a compact status view:

```shell
ip -brief link
ip -brief address
ip -4 route show
ip -6 route show
```

Unprivileged users can inspect most state. Address, route, neighbour, link, and namespace changes require elevated privileges. Successful mutation commands often produce no output, so administrators should follow each change with a separate inspection or connectivity test.
### Addresses and routes
The following commands inspect IPv4 and IPv6 addresses, limit output to one interface, and add or remove a temporary IPv4 address:

```shell
ip address show
ip -4 address show dev eth1
ip -6 address show dev eth1
sudo ip address add 172.16.1.1/16 dev eth1
sudo ip address del 172.16.1.1/16 dev eth1
```

`ip address add` changes the running system. The address disappears when the interface resets or the host restarts unless NetworkManager also stores it in a connection profile. An interface can hold several addresses, so adding an address does not normally replace an existing one.

Each address includes a prefix length that defines its local subnet. `172.16.1.1/16`, for example, belongs to `172.16.0.0/16`. Omitting or misreading the prefix can create an unintended connected route and send traffic to the wrong interface. `ip -details address show` reveals lifetimes, scope, flags, and other properties when the normal display lacks enough information.

`ip route show` displays the main routing table. The kernel creates connected routes for directly attached networks, and NetworkManager or another service commonly supplies the default route. A static route must identify the correct destination prefix and either a reachable next hop or an output device:

```shell
ip route show
sudo ip route add 192.168.100.0/24 via 10.0.0.1 dev veth0
sudo ip route del 192.168.100.0/24
```

Private IPv4 networks can cross routers when administrators configure routes for them. Internet routers do not normally advertise these private prefixes, but the addresses remain routable inside controlled networks.

Route selection uses the most specific matching prefix. When two routes have the same prefix length, metrics and policy-routing rules can influence the result. `ip route get <destination>` asks the kernel which path it would use and reports the selected interface, source address, and next hop:

```shell
ip route get 192.168.100.1
```

This query often exposes errors faster than a complete routing-table listing. A runtime route follows the same persistence rule as a runtime address. NetworkManager must store any route required after reconnection or restart.
### Neighbour discovery
The neighbour table maps network-layer addresses to link-layer addresses on the local link. IPv4 uses the Address Resolution Protocol, while IPv6 uses the Neighbour Discovery Protocol. Remote internet hosts do not appear as local neighbours. The local host instead resolves the link-layer address of the next-hop router.

```shell
ip neighbour show
sudo ip neighbour del 192.168.33.12 dev eth1
```

Deleting an entry forces the kernel to resolve it again when traffic next needs that neighbour. States such as `REACHABLE`, `STALE`, `DELAY`, and `PROBE` describe the kernel's confidence in the cached information. A `STALE` entry can still be used while the kernel confirms reachability.

Neighbour entries apply only to directly reachable peers. A successful connection to `1.0.0.1`, for example, normally creates or refreshes an entry for the default gateway rather than the remote address. DNS traffic similarly refreshes the local resolver's entry only when that resolver shares the link. The table therefore helps distinguish local link resolution from end-to-end routing.

The `net.ipv4.neigh.default.gc_stale_time` sysctl defaults to 60 seconds on standard kernels. It governs when an unused neighbour entry becomes stale and eligible for garbage collection. It does not directly define the transition from `REACHABLE` to `STALE`. Neighbour Unreachability Detection uses other timers, including `base_reachable_time_ms`, for that decision.

Administrators can inspect the current values with `sysctl`:

```shell
sysctl net.ipv4.neigh.default.gc_stale_time
sysctl -a | grep gc_stale_time
```

A runtime change uses `sysctl -w`, while a file under `/etc/sysctl.d/` preserves an approved value across restarts. Changing a `default` value primarily affects interfaces created later. Existing interfaces can retain interface-specific values, so administrators should inspect both the default and per-interface keys.

An approved persistent override can use a file such as `/etc/sysctl.d/90-neighbour.conf`. `sysctl --system` loads all configured sysctl files, and a subsequent query confirms the effective value. Larger cache lifetimes reduce resolution traffic but can retain incorrect link-layer information longer. Smaller values increase probing and garbage-collection activity. Administrators should change defaults only for a measured operational reason.
### Isolated network namespaces
A network namespace provides an independent set of interfaces, addresses, routes, neighbour entries, and firewall state. Containers and virtual networking systems use namespaces to isolate network stacks. A virtual Ethernet pair connects two namespaces like a short cable.

The following sequence creates `net1`, places one end of a virtual Ethernet pair inside it, assigns addresses, and activates the links:

```shell
sudo ip netns add net1
sudo ip link add veth0 type veth peer name veth1
sudo ip link set veth1 netns net1

sudo ip address add 10.0.0.2/24 dev veth0
sudo ip link set veth0 up

sudo ip netns exec net1 ip link set lo up
sudo ip netns exec net1 ip address add 10.0.0.1/24 dev veth1
sudo ip netns exec net1 ip link set veth1 up
```

Commands run in the default namespace unless `ip netns exec` selects another one. The two endpoints can communicate after both links have addresses from the same subnet. The following commands add a second subnet inside `net1`, install the correct route in the default namespace, and test the path:

```shell
sudo ip netns exec net1 ip address add 192.168.100.1/24 dev veth1
sudo ip route add 192.168.100.0/24 via 10.0.0.1 dev veth0
ping -c 1 192.168.100.1
```

The route uses `192.168.100.0/24`, not `192.168.0.0/24`, and its next hop is the peer address `10.0.0.1`, not the local address `10.0.0.2`. Deleting `net1` also removes its namespace-side devices and the associated peer:

```shell
sudo ip netns del net1
```

`ip netns list` displays named namespaces, while `ip netns exec net1 ip address show` and `ip netns exec net1 ip route show` expose isolated state. Creating a namespace does not provide external connectivity. Administrators must deliberately add links, addresses, routes, forwarding, network address translation, or bridges according to the required topology. This explicit construction makes namespaces useful for testing routes and firewall rules without altering a physical network.
## Persistent configuration with NetworkManager
RHEL 8 uses NetworkManager by default. A device is a network interface, while a connection profile contains the settings that NetworkManager can apply to a device. Several profiles can target the same interface, but only a compatible active profile supplies its current configuration.

```shell
systemctl is-enabled NetworkManager
systemctl is-active NetworkManager
nmcli device status
nmcli connection show
```

RHEL 8 commonly stores NetworkManager profiles in `ifcfg` format under `/etc/sysconfig/network-scripts/`. It can also use keyfile profiles under `/etc/NetworkManager/system-connections/`. The separate legacy `network-scripts` implementation is deprecated. Administrators should manage normal RHEL 8 connections through NetworkManager rather than install the legacy service.

Traditional `ifcfg` profiles can contain keys such as `DEVICE`, `BOOTPROTO`, `IPADDR`, `PREFIX`, `DNS1`, and `ONBOOT`. `ONBOOT=yes` corresponds to automatic activation, while `BOOTPROTO=dhcp` requests dynamic IPv4 configuration. A profile with a static address records the address and prefix instead. Hand edits remain possible, but `nmcli` validates supported properties and keeps runtime and stored state easier to reconcile. If an administrator edits a profile file directly, `nmcli connection reload` makes NetworkManager reread stored profiles.
### Creating a static profile
`nmcli` writes persistent profile data and can activate it immediately. Shell completion exposes available objects and properties, which reduces errors in long commands.

```shell
sudo nmcli connection add \
  type ethernet \
  ifname eth1 \
  con-name cafe \
  ipv4.method manual \
  ipv4.addresses 192.168.100.1/24

sudo nmcli connection up cafe
nmcli connection show cafe
ip -4 address show dev eth1
```

Activating `cafe` can displace another profile on `eth1`. Administrators should confirm the profile name, interface, address, prefix, gateway, routes, and DNS settings before activation, especially over SSH. `connection.autoconnect` controls whether NetworkManager activates the profile during startup.

The profile can also carry a gateway and persistent routes:

```shell
sudo nmcli connection modify cafe ipv4.gateway 192.168.100.254
sudo nmcli connection modify cafe +ipv4.routes "198.51.100.0/24 192.168.100.254"
```

The leading `+` appends to a multi-value property. Omitting it replaces the existing route list. Administrators should use documentation prefixes only in examples and substitute addresses that fit the deployed network.

Profile changes remain separate from ad hoc `ip` changes. `nmcli connection modify` updates the stored profile, and reactivation applies it to the device. Administrators can inspect the effective runtime state with `ip`, while `nmcli connection show` reports the stored and active NetworkManager settings.

Runtime and stored settings can diverge after manual `ip` commands, direct file edits, DHCP renewal, or partial profile activation. A disciplined check compares three views: the connection profile, NetworkManager's active device data, and the kernel state. The following commands expose those layers without changing them:

```shell
nmcli connection show cafe
nmcli device show eth1
ip address show dev eth1
ip route show
```

An unexpected difference does not always indicate failure. DHCP leases, automatically generated routes, and IPv6 link-local addresses can appear only in active state. Administrators should identify which component owns each value before correcting it. Repeatedly forcing runtime values with `ip` can conceal a defective profile that fails again after the next restart.
### DNS selection
NetworkManager normally writes `/etc/resolv.conf`. Direct edits to that file can disappear when a connection changes. DNS servers, search domains, automatic DNS behaviour, and priority belong in connection profiles.

```shell
sudo nmcli connection modify cafe ipv4.dns "<dns-server>"
sudo nmcli connection modify cafe ipv4.ignore-auto-dns yes
sudo nmcli connection modify cafe ipv4.dns-priority 10
sudo nmcli connection up cafe
```

A lower numerical DNS priority has higher precedence. The default is normally 100 for non-VPN profiles and 50 for VPN profiles when no global override applies. A negative priority excludes DNS data from active profiles with greater numerical values. If several profiles use negative priorities, only those with the lowest numerical value contribute. This behaviour can prevent DNS leakage, but an incorrect value can also remove required internal resolvers.

Priority orders DNS data from different profiles. It does not reorder servers within one profile, so administrators should list those servers in the required order. `ipv4.ignore-auto-dns yes` rejects automatically supplied IPv4 DNS data for that profile.

DNS changes can affect active sessions even when interface addressing stays intact. `nmcli device show` reports the DNS data that NetworkManager applied, while `/etc/resolv.conf` shows what a traditional resolver can consume. Systems configured with a local caching or split-DNS plugin can route queries by domain instead of relying solely on file order.
## Host firewalls with `firewalld`
Host firewalls complement network perimeter controls. They limit exposure on each server, reduce lateral movement, and enforce service-specific access close to the workload. RHEL 8 uses the `nftables` kernel framework, and `firewalld` provides the usual high-level management interface.

Zones classify traffic by trust. NetworkManager connections or interfaces can select a zone, and source prefixes can direct matching traffic to one. Services group the ports and protocols required by an application. Ports that a zone does not explicitly allow remain blocked unless the zone target permits them.

The default zone handles traffic that has no more specific assignment. An active zone normally has at least one interface or source. A zone definition can also include rich rules, port forwards, masquerading, ICMP controls, and protocols. Administrators should keep zone assignments aligned with connection profiles so that a portable host does not retain office access rules on an untrusted network.

```shell
sudo firewall-cmd --state
sudo firewall-cmd --get-default-zone
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --list-all
sudo firewall-cmd --list-all --permanent
```

The default zone often starts as `public`, but administrators must inspect the host rather than assume a value. Vendor service and zone definitions reside under `/usr/lib/firewalld/`. Local definitions and overrides reside under `/etc/firewalld/`, which takes precedence. Package updates can replace vendor files, so local changes belong under `/etc`.
### Runtime and permanent rules
`firewalld` maintains separate runtime and permanent configurations. Runtime changes take effect immediately and disappear after a reload, restart, or reboot. Permanent changes become active after a reload or restart. This separation supports a test-first workflow:

```shell
sudo dnf install httpd
sudo systemctl enable --now httpd
sudo firewall-cmd --add-service=http
sudo firewall-cmd --list-services
sudo firewall-cmd --runtime-to-permanent
```

The `http` service opens the ports defined by its service object, normally TCP port 80. A successful local request does not prove that a remote client can pass through the firewall because loopback traffic follows a different path. Administrators should test from an authorised remote host before copying runtime state to permanent configuration.

`--runtime-to-permanent` copies the complete runtime configuration, including unrelated changes made during the same period. Administrators should compare runtime and permanent state before invoking it. A plain `firewall-cmd --reload` performs the opposite transition by rebuilding runtime state from permanent files, which discards uncommitted experiments.

A timeout creates a temporary runtime rule:

```shell
sudo firewall-cmd --add-port=443/tcp --timeout=10m
```

The rule expires automatically and cannot form part of permanent configuration. Timeouts reduce the consequences of some remote tests, although they do not replace console access and a rollback plan.
### Zones, interfaces, and sources
Adding a service to an interface's zone can expose it to every source that reaches that interface. A source-based zone narrows access to an address or prefix:

```shell
sudo firewall-cmd --zone=internal --add-service=http
sudo firewall-cmd --zone=internal --add-source=192.168.33.0/24
sudo firewall-cmd --zone=internal --list-all
sudo firewall-cmd --runtime-to-permanent
```

Traffic from `192.168.33.0/24` now enters the `internal` zone and can use its HTTP service. Source addresses do not authenticate clients, and upstream address translation or spoofing can weaken this control. Administrators should combine source restrictions with application authentication, encrypted transport, and suitable network filtering.

When source ranges overlap across zones, firewalld cannot apply both zone rule sets to one packet. Administrators should avoid ambiguous ranges and verify classification with `--get-active-zones`, `--list-sources`, and remote tests. Assigning a NetworkManager profile to a zone with `nmcli connection modify <profile> connection.zone <zone>` keeps network activation and firewall classification together.

A dedicated custom service keeps related ports under a clear name without changing the meaning of a vendor service:

```shell
sudo firewall-cmd --permanent --new-service=web
sudo firewall-cmd --permanent --service=web --add-port=80/tcp
sudo firewall-cmd --permanent --service=web --add-port=443/tcp
sudo firewall-cmd --reload
sudo firewall-cmd --zone=internal --add-service=web
sudo firewall-cmd --runtime-to-permanent
```

This approach creates a local service definition under `/etc/firewalld/services/`. It avoids redefining the built-in `http` service to include HTTPS, which could surprise other administrators and applications.
### Fail2Ban for repeated authentication failures
Fail2Ban reads authentication events, matches configured failure patterns, and invokes a firewall action to ban offending sources. It supplements strong SSH controls. Public servers should prefer key-based authentication, disable direct root login, restrict administrative sources where possible, and apply rate controls. An IP ban alone does not establish identity, and shared or translated addresses can produce false positives.

On RHEL 8, Fail2Ban is available from Extra Packages for Enterprise Linux. Administrators must enable the appropriate EPEL repository and install the packages required for the chosen logging and firewall backends. A local jail file can enable SSH monitoring through the systemd journal:

```ini
[DEFAULT]
bantime = 259200
findtime = 600
maxretry = 3

[sshd]
enabled = true
backend = systemd
```

The example bans a source for 72 hours after three failures within 10 minutes. Production values should reflect user behaviour, exposure, address sharing, and support procedures. After saving the configuration under `/etc/fail2ban/jail.d/`, administrators can start the service and inspect the jail:

```shell
sudo systemctl enable --now fail2ban
sudo fail2ban-client status sshd
sudo fail2ban-client set sshd unbanip <ip-address>
```

Logs and the client status should confirm that the filter sees genuine SSH failures and that the selected firewall action creates and removes bans. A configuration that starts successfully can still watch the wrong log source.

Fail2Ban counts events that its filter recognises, not every unsuccessful interaction with a service. Administrators should test sample journal entries against the installed filter, check the configured action, and confirm that the firewall contains the resulting ban. Trusted management addresses can be allowlisted through `ignoreip`, but broad exceptions can hide hostile traffic. Ban records also require monitoring so support staff can distinguish an attack from a locked-out legitimate user.
## Direct firewall control with `nftables`
`nftables` provides the kernel packet-filtering framework that supersedes the legacy `iptables` command family. The `nft` utility creates, inspects, and updates rules. An `inet` table can process both IPv4 and IPv6, which removes the need to duplicate many rules across separate command families.

Linux introduced nftables with kernel 3.13. RHEL 8 uses it as the default firewall backend, although compatibility commands can still translate legacy iptables rules into the `nf_tables` API. New native designs should use `nft` syntax. Available table families include `ip`, `ip6`, `inet`, `arp`, `bridge`, and `netdev`. The selected family controls which packet types and hooks the table can process.

Administrators should run only one firewall manager on a RHEL host. `firewalld`, the `nftables` service, and legacy `iptables` services can interfere with one another when they control the same hooks. Direct `nftables` management therefore requires stopping and disabling `firewalld`:

```shell
sudo systemctl disable --now firewalld
sudo nft list ruleset
```

Stopping `firewalld` does not guarantee that every rule has disappeared. Administrators should inspect the ruleset before replacing it. `nft flush ruleset` removes all tables and rules, so remote execution can expose the host or interrupt access when another command fails. A console, a tested rules file, and an atomic load reduce that risk.

Basic inspection commands separate structure from detail:

```shell
sudo nft list tables
sudo nft list ruleset
sudo nft --handle list ruleset
```

Handles identify rules for targeted deletion. They can change after a ruleset reload, so scripts should not assume that a previously observed handle remains stable.
### Tables, chains, and rules
A table groups chains, sets, maps, counters, and other objects within one address family. A regular chain organises rules. A base chain connects to a packet-processing hook such as `input`, `forward`, `output`, `prerouting`, or `postrouting`. Its type, hook, priority, and policy define how it participates in the network stack.

Table and chain names carry no built-in behaviour. A chain called `input` handles inbound packets only when its base-chain hook is `input`. Hosts that do not route packets may not need a `forward` base chain, and hosts that allow all outbound traffic may not need an `output` base chain.

Rules run in order. Match expressions examine packet metadata, addresses, protocols, ports, interfaces, connection state, and other fields. Statements then count, log, modify, accept, reject, or drop matching packets. `accept`, `drop`, and `reject` are terminal verdicts for the relevant processing path. A counter can share a rule with a verdict, so `counter drop` records and blocks the same packets without a second rule.

The following file defines a compact host firewall. It accepts established traffic, loopback traffic, essential ICMP and ICMPv6 control traffic, and SSH from an authorised subnet. Its default policy drops unmatched input:

```nft
flush ruleset

table inet filter {
  chain input {
    type filter hook input priority filter
    policy drop

    ct state invalid drop
    ct state established,related accept
    iifname "lo" accept

    ip protocol icmp accept
    meta l4proto ipv6-icmp accept

    ip saddr 192.168.33.0/24 tcp dport 22 accept
    counter
  }
}
```

Dropping every ICMP packet does not automatically improve security. IPv6 depends on ICMPv6 for core functions, including neighbour discovery and path maximum transmission unit handling. IPv4 also uses ICMP for diagnostics and network error reporting. Production rules can narrow accepted ICMP types, but they must preserve the messages required by the network design.

The `counter` statement records packets that reach the end of the chain. The chain policy then drops them. `nft list ruleset` displays packet and byte counts, which helps validate rule order and diagnose blocked traffic.
### Validation and persistence
Administrators should store custom scripts under `/etc/nftables/`, validate them without applying them, and then load them atomically:

```shell
sudo nft --check --file /etc/nftables/filter.nft
sudo nft --file /etc/nftables/filter.nft
sudo nft list ruleset
```

`nft --check` verifies syntax and rule construction without installing the candidate ruleset. It cannot prove that the policy allows every required workflow or blocks every prohibited path. Administrators should review terminal verdicts, confirm the order of overlapping matches, and test from networks represented by each rule. An early broad `accept` can bypass a later restriction, while an early broad `drop` can make later permits unreachable.

Atomic file loading protects the kernel from a partly applied ruleset when parsing fails. It does not protect an administrator from a valid but incorrect policy. Remote changes therefore need a recovery path, an explicit management-access rule, and a post-load connectivity check. Rate-limited logging can assist diagnosis, but unrestricted logging of denied traffic can exhaust storage or obscure useful events during a scan.

The rules file should begin with `flush ruleset` when it owns the complete firewall. Without that instruction, repeated loads can retain or duplicate earlier objects. A partial rules file should target only the objects it owns instead of clearing unrelated rules.

The RHEL 8 `nftables` service reads include statements from `/etc/sysconfig/nftables.conf`. The following line loads the custom file during service startup:

```nft
include "/etc/nftables/filter.nft"
```

After adding the include, administrators can enable the service:

```shell
sudo systemctl enable --now nftables
sudo systemctl reload nftables
sudo nft list ruleset
```

Saving the output of `nft list ruleset` can also capture live counter values. Loading that output restores those non-zero starting values. `nft --stateless list ruleset` or manual removal of counter values produces a cleaner persistent definition.

A final verification should test every allowed service from an authorised client, confirm that prohibited traffic fails, inspect counters, and repeat the test after a service reload or host restart. The smallest useful ruleset is easier to audit, but it must still support IPv4, IPv6, management access, application traffic, and required control protocols.