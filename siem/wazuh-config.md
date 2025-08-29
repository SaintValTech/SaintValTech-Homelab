# Wazuh XDR Lab

## Objective
Monitor endpoints and servers for file integrity, login events, and malware.

## Steps
1. Deployed Wazuh Manager.
2. Installed Wazuh Agent on Windows/Linux endpoints.
3. Enabled FIM, Sysmon (Windows), and key rules.
4. Forwarded events to dashboards.

## Detections Tested
- Brute-force logins, suspicious processes, file changes.
