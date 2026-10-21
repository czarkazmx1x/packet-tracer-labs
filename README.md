# Packet Tracer Labs

Hands-on Cisco Packet Tracer labs from my CompTIA Network+ prep. Each folder has the `.pkt` file (open it in Packet Tracer 9.0+) and screenshots of the results.

| Lab | What it covers | Result |
|-----|----------------|--------|
| [Lab 1: First LAN](lab01-first-lan/) | Switching, IPv4 addressing, ARP | PC-to-PC ping, 4/4 replies |
| [Lab 2: Two LANs with a Router](lab02-two-lans-router/) | Routing between subnets, router CLI, default gateways | Cross-subnet ping, 4/4 replies |

---

## Lab 1: First LAN

Two PCs on one switch, same subnet. No router, no gateway.

![Lab 1 topology](lab01-first-lan/screenshots/lab01_topology.jpg)

**Devices:** 1x Cisco 2960 switch, 2x PCs, copper straight-through cables

| Device | Interface | IPv4 | Mask | Gateway |
|--------|-----------|------|------|---------|
| PC1 | Fa0 | 192.168.10.10 | 255.255.255.0 | none |
| PC2 | Fa0 | 192.168.10.20 | 255.255.255.0 | none |

**Test:** ping 192.168.10.20 from PC1

![Lab 1 ping](lab01-first-lan/screenshots/lab01_ping.jpg)

**Notes**
- The switch forwards frames by MAC address, using its MAC address table.
- Before the first ping, PC1 sends an ARP broadcast to learn PC2's MAC. You need the MAC to build the Ethernet frame.
- No default gateway needed, because both hosts are on the same subnet.
- TTL=128: the packet never crossed a router.

---

## Lab 2: Two LANs with a Router

Two classrooms on different subnets, joined by a router.

![Lab 2 topology](lab02-two-lans-router/screenshots/lab02_topology.jpg)

**Topology:** PC1 – SW1 – R1 – SW2 – PC2
**Devices:** 1x Cisco 1941 router, 2x Cisco 2960 switches, 2x PCs

| Device | Interface | IPv4 | Mask | Gateway |
|--------|-----------|------|------|---------|
| R1 | G0/0 | 192.168.10.1 | 255.255.255.0 | n/a |
| R1 | G0/1 | 192.168.20.1 | 255.255.255.0 | n/a |
| PC1 | Fa0 | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| PC2 | Fa0 | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 |

**Router config**
```
enable
configure terminal
hostname R1
interface g0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown
interface g0/1
 ip address 192.168.20.1 255.255.255.0
 no shutdown
end
show ip interface brief
```

**Verification:** both router interfaces up/up

![show ip interface brief](lab02-two-lans-router/screenshots/lab02_show_ip_int_brief.jpg)

**Test:** ping 192.168.20.10 from PC1

![Lab 2 ping](lab02-two-lans-router/screenshots/lab02_ping.jpg)

**Notes**
- Router interfaces start administratively down. The links stayed red until `no shutdown`.
- PC1 sees that 192.168.20.10 is off-subnet, so it sends the packet to its default gateway (R1 G0/0). R1 routes it out G0/1 to PC2.
- The first ping lost one request while ARP resolved. The retry got 4/4.
- TTL=127 instead of 128: the router decremented it by 1, so the packet crossed one hop.

---

**Tools:** Cisco Packet Tracer 9.0.1
