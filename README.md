# Wazuh XDR/SIEM Lab Deployment

![Wazuh](https://img.shields.io/badge/Wazuh-4.14.5-blue?style=flat-square)
![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04_LTS-E95420?style=flat-square&logo=ubuntu&logoColor=white)
![Windows](https://img.shields.io/badge/Windows-10-0078D6?style=flat-square&logo=windows&logoColor=white)
![Kali Linux](https://img.shields.io/badge/Kali_Linux-Agent-557C94?style=flat-square&logo=kalilinux&logoColor=white)
![VirtualBox](https://img.shields.io/badge/VirtualBox-7.1.4-183A61?style=flat-square)
![Status](https://img.shields.io/badge/Status-Complete-success?style=flat-square)

A full home lab deployment of **Wazuh 4.14.5** — an open-source XDR (Extended Detection and Response) and SIEM (Security Information and Event Management) platform — across three VirtualBox virtual machines running Ubuntu 24.04, Windows 10, and Kali Linux.

This project covers the end-to-end process: installing the Wazuh Manager stack on Ubuntu, enrolling a Windows 10 endpoint and a Kali Linux endpoint as monitored agents, and verifying live telemetry in the Wazuh Dashboard.

📝 **Full write-up on Medium:** [Read the article](https://medium.com/@ckuldeep28)

---

## Table of Contents

- [Lab Architecture](#lab-architecture)
- [Prerequisites](#prerequisites)
- [Part 1: Wazuh Manager Installation](#part-1-wazuh-manager-installation-ubuntu-24)
- [Part 2: Windows 10 Agent Deployment](#part-2-windows-10-agent-deployment)
- [Part 3: Kali Linux Agent Deployment](#part-3-kali-linux-agent-deployment)
- [Dashboard Overview](#dashboard-overview)
- [Key Takeaways](#key-takeaways)
- [Project Structure](#project-structure)

---

## Lab Architecture

Three VMs running on Oracle VirtualBox, connected via a host-only network adapter (192.168.4.0/24):

| Role | OS | Agent Name | IP Address |
|---|---|---|---|
| Wazuh Manager + Dashboard | Ubuntu 24.04 LTS | — | 192.168.4.55 |
| Monitored Endpoint 1 | Windows 10 | windows | 192.168.4.x |
| Monitored Endpoint 2 | Kali Linux | kali | 192.168.4.x |

The host-only adapter keeps all lab traffic isolated from the internet — the correct approach for any security lab environment.

**Wazuh Stack Components (all installed on Ubuntu 24):**
- **Wazuh Indexer** — OpenSearch-based data store for all agent events and alerts
- **Wazuh Manager** — processes logs, applies detection rules, generates alerts
- **Wazuh Dashboard** — web-based UI for monitoring, investigation, and reporting

---

## Prerequisites

- Oracle VirtualBox 7.1.4 or later
- Ubuntu 24.04 LTS VM (minimum 4GB RAM, 50GB disk recommended for Wazuh Manager)
- Windows 10 VM
- Kali Linux VM
- All VMs on the same host-only network, able to ping each other
- Internet access on the Ubuntu VM during installation (to download packages)
- Root/Administrator privileges on all machines

---

## Part 1: Wazuh Manager Installation (Ubuntu 24)

**Reference:** [Official Wazuh Quickstart Guide](https://documentation.wazuh.com/current/quickstart.html)

> ![Screenshot 1 — Official Wazuh documentation](screenshots/01_wazuh_documentation.jpg)
> *Official Wazuh 4.14 Quickstart documentation*

### Installation Command

Run the following on your Ubuntu 24 machine as root:

```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```

The `-a` flag deploys all three components (Indexer, Manager, Dashboard) on the same host.

> ![Screenshot 2 — Installation starting](screenshots/02_installation_start.jpg)
> *Installation starting — Wazuh 4.14.5 confirmed, dependencies resolving*

### Installation Sequence

The installer works through the following stages automatically:

```
02:10  INFO: Starting Wazuh installation assistant. Wazuh version: 4.14.5
02:11  INFO: Installing dependencies (gawk, apt-transport-https, debhelper)
02:11  INFO: Hardware requirements verified. Web interface port: 443
02:55  INFO: Wazuh indexer cluster initialized
02:59  INFO: Wazuh manager installation finished
02:59  INFO: wazuh-manager service started
03:04  INFO: Filebeat installation finished
03:04  INFO: filebeat service started
03:10  INFO: Wazuh dashboard installation finished
03:10  INFO: wazuh-dashboard service started
03:12  INFO: Wazuh dashboard web application initialized
03:12  INFO: Installation finished
```

**Total installation time: approximately 1 hour**

### Completion Output

```
INFO: --- Summary ---
INFO: You can access the web interface https://<wazuh-dashboard-ip>:443
      User: admin
      Password: [auto-generated — save immediately, shown only once]
INFO: Installation finished.
```

> ⚠️ **Save your credentials immediately.** The auto-generated password is displayed only once at the end of installation.

> ![Screenshot 3 — Installation complete](screenshots/03_installation_complete.jpg)
> *Installation complete — all three components running, credentials shown at bottom *

### Accessing the Dashboard

If accessing from the same machine where Wazuh is installed:
```
https://localhost
```

If accessing from a different machine on the same network:
```
https://192.168.4.55
```

> ![Screenshot 4 — Dashboard first login](screenshots/04_dashboard_first_login.jpg)
> *Wazuh Dashboard on first login — no agents registered yet, manager already generating baseline alerts*

---

## Part 2: Windows 10 Agent Deployment

### Step 1 — Generate the Deployment Command

In the Wazuh Dashboard, navigate to:

```
Endpoints → Deploy new agent
```

Configure the wizard with:
- **Package:** Windows → MSI 32/64 bits
- **Server address:** `192.168.4.55`
- **Agent name:** `windows`
- **Group:** Default

> ⚠️ Agent names are permanent. They cannot be changed after enrolment.

> ![Screenshot 5 — Deploy wizard Windows package](screenshots/05_deploy_agent_windows_package.jpg)
> *Deploy new agent — Windows MSI 32/64 bit selected*

> ![Screenshot 6 — Deploy wizard Windows config](screenshots/06_deploy_agent_windows_config.jpg)
> *Server address and agent name configured*

### Step 2 — Run the Command on Windows 10

Open **PowerShell as Administrator** and run the generated command:

```powershell
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.5-1.msi `
  -OutFile $env:tmp\wazuh-agent; msiexec.exe /i $env:tmp\wazuh-agent /q `
  WAZUH_MANAGER='192.168.4.55' WAZUH_AGENT_NAME='windows'
```

> ⚠️ **Administrator privileges are mandatory.** The installation will fail silently without elevation.

> ![Screenshot 7 — Windows PowerShell command](screenshots/07_deploy_agent_windows_command.jpg)
> *Wazuh-generated PowerShell command ready to run*

> ![Screenshot 8 — Windows agent downloading](screenshots/08_windows_agent_downloading.jpg)


### Step 3 — Start the Agent Service

```powershell
NET START Wazuh
```

Expected output:
```
The Wazuh service is starting.
The Wazuh service was started successfully.
```

> ![Screenshot 9 — Windows agent started](screenshots/09_windows_agent_started.jpg)
> *Wazuh service started successfully on Windows 10*

### Step 4 — Verify in Dashboard

The Agents Summary in the Wazuh Dashboard should now show:

```
Active (1)    Disconnected (0)
```

> ![Screenshot 10 — Dashboard Active 1](screenshots/10_dashboard_active_1.jpg)
> *Wazuh Dashboard confirming Windows 10 agent is live*

---

## Part 3: Kali Linux Agent Deployment

### Step 1 — Generate the Deployment Command

In the Wazuh Dashboard, navigate to:

```
Endpoints → Deploy new agent
```

Configure the wizard with:
- **Package:** Linux → DEB amd64
- **Server address:** `192.168.4.55`
- **Agent name:** `kali`
- **Group:** Default

> ![Screenshot 11 — Deploy wizard Kali config](screenshots/11_deploy_agent_kali_config.jpg)
> *Linux DEB amd64 selected, agent named "kali"*

### Step 2 — Run the Command on Kali Linux

```bash
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.5-1_amd64.deb \
  && sudo WAZUH_MANAGER='192.168.4.55' WAZUH_AGENT_NAME='kali' dpkg -i ./wazuh-agent_4.14.5-1_amd64.deb
```

> ![Screenshot 12 — Kali agent command](screenshots/12_deploy_agent_kali_command.jpg)
> *Wazuh-generated wget command for the Linux DEB agent*

> ![Screenshot 13 — Kali agent installing](screenshots/13_kali_agent_installing.jpg)
> *Agent downloading at 1.97 MB/s and installing via dpkg on Kali Linux*

### Step 3 — Enable and Start the Agent

```bash
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

The `enable` command creates a systemd symlink ensuring the agent starts automatically on reboot.

### Step 4 — Verify Both Agents in Dashboard

The Agents Summary should now show:

```
Active (2)    Disconnected (0)
```

> ![Screenshot 14 — Dashboard Active 2](screenshots/14_dashboard_active_2.jpg)
> *Both Windows 10 and Kali Linux agents active — 533 medium and 737 low severity alerts already live*

---

## Dashboard Overview

With two active agents enrolled, the following Wazuh capabilities are active and generating real telemetry:

| Category | Capability | Description |
|---|---|---|
| Endpoint Security | Configuration Assessment | Benchmarks endpoints against CIS hardening standards |
| Endpoint Security | Malware Detection | Detects indicators of compromise |
| Endpoint Security | File Integrity Monitoring | Alerts on file and directory changes |
| Threat Intelligence | Vulnerability Detection | Correlates software versions against CVE databases |
| Threat Intelligence | MITRE ATT&CK | Maps alerts to the ATT&CK framework |
| Threat Intelligence | Threat Hunting | Query-based investigation across all events |
| Security Operations | IT Hygiene | Identifies OS and network-layer misconfigurations |

---

## Key Takeaways

**1. Single-command installation** — The Wazuh quickstart script handles all three components, dependency resolution, SSL certificate generation, and internal user setup in one run.

**2. Agent names are permanent** — Choose them carefully before enrolment. They cannot be changed afterwards.

**3. Network connectivity is critical** — Confirm all VMs can ping the manager IP (192.168.4.55) before deploying agents. A misconfigured network adapter is the most common cause of an agent installing successfully but not appearing active in the dashboard.

**4. Administrator privileges required on Windows** — The MSI installation will fail without PowerShell running as Administrator.

**5. Alert volume reflects real activity** — Within hours of enrolling two endpoints, over 1,200 alerts had been generated from normal system activity. This baseline data is what you tune detection rules against.

---

## Project Structure

```
wazuh-xdr-siem-deployment/
├── README.md
└── screenshots/
    ├── 01_wazuh_documentation.jpg
    ├── 02_installation_start.jpg
    ├── 03_installation_complete.jpg
    ├── 04_dashboard_first_login.jpg
    ├── 05_deploy_agent_windows_package.jpg
    ├── 06_deploy_agent_windows_config.jpg
    ├── 07_deploy_agent_windows_command.jpg
    ├── 08_windows_agent_downloading.jpg
    ├── 09_windows_agent_started.jpg
    ├── 10_dashboard_active_1.jpg
    ├── 11_deploy_agent_kali_config.jpg
    ├── 12_deploy_agent_kali_command.jpg
    ├── 13_kali_agent_installing.jpg
    └── 14_dashboard_active_2.jpg
```

---

## Related Projects

- [Metasploitable 2 — Full VAPT (38 Vulnerabilities)](https://github.com/Deep2511)
- Suricata IDS Deployment *(coming soon)*

---

*All work performed in an isolated VirtualBox lab environment for educational and portfolio purposes.*

**Author:** Kuldeep | [GitHub](https://github.com/Deep2511) | [Medium](https://medium.com/@ckuldeep28)
