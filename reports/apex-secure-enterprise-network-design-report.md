# Apex Smart Manufacturing: Secure Enterprise Network Design Report

*Author: Ghazal Mahdian (individual work). CYB555 final project. **Corrected version.***

> **About this version.** This is a corrected edition of the submitted four-page report. Where the original described the platform, routing method, or test results differently from the lab evidence, the text has been changed to match the evidence. Every change is listed in [Corrections](#corrections). The original PDF is kept privately.

---

## 1. Network design

In this project, I designed and implemented a secure enterprise network for Apex Smart Manufacturing using Cisco Packet Tracer. The company operates three sites: a headquarters in Los Angeles (Site A), a branch office in Phoenix (Site B), and an international office added for expansion (Site C).

Each site follows the same architecture:

- An ISP connection that simulates Internet access (the ISP router hosts 8.8.8.8 as a loopback)
- An edge router (Cisco 2911) with a public address in 203.0.113.0/24
- A **Cisco 1841 router acting as the site firewall** (FW-SiteA, FW-SiteB, FW-SiteC). It enforces policy with IOS access control lists and terminates the site-to-site VPN. I used a router in place of a Cisco ASA in this Packet Tracer build.
- A **Cisco 3650 multilayer switch** for internal distribution and inter-VLAN routing
- Wireless access points for Wi-Fi connectivity
- End devices: PCs, laptops, and IoT devices (webcams, smart LEDs, RFID readers)

To organize the network, I implemented VLAN segmentation. Each site has four VLANs:

| Zone | Site A | Site B | Site C |
|---|---|---|---|
| Corporate (CORP) – employee devices | VLAN 10 – 192.168.10.0/24 | VLAN 40 – 192.168.40.0/24 | VLAN 70 – 192.168.70.0/24 |
| Guest (GUEST) – visitors | VLAN 20 – 192.168.20.0/24 | VLAN 50 – 192.168.50.0/24 | VLAN 80 – 192.168.80.0/24 |
| IoT (IOT) – cameras, sensors, lights | VLAN 30 – 192.168.30.0/24 | VLAN 60 – 192.168.60.0/24 | VLAN 90 – 192.168.90.0/24 |
| Management (MGMT) – network administration | VLAN 99 – 192.168.99.0/24 | VLAN 99 | VLAN 99 – 192.168.199.0/24 |

Each VLAN has its own IP subnet, which improves network organization, performance, and security. **Inter-VLAN routing is performed on the multilayer switches using switched virtual interfaces (SVIs)**, with each switch connected to its site firewall over a point-to-point link (10.10.10.0/30 at Site A).

Wireless networks were configured at each site with WPA2-PSK:

- CorpWiFi → mapped to the Corporate VLAN
- GuestWiFi → mapped to the Guest VLAN

This places wireless users into the correct security zone automatically.

For monitoring, the Site A multilayer switch sends its logs to a Syslog server in the management VLAN (192.168.99.10, UDP 514).

## 2. VPN security

A key requirement of this project was to connect the sites securely. I implemented site-to-site IPsec VPNs from the Site A firewall to the Site B and Site C firewalls.

The VPN creates an encrypted tunnel over the simulated Internet. Only Corporate-to-Corporate traffic is encrypted:

- Site A Corporate (192.168.10.0/24) ↔ Site B Corporate (192.168.40.0/24)
- Site A Corporate (192.168.10.0/24) ↔ Site C Corporate (192.168.70.0/24)

The VPN configuration included:

- ISAKMP (IKE Phase 1) to establish a secure control channel
- Pre-shared keys for authentication
- A crypto ACL (`VPN-TRAFFIC`) that defines the "interesting traffic" to be encrypted
- Transform sets that define encryption methods
- A crypto map (`VPN-MAP`) applied to the firewall's outside interface

I verified the VPN with:

```
show crypto isakmp sa
show crypto ipsec sa
```

FW-SiteA showed ISAKMP security associations in the QM_IDLE state with both remote firewalls. Its IPsec security associations showed encrypted and decrypted packets for both tunnels: 4 out / 4 in to Site B, and 18 out / 6 in to Site C. Pings between Corporate hosts at different sites succeeded after the first packets, which timed out while the tunnel was being negotiated.

The VPN provides:

- Data confidentiality through encryption
- Secure communication over untrusted networks
- Protection against eavesdropping and interception

## 3. IoT risk analysis

IoT devices such as smart cameras, smart lights, and RFID readers were deployed in this network. While these devices improve automation and monitoring, they also introduce significant security risks.

Common IoT risks include:

- Weak authentication or default credentials
- Lack of regular updates or patches
- Limited built-in security features
- Exposure to the Internet

If compromised, IoT devices can be used by attackers to:

- Spy on internal activity
- Launch attacks inside the network
- Act as entry points for further exploitation

To reduce these risks, I isolated the IoT devices:

- All IoT devices were placed in a dedicated VLAN.
- Traffic from the IoT VLAN to any destination through the firewall is denied (`INSIDE-OUT` entry 40), which blocks IoT access to the Internet.
- Corporate users are permitted to reach the IoT VLAN for monitoring (`INSIDE-OUT` entry 10). A ping from a Site A Corporate PC to an IoT device succeeded.

During testing, I verified that IoT devices could not reach external addresses such as `ping 8.8.8.8`.

Blocking IoT → Corporate traffic *inside the same site* requires filtering on the multilayer switch. Local inter-VLAN traffic is routed on the switch and does not pass through the firewall (see section 4).

## 4. Segmentation strategy

The segmentation strategy is based on Zero Trust principles: no traffic should be trusted by default.

Each VLAN represents a security zone:

- **CORP (Trusted)** – employees with controlled access
- **GUEST (Untrusted)** – public users with Internet-only access
- **IOT (Restricted)** – isolated devices with minimal permissions
- **MGMT (Admin)** – administrative access (standard ACL 99 permits only 192.168.99.0/24)
- **OUTSIDE (Internet)** – untrusted external network

The site firewall enforces segmentation policies through ACLs.

**Intended allowed traffic**
- Corporate users → Internet
- Corporate users ↔ Corporate users (via VPN)
- Management VLAN → all devices (for administration)
- Corporate users → IoT (for monitoring)

**Intended denied traffic**
- Guest → Corporate
- Guest → IoT
- IoT → Internet
- IoT → Corporate

### Investigation: guest access to an IoT camera

The scenario reported that a Guest user was able to reach an IoT camera, which indicated a misconfiguration. I reviewed the firewall ACLs and concluded that the Guest deny rules were missing or incorrectly ordered, and I added rules to deny Guest → IoT and Guest → Corporate while permitting Guest → Internet.

**Final test results.** The final configuration and recording show:

| Test | Result |
|---|---|
| Guest → Internet (8.8.8.8) | Succeeded (expected) |
| Guest → Site B IoT (192.168.60.10) | Failed (expected) |
| Guest → Site C IoT gateway (192.168.90.1) | Failed (expected) |
| Guest → Site A Corporate host (192.168.10.10) | **Succeeded (not expected)** |

**Root cause of the remaining gap.** Two problems remain in the final configuration:

1. **Rule order.** In FW-SiteA's `INSIDE-OUT` list, entry 30 (`permit ip 192.168.20.0 0.0.0.255 any`) comes before the Guest denies at entries 50 and 60. ACLs are processed top-down and stop at the first match, so every Guest packet matches entry 30 and the denies never apply.
2. **Where routing happens.** At Site A the Guest, Corporate, and IoT VLANs are all routed by the multilayer switch's SVIs. Guest-to-Corporate traffic inside the site never reaches the firewall, so firewall ACLs alone cannot block it.

**Required fix (not re-tested in the submitted lab).**
- Place the Guest deny entries *above* the Guest permit-any entry in `INSIDE-OUT`, so the Guest rules read: deny Guest → Corporate, deny Guest → IoT, then permit Guest → any.
- Apply inbound ACLs on the Guest and IoT SVIs of each multilayer switch (or VLAN access maps) so that intra-site traffic between zones is filtered where it is routed.
- Re-test Guest → Corporate and Guest → IoT at the same site and confirm the pings fail.

### Further improvements

- **IDS/IPS** at each site firewall to detect scanning and lateral movement between zones.
- **Central logging / SIEM:** extend syslog from the Site A switch to all firewalls and switches, and alert on denied Guest and IoT traffic.
- **Tighter default deny:** remove the broad `permit icmp any any` and `permit ip any 192.168.0.0 0.0.255.255` entries, and allow only the flows listed above.

---

## Corrections

Changes from the submitted PDF, with the evidence each one is based on (see [docs/evidence-index.md](../docs/evidence-index.md)):

1. **Firewall platform.** "Cisco ASA firewall" → "Cisco 1841 router acting as the site firewall". Topology screenshots label FW-SiteA/B/C as 1841 devices, and the saved file is named "555-Final with PT-Router".
2. **Inter-VLAN routing.** "router-on-a-stick" → Layer 3 switching with SVIs on 3650 multilayer switches. SW-SiteC shows SVIs (Vlan80/90/99), and FW-SiteA routes the Site A VLANs to the switch at 10.10.10.2.
3. **Sites and VPN scope.** Site C addressing was added, along with the Site A ↔ Site C tunnel (`VPN-TRAFFIC` entry 20; IPsec SA to 10.0.2.2), which the original mentioned only as "an additional international site".
4. **VPN verification.** Actual counters were added, and the note on first-packet timeouts.
5. **IoT section.** "IoT devices were blocked from accessing Corporate networks" was changed to describe the actual firewall rule and the intra-site limitation. "Only Corporate users were allowed limited access" was changed to match the ACL, which permits all IP traffic from Corporate to IoT. The IoT device list now matches the topology (webcams, smart LEDs, RFID readers).
6. **Investigation results.** The original said Guest → Corporate failed after the fix. The final recording shows Guest Laptop2 reaching 192.168.10.10. The results table, root cause, and required fix replace the original "segmentation was correctly enforced" conclusion.
7. **Additions.** Syslog, management ACL 99, and the improvements list were added. The original report did not cover them, but they appear in the lab evidence or the assignment.
8. **Unchanged claim, not independently evidenced:** the IoT `ping 8.8.8.8` failure test (section 3) is reported as in the original. No screenshot of that test was supplied.
