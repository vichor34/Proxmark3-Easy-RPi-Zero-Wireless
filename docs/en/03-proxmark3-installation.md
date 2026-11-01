# 03 — Install and update Proxmark3 Iceman

[Version française](../fr/03-installation-proxmark3.md)

This chapter explains how to build and run the **Iceman firmware (RfidResearchGroup)** on a **Proxmark3 Easy** connected by USB to a **Raspberry Pi Zero 2 W** running 64-bit Raspberry Pi OS Lite. It also covers a cautious upgrade from an existing installation **without overwriting the bootloader**.

> **Warning:** flashing carries a risk of making your device unusable if the wrong firmware or settings are used. Confirm your hardware model before writing anything, and do not unplug the device while flashing.

## 1. Check the USB connection

Connect the Proxmark3 Easy to the Raspberry Pi's USB data port using a suitable OTG adapter and power supply.

```bash
ls -l /dev/ttyACM*
```

Our device appeared as `/dev/ttyACM0`; the number may differ.

## 2. Install build tools

Update the system and install the main build dependencies:

```bash
sudo apt update
sudo apt install -y git build-essential make gcc-arm-none-eabi \
  libreadline-dev libusb-0.1-4-dev libpcsclite-dev pkg-config \
  libbz2-dev liblz4-dev libssl-dev python3 python3-dev
```

Dependencies may change as Iceman evolves. Refer to the official repository's build instructions if `make` reports a missing library.

## 3. Download Iceman

For a **fresh installation**, use `~/proxmark3-new`:

```bash
cd ~
git clone https://github.com/RfidResearchGroup/proxmark3.git proxmark3-new
cd ~/proxmark3-new
```

When **upgrading an existing installation**, leave the old `~/proxmark3` directory intact and clone into a separate folder. This keeps the previous software available for recovery.

Our tested build used commit `754bbae3d` (October 30, 2026). Later versions may differ.

## 4. Configure the Proxmark3 Easy

Create `Makefile.platform` in the new repository's root:

```bash
cat > Makefile.platform <<'EOF_CONFIG'
PLATFORM=PM3GENERIC
LED_ORDER=PM3EASY
EOF_CONFIG
```

These settings match the **generic Proxmark3 Easy** tested here (AT91SAM7S512 microcontroller and `xc2s30` FPGA). **Do not apply them blindly to another model.**

Our build did not enable `PLATFORM_EXTRAS=FLASH`, even though external flash memory was detected by the hardware.

## 5. Build the client and firmware

```bash
cd ~/proxmark3-new
make -j2
```

The Raspberry Pi Zero 2 W has limited resources, so `-j2` keeps parallelism moderate. Wait for the build to finish and check for errors.

Relevant outputs include:

- `armsrc/obj/fullimage.elf`: ARM firmware;
- `bootrom/obj/bootrom.elf`: bootloader;
- `pm3`: client launcher.

**Do not flash the bootloader automatically** just because it was built.

## 6. Back up an older installation (upgrades)

If you have an existing `~/proxmark3` directory, archive it before upgrading:

```bash
cd ~
tar -czf proxmark3-backup-$(date +%F).tar.gz proxmark3
```

This backs up the **software files**, not a binary image of the Proxmark3's internal flash. Keep personal data and settings separately; never publish RFID dumps or keys to GitHub.

## 7. Detect the device and flash only the ARM firmware

From the **new repository**:

```bash
cd ~/proxmark3-new
./pm3-flash --list
```

Confirm your device is detected (ours was `/dev/ttyACM0`), then run:

```bash
./pm3-flash armsrc/obj/fullimage.elf
```

Wait for the transfer to complete and for `All done` to appear.

**Important:** do **not** use `-b` in this procedure. That option enables writing to the bootloader area. Our successful upgrade replaced only the ARM firmware and preserved the previous bootloader.

## 8. Verify the installation

Launch the new client:

```bash
cd ~/proxmark3-new
./pm3
```

At the Proxmark3 prompt, enter:

```text
hw version
hw status
```

Check that **Client** and **OS** report the new version. **Bootrom** may show an older version: that is expected when it was preserved.

Next, use **your own ISO 14443-A NFC tag** for a non-destructive read test:

```text
hf 14a reader
```

In our test, USB communication, FPGA images, and NFC reading all worked after flashing. A `suspect` suffix in a version string does not, by itself, indicate a hardware fault.

## 9. Make `pm3` launch the new client

If your `PATH` already contains `~/proxmark3`, put the new directory **before** the old one. For Bash:

```bash
# Only add the line if it is not already present
if ! grep -Fqx 'export PATH="$HOME/proxmark3-new:$PATH"' ~/.bashrc; then
  echo 'export PATH="$HOME/proxmark3-new:$PATH"' >> ~/.bashrc
fi
source ~/.bashrc
command -v pm3
```

Expected output:

```text
/home/victor/proxmark3-new/pm3
```

Replace `victor` with your username as appropriate. The old installation remains accessible via `~/proxmark3/pm3`, but avoid using its old client with the new firmware.

## 10. Keep a terminal session running across SSH disconnects

To install **tmux** and create the short `pm` command, see [Chapter 2 — Raspberry Pi setup](02-raspberry-installation.md). `tmux` keeps processes running on the Raspberry Pi after SSH disconnects; it does not keep the network connection itself alive.

## Reference

Official source code: [RfidResearchGroup/proxmark3](https://github.com/RfidResearchGroup/proxmark3).
