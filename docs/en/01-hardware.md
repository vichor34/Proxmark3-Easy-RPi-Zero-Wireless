# 🔧 Required Hardware and Connections

🌐 **Language:** [🇫🇷 Français](../fr/01-materiel.md) | 🇬🇧 English

[🏠 Back to Home](../../README.md)

---

## 1. Hardware Overview

This project uses a **Raspberry Pi Zero 2 W** to control a **Proxmark3 Easy** remotely over Wi-Fi.

The setup relies on a powered USB OTG adapter, allowing the Proxmark3 to communicate with the Raspberry Pi while providing external power.

### Components

| Component                | Quantity | Description                               |
| ------------------------ | -------- | ----------------------------------------- |
| Raspberry Pi Zero 2 W    | 1        | Small computer with built-in Wi-Fi        |
| Proxmark3 Easy           | 1        | RFID/NFC analysis tool                    |
| Powered USB OTG adapter  | 1        | OTG adapter with external power input     |
| USB-A to micro-USB cable | 1        | Connects the Proxmark3 to the OTG adapter |
| 5V USB power supply      | 1        | Powers the setup                          |
| MicroSD card             | 1        | 16 GB minimum recommended                 |
| Smartphone or computer   | 1        | Remote access via SSH                     |

---

## 2. Powered USB OTG Adapter

The adapter used in this project has three connectors:

* **Male micro-USB**: connects to the Raspberry Pi USB port.
* **Female USB-A**: connects to the Proxmark3 Easy.
* **Female micro-USB**: connects to the external 5V power supply.

This configuration eliminates the need for a conventional USB hub and keeps the setup compact.

**Important:** Not all powered OTG adapters work in the same way. Some models do not provide power to the Raspberry Pi or are incompatible with its USB OTG functionality.

Choosing a compatible adapter and verifying its operation is essential.

---

## 3. Wiring Diagram

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

### Connection Procedure

1. Insert the microSD card into the Raspberry Pi Zero 2 W.
2. Connect the male micro-USB connector of the OTG adapter to the Raspberry Pi **USB** port.
3. Connect the Proxmark3 USB cable to the adapter's female USB-A port.
4. Connect the other end of the cable to the Proxmark3 Easy.
5. Connect the 5V power supply to the adapter's female micro-USB port.
6. Verify that the Raspberry Pi boots and the Proxmark3 receives power.

---

## 4. Important Notes

### Raspberry Pi USB Port

The Raspberry Pi Zero 2 W has two micro-USB connectors:

* **USB**: USB OTG data port.
* **PWR IN**: dedicated power input.

The OTG adapter must be connected to the **USB** port to enable communication with the Proxmark3.

### Power Supply

Use a regulated 5V power supply capable of delivering sufficient current for both the Raspberry Pi and the Proxmark3.

A 5V / 2.5A power supply is a reasonable starting point, provided the OTG adapter supports this configuration.

Avoid low-quality power supplies, which may cause USB disconnections or unexpected restarts.

### Compatibility

This guide documents the hardware configuration used in this project.

Compatibility with other powered OTG adapters is not guaranteed.

---

## 5. Hardware Verification

Once Raspberry Pi OS has been installed and the system has booted, you can check whether the Proxmark3 is detected:

```bash
lsusb
```

To locate the associated serial device:

```bash
ls /dev/ttyACM*
```

Depending on the firmware and system configuration, the Proxmark3 may appear as:

```text
/dev/ttyACM0
```

These checks will be explained in more detail in the Proxmark3 installation chapter.

---

## ➡️ Next Step

**Installing and configuring Raspberry Pi OS**

We will prepare the microSD card, enable SSH, and configure Wi-Fi connectivity.
