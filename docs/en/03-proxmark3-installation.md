# 🔧 Step 3 — Installing and Updating Proxmark3 Iceman

🌐 **Language:** [🇫🇷 Français](../fr/03-installation-proxmark3.md) | 🇬🇧 English

[🏠 Back to Table of Contents](../../README.md)

---

## 📖 Introduction

This third chapter explains how to install, compile, and use the **Proxmark3 Iceman** firmware, developed by the **RfidResearchGroup** community, on a **Proxmark3 Easy** connected to a **Raspberry Pi Zero 2 W**.

The Raspberry Pi communicates with the Proxmark3 over USB and allows you to control it remotely from a computer or smartphone using Wi-Fi and SSH.

We will also explain how to update an existing installation while preserving the original bootloader.

The procedure described here corresponds to the hardware configuration used and tested in this project.

### 🎯 Objectives

By the end of this chapter, you will be able to:

- Verify the Proxmark3 Easy USB connection.
- Install the required compilation tools.
- Download the official Iceman source code.
- Configure the build for a generic Proxmark3 Easy.
- Compile the client and firmware.
- Back up an existing installation.
- Update only the ARM firmware.
- Verify that the Proxmark3 works correctly.
- Configure the `pm3` command.
- Use the Iceman client inside a `tmux` session.

> ⚠️ **Important warning**
>
> Flashing incompatible firmware may render the Proxmark3 unusable.
>
> Always verify your device model and hardware configuration before flashing.
>
> Never disconnect the Proxmark3 during a firmware update.
>
> This procedure was tested on the hardware used in this project. Compatibility with every Proxmark3 Easy variant cannot be guaranteed.

---

## 1. Reference Configuration

This tutorial uses the following configuration:

| Component | Configuration |
|---|---|
| Raspberry Pi | Zero 2 W |
| Operating system | Raspberry Pi OS Lite |
| Architecture | ARM64 / aarch64 |
| Debian base | Debian 13 (Trixie) |
| Proxmark3 | Generic Easy |
| Microcontroller | AT91SAM7S512 |
| Internal flash memory | 512 KB |
| FPGA | Xilinx XC2S30 |
| Firmware | Iceman — RfidResearchGroup |
| Connection | USB through a powered OTG adapter |
| Remote access | SSH |
| Persistent sessions | tmux |

### 💡 Why Use Iceman?

Iceman is an alternative Proxmark3 firmware maintained by the RfidResearchGroup community.

It provides:

- A comprehensive command-line interface.
- Numerous RFID and NFC analysis features.
- Hardware diagnostic tools.
- Protocol reading and analysis functions.
- Regular improvements and updates.

The Iceman client runs directly in the Raspberry Pi Linux terminal.

A graphical desktop environment is therefore unnecessary.

### 📚 Prerequisites

Before continuing, complete:

- [Step 1 — Hardware and Connections](01-hardware.md)
- [Step 2 — Raspberry Pi Installation](02-raspberry-installation.md)

The Raspberry Pi must be accessible over SSH and connected to the Internet.

---

## 2. Verify the USB Connection

The Proxmark3 Easy must be connected to the Raspberry Pi Zero 2 W USB data port.

The connections are described in Step 1.

### 🔍 2.1 — List USB Devices

From the SSH terminal:

```bash
lsusb
```

This command displays USB devices detected by Linux.

### 🔍 2.2 — Check the Serial Port

Run:

```bash
ls -l /dev/ttyACM*
```

In our installation, the Proxmark3 appears as:

```text
/dev/ttyACM0
```

The device number may vary depending on the connected peripherals.

> 💡 **Note**
>
> If no `/dev/ttyACM*` device appears, check:
>
> - The USB cable.
> - The powered OTG adapter.
> - The Raspberry Pi power supply.
> - The USB port being used.
> - The Proxmark3 operating mode.
>
> A missing device entry does not necessarily mean that the Proxmark3 is defective.

---

## 3. Install the Compilation Tools

Building Iceman requires several development tools.

### 🔄 3.1 — Update Package Lists

Run:

```bash
sudo apt update
```

### 📦 3.2 — Install Dependencies

Run:

```bash
sudo apt install -y git build-essential make gcc-arm-none-eabi \
  libreadline-dev libusb-0.1-4-dev libpcsclite-dev pkg-config \
  libbz2-dev liblz4-dev libssl-dev python3 python3-dev
```

These packages are used to:

- Download source code with Git.
- Compile Linux applications.
- Compile ARM firmware.
- Provide libraries required by the client.
- Run the necessary Python tools.

> 💡 **Note**
>
> Dependencies may change in future Iceman releases.
>
> If the build reports a missing library, consult the official repository instructions.

