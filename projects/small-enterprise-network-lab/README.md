# Small Enterprise Network Lab

## Goal

Build and document a small company network in EVE-NG, starting with basic IPv4 routing and growing toward the skills repeatedly seen in junior network engineering roles: switching, VLANs, routing, troubleshooting, Linux services, monitoring, and security.

This is a learning project. Each stage adds a small number of concepts and includes observable checks so that failures can be diagnosed rather than hidden.

## Design principles

- Begin with the smallest topology that demonstrates the concept.
- Keep an address plan and topology diagram with every stage.
- Save device configurations and verification evidence.
- Introduce faults deliberately after normal connectivity works.
- Use device images available to the learner; exact vendors and commands may vary.
- Keep the lab isolated from production networks and omit secrets from version control.

## Progressive topology roadmap

| Stage | Topology focus | Concepts |
|---|---|---|
| 1 | Two hosts routed by R1 | IPv4, masks, gateways, interfaces, connected routes, ping |
| 2 | Hosts connected through a switch | Ethernet frames, MAC learning, broadcast domains |
| 3 | Two or more IP subnets | Static routes, next hops, routing tables, traceroute |
| 4 | Two switches with VLANs and a trunk | VLAN membership, access ports, 802.1Q |
| 5 | Router-on-a-stick and service host | Inter-VLAN routing, DHCP, DNS |
| 6 | Three routers and multiple LANs | OSPF neighbors, route selection, link failure, multi-area OSPF |
| 7 | Linux monitoring node | Syslog, SNMP or supported telemetry, availability checks |
| 8 | Segmented network edge | ACLs, firewall policy, VPN concepts, IDS/Suricata |
| 9 | Redundant links and repeatable configuration | STP, LACP where supported, Python/Ansible |

### Stage 1 diagram

```text
PC1 (192.168.10.10/24) --- R1 --- (192.168.20.10/24) PC2
                              |
                     routes between the
                      directly connected LANs
```

The router ports connect to separate host segments. In EVE-NG, each Ethernet segment can be a direct link to a VPCS node or an isolated network object. The two LANs must remain separate.

See [Stage 1 — Routed foundation](stages/01-routed-foundation.md) for the exact address plan and current R1 configuration.

## Current completed stages

- [Stage 1 — Routed foundation](stages/01-routed-foundation.md)
- [Stage 2 — Switching](stages/02-switching.md)
- [Stage 3 — Static routing](stages/03-static-routing.md)
- [Stage 4 — VLANs and trunk](stages/04-vlans-trunk.md)
- [Stage 5 — Inter-VLAN routing, DHCP, and DNS](stages/05-inter-vlan-routing.md)
- [Stage 6 — OSPF, route selection, and multi-area routing](stages/06-ospf-multi-area.md)

## Completion criteria

A stage is considered complete when its configuration is saved, the intended traffic passes, relevant command output is recorded, and at least one likely fault has been diagnosed. The project status is updated only after those checks are performed in EVE-NG.

## Next work

Begin Stage 7: add a Linux monitoring node for syslog, availability checks, and SNMP or supported telemetry.
