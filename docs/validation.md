# Validation

Status key:

- **Demonstrated**: visible in a supplied screenshot or recording still.
- **Partial**: shown, with failures, missing steps, or configuration only.
- **Reported**: stated in the report; no supporting capture.
- **Failed**: evidence shows the expected outcome did not occur.
- **Not shown**: required or proposed; no evidence.
- **Not implemented**: a different approach was used.

## Assignment "Must Work" / "Must Fail" tests

| Test | Expected outcome | Observed outcome | Evidence | Status |
|---|---|---|---|---|
| CORP Site A ↔ Site B over VPN | Reachable and encrypted | PC1 → 192.168.40.10: 3/4 (first timed out), TTL 124; an earlier attempt at the top of the capture shows 0/4. IPsec SA 10.0 → 40.0: 4 encaps / 4 decaps | `pc1-ping-iot-vpn-internet.png`, `recording-fw-sitea-ipsec-sa-to-site-b.png` | Demonstrated (with initial loss) |
| CORP Site A ↔ Site C over VPN (expansion) | Reachable and encrypted | PC1 → 192.168.70.2: 0/4, then 4/4 TTL 124 (recording about 16:00). SA 10.0 → 70.0: 18 encaps / 6 decaps | recording; `recording-fw-sitea-ipsec-sa-to-site-c.png` | Demonstrated (with initial failure) |
| CORP → Internet | Reachable | PC1 → 8.8.8.8: 4/4 | `pc1-ping-iot-vpn-internet.png` | Demonstrated |
| GUEST → Internet | Reachable | Guest Laptop2 → 8.8.8.8: 4/4 | `guest-laptop2-ping-iot-and-internet.png` | Demonstrated |
| WiFi connectivity | Corp and Guest clients associate and get addresses | SSID list with WPA2-PSK; one association success (SSID not visible); Corp Laptop0 DHCP 192.168.10.21 on Wireless0 | `laptop-wireless-*.png`, `corp-laptop0-wireless-dhcp-vlan10.png` | Partial (Guest wireless lease not shown) |
| IOT → Internet | Fails | INSIDE-OUT 40 denies 192.168.30.0/24 → any. Report says `ping 8.8.8.8` from IoT failed; no capture | ACL still; report p. 3 | Reported (configuration shown) |
| GUEST → CORP | Fails | Guest Laptop2 → 192.168.10.10: **4/4 replies**, TTL 127 (recording about 19:00, 13 May 2026) | `recording-guest-laptop2-ping-corp-host-succeeds.png` | **Failed** |
| GUEST → IOT (remote site) | Fails | Guest Laptop2 → 192.168.60.10 (Site B IoT): 0/4; → 192.168.90.1 (Site C IoT gateway): 0/4 | `guest-laptop2-ping-iot-and-internet.png`, recording still | Demonstrated |
| GUEST → IOT (same site) | Fails | No Guest → 192.168.30.x test captured | — | Not shown |

## Other requirements

