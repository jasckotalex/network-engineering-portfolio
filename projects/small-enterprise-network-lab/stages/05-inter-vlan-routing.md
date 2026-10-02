# Stage 5 — Inter-VLAN Routing, DHCP, and DNS

## Objective

Enable controlled communication between VLAN 10 (USERS) and VLAN 20 (SERVERS) with router-on-a-stick, then add DHCP and DNS services after routed connectivity is verified.

## Topology and addressing

```text
PC1 — SW1 access VLAN 10                 PC3 — SW2 access VLAN 20
PC5 — SW1 access VLAN 20                 SW1 Gi0/3 ══ trunk ══ SW2 Gi0/0

SW1 Gi0/0 ══ trunk (VLANs 10,20) ══ R1 Gi0/0
R1 Gi0/1 ↔ R2 Gi0/1 over 192.168.100.0/30
R1 Gi0/2 ↔ PC2 on 192.168.20.0/24
R2 Gi0/0 ↔ PC4 on 192.168.30.0/24
```

| VLAN | Name | Network | R1 subinterface / gateway |
|---|---|---|---|
| 10 | USERS | 192.168.10.0/24 | Gi0/0.10 — 192.168.10.1/24 |
| 20 | SERVERS | 192.168.40.0/24 | Gi0/0.20 — 192.168.40.1/24 |

Current endpoint addressing:
- PC1 uses DHCP and received `192.168.10.21/24`, gateway `192.168.10.1`.
- PC3 uses DHCP and received `192.168.40.21/24`, gateway `192.168.40.1`.
- PC5 remains static at `192.168.40.10/24`, gateway `192.168.40.1`.

## Router-on-a-stick configuration

R1's physical Gi0/0 has no IP address. VLAN gateway addresses are on dot1Q subinterfaces:

```cisco
interface gigabitEthernet0/0
 no ip address
 no shutdown
!
interface gigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
!
interface gigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.40.1 255.255.255.0
```

SW1 Gi0/0, connected to R1, is an 802.1Q trunk allowing VLANs 10 and 20. SW1 Gi0/3 to SW2 Gi0/0 remains a trunk allowing VLANs 1, 10, and 20.

## DHCP configuration

R1 excludes .1 through .20 in each LAN so gateway addresses and existing static assignments remain reserved. It provides two DHCP pools:

```cisco
ip dhcp excluded-address 192.168.10.1 192.168.10.20
ip dhcp excluded-address 192.168.40.1 192.168.40.20
!
ip dhcp pool VLAN10-USERS
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
!
ip dhcp pool VLAN20-SERVERS
 network 192.168.40.0 255.255.255.0
 default-router 192.168.40.1
```

No DNS server option has been configured in the pools yet.

## Work completed

1. Inspected the current SW1 Gi0/0 access configuration and R1 Gi0/0 IP before making changes.
2. Saved the existing device configurations as a baseline.
3. Added R1 subinterfaces for VLAN 10 and VLAN 20, keeping the VLAN 10 gateway address unchanged.
4. Changed SW1 Gi0/0 from access VLAN 10 to an 802.1Q trunk carrying VLANs 10 and 20.
5. Set the VLAN 20 default gateway on PC3 and PC5.
6. Added a static route on R2 for the VLAN 20 network via R1 after troubleshooting showed the network was missing from R2's routing table:

```cisco
ip route 192.168.40.0 255.255.255.0 192.168.100.1
```

7. Configured DHCP pools for VLANs 10 and 20 on R1, with excluded address ranges for gateways and static hosts.

## Verification recorded

- R1 `show ip interface brief` showed Gi0/0.10 at `192.168.10.1` and Gi0/0.20 at `192.168.40.1`; both subinterfaces were `up/up`.
- SW1 `show interfaces trunk` showed Gi0/0 and Gi0/3 trunking with 802.1Q encapsulation. Gi0/0 carried VLANs 10 and 20; Gi0/3 carried VLANs 1, 10, and 20. The allowed VLANs were active and in STP forwarding state.
- The user confirmed that PC1, PC3, and PC5 could ping their respective R1 gateways.
- After configuring the VLAN 20 endpoints' default gateways, the user confirmed that cross-VLAN pings succeeded. This verifies inter-VLAN routing between VLANs 10 and 20.
- A ping from PC3 to PC4 (`192.168.30.10`) initially timed out. R2's `show ip route 192.168.40.0` reported that the network was not in the table. After adding the R2 static route for `192.168.40.0/24` via `192.168.100.1`, the user confirmed the ping from PC3 to PC4 succeeded.
- The user also confirmed that PC3 can ping PC2 at `192.168.20.10`.
- R1 `show ip dhcp pool` listed both pools with zero leases before clients requested addresses.
- PC1 received `192.168.10.21/24` via DHCP, with gateway `192.168.10.1`, and successfully pinged its gateway.
- PC3 received `192.168.40.21/24` via DHCP, with gateway `192.168.40.1`, and successfully pinged its gateway.
- R1 `show ip dhcp binding` confirmed automatic leases for `192.168.10.21` and `192.168.40.21`.

## Current status

Router-on-a-stick, tested routed reachability, and DHCP assignment in both VLANs are working. Stage 5 remains in progress: configure and verify DNS. No DNS service has been set up yet.

## Next work

Confirm that PC1 and PC3 startup configurations are saved with DHCP enabled. Then select a service host and configure DNS.
