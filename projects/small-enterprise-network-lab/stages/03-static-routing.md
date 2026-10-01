# Stage 3 — Multiple Subnets and Static Routing

## Objective

Connect a new LAN behind R2 and teach both routers static routes so hosts on each side can reach the remote networks.

## Topology

```text
PC1 ─┐
PC3 ─┴─ SW1 ── R1 ── PC2
               │ Gi0/1       Gi0/1 │
               └── 192.168.100.0/30 ──┘
                                      R2 Gi0/0 ── PC4
```

- R1 Gi0/0 serves LAN 192.168.10.0/24.
- R1 Gi0/2 serves LAN 192.168.20.0/24.
- R1 Gi0/1 connects to R2 Gi0/1 over 192.168.100.0/30.
- R2 Gi0/0 serves LAN 192.168.30.0/24.

## Address plan

| Segment | Device/interface | Address |
|---|---|---|
| Users LAN A | R1 Gi0/0 | 192.168.10.1/24 |
| Users LAN A | PC1 | 192.168.10.10/24, gateway 192.168.10.1 |
| Users LAN A | PC3 | 192.168.10.20/24, gateway 192.168.10.1 |
| Users LAN B | R1 Gi0/2 | 192.168.20.1/24 |
| Users LAN B | PC2 | 192.168.20.10/24, gateway 192.168.20.1 |
| Transit link | R1 Gi0/1 | 192.168.100.1/30 |
| Transit link | R2 Gi0/1 | 192.168.100.2/30 |
| Remote LAN | R2 Gi0/0 | 192.168.30.1/24 |
| Remote LAN | PC4 | 192.168.30.10/24, gateway 192.168.30.1 |

## Interface configuration

R1's transit interface:

```cisco
interface gigabitEthernet0/1
 ip address 192.168.100.1 255.255.255.252
 no shutdown
```

R2's transit interface and PC4 LAN interface:

```cisco
interface gigabitEthernet0/1
 ip address 192.168.100.2 255.255.255.252
 no shutdown
interface gigabitEthernet0/0
 ip address 192.168.30.1 255.255.255.0
 no shutdown
```

PC4:

```text
ip 192.168.30.10/24 192.168.30.1
save
```

## Static routes

Command format:

```text
ip route <destination-network> <destination-mask> <next-hop-IP>
```

On R1:

```cisco
ip route 192.168.30.0 255.255.255.0 192.168.100.2
```

This says that R1 should forward packets for 192.168.30.0/24 to R2's transit address, 192.168.100.2.

On R2:

```cisco
ip route 192.168.10.0 255.255.255.0 192.168.100.1
ip route 192.168.20.0 255.255.255.0 192.168.100.1
```

These routes provide the return paths through R1. A packet needs a usable route in both directions for a ping exchange to succeed.

## Verification recorded

- R1 Gi0/1 and R2 Gi0/1 were up/up.
- R2 pinged 192.168.100.1 with 5/5 replies.
- R1 pinged 192.168.100.2; the first attempt had 4/5 replies, then a repeat had 5/5 replies.
- PC4 pinged its gateway 192.168.30.1 with 5/5 replies.
- Routing-table screenshots show R1's static route to 192.168.30.0/24 via 192.168.100.2 and R2's static routes to 192.168.10.0/24 and 192.168.20.0/24 via 192.168.100.1.
- PC4 pinged PC1 (192.168.10.10) and PC2 (192.168.20.10) successfully, with five replies to each.
- PC1 pinged PC4 (192.168.30.10) successfully, with five replies.
- The supplied screenshots include an earlier “Destination host unreachable” response from PC4's gateway before the successful end-to-end tests.

Static routes are marked `S` in the Cisco routing table. Connected routes appear as `C`, and local interface addresses appear as `L`.

## Save configuration

The screenshots show `write memory` completing successfully on R2 and R1, with `[OK]`. PC4's `save` command also completed with `done`.

## Status

Static routing and end-to-end reachability between PC4 and both existing LANs are verified. Stage 3 is complete.
