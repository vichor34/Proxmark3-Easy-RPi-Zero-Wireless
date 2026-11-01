# 🔧 Step 1 — Required Hardware and Connections

🌐 **Language:** [🇫🇷 Français](../fr/01-materiel.md) | 🇬🇧 English

[🏠 Back to Table of Contents](../../README.md)

---

## 📖 Introduction

This first chapter presents the hardware required to turn a **Proxmark3 Easy** into a remotely accessible RFID/NFC tool using a **Raspberry Pi Zero 2 W**.

The Raspberry Pi communicates with the Proxmark3 through USB, while a smartphone or computer can control the system remotely using Wi-Fi and SSH.

The setup uses a **powered USB OTG adapter**, allowing the Proxmark3 to connect to the Raspberry Pi while providing power to the system.

### 🎯 Objectives

By the end of this chapter, you will be able to:

- Identify the components required for the project.
- Understand the purpose of the powered USB OTG adapter.
- Connect the hardware correctly.
- Identify the Raspberry Pi Zero 2 W USB ports.
- Check the power supply configuration.
- Prepare the hardware for Raspberry Pi OS installation.

---

## 1. Required Hardware

### 📦 Components List

| Component | Quantity | Description |
|---|:---:|---|
| Raspberry Pi Zero 2 W | 1 | Small computer with built-in Wi-Fi |
| Proxmark3 Easy | 1 | RFID/NFC analysis tool |
| Powered USB OTG adapter | 1 | USB adapter with external power input |
| USB-A to micro-USB cable | 1 | Connects the Proxmark3 to the OTG adapter |
| 5V USB power supply | 1 | Provides electrical power to the setup |
| MicroSD card | 1 | 16 GB minimum recommended |
| Smartphone or computer | 1 | Remote access via SSH |

### 💡 Why Use a Raspberry Pi Zero 2 W?

The Raspberry Pi Zero 2 W offers several advantages:

- Compact size and small footprint.
- Built-in 2.4 GHz Wi-Fi.
- Low power consumption.
- Compatibility with Raspberry Pi OS.
- Remote administration through SSH.
- Headless operation without a monitor, keyboard, or mouse.

These features make it a suitable solution for building a portable wireless RFID system.

---

## 2. Powered USB OTG Adapter

### 🔌 Overview

The Raspberry Pi Zero 2 W includes a USB port supporting OTG (*On-The-Go*) functionality.

The adapter used in this project has three connectors:

| Connector | Function |
|---|---|
| Male micro-USB | Connects to the Raspberry Pi USB port |
| Female USB-A | Connects to the Proxmark3 Easy |
| Female micro-USB | Connects to the external 5V power supply |

This adapter keeps the setup compact without requiring a conventional USB hub.

### ⚠️ Compatibility Warning

Not all powered USB OTG adapters work in the same way.

Some models:

- Do not transmit USB data correctly.
- Do not provide power to the Raspberry Pi through the USB port.
- Do not support simultaneous power delivery and USB data communication.
- May be incompatible with the Raspberry Pi USB OTG functionality.

It is therefore important to choose a compatible adapter and verify its operation.

> ⚠️ **Important**
>
> The presence of a power input on an OTG adapter does not guarantee that it will also power the Raspberry Pi.
>
> The wiring described here corresponds to the configuration used in this project. Compatibility with other adapters is not guaranteed.

---

## 3. Wiring Diagram

The setup uses a direct USB connection between the Raspberry Pi and the Proxmark3 Easy.

The 5V power supply connects to the powered USB OTG adapter.

### 🔗 Connection Diagram

```text
                   5V USB Power Supply
                            |
                            |
                     Female micro-USB
                            |
                 ┌────────────────────┐
                 │     Powered USB    │
                 │     OTG Adapter    │
                 └────────────────────┘
                      |           |
                      |           |
                Male micro-USB  Female USB-A
                      |           |
                      |           |
               ┌────────────┐     |
               │ Raspberry  │     |
               │ Pi Zero 2 W│     |
               │            │     |
               │ USB Port   │     |
               └────────────┘     |
                                  |
                           USB-A to
                           micro-USB cable
                                  |
                                  |
                           ┌────────────┐
                           │ Proxmark3  │
                           │    Easy    │
                           └────────────┘
```

### 🛠️ Connection Procedure

**Before starting, disconnect the power supply.**

