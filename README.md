# Setting Up Node Cluster for Homelab

## Table of Contents

- [Phase 1: Physical Layer for Virtualization](#phase-1-setting-up-the-physical-layer-for-virtualization)
  - [RAM Upgrade](#ram-upgrade)
  - [Headless Setup](#headless-setup)
  - [Why an Intel Mac Mini?](#why-an-intel-mac-mini)
- [Phase 2: Headless Command & Control](#phase-2-command--control-headless-setup-between-intel-mac-mini-and-macbook-pro)
  - [SSH Setup](#1-ssh-secure-shell)
  - [VNC Setup](#2-vnc-virtual-network-computing)
- [Phase 3: Virtualization Setup](#phase-3-virtualization-setup)
  - [Installing UTM](#1-installing-utm-hypervisor)
  - [Installing Ubuntu Server ISO](#2-installing-ubuntu-server-iso-on-the-mac-mini)
  - [Spinning Up a VM](#3-spinning-up-our-first-vm-in-utm)

---

## Phase 1: Setting Up the Physical Layer for Virtualization

The first phase involves physically preparing the 2018 Intel Mac Mini with 8GB RAM for virtualization. This includes:

### 💾 RAM Upgrade

We upgraded the RAM on the Intel Mac mini from **8GB to 32GB** using Crucial RAM. This allows for smoother operation of multiple VMs and more efficient resource management.
Two Crucial 16GB DDR4 3200 MHz (PC4-25600) SODIMM 260-Pin Memory sticks were manually installed.

### 🖥️ Headless Setup

The goal was to run the Mac mini headless—without peripheral devices like a dedicated monitor, mouse, or keyboard.

However, **macOS doesn't display video output without a connected screen**, so we used an **HDMI emulator (display emulator dongle)** to trick macOS into thinking a display is connected. This enables screen sharing and full resolution via remote access.

### 🤔 Why an Intel Mac Mini?

While newer M-series Mac minis are more powerful, they are **less flexible** for virtualization, especially with third-party hypervisors like VMware Fusion or VirtualBox. Intel chips provide broader compatibility, especially with tools like **UTM**, a lightweight and open-source virtualization solution.

---

## Phase 2: Command & Control Headless Setup Between Intel Mac Mini and MacBook Pro

To operate the Mac mini without direct peripherals, we needed two key remote control protocols:

### 1. 🔐 SSH (Secure Shell)

**SSH** allows secure command-line access to the Mac mini from another device (in this case, the MacBook Pro).

#### Setting up SSH on the Mac Mini:

- Enable SSH:
  ```bash
  sudo systemsetup -setremotelogin on
  ```
- Check if it's running:
  ```bash
  sudo systemsetup -getremotelogin
  ```
- From the MacBook Pro, connect via:
  ```bash
  ssh username@<mac_mini_ip_address>
  ```
- To find the IP address (look for en0):
  ```bash
  ifconfig -a
  ```

### 2. 🖥️ VNC (Virtual Network Computing)

VNC allows GUI-based control of the Mac mini from the MacBook Pro.

#### Setting up VNC:

- Enable **Screen Sharing** on the Mac mini:

  - Go to: System Preferences > Sharing > Screen Sharing ✅
  - If Advanced Remote Management is preferred, make sure to set a password via terminal with the bash command

  ```bash
  sudo /System/Library/CoreServices/RemoteManagement/ARDAgent.app/Contents/Resources/kickstart \
  -configure -clientopts -setvnclegacy -vnclegacy yes \
  -configure -clientopts -setvncpw -vncpw $(echo -n 'yourpassword' | xxd -p) \
  -restart -agent
  ```

- then run this commoand via Terminal to restart vnc once more and make sure the password is changed:

  ```bash
  sudo /System/Library/CoreServices/RemoteManagement/ARDAgent.app/Contents/Resources/kickstart \
    -activate -configure -access -on -restart -agent -privs -all
  ```

- Ensure the VNC port (5900) is open and listening:

  ```bash
  sudo lsof -iTCP -sTCP:LISTEN | grep 5900
  ```

- Connect using the built-in Screen Sharing app on macOS:

  ```bash
  open vnc://<mac_mini_ip_address>
  ```

---

## Phase 3: Virtualization Setup

### 1. 🧱 Installing UTM (Hypervisor)

We originally tested VMware Fusion but switched to **UTM** due to licensing and future-proofing concerns (aka the Broadcom acquisition).

- UTM is available at: [https://mac.getutm.app/](https://mac.getutm.app/)
- Installation is drag-and-drop from the downloaded `.dmg` file to Applications.

### 2. 📦 Installing Ubuntu Server ISO on the Mac Mini

We used `scp` to securely transfer the ISO to the Mac mini via SSH.

#### Download ISO:

- Ubuntu Server ISO: [https://ubuntu.com/download/server](https://ubuntu.com/download/server)
- Choose: **Ubuntu Server 64-bit (x86\_64)**

#### Example `scp` command:

```bash
scp ~/Downloads/ubuntu-xx.xx-live-server-amd64.iso username@<mac_mini_ip_address>:/Users/username/Desktop/
```

### 3. 🚀 Spinning Up Our First VM in UTM

#### Steps:

1. Open UTM and click **Create a New Virtual Machine**
2. Choose **Virtualize > Linux**
3. Attach the Ubuntu Server ISO as a boot medium
4. Set up basic resources:
   - **RAM**: 2–4 GB
   - **CPU**: 1–2 cores
   - **Disk size**: 20 GB or more
5. Boot the VM and follow the Ubuntu Server installation wizard
6. After setup, you’ll have a lightweight, headless Ubuntu VM running on your Mac mini

---

> 📌 **Next Steps**: We'll expand the cluster, we'll add the following:
>  A) Configured firewall
>  B) Docker & Kubernetes for web applications
>  C) Kali Linux VM
>  D) IDS and IPS 

