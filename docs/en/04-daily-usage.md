# 🖥️ Step 4 — Daily Usage and Remote Access to Proxmark3

🌐 **Language:** [🇫🇷 Français](../fr/04-utilisation-quotidienne.md) | 🇬🇧 English

[🏠 Back to Table of Contents](../../README.md)

---

## 📖 Introduction

The first three chapters covered hardware preparation, Raspberry Pi OS installation, and **Proxmark3 Iceman** configuration.

Our installation is now operational.

In this chapter, we will learn how to use the **Proxmark3 Easy remotely** from a computer connected to the same network as the Raspberry Pi Zero 2 W.

The goal is to control the Proxmark3 without connecting a monitor, keyboard, or mouse directly to the Raspberry Pi.

We will mainly use three tools:

- **SSH** to access the Raspberry Pi remotely.
- **tmux** to keep a terminal session running.
- **Iceman** to communicate with the Proxmark3 Easy.

We will also cover initial diagnostics, common troubleshooting procedures, and proper system shutdown.

> 💡 **Note**
>
> This chapter covers usage from Windows, macOS, and Linux.
>
> Connections from an iPhone, iPad, or Android device will be explained in detail in Step 5.

### 🎯 Objectives

By the end of this chapter, you will be able to:

- Start the Raspberry Pi and Proxmark3 correctly.
- Verify their USB connection.
- Connect over SSH from a computer.
- Use the `pm` and `pm3` commands.
- Recover a persistent session after disconnecting.
- Use the main Iceman diagnostic commands.
- Perform an initial RFID/NFC reading test.
- Troubleshoot common connection problems.
- Shut down the system safely to avoid corrupting the microSD card.

---

## 1. Reference Configuration

Our installation uses the following components:

| Component | Configuration |
|---|---|
| Raspberry Pi | Zero 2 W |
| Operating system | Raspberry Pi OS Lite 64-bit |
| Proxmark3 | Generic Easy |
| Firmware | Iceman — RfidResearchGroup |
| Proxmark3 connection | USB |
| Network connection | 2.4 GHz Wi-Fi |
| Remote access | SSH |
| Persistent sessions | tmux |
| Raspberry Pi hostname | `proxmark` |
| Example username | `piuser` |
| Session command | `pm` |
| Client command | `pm3` |

### 📚 Prerequisites

Before continuing, complete:

- [Step 1 — Hardware and Connections](01-hardware.md)
- [Step 2 — Raspberry Pi Installation](02-raspberry-installation.md)
- [Step 3 — Proxmark3 Iceman Installation](03-proxmark3-installation.md)

The `pm` and `pm3` commands must already be configured as described in the previous chapters.

---

## 2. Start the System

### 🔌 2.1 — Check the Connections

Before powering on the Raspberry Pi, verify:

1. The microSD card is correctly inserted.
2. The Proxmark3 Easy is connected to the USB data port.
3. The OTG adapter is properly connected.
4. The power supply is suitable for the Raspberry Pi and its peripherals.
5. The configured Wi-Fi network is available.

> ⚠️ **Warning**
>
> An insufficient power supply can cause USB disconnections, unexpected reboots, or communication errors.
>
> If the Proxmark3 is not detected, check the power supply before changing the software configuration.

### ⚡ 2.2 — Power On the Raspberry Pi

Connect the Raspberry Pi power supply.

The system starts automatically.

Wait approximately one to two minutes before attempting an SSH connection.

The required time may vary depending on the microSD card and installed services.

### 📶 2.3 — Check Network Connectivity

The Raspberry Pi should automatically connect to a saved Wi-Fi network.

In our configuration, its hostname is:

```text
proxmark
```

It may be reachable at:

```text
proxmark.local
```

> 💡 **Note**
>
> The `.local` hostname generally relies on the mDNS protocol.
>
> If your computer or network cannot resolve it, use the Raspberry Pi's IP address directly.

---

## 3. Connect to the Raspberry Pi from a Computer

The connection uses the SSH protocol.

SSH provides secure remote terminal access to the Raspberry Pi over the network.

### 💻 3.1 — From Windows

Windows 10 and Windows 11 generally include a built-in SSH client.

Open:

**PowerShell** or **Windows Terminal**.

Run:

```bash
ssh piuser@proxmark.local
```

Enter the password configured during Raspberry Pi installation.

On the first connection, SSH may ask you to confirm the server's host key fingerprint.

Verify that it belongs to your Raspberry Pi before accepting it.

### 🍎 3.2 — From macOS

Open the **Terminal** application.

Run:

```bash
ssh piuser@proxmark.local
```

Enter your password.

The procedure is the same as on Windows.

### 🐧 3.3 — From Linux

Open a terminal.

Run:

```bash
ssh piuser@proxmark.local
```