1. Insert the microSD card into the Raspberry Pi Zero 2 W.
2. Identify the **USB** port on the Raspberry Pi.
3. Connect the male micro-USB connector of the OTG adapter to this port.
4. Connect the Proxmark3 USB cable to the female USB-A port of the adapter.
5. Connect the other end of the cable to the Proxmark3 Easy.
6. Connect the 5V power supply to the female micro-USB connector of the OTG adapter.
7. Verify that the Raspberry Pi boots and that the Proxmark3 receives power.

> 💡 **Tip**
>
> If Raspberry Pi OS has not yet been installed on the microSD card, the system will not boot normally.
>
> Operating system installation will be covered in Step 2.

---

## 4. Important Notes

### 🔌 4.1 — Raspberry Pi USB Ports

The Raspberry Pi Zero 2 W has two micro-USB connectors:

| Port | Function |
|---|---|
| USB | USB OTG data port |
| PWR IN | Dedicated power input |

The OTG adapter must be connected to the **USB** port to enable communication with the Proxmark3.

The **PWR IN** port is intended for power input and does not support USB data communication with the Proxmark3.

> ⚠️ **Warning**
>
> Do not confuse the **USB** and **PWR IN** ports.
>
> Connecting the OTG adapter to the PWR IN port will prevent USB communication between the Raspberry Pi and the Proxmark3.

### ⚡ 4.2 — Power Supply

Use a regulated 5V power supply capable of delivering sufficient current for both the Raspberry Pi and the Proxmark3.

A **5V / 2.5A** power supply is a reasonable starting point, provided the OTG adapter supports this configuration.

Avoid low-quality power supplies, which may cause:

- USB disconnections.
- Unexpected Raspberry Pi restarts.
- Communication problems with the Proxmark3.
- General system instability.

### 🔧 4.3 — Hardware Compatibility

This guide documents the hardware configuration used in this project.

Compatibility with other powered USB OTG adapters is not guaranteed.

Before continuing, verify that:

- The Raspberry Pi receives sufficient power.
- The Proxmark3 Easy is correctly connected.
- All USB connectors are properly inserted.
- The setup does not experience unexpected disconnections or restarts.

---

## 5. Hardware Verification

Once Raspberry Pi OS has been installed and the system has booted, you can verify whether the Proxmark3 is correctly detected.

These checks require terminal access to the Raspberry Pi.

System configuration and SSH access will be explained in Step 2.

### 🔍 5.1 — Check USB Devices

Run the following command:

```bash
lsusb
```

This command displays the USB devices detected by the Raspberry Pi.

The Proxmark3 should appear in the list of connected devices when it is correctly recognized.

### 🔍 5.2 — Check the Serial Port

To search for ACM-type USB serial devices, run:

```bash
ls /dev/ttyACM*
```

Depending on the firmware and system configuration, the Proxmark3 may appear as:

```text
/dev/ttyACM0
```

The port number may vary depending on the connected devices.

> 💡 **Note**
>
> If `/dev/ttyACM0` does not exist, this does not necessarily mean that the Proxmark3 is defective.
>
> Device detection depends on the installed firmware and the operating mode of the device.
>
> Additional checks will be covered during the Proxmark3 Iceman software installation.

---

## 6. Checklist Before Continuing

Before proceeding to the next step, make sure that:

- [ ] All required components are available.
- [ ] The microSD card is ready for use.
- [ ] The OTG adapter is connected to the Raspberry Pi USB port.
- [ ] The Proxmark3 Easy is correctly connected.
- [ ] The 5V power supply is suitable for the setup.
- [ ] All connections have been checked.

---

## ✅ Step 1 Complete — Hardware and Connections

The required hardware has now been identified, and the USB setup is ready.

We can now proceed with installing and configuring the Raspberry Pi operating system.

In the next step, we will:

- Install Raspberry Pi OS Lite (64-bit).
- Prepare and configure the microSD card.
- Enable SSH access.
- Configure Wi-Fi connectivity.
- Perform the first boot.
- Verify that the system is working correctly.

---

## 🧭 Navigation

| 🏠 Table of Contents | ➡️ Next Step |
|---|---|
| [Back to Home](../../README.md) | [Step 2 — Raspberry Pi Installation](02-raspberry-installation.md) |

---

📡 **Project: Proxmark3 Easy — Raspberry Pi Zero 2 W Wireless**
