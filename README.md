# Basic Security Monitoring & Incident Detection Report

![Domain](https://img.shields.io/badge/Domain-Cyber%20Security-blue.svg)
![Category](https://img.shields.io/badge/Category-Defensive%20Monitoring-purple.svg)
![OS](https://img.shields.io/badge/OS-Kali%20Linux%20VM-dragon.svg)

## 📌 Project Overview
Defensive security monitoring relies on continuous evaluation of system telemetry to detect potential privilege escalation, unauthorized access, or suspicious process execution. 

This repository documents a basic security monitoring pass on a Kali Linux Virtual Machine. The assessment evaluated system authentication history, process trees, network sockets, and system logs to establish an operational baseline and identify suspicious indicators. Two primary security findings were flagged during the audit.

---

## 🛠️ Monitoring Utilities & Command Toolkit

Routine defensive checks were performed using native Linux system tools and log parsers:

| Command / Log Query | Audit Category | Purpose |
| :--- | :--- | :--- |
| `journalctl -p err -b` | Error Logs | Review system error messages and authentication failures in the current boot. |
| `sudo last \| head -20` | User Sessions | Inspect recent user login sessions, terminal types, and session durations. |
| `ps aux --sort=-%cpu` | Process Tree | Audit running processes sorted by resource utilization. |
| `top -b -n1` | System Resources | Capture snapshot of current process load, CPU usage, and RAM consumption. |
| `sudo ss -tulnp` | Network Sockets | Enumerate open TCP/UDP listening ports and bound daemon processes. |
| `grep "Failed password" /var/log/auth.log` | Auth History | Search authentication logs for brute-force SSH or authentication anomalies. |

---

## 🚨 Security Anomaly Findings

### Finding 1: Repeated Failed Sudo Authentication Attempts
* **Observation Window**: August 15, 2026 (05:44:19 – 05:46:24 local time).
* **Detection Channel**: Systemd journal error-level log (`journalctl -p err -b`).
* **Event Details**: Recorded 5 separate `sudo` authentication failures within a ~2-minute window. Each entry logged 3 incorrect password attempts (15 total failed attempts) while attempting to execute privileged diagnostic commands (`ss -tulnp`, `journalctl -u ssh -u sshd`).
* **Risk Assessment**: Indicates potential local privilege escalation attempts or password guessing against an active account.
* **Mitigation Strategy**:
  * Verify whether attempts were legitimate user mistypes.
  * Check `fail2ban` service configurations to ensure PAM/sudo authentication triggers IP/account lockouts.
  * Review `/var/log/auth.log` around the timestamp to trace originating TTY/session.
  * Rotate account credentials and audit `/etc/sudoers` group membership if unauthorized.

---

### Finding 2: Unrecognized Local User Account Login
* **Observation Window**: Saturday, August 8, 2026 (13:47 – 13:51).
* **Detection Channel**: Login history (`sudo last | head -20`).
* **Event Details**: Active login session recorded for user account `HasnainA` on console `tty7`. Standard login activity on this host is limited to expected system profiles (`kali`, `lightdm`, `root`).
* **Risk Assessment**: An unrecognized local user logging into a local console session indicates potential unauthorized account creation or use of an undocumented test profile.
* **Mitigation Strategy**:
  * Verify account legitimacy with system administrators.
  * Inspect account privileges and creation timestamps via `cat /etc/passwd` and `id HasnainA`.
  * If unauthorized, delete the user and home directory (`sudo userdel -r HasnainA`) and rotate all system passwords.
  * Deploy `auditd` or enable automated `lastlog` monitoring for real-time console alerts.

---

## ✅ Verified Baseline Checks (No Anomalies)

* **Process Execution**: Analysis via `ps aux` and `top` confirmed expected system daemons (`Xorg`, `VBoxClient`, `xfce4-panel`, `containerd`, `dockerd`, `fail2ban-server`). No unauthorized binaries were running from volatile directories like `/tmp`.
* **Network Ports**: `sudo ss -tulnp` revealed a single active listening socket bound to loopback only (`127.0.0.1:41991` owned by `containerd`). No externally exposed ports or suspicious listening daemons were identified.
* **SSH Authentication Logs**: Queries against `/var/log/auth.log` for failed passwords returned zero remote SSH brute-force indicators.

---

## 🏁 Conclusion

While running processes and listening network interfaces were verified clean, the repeated `sudo` authentication failures and the login session by user `HasnainA` warrant verification. Neither finding confirms compromise, but both meet defensive thresholds for incident verification and privilege auditing.

---

## 👤 Author & Acknowledgments

* **Author**: Hasnain Ali
* **Program**: GLAXIT Internship Program — Advanced Cyber Security
* **Supervisor**: Sir Saifullah
* **Submission Date**: August 15, 2026
