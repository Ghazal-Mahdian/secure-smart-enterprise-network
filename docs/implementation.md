# Implementation

No configuration files were supplied. The blocks below are **transcriptions of screenshots or recording stills**, not exported configuration. Each names its source, and gaps are listed at the end.

## 1. VLANs

Source: `sw-sitea-show-vlan-brief.png` (SW-SiteA, 10 May 2026)

```text
VLAN Name      Ports (transcription)
1    default   Gig1/0/8–1/0/22, Gig1/1/1–1/1/4
10   CORP      Gig1/0/1, Gig1/0/2, Gig1/0/6
11   VLAN0011  Gig1/0/24
20   GUEST     Gig1/0/7
30   IOT       Gig1/0/3, Gig1/0/4, Gig1/0/5
99   MGMT      Gig1/0/23
```

Source: `sw-siteb-show-vlan-brief.png` (SW-SiteB)

```text
40   CORP      Gig1/0/1, Gig1/0/2, Gig1/0/7
50   GUEST     Gig1/0/8
60   IOT       Gig1/0/4, Gig1/0/5, Gig1/0/6
99   MGMT      (no ports)
```

Source: `sw-sitec-show-vlan-brief.png` (SW-SiteC; the top of the capture shows the end of `show ip interface brief`)

```text
Vlan80   192.168.80.1    up   up
Vlan90   192.168.90.1    up   up
Vlan99   192.168.199.1   up   down
...
11   VLAN0011  Gig1/0/24
70   CORP      Gig1/0/1, Gig1/0/2, Gig1/0/3
80   GUEST     Gig1/0/4
90   IOT       Gig1/0/5, Gig1/0/6
99   MGMT      (no ports)
```

## 2. Routing

Source: `fw-sitea-show-ip-route.png`

```text
FW-SiteA#show ip route              (transcription)
Gateway of last resort is 10.0.0.1 to network 0.0.0.0
C   10.0.0.0/30 is directly connected, FastEthernet0/0
C   10.10.10.0/30 is directly connected, FastEthernet0/1
S   192.168.10.0/24 [1/0] via 10.10.10.2
S   192.168.20.0/24 [1/0] via 10.10.10.2
S   192.168.30.0/24 [1/0] via 10.10.10.2
S*  0.0.0.0/0 [1/0] via 10.0.0.1
```

Source: `isp-router-show-ip-route.png`

```text
ISP Router#show ip route            (transcription)
C   8.8.8.8/32 is directly connected, Loopback0
S   10.0.0.0/30 [1/0] via 203.0.113.1
S   10.0.1.0/30 [1/0] via 203.0.113.5
S   10.0.2.0/30 [1/0] via 203.0.113.10
S   192.168.10.0/24 [1/0] via 203.0.113.1
S   192.168.40.0/24 [1/0] via 203.0.113.5
S   192.168.70.0/24 [1/0] via 203.0.113.10
C   203.0.113.0/30 G0/0   C 203.0.113.4/30 G0/1   C 203.0.113.8/30 G0/2
S*  0.0.0.0/0 is directly connected, GigabitEthernet0/0
```

