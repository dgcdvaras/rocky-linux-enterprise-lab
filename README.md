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
