# 🔧 Installing Proxmark3 Iceman

🌐 **Language:** [🇫🇷 Français](../fr/03-installation-proxmark3.md) | 🇬🇧 English

[🏠 Back to Home](../../README.md)

---

## 1. Overview

This chapter explains how to install, compile, and update **Proxmark3 Iceman** firmware on a Raspberry Pi Zero 2 W.

The Proxmark3 Easy connects to the Raspberry Pi through USB. Its command-line interface can be accessed remotely using SSH.

The following procedure was tested on the hardware used in this project.

### Reference Configuration

| Component        | Configuration                   |
| ---------------- | ------------------------------- |
| Raspberry Pi     | Zero 2 W                        |
| Operating System | Raspberry Pi OS Lite 64-bit     |
| Architecture     | ARM64 / aarch64                 |
| Proxmark3        | Easy                            |
| Microcontroller  | AT91SAM7S512                    |
| Internal Flash   | 512 KB                          |
| FPGA             | Xilinx XC2S30                   |
| Firmware         | Iceman — RfidResearchGroup      |
| Connection       | USB through powered OTG adapter |
| Remote Access    | SSH                             |

### Objectives

We will:

1. Verify the USB connection.
2. Install the required development tools.
3. Download Iceman firmware.
4. Configure the build for Proxmark3 Easy.
5. Compile the client and firmware.
6. Back up an existing installation.
7. Flash the new firmware.
8. Verify proper operation.
9. Simplify client startup.

> ⚠️ **Warning:** flashing incompatible firmware may render your Proxmark3 unusable. Verify your hardware model and never disconnect the device while flashing.

---

## 2. Verify the USB Connection

The Proxmark3 Easy must be connected to the Raspberry Pi Zero 2 W USB data port.

The wiring is described in [Chapter 1 — Required Hardware and Connections](01-hardware.md).

### Detect the Device

From your SSH terminal:

```bash
lsusb
```

Then:

```bash
ls /dev/ttyACM*
```

In our setup, the Proxmark3 appears as:

```text
/dev/ttyACM0
```

The device number may vary.

If no device is detected, check the USB cable, powered OTG adapter, and power supply.

---

## 3. Install the Required Tools

Compiling Iceman requires several development packages.

### Update the System

```bash
sudo apt update
```

### Install Dependencies

```bash
sudo apt install -y git build-essential make gcc-arm-none-eabi \
  libreadline-dev libusb-0.1-4-dev libpcsclite-dev pkg-config \
  libbz2-dev liblz4-dev libssl-dev python3 python3-dev
```

Dependencies may change between Iceman releases.

If compilation fails because of a missing library, consult the official repository documentation.

### Verify the ARM Compiler

```bash
arm-none-eabi-gcc --version
```

The compiler version should be displayed.

---

## 4. Download Proxmark3 Iceman

The firmware is developed and maintained by the **RfidResearchGroup** community.