### 🔍 3.3 — Verify the ARM Compiler

Run:

```bash
arm-none-eabi-gcc --version
```

The terminal should display the compiler version.

This confirms that the ARM firmware compiler is available.

---

## 4. Download Proxmark3 Iceman

Iceman is developed and maintained by the RfidResearchGroup community.

🔗 [Official Proxmark3 Iceman Repository](https://github.com/RfidResearchGroup/proxmark3)

### 📥 4.1 — Download the Source Code

Go to your home directory:

```bash
cd ~
```

Clone the official repository:

```bash
git clone https://github.com/RfidResearchGroup/proxmark3.git proxmark3-new
```

Enter the directory:

```bash
cd ~/proxmark3-new
```

> 💡 **Why use `proxmark3-new`?**
>
> This directory name allows an existing installation in `~/proxmark3` to remain untouched.
>
> You can prepare and test the new version without immediately deleting the previous files.

### 🔍 4.2 — Check the Downloaded Version

Run:

```bash
git log -1 --oneline
```

This command displays the latest commit in the downloaded working tree.

### 📌 Version Used in This Project

Our tested installation used commit:

```text
754bbae3d
```

Dated October 30, 2026.

Later releases may introduce differences in compilation or operation.

> ⚠️ **Important**
>
> `git clone` downloads the version available when the command is executed.
>
> It does not guarantee that you are using the same commit as our tested installation.
>
> To reproduce that version, you can select the corresponding commit:
>
> ```bash
> git checkout 754bbae3d
> ```
>
> Run this command from `~/proxmark3-new`.
>
> For a new installation, use a version verified to work with your hardware.

---

## 5. Configure the Proxmark3 Easy

The firmware must be compiled for the correct hardware configuration.

### 🔧 5.1 — Create the Configuration File

Go to the project directory:

```bash
cd ~/proxmark3-new
```

Create `Makefile.platform`:

```bash
cat > Makefile.platform <<'EOF_CONFIG'
PLATFORM=PM3GENERIC
LED_ORDER=PM3EASY
EOF_CONFIG
```

### 📋 5.2 — Build Parameters

| Parameter | Value | Description |
|---|---|---|
| PLATFORM | PM3GENERIC | Generic Proxmark3 platform |
| LED_ORDER | PM3EASY | Proxmark3 Easy LED configuration |

These settings correspond to the generic Proxmark3 Easy used in this project.

Our build did not enable:

```text
PLATFORM_EXTRAS=FLASH
```

Even though the hardware detected external flash memory.

> ⚠️ **Warning**
>
> Do not automatically reuse these settings for another Proxmark3 model.
>
> Hardware variants may use different components or configurations.

### 🔍 5.3 — Verify the Configuration

Run:

```bash
cat Makefile.platform
```

Expected output:

```text
PLATFORM=PM3GENERIC
LED_ORDER=PM3EASY
```

If both lines appear correctly, you can proceed with compilation.

---

## 6. Compile the Client and Firmware

Compilation may take several minutes on a Raspberry Pi Zero 2 W.

### 🛠️ 6.1 — Start Compilation

From the project directory:

```bash
cd ~/proxmark3-new
```

Run:

```bash
make -j2
```

Immediately after compilation finishes, check its exit code:

```bash
echo $?
```

An exit code of `0` indicates that compilation completed successfully.

> ⚠️ **Important**
>
> Run `echo $?` immediately after `make -j2`, without executing any other command between them.
>
> If compilation fails, resolve the error before proceeding to firmware flashing.

The `-j2` option allows two compilation jobs to run simultaneously.

This limits parallelism to account for the Raspberry Pi Zero 2 W's available resources.

> 💡 **Tip**
>
> For a long compilation, use a `tmux` session as explained in Step 2.
>
> This allows you to recover the terminal after an SSH disconnection.

### 🔍 6.2 — Check the Result

Wait for compilation to finish.

Make sure no errors are displayed.

The `make` command must return exit code `0` to indicate a successful build.

If compilation fails, resolve the error before continuing.

### 📦 6.3 — Generated Files

The main files used in this tutorial are:

| File | Description |
|---|---|
| `armsrc/obj/fullimage.elf` | ARM firmware |
| `bootrom/obj/bootrom.elf` | Bootloader |
| `pm3` | Client launcher script |
| `pm3-flash` | Firmware flashing tool |

### ⚠️ Bootloader Precaution

Compilation may generate a new bootloader.

**This does not mean that the bootloader must be flashed.**

In our project, we chose to preserve the existing bootloader.

This avoids making unnecessary changes to a sensitive part of the Proxmark3.

---

## 7. Back Up a Previous Installation

This section applies to users who already have a working Iceman installation.

For a first-time installation, you can proceed to the next section.

### 💾 7.1 — Create a Backup

If the previous installation is located in:

```text
~/proxmark3
```

Go to your home directory:

```bash
cd ~
```

Create an archive:

```bash
tar -czf proxmark3-backup-$(date +%F).tar.gz proxmark3
```

This creates a compressed archive containing the previous directory.

### 🔍 7.2 — Verify the Backup

Run:

```bash
tar -tzf proxmark3-backup-$(date +%F).tar.gz > /dev/null && echo "Archive OK"
```

Expected output:

```text
Archive OK
```

### ⚠️ Backup Limitations

This archive contains the software files from the previous installation.

It is **not a complete backup of the Proxmark3's internal memory**.

Keep your personal data and configuration files backed up separately.

Never publish private RFID dumps or authentication keys on GitHub.

---

## 8. Update the ARM Firmware

We will now transfer the new firmware to the Proxmark3 Easy.

In our tested installation, we chose to update only the ARM firmware while preserving the existing bootloader.

### ⚠️ Warning Before Flashing

Flashing firmware involves risks.

Before continuing:

- Verify the exact Proxmark3 model.
- Verify the build configuration.
- Make sure compilation completed successfully.
- Use a stable power supply.
- Never disconnect the Proxmark3 during flashing.
- Do not flash the bootloader unless there is a clearly identified need.

### 🔍 8.1 — Check the Device

From the new repository:

```bash
cd ~/proxmark3-new
```

Run:

```bash
./pm3-flash --list
```

Verify that the Proxmark3 is detected.

In our installation, it appeared as:

```text
/dev/ttyACM0
```

The device number may vary.

### ⚡ 8.2 — Flash Only the ARM Firmware

> ⚠️ **Before Flashing**
>
> Verify that `armsrc/obj/fullimage.elf` was generated from a build configured specifically for your Proxmark3 Easy.
>
> Make sure the Iceman client and ARM firmware come from the same source code version.
>
> This procedure preserves the existing bootloader. Do not apply it to another hardware variant without checking compatibility.
>
> If your Proxmark3 is already working correctly and you do not need a new version, firmware flashing is not necessary.
>
> Ensure that the Raspberry Pi has a stable power supply and that the USB connection will not be interrupted.

**Continue only if your hardware matches the documented configuration and the firmware has been compiled successfully.**

Run:

```bash
./pm3-flash armsrc/obj/fullimage.elf
```

Wait for the transfer to complete.

In our installation, the operation ended with:

```text
All done
```

### 🔒 8.3 — Preserve the Bootloader

In this procedure, we do not use the option:

```text
-b
```

This option enables writing to the bootloader area.

Our successful update replaced only the ARM firmware while preserving the previous bootloader.

> ⚠️ **Important**
>
> Do not automatically flash `bootrom.elf`.
>
> A bootloader update requires a dedicated procedure appropriate for the specific hardware model.

---

## 9. Verify Operation

After flashing, we will verify communication between the Raspberry Pi and the Proxmark3.

### 💻 9.1 — Start the Client

From the new repository:

```bash
cd ~/proxmark3-new
```

Run:

```bash
./pm3
```

The Iceman client should start and establish communication with the Proxmark3.

### 🔍 9.2 — Check Versions

Inside the Proxmark3 client:

```text
hw version
```

Then:

```text
hw status
```

These commands display hardware and firmware information.

### 📋 9.3 — Check Version Consistency

Verify:

- The client version.
- The ARM firmware version.
- The bootloader version.
- The hardware information.
- The device's general status.

The **Client** and **OS** versions should match the newly built code.

The **Bootrom** version may be older.

This is expected because we deliberately preserved the previous bootloader.

> 💡 **Note**
>
> The word `suspect` in some version information does not necessarily indicate a hardware failure.
>
> Interpret this information alongside the other diagnostic results.

### 📡 9.4 — Test NFC Reading

To verify high-frequency operation, use an ISO 14443-A NFC tag that you own.

Inside the Proxmark3 client:

```text
hf 14a reader
```

Place the tag near the HF antenna.

The Proxmark3 should attempt to detect the tag and display the available information.

This is a non-destructive reading operation.

> ⚠️ **Responsible Use**
>
> Only test tags and equipment that you own or are explicitly authorized to examine.

### ✅ Results of Our Installation

Our tests confirmed:

- Working USB communication.
- Successful FPGA image loading.
- Successful Iceman client startup.
- Communication with the ARM firmware.
- NFC reading after flashing.

---

## 10. Use the New Client with the `pm3` Command

We will now simplify how the Iceman client is launched.

The objective is to run:

```bash
pm3
```

from any directory in the terminal.

### 🔧 10.1 — Add the New Directory to PATH

If a previous installation exists, we want the new client to take priority.

Run:

```bash
if ! grep -Fqx 'export PATH="$HOME/proxmark3-new:$PATH"' ~/.bashrc; then
  echo 'export PATH="$HOME/proxmark3-new:$PATH"' >> ~/.bashrc
fi
```

This command adds the new directory to `PATH` only if the line does not already exist.

### 🔄 10.2 — Reload the Configuration

Run:

```bash
source ~/.bashrc
```

### 🔍 10.3 — Verify the Command

Run:

```bash
command -v pm3
```

The result should point to the new directory.

For example:

```text
/home/piuser/proxmark3-new/pm3
```

The username `piuser` is the example used throughout this tutorial.

If you chose another username, the path will be different.

### 💡 Keep the Previous Installation

If an older installation exists in:

```text
~/proxmark3
```

You can still access it explicitly:

```bash
~/proxmark3/pm3
```

> ⚠️ **Warning**
>
> Avoid using an older Iceman client with newer firmware.
>
> Version differences may cause compatibility problems.

---

## 11. Use Proxmark3 with tmux

In Step 2, we installed `tmux` and created the alias:

```bash
pm
```

This alias automatically creates or reconnects to a persistent session named `proxmark`.

### 💻 11.1 — Connect to the Raspberry Pi

From a computer or smartphone with an SSH client:

```bash
ssh piuser@proxmark.local
```

### 🔄 11.2 — Open or Recover the Session

Run:

```bash
pm
```

If a session already exists, you will reconnect to it.

Otherwise, a new session will be created.

### 📡 11.3 — Start Proxmark3 Iceman

Inside the `tmux` session:

```bash
pm3
```

The Iceman client should start.

### 🔌 11.4 — Detach the Session

To disconnect temporarily without stopping the program:

1. Press `Ctrl + B`.
2. Release the keys.
3. Press `D`.

The session continues running on the Raspberry Pi.

### 🔄 11.5 — Resume Your Work

During your next SSH connection:

```bash
pm
```

You will return to the previous session.

> 💡 **Important**
>
> `tmux` preserves programs running inside its session when SSH disconnects.
>
> It does not maintain the network connection itself.
>
> The Raspberry Pi must remain powered on and the Proxmark3 connected.

For more information, see:

[Step 2 — Raspberry Pi Configuration and tmux](02-raspberry-installation.md)

---

## 12. Final Checklist

Before considering the installation complete, verify that:

- [ ] The Proxmark3 Easy is detected by the Raspberry Pi.
- [ ] Compilation dependencies are installed.
- [ ] The Iceman repository has been downloaded.
- [ ] `Makefile.platform` matches the hardware.
- [ ] Compilation completed without errors.
- [ ] The previous installation was backed up if necessary.
- [ ] The ARM firmware was updated successfully, if flashing was required.
- [ ] The existing bootloader was preserved.
- [ ] The Iceman client starts correctly.
- [ ] `hw version` and `hw status` work.
- [ ] An authorized NFC reading test was performed.
- [ ] The `pm3` command points to the new client.
- [ ] The `pm` command reconnects to a `tmux` session.

---

## ✅ Step 3 Complete — Proxmark3 Iceman Installed

The Raspberry Pi Zero 2 W is now configured to operate the Proxmark3 Easy remotely.

You now have:

- A Raspberry Pi accessible over Wi-Fi and SSH.
- A Proxmark3 Easy connected over USB.
- The Iceman client and firmware.
- A configuration adapted to the hardware.
- A previous installation preserved when applicable.
- A `pm3` command to launch the client.
- A `pm` command to recover a persistent session.

---

## 📚 References

- [Official Proxmark3 Iceman Repository — RfidResearchGroup](https://github.com/RfidResearchGroup/proxmark3)
- [Step 1 — Hardware and Connections](01-hardware.md)
- [Step 2 — Raspberry Pi Installation](02-raspberry-installation.md)

---

## 🧭 Navigation

| ⬅️ Previous Step | 🏠 Table of Contents | ➡️ Next Step | 🌐 Français |
|---|---|---|---|
| [Step 2 — Raspberry Pi Installation](02-raspberry-installation.md) | [Back to Home](../../README.md) | [Step 4 — Daily Usage](04-daily-usage.md) | [Étape 3 — Installation de Proxmark3 Iceman](../fr/03-installation-proxmark3.md) |

---

📡 **Project: Proxmark3 Easy — Raspberry Pi Zero 2 W Wireless**
