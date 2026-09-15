# Enterprise Wazuh SIEM Detection, Compliance & Active Defense Labs

## Executive Summary
This repository documents an end-to-end host-based SIEM engineering, compliance auditing, and active threat mitigation lab deployed using Wazuh. Spanning hybrid Windows Server and Ubuntu Linux endpoints, the lab covers baseline telemetry collection, KQL threat hunting, kernel-level File Integrity Monitoring (FIM), CIS benchmark hardening, CVE vulnerability management, custom rule/decoder engineering, automated Active Response containment, and VirusTotal threat intelligence integration.

---

## Ingestion & Defense Architecture

```text
+-------------------------------------------------------------------------------+
|                      Wazuh Detection & Defense Pipeline                       |
+-------------------------------------------------------------------------------+
|                                                                               |
|  [ Windows Server Endpoint ]                 [ Ubuntu 24.04 Target ]          |
|   - Security / System Event Logs              - Journald System Logs          |
|   - Real-Time FIM & Registry Keys             - Apache / XAMPP Access Logs    |
|   - Whodata SACL Tracking                     - MariaDB General Audit Log     |
|            \                                         /                        |
|             \                                       /                         |
|              +------------------+------------------+                          |
|                                 |                                             |
|                                 | Encrypted TCP Log Stream (Port 1514)        |
|                                 v                                             |
|                     +-----------------------+                                 |
|                     |     Wazuh Server      | (Rule Engine, Custom Decoders,  |
|                     |    (Linux Host)       |  Active Response Daemon)        |
|                     +-----------+-----------+                                 |
|                                 |                                             |
|                                 +-----------------------+                     |
|                                 |                       |                     |
|                                 v                       v                     |
|                     +-----------------------+  +-------------------+          |
|                     |     Wazuh Indexer     |  | VirusTotal API v3 |          |
|                     |   (OpenSearch Store)  |  +-------------------+          |
|                     +-----------+-----------+                                 |
|                                 |                                             |
|                                 v                                             |
|                     +-----------------------+                                 |
|                     |    Wazuh Dashboard    | (SOC Threat Hunting UI)         |
|                     +-----------------------+                                 |
+-------------------------------------------------------------------------------+
```

---

## MITRE ATT&CK & CIS Compliance Coverage

| Framework Domain | Tactic / Benchmark | Identifier | Monitored Telemetry | Lab Module |
|---|---|---|---|---|
| **MITRE ATT&CK** | Initial Access / Exploitation | T1190 | Apache / XAMPP Access Logs | `06-Custom-Rules-and-Web-Attack-Detection` |
| **MITRE ATT&CK** | Credential Access (Brute Force) | T1110 | Windows Security Event ID 4625 | `02-Dashboard-Navigation-and-Threat-Hunting` |
| **MITRE ATT&CK** | Persistence (Registry Run Keys) | T1547.001 | HKLM\...\CurrentVersion\Run | `03-File-Integrity-Monitoring-FIM` |
| **MITRE ATT&CK** | Impair Defenses (Firewall Tampering) | T1562.004 | UFW Daemon Configuration | `04-Security-Configuration-Assessment-SCA` |
| **MITRE ATT&CK** | Automated Containment | Dynamic Block | `firewall-drop` (iptables) | `07-Active-Response-and-Threat-Mitigation` |
| **CIS Benchmark**| Linux System Hardening | CIS Ubuntu 24.04 | Audit Rule 35622 (UFW Service) | `04-Security-Configuration-Assessment-SCA` |
| **Vulnerability**| Known CVE Exploitation | NVD / CVE Trackers | `bluez` (CVE-2023-51596), `wget` | `05-Vulnerability-Detection-and-Patching` |

---

## Repository Lab Breakdown

### [01. Architecture & Multi-Platform Agent Deployment](./01-Architecture-and-Agent-Deployment)
* Analyzed component interaction between Agent, Server, Indexer, and Dashboard.
* Engineered endpoint log forwarding policies via `ossec.conf` and centralized `agent.conf` groups.

### [02. Dashboard Navigation & KQL Threat Hunting](./02-Dashboard-Navigation-and-Threat-Hunting)
* Decoded alert schema fields (`rule.id`, `rule.level`, `data.win.system.eventID`).
* Formulated multi-conditional KQL queries to isolate targeted administrator brute-force attempts.

### [03. File Integrity Monitoring & Registry Auditing](./03-File-Integrity-Monitoring-FIM)
* Configured real-time filesystem hooks (`ReadDirectoryChangesW`) and baseline cryptographic hashing.
* Monitored registry persistence paths and enabled Whodata SACL auditing (Event ID 4663) to identify process and user attribution.

### [04. Security Configuration Assessment & CIS Hardening](./04-Security-Configuration-Assessment-SCA)
* Executed baseline compliance audit using the CIS Ubuntu Linux 24.04 LTS Benchmark.
* Remediated non-compliant host controls (Rule 35622: UFW firewall activation) and validated pass state.

### [05. Vulnerability Detection & Remediation Lifecycle](./05-Vulnerability-Detection-and-Patching)
* Cross-referenced installed packages against live Canonical CVE and NVD databases.
* Emulated a complete vulnerability remediation lifecycle via controlled package downgrade (`wget`) and patch verification.

### [06. Custom Decoders & Web Application Attack Detection](./06-Custom-Rules-and-Web-Attack-Detection)
* Ingested Apache web server and MariaDB query audit logs into the SIEM pipeline.
* Authored custom XML detection rules for SQL Injection (`100003`), Cross-Site Scripting (`100004`), and schema destruction (`100005`).

### [07. Automated Active Response & Threat Intelligence](./07-Active-Response-and-Threat-Mitigation)
* Built an automated containment pipeline executing `firewall-drop` via iptables to temporarily drop attacking IPs for 300 seconds.
* Integrated the VirusTotal API to automate reputation lookups on newly detected binary drops.
