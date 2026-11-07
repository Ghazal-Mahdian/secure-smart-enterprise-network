# Architecture

## Requirements (summarized from the assignment)

- **Two sites** (A: Los Angeles HQ, B: Phoenix branch), each with an ISP connection, an edge router with a public IP, and an ASA firewall.
- **VLAN zones per site.** A: CORP 10, GUEST 20, IOT 30, MGMT 99 (192.168.99.0/24). B: CORP 40, GUEST 50, IOT 60, MGMT 99 (192.168.199.0/24).
- **Wireless.** CorpWiFi → CORP and GuestWiFi → GUEST with WPA2 at each site.
- **IoT.** A smart camera, motion sensor, and smart light per site, isolated in the IOT VLAN.
- **Policy.** Allow CORP → Internet, CORP ↔ CORP over VPN, MGMT → all, and CORP → IOT for monitoring. Deny GUEST → CORP, GUEST → IOT, IOT → Internet, and IOT → CORP. Default deny.
- **IPsec VPN.** 192.168.10.0 ↔ 192.168.40.0 using an ISAKMP policy, pre-shared key, crypto map, and interesting-traffic ACL.
- **Optional expansion.** A third site, DHCP per VLAN, syslog/monitoring, VPN restricted to specific VLANs, and simulated attack traffic.
- **Investigation.** A guest user reached an IoT camera at the branch; identify the failure point, fix it, and propose improvements.

## Built design

| Element | Site A | Site B | Site C (expansion) |
|---|---|---|---|
| Edge router | Router 1 (2911) | Router 2 (2911) | Router 3 – edge router (2911) |
| "Firewall" | FW-SiteA (1841 router) | FW-SiteB (1841 router) | FW-SiteC (1841 router) |
| Switch | Multilayer Switch 1 / SW-SiteA (3650-24PS) | Multilayer Switch 2 / SW-SiteB (3650-24PS) | SW-SiteC (3650-24PS) |
| CORP | VLAN 10 – 192.168.10.0/24 | VLAN 40 – 192.168.40.0/24 | VLAN 70 – 192.168.70.0/24 |
| GUEST | VLAN 20 – 192.168.20.0/24 | VLAN 50 – 192.168.50.0/24 (per assignment) | VLAN 80 – 192.168.80.0/24 |
| IOT | VLAN 30 – 192.168.30.0/24 | VLAN 60 – 192.168.60.0/24 | VLAN 90 – 192.168.90.0/24 |
| MGMT | VLAN 99 – 192.168.99.0/24 | VLAN 99 (no ports assigned) | VLAN 99 – SVI 192.168.199.1 |
| Wireless | Access Point 1, Access Point 2 | Access Point 3, Access Point 4 | AP-CorpC, AP-GuestC |
| IoT devices | Webcam IoT-1, Smart LED IoT0, RFID Reader IoT1 | Webcam IoT 4, Smart LED IoT2, RFID Reader IoT3 | Smart LED IoT5, IoT6 |
| Other hosts | PC 1, PC 2, Corp Laptop0, Guest Laptop2, Syslog-Server | PC3, PC4, Laptop1 | PC 5, PC 6, Laptop3 |

Addresses come from VLAN tables, `show ip route`, SVI output, and host IP settings in the screenshots. The Site B GUEST subnet and the Site B MGMT addressing are not shown and are listed as specified in the assignment.

### Unexplained item

VLAN 11 (`VLAN0011`) exists on SW-SiteA and SW-SiteC with port Gig1/0/24 assigned. Its purpose is not documented. It may be the uplink to the firewall, since FW-SiteA reaches the switch over 10.10.10.0/30.

### Addressing conflict

The assignment sets Site B MGMT to 192.168.199.0/24. SW-SiteC uses 192.168.199.1 on Vlan99, so if Site B followed the assignment, Sites B and C would share a MGMT subnet. The AI-generated illustration shows the same overlap. Site B's MGMT addressing was not captured, so the conflict cannot be confirmed or ruled out.

## Routed path (Site A, observed)

```
Hosts ── SW-SiteA SVIs ── 10.10.10.0/30 ── FW-SiteA ── 10.0.0.0/30 ── Router 1 ── 203.0.113.0/30 ── ISP Router
 (VLAN 10/20/30/99)      (.2 switch, .1 fw)  Fa0/1    Fa0/0 (.2)   (.1) G0/1   G0/0 (.1)       (.2)  G0/0
                                                                                                    │ G0/1 203.0.113.4/30 → Site B
                                                                                                    │ G0/2 203.0.113.8/30 → Site C
                                                                                                    └ Loopback0 8.8.8.8/32 ("Internet")
```

- FW-SiteA: connected 10.0.0.0/30 (Fa0/0) and 10.10.10.0/30 (Fa0/1); static routes for 192.168.10/20/30.0 via 10.10.10.2; default via 10.0.0.1.
- Router 1: connected 10.0.0.0/30 (G0/1) and 203.0.113.0/30 (G0/0); static routes for the Site A VLANs and 10.10.10.0/30 via 10.0.0.2; 10.0.1.0/30 and default via 203.0.113.2.
- ISP Router: static routes to the firewall transit networks (10.0.0.0, 10.0.1.0, 10.0.2.0 /30) and to the CORP subnets 192.168.10.0, 192.168.40.0, and 192.168.70.0 via each site's edge router.

The firewalls' VPN peer addresses (10.0.0.2, 10.0.1.2, 10.0.2.2) are private and reachable only because the ISP router holds routes to them. The edge routers carry the 203.0.113.x addresses.

## Security policy: intended vs. evidenced

| Policy (assignment) | Mechanism seen | Evidence |
|---|---|---|
| CORP → Internet allowed | INSIDE-OUT 20 `permit ip 192.168.10.0/24 any`; ping 8.8.8.8 succeeds | ACL still; `pc1-ping-iot-vpn-internet.png` |
| CORP ↔ CORP via VPN | VPN-TRAFFIC 10→40 and 10→70; SAs with encaps/decaps | recording stills |
| MGMT → all | Standard ACL 99 permits 192.168.99.0/24; where it is applied is not shown | ACL still |
| CORP → IOT (monitoring) | INSIDE-OUT 10 `permit 10→30`; PC1 → 192.168.30.11 succeeds | ACL still; PC1 ping |
| GUEST → CORP denied | INSIDE-OUT 50 `deny 20→10`, placed after 30 `permit 20 any` | ACL still; Guest → 192.168.10.10 **succeeds** |
| GUEST → IOT denied | INSIDE-OUT 60 `deny 20→30`, placed after 30 `permit 20 any` | Same-site test not captured; remote IoT (60.10, 90.1) unreachable |
| IOT → Internet denied | INSIDE-OUT 40 `deny 192.168.30.0/24 any` | ACL still; report p. 3 (test not captured) |
| IOT → CORP denied | Covered by INSIDE-OUT 40 for routed traffic only; local SVI routing not filtered | No test captured |

INSIDE-OUT, OUTSIDE-IN, and NAT-ACL are named as if bound to interfaces and NAT, but the binding commands are not visible.
