# 📡 Proxmark3 Easy — Raspberry Pi Zero 2 W Wireless

🌐 **Language:** [🇫🇷 Français](README.fr.md) | 🇬🇧 English

---

## 📖 Overview

**Turn your Proxmark3 Easy into a portable wireless RFID tool using a Raspberry Pi Zero 2 W!**

This project provides a step-by-step guide to connecting a **Proxmark3 Easy** to a **Raspberry Pi Zero 2 W** and controlling it remotely from a smartphone or computer.

The goal is to create a compact, portable, and standalone RFID system that allows users to access Proxmark3 features without connecting the device directly to a computer.

The Raspberry Pi handles communication with the Proxmark3, while a smartphone or computer provides access to its command-line interface over Wi-Fi.

### ✨ Features

* 🔌 Connect the Proxmark3 Easy to the Raspberry Pi via USB.
* 💻 Install the Proxmark3 Iceman firmware and client.
* 📶 Establish wireless communication over Wi-Fi.
* 📱 Control the system from an iPhone using SSH.
* 🤖 Compatible with Android smartphones and computers.
* 🛠️ Execute Proxmark3 commands remotely.
* 🎒 Build a portable and standalone RFID system.

---

## 🔧 Required Hardware

| Hardware               | Description                                        |
| ---------------------- | -------------------------------------------------- |
| Raspberry Pi Zero 2 W  | Computer responsible for controlling the Proxmark3 |
| Proxmark3 Easy         | RFID/NFC reader and analysis tool                  |
| MicroSD card           | Storage for Raspberry Pi OS                        |
| USB OTG adapter        | Connects the Proxmark3 to the Raspberry Pi         |
| 5V power supply        | Powers the Raspberry Pi and Proxmark3              |
| Smartphone or computer | Remote control interface                           |

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

---

## 📚 Documentation

The tutorial is organized into several chapters:

1. 📖 [Required Hardware and Connections](docs/en/01-hardware.md)
2. 📖 [Raspberry Pi Installation](docs/en/02-raspberry-installation.md)
3. 📖 [Proxmark3 Iceman Installation](docs/en/03-proxmark3-installation.md)
4. ⏳ Wi-Fi Configuration
5. ⏳ Connecting from an iPhone
6. ⏳ Using the Proxmark3
7. ⏳ Troubleshooting

⏳ Additional chapters are currently being written.
---

## 🚧 Project Status

🟡 **Documentation is currently under development.**

This project is based on a personal setup using a Raspberry Pi Zero 2 W and a Proxmark3 Easy.

Installation and configuration procedures will be progressively documented and verified.

---

## 🤝 Contributing

Contributions are welcome!

You can help improve this project by:

* Reporting bugs or issues.
* Suggesting improvements.
* Sharing your experiences.
* Helping translate the documentation.

Feel free to open an *Issue* or submit a *Pull Request*.

---

## ⚠️ Disclaimer

This project is intended for educational purposes, RFID security research, and authorized testing on devices and systems you own or have explicit permission to test.

All activities must comply with applicable laws and regulations.

The authors are not responsible for any illegal or unauthorized use of the hardware or information provided.

---

## 📜 License

This project is released under the MIT License. See the [LICENSE](LICENSE) file for details.
