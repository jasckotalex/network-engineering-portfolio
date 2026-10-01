# Stage 2 — Ethernet Switching and MAC Learning

## Objective

Add a Layer 2 switch to the routed lab. PC1 and PC3 share one subnet through SW1; PC2 remains in the other subnet behind R1. This demonstrates both local switching and routed traffic.

## Topology

```text
PC1 ── SW1 Gi0/1
          │
PC3 ── SW1 Gi0/2
          │ Gi0/0
          R1 Gi0/0 (192.168.10.1/24)
          R1 Gi0/2 (192.168.20.1/24) ── PC2
```

In the EVE-NG diagram, PC1 and PC3 connect to SW1; SW1 Gi0/0 connects to R1 Gi0/0; R1 Gi0/2 connects to PC2. The router interfaces were adjusted when the topology was changed.

## Address plan

| Node | Interface | IPv4 address | Default gateway |
|---|---|---|---|
| PC1 | Ethernet | 192.168.10.10/24 | 192.168.10.1 |
| PC3 | Ethernet | 192.168.10.20/24 | 192.168.10.1 |
| R1 | Gi0/0 | 192.168.10.1/24 | — |
| PC2 | Ethernet | 192.168.20.10/24 | 192.168.20.1 |
| R1 | Gi0/2 | 192.168.20.1/24 | — |

## PC3 configuration

```text
ip 192.168.10.20/24 192.168.10.1
save
```

## Verification and observed results

- PC3 pinged PC1 at 192.168.10.10 successfully. This verifies same-subnet connectivity through SW1.
- SW1's `show mac address-table` output showed two dynamically learned MAC addresses in VLAN 1 on Gi0/1 and Gi0/2.
- After the interface/topology correction, PC3 pinged its gateway 192.168.10.1 successfully.
- PC3 then pinged PC2 at 192.168.20.10 successfully. Replies had TTL 63, consistent with passing through R1 from the VPCS host's initial TTL.
- The earlier attempt to ping PC2 reported the gateway as unreachable; the subsequent gateway and PC2 pings succeeded after correcting the topology/interface configuration.

The MAC table maps a learned source MAC address to the switch port where its frame arrived. ARP broadcasts generated during the first ping also let the switch learn host MAC addresses.

## Commands

On SW1:

```cisco
show mac address-table
```

On PC3:

```text
show ip
ping 192.168.10.10
ping 192.168.10.1
ping 192.168.20.10
save
```

## Learning notes

- PC1 and PC3 share a subnet, so their traffic is switched locally and does not need routing by R1.
- Traffic from PC3 to PC2 is sent to the default gateway and routed by R1 between 192.168.10.0/24 and 192.168.20.0/24.
- The initial failed attempt was resolved by correcting the topology/interface configuration.

## Status

Same-subnet switching, dynamic MAC learning, gateway reachability, and end-to-end routed connectivity from PC3 to PC2 are verified from the supplied screenshots. PC3's configuration was saved.
