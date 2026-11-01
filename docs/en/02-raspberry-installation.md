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
| Persistent Sessions | tmux                 |

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

## 11. Keep Terminal Sessions Running with tmux

When accessing the Raspberry Pi remotely, SSH connections may be interrupted if:

* The computer enters sleep mode.
* The Wi-Fi connection drops.
* The terminal is accidentally closed.
* The user switches networks or devices.

Without additional protection, programs running in the terminal may be interrupted.

**tmux** allows terminal sessions to remain active on the Raspberry Pi independently of the SSH connection.

This is especially useful when compiling Proxmark3 Iceman firmware or running long operations.

### 11.1 Install tmux

From your SSH terminal:

```bash
sudo apt update
sudo apt install tmux -y
```

Verify the installation:

```bash
tmux -V
```

The installed version should be displayed.

### 11.2 Create a Session

Run:

```bash
tmux new -s proxmark
```

A new session named `proxmark` is created.

You can now execute commands inside this session, for example:

```bash
pm3
```

The `pm3` command becomes available after installing the Iceman client, as described in Chapter 3.

### 11.3 Detach from a Session

To leave the session without stopping its programs:

1. Press `Ctrl + B`.
2. Release both keys.
3. Press `D`.

You will return to the regular SSH terminal.

Programs running inside `tmux` continue running on the Raspberry Pi.

### 11.4 Reattach to a Session

After reconnecting through SSH:

```bash
tmux attach -t proxmark
```

This command restores the previous session.

To list existing sessions:

```bash
tmux ls
```

### 11.5 Simplify Access with the `pm` Command

Instead of typing a long command every time, we can create a Bash alias.

Run:

```bash
echo "alias pm='tmux new-session -A -s proxmark'" >> ~/.bashrc
```

Reload the configuration:

```bash
source ~/.bashrc
```

From now on, you only need:

```bash
pm
```

The behavior is automatic:

* If the `proxmark` session already exists, it is restored.
* If no session exists, a new one is created.

The alias will be available in future SSH sessions using Bash.

### 11.6 Daily Usage

The workflow becomes very simple.

**1. Connect to the Raspberry Pi:**

```bash
ssh piuser@proxmark.local
```

**2. Open or restore your session:**

```bash
pm
```

**3. Launch Proxmark3:**

```bash
pm3
```

**4. To detach without stopping programs:**

Press `Ctrl + B`, then `D`.

The next time you connect, simply type `pm` to restore your session.

### 11.7 Close a Session Permanently

To close a session, first exit any running applications, then execute:

```bash
exit
```

The `tmux` session ends when no windows or programs remain active.

### 11.8 Important Limitations

**tmux does not keep the SSH connection itself alive.**

It keeps programs running on the Raspberry Pi when the SSH connection is interrupted.

The Raspberry Pi must remain powered on, and any required USB devices must remain connected.

Power loss or a Raspberry Pi reboot will terminate existing `tmux` sessions.

---

## ✅ Installation Complete

The Raspberry Pi Zero 2 W is now ready for the Proxmark3 Iceman client installation.

We now have:

* A lightweight, updated Linux system.
* Remote access through SSH.
* Configurable Wi-Fi connectivity.
* Persistent terminal sessions using `tmux`.
* A simplified `pm` command to resume work after disconnection.

➡️ [Chapter 3 — Proxmark3 Installation and Firmware Update](03-proxmark3-installation.md)
