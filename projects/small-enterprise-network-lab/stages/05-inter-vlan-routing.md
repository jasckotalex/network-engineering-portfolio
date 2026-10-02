# Stage 5 — Inter-VLAN Routing, DHCP, and DNS

## Objective

Enable controlled communication between VLAN 10 (USERS) and VLAN 20 (SERVERS) with router-on-a-stick, then add DHCP and DNS services after routed connectivity is verified.

## Starting topology

```text
PC1 — SW1 access VLAN 10
PC5 — SW1 access VLAN 20
PC3 — SW2 access VLAN 20
SW1 Gi0/3 ══ 802.1Q trunk ══ SW2 Gi0/0

SW1 Gi0/0 — R1 Gi0/0 (currently routed as 192.168.10.1/24)
```

The existing transit and remote LAN remain connected:
- R1 Gi0/1 ↔ R2 Gi0/1 over 192.168.100.0/30.
- R1 Gi0/2 ↔ PC2 on 192.168.20.0/24.
- R2 Gi0/0 ↔ PC4 on 192.168.30.0/24.

## Proposed router-on-a-stick addressing

| VLAN | Name | Network | R1 subinterface / gateway |
|---|---|---|---|
| 10 | USERS | 192.168.10.0/24 | Gi0/0.10 — 192.168.10.1/24 |
| 20 | SERVERS | 192.168.40.0/24 | Gi0/0.20 — 192.168.40.1/24 |

The physical R1 Gi0/0 link and SW1 Gi0/0 link will need to become a trunk. Before making changes, inspect the current interface configuration and save device configurations. Preserve existing routing through R1 Gi0/1 and Gi0/2.

## Work sequence

1. Read-only inspection of SW1 Gi0/0 and R1 Gi0/0; verify their current state and the existing IP address.
2. Save current configurations and convert the SW1–R1 link to an 802.1Q trunk carrying VLANs 10 and 20.
3. Move the VLAN 10 gateway address from the physical R1 interface to subinterface Gi0/0.10 and create Gi0/0.20 as the VLAN 20 gateway.
4. Configure PC3 and PC5 with gateway 192.168.40.1. Verify each VLAN's gateway and inter-VLAN reachability.
5. Only after routing works, add DHCP and DNS services and verify address assignment and name resolution.

## Current status

Preparation only. No Stage 5 device configuration changes have been made yet. First collect read-only interface output from SW1 and R1, then confirm the active configuration before proceeding.

## Verification plan

- SW1 reports Gi0/0 as a trunk carrying VLANs 10 and 20.
- R1 reports both subinterfaces up/up with 802.1Q VLAN IDs 10 and 20.
- PC1 reaches 192.168.10.1; PC3 and PC5 reach 192.168.40.1.
- Hosts in VLAN 10 and VLAN 20 can communicate through R1.
- Existing reachability to PC2 and PC4 still works.
- DHCP leases and DNS lookups are verified in the later service portion of this stage.