| Test | Expected outcome | Observed outcome | Evidence | Status |
|---|---|---|---|---|
| VLANs per site | 10/20/30/99, 40/50/60/99 (+ 70/80/90/99) | Present on SW-SiteA/B/C; MGMT VLAN 99 has no ports at B and C; extra VLAN 11 at A and C | `sw-site*-show-vlan-brief.png` | Demonstrated |
| Inter-VLAN routing | Router-on-a-stick or L3 switch | SVIs on the multilayer switches; FW-SiteA routes local VLANs via the switch | `sw-sitec-show-vlan-brief.png`, `fw-sitea-show-ip-route.png` | Demonstrated (L3 switch; report says router-on-a-stick) |
| CORP → IOT monitoring | Allowed | PC1 → 192.168.30.11: 4/4 | `pc1-ping-iot-vpn-internet.png` | Demonstrated |
| IOT → CORP | Fails | No test captured | — | Not shown |
| ASA with inside/outside, NAT (PAT), ACLs | ASA configured | 1841 routers with IOS ACLs; NAT-ACL present; no ASA | topology; ACL still | Not implemented (IOS router substitute) |
| Default deny between VLANs | Only required traffic allowed | INSIDE-OUT ends with `permit icmp any any`; OUTSIDE-IN 20 permits any → 192.168.0.0/16; intra-site inter-VLAN routing is not filtered in any capture | ACL still | Not shown / contradicted |
| Management restricted to VLAN 99 | Admin access only from MGMT | Standard ACL 99 permits 192.168.99.0/24; application not shown | ACL still | Partial |
| Syslog / monitoring server | Devices log to server | SW-SiteA logs to 192.168.99.10 (7 lines); server received 5 messages | `multilayer-switch1-syslog-logging-host.png`, `syslog-server-received-messages.png` | Demonstrated (Site A switch only) |
| DHCP per VLAN | Pools for each VLAN | Corp wireless (VLAN 10) and IoT5 (VLAN 90) use DHCP; IoT-1 is static | host config captures | Partial |
| VPN restricted to specific VLANs | Only CORP tunneled | VPN-TRAFFIC lists only 192.168.10.0/24 sources | ACL still | Demonstrated (Site A side) |
| Third site | Site C built and connected | Site C topology, VLANs, VPN SA, PC1 → 70.2 reachable | `topology-site-c-expansion-packet-tracer.png`, recording | Demonstrated |
| Simulated attack traffic | Attack simulated | Not shown | — | Not shown |
| Encrypted communication verified | `show crypto` output | ISAKMP and IPsec SAs on FW-SiteA only | recording stills | Demonstrated (one side) |

## Investigation task analysis

**Scenario:** a guest user reached an IoT camera at the branch site.

**Original report (p. 4, since corrected):** the deny rule was "either missing or incorrectly ordered". After `deny GUEST → IOT`, `deny GUEST → CORP`, and `permit GUEST → Internet` were added, Guest → IoT and Guest → Corporate failed and Guest → Internet still worked.

**What the evidence shows:**

1. The FW-SiteA `INSIDE-OUT` list (recording, 13 May) has `30 permit ip 192.168.20.0 0.0.0.255 any` ahead of `50 deny 20 → 10` and `60 deny 20 → 30`. With first-match processing, every Guest packet matches entry 30, so 50 and 60 never apply. Entry 30 had 20 matches; 50 and 60 show none.
2. At Site A, the Guest, Corp, and IoT VLANs all terminate on the multilayer switch's SVIs, and FW-SiteA routes those subnets back to the switch (10.10.10.2). Traffic between local VLANs is therefore routed on the switch and never crosses FW-SiteA. Edge-firewall ACLs cannot block Guest → Corp or Guest → IoT within the same site unless the switch also filters.
3. Consistent with points 1 and 2, Guest Laptop2 → 192.168.10.10 succeeds in the recording (TTL 127, one routed hop).
4. Guest → Site B IoT (192.168.60.10) and Site C IoT gateway (192.168.90.1) fail. The captures do not show whether this is caused by an ACL or by routing: the ISP router has no routes to 192.168.60.0 or 192.168.90.0, and the VPN only carries CORP subnets.

**Conclusion:** The diagnosis (ordering of deny rules) is correct in principle. The captured configuration still has the ordering problem, and the original report's claim that Guest → Corporate was blocked is contradicted by the recording. The [corrected report](../reports/apex-secure-enterprise-network-design-report.md#investigation-guest-access-to-an-iot-camera) now states this. Two fixes would close the gap: move the Guest denies above entry 30, and apply ACLs on the switch SVIs (or VLAN ACLs) for intra-site separation. Both should be re-tested with captured failing pings.

**Proposed improvements in the assignment** (IDS/IPS, logging/SIEM, stronger segmentation) were not discussed in the original submission beyond the syslog server; the corrected report adds them as recommendations (not implemented).
