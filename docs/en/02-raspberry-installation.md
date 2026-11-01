# 💻 Installing Raspberry Pi OS

🌐 **Language:** [🇫🇷 Français](../fr/02-installation-raspberry.md) | 🇬🇧 English

[🏠 Back to Home](../../README.md)

---

## 1. Overview

This chapter explains how to install and configure Raspberry Pi OS on a Raspberry Pi Zero 2 W for use with the Proxmark3 Easy.

The installation is performed without a monitor, keyboard, or mouse. The Raspberry Pi will be managed remotely using SSH.

### Reference Configuration

| Component           | Configuration        |
| ------------------- | -------------------- |
| Raspberry Pi        | Zero 2 W             |
| Operating System    | Raspberry Pi OS Lite |
| Architecture        | 64-bit               |
| Debian Base         | Debian 13 (Trixie)   |
| Desktop Environment | None                 |
| Network Manager     | NetworkManager       |
| Remote Access       | SSH                  |

This configuration matches the system used in this project.

---

## 2. Download Raspberry Pi Imager

Download and install **Raspberry Pi Imager**, the official tool for preparing Raspberry Pi operating system images.

🔗 [Download Raspberry Pi Imager](https://www.raspberrypi.com/software/)

Insert the microSD card into your computer.

A microSD card with at least 16 GB of storage is recommended.

---

## 3. Prepare the microSD Card

Launch Raspberry Pi Imager.

### Step 1 — Select Your Device

Choose:

**Raspberry Pi Zero 2 W**

### Step 2 — Select the Operating System

Choose:

**Raspberry Pi OS Lite (64-bit)**

This version does not include a graphical desktop and is ideal for remote operation.

### Step 3 — Select Storage

Select your microSD card.

⚠️ Warning: all existing data on the card will be erased.

---

## 4. Initial Configuration

Before writing the image, Raspberry Pi Imager allows you to customize the operating system.

### Hostname

Set the hostname to:

```text
proxmark
```

The Raspberry Pi should then be accessible on the local network using:

```text
proxmark.local
```

Hostname resolution depends on mDNS support on the network and client device.

### User Account

Create a user account and choose a secure password.

In this tutorial, we will use:

```text
piuser
```

You may choose a different username.

### Wi-Fi Configuration

Enter:

* Your Wi-Fi network name (SSID).
* Your Wi-Fi password.
* Wireless LAN country: France (FR).

The Raspberry Pi Zero 2 W supports 2.4 GHz Wi-Fi. Make sure your network is compatible.

### Enable SSH

In Raspberry Pi Imager settings:

1. Enable SSH.
2. Select password authentication for the initial setup.
3. Make sure the user account has a secure password.

SSH key authentication can be configured later for improved security.

---

## 5. Write the microSD Card

Once configuration is complete:

1. Start writing the image.
2. Wait for the process to finish.
3. Allow Raspberry Pi Imager to verify the installation.
4. Safely remove the microSD card.

---

## 6. First Boot

1. Insert the microSD card into the Raspberry Pi Zero 2 W.
2. Connect the power supply.
3. Wait a few minutes for the first boot to complete.

The Raspberry Pi should automatically connect to the configured Wi-Fi network.

---

## 7. First SSH Connection

From a computer connected to the same network, open a terminal.

On Windows, use PowerShell.

Run:

```bash
ssh piuser@proxmark.local
```

Replace `piuser` with the username selected during setup.

On the first connection, verify that you are connecting to your Raspberry Pi before accepting the SSH host key.

Enter your password.

A successful connection should display a prompt similar to:

```text
piuser@proxmark:~ $
```

The Raspberry Pi can now be managed remotely.

### Troubleshooting SSH

If `proxmark.local` does not resolve:

1. Verify that the Raspberry Pi is connected to Wi-Fi.
2. Find its IP address in your router's administration interface.
3. Connect directly using its IP address:

```bash
ssh piuser@192.168.1.100
```

Replace `192.168.1.100` with the actual IP address of your Raspberry Pi.

---

## 8. Update the System

Once connected via SSH, update the package lists:

```bash
sudo apt update
```

Then upgrade installed packages:

```bash
sudo apt full-upgrade -y
```

Reboot if necessary:

```bash
sudo reboot
```

Wait for the Raspberry Pi to restart before reconnecting.

---

## 9. Verify the Installation

### Operating System Version

```bash
cat /etc/os-release
```

The reference setup uses Debian 13 (Trixie).

### System Architecture

```bash
uname -m
```

Expected output:

```text
aarch64
```

This indicates a 64-bit ARM operating system.

### Network Status

```bash
nmcli device status
```

The `wlan0` interface should be connected.

### SSH Status

```bash
systemctl status ssh
```

The SSH service should be active (`active (running)`).

---

## 10. Add a Secondary Wi-Fi Network

Multiple Wi-Fi networks can be saved.

To connect to another network:

```bash
sudo nmcli device wifi connect "WIFI_NAME" --ask
```

NetworkManager will prompt for the password if required.

⚠️ Warning: this command may interrupt your current SSH session if the Raspberry Pi immediately switches networks.

To list saved connections:

```bash
nmcli connection show
```

The Raspberry Pi can reconnect to saved networks when they become available, depending on NetworkManager priorities and settings.

---

## ✅ Installation Complete

The Raspberry Pi Zero 2 W is now ready for the Proxmark3 Iceman client installation.

The next chapter will explain how to install the required dependencies, download the source code, and compile the software.
