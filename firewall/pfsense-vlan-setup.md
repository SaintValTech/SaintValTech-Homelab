# pfSense VLAN & Segmentation

## Objective
Segment homelab into VLANs (Lab, IoT, Guest) and restrict lateral movement.

## Steps
1. Created VLANs on LAN interface.
2. Assigned interfaces and DHCP scopes.
3. Wrote inter-VLAN firewall rules (default deny, allow specific services).
4. Enabled DNS resolver with filtering.

## Validation
- Ping tests, service access checks, and rule logging.
