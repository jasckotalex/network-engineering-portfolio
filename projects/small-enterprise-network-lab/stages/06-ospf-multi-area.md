# Stage 6 — OSPF, Route Selection, and Multi-Area Routing

## Objective

Add a third router, replace the earlier static routing design with OSPF, verify route selection and link-failure recovery, then extend the lab with a small multi-area OSPF design.

## Topology and addressing

```text
                    OSPF area 0

              192.168.100.0/30
        R1 --------------------- R2
         \                       /
          \                     /
 192.168.101.0/30       192.168.102.0/30
            \                 /
                    R3
                    |
                    | OSPF area 10
                    |
              PC6 LAN 192.168.50.0/24
```

Router links and LAN gateways:

| Device | Interface | Address | Purpose |
|---|---|---|---|
| R1 | Gi0/0.10 | 192.168.10.1/24 | VLAN 10 gateway |
| R1 | Gi0/0.20 | 192.168.40.1/24 | VLAN 20 gateway |
| R1 | Gi0/1 | 192.168.100.1/30 | Link to R2 |
| R1 | Gi0/2 | 192.168.20.1/24 | PC2 LAN gateway |
| R1 | Gi0/3 | 192.168.101.1/30 | Link to R3 |
| R2 | Gi0/0 | 192.168.30.1/24 | PC4 LAN gateway |
| R2 | Gi0/1 | 192.168.100.2/30 | Link to R1 |
| R2 | Gi0/2 | 192.168.102.1/30 | Link to R3 |
| R3 | Gi0/0 | 192.168.101.2/30 | Link to R1 |
| R3 | Gi0/1 | 192.168.102.2/30 | Link to R2 |
| R3 | Gi0/2 | 192.168.50.1/24 | PC6 LAN gateway |

Endpoint addressing confirmed during the stage:

| Device | Address | Gateway | Notes |
|---|---|---|---|
| PC1 | 192.168.10.21/24 | 192.168.10.1 | VLAN 10 host |
| PC3 | 192.168.40.21/24 | 192.168.40.1 | VLAN 20 host |
| PC4 | 192.168.30.10/24 | 192.168.30.1 | R2 LAN host |
| PC5 | 192.168.40.10/24 | 192.168.40.1 | VLAN 20 host |
| PC6 | 192.168.50.10/24 | 192.168.50.1 | R3 LAN host |
| dns01 | 192.168.40.53/24 | 192.168.40.1 | DNS server |
| LinuxClient | 192.168.10.22/24 | 192.168.10.1 | Linux client |

## OSPF design

| Router | Router ID | Areas | Role |
|---|---|---|---|
| R1 | 1.1.1.1 | Area 0 | Internal router |
| R2 | 2.2.2.2 | Area 0 | Internal router |
| R3 | 3.3.3.3 | Area 0, Area 10 | ABR |

The inter-router links stay in area 0. The PC6 LAN was moved to area 10 as an extra multi-area exercise, making R3 an Area Border Router.

## R1 OSPF configuration

```cisco
router ospf 1
 router-id 1.1.1.1
 passive-interface default
 no passive-interface GigabitEthernet0/1
 no passive-interface GigabitEthernet0/3
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0
 network 192.168.40.0 0.0.0.255 area 0
 network 192.168.100.0 0.0.0.3 area 0
 network 192.168.101.0 0.0.0.3 area 0
```

## R2 OSPF configuration

```cisco
router ospf 1
 router-id 2.2.2.2
 passive-interface default
 no passive-interface GigabitEthernet0/1
 no passive-interface GigabitEthernet0/2
 network 192.168.30.0 0.0.0.255 area 0
 network 192.168.100.0 0.0.0.3 area 0
 network 192.168.102.0 0.0.0.3 area 0
```

## R3 OSPF configuration

```cisco
router ospf 1
 router-id 3.3.3.3
 passive-interface default
 no passive-interface GigabitEthernet0/0
 no passive-interface GigabitEthernet0/1
 network 192.168.50.0 0.0.0.255 area 10
 network 192.168.101.0 0.0.0.3 area 0
 network 192.168.102.0 0.0.0.3 area 0
```

## Work completed

1. Added R3 and PC6 to the topology.
2. Built a triangle between R1, R2, and R3 using /30 point-to-point networks.
3. Assigned R3 LAN address `192.168.50.1/24` and PC6 address `192.168.50.10/24` with gateway `192.168.50.1`.
4. Configured OSPF process 1 on R1, R2, and R3.
5. Used explicit router IDs: `1.1.1.1`, `2.2.2.2`, and `3.3.3.3`.
6. Enabled `passive-interface default` and allowed OSPF hello packets only on router-to-router links.
7. Removed old static routes that were hiding OSPF routes because static routes have administrative distance 1, while OSPF has administrative distance 110.
8. Verified OSPF neighbors in FULL state.
9. Verified OSPF route selection in `show ip route ospf`.
10. Tested link failure by shutting down router links and confirming that traffic moved to the alternate path.
11. Moved `192.168.50.0/24` from area 0 to area 10.
12. Verified that R3 became an ABR and that R1/R2 learned the PC6 LAN as an inter-area route.
13. Saved the router configurations to startup configuration.

## Verification recorded

- R1 formed FULL OSPF adjacencies with R2 and R3.
- R2 formed FULL OSPF adjacencies with R1 and R3.
- R3 formed FULL OSPF adjacencies with R1 and R2.
- R1 learned `192.168.30.0/24` through R2 and `192.168.50.0/24` through R3.
- R2 learned `192.168.10.0/24`, `192.168.20.0/24`, and `192.168.40.0/24` through R1, and `192.168.50.0/24` through R3.
- R3 learned `192.168.10.0/24`, `192.168.20.0/24`, and `192.168.40.0/24` through R1, and `192.168.30.0/24` through R2.
- PC6 successfully pinged its gateway `192.168.50.1`.
- PC6 successfully pinged PC1 at `192.168.10.21`.
- PC6 successfully pinged PC4 at `192.168.30.10`.
- After the R1-R3 path was interrupted, R3 selected the alternate path through R2.
- After the R1-R2 path was interrupted, traffic still passed through the triangle using the alternate path.
- After moving PC6 LAN to area 10, R2 showed `O IA 192.168.50.0/24` via R3.
- R1 showed the route to `192.168.50.0/24` as OSPF inter-area, sourced from router ID `3.3.3.3`.
- `show startup-config | section router ospf` confirmed that OSPF configuration was saved on R1, R2, and R3.

## Learning notes

- `passive-interface default` keeps OSPF enabled for matching networks but stops hello packets on all OSPF-enabled interfaces.
- `no passive-interface` is applied only to links where OSPF neighbors should form.
- OSPF neighbors are discovered dynamically through hello packets on shared OSPF-enabled links.
- `O` in the routing table means an intra-area OSPF route.
- `O IA` means an inter-area OSPF route.
- A router becomes an ABR when it has active OSPF interfaces in more than one area.
- DR and BDR are elected per broadcast segment. They are not the main routers for the whole OSPF domain.
- Static routes can hide OSPF-learned routes because they have a lower administrative distance.
- The `from` router ID in detailed route output identifies the router that originated the OSPF information, while `via` identifies the next-hop address used to forward packets.

## Current status

Stage 6 is complete. The lab now demonstrates three-router OSPF, multiple LAN reachability, route selection, link failure recovery, and a small multi-area OSPF design with R3 acting as an ABR.

## Next work

Begin Stage 7: use a Linux monitoring node for syslog collection, availability checks, and SNMP or supported telemetry.
