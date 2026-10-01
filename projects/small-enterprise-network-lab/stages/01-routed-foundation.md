# Stage 1 — Routed Foundation

## Objective

Connect two hosts in different IPv4 subnets through one router. Practice interface configuration, addressing, connected routes, and basic verification.

## Topology

```text
PC1 ───────── R1 ───────── PC2
       Gi0/1       Gi0/2
```

Use separate EVE-NG links/network segments for the two sides. Do not connect both hosts to the same Layer 2 segment.

## Address plan

| Node | Interface | IPv4 address | Default gateway |
|---|---|---|---|
| PC1 | Ethernet | 192.168.10.10/24 | 192.168.10.1 |
| R1 | GigabitEthernet0/1 | 192.168.10.1/24 | — |
| R1 | GigabitEthernet0/2 | 192.168.20.1/24 | — |
| PC2 | Ethernet | 192.168.20.10/24 | 192.168.20.1 |

The two networks are 192.168.10.0/24 and 192.168.20.0/24. R1 has an interface in each, so it installs both as directly connected routes when the interfaces are up.

## R1 configuration

The screenshot from the initial lab session shows the following configuration being entered and saved:

```cisco
enable
configure terminal
interface gigabitEthernet0/1
 ip address 192.168.10.1 255.255.255.0
 no shutdown
exit
interface gigabitEthernet0/2
 ip address 192.168.20.1 255.255.255.0
 no shutdown
end
write memory
```

The screenshot shows both interfaces reporting `up` and `line protocol up`, and the save operation returning `[OK]`. The host configuration and end-to-end ping checks have not yet been recorded.

## Configure the hosts

For Cisco VPCS nodes:

```text
PC1> ip 192.168.10.10/24 192.168.10.1
PC2> ip 192.168.20.10/24 192.168.20.1
```

If your node uses a different host type, set the same address, prefix length, and gateway in its network settings.

## Verification checklist

On R1:

```cisco
show ip interface brief
show ip route
```

Expected: Gi0/1 and Gi0/2 are up/up, and the routing table contains connected routes for both /24 networks.

On each VPCS node:

```text
show ip
ping 192.168.10.1
ping 192.168.20.1
```

Then test across the router:

- From PC1, ping 192.168.20.10.
- From PC2, ping 192.168.10.10.

The first ping may time out while ARP resolves; repeat it once before treating that as a fault.

## Troubleshooting sequence

1. Check link state in EVE-NG and confirm each host is on the intended router interface.
2. Check `show ip interface brief` for address and up/up state.
3. Check each host's IP, prefix, and default gateway.
4. Ping the local router interface from each host.
5. Check `show ip route` for both connected /24 routes.
6. If local gateway pings work but cross-subnet pings fail, inspect the remote host address and gateway, then check for host firewall rules.

## Learning notes

Complete after running the lab:

- What information does a host use to decide that a destination is outside its local subnet?
- What does the host do with that packet?
- Why does R1 need no static route for either LAN?
- Record actual ping results and any issue diagnosed here.

## Status

Configuration of R1's two interfaces is documented from the screenshot. Host setup and end-to-end verification remain to be completed.
