# Network Engineering Portfolio

Hands-on networking, Linux, cybersecurity, Cisco, and EVE-NG labs.

## Featured project: Small Enterprise Network Lab

A progressive EVE-NG project that builds a small company network from basic routed connectivity to VLAN segmentation, dynamic routing, monitoring, security, and automation.

The project is documented as it is built. Each stage records the goal, topology, configuration, verification, troubleshooting, and lessons learned.

### Current status

- [x] Stage 1 — Routed foundation: two LANs communicate through R1
- [x] Stage 2 — Switching: PC1 and PC3 communicate through SW1; dynamic MAC learning verified
- [x] Stage 3 — Static routing: PC4 reaches both existing LANs, and PC1 reaches PC4
- [~] Stage 4 — VLAN 10 trunk connectivity verified; VLAN 20 endpoint and segmentation check remain

### Lab roadmap

1. [Stage 1 — Routed foundation](projects/small-enterprise-network-lab/stages/01-routed-foundation.md)
2. [Stage 2 — Ethernet switching and MAC learning](projects/small-enterprise-network-lab/stages/02-switching.md)
3. [Stage 3 — Multiple subnets and static routing](projects/small-enterprise-network-lab/stages/03-static-routing.md)
4. [Stage 4 — VLANs and 802.1Q trunk](projects/small-enterprise-network-lab/stages/04-vlans-trunk.md)
5. Stage 5 — Inter-VLAN routing, DHCP, and DNS
6. Stage 6 — Multi-router routing and OSPF
7. Stage 7 — Linux-based monitoring and syslog
8. Stage 8 — ACLs, firewall policy, and IDS
9. Stage 9 — Redundancy and network automation

See the [project overview](projects/small-enterprise-network-lab/README.md) for design principles and the full roadmap.

## Repository structure

- `projects/` — larger, progressive lab projects
- `labs/` — focused standalone exercises
- `docs/` — shared notes and reference material

## Lab safety

All configurations are for an isolated EVE-NG lab. No production credentials, private keys, or sensitive configuration should be committed.