🔗 [Official Proxmark3 Iceman Repository](https://github.com/RfidResearchGroup/proxmark3)

### Download the Source Code

Go to your home directory:

```bash
cd ~
```

Clone the repository:

```bash
git clone https://github.com/RfidResearchGroup/proxmark3.git proxmark3-new
```

Enter the directory:

```bash
cd ~/proxmark3-new
```

### Check the Downloaded Version

```bash
git log -1 --oneline
```

This displays the latest commit.

### Version Used in This Project

Our tested installation used commit:

```text
754bbae3d
```

Dated October 30, 2026.

Later releases may differ.

> 💡 **Tip:** if an older installation already exists in `~/proxmark3`, keep that directory and use `~/proxmark3-new` for the new version.

---

## 5. Configure the Proxmark3 Easy

The firmware must be compiled for the correct hardware.

### Create the Configuration File

Enter the project directory:

```bash
cd ~/proxmark3-new
```

Create `Makefile.platform`:

```bash
cat > Makefile.platform <<'EOF'
PLATFORM=PM3GENERIC
LED_ORDER=PM3EASY
EOF
```

### Configuration Parameters

| Parameter | Value      | Description                      |
| --------- | ---------- | -------------------------------- |
| PLATFORM  | PM3GENERIC | Generic Proxmark3 platform       |
| LED_ORDER | PM3EASY    | Proxmark3 Easy LED configuration |

These settings match the Proxmark3 Easy used in this project.

Our build did not enable `PLATFORM_EXTRAS=FLASH`, even though external flash memory was detected.

> ⚠️ **Important:** do not automatically use these settings for other Proxmark3 models.

---

## 6. Compile the Firmware

Compilation may take several minutes on a Raspberry Pi Zero 2 W.

### Start Compilation

```bash
cd ~/proxmark3-new
```

Then:

```bash
make -j2
```

The `-j2` option runs two compilation jobs simultaneously.

This limits resource usage on the Raspberry Pi Zero 2 W.

### Check the Result

When compilation finishes:

```bash
echo $?
```

An exit code of `0` indicates that the previous command completed successfully.

### Generated Files

| File                       | Description            |
| -------------------------- | ---------------------- |
| `armsrc/obj/fullimage.elf` | ARM firmware           |
| `bootrom/obj/bootrom.elf`  | Bootloader             |
| `pm3`                      | Client launcher        |
| `pm3-flash`                | Firmware flashing tool |

> ⚠️ **Warning:** compiling a new bootloader does not mean it must be flashed.

In this project, we kept the previous bootloader.

---

## 7. Back Up an Existing Installation

This step is intended for users who already have a working Iceman installation.

For a first-time installation, continue to the next section.

### Create a Backup Archive

Go to your home directory:

```bash
cd ~
```

Then:

```bash
tar -czf proxmark3-backup-$(date +%F).tar.gz proxmark3
```

This creates a compressed archive of the existing `proxmark3` directory.

### Verify the Archive

```bash
tar -tzf proxmark3-backup-$(date +%F).tar.gz > /dev/null && echo "Archive OK"
```

Expected output:

```text
Archive OK
```

### Important

This archive contains the previous installation's software files.

It is **not a complete backup of the Proxmark3 internal flash memory**.

Keep personal data separately.

Never publish private RFID dumps or keys on GitHub.

---

## 8. Flash the New Firmware

We will now transfer the new firmware to the Proxmark3 Easy.

For our tested setup, we chose to **update only the ARM firmware**, leaving the existing bootloader unchanged.

### Detect the Proxmark3

```bash
cd ~/proxmark3-new
```

Then:

```bash
./pm3-flash --list
```

The Proxmark3 should be detected.

In our case:

```text
1: /dev/ttyACM0
```

### Start Flashing

```bash
./pm3-flash armsrc/obj/fullimage.elf
```

Wait for the operation to complete.

### Expected Result

```text
Flashing [===============================================================] 100%

[+] All done
[=] Have a nice day!
```

The new firmware is now installed.

### Why Keep the Bootloader?

The bootloader is responsible for loading firmware, among other functions.

After our upgrade:

| Component    | Version                         |
| ------------ | ------------------------------- |
| Client       | October 2026                    |
| ARM Firmware | October 2026                    |
| FPGA Images  | November 2026, locally compiled |
| Bootloader   | June 2026, preserved            |

This combination worked successfully on our hardware.

> ⚠️ **Important:** do not add `-b` to the flashing command. This option enables writing to the bootloader area.

Never disconnect the device during flashing.

---

## 9. Verify the Installation

After flashing, we must verify that the new client communicates correctly with the Proxmark3.

### Launch the New Client

```bash
cd ~/proxmark3-new
```

Then:

```bash
./pm3
```

The Proxmark3 client interface should appear.

### Check Firmware Versions

At the Proxmark3 prompt:

```text
hw version
```

This command displays the client, ARM firmware, bootloader, and FPGA versions.

The client and ARM firmware should match the new release.

The bootloader may display an older version if it was preserved.

### Check Hardware Status

```text
hw status
```

This command provides information about:

* Available memory.
* FPGA configuration.
* Radio settings.
* Communication between the Raspberry Pi and Proxmark3.

### Test NFC Reading

Place your own ISO 14443-A compatible NFC tag on the antenna.

Run:

```text
hf 14a reader
```

The Proxmark3 should detect the tag and display the available information.

This command reads the tag without modifying it.

### Results from Our Installation

Our tests confirmed:

* Successful client operation.
* Working USB communication.
* Correct FPGA image loading.
* Successful NFC reading.
* Compatibility with the preserved bootloader.

> 💡 A `suspect` suffix may appear in version information. This alone does not indicate defective hardware.

---

## 10. Simplify Proxmark3 Startup

After installation, we want to launch the new client from any directory using:

```bash
pm3
```

### Check the Current Command

First, exit the Proxmark3 client:

```text
quit
```

From the Linux terminal:

```bash
type -a pm3
```

If an older installation exists, the command may still point to:

```text
/home/piuser/proxmark3/pm3
```

### Prioritize the New Installation

We will add the new directory to the beginning of the `PATH` variable.

To avoid duplicate entries in the configuration file:

```bash
if ! grep -Fqx 'export PATH="$HOME/proxmark3-new:$PATH"' ~/.bashrc; then
  echo 'export PATH="$HOME/proxmark3-new:$PATH"' >> ~/.bashrc
fi
```

Reload the configuration:

```bash
source ~/.bashrc
```

### Verify

```bash
command -v pm3
```

Expected output:

```text
/home/piuser/proxmark3-new/pm3
```

Replace `piuser` with your actual username.

### Daily Usage

You can now launch Proxmark3 with:

```bash
pm3
```

The old installation remains available in its original directory.

> 💡 To keep your Proxmark3 session running after an SSH disconnection, use `tmux` and the simplified `pm` command described in [Chapter 2 — Installing Raspberry Pi OS](02-raspberry-installation.md).

---

## 11. Command Summary

| Command              | Function                                              |
| -------------------- | ----------------------------------------------------- |
| `pm3`                | Launch the Proxmark3 client                           |
| `hw version`         | Display firmware versions                             |
| `hw status`          | Check hardware status                                 |
| `hf 14a reader`      | Detect a compatible NFC tag                           |
| `./pm3-flash --list` | Detect the device for flashing                        |
| `make -j2`           | Compile Iceman                                        |
| `pm`                 | Create or restore a tmux session, after configuration |

The `hw version`, `hw status`, and `hf 14a reader` commands run inside the Proxmark3 client.

Other commands run in the Linux terminal.

---

## ✅ Installation Complete

The Proxmark3 Easy is now running Iceman firmware through the Raspberry Pi Zero 2 W.

Our setup now includes:

* Recent Iceman firmware.
* A client compiled for Raspberry Pi OS 64-bit.
* Working USB communication.
* Functional RFID/NFC reading.
* A preserved backup of the previous installation.
* Simplified startup using `pm3`.
* Persistent SSH terminal sessions using `tmux`.

---

## ➡️ Next Step

**Using Proxmark3 Easy Remotely**

We will learn how to access the Proxmark3 from a computer or smartphone, use essential commands, and organize remote working sessions.

[🏠 Back to Home](../../README.md)
