# SysAdmin Lab Environment: Rocky Linux 9 on VirtualBox

This repository contains the technical documentation, network architecture details, and initialization steps used to deploy and configure a local laboratory environment tailored for enterprise systems administration.

The primary objective of this project is to simulate a secure, scalable production environment using **Rocky Linux 9** hosted on **Oracle VM VirtualBox**.

---

## 🛠️ Lab Architecture & Network Topology

To simulate an enterprise environment while maintaining direct accessibility, the laboratory network is configured as follows:

1. **Bridged Adapter:** The virtual machine’s network interface card (NIC) is bridged directly to the host's physical network. This allows the VM to receive an IP address (`192.168.100.0/24` subnet) from the local DHCP server, enabling seamless inbound and outbound connectivity, SSH access from the host machine, and internet access for package management via `dnf`.

### Instance Specifications
* **OS:** Rocky Linux 9 (Minimal ISO)
* **Hypervisor:** Oracle VM VirtualBox 7.x
* **Base Resources:** 2 vCPUs, 2 GB RAM, 20 GB VDI Dynamic Allocation
* **Target IP Address:** 192.168.100.105 (Configured via DHCP/Static Reservation

---

## 🚀 Step-by-Step Deployment Guide

### 1. Hypervisor Configuration
* Created the virtual machine using dynamic storage allocation to optimize host disk space.
* Pre-configured network adapters (NAT + Host-Only) prior to the initial boot.

### 2. Rocky Linux Base Installation
* Standard server-focused partitioning (or LVM setup, if applicable).
* Disabled direct `root` login during the installation wizard as a baseline security best practice.
* Created a dedicated system administrator user with full `sudo` privileges.

### 3. Initial Post-Installation Hardening
Once the base OS is up and running, the following essential maintenance and security tasks are executed:

```bash
# 1. Update the system to the latest stable release
sudo dnf update -y

# 2. Verify network interface configuration and IP assignment
# (In this environment, the interface receives 192.168.100.105 from the local subnet)
ip a show enp0s3

# 3. Secure the SSH service
# Edit /etc/ssh/sshd_config to enforce security baselines:
# - PermitRootLogin no
# - PasswordAuthentication yes (Temporary, moving towards SSH keys)
sudo systemctl restart sshd
```


## 💾 Storage Management: Logical Volume Manager (LVM) Lab

This section documents the live deployment, formatting, and online expansion of storage using **LVM (Logical Volume Manager)** on a running Rocky Linux 9 instance without service interruption.

### 1. Storage Architecture Overview
A secondary 5 GB virtual disk (`/dev/sdb`) was attached to the instance to simulate an enterprise storage expansion scenario, structured across the three core LVM layers:
* **Physical Volume (PV):** Raw disk initialization.
* **Volume Group (VG):** Aggregation of physical volumes into a common storage pool (`vg_datos`).
* **Logical Volume (LV):** Flexible allocation sliced from the pool (`lv_reportes`), formatted with the **XFS** filesystem.

---

### 2. Step-by-Step Implementation & Configuration

#### Phase A: Base LVM Provisioning & Mounting
1. **Initialize the Physical Volume:**
   ```bash
   sudo pvcreate /dev/sdb
Create the Volume Group:

```bash
sudo vgcreate vg_datos /dev/sdb
```
Allocate the Logical Volume (Initial 4 GB):

```bash
sudo lvcreate -L 4G -n lv_reportes vg_datos
```
Format with Enterprise XFS Filesystem:

```bash
sudo mkfs.xfs /dev/vg_datos/lv_reportes
```

Establish Point of Mount & Assign Ownership:
```bash
sudo mkdir -p /data/reportes
sudo mount /dev/vg_datos/lv_reportes /data/reportes
sudo chown -R soporte:soporte /data/reportes
```
Phase B: Online Filesystem Expansion (On-the-Fly)
To demonstrate LVM's capability to scale storage under production-like demands without unmounting the filesystem or causing downtime, the volume was extended using the remaining unallocated space in the VG:


## Extend the Logical Volume and scale the XFS filesystem simultaneously
```bash
sudo lvextend -l +100%FREE -r /dev/vg_datos/lv_reportes
```
### 3. Verification & Verification Metrics
Data Integrity Check: Verified that user data and file permissions remained completely unaffected post-expansion via standard I/O operations.

Storage Allocation Audit: Confirmed the successful runtime expansion from 4.0 GB to 5.0 GB utilizing the storage file-system monitoring utility:

```bash
df -h /data/reportes
```
#### 📸 Deployment Evidence

Here is the confirmation of the storage layout and runtime expansion from the hypervisor and guest OS:

![VirtualBox Storage Configuration](Screenshot 2026-06-04 171450.png)
*Figure 1: Storage controller configuration in VirtualBox showing the secondary 5GB VDI attached.*

![Storage Verification Command](Screenshot 2026-06-04 171450.png)
*Figure 2: Execution of 'df -h' confirming the online expansion to 5.0 GB.*

### 🛠️ Troubleshooting & Lessons Learned

During the deployment, a standard real-world permissions issue was encountered and resolved:

* **Issue: `Permission denied` on initial file write operations**
  * *Context:* After successfully mounting the logical volume to `/data/reportes`, attempting to create a test file (`archivo_importante.txt`) as a standard non-root user (`soporte`) resulted in a bash permission failure.
  * *Root Cause Analysis:* The mount point directory was initially provisioned using `sudo mkdir`, assigning explicit ownership to the `root` user by default, thereby locking out standard user write privileges.
  * *Resolution:* Applied system administration best practices by recursively modifying the directory's user and group ownership to match the local operations user:
    ```bash
    sudo chown -R soporte:soporte /data/reportes
    ```
    This successfully restored standard I/O operations without relying on loose security workarounds like `chmod 777`.
