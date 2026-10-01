# Stage 4 — VLANs and 802.1Q Trunk

## Objective

Create VLANs on two switches and carry them across one trunk while preserving the existing routed lab. Verify that hosts in the same VLAN can communicate across the trunk and that hosts in different VLANs remain separated until inter-VLAN routing is added.

## Current topology

```text
PC1 (VLAN 10) ── SW1 Gi0/1       SW1 Gi0/3 ══ trunk ══ SW2 Gi0/0 ── SW2 Gi0/1 ── PC3 (VLAN 20)
PC5 (VLAN 20) ── SW1 Gi0/2
                    SW1 Gi0/0 ── R1 Gi0/0

R1 Gi0/2 ── PC2
R1 Gi0/1 ── R2 Gi0/1
R2 Gi0/0 ── PC4
```

The lab uses SW1 Gi0/3 to SW2 Gi0/0 as the trunk. PC1 and PC5 connect to SW1; PC3 connects to SW2. SW1 Gi0/0 remains connected to R1 as an access port in VLAN 10.

## VLAN and endpoint plan

| VLAN | Name | Subnet | Endpoints | Status |
|---|---|---|---|---|
| 1 | default | — | Native VLAN on the trunk | Active |
| 10 | USERS | 192.168.10.0/24 | PC1: 192.168.10.10 | Active |
| 20 | SERVERS | 192.168.40.0/24 | PC5: 192.168.40.10; PC3: 192.168.40.20 | Active |

VLAN 10 and VLAN 20 are created on both switches. The trunk carries VLANs 1, 10, and 20. PC1's gateway remains 192.168.10.1. PC3 has no gateway configured, so it can communicate only within its directly connected subnet until routing is added.

## Trunk configuration

SW1 Gi0/3 and SW2 Gi0/0 use 802.1Q trunking. Both are configured to allow VLANs 1, 10, and 20. Verification showed:

- Mode: on
- Encapsulation: 802.1Q
- Status: trunking
- Native VLAN: 1
- VLANs 1, 10, and 20 allowed and active
- VLANs 1, 10, and 20 in STP forwarding state

The switch image requires trunk encapsulation to be explicitly set to dot1q before `switchport mode trunk`; an initial trunk-mode attempt was rejected while encapsulation was Auto.

## Access ports

On SW1:
- Gi0/0 — access VLAN 10, connected to R1 Gi0/0.
- Gi0/1 — access VLAN 10, connected to PC1.
- Gi0/2 — access VLAN 20, connected to PC5.

On SW2:
- Gi0/1 — access VLAN 20, connected to PC3.

## Verification recorded

- `show interfaces trunk` confirmed the SW1 Gi0/3 and SW2 Gi0/0 trunks, dot1q encapsulation, allowed VLANs 1, 10, and 20, and VLANs in the STP forwarding state.
- PC1 and PC3 successfully communicated when both were temporarily assigned to VLAN 10, confirming VLAN 10 could cross the trunk.
- After assigning PC3 and PC5 to VLAN 20 and configuring addresses in 192.168.40.0/24, PC3 successfully pinged PC5 at 192.168.40.10 (five replies, TTL 64). This confirms same-VLAN connectivity across the SW2–SW1 trunk.
- PC3's startup configuration was saved with `save`.
- PC3's ping to PC1 at 192.168.10.10 returned `No gateway found`. This is expected because PC3 has no default gateway; it confirms that inter-VLAN routing is not currently available, but by itself is not a test of VLAN filtering.
- Earlier checks showed existing routed reachability between PC1, PC2, and PC4 remained operational during the VLAN 10 setup.

## Result

Stage 4 is complete: VLAN 10 and VLAN 20 are configured on both switches, both VLANs are allowed across the trunk, access ports place endpoints into their intended VLANs, and the VLAN 20 endpoints communicate across the trunk. Inter-VLAN routing has not been configured and remains a later stage.

## Commands used

Example trunk configuration (apply the corresponding interface on each switch):

```cisco
vlan 10
 name USERS
vlan 20
 name SERVERS
!
interface gigabitEthernet0/3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 1,10,20
```

Example access port configuration:

```cisco
interface gigabitEthernet0/2
 switchport mode access
 switchport access vlan 20
```

Endpoint IP configuration on VPCS:

```text
PC5> ip 192.168.40.10/24
PC3> ip 192.168.40.20/24
PC3> save
```

Interface assignments differ by switch as shown above. Do not apply trunk configuration to an endpoint access port.
