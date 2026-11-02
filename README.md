# Segmented Network Lab: SonicWall + Cisco Switching

A small on-prem lab that separates users and servers into firewall zones, so a compromised workstation can't freely reach the servers.

> **Status:** retired. The lab has been torn down; this write-up documents the design and the reasoning behind it.

## Goal

Limit lateral movement. On a flat network, one infected laptop can reach every server. Segmenting the network means a compromised device can only reach what is explicitly allowed.

**Anchor:** Zones separate. Default denies. Rules connect.

## Design

| Component | Role |
|---|---|
| Cisco switch | Layer 2 only: access ports for endpoints, trunks to the SonicWall and the ESXi host |
| SonicWall | Handles all inter-VLAN routing using VLAN subinterfaces |
| VLAN 4 (Users zone) | User devices, the most likely to get phished or infected |
| VLAN 5 (Servers zone) | Domain controller, RD Gateway, and other VMs on ESXi |

```
 User PCs ──(access, VLAN 4)──┐
                              ├── Cisco switch ══(trunk)══ SonicWall
 ESXi host ══(trunk, VLAN 5)──┘     (Layer 2 only)        (routes + filters)
```

## How it works

1. **Zones separate.** Each VLAN sits in its own SonicWall zone, so any traffic between VLANs is inter-zone and has to pass the firewall's rules.
2. **Default denies.** Auto-generated zone rules were turned off. With no allow rules, the implicit deny blocks all inter-zone traffic.
3. **Rules connect.** Access rules were written by hand to allow only the specific services each side needed.
4. **No shortcut around the firewall.** The switch does no routing, so every inter-VLAN packet goes through the SonicWall.

## Result

A compromised machine in the user VLAN can reach only the services explicitly allowed, not the rest of the server zone or the hypervisor management.

## Skills demonstrated

Network segmentation · Firewall zones and access rules · Least privilege · VLANs, access and trunk ports · Inter-VLAN routing
