# Wazuh Endpoint Security Monitoring Assessment

**Project:** SIEM Deployment and File Integrity Monitoring  
**Role:** Security Analyst / Systems Engineer  
**Technologies:** Wazuh Manager, Wazuh Agent, VMware Workstation Pro, Windows 11, Linux, File Integrity Monitoring (FIM), PCI DSS, GDPR  

## 📌 1. Introduction
This report details the successful setup, configuration, and validation of a Wazuh-based endpoint security monitoring solution. The goal of this assessment was to create a Wazuh Manager virtual machine on a Windows host, deploy a Windows endpoint agent, set up security monitoring policies, and confirm that security events and compliance data are accurately collected and displayed in the Wazuh Dashboard. The implementation shows a practical understanding of Security Information and Event Management (SIEM), endpoint monitoring, and regulatory compliance concepts.

## 🏗️ 2. Environment and Prerequisites
### 2.1 Host and Virtualization Environment
*   **Host Operating System:** Windows 11
*   **Virtualization Platform:** VMware Workstation Pro

---

## 🚀 3. Wazuh Manager Deployment

### Importing the Wazuh OVA
The Wazuh Manager was set up by importing the official OVA file into VMware using the open Virtual Machine option. The default settings were checked and adjusted to meet the minimum system requirements.

![VMware Import Screen](wazuh-images/page_1_img_1.png)

### 3.1 Wazuh Manager Virtual Machine Specs
*   **Appliance:** Official Wazuh OVA (version 4.14.1)
*   **Allocated Resources:**
    *   Memory: 6 GB RAM
    *   CPU: 2 virtual CPUs
    *   Storage: 50 GB
*   **Network Mode:** Bridged Adapter (as configured)

![VM Settings](wazuh-images/page_2_img_1.png)

### 3.2 Initial Login and IP Address Discovery
After starting the VM, the system was accessed through the terminal using the default credentials. The assigned IP address was found using the `ip a` command (`192.168.128.169`). This IP address is necessary for dashboard access and agent registration.

![IP Discovery](wazuh-images/page_2_img_2.png)

### 3.3 Dashboard Access & Troubleshooting
I attempted to use a web browser on the Windows host to access the Wazuh Dashboard via HTTPS with the VM’s IP address (`192.168.128.169`) but it was failing. I did some troubleshooting via the terminal:

```bash
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-dashboard
```

![Failing Services](wazuh-images/page_3_img_1.png)

I found out that only the `wazuh-dashboard` service was running; the `wazuh-indexer` and `wazuh-manager` were failing. I had to manually start them with the following commands:
```bash
sudo systemctl start wazuh-indexer
sudo systemctl start wazuh-manager
```
After waiting 5 minutes, I tried accessing the dashboard in the browser again and successfully logged in with the default credentials (`admin:admin`), confirming the services were fully operational.

![Dashboard Login](wazuh-images/page_3_img_2.png)

---

## 🛡️ 4. Windows Endpoint Agent Deployment

Inside the Wazuh Dashboard, I clicked the **"Deploy new agent"** button to generate an installation command for my Windows 11 host. 

I set the server address to the IP address of my Wazuh manager (`192.168.128.169`) and assigned the agent name as `mywindows`.

### 4.1 Agent Deployment Command Generation
The dashboard generated a Windows-specific agent deployment command to ensure the agent had the correct manager IP address and registration settings:

```powershell
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.1-1.msi -OutFile $env:tmp\wazuh-agent; msiexec.exe /i $env:tmp\wazuh-agent /q WAZUH_MANAGER='192.168.128.169' WAZUH_AGENT_NAME='mywindows'
```

### 4.2 Agent Installation on Windows Endpoint
The generated PowerShell command was run on the Windows endpoint with administrator permissions. The installation completed silently.

![PowerShell Execution](wazuh-images/page_5_img_1.png)

I then started the service manually using:
```cmd
net start wazuh
```

### 4.3 Agent Connectivity Verification
After installation, the Windows endpoint appeared in the Wazuh Dashboard with an **Active** status, confirming successful communication between the agent and the manager.

![Active Agent](wazuh-images/page_6_img_1.png)

---

## 📁 5. File Integrity Monitoring (FIM) Configuration

I created a folder in my Downloads directory called `FIMtest`, and inside it, a text document named `testfile.txt`. 

> [!NOTE]
> **Troubleshooting Disconnected Agent:** While configuring FIM, I noticed my Wazuh IP had been dynamically changed by my ISP to `192.168.142.169`, causing the agent to disconnect. I resolved this by editing the `ossec.conf` file located in the `ossec-agent` folder on the Windows host, changing the `<address>` tag to the new IP, and restarting the Wazuh service.

### 5.1 Manager Configuration Update
File Integrity Monitoring was set up by editing the `ossec.conf` file on the Windows endpoint. I designated the `FIMtest` folder for monitoring in the `<syscheck>` block, and reduced the scan frequency to 60 seconds so updates would register faster:

```xml
<directories check_all="yes" report_changes="yes" realtime="yes">C:\Users\hp\Downloads\FIMtest</directories>
```

### 5.2 FIM Testing
I restarted the Wazuh Manager service (`Restart-Service -Name Wazuh`) to apply changes, then conducted two test cases on the Windows endpoint:
1.  **Modification:** Modified `testfile.txt`.
2.  **Deletion:** Deleted `testfile.txt`.

These actions successfully generated real-time FIM alerts within the Wazuh Dashboard.

---

## 📊 6. Monitoring and Alert Verification

### 6.1 File Integrity Monitoring Alerts
The Wazuh Dashboard successfully captured the FIM alerts. Alerts related to file modifications and deletions were observed and mapped to relevant rule IDs.

| Action | Description | Rule ID |
| :--- | :--- | :--- |
| **File Modified** | Detected change in file content/attributes | `550` |
| **File Deleted** | Detected removal of monitored file | `553` |

![FIM Alerts](wazuh-images/page_8_img_1.png)

---

## 📜 7. Regulatory Compliance Monitoring

### 7.1 Compliance Dashboards
The **GDPR** and **PCI DSS** compliance dashboards were accessed to observe how the collected security events relate directly to regulatory requirements, highlighting Wazuh’s capability to support compliance auditing.

![PCI DSS Dashboard](wazuh-images/page_8_img_2.png)

### 7.2 Compliance Event Filtering
Security events were filtered using compliance tags (`rule.pci_dss:*`) to show only events relevant to specific regulations.

![Compliance Filtering](wazuh-images/page_9_img_1.png)

---

## 🛠️ 8. Challenges Encountered and Resolutions

During the setup, delays in service startup and connectivity issues arose due to limited system resources and IP address changes after VM restarts. These problems were fixed by:
*   **Service Tuning:** Adjusting system service timeouts.
*   **Endpoint Configuration:** Ensuring the correct manager IP configuration on the Windows agent (`ossec.conf`).
*   **Service Orchestration:** Restarting relevant Wazuh services in the proper order.

This troubleshooting significantly improved my understanding of service dependencies and communication between the endpoint and the manager.

---

## 🎯 9. Conclusion
The assessment goals were fully achieved. The Wazuh Manager was deployed, a Windows endpoint agent was connected, File Integrity Monitoring was configured and tested, and security/compliance events were validated through the dashboard. This implementation showcases a functional endpoint security monitoring solution capable of detecting file changes, collecting security logs, and supporting automated compliance monitoring.
