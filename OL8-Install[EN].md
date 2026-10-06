# Oracle Linux 8.9 Enterprise Installation and Base Configuration Guide

## 1. Overview and Operational Context

This document provides a standardized, production-ready guide for installing and configuring **Oracle Linux 8.9 (x86_64)**. The installation procedure is optimized for enterprise database deployments, application servers, and infrastructure workloads, establishing a lean, high-performance base system configured with Logical Volume Management (LVM), static networking, and the Unbreakable Enterprise Kernel (UEK).

The configuration guidelines outlined in this document reflect best practices for enterprise Linux provisioning, ensuring proper disk layout segregation (specifically reserving `/u01` for Oracle binaries and data files), optimized software package selection, and predictable network behavior.

---

## 2. Media Boot and Installer Initialization

1. Attach or insert the official **Oracle Linux 8.9 ISO** installation media into the target physical server or virtual machine (e.g., VMware vSphere hypervisor).
2. Power on the system and enter the boot menu.
3. Select **Install Oracle Linux 8.9.0** from the GRUB boot menu to launch the Anaconda graphical installer.
4. On the **Welcome to Oracle Linux 8.9** screen:
   - Select the preferred installation language (e.g., **Portuguese (Brazil)** or **English (United States)**).
   - Click **Continue** to proceed to the main **Installation Summary** dashboard.

---

## 3. Localization and Time Zone Setup

From the **Installation Summary** screen, configure the localization parameters:

* **Keyboard Layout**: Ensure the primary keyboard layout matches the system console (e.g., `Portuguese (Brazil)`).
* **Language Support**: Confirm regional language support settings.
* **Time & Date**:
  - Select the region and city aligned with the client infrastructure standard (e.g., **Americas/Sao_Paulo** for Brasilia time zone).
  - Verify that Network Time Protocol (NTP) synchronization is active if network access is present.

---

## 4. Software Selection Profile

Oracle Linux 8 introduces modular Application Streams (AppStreams). For enterprise database and middleware environments, a minimal software footprint combined with core development tools provides maximum security and operational efficiency.

1. Click **Software Selection** on the Installation Summary screen.
2. Under **Base Environment**, select **Minimal Install** (*Instalações Mínimas*).
3. Under **Additional Software for Selected Environment**, select **Development Tools** (*Ferramentas de Desenvolvimento*).
4. Click **Done** to return to the summary screen.

> **Technical Note**: Selecting Minimal Install minimizes the system attack surface and resource overhead by excluding unneeded graphical user interfaces and daemon services. The Development Tools package group installs essential compilers, header files, and build utilities required for database driver compilation and kernel module management.

---

## 5. Storage Target and LVM Manual Partitioning Strategy

Proper partitioning is critical to isolate operating system logs, core binaries, swap space, and application storage paths.

### 5.1 Target Disk Selection

1. Click **Installation Destination** on the Installation Summary dashboard.
2. Under **Local Standard Disks**, select the intended system installation disk (e.g., `VMware Virtual disk sda` with 80 GiB capacity).
3. Under **Storage Configuration**, select the **Custom** (*Personalizada*) radio button.
4. Click **Done** to enter the Manual Partitioning interface.

### 5.2 LVM Partitioning Layout Setup

1. In the **Manual Partitioning** window, set the partitioning scheme dropdown to **LVM** (Logical Volume Manager).
2. Click the link **Click here to create them automatically** to generate a baseline partition set.
3. Modify the automatically created mount points to match the exact enterprise storage specifications below:

| Mount Point | Volume Name | File System | Size / Capacity | Description & Purpose |
| :--- | :--- | :--- | :--- | :--- |
| `/boot/efi` | `sda1` | `EFI System Partition` | **600 MiB** | UEFI Bootloader partition |
| `/boot` | `sda2` | `xfs` / `ext4` | **1024 MiB** | Linux Kernel and initramfs storage |
| `/` (Root) | `ol-root` | **`ext4`** | **15 GiB** | Operating system core directory tree |
| `swap` | `ol-swap` | `swap` | **8 GiB** | Virtual memory swap space |
| `/u01` | `ol-u01` | **`xfs`** | **Remaining Space (~55.41 GiB)** | Oracle software installation and database path |

### 5.3 Detailed Partition Configuration Steps