Router 1 (`router1-site-a-edge-show-ip-route.png`) is summarized in [architecture.md](architecture.md#routed-path-site-a-observed).

## 3. FW-SiteA access lists

Source: `recording-fw-sitea-show-access-lists.png` (recording still, about 21:00, 13 May 2026)

```text
FW-SiteA#show access-lists          (transcription)
Extended IP access list VPN-TRAFFIC
    10 permit ip 192.168.10.0 0.0.0.255 192.168.40.0 0.0.0.255
    20 permit ip 192.168.10.0 0.0.0.255 192.168.70.0 0.0.0.255 (2 match(es))
Extended IP access list NAT-ACL
    10 deny ip 192.168.10.0 0.0.0.255 192.168.40.0 0.0.0.255
    20 permit ip 192.168.10.0 0.0.0.255 any
    30 permit ip 192.168.20.0 0.0.0.255 any
Extended IP access list INSIDE-OUT
    10 permit ip 192.168.10.0 0.0.0.255 192.168.30.0 0.0.0.255
    20 permit ip 192.168.10.0 0.0.0.255 any (54 match(es))
    30 permit ip 192.168.20.0 0.0.0.255 any (20 match(es))
    40 deny ip 192.168.30.0 0.0.0.255 any
    50 deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
    60 deny ip 192.168.20.0 0.0.0.255 192.168.30.0 0.0.0.255
    70 permit icmp any any
Extended IP access list OUTSIDE-IN
    10 permit icmp any any (21 match(es))
    20 permit ip any 192.168.0.0 0.0.255.255
    30 permit ip 192.168.40.0 0.0.0.255 192.168.10.0 0.0.0.255
    40 permit udp any any eq isakmp (687 match(es))
    50 permit esp any any (14 match(es))
Standard IP access list 99
    10 permit 192.168.99.0 0.0.0.255
```

Earlier frames of the recording (about 9:00) show the same lists with lower hit counts (38 and 12 matches on INSIDE-OUT 20 and 30).

Observations, for the author's review:

- **INSIDE-OUT 30** permits all Guest traffic before the Guest denies at 50 and 60, so those denies would never match.
- **OUTSIDE-IN 20** permits any source to all of 192.168.0.0/16, which makes the narrower entry at 30 redundant and is broader than the default-deny goal.
- **NAT-ACL** exempts 10 → 40 but not 10 → 70. On real IOS, Site C–bound Corp traffic would be translated before crypto matching, yet the 10 → 70 SA shows traffic. This may be a Packet Tracer behaviour, or the NAT statement may use a different list. It cannot be resolved without the running configuration.

## 4. FW-SiteA IPsec

Sources: `recording-fw-sitea-show-crypto-isakmp-sa.png` (about 14:35), `recording-fw-sitea-ipsec-sa-to-site-b.png` (about 14:50), `recording-fw-sitea-ipsec-sa-to-site-c.png` (about 15:05)

```text
FW-SiteA#show crypto isakmp sa      (transcription)
dst        src        state     conn-id slot status
10.0.1.2   10.0.0.2   QM_IDLE   1083    0    ACTIVE (deleted)
10.0.2.2   10.0.0.2   QM_IDLE   1048    0    ACTIVE
```

| Local → remote ident | Peer | Encaps | Decaps | Errors | Interface / map |
|---|---|---|---|---|---|
| 192.168.10.0/24 → 192.168.40.0/24 | 10.0.1.2 | 4 | 4 | recv 6 | Fa0/0, crypto map `VPN-MAP`, local 10.0.0.2 |
| 192.168.10.0/24 → 192.168.70.0/24 | 10.0.2.2 | 18 | 6 | 0 | Fa0/0 |

`current outbound spi: 0x0(0)` appears in both captures, and the ISAKMP SA to Site B is listed as deleted in the same output. Together these suggest the Site B SA was being renegotiated at capture time. Transform set, ISAKMP policy, and pre-shared key are not visible.

## 5. Wireless and hosts

| Item | Observed | Source |
|---|---|---|
| SSIDs visible to a laptop | GuestWiFi, CorpWiFi, CorpWiFi-B, GuestWiFi-B (all channel 1) | `laptop-wireless-ssid-list-wpa2-psk.png` |
| Security | WPA2-PSK, Mixed B/G, Infrastructure | same |
| Association | "You have successfully connected to the access point" (SSID not shown) | `laptop-wireless-association-success.png` |
| Corp Laptop0 (Wireless0) | DHCP 192.168.10.21 /24, gw 192.168.10.1, DNS 8.8.8.8 | `corp-laptop0-wireless-dhcp-vlan10.png` |
| IoT-1 (webcam, Site A) | Static 192.168.30.10 /24, gw 192.168.30.1, no IoT server | `iot1-static-ip-vlan30.png`, `iot1-gateway-settings.png` |
| IoT5 (smart LED, Site C) | DHCP, gw 192.168.90.1, DNS 8.8.8.8 | `iot5-site-c-dhcp-settings.png` |

## 6. Syslog

Source: `multilayer-switch1-syslog-logging-host.png`

```text
Trap logging: level debugging, 44 message lines logged      (transcription)
    Logging to 192.168.99.10 (udp port 514, audit disabled, ... link up),
        7 message lines logged
```

Source: `syslog-server-received-messages.png`: five entries from 192.168.99.1 (%LINK-5-CHANGED, %LINEPROTO-5-UPDOWN, %LINK-3-UPDOWN, %SYS-5-CONFIG_I), timestamped with Packet Tracer's default 1993 clock.

## Not captured

- Running configurations of any device; ACL-to-interface bindings; NAT statements.
- ISAKMP policy, pre-shared key, transform set, crypto map entries.
- Access point SSID-to-VLAN configuration and WPA2 passphrases.
- Any output from FW-SiteB, FW-SiteC, Router 2, Router 3, or SW-SiteB beyond `show vlan brief`.
