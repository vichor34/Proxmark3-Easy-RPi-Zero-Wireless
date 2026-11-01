# 💻 Step 2 — Raspberry Pi Installation and Configuration

🌐 **Language:** [🇫🇷 Français](../fr/02-installation-raspberry.md) | 🇬🇧 English

[🏠 Back to Table of Contents](../../README.md)

---

## 📖 Introduction

This chapter explains how to install and configure **Raspberry Pi OS** on a **Raspberry Pi Zero 2 W** for use with the **Proxmark3 Easy**. The installation is performed *headlessly*, without a monitor, keyboard, or mouse. You will manage the Raspberry Pi remotely over **Wi-Fi and SSH**.

We will also install **tmux**, which keeps terminal sessions running when an SSH connection drops.

### 🎯 Objectives

By the end of this chapter, you will be able to:

- Install Raspberry Pi OS Lite (64-bit) using Raspberry Pi Imager.
- Set the hostname, user account, Wi-Fi, and SSH access.
- Update the operating system and check network services.
- Save additional Wi-Fi networks using NetworkManager.
- Install `tmux` and create the `pm` shortcut for persistent sessions.

---

## 1. Reference Configuration

| Item | Configuration |
|---|---|
| Raspberry Pi | Zero 2 W |
| Operating system | Raspberry Pi OS Lite |
| Architecture | 64-bit |
| Debian base | Debian 13 (Trixie) |
| Desktop environment | None |
| Network manager | NetworkManager |
| Network | 2.4 GHz Wi-Fi |
| Remote access | SSH |
| Persistent terminal sessions | tmux |
| Example hostname | `proxmark` |
| Example username | `piuser` |

> 💡 **Note**
>
> This is the reference configuration used in this project. If you choose a different hostname or username, adapt the commands accordingly. Raspberry Pi OS releases may change over time; check the version installed on your device.

---

## 2. Download Raspberry Pi Imager

**Raspberry Pi Imager** is the official tool for writing Raspberry Pi operating systems to microSD cards. It also allows you to preconfigure Wi-Fi, a user account, and SSH before the first boot.