The OpenSSH client is installed by default on many Linux distributions.

If it is missing, install it using your distribution's package manager.

### 🌐 3.4 — Connect Using the IP Address

If `proxmark.local` does not work, find the Raspberry Pi's IP address in your router's administration interface.

For example:

```bash
ssh piuser@192.168.1.100
```

> ⚠️ **Important**
>
> `192.168.1.100` is only an example.
>
> Replace it with your Raspberry Pi's actual IP address.

### ✅ 3.5 — Verify the Connection

Once connected, you should see a command prompt similar to:

```text
piuser@proxmark:~ $
```

You are now controlling the Raspberry Pi remotely.

---

## 4. Verify the Proxmark3 Easy Connection

Before launching Iceman, we will verify that the USB device is detected.

### 🔍 4.1 — List USB Devices

In the SSH terminal:

```bash
lsusb
```

This command lists connected USB devices.

The Proxmark3 should normally appear in this list when operating in its usual mode.

### 🔍 4.2 — Check the Serial Port

Run:

```bash
ls -l /dev/ttyACM*
```

In our installation, the Proxmark3 appears as:

```text
/dev/ttyACM0
```

The device number may vary.

If several serial devices are connected, the Proxmark3 may use a different number.

### 🔍 4.3 — Examine Recent System Messages

If the device does not appear, run:

```bash
sudo dmesg | tail -n 30
```

This command displays recent Linux kernel messages.

They may help identify:

- A USB connection.
- An unexpected disconnection.
- A communication error.
- A power supply problem.
- A newly detected serial device.

> 💡 **Tip**
>
> If the Proxmark3 worked previously, check the cable, OTG adapter, and power supply before reinstalling the firmware.

---

## 5. Understand the `pm` and `pm3` Commands

We configured two commands in the previous chapters.

They serve different purposes.

| Command | Function |
|---|---|
| `pm` | Create or recover a tmux session |
| `pm3` | Launch the Proxmark3 Iceman client |

### 🖥️ 5.1 — The `pm` Command

Run:

```bash
pm
```

This command uses the alias created in Step 2:

```bash
alias pm='tmux new-session -A -s proxmark'
```

It allows you to:

- Create a session named `proxmark`.
- Reconnect to that session if it already exists.
- Keep programs running inside the session after an SSH disconnection.

### 📡 5.2 — The `pm3` Command

Once inside the tmux session:

```bash
pm3
```

This command launches the Iceman client installed in Step 3.

If the Proxmark3 is connected and recognized, the client should establish communication with the device.

### 🔄 5.3 — Complete Procedure

From your computer:

**1. Connect over SSH:**

```bash
ssh piuser@proxmark.local
```

**2. Open the persistent session:**

```bash
pm
```

**3. Launch the Iceman client:**

```bash
pm3
```

You can now use Proxmark3 commands.

### 💡 Why Two Commands?

`pm` does not directly launch the Proxmark3.

It opens a persistent working environment.

`pm3` then launches the software used to communicate with the Proxmark3.

This separation also allows you to use the tmux session for other operations, such as maintenance or compilation.

---

## 6. Use a Persistent Session with tmux

One of the advantages of our installation is the ability to keep a session running after an SSH disconnection.

### 🔌 6.1 — Detach a Session

While inside tmux, press:

```text
Ctrl + B
```

Release the keys, then press:

```text
D
```

You return to the regular SSH terminal.

Programs running inside the tmux session continue running.

### 🔄 6.2 — Recover a Session

After reconnecting over SSH:

```bash
pm
```

You return to the previous session.

If the Iceman client is still running, its terminal reappears.

### 🔍 6.3 — List Existing Sessions

From a terminal outside tmux:

```bash
tmux ls
```

This command lists available sessions.

### 🛑 6.4 — Close a Session Permanently

To close the session properly:

1. Exit the Iceman client.
2. Return to the Linux command prompt.
3. Run:

```bash
exit
```

If no other programs or windows remain active inside the tmux session, it ends.

> ⚠️ **Important**
>
> Detaching a session is not the same as closing it.
>
> A detached session can be recovered.
>
> A terminated session cannot be restored.

### ⚠️ 6.5 — tmux Limitations

tmux does not protect against:

- A power failure.
- A Raspberry Pi reboot.
- A physical Proxmark3 disconnection.
- The Iceman application stopping.
- A hardware failure.

The Raspberry Pi must remain powered on for the session to continue running.

---

## 7. Discover Iceman Commands

The Iceman client provides many commands.

We will start with help and diagnostic commands.

> ⚠️ **Responsible Use**
>
> Only use the Proxmark3 on cards, tags, and equipment that you own or are explicitly authorized to examine.

### 📖 7.1 — Display General Help

Inside the Iceman client:

```text
help
```

