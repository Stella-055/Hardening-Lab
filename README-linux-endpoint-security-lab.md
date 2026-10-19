# Linux Endpoint Security & Hardening Lab

**Type:** Independent project  
**Environment:** Fedora Linux 44 workstation  
**Date:** 19 October 2026

## Overview
This project practises foundational endpoint-security tasks on a personal Fedora Linux workstation: baseline inspection, listening-service review, basic service hardening, firewall-rule review, and verification.

## Objectives
- Inspect network interfaces, routes, firewall rules, and listening sockets.
- Distinguish services bound to all interfaces from services bound to localhost.
- Stop and disable services no longer needed.
- Remove broad high-port firewall allowances after preserving baseline evidence.
- Document findings, remediation, verification, and limitations.

## Scope and safety
Work was limited to the user's own laptop. No third-party systems were scanned or accessed. No real security incident was established, and no enterprise SIEM deployment or professional incident response is claimed.

## Baseline
- Fedora Linux 44.
- Wi-Fi interface `wlo1` was up with private address `10.183.183.95/24`; default gateway `10.183.183.121`.
- Approximately 7.6 GiB RAM and 352 GB free on the root filesystem at inspection time.
- Active zone: `FedoraWorkstation`.
- Initial firewall allowed services `dhcpv6-client`, `samba-client`, `ssh`, plus TCP and UDP ports `1025–65535`.
- Initial listeners included Apache on TCP 80 and MariaDB on TCP 3306 bound to all interfaces; PostgreSQL on TCP 5432 was bound to loopback.

## Findings
1. **Broad firewall allowances:** TCP/UDP ports `1025–65535` were allowed in the workstation zone. This is broad and should be reviewed against actual application needs; it does not prove compromise or internet reachability.
2. **Unneeded services:** Apache (`httpd`) and MariaDB were running and enabled despite not being currently needed.
3. **Network binding:** Apache and MariaDB listened on all interfaces, unlike PostgreSQL, which listened on loopback.
4. **Other listeners:** DNS resolution, Avahi/mDNS, CUPS, chronyd, `wsdd`, and browser multicast sockets were present. These were not automatically classified as malicious.

## Remediation
- Captured baseline outputs under `~/cybersecurity-lab/evidence/`.
- Stopped and disabled Apache and MariaDB:
  ```bash
  sudo systemctl disable --now httpd mariadb
  ```
- Backed up the firewalld configuration:
  ```bash
  sudo cp -a /etc/firewalld ~/cybersecurity-lab/evidence/firewalld-backup
  ```
- Removed the broad permanent port allowances and reloaded firewalld:
  ```bash
  sudo firewall-cmd --permanent --zone=FedoraWorkstation --remove-port=1025-65535/tcp --remove-port=1025-65535/udp
  sudo firewall-cmd --reload
  ```
- Post-change output showed no Apache listener on port 80 or MariaDB listener on port 3306. `systemctl is-enabled` reported `httpd` and `mariadb` as disabled; PostgreSQL remained enabled.

## Verification
Review sockets and service startup state:
```bash
sudo ss -tulpen
systemctl is-enabled httpd mariadb postgresql
sudo firewall-cmd --zone=FedoraWorkstation --list-all
```
Save the final firewall output as `firewall-after.txt` in the evidence folder if not already done.

## Limitations
- This was a local learning lab, not an enterprise security assessment.
- No Wazuh/SIEM, IDS/IPS, EDR, vulnerability scan, or external penetration test was performed.
- No real attack was identified.
- Router/NAT exposure and reachability from other devices were not independently tested.
- Application testing is needed to ensure firewall changes did not affect SSH, Samba, development services, or network discovery.

## Lessons learned
- Listening on all interfaces differs from listening on localhost.
- Enabled and running are different service states.
- A listening port is not automatically a vulnerability.
- A broad firewall rule does not prove compromise.
- Capture evidence before and after changes and document limitations honestly.

## Next steps
1. Preserve final firewall output.
2. Review system logs and distinguish routine messages from security-relevant evidence.
3. Decide whether PostgreSQL is still needed before disabling it.
4. Review SSH, Samba client functionality, Avahi, `wsdd`, and CUPS based on actual use.
5. Publish sanitized evidence only; remove unnecessary private details.

## CV entry
**Linux Endpoint Security & Hardening Lab — Independent Project**
- Assessed Fedora Linux network configuration, listening sockets, and firewall rules.
- Identified unnecessary network-facing services and broad port allowances.
- Stopped and disabled unused Apache and MariaDB services and applied local firewall hardening.
- Documented evidence, remediation, verification, and limitations.
