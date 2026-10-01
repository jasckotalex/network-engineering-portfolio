# Stage 2 — Ethernet Switching and MAC Learning

## Objective

Add a Layer 2 switch to the working routed lab. Put PC1 and a new PC3 in the same subnet, then observe how the switch learns MAC addresses. PC2 remains on the other subnet so the original routed path stays in place.

## Topology

```text
PC1 ──┐
      ├── SW1 ─── R1 Gi0/1 (192.168.10.1)
PC3 ──┘                         R1 Gi0/2 ─── PC2
                                      192.168.20.0/24
```

The switch ports shown in the MAC table are Gi0/1 and Gi0/2. Confirm in EVE-NG which host is connected to each port before interpreting the port mapping.

## Address plan

| Node | Interface | IPv4 address | Default gateway |
|---|---|---|---|
| PC1 | Ethernet | 192.168.10.10/24 | 192.168.10.1 |
| PC3 | Ethernet | 192.168.10.20/24 | 192.168.10.1 |
| R1 | Gi0/1 | 192.168.10.1/24 | — |
| PC2 | Ethernet | 192.168.20.10/24 | 192.168.20.1 |

## PC3 configuration

```text
ip 192.168.10.20/24 192.168.10.1
save
```

## Verification and observed result

PC3 successfully pinged PC1 at 192.168.10.10. This confirms same-subnet connectivity through SW1.

The IOSv switch output from `show mac address-table` showed two dynamically learned entries in VLAN 1:

| MAC address | Port |
|---|---|
| 0050.7966.6801 | Gi0/1 |
| 0050.7966.6805 | Gi0/2 |

The MAC table associates a source MAC with the switch port where its frame was learned. The first ping also causes ARP broadcast traffic, allowing the switch to learn source MAC addresses as frames pass.

## Commands

On SW1:

```cisco
show mac address-table
```

On PC3:

```text
show ip
ping 192.168.10.10
save
```

## Learning notes

- PC1 and PC3 share the same subnet, so their traffic is switched locally and does not need to be routed by R1.
- R1 remains the default gateway for traffic leaving 192.168.10.0/24, including traffic to PC2.
- The switch learned two dynamic MAC entries after traffic was generated.

## Status

Same-subnet ping and dynamic MAC learning are verified from the supplied EVE-NG screenshots. PC3's `save` command reported `done`.