1. Select the `/` (Root) partition:
   - Mount Point: `/`
   - Desired Capacity: `15 GiB`
   - Device Type: `LVM`
   - File System: **`ext4`**
2. Select the `swap` partition:
   - Mount Point: `swap`
   - Desired Capacity: `8 GiB`
3. Select the `/home` partition (if generated automatically):
   - Change Mount Point to: **`/u01`**
   - Desired Capacity: Set to consume all remaining disk space (e.g., `55.41 GiB` on an 80 GiB disk).
   - Device Type: `LVM`
   - File System: **`xfs`**
   - Label: `u01`
   - Name: `u01`
4. Click **Done**, review the **Summary of Changes** dialog, and click **Accept Changes**.

---

## 6. System Services and Security Hardening

### 6.1 KDUMP Deactivation

KDUMP is a kernel crash dumping mechanism. In managed virtualized environments or dedicated database servers, disabling KDUMP frees up system memory that would otherwise be reserved exclusively for crash dumps.

1. Click **KDUMP** on the Installation Summary screen.
2. Uncheck the **Enable kdump** (*Ativar kdump*) checkbox.
3. Click **Done**.

### 6.2 Security Policy Profiles

1. Click **Security Policy** on the Installation Summary screen.
2. Toggle off **Apply security policy** (*Aplicar política de segurança*).
3. Confirm that no security profile (e.g., ANSSI-BP-028) is selected during initial OS installation to avoid unexpected permission restrictions during automated provisioning.
4. Click **Done**.

---

## 7. Network Interface and Static Host Configuration

Predictable static networking is mandatory for database servers and enterprise network nodes.

1. Click **Network & Host Name** on the Installation Summary screen.
2. In the **Host Name** field at the bottom left, enter the fully qualified hostname (e.g., `oracleLinux8`) and click **Apply**.
3. Toggle the switch for the active network interface (e.g., `Ethernet (ens192)`) to **ON**.
4. Click the **Configure...** button in the lower right corner to open connection settings.

### 7.1 General Settings
* Navigate to the **General** tab.
* Check the box: **Connect automatically with priority** (*Conectar automaticamente com prioridade*).

### 7.2 IPv4 Addressing
* Navigate to the **IPv4 Settings** tab.
* Change **Method** from *Automatic (DHCP)* to **Manual**.
* Click **Add** under Addresses and specify the static IP details:
  - **IP Address**: `192.168.239.13` (or customer-allocated IP)
  - **Netmask**: `24` (or `255.255.255.0`)
  - **Gateway**: `192.168.239.1`
* In the **DNS servers** field, enter primary and secondary DNS resolvers separated by commas:
  - `192.168.239.1, 8.8.8.8`

### 7.3 IPv6 Configuration
* Navigate to the **IPv6 Settings** tab.
* Change **Method** to **Ignore** (*Ignorar*).
* Click **Save** and then click **Done**.

---

## 8. User Authentication and Installation Execution

1. Under **User Settings** on the Installation Summary dashboard, click **Root Password**.
2. Enter a robust administrative password in the **Root Password** and **Confirm** fields.
3. Click **Done** (press twice if the password is flagged as simple/short).
4. Verify that all required sections show completed status on the main dashboard.
5. Click **Begin Installation** (*Iniciar a instalação*) to start package payload deployment and storage formatting.

---

## 9. Post-Installation Reboot and Kernel Verification

1. Once package installation completes, click **Reboot System** (*Reiniciar o sistema*).
2. During system POST and GRUB initialization, the boot menu presents available kernels:
   - **Oracle Linux Server 8 (5.15.0-200.131.27.el8uek.x86_64)** (Unbreakable Enterprise Kernel - UEK)
   - **Oracle Linux Server 8 (4.18.0-513.5.1.el8_9.x86_64)** (Red Hat Compatible Kernel - RHCK)
3. Select the latest **Unbreakable Enterprise Kernel (UEK)** version (or allow GRUB to auto-boot after the default timeout).
4. Verify system boot completion at the Linux console prompt:
   ```text
   Oracle Linux Server 8.9
   Kernel 5.15.0-200.131.27.el8uek.x86_64 on an x86_64

   oracleLinux8 login:
   ```
5. Log in with `root` credentials to complete post-install database prerequisite tasks (such as `oracle-database-preinstall-19c` package installation).
