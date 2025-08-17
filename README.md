#  EndeavourOS Installation Guide

**Author:** [Hasnain Ahmed Alve/has9alv3]

**Date:** August 17, 2025

**Hardware:** HP EliteBook 845 14 inch G10 Notebook PC (AMD Ryzen 7 PRO 7840U w/ Radeon 780M)

**Operating System:** EndeavourOS (Arch Linux)

**Desktop Environment:** KDE Plasma

**Filesystem:** Btrfs with System Snapshots

---

## Introduction

This document provides a complete walkthrough for installing and configuring EndeavourOS on a modern AMD-based laptop. The guide covers pre-installation steps, a detailed installation process, and extensive post-installation configuration to create a stable, power-efficient, and resilient system with automated snapshot recovery, fingerprint login, and optimized battery life.

---

## Table of Contents

1.  [Phase 1: Pre-Installation Preparations](#phase-1-pre-installation-preparations)
2.  [Phase 2: UEFI/BIOS Configuration](#phase-2-uefibios-configuration)
3.  [Phase 3: The Installation Process](#phase-3-the-installation-process)
4.  [Phase 4: Post-Installation Configuration](#phase-4-post-installation-configuration)
    * [4.1 Initial System Update](#41-initial-system-update)
    * [4.2 Battery Life Optimization](#42-battery-life-optimization)
    * [4.3 Btrfs Snapshot & Boot Menu Integration](#43-btrfs-snapshot--boot-menu-integration)
    * [4.4 Fingerprint Reader Setup](#44-fingerprint-reader-setup)
5.  [Appendix: Useful Information](#appendix-useful-information)

---
## Phase 1: Pre-Installation Preparations

### 1.1. Data Backup
Ensure all critical personal data from any existing operating system is backed up to an external drive or cloud service. This process will wipe the internal drive.

### 1.2. Create a Bootable USB
1.  Download the latest EndeavourOS ISO from the official website.
2.  Download **Ventoy** and install it to a USB drive (8GB or larger).
3.  Copy the downloaded EndeavourOS ISO file directly onto the Ventoy USB drive.

---
## Phase 2: UEFI/BIOS Configuration

1.  Power on the laptop and immediately press **F10** to enter the BIOS setup.
2.  Navigate to security/boot options and **Disable Secure Boot**.
3.  Ensure the boot order is set to boot from the USB Drive first.
4.  Save changes and exit.

---
## Phase 3: The Installation Process

### 3.1. Boot and Prepare the Live Environment
1.  Boot the laptop from the prepared USB drive. Select the EndeavourOS ISO from the Ventoy menu.
2.  Once the live desktop loads, connect to the internet. **Note:** If Wi-Fi proves unstable during installation, the most reliable method is to use **USB tethering** with a smartphone.
3.  Open a terminal and refresh the mirror list to ensure a fast and successful installation:
    ```bash
    reflector-simple
    ```
### 3.2. Running the Calamares Installer
1.  Launch the installer from the Welcome application and select the **Online** installation method.
2.  **Location:** Confirm your geographical region.
3.  **Keyboard:** Set the layout to **`Spanish (no dead keys)`**.
4.  **Partitions:**
    * Select the **"Erase disk"** option for a simple and clean installation.
    * Set the filesystem dropdown to **`btrfs`**.
    * Set the swap dropdown to **`Swap (with Hibernate)`**. This is ideal for laptops.
5.  **Desktop:** Select **KDE Plasma**.
6.  **Packages:** Proceed with the default selections.
7.  **Bootloader:** Confirm **GRUB** is selected.
8.  **Users:** Create your user account and password.
9.  **Summary:** Review the planned actions and click **Install**.
10. **Reboot:** After the installation completes, reboot the system and **remove the USB drive** when prompted.

---
## Phase 4: Post-Installation Configuration

### 4.1. Initial System Update
Open the terminal (Konsole) and run a full system update:
```bash
sudo pacman -Syu
```

### 4.2. Battery Life Optimization
1.  **Install Power Management Tools:**
    ```bash
    sudo pacman -S tlp powertop
    ```
    When prompted that `tlp` conflicts with `power-profiles-daemon`, approve the removal of `power-profiles-daemon` by typing `y`.

2.  **Enable TLP:**
    ```bash
    sudo systemctl enable --now tlp.service
    ```

3.  **Apply Powertop Tunables:**
    ```bash
    sudo powertop --auto-tune
    ```

4.  **Reboot** the system to apply all changes.

### 4.3. Btrfs Snapshot & Boot Menu Integration
This setup provides a powerful system recovery feature.
1.  **Install Necessary Packages:** Install Timeshift, the GRUB integration script, and its required dependency.
    ```bash
    sudo pacman -S timeshift grub-btrfs inotify-tools
    ```
2.  **Configure Timeshift:**
    * Launch Timeshift.
    * Select **Btrfs** as the snapshot type.
    * Confirm the snapshot location.
    * Set a schedule (e.g., keep 3-5 Daily snapshots).
    * Under the "Users" tab, ensure all users are set to **"Exclude All"**.
3.  **Fix the `grub-btrfsd` Service:** The default service configuration looks for snapshots in the wrong directory for Timeshift. Create a service override to fix this.
    * Create the override directory:
        ```bash
        sudo mkdir -p /etc/systemd/system/grub-btrfsd.service.d/
        ```
    * Create and edit a new configuration file:
        ```bash
        sudo nano /etc/systemd/system/grub-btrfsd.service.d/override.conf
        ```
    * Paste the following content into the file:
        ```ini
        [Service]
        ExecStart=
        ExecStart=/usr/bin/grub-btrfsd --syslog --timeshift-auto
        ```
    * Save the file and exit (`Ctrl+O`, `Enter`, `Ctrl+X`).
4.  **Reload and Enable the Service:**
    ```bash
    sudo systemctl daemon-reload
    sudo systemctl enable --now grub-btrfsd.service
    ```
5.  **Perform an initial manual GRUB update:**
    ```bash
    sudo grub-mkconfig -o /boot/grub/grub.cfg
    ```

### 4.4. Fingerprint Reader Setup
1.  **Identify Hardware:** The device ID for the fingerprint reader on this model is `04f3:0c82` (Elan Microelectronics Corp.).
2.  **Install Software:**
    ```bash
    sudo pacman -S fprintd
    ```
3.  **Enroll Fingerprint:**
    ```bash
    fprintd-enroll
    ```
    Follow the prompts to scan your finger.
4.  **Enable in KDE:** Go to `System Settings` > `Users`, click your user, and select **"Configure Fingerprint Authentication..."**.
5.  **(Optional) Enable for `sudo`:**
    * Edit the file: `sudo nano /etc/pam.d/sudo`
    * Add this line to the very top: `auth sufficient pam_fprintd.so`

---
## Appendix: Useful Information

* **GRUB Update Command:** The correct command to update the GRUB menu on Arch-based systems is `sudo grub-mkconfig -o /boot/grub/grub.cfg`, not `update-grub`.
* **Future RAM Upgrade:** The system was installed with 16GB of RAM and hibernation enabled. If the RAM is upgraded to 32GB, the swap file must be resized for hibernation to continue working. This can be done by turning off the swap, deleting the old `/swapfile`, creating a new larger one (e.g., 33GB) with `sudo fallocate -l 33G /swapfile`, and re-initializing it with `sudo mkswap /swapfile` before turning it on with `sudo swapon /swapfile`.
