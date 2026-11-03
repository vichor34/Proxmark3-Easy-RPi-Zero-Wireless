# 📡 Proxmark3 Easy — Raspberry Pi Zero 2 W Wireless

🌐 **Language:** [🇫🇷 Français](README.fr.md) | 🇬🇧 English

---

## 📖 Overview

**Turn your Proxmark3 Easy into a portable wireless RFID tool using a Raspberry Pi Zero 2 W!**

This project provides a step-by-step guide to connecting a **Proxmark3 Easy** to a **Raspberry Pi Zero 2 W** and controlling it remotely from a smartphone or computer.

The goal is to create a compact, portable, and standalone RFID system that allows users to access Proxmark3 features without connecting the device directly to a computer.

The Raspberry Pi handles communication with the Proxmark3, while a smartphone or computer provides access to its command-line interface over Wi-Fi.

### ✨ Features

- 🔌 Connect the Proxmark3 Easy to the Raspberry Pi via USB.
- 💻 Install the Proxmark3 Iceman firmware and client.
- 📶 Establish wireless communication over Wi-Fi.
- 📱 Control the system from an iPhone or iPad using SSH.
- 🤖 Support for Android smartphones.
- 🖥️ Remote access from Windows, macOS, and Linux.
- 🛠️ Execute Proxmark3 commands remotely.
- 🔄 Use persistent terminal sessions with tmux.
- 💾 Back up and maintain the installation.
- 🎒 Build a portable and standalone RFID system.
- 📶 **Automatic Wi-Fi hotspot**: connects to known Wi-Fi networks first and creates a standalone access point if none is available at startup.

---

## 🔧 Required Hardware

| Hardware | Description |
|---|---|
| Raspberry Pi Zero 2 W | Computer responsible for controlling the Proxmark3 |
| Proxmark3 Easy | RFID/NFC reader and analysis tool |
| MicroSD card | Storage for Raspberry Pi OS |
| USB OTG adapter | Connects the Proxmark3 to the Raspberry Pi |
| Suitable 5V power supply | Powers the Raspberry Pi and Proxmark3 |
| Smartphone or computer | Remote control interface |

> 💡 **Note**
>
> This tutorial is based on a **generic Proxmark3 Easy** and a **Raspberry Pi Zero 2 W**.
>
> Some Proxmark3 hardware variants may require a different configuration.
>
> Hardware specifications and power supply precautions are covered in the first chapter.

---

## 🔗 System Architecture

```text
       📱 iPhone / Android / PC
                  |
                  |
                Wi-Fi
                  |
                  |
         Raspberry Pi Zero 2 W
                  |
                  |
                 USB
                  |
                  |
            Proxmark3 Easy
                  |
                  |
               RFID / NFC
```

The Raspberry Pi Zero 2 W communicates directly with the Proxmark3 Easy via USB.

Users connect to the Raspberry Pi over Wi-Fi and establish an SSH session to access the Proxmark3 Iceman client.

Commands are executed on the Raspberry Pi and transmitted to the Proxmark3.

### 🖥️ Remote Access and Persistent Sessions

The installation mainly relies on three tools:

| Tool | Function |
|---|---|
| SSH | Access the Raspberry Pi remotely |
| tmux | Keep a terminal session running after an SSH disconnection |
| Proxmark3 Iceman | Communicate with the Proxmark3 Easy |

Two commands simplify daily operation:

| Command | Function |
|---|---|
| `pm` | Create or reconnect to a tmux session named `proxmark` |
| `pm3` | Launch the Proxmark3 Iceman client |

> 💡 **Important**
>
> If an existing tmux session already contains a running Iceman client, there is no need to launch another instance using `pm3`.
>
> tmux sessions survive SSH disconnections, but not Raspberry Pi reboots.

---

## 📚 Documentation

The tutorial is organized into **7 chapters**, available in both English and French.

Each chapter covers a specific step of the installation, configuration, or operation of the system.

| Step | Chapter | Description |
|---|---|---|
| 1 | [📖 Required Hardware and Connections](docs/en/01-hardware.md) | Hardware overview, USB connections, and power supply |
| 2 | [💻 Raspberry Pi Installation](docs/en/02-raspberry-installation.md) | Install Raspberry Pi OS and configure Wi-Fi, SSH, and tmux |
| 3 | [📡 Proxmark3 Iceman Installation](docs/en/03-proxmark3-installation.md) | Install, compile, and configure the Iceman client and firmware |
| 4 | [🖥️ Daily Usage](docs/en/04-daily-usage.md) | SSH access, tmux sessions, Iceman commands, and diagnostics |
| 5 | [📱 Mobile Connection](docs/en/05-mobile-connection.md) | Connect from an iPhone, iPad, or Android smartphone |
| 6 | [🔧 Maintenance, Backups and Updates](docs/en/06-maintenance.md) | Backups, updates, recovery, and troubleshooting |
| 7 | [📶 Automatic Wi-Fi Hotspot](docs/en/07-wifi-hotspot.md) | Connect to known Wi-Fi networks or create a standalone hotspot |

### 🌐 French Documentation

The complete French version is also available:

| Étape | Documentation française |
|---|---|
| 1 | [Matériel nécessaire et branchements](docs/fr/01-materiel.md) |
| 2 | [Installation du Raspberry Pi](docs/fr/02-installation-raspberry.md) |
| 3 | [Installation du Proxmark3 Iceman](docs/fr/03-installation-proxmark3.md) |
| 4 | [Utilisation quotidienne](docs/fr/04-utilisation-quotidienne.md) |
| 5 | [Connexion mobile](docs/fr/05-connexion-mobile.md) |
| 6 | [Maintenance, sauvegardes et mises à jour](docs/fr/06-maintenance.md) |

