# Security Onion SIEM Lab

## Objective
Deploy Security Onion to monitor network traffic in my homelab.

## Environment
- Hypervisor: (Proxmox/VirtualBox/VMware)
- Network: VLANs for lab/IoT/guest (sanitized)
- Sensors: Zeek, Suricata, Osquery

## Steps
1. Provisioned VM (CPU/RAM/Disk).
2. Ran SOSetup, selected production mode.
3. Enabled Zeek/Suricata and confirmed logs in the SOC UI.
4. Tuned noisy alerts and created detection rules.

## Validation
- Generated traffic (nmap, failed logins, malware test samples like EICAR).
- Verified alerts, PCAPs, and timelines.

## Lessons Learned
- Tuning reduces false positives significantly.
- VLAN segmentation makes detections clearer.
