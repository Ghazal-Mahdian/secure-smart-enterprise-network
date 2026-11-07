# Evidence Index

Sources are from the course archive `InfoAssurance555-Final.zip`. Screenshot times are the macOS capture timestamps; recording times are offsets into the 23:38 walkthrough video.

## Published screenshots

| Published file | Original | Shows | Redaction |
|---|---|---|---|
| `topology-sites-a-b-packet-tracer.png` | Screenshot 2026-05-10 11.50.26 PM | Sites A and B, ISP, device models | — |
| `topology-site-c-expansion-packet-tracer.png` | 2026-05-10 11.50.33 PM | Site C | — |
| `illustration-ai-generated-network-overview.png` | ChatGPT Image May 13, 2026 | **Illustration only**, not evidence | — |
| `sw-sitea-show-vlan-brief.png` | 2026-05-10 11.45.54 PM | SW-SiteA VLANs | file path in title bar |
| `sw-siteb-show-vlan-brief.png` | 2026-05-10 11.46.20 PM | SW-SiteB VLANs | file path |
| `sw-sitec-show-vlan-brief.png` | 2026-05-10 11.46.33 PM | SW-SiteC SVIs + VLANs | file path |
| `fw-sitea-show-ip-route.png` | 2026-05-10 11.47.12 PM | FW-SiteA routing table | file path |
| `router1-site-a-edge-show-ip-route.png` | 2026-05-10 11.47.51 PM | Router 1 routing table | file path |
| `isp-router-show-ip-route.png` | 2026-05-10 11.48.37 PM | ISP routing table, 8.8.8.8 loopback | file path |
| `corp-laptop0-wireless-dhcp-vlan10.png` | 2026-05-10 10.58.30 PM | Wireless DHCP lease 192.168.10.21 | file path |
| `laptop-wireless-ssid-list-wpa2-psk.png` | 2026-05-10 9.17.10 AM | Four SSIDs, WPA2-PSK | — |
| `laptop-wireless-association-success.png` | 2026-05-10 9.17.52 AM | Association success | — |
| `iot1-static-ip-vlan30.png` | 2026-05-10 5.15.28 PM | IoT-1 192.168.30.10 static | — |
| `iot1-gateway-settings.png` | 2026-05-10 5.15.37 PM | IoT-1 gateway 192.168.30.1 | — |
| `iot5-site-c-dhcp-settings.png` | 2026-05-13 11.50.59 AM | IoT5 DHCP, gw 192.168.90.1 | — |
| `pc1-ping-iot-vpn-internet.png` | 2026-05-10 10.48.02 PM | PC1 → IoT / Site B Corp / 8.8.8.8 | — |
| `guest-laptop2-ping-iot-and-internet.png` | 2026-05-13 11.50.45 AM | Guest → 60.10 fail, 8.8.8.8 ok, 90.1 fail | — |
| `multilayer-switch1-syslog-logging-host.png` | 2026-05-10 11.12.49 PM | Logging to 192.168.99.10 | file path |
| `syslog-server-received-messages.png` | 2026-05-10 11.13.09 PM | Syslog server entries | file path |

## Recording stills

Source: `video1524848351.mp4` (1920×1080, 23:38, recorded 13 May 2026). Each still is cropped to the device window, which removes the desktop, menu bar, and file path.

| Published file | Offset | Shows |
|---|---|---|
| `recording-fw-sitea-show-crypto-isakmp-sa.png` | ~14:35 | ISAKMP SAs to 10.0.1.2 and 10.0.2.2 |
| `recording-fw-sitea-ipsec-sa-to-site-b.png` | ~14:50 | SA 10.0 → 40.0, 4/4, crypto map VPN-MAP |
| `recording-fw-sitea-ipsec-sa-to-site-c.png` | ~15:05 | SA 10.0 → 70.0, 18 encaps / 6 decaps |
| `recording-guest-laptop2-ping-corp-host-succeeds.png` | ~19:00 | Guest → 192.168.10.10 succeeds; → 60.10 fails |
| `recording-fw-sitea-show-access-lists.png` | ~21:00 | All FW-SiteA ACLs with hit counts |

