# ☁️ Azure IaaS Project:  Secure Personal Cloud Environment
[![Microsoft Azure](https://img.shields.io/badge/Cloud-Microsoft%20Azure-0078D4?style=for-the-badge&logo=azure)](https://azure.microsoft.com/)
[![IaaS](https://img.shields.io/badge/Service-IaaS-005C94?style=for-the-badge)](https://azure.microsoft.com/overview/what-is-iaas/)
[![PowerShell](https://img.shields.io/badge/Automation-PowerShell-5391F5?style=for-the-badge&logo=powershell)](https://docs.microsoft.com/en-us/powershell/azure/)

## 🌟 Project Summary
This project outlines the design, deployment, and management of a **secure, two-tier personal cloud environment** using **Microsoft Azure Infrastructure as a Service (IaaS)**. The goal was to gain practical, hands-on experience across Azure's core pillars: Compute, Networking, Storage, Security, Configuration Management, and Automation.

The final architecture features a protected backend server (`VM-DB`), only accessible via a hardened **Jumpbox** (`VM-Lab`). The solution includes essential configuration of Windows Server roles (IIS, DHCP, DNS), **PowerShell automation**, robust access control (IAM, SAS), and full implementation of **Azure Backup and Monitoring**.

---

### 🎯 Key Objectives Achieved
* Deployed and secured a **two-tier segmented network** (Frontend/Backend).
* Configured a secure **Jumpbox** model using **NSGs** to protect backend resources.
* Implemented **storage expansion and snapshots** for reliability. 
* Configured essential Windows Server roles (**IIS, DHCP, and DNS**) for administrative services.
* Utilized **PowerShell** for VM lifecycle management and **disk provisioning automation**.
* Secured Storage Account access using **IAM (RBAC), SAS, and Access Keys**.
* Established robust **BCDR and Governance** through Azure Backup, Monitoring, and Resource Tags.

---

## 🏗️ Architecture Overview

The environment utilizes a classic tiered design to separate administrative access from sensitive data resources, enhancing security and manageability.

### 🖼️ Topology Diagram (Conceptual)
<img width="1199" height="713" alt="Deployment drawio" src="https://github.com/user-attachments/assets/45f26c77-627a-4c49-8a62-20452e760760" />


### 🧩 Environment Components

| Category | Component Name(s) | Purpose & Key Detail |
| :--- | :--- | :--- |
| **Resource Group** | `myVm_group` | Logical container for unified lifecycle management in **France Central**. |
| **Networking** | `VNet-Lab` (10.0.0.0/16) | Isolated private network with segmented address space. |
| **Subnets** | `Frontend` (10.0.1.0/24) | For public-facing/jumpbox VMs (`VM-Lab`). |
| | `Backend` (10.0.2.0/24) | For protected, private resources (`VM-DB`). **No Public IP**. |
| **Compute (VMs)** | `VM-Lab` (Jumpbox) | **Standard B1ms** Windows Server 2025. Administrative gateway. |
| | `VM-DB` (Database) | Protected server. Access restricted to the Jumpbox. |
| **Security** | **NSG-Frontend** | Controls RDP access (3389) and enforces the secure access model. |
| **Storage** | **Standard SSD (LRS)** | Reliable managed disk type for all volumes. |
| **Backup** | `RSV-Lab` | Recovery Services Vault storing protected backups for `VM-DB`. |
| **Governance** | Tags | Applied for **Cost Tracking** (`Project`, `Environment`, `Owner`). |

---

## 🪜 Detailed Deployment and Configuration Guide

### 1. 🌐 Networking Foundation and Compute Deployment
**Goal:** Establish the network structure and deploy  Virtual Machines.

1.  **Resource Group Creation:** Created `myVm_group` in **France Central** to contain all resources.
2.  **Virtual Network (VNet):** Deployed `VNet-Lab` with the address space **10.0.0.0/16**.
3.  **Subnet Segmentation:** Added the **`Frontend` (10.0.1.0/24)** and **`Backend` (10.0.2.0/24)** subnets for tier separation.
4.  **VM Deployment:**
    * **`VM-Lab` (Jumpbox):** Deployed a **Standard B1ms** Windows Server 2025 into the `Frontend` subnet. **Public IP** enabled for initial RDP access.
    * **`VM-DB` (Database Server):** Deployed into the **`Backend`** subnet with **No Public IP**, ensuring it's isolated from direct internet access.

### 2. 🛡️ Network Security and Tier Isolation
**Goal:** Enforce least-privilege network access to secure the back-end VM.

1.  **NSG Configuration:** Created a **Network Security Group (NSG)** and associated it with the relevant NICs/Subnets.
2.  **RDP Access Control:** Configured two specific inbound rules:
    * **Rule 1 (Internet to Jumpbox):** Allowed RDP (TCP/3389) only to `VM-Lab`'s Public IP.
    * **Rule 2 (Jumpbox to DB):** Created an internal rule allowing RDP traffic *from* the **Frontend Subnet (Source: 10.0.1.0/24)** *to* the **Backend Subnet (Destination: 10.0.2.0/24)**.
3.  **Security Posture:** Applied a low-priority **Deny-All-Inbound** rule for extra hardening, relying solely on the explicit Allow rules.

### 3. ⚙️ Configuration Management and PowerShell Automation
**Goal:** Configure essential server roles and automate routine disk provisioning tasks.

1.  **Server Role Installation:** Connected to `VM-Lab` and used **Server Manager** to install and configure:
    * **Internet Information Services (IIS):** Used for testing web connectivity and future application hosting.
    * **DHCP and DNS:** Configured these roles to simulate a realistic domain administrative environment.
2.  **PowerShell Automation:** Created scripts for two critical automation tasks:
    * **VM Lifecycle:** Used `Start-AzVM` and `Stop-AzVM` for efficient resource management.
    * **Disk Provisioning:** Scripted the process for the new data disk: `Initialize-Disk -PartitionStyle GPT`, `New-Partition`, and `Format-Volume -FileSystem NTFS -DriveLetter D`.

### 4. 💾 Storage Management and Secure Access
**Goal:** Expand VM storage, implement quick restore options, and configure robust storage access control.

1.  **Data Disk Setup:** Attached a **10 GB Standard SSD** data disk to a VM and utilized the **PowerShell script** (from Step 3) to initialize and mount it as the **D:** volume.
2.  **Snapshot:** Created an **Incremental Snapshot** of the data disk via the Azure Portal for reliable point-in-time recovery.
3.  **Storage Account Access Control:** Secured access to the storage account (`storagelab<initials>`) using a layered security model:
    * **IAM (RBAC):** Assigned specific Azure roles (e.g., *Storage Blob Data Reader*) to identity accounts.
    * **Shared Access Signature (SAS):** Generated restricted tokens to grant temporary, granular access to specific storage resources (e.g., a single container).
    * **Access Keys:** Utilized for administrative, legacy access (documented as high-privilege).

### 5. 📦 Business Continuity (BCDR) and Governance
**Goal:** Implement full backup coverage, proactive monitoring, and organizational governance.

1.  **Backup Implementation:** Created a **Recovery Services Vault (`RSV-Lab`)** and enabled **Azure Backup** for the critical `VM-DB` using a daily policy.
2.  **Proactive Monitoring:** Configured **Azure Monitor** on `VM-DB`:
    * **Metric:** CPU Percentage.
    * **Alert Rule:** Triggered an email notification if CPU usage exceeded **80% for 5 minutes**.
3.  **Governance:** Applied mandatory **Resource Tags** across **all** resources in `myVm_group` for cost allocation and organization:
    * `Project: PersonalCloudLab`, `Environment: Lab`, `Owner: Louange`.

---

## 📈 Skills and Technologies Mastered

| Category | Core Skills Demonstrated | Technologies Used |
| :--- | :--- | :--- |
| **Cloud Fundamentals** | Resource Lifecycle Management, IaaS Deployment, Multi-VM Orchestration | Azure Resource Manager (ARM), Azure Portal |
| **Networking & Security** | Virtual Networking (VNet/Subnets), Network Segmentation, Secure Jumpbox Model | **Network Security Groups (NSGs)** |
| **Configuration** | **Windows Server Role Installation (IIS, DHCP, DNS)**, Server Manager, Disk Management | IIS, DHCP, DNS, Server Manager |
| **Automation** | **VM Management Scripting**, **Disk Provisioning Automation** | **PowerShell** |
| **Access Control** | **Role-Based Access Control (RBAC)**, Shared Access Policies | **Azure IAM**, **Shared Access Signature (SAS)**, Access Keys |
| **Data & Storage** | Managed Disks, Volume Initialization, Snapshots, Object Storage | Standard SSD, Azure Blob Storage |
| **BCDR & Ops** | Backup Policy Configuration, Metric-Based Alerting, Resource Tagging | Azure Backup, Azure Monitor, Azure Alerts |

---

## 🖼️ Project Visuals
*To view images of components Click on their appropriate link .*

| Component | Description | Screenshot (Conceptual Link) |
| :--- | :--- | :--- |
| **Resource Group** | Consolidated view of all resources in `myVm_group`. | [Resource Group Screenshots](./images/resource-group/)|
| **Virtual Network** | `VNet-Lab` structure showing the secure Frontend and Backend subnets. | [VNet-Lab with Subnets](./images/vnet-lab/) |
| **Disk Management** | The new data disk initialized and mounted as the D: volume. | [Disk Management Console](./images/disk-management/)|
| **Azure Backup** | `VM-DB` protected by the Recovery Services Vault. | [Azure Backup Status](./images/backup/)|
| **Monitoring Alert** | CPU Alert Rule configured for performance management. | [Monitoring/Alerts Configuration](./images/monitoring/) |

---

## 🚀 Next Projects on my Learning List
* **Project 2 (PaaS Focus):** Deploy a scalable web application using **Azure App Service** and connect it to a managed database (Azure SQL).
* **Advanced IaC:** Translate this entire detailed deployment into repeatable Infrastructure-as-Code (IaC) templates using **Bicep** or **Terraform**.
* **Project 3:** Create a **Static Web Portfolio** using **Azure Storage + CDN**. 
