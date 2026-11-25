# Host-Based Endpoint Telemetry & Incident Triage with Microsoft Sysmon

## Project Overview
Standard, out-of-the-box Windows event auditing leaves substantial forensic blind spots during incident response—often failing to record explicit process command-line arguments, parent-child execution lineages, or process-to-network socket bindings. 

This defensive cybersecurity project demonstrates the deployment, configuration, and analysis of **Microsoft Sysinternals System Monitor (Sysmon)** inside a virtualized Windows 10 enterprise workstation. By pairing Sysmon with the community-standard **SwiftOnSecurity configuration schema**, this lab emulates three common adversary behaviors mapped to the **MITRE ATT&CK framework** to evaluate kernel-level event telemetry and document critical log artifacts for Tier-1 Security Operations Center (SOC) alert triage.

---

## Lab Architecture & Environment Specifications

| Component | Specification | Purpose |
| :--- | :--- | :--- |
| **Hypervisor** | Oracle VM VirtualBox (v7.x) | Hardware virtualization & test isolation |
| **Guest Endpoint** | Windows 10 Enterprise / Pro (`Win10-Endpoint`) | Target workstation running execution scenarios |
| **Networking** | VirtualBox NAT Adapter (`10.0.2.15/24`) | Outbound communication testing |
| **Telemetry Agent** | Microsoft Sysinternals Sysmon (v15+) | Kernel-mode driver (`SysmonDrv`) & event logging |
| **Telemetry Schema** | SwiftOnSecurity (`sysmonconfig-export.xml`) | Rule-based noise filtering & high-fidelity event capture |
| **Analysis Platform** | Windows Event Viewer (`eventvwr.msc`) | Local triage of `Microsoft-Windows-Sysmon/Operational` |

---

## Deployment & Baseline Procedure

1. **Working Directory & Binary Acquisition:**  
   Created a dedicated administration staging directory `C:\Tools\Sysmon\` and retrieved the official Sysinternals binaries and SwiftOnSecurity rule set using PowerShell.
2. **Cryptographic & Protocol Hardening:**  
   Enforced TLS 1.2 runtime protocol negotiation within the session to secure remote artifact delivery:
   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Invoke-WebRequest -Uri "[https://download.sysinternals.com/files/Sysmon.zip](https://download.sysinternals.com/files/Sysmon.zip)" -OutFile "Sysmon.zip" -UseBasicParsing
   Expand-Archive -Path "Sysmon.zip" -DestinationPath "C:\Tools\Sysmon" -Force
   Invoke-WebRequest -Uri "[https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml](https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml)" -OutFile "sysmonconfig.xml" -UseBasicParsing
