# Oracle Linux 7.9 Installation & System Configuration Guide

This guide details the step-by-step installation and system configuration procedure for **Oracle Linux 7.9** tailored for enterprise Oracle Database deployments. It provides precise instructions for installer boot, software profile selection, LVM storage partitioning, static network parameters, and post-installation kernel verification.

---

## 1. Installation Media Boot & Language Selection

To initiate the operating system deployment, boot the physical host or virtual machine using the official Oracle Linux 7.9 ISO image.

1. **Boot Selection**: At the boot loader screen, select **Install Oracle Linux 7.9** and press `Enter`.
2. **Installer Language**: On the welcome screen (*Welcome to Oracle Linux 7.9*), select the preferred language for the installation process (e.g., *English* or *Portuguese (Brazil)*) and click **Continue**.

---

## 2. Localization & Base Software Selection

From the **Installation Summary** menu, configure the initial system parameters before defining storage and network options.

### Date & Time Configuration
* Select **Date & Time**.
* Adjust the timezone according to the client's infrastructure location (e.g., `Americas/Sao_Paulo` for Brasilia timezone).
* Verify system clock accuracy and click **Done**.

### Software Selection Profile
To maintain a lightweight host footprint while including necessary compilation headers for Oracle Database software pre-requisites, select the following software environment:

* Navigate to **Software Selection**.
* Under **Base Environment**, select **Minimal Install**.
* Under **Add-ons for Selected Environment**, select **Development Tools**.
* Click **Done** to return to the summary screen.

---

## 3. Storage Allocation & Manual LVM Partitioning

Proper disk layout and file system selection are critical for database performance and maintenance. The installation uses Logical Volume Manager (LVM) on a standard 80 GiB virtual disk example (`/dev/sda`).

### Initial Storage Selection
1. Click **Installation Destination**.
2. Under **Local Standard Disks**, select the target installation disk (e.g., `VMware Virtual disk sda - 80 GiB`).
3. Under **Other Storage Options**, mark the radio button **I will configure partitioning**.
4. Click **Done** to enter the Manual Partitioning interface.

### LVM Layout Creation & Mount Configuration
1. In the **Manual Partitioning** screen, select **LVM** from the partitioning scheme dropdown.
2. Click **Click here to create them automatically** to generate initial volume defaults.
3. Modify the automatically created mount points to match the following enterprise database layout:

| Mount Point | Volume Name | Target Capacity | File System Type | Device / Volume Group | Description |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `/boot` | `sda1` | `1024 MiB` | `ext4` | Standard Partition | Boot loader partition |
| `/` | `ol-root` | `15 GiB` | `ext4` | LVM (`ol` VG) | Operating system root file system |
| `swap` | `ol-swap` | `8188 MiB` (`8 GiB`) | `swap` | LVM (`ol` VG) | System swap memory space |
| `/u01` | `ol-u01` | Remaining Space (~`56 GiB`) | `xfs` | LVM (`ol` VG) | Oracle Software & Data mount point |

#### Detailed Mount Point Adjustment Steps:
* **Root Partition (`/`)**: Select `/`, set **Desired Capacity** to `15 GiB`, set **Device Type** to `LVM`, and set **File System** to `ext4`.
* **Swap Partition**: Select `swap` and set **Desired Capacity** to `8188 MiB` (8 GiB).
* **Oracle Mount Point (`/u01`)**: Select the default `/home` partition, change its **Mount Point** name to `/u01`, assign all remaining disk space (~`56 GiB`), set **Device Type** to `LVM`, set **File System** to `xfs`, and label/name it `u01`.
* Click **Done** and accept the changes to write the partition table.

---

## 4. Kernel Crash Dump (KDUMP) Configuration

To avoid reservation of dedicated system memory for kernel crash dumps in dedicated database hosts, KDUMP is disabled by default during deployment.

1. Click **KDUMP** on the Installation Summary screen.
2. Uncheck the **Enable kdump** checkbox.
3. Click **Done**.

---

## 5. Network Interface, Hostname, and Static IPv4 Setup

Configuring network parameters ensures proper domain resolution and static connectivity required by Oracle Listener and Interconnect services.

### Hostname & Interface Activation
1. Select **Network & Host Name**.
2. In the **Host name** field at the bottom, enter the designated host name (e.g., `oracleLinux7`) and click **Apply**.
3. Toggle the network interface switch (e.g., `ens192`) to **ON**.
4. Click **Configure...** in the bottom right corner.

### Connection & Protocol Settings
* **General Tab**: Check the option **Automatically connect to this network when it is available**.
* **IPv4 Settings Tab**:
  * Change **Method** from *Automatic (DHCP)* to **Manual**.
  * Click **Add** and enter the static network parameters:
    * **IP Address**: e.g., `192.168.239.12`
    * **Netmask / Prefix**: `24` (255.255.255.0)
    * **Gateway**: e.g., `192.168.239.1`
    * **DNS Servers**: e.g., `192.168.239.1, 8.8.8.8`
* **IPv6 Settings Tab**:
  * Change **Method** to **Ignore**.
* Click **Save** and then click **Done**.

---

## 6. Security Policy & Administrative Account Configuration

1. **Security Policy**:
   * Click **Security Policy**.
   * Toggle the **Apply security policy** switch to **OFF** (to prevent default Anaconda compliance profile restrictions from locking database installation accounts).
   * Click **Done**.
2. **Start Installation**:
   * Click **Begin Installation** at the bottom right of the summary screen.
3. **User Credentials Setup**:
   * While the installation process runs in the background, click **Root Password**.
   * Enter and confirm the secure `root` password.
   * Click **Done** (if a weak password is specified, click **Done** twice to confirm).

---

## 7. Installation Completion, First Boot, and Verification

Once package installation and post-installation scripts complete, finalize the deployment.

1. **Reboot Host**: Click the blue **Reboot** button at the bottom right.
2. **Kernel Selection at Boot (GRUB)**:
   * During bootloader initialization, select the latest kernel version.
   * Default recommended kernel: `Oracle Linux Server (5.4.17-2011.6.2.el7uek.x86_64 with Unbreakable Enterprise Kernel)`.
3. **Post-Boot Storage Verification**:
   * Log in as `root` via console or SSH.
   * Execute the disk space usage utility to confirm mount point creation and volume capacity:

```bash
df -h
```

#### Expected System Output Sample:

```text
Filesystem            Size  Used Avail Use% Mounted on
devtmpfs              7.9G     0  7.9G   0% /dev
tmpfs                 7.9G     0  7.9G   0% /dev/shm
tmpfs                 7.9G  8.7M  7.9G   1% /run
tmpfs                 7.9G     0  7.9G   0% /sys/fs/cgroup
/dev/mapper/ol-root    15G  1.9G   12G  14% /
/dev/mapper/ol-u01     56G   33M   56G   1% /u01
/dev/sda1             1014M  178M  845M  17% /boot
tmpfs                 1.6G     0  1.6G   0% /run/user/0
```

The system is now fully installed, updated, and ready for Oracle Database pre-installation RPMs (`oracle-database-preinstall-19c`) and database software deployment.