---

## 🚀 Quick Start

To complete the installation, follow the chapters in order.

### 1. Prepare the Hardware

Connect the Proxmark3 Easy to the Raspberry Pi Zero 2 W using a USB OTG adapter.

Make sure the power supply is suitable for the system.

See [Chapter 1 — Required Hardware and Connections](docs/en/01-hardware.md).

### 2. Install Raspberry Pi OS

Install Raspberry Pi OS Lite 64-bit on the microSD card.

Configure the following:

- Raspberry Pi hostname.
- User account.
- Wi-Fi network.
- SSH access.
- tmux environment.

See [Chapter 2 — Raspberry Pi Installation](docs/en/02-raspberry-installation.md).

### 3. Install Proxmark3 Iceman

Install and compile the Iceman project using the configuration appropriate for a generic Proxmark3 Easy.

The reference configuration uses:

```text
PLATFORM=PM3GENERIC
LED_ORDER=PM3EASY
```

> ⚠️ **Warning**
>
> Firmware flashing must only be performed with files compatible with your hardware.
>
> This tutorial prioritizes preserving the existing bootloader.
>
> Do not flash the Proxmark3 without reading the precautions in Chapter 3.

See [Chapter 3 — Proxmark3 Iceman Installation](docs/en/03-proxmark3-installation.md).

### 4. Connect Remotely

Once the system is installed, connect to the Raspberry Pi from a terminal:

```bash
ssh piuser@proxmark.local
```

> 💡 **Note**
>
> `piuser` and `proxmark` are example values used throughout this tutorial.
>
> Adjust them to match your own configuration.
>
> If `proxmark.local` cannot be resolved, use the Raspberry Pi's IP address instead.

Open the tmux session:

```bash
pm
```

If Iceman is not already running inside the session, start the client:

```bash
pm3
```

You can now use Iceman commands.

See [Chapter 4 — Daily Usage](docs/en/04-daily-usage.md).

### 5. Connect from a Smartphone

The system can also be controlled from:

- An iPhone.
- An iPad.
- An Android smartphone.

The connection uses SSH over a Wi-Fi network accessible to both devices.

A smartphone hotspot can also be used, provided that the network allows communication between the connected devices.

See [Chapter 5 — Mobile Connection](docs/en/05-mobile-connection.md).

---

## 🛠️ Maintenance and Backups

Once the installation is working, keeping a backup of your configuration is recommended.

The tutorial explains how to:

- Check the Raspberry Pi's status.
- Identify installed software versions.
- Back up network settings.
- Back up the Iceman client configuration.
- Create a complete microSD card image.
- Update Raspberry Pi OS.
- Prepare an Iceman client update.
- Restore a previous installation.
- Troubleshoot common problems.

> ⚠️ **Important**
>
> Updating Raspberry Pi OS, compiling the Iceman client, and flashing the Proxmark3 firmware are separate operations.
>
> If your installation works correctly, there is no need to flash the Proxmark3 regularly.
>
> Backups containing network credentials must be stored in a private, protected location.

See [Chapter 6 — Maintenance, Backups and Updates](docs/en/06-maintenance.md).

---

## 🚧 Project Status

🟢 **All seven tutorial chapters are available in English and French.**

This project is based on a personal setup using a Raspberry Pi Zero 2 W and a generic Proxmark3 Easy.

The documentation covers the reference configuration, installation procedures, remote operation, and maintenance tasks.

Feedback, corrections, and improvements are welcome.

> 💡 **Note**
>
> Commands and procedures are documented for the hardware configuration presented in this tutorial.
>
> Their behavior may vary depending on hardware revisions, operating system versions, or firmware versions.

---

## 🤝 Contributing

Contributions are welcome!

You can help improve this project by:

- Reporting bugs or issues.
- Suggesting improvements.
- Sharing your experiences.
- Helping translate the documentation.
- Proposing technical corrections.

Feel free to open an *Issue* or submit a *Pull Request*.

---

## ⚠️ Disclaimer

This project is intended for educational purposes, RFID security research, and authorized testing on devices and systems you own or have explicit permission to test.

All activities must comply with applicable laws and regulations.

The authors are not responsible for any illegal or unauthorized use of the hardware or information provided.

---

## 📜 License and Usage

© 2026 Victor HANSEMANN.

The original documentation of this project is licensed under **Creative Commons BY-NC-SA 4.0** (Attribution-NonCommercial-ShareAlike).

This license allows users to read, share, and adapt the documentation, subject to its terms.

An **additional royalty-free permission** also allows organizations, businesses, and educational institutions to use and adapt the documentation for their internal purposes.

Adaptations distributed under the Creative Commons license must comply with its ShareAlike requirements.

Commercial exploitation of protected materials beyond the granted permissions requires a separate agreement.

The documentation license does not, by itself, restrict the manufacture or sale of independent hardware using the technical knowledge described.

### 📄 Licensing Documents

- [CC BY-NC-SA 4.0 License](LICENSE)
- [Additional Permissions](PERMISSIONS.md)
- [Commercial Licensing](COMMERCIAL.md)
- [Contributing Guidelines](CONTRIBUTING.md)

---

📡 **Project: Proxmark3 Easy — Raspberry Pi Zero 2 W Wireless**

🌐 [🇫🇷 Lire cette documentation en français](README.fr.md)
