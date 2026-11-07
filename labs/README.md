# Packet Tracer file (not published)

The simulation file for this lab (`555-Final with PT-Router-1.pkt`, about 155 KB, saved with Cisco Packet Tracer 9.0.0) is **not included** in this repository. It contains the IPsec pre-shared keys, the WPA2 passphrases for the Corp and Guest wireless networks, and device passwords.

All configuration evidence in this repository comes from screenshots and stills from the recording (see [docs/evidence-index.md](../docs/evidence-index.md)). The file is available to reviewers on request.

## Publishing a sanitized copy later

If you want to add the file back:

1. Open the `.pkt` in Packet Tracer and **Save As** a new file (keep the original private).
2. On FW-SiteA, FW-SiteB, and FW-SiteC, replace the ISAKMP pre-shared key with the same placeholder on both ends of each tunnel. Example: `crypto isakmp key LAB-PSK-REDACTED address <peer>`. Also replace any enable secrets and console/VTY passwords.
3. On every access point (Site A, B, and C), change the WPA2 passphrases to placeholders. Update the wireless clients to match so they can still associate.
4. Clear the credentials on IoT devices and the Packet Tracer user profile (Options → User Profile).
5. Re-test before saving: `show crypto isakmp sa` should still show QM_IDLE, and a Corp PC should still ping 192.168.40.10 across the VPN.
6. Save, then add the file as `labs/apex-three-site-secure-network.pkt` and update this README and the main README.
