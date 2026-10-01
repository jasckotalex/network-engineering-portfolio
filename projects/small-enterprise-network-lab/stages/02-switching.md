# Stage 2 — Ethernet Switching and MAC Learning

## Objective

Add one Layer 2 switch and place two hosts in the same IPv4 subnet. Observe how frames are forwarded using learned MAC addresses.

## Topology

```text
PC1 ──┐
      SW1
PC2 ──┘
```

Both PCs connect to access ports on SW1. No router is required in this stage.

## Address plan

| Node | Example address |
|---|---|
| PC1 | 192.168.10.10/24 |
| PC2 | 192.168.10.20/24 |

Do not configure a default gateway for this same-subnet exercise.

## Tasks

1. Add SW1 and connect both VPCS nodes to separate switch ports.
2. Start the nodes and confirm they are in the same subnet.
3. Ping PC2 from PC1.
4. Inspect the switch MAC address table before and after generating traffic, using the command supported by the selected switch image (for Cisco IOS, `show mac address-table`).
5. Clear or wait out dynamic MAC learning, then ping again and observe relearning if the image supports it.

## Expected result

The hosts can communicate within the same subnet. SW1 learns each source MAC address on the port where the frame arrives and uses that table to forward later unicast frames. Initial ARP traffic may be broadcast.

## Troubleshooting

Check link state, host addresses and masks, VLAN membership, and learned MAC addresses. If the switch image does not support the expected IOS command, use the equivalent command for that image.

## Status

Planned. Run after Stage 1 is verified and record actual switch output and observations.