This command displays the main available command categories.

### 🔍 7.2 — Display Hardware Information

```text
hw version
```

This command provides information about the client, firmware, and bootloader versions.

### 🩺 7.3 — Check Hardware Status

```text
hw status
```

This command provides information about the Proxmark3's status.

It can help identify certain communication or configuration problems.

### 🧪 7.4 — Perform a Hardware Diagnostic

```text
hw tune
```

This command performs measurements related to the Proxmark3 antennas.

It can help verify their behavior.

> 💡 **Note**
>
> Results depend on the Proxmark3 model, antennas, and surrounding environment.
>
> A single measurement is not necessarily enough to diagnose a hardware failure.

### 📡 7.5 — Test NFC Reading

For an initial test, use an ISO 14443-A NFC tag that you own.

Run:

```text
hf 14a reader
```

Place the tag near the Proxmark3 high-frequency antenna.

The software should attempt to detect the tag and display the available information.

This command is suitable for an initial non-destructive test.

### 💡 7.6 — Difference Between HF and LF

The Proxmark3 supports different RFID technology families.

| Category | Common Frequency | Examples |
|---|---|---|
| LF — Low Frequency | 125 kHz / 134.2 kHz | Certain identification tags |
| HF — High Frequency | 13.56 MHz | NFC, ISO 14443, ISO 15693 |

The commands and antennas used depend on the tag technology.

For your first test, use a tag whose type you already know.

---

## 8. Use the Proxmark3 from Multiple Computers

The Raspberry Pi can be accessed from different computers connected to its network.

For example:

- A Windows computer.
- A Mac.
- A Linux computer.

Each device uses an SSH connection.

### 🔄 8.1 — Switch from One Computer to Another

From the first computer:

1. Open the SSH session.
2. Run `pm`.
3. Run `pm3`.
4. Detach the session with `Ctrl + B`, then `D`.
5. Close the SSH connection.

From the second computer:

```bash
ssh piuser@proxmark.local
```

Then:

```bash
pm
```

You return to the existing tmux session.

### ⚠️ 8.2 — Avoid Simultaneous Access

It is preferable for only one person to control the Proxmark3 at a time.

Multiple SSH connections can coexist, but this does not mean multiple Iceman clients should access the same USB device simultaneously.

Also avoid sending competing commands from multiple terminals attached to the same tmux session.

> 💡 **Tip**
>
> To switch devices, detach the session on the first device before reconnecting from the second.

---

## 9. Troubleshoot Common Problems

This section covers the most frequent problems encountered during daily use.

### ❌ 9.1 — Unable to Connect over SSH

Example:

```text
ssh: Could not resolve hostname proxmark.local
```

Check:

1. The Raspberry Pi is powered on.
2. It is connected to Wi-Fi.
3. Your computer can access the same network.
4. The hostname `proxmark.local` is spelled correctly.

Then try connecting using the IP address.

### ❌ 9.2 — SSH Connection Refused

Example:

```text
Connection refused
```

This may indicate that the SSH service is not accessible at the specified address.

Check the Raspberry Pi's SSH configuration and network address.

If you have local access to the Raspberry Pi, check the service with:

```bash
systemctl status ssh
```

### ❌ 9.3 — Proxmark3 Not Detected

Start with:

```bash
lsusb
```

Then:

```bash
ls -l /dev/ttyACM*
```

And:

```bash
sudo dmesg | tail -n 30
```

Check the connections and power supply.

If the Proxmark3 appears under another serial port, adjust the client configuration if necessary.

### ❌ 9.4 — The `pm` Command Does Not Work

Example:

```text
pm: command not found
```

Check whether the alias exists:

```bash
alias pm
```

If the alias is missing, refer to Step 2.

You can also run:

```bash
tmux new-session -A -s proxmark
```

### ❌ 9.5 — The `pm3` Command Does Not Work

Example:

```text
pm3: command not found
```

Check the client path:

```bash
command -v pm3
```

If no path is displayed, check the installation and `PATH` configuration described in Step 3.

You can try launching the client directly:

```bash
~/proxmark3-new/pm3
```

### ❌ 9.6 — A tmux Session Is Already Open

If the session is already attached to another terminal, detach it from that terminal by pressing:

```text
Ctrl + B
```

Then:

```text
D
```

Reconnect using:

```bash
pm
```

### ❌ 9.7 — The Iceman Client Stops Responding

If the client stops responding, first check that the SSH connection is still active.

If necessary, try interrupting the current command with:

```text
Ctrl + C
```

> ⚠️ **Warning**
>
> Do not interrupt an ongoing firmware flashing operation.
>
> Interrupting firmware writing may render the device unusable.

If the client is genuinely frozen, you may need to close and restart it.

Avoid unplugging the Proxmark3 before identifying the cause of the problem.