🔗 [Download Raspberry Pi Imager](https://www.raspberrypi.com/software/)

Install the application on your computer, then insert a microSD card. **16 GB or more** is recommended.

---

## 3. Prepare the microSD Card

Open **Raspberry Pi Imager**.

### 🔧 3.1 — Select the Device

Choose **Raspberry Pi Zero 2 W**.

### 💻 3.2 — Select the Operating System

Choose **Raspberry Pi OS Lite (64-bit)**. This edition has no graphical desktop and is suitable for remote administration, with lower resource requirements than a desktop edition. The Proxmark3 Iceman client runs in a Linux terminal.

### 💾 3.3 — Select Storage

Select the intended microSD card.

> ⚠️ **Warning**
>
> Writing the image **erases all data** on the selected card. Double-check the target device before proceeding.

---

## 4. Initial Configuration

Before writing the image, use Raspberry Pi Imager's OS customization settings to prepare a headless installation.

### 🖥️ 4.1 — Hostname

Set the hostname to:

```text
proxmark
```

On networks that support mDNS, the Raspberry Pi may then be reachable as:

```text
proxmark.local
```

> 💡 **Note**
>
> `.local` resolution depends on mDNS support on both the network and your client device. If it fails, use the Raspberry Pi's IP address instead.

### 👤 4.2 — User Account

Create a user and set a **strong password**. The example username used throughout this tutorial is:

```text
piuser
```

You may use another username, but remember to adjust all SSH commands.

> ⚠️ **Security**
>
> Do not leave the account without a password or reuse a weak password.

### 📶 4.3 — Wi-Fi Configuration

Enter the following settings:

| Setting | Value |
|---|---|
| SSID | Your Wi-Fi network name |
| Password | Your Wi-Fi password |
| Wireless LAN country | France (FR), or your actual country |

The Raspberry Pi Zero 2 W supports **2.4 GHz Wi-Fi**. Make sure that band is enabled on your router. If your router combines 2.4 GHz and 5 GHz under one SSID, the Raspberry Pi will use its supported band.

### 🔐 4.4 — Enable SSH

In Raspberry Pi Imager:

1. Enable **SSH**.
2. Choose **password authentication** for the initial setup.
3. Confirm that the user account has a strong password.
4. Save the customization settings.

SSH public-key authentication can be configured later for stronger security.

> 💡 **Why SSH?**
>
> SSH lets you install software, change settings, and operate the Proxmark3 from a computer or smartphone without attaching a display to the Raspberry Pi.

---

## 5. Write the microSD Card

1. Start writing the image.
2. Confirm that the selected card may be erased.
3. Wait until writing finishes.
4. Allow Raspberry Pi Imager to verify the written data.
5. Safely eject the microSD card.

---

## 6. First Boot

### 🔌 6.1 — Power On

1. Insert the microSD card into the Raspberry Pi Zero 2 W.
2. Check the wiring described in Step 1.
3. Connect the power supply.
4. Wait a few minutes for the initial boot.

The Raspberry Pi should connect automatically to the configured Wi-Fi network. The first boot may take longer than subsequent boots.

### 📶 6.2 — Check Network Connectivity

Look for a device named `proxmark` in your router's administration interface. Note its local IP address in case `proxmark.local` cannot be resolved.

---

## 7. First SSH Connection

### 💻 7.1 — Open a Terminal

On Windows, open **PowerShell**. On Linux or macOS, open a terminal. Your computer must be able to reach the Raspberry Pi over the network.

### 🔐 7.2 — Connect

```bash
ssh piuser@proxmark.local
```

Replace `piuser` if you selected a different username. On first connection, verify that the SSH host key belongs to your Raspberry Pi before accepting it. Enter the account password when prompted.

A successful login displays a prompt similar to:

```text
piuser@proxmark:~ $
```

### 🔍 7.3 — Troubleshooting

If `proxmark.local` does not resolve, find the Pi's IP address in your router and connect directly:

```bash
ssh piuser@192.168.1.100
```

`192.168.1.100` is **only an example**; replace it with the real address.

| Error | Possible cause | Check |
|---|---|---|
| `Could not resolve hostname` | mDNS name unavailable | Use the IP address |
| `Connection refused` | SSH not listening | Check that SSH was enabled |
| `Connection timed out` | Network or address problem | Check Wi-Fi and the IP address |
| `Permission denied` | Incorrect credentials | Check username and password |

---

## 8. Update the System

### 🔄 8.1 — Refresh Package Lists

```bash
sudo apt update
```

### 📦 8.2 — Install Available Updates

```bash
sudo apt full-upgrade -y
```

The update may take several minutes.

### 🔁 8.3 — Reboot if Necessary

```bash
sudo reboot
```

SSH will disconnect while the Pi reboots. Wait for it to come back, then reconnect:

```bash
ssh piuser@proxmark.local
```

---

## 9. Verify the Installation

### 🔍 9.1 — Operating System Version

```bash
cat /etc/os-release
```

The reference installation uses **Debian GNU/Linux 13 (Trixie)**.

### 🔍 9.2 — System Architecture

```bash
uname -m
```

Expected output for the 64-bit system:

```text
aarch64
```

### 📶 9.3 — Network Status

```bash
nmcli device status
```

The Wi-Fi interface `wlan0` should normally show as connected.

### 🔐 9.4 — SSH Service

```bash
systemctl status ssh
```

The service should show `active (running)`. Press `Q` to exit the status display.

---

## 10. Add a Secondary Wi-Fi Network

The Raspberry Pi can save multiple Wi-Fi connections—for example, a home network, an office network, or a smartphone hotspot.

### 📶 10.1 — Add a Network

```bash
sudo nmcli device wifi connect "WIFI_NAME" --ask
```

Replace `WIFI_NAME` with the SSID. NetworkManager will request the password if needed.

> ⚠️ **Warning**
>
> Connecting to another network can immediately interrupt your current SSH session. Make sure you can discover the Raspberry Pi's new IP address before switching.

### 🔍 10.2 — List Saved Connections

```bash
nmcli connection show
```

NetworkManager can reconnect to saved networks when they become available. Selection depends on network availability, connection settings, and priorities. Your SSH device must be able to reach the Pi on the chosen network.

---

## 11. Keep a Terminal Session Running with tmux

### 📖 Introduction

An SSH connection may drop when a laptop sleeps, Wi-Fi disconnects, a terminal closes, or you switch devices. Programs running directly in that SSH terminal may then stop.

**tmux** keeps a terminal session running on the Raspberry Pi independently of your SSH connection. It is useful for long builds, Proxmark3 Iceman operations, and switching between a computer and a phone.

> 💡 **Important**
>
> `tmux` preserves the programs running inside its session; it **does not** keep the SSH connection itself alive.

### 🛠️ 11.1 — Install tmux

```bash
sudo apt update
sudo apt install tmux -y
```

Verify the installed version:

```bash
tmux -V
```

### 💻 11.2 — Create a Session

```bash
tmux new -s proxmark
```

This creates a session named `proxmark`. You can run terminal commands inside it, including the Proxmark3 client after Step 3:

```bash
pm3
```

> ⚠️ **Note**
>
> The `pm3` command is not available until the Proxmark3 Iceman client has been installed in Step 3. It is normal for this command to fail at this point.

### 🔌 11.3 — Detach Without Stopping Programs

1. Press `Ctrl + B`.
2. Release the keys.
3. Press `D`.

You return to the regular SSH shell while programs inside the tmux session continue running.

### 🔄 11.4 — Reattach to a Session

After reconnecting over SSH:

```bash
tmux attach -t proxmark
```

To list existing sessions:

```bash
tmux ls
```

### ⚡ 11.5 — Create the `pm` Shortcut

To create or reattach to the `proxmark` session with one command, add a Bash alias:

```bash
echo "alias pm='tmux new-session -A -s proxmark'" >> ~/.bashrc
```

Reload the Bash configuration:

```bash
source ~/.bashrc
```

Then run:

```bash
pm
```

| Situation | Result |
|---|---|
| Session `proxmark` already exists | Reattach to it |
| No `proxmark` session exists | Create a new one |

The alias is available in future SSH logins that use Bash. The `-A` option enables the create-or-attach behavior.

> 💡 **Tip**
>
> Add the alias only once. Repeating the `echo` command appends duplicate definitions to `~/.bashrc`.

### 📱 11.6 — Everyday Workflow

**1. Connect over SSH:**

```bash
ssh piuser@proxmark.local
```

**2. Create or recover the session:**

```bash
pm
```

**3. Start Proxmark3 after installing Iceman in Step 3:**

```bash
pm3
```

**4. Temporarily leave the session:** press `Ctrl + B`, release, then press `D`.

The next time you connect, use `pm` to return to the existing session.

### 🛑 11.7 — Close a Session Permanently

Exit any running application, then run:

```bash
exit
```

The tmux session ends when its last window and program have closed. **Closing** a session is different from **detaching**: a closed session cannot be recovered.

### ⚠️ 11.8 — Important Limitations

- The Raspberry Pi must remain powered on.
- Required USB peripherals must remain connected.
- The program must have been started **inside** tmux.
- The tmux session must not be closed.
- A reboot or power failure terminates tmux sessions.

`tmux` is especially recommended for long Proxmark3 operations and when Wi-Fi connectivity may be interrupted.

---

## 12. Checklist Before Continuing

- [ ] Raspberry Pi OS Lite (64-bit) is installed.
- [ ] The Raspberry Pi boots successfully.
- [ ] Wi-Fi is connected.
- [ ] `proxmark.local` or the Pi's IP address is reachable.
- [ ] SSH login works.
- [ ] System updates are installed.
- [ ] `uname -m` reports `aarch64`.
- [ ] NetworkManager and SSH are working.
- [ ] `tmux` is installed.
- [ ] The `pm` command creates or reattaches to a session.

---

## ✅ Step 2 Complete — Raspberry Pi Configured

The Raspberry Pi Zero 2 W is now ready for the **Proxmark3 Iceman** client installation. You have a lightweight Linux system, SSH remote access, configurable Wi-Fi, and persistent terminal sessions through `tmux` and the `pm` alias.

### 🚀 In the Next Step

We will install the required dependencies, download the Iceman source code, build and install the client, check communication with the Proxmark3 Easy, and cover firmware updates.

---

## 🧭 Navigation

| ⬅️ Previous Step | 🏠 Table of Contents | ➡️ Next Step |
|---|---|---|
| [Step 1 — Hardware and Connections](01-hardware.md) | [Back to Home](../../README.md) | [Step 3 — Proxmark3 Iceman Installation](03-proxmark3-installation.md) |

---

📡 **Project: Proxmark3 Easy — Raspberry Pi Zero 2 W Wireless**
