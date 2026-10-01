# Stage 4 — VLANs and 802.1Q Trunk

## Objective

Create VLANs on two switches and carry them across one trunk while preserving the existing routed lab.

## Current topology

```text
PC1 ── SW1 Gi0/1       SW1 Gi0/3 ══ trunk ══ SW2 Gi0/0
         SW1 Gi0/0 ── R1 Gi0/0                SW2 Gi0/1 ── PC3

R1 Gi0/2 ── PC2
R1 Gi0/1 ── R2 Gi0/1
R2 Gi0/0 ── PC4
```

The EVE-NG lab currently uses:
- SW1 Gi0/3 to SW2 Gi0/0 as the trunk.
- SW1 Gi0/0 to R1, Gi0/1 to PC1, and Gi0/2 as an access port.
- SW2 Gi0/1 as the VLAN 10 access port to PC3.

## VLAN plan

| VLAN | Name | Purpose | Status |
|---|---|---|---|
| 1 | default | Native/legacy default VLAN; temporarily allowed on trunk to preserve existing behavior during migration | Active |
| 10 | USERS | Existing user LAN 192.168.10.0/24 | Active; PC1 and PC3 assigned |
| 20 | SERVERS | Reserved for a later endpoint and segmentation exercise | Created and allowed on trunk; no access ports assigned yet |

## Trunk configuration

SW1 Gi0/3 and SW2 Gi0/0 use 802.1Q trunking. Both are configured to allow VLANs 1, 10, and 20. The trunk output showed:
- Mode: on
- Encapsulation: 802.1Q
- Status: trunking
- Native VLAN: 1
- VLANs 1, 10, and 20 allowed and active
- VLANs 1, 10, and 20 in STP forwarding state

The switch image requires trunk encapsulation to be explicitly set to dot1q before `switchport mode trunk`; an initial trunk-mode attempt was rejected while encapsulation was Auto.

## Access ports

On SW1:
- Gi0/0, Gi0/1, and Gi0/2 were configured as access ports in VLAN 10.

On SW2:
- Gi0/1 was configured as an access port in VLAN 10 for PC3.
- SW2 reported VLAN 10 active with Gi0/1 as a member.

VLAN 20 exists on both switches but is not yet assigned to an access port.

## Verification recorded

- `show interfaces trunk` reports SW1 Gi0/3 and SW2 Gi0/0 trunking with 802.1Q and VLANs 1, 10, and 20 allowed.
- Both switches report VLANs 1, 10, and 20 in the STP forwarding state.
- PC1 successfully pinged PC3 at 192.168.10.20 after PC3 was connected through SW2 Gi0/1. Replies had TTL 64, consistent with both hosts communicating in the same subnet without routing.
- PC1 still successfully pinged PC2 at 192.168.20.10 (TTL 63) and PC4 at 192.168.30.10 (TTL 62), confirming the existing router paths remained operational after moving access ports to VLAN 10.
- SW2's `write memory` command completed with `[OK]`.

## Current status

VLAN 10 connectivity across the trunk and existing routed reachability are verified. Stage 4 is in progress: VLAN 20 still needs an access endpoint and a focused segmentation check. No inter-VLAN routing has been configured.

## Commands used

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
!
interface gigabitEthernet0/0
 switchport mode access
 switchport access vlan 10
```

Interface assignments differ by switch as shown above. Do not apply a trunk configuration to an endpoint access port.