---

## 10. Shut Down the System Properly

The Raspberry Pi uses a microSD card for its operating system.

Suddenly removing power during a write operation may corrupt files.

It is therefore recommended to shut down Linux properly.

### 🛑 10.1 — Exit the Iceman Client

Inside the Proxmark3 client:

```text
quit
```

You return to the Linux terminal.

### 🔚 10.2 — Exit the tmux Session

If you want to terminate the session:

```bash
exit
```

You return to the regular SSH terminal.

### 🔌 10.3 — Shut Down the Raspberry Pi

Run:

```bash
sudo shutdown -h now
```

The Raspberry Pi begins its shutdown procedure.

The SSH connection will automatically close.

Wait until system activity has stopped before disconnecting the power supply.

> ⚠️ **Important**
>
> The Raspberry Pi Zero 2 W does not have a built-in power button that automatically disconnects its power supply.
>
> The `shutdown` command stops the operating system, but electrical power remains physically connected.

### 🔄 10.4 — Restart the Raspberry Pi

To restart without disconnecting power:

```bash
sudo reboot
```

Wait until the system is accessible again before reconnecting over SSH.

---

## 11. Simplified Daily Procedure

Here is the procedure to remember for everyday Proxmark3 use.

### 🚀 Startup

**1. Power on the Raspberry Pi and Proxmark3.**

Wait for the system to boot.

**2. Connect over SSH:**

```bash
ssh piuser@proxmark.local
```

**3. Open the persistent session:**

```bash
pm
```

**4. Launch Iceman if the client is not already running:**

```bash
pm3
```

**5. Use the Proxmark3.**

For example:

```text
hw version
```

### 🔌 Temporary Disconnection

To keep the session running:

```text
Ctrl + B
```

Then:

```text
D
```

You can then close the SSH connection.

### 🔄 Reconnection

From the same computer or another one:

```bash
ssh piuser@proxmark.local
```

Then:

```bash
pm
```

You return to your session.

### 🛑 Complete Shutdown

Exit the Iceman client:

```text
quit
```

Exit the tmux session:

```bash
exit
```

Then shut down the Raspberry Pi:

```bash
sudo shutdown -h now
```

---

## 12. Final Checklist

Before moving on to the next chapter, verify that:

- [ ] The Raspberry Pi boots correctly.
- [ ] The Proxmark3 Easy is connected and powered.
- [ ] The Raspberry Pi automatically joins Wi-Fi.
- [ ] SSH access works from a computer.
- [ ] The Proxmark3 is detected over USB.
- [ ] The `pm` command opens a tmux session.
- [ ] The `pm3` command launches Iceman.
- [ ] The `hw version` and `hw status` commands work.
- [ ] An authorized NFC reading test works.
- [ ] You know how to detach a tmux session.
- [ ] You know how to recover an existing session.
- [ ] You know how to shut down the Raspberry Pi properly.

---

## ✅ Step 4 Complete — Proxmark3 Ready for Daily Use

Our installation can now be operated remotely from a Windows, macOS, or Linux computer.

We know how to:

- Start the Raspberry Pi and Proxmark3.
- Access the system using SSH.
- Use the `pm` and `pm3` commands.
- Keep an Iceman session running with tmux.
- Perform diagnostics and RFID/NFC tests.
- Troubleshoot common connection problems.
- Shut down the Raspberry Pi properly.

### 🚀 In the Next Step

We will learn how to use the same installation from a mobile device:

- **iPhone and iPad (Apple / iOS / iPadOS).**
- **Android smartphones and tablets.**

We will explain how to install a suitable SSH client, save a connection to the Raspberry Pi, and use the Proxmark3 from a mobile terminal.

The goal is to have a **wireless Proxmark3 Easy controlled from a smartphone**, without modifying the hardware.

---

## 📚 References

- [Official Proxmark3 Iceman Repository — RfidResearchGroup](https://github.com/RfidResearchGroup/proxmark3)
- [Step 1 — Hardware and Connections](01-hardware.md)
- [Step 2 — Raspberry Pi Installation](02-raspberry-installation.md)
- [Step 3 — Proxmark3 Iceman Installation](03-proxmark3-installation.md)

---

## 🧭 Navigation

| ⬅️ Previous Step | 🏠 Table of Contents | 🌐 Français |
|---|---|---|
| [Step 3 — Proxmark3 Iceman Installation](03-proxmark3-installation.md) | [Back to Home](../../README.md) | [Étape 4 — Utilisation quotidienne](../fr/04-utilisation-quotidienne.md) |

> 💡 **Temporary navigation**
>
> Navigation links to the following chapters will be harmonized once all tutorial pages have been created.

---

📡 **Project: Proxmark3 Easy — Raspberry Pi Zero 2 W Wireless**