### What the recording covers (from frames sampled once per minute plus targeted frames)

| Offset | Content |
|---|---|
| 0:00–3:30 | Packet Tracer topology; the AI-generated overview illustration |
| 4:00–6:00 | Multilayer switch `show vlan brief` and interface output |
| 6:30–7:00 | Router 1 `show ip interface brief` |
| 7:30–8:00 | Host ping tests to 192.168.10.21, 192.168.10.10, 192.168.70.2, 192.168.30.11 (source host not legible at sampled resolution) |
| 8:30–10:30 | FW-SiteA `show access-lists` |
| 11:00–13:00 | Wireless client, Guest Laptop2 tests |
| 14:30–15:30 | FW-SiteA `show crypto isakmp sa`, `show crypto ipsec sa` |
| 16:00–18:00 | PC1 → 192.168.70.2 (fail, then success), 8.8.8.8 |
| 18:30–19:30 | Guest Laptop2 → 192.168.10.10 (success), → 192.168.60.10 (fail) |
| 20:00–21:30 | FW-SiteA `show access-lists` again |
| 22:00–23:38 | Guest Laptop2 tests; overview illustration |

**The audio was not reviewed.** No transcription tool was available, and the separate `audio1524848351.m4a` (same 23:38 duration) was not listened to. Neither file is included in this repository.

## Originals not published

| Original | Reason |
|---|---|
| `video1524848351.mp4` (34 MB) | Audio unreviewed; shows local file path and desktop; key frames published as stills instead |
| `audio1524848351.m4a` | Unreviewed audio, probably the recording's soundtrack |

All 18 original screenshots and the illustration are published (renamed). There were no exact duplicates.

## Report

`reports/apex-secure-enterprise-network-design-report.md` is the **corrected** version. The original four-page PDF is kept privately. Page references in these docs ("report p. N") refer to the original, which maps onto the corrected sections as follows:

| Original page | Content | Corrected section |
|---|---|---|
| 1 | Network design: sites, VLAN zones, "router-on-a-stick" (corrected), wireless | §1 |
| 2 | VPN: ISAKMP, PSK, crypto ACL, transform set, crypto map; verification | §2 |
| 3 | IoT risks and isolation; `ping 8.8.8.8` from IoT failed (claim) | §3 |
| 4 | Zero-trust zones, allowed/denied flows, investigation fix and results (corrected) | §4 |

The corrected report lists each change and its evidence under "Corrections".

## Claim → source

| Claim | Source |
|---|---|
| Firewalls are 1841 routers, not ASA | Topology screenshots (labels "1841 FW-SiteA/B/C"); original file name "555-Final with PT-Router.pkt" |
| Original report said "Cisco ASA firewall" (corrected) | Original report pp. 1, 4 |
| Original report said router-on-a-stick (corrected) | Original report p. 1 |
| L3 switch SVIs | `sw-sitec-show-vlan-brief.png`; `fw-sitea-show-ip-route.png` (VLANs via 10.10.10.2) |
| Guest denies shadowed by permit | `recording-fw-sitea-show-access-lists.png` |
| Guest → Corp succeeds | `recording-guest-laptop2-ping-corp-host-succeeds.png` |
| Original report said Guest → Corporate failed after fix (corrected) | Original report p. 4 |
| IoT → Internet blocked (claim) | Report p. 3; ACL entry INSIDE-OUT 40 |
| ISP has routes to CORP subnets | `isp-router-show-ip-route.png` |

## Simulation file

| File | Original | Verified? |
|---|---|---|
| Not published | `555-Final with PT-Router-1.pkt` (155 KB) | Contains VPN pre-shared keys and Wi-Fi passphrases; see `labs/README.md` |
