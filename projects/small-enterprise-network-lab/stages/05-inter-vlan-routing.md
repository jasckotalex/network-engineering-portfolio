# Stage 5 — Inter-VLAN Routing, DHCP, and DNS

## Objective

Enable communication between VLAN 10 (USERS) and VLAN 20 (SERVERS) with router-on-a-stick, provide IPv4 settings through DHCP, and add an internal DNS service.

## Topology and addressing

```text
PC1 — SW1 access VLAN 10                 PC3 — SW2 access VLAN 20
LinuxClient — SW2 Gi0/3 access VLAN 10   PC5 — SW1 access VLAN 20
                                          Linux dns01 — SW2 Gi0/2 access VLAN 20
                                          SW1 Gi0/3 ══ trunk ══ SW2 Gi0/0

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
- LinuxClient uses DHCP and received `192.168.10.22/24`, gateway `192.168.10.1`, DNS `192.168.40.53`.
- PC3 uses DHCP and received `192.168.40.21/24`, gateway `192.168.40.1`.
- PC5 remains static at `192.168.40.10/24`, gateway `192.168.40.1`.
- Linux DNS server `dns01` uses static address `192.168.40.53/24`, gateway `192.168.40.1`.

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

SW1 Gi0/0, connected to R1, is an 802.1Q trunk allowing VLANs 10 and 20. SW1 Gi0/3 to SW2 Gi0/0 remains a trunk allowing VLANs 1, 10, and 20. SW2 Gi0/2 is an access port in VLAN 20 for the Linux DNS server; SW2 Gi0/3 is an access port in VLAN 10 for LinuxClient.

## DHCP configuration

R1 excludes .1 through .20 in each LAN, reserving gateway and static addresses. The DNS server address .53 is also excluded from DHCP. R1 provides two pools and gives clients the DNS server address:

```cisco
ip dhcp excluded-address 192.168.10.1 192.168.10.20
ip dhcp excluded-address 192.168.40.1 192.168.40.20
ip dhcp excluded-address 192.168.40.53
!
ip dhcp pool VLAN10-USERS
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 192.168.40.53
!
ip dhcp pool VLAN20-SERVERS
 network 192.168.40.0 255.255.255.0
 default-router 192.168.40.1
 dns-server 192.168.40.53
```

## DNS service

Ubuntu Server `dns01` runs BIND9. Netplan assigns `192.168.40.53/24` statically on `ens3`, with default route via `192.168.40.1`. Cloud-init network configuration is disabled so that it does not overwrite the manually managed Netplan configuration.

The forward zone `lab.test` is configured in `/etc/bind/named.conf.local` and stored in `/etc/bind/db.lab.test`. It contains these A records:

| Name | Address |
|---|---|
| dns01.lab.test | 192.168.40.53 |
| pc1.lab.test | 192.168.10.21 |
| pc3.lab.test | 192.168.40.21 |

Two reverse zones provide PTR lookups:

| Reverse zone | PTR records verified |
|---|---|
| `40.168.192.in-addr.arpa` | 192.168.40.53 → dns01.lab.test; 192.168.40.21 → pc3.lab.test |
| `10.168.192.in-addr.arpa` | 192.168.10.21 → pc1.lab.test |

R1 is configured as a DNS client with:

```cisco
ip name-server 192.168.40.53
```

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

7. Configured DHCP pools for VLANs 10 and 20 on R1, with excluded address ranges and DNS server option.
8. Installed Ubuntu Server and BIND9 on the Linux node; assigned the server static address `192.168.40.53/24`.
9. Configured forward DNS records for `dns01.lab.test`, `pc1.lab.test`, and `pc3.lab.test`.
10. Configured reverse DNS zones for VLAN 10 and VLAN 20, then verified PTR records.
11. Configured R1 to use the internal DNS server.
12. Added LinuxClient to SW2 Gi0/3 in access VLAN 10 and used it as a regular Linux DNS client.

## Verification recorded

- R1 `show ip interface brief` showed Gi0/0.10 at `192.168.10.1` and Gi0/0.20 at `192.168.40.1`; both subinterfaces were `up/up`.
- SW1 `show interfaces trunk` showed Gi0/0 and Gi0/3 trunking with 802.1Q encapsulation. Gi0/0 carried VLANs 10 and 20; Gi0/3 carried VLANs 1, 10, and 20. Allowed VLANs were active and in STP forwarding state.
- SW2 Gi0/3 was configured as an access port in VLAN 10 for LinuxClient; SW2 Gi0/2 remains an access port in VLAN 20 for dns01.
- The user confirmed that PC1, PC3, and PC5 could ping their respective R1 gateways.
- After configuring the VLAN 20 endpoints' default gateways, cross-VLAN pings succeeded.
- After adding the R2 static route for `192.168.40.0/24` via `192.168.100.1`, PC3 could ping PC4 at `192.168.30.10`; PC3 could also ping PC2 at `192.168.20.10`.
- R1 `show ip dhcp binding` confirmed automatic leases for PC1 `192.168.10.21` and PC3 `192.168.40.21`. LinuxClient received `192.168.10.22/24` through DHCP.
- PC1 and PC3 received DNS server `192.168.40.53` from DHCP. LinuxClient's `resolvectl status ens3` showed DNS server `192.168.40.53`.
- BIND9 was `active`; `named-checkzone` returned `OK` for the forward zone and both reverse zones, while `named-checkconf` returned no errors.
- `dig @192.168.40.53 dns01.lab.test`, `pc1.lab.test`, and `pc3.lab.test` returned the configured A records.
- Reverse lookups with `dig @192.168.40.53 -x` returned the expected PTR records for `192.168.40.53`, `192.168.40.21`, and `192.168.10.21`.
- R1 successfully resolved and pinged a DNS name using `ip name-server 192.168.40.53`.
- LinuxClient's `resolvectl query` resolved `dns01.lab.test` to `192.168.40.53` and `pc3.lab.test` to `192.168.40.21`. Pings by both names succeeded from LinuxClient, verifying DNS resolution and routed reachability from VLAN 10.
- PC1 VPCS still reports `Cannot resolve` for DNS names even though it receives DNS `192.168.40.53`, can ping the DNS server, and packet capture showed a query and reply. The same DNS records work from R1 and LinuxClient; the VPCS-specific failure is retained as a troubleshooting note and does not block the stage.

## Current status

Stage 5 is complete. Router-on-a-stick, inter-VLAN reachability, DHCP in both VLANs, BIND9 forward and reverse DNS, and DNS resolution from a regular Linux client are configured and verified. The VPCS client limitation is documented separately.

## Next work

Begin Stage 6: add a third router and configure OSPF, then verify neighbor formation and route exchange.
