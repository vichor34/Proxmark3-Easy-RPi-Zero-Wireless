# 🔧 Step 6 — Maintenance, Backups and Updates

🌐 **Language:** [🇫🇷 Français](../fr/06-maintenance.md) | 🇬🇧 English

[🏠 Back to Table of Contents](../../README.md)

---

## 📖 Introduction

Our **Proxmark3 Easy + Raspberry Pi Zero 2 W** setup is now operational.

We can use the Proxmark3 remotely from a computer, iPhone, iPad, or Android device.

To keep the installation reliable, it is important to know how to:

- Check the Raspberry Pi's status.
- Back up important files.
- Create a complete microSD card backup.
- Update Raspberry Pi OS.
- Update the Proxmark3 Iceman client.
- Update the Proxmark3 firmware when necessary.
- Restore a working configuration.
- Diagnose hardware and software issues.

This chapter introduces a progressive maintenance procedure adapted to our hardware.

> ⚠️ **Important**
>
> Updating Linux, updating the Iceman client, and flashing the Proxmark3 firmware are three different operations.
>
> They must not be confused.
>
> **If your installation works correctly, there is no need to flash the Proxmark3 regularly.**

### 🎯 Objectives

By the end of this chapter, you will be able to:

- Identify the currently installed versions.
- Check disk space and system status.
- Back up the Raspberry Pi configuration.
- Create a microSD card image.
- Update Linux packages.
- Prepare and compile a new Iceman version.
- Understand the precautions required before flashing.
- Restore a previous version.
- Perform basic troubleshooting in case of failure.

---

## 1. Reference Configuration

This tutorial covers the installation completed in the previous chapters.

| Component | Configuration |
|---|---|
| Raspberry Pi | Zero 2 W |
| Operating system | Raspberry Pi OS Lite 64-bit |
| Proxmark3 | Generic Easy |
| Microcontroller | AT91SAM7S512 |
| Microcontroller flash memory | 512 KB |
| FPGA | Xilinx XC2S30 |
| Firmware | Iceman — RfidResearchGroup |
| Build platform | `PM3GENERIC` |
| LED configuration | `PM3EASY` |
| Project directory | `~/proxmark3-new` |
| Remote access | SSH |
| Persistent sessions | tmux |

### 📚 Prerequisites

You should have completed:

- [Step 2 — Raspberry Pi Installation](02-raspberry-installation.md)
- [Step 3 — Proxmark3 Iceman Installation](03-proxmark3-installation.md)
- [Step 4 — Daily Usage](04-daily-usage.md)

> 💡 **Note**
>
> The commands assume your username is `piuser` and Iceman is installed in `~/proxmark3-new`.
>
> Adjust the paths if your configuration is different.

---

## 2. Check the System Status

Before performing maintenance, we will verify that the Raspberry Pi is operating normally.

### 🔍 2.1 — Check the Operating System Version

Connect via SSH:

```bash
ssh piuser@proxmark.local
```

Display system information:

```bash
cat /etc/os-release
```

Check the Linux kernel version:

```bash
uname -a
```

This information is useful when troubleshooting.

### 💾 2.2 — Check Disk Space

Run:

```bash
df -h
```

This command displays available space on mounted filesystems.

Pay particular attention to the root filesystem:

```text
/
```

Insufficient disk space may prevent:

- Installing updates.
- Compiling Iceman.
- Creating files.
- Normal operation of certain services.

### 🧠 2.3 — Check Memory

Run:

```bash
free -h
```

This command displays available RAM and swap space.

The Raspberry Pi Zero 2 W has limited resources.

Avoid running multiple compilations simultaneously.

### 🌡️ 2.4 — Check Temperature

Run:

```bash
vcgencmd measure_temp
```

This command displays the processor temperature.

Example:

```text
temp=45.0'C
```

The temperature depends on system load and cooling conditions.

### ⚡ 2.5 — Check Power Supply Warnings

Run:

```bash
vcgencmd get_throttled
```

Example:

```text
throttled=0x0
```

A value of `0x0` indicates that no throttling or undervoltage flags are reported.

A different value should be interpreted according to the corresponding status bits.

> ⚠️ **Warning**
>
> An insufficient power supply can cause USB errors and communication interruptions with the Proxmark3.
>
> Resolve these issues before attempting any firmware flashing operation.

### 📡 2.6 — Check the Proxmark3

Open the tmux session:

```bash
pm
```

Start Iceman if necessary:

```bash
pm3
```

Inside the client:

```text
hw version
```

Then:

```text
hw status
```

Record the version information before updating.

---

## 3. Understand the Different Types of Updates

Our installation contains several independent software components.

| Component | Function | Update method |
|---|---|---|
| Raspberry Pi OS | Operating system | `apt` |
| Iceman client | Software running on the Raspberry Pi | Git + compilation |
| ARM firmware | Program running on the Proxmark3 | Flashing |
| FPGA | Programmable logic inside the Proxmark3 | Depends on firmware components |
| Bootloader | Proxmark3 startup program | Specific procedure |

### 🐧 3.1 — Raspberry Pi OS

System updates may include:

- Linux kernel updates.
- Libraries.
- System utilities.
- Security patches.

They do not automatically update the Proxmark3 firmware.

### 💻 3.2 — Iceman Client

The Iceman client runs on the Raspberry Pi.

It sends commands to the Proxmark3.

A new version may introduce:

- Bug fixes.
- New commands.
- Compatibility improvements.
- Changes to communication protocols.

### 📡 3.3 — Proxmark3 Firmware

The firmware runs directly on the Proxmark3.

It must be compatible with:

- The hardware.
- The build configuration.
- The Iceman client.

> ⚠️ **Important**
>
> Using a client and firmware from the same build is recommended.
>
> Mixing different versions may cause errors or incompatibilities.

### 🔒 3.4 — Bootloader

The bootloader starts the device and supports certain programming operations.

It should not be modified without a clearly identified reason.

**In our configuration, we preserve the existing bootloader.**

---

## 4. Back Up Important Information

Before updating, we will create a backup of useful files and settings.

### 📁 4.1 — Create a Backup Directory

On the Raspberry Pi:

```bash
mkdir -p ~/backups
chmod 700 ~/backups
```

Create a timestamped directory:

```bash
BACKUP_DIR="$HOME/backups/$(date +%Y-%m-%d_%H-%M-%S)"
mkdir -p "$BACKUP_DIR"
chmod 700 "$BACKUP_DIR"
```

> 🔐 **Security**
>
> The `chmod 700` command restricts access to these directories to their owner.
>
> This is important because backups may contain network credentials and other sensitive information.

Check the directory:

```bash
echo "$BACKUP_DIR"
```

> 💡 **Tip**
>
> Keep the displayed path.
>
> It will help you locate this backup later.

### 📋 4.2 — Save System Information

Run:

```bash
cat /etc/os-release > "$BACKUP_DIR/os-release.txt"
```

Then:

```bash
uname -a > "$BACKUP_DIR/kernel.txt"
```

Save the installed package list:

```bash
dpkg --get-selections > "$BACKUP_DIR/packages.txt"
```

### 📶 4.3 — Back Up Network Configuration

Save the NetworkManager connection list:

```bash
nmcli connection show > "$BACKUP_DIR/network-connections.txt"
```

This command saves a list of connections, but not necessarily all their settings.

To back up network profiles:

```bash
sudo tar -czf "$BACKUP_DIR/networkmanager.tar.gz" \
  -C /etc NetworkManager/system-connections
```

This command uses administrator privileges to read protected network profiles.

The resulting archive may be owned by `root`. To allow your user to manage the backup:

```bash
sudo chown "$(id -u):$(id -g)" "$BACKUP_DIR/networkmanager.tar.gz"
```

Then protect its contents:

```bash
chmod 600 "$BACKUP_DIR/networkmanager.tar.gz"
```

> 🔐 **Security**
>
> `chmod 600` allows only the file owner to read and modify the archive.
>
> NetworkManager profiles may contain Wi-Fi passwords. Never publish this archive in a GitHub repository.

### 🖥️ 4.4 — Back Up Bash Configuration

Our `~/.bashrc` file contains the `pm` alias and the `PATH` configuration.

Back it up:

```bash
cp ~/.bashrc "$BACKUP_DIR/bashrc"
```

### 📡 4.5 — Record the Iceman Git Version

Go to the project directory:

```bash
cd ~/proxmark3-new
```

Save the current commit:

```bash
git rev-parse HEAD > "$BACKUP_DIR/iceman-commit.txt"
```

Save the repository status:

```bash
git status --short > "$BACKUP_DIR/iceman-status.txt"
```

Save the build configuration:

```bash
cp Makefile.platform "$BACKUP_DIR/Makefile.platform"
```

> ⚠️ **Important**
>
> If `git status --short` displays modified files, identify these changes before updating.
>
> Do not overwrite them without backing them up.

### 📦 4.6 — Create a Backup Archive

Return to your home directory:

```bash
cd ~
```

Create an archive of the backup directory:

```bash
tar -czf "${BACKUP_DIR}.tar.gz" -C "$HOME/backups" "$(basename "$BACKUP_DIR")"
```

Protect the archive:

```bash
chmod 600 "${BACKUP_DIR}.tar.gz"
```

> 🔐 **Important**
>
> This archive contains the previously backed-up files, including any saved Wi-Fi profiles.
>
> Keep it private. Never publish it on GitHub or share it publicly.

Check that the archive exists:

```bash
ls -lh "${BACKUP_DIR}.tar.gz"
```

### 💻 4.7 — Copy the Backup to a Computer

From a terminal running **on your computer**, not inside the Raspberry Pi SSH session:

```bash
scp piuser@proxmark.local:/home/piuser/backups/YYYY-MM-DD_HH-MM-SS.tar.gz .
```

Replace `YYYY-MM-DD_HH-MM-SS` with the actual archive name.

> 💡 **Tip**
>
> A backup stored only on the microSD card does not protect against card failure.
>
> Always copy important backups to another device.

> 🔐 **Backup Protection**
>
> The `scp` command transfers files over an encrypted SSH connection.
>
> However, the resulting archive remains unencrypted on the destination computer.
>
> Store it in a private location, preferably on encrypted storage, and delete unnecessary temporary copies.

---

## 5. Create a Complete microSD Card Backup

The most comprehensive method is to create an image of the microSD card.

This image can be used to restore the Raspberry Pi to an earlier state.

### 💾 5.1 — Why Create an Image?

A complete image preserves:

- The operating system.
- Installed software.
- Network settings.
- User accounts.
- The Iceman client.
- SSH and tmux configurations.

> ⚠️ **Warning**
>
> A microSD card image does not back up the firmware already installed in the Proxmark3's internal memory.
>
> It only backs up the Raspberry Pi's microSD card.

### 🛑 5.2 — Shut Down the Raspberry Pi Properly

Before removing the microSD card:

```bash
sudo shutdown -h now
```

Wait for the system to shut down.

Disconnect the power supply.

Remove the microSD card.

### 💻 5.3 — Create an Image from a Computer

Insert the microSD card into a card reader connected to your computer.

Use software capable of creating a raw image of a storage device.

The procedure depends on your operating system.

On Linux, you can use `dd`, but this command requires extreme care.

Selecting the wrong device can result in data loss.

For beginners, a graphical tool that clearly identifies the source card is preferable.

### 🔍 5.4 — Verify the Backup

Once the image has been created:

1. Verify that the file exists.
2. Check that its size is reasonable.
3. Store it on a separate device.
4. Clearly identify the backup date and version.

Example filename:

```text
proxmark3-rpi-zero2w-2026-11-01.img
```

> 💡 **Tip**
>
> The best verification method is to test the image by restoring it to another compatible microSD card.
>
> An untested backup may be unusable when you need it.

---

## 6. Update Raspberry Pi OS

Operating system updates are independent of Proxmark3 updates.

### 🔐 6.1 — Connect via SSH

From your computer:

```bash
ssh piuser@proxmark.local
```

### 📋 6.2 — Refresh the Package List

Run:

```bash
sudo apt update
```

This command refreshes the list of available package versions.

It does not install updates yet.

### 🔄 6.3 — Install Updates

Run:

```bash
sudo apt full-upgrade
```

Review the proposed changes before confirming.

> ⚠️ **Important**
>
> A significant update may require a reboot.
>
> Avoid performing this operation while an Iceman session is actively running.

### 🔁 6.4 — Reboot if Necessary

```bash
sudo reboot
```

Wait for the Raspberry Pi to restart.

Reconnect:

```bash
ssh piuser@proxmark.local
```

### ✅ 6.5 — Verify Operation

Check the system:

```bash
cat /etc/os-release
```

Then:

```bash
lsusb
```

Verify that the Iceman client still works:

```bash
pm
```

If the Iceman client is not already running inside the tmux session, start it:

```bash
pm3
```

> 💡 **Note**
>
> If `pm` reconnects you to an existing tmux session where Iceman is already running, do not start another instance of the client.
>
> After rebooting the Raspberry Pi, previous tmux sessions no longer exist.

Inside Iceman:

```text
hw version
```

> 💡 **Note**
>
> `apt full-upgrade` updates packages within the currently configured distribution.
>
> Upgrading to a new major Debian or Raspberry Pi OS release is a separate operation requiring its own procedure.

---

## 7. Prepare an Iceman Update

Updating Iceman requires additional precautions.

Our generic Proxmark3 Easy has a specific hardware configuration.

We must preserve the settings appropriate for this model.

### ⚠️ 7.1 — Do Not Modify the Working Installation Directly

The recommended method is to preserve the current directory:

```text
~/proxmark3-new
```

We will prepare a new build in a separate directory.

This makes it easier to return to the previous client.

### 📁 7.2 — Create a New Directory

On the Raspberry Pi:

```bash
cd ~
```

Clone the official repository:

```bash
git clone https://github.com/RfidResearchGroup/proxmark3.git proxmark3-update
```

Enter the directory:

```bash
cd ~/proxmark3-update
```

### 🔍 7.3 — Identify the Downloaded Version

Run:

```bash
git rev-parse HEAD
```

This command displays the exact commit identifier.

Keep this information.

> 💡 **Tip**
>
> For a reproducible installation, use an identified and tested commit rather than choosing a version solely because it is the latest.

### ⚙️ 7.4 — Check the Build Configuration

Open the file:

```bash
nano Makefile.platform
```

Our reference configuration uses:

```makefile
PLATFORM=PM3GENERIC
LED_ORDER=PM3EASY
```

Also verify that the configuration does not contain:

```makefile
PLATFORM_EXTRAS=FLASH
```

> ⚠️ **Important**
>
> These settings correspond to our reference installation.
>
> They must not be applied automatically to every Proxmark3 model.
>
> Check the Iceman repository instructions for the revision you intend to build.

Save the file if you made changes.

### 🔧 7.5 — Compile Iceman

Run:

```bash
make -j2
```

Immediately after compilation, check its exit status:

```bash
echo $?
```

An exit status of `0` indicates that compilation completed successfully.

> ⚠️ **Important**
>
> Run `echo $?` immediately after `make -j2`, without executing another command in between.
>
> If compilation fails, do not proceed with any firmware flashing operation.

The Raspberry Pi Zero 2 W has limited resources.

The `-j2` option limits the number of simultaneous compilation jobs.

Compilation may take several minutes.

### ✅ 7.6 — Verify the Build

If compilation finishes without errors, check that the expected files exist:

```bash
ls -lh pm3
```

Then:

```bash
ls -lh armsrc/obj/fullimage.elf
```

> ⚠️ **Warning**
>
> The presence of these files does not guarantee firmware compatibility with your Proxmark3.
>
> It only confirms that the expected files were generated.

---

## 8. Understand Proxmark3 Firmware Flashing

Flashing means writing a new program to the Proxmark3's internal memory.

This operation is different from compiling the Iceman client.

### 📡 8.1 — Why Flash the Firmware?

Flashing may be necessary when:

- A new client version requires compatible firmware.
- A feature requires an update.
- An important fix affects the firmware.
- The client and firmware are incompatible.

### ⚠️ 8.2 — Flashing Risks

Incorrect flashing may cause:

- Loss of communication with the Proxmark3.
- Unusable firmware.
- Hardware incompatibility.
- The need for a recovery procedure.

Before flashing:

1. Verify the exact Proxmark3 model.
2. Check the build configuration.
3. Confirm that the firmware compiled successfully.
4. Ensure a stable power supply.
5. Check that the USB cable works correctly.
6. Keep a copy of your previous installation.
7. Read the instructions for the Iceman version you are using.

### 🔒 8.3 — Preserve the Bootloader

Our installation uses a generic Proxmark3 Easy.

During the initial installation, we preserved the existing bootloader.

Our reference procedure used only the ARM firmware:

```bash
./pm3-flash armsrc/obj/fullimage.elf
```

This command is shown here to identify the method used in our installation.

**Do not execute it automatically after every compilation.**

> ⚠️ **Warning**
>
> Do not flash the bootloader without a specific reason and a recovery procedure appropriate for your hardware.
>
> Flashing commands, expected files, and precautions may change between repository versions.
>
> Before an actual update, verify the documentation corresponding to the compiled commit.

### 🛑 8.4 — Do Not Interrupt Flashing

During a flashing operation:

- Do not disconnect the USB cable.
- Do not disconnect the power supply.
- Do not reboot the Raspberry Pi.
- Do not start a second Iceman client.
- Do not intentionally interrupt the flashing program.

> 💡 **Recommendation**
>
> If your installation works and you do not need a new feature, keep your current version.
>
> Stability is generally preferable to an unnecessary update.

---

## 9. Verify a New Iceman Version

After a successful update, verify that the system works correctly.

### 🔍 9.1 — Check Versions

Inside the Iceman client corresponding to the new build:

```text
hw version
```

Review the displayed information.

### 🩺 9.2 — Check the Hardware

```text
hw status
```

### 📡 9.3 — Perform an NFC Test

Using an ISO 14443-A tag that belongs to you:

```text
hf 14a reader
```

Place the tag near the HF antenna.

Check that the Proxmark3 detects it correctly.

### 🧪 9.4 — Check the Antennas

If necessary:

```text
hw tune
```

Compare the results with those recorded before the update, if available.

### ✅ 9.5 — Validate the Update

Consider the update successful only when:

- The client starts correctly.
- The Proxmark3 is detected.
- The versions are consistent.
- Diagnostic commands work.
- Authorized RFID/NFC tests work.
- No new errors appear.

---

## 10. Return to a Previous Version

The recovery method depends on what was changed.

### 🐧 10.1 — If Only Raspberry Pi OS Was Updated

A complete microSD card image created before the update is the simplest recovery method.

It restores the system to its state at the time of the backup.

### 💻 10.2 — If Only the Iceman Client Was Updated

If you kept the old directory:

```text
~/proxmark3-new
```

You can restart the previous client from that directory:

```bash
cd ~/proxmark3-new
```

Then:

```bash
./pm3
```

> ⚠️ **Warning**
>
> This assumes that the firmware installed on the Proxmark3 remains compatible with the previous client.
>
> If the firmware was also changed, restoring only the client may not be sufficient.

### 📡 10.3 — If the Firmware Was Updated

Firmware recovery requires a flashing procedure compatible with your hardware.

You may need:

- A previous working build.
- The corresponding firmware files.
- A functioning bootloader.
- A hardware-specific recovery procedure.

> ⚠️ **Important**
>
> Restoring a microSD card backup does not automatically restore the Proxmark3 firmware.
>
> If flashing fails, avoid repeatedly trying unverified firmware files.

### 💾 10.4 — Restore a microSD Card Image

From a computer:

1. Shut down and disconnect the Raspberry Pi.
2. Remove the microSD card.
3. Insert it into a card reader.
4. Use suitable software to write the saved image.
5. Carefully verify the destination device.
6. Wait until writing is complete.
7. Reinsert the card into the Raspberry Pi.
8. Restart and verify operation.

> ⚠️ **Warning**
>
> Restoring an image overwrites the data on the destination card.
>
> Always verify that you selected the correct device.

---

## 11. Advanced Troubleshooting

### ❌ 11.1 — The Proxmark3 No Longer Appears as a USB Device

Run:

```bash
lsusb
```

Then:

```bash
ls -l /dev/ttyACM*
```

Check system messages:

```bash
sudo dmesg | tail -n 50
```

Verify:

- The USB cable.
- The OTG adapter.
- The power supply.
- The USB port.
- The Proxmark3's condition.

### ❌ 11.2 — The Iceman Client No Longer Starts

Check:

```bash
command -v pm3
```

Then:

```bash
ls -lh ~/proxmark3-new/pm3
```

Try:

```bash
cd ~/proxmark3-new
```

Then:

```bash
./pm3
```

If the program starts but cannot communicate with the Proxmark3, review the displayed error messages.

### ❌ 11.3 — The Serial Port Cannot Be Accessed

Check:

```bash
ls -l /dev/ttyACM*
```

Then check your user's groups:

```bash
groups
```

On Raspberry Pi OS, serial port access is generally controlled by device permissions and membership in groups such as `dialout`.

Review the permission configuration before making changes.

> ⚠️ **Tip**
>
> Avoid routinely launching Iceman with `sudo`.
>
> It is better to fix permission issues properly.

### ❌ 11.4 — The Raspberry Pi Runs Out of Memory

Check:

```bash
free -h
```

Then:

```bash
df -h
```

During compilation, reduce the number of simultaneous jobs.

For example:

```bash
make -j1
```

Compilation will be slower but may require less memory.

### ❌ 11.5 — Compilation Fails

Check:

1. Required dependencies are installed.
2. The hardware configuration is correct.
3. There is enough disk space.
4. Enough memory is available.
5. The repository has no unexpected modifications.

Display the repository status:

```bash
git status
```

Check the commit:

```bash
git rev-parse HEAD
```

> 💡 **Tip**
>
> Keep the complete error output.
>
> The last few lines of a failed compilation do not always reveal the original cause.

### ❌ 11.6 — The Raspberry Pi No Longer Responds via SSH

Check:

- Power supply.
- Wi-Fi connection.
- IP address.
- The `proxmark.local` hostname.
- SSH service status.

If you have local access:

```bash
systemctl status ssh
```

Also check the network connection:

```bash
nmcli device status
```

### ❌ 11.7 — A tmux Session Has Disappeared

List existing sessions:

```bash
tmux ls
```

If no session exists, create one:

```bash
pm
```

Then restart Iceman:

```bash
pm3
```

> 💡 **Reminder**
>
> tmux does not preserve sessions after a reboot or power loss.

---

## 12. Maintenance Best Practices

To keep your installation stable, we recommend the following practices.

### 🔒 12.1 — Keep a Working Version

Before an important update:

- Record the Git commit.
- Back up `Makefile.platform`.
- Save configuration files.
- Create a microSD card image.
- Keep a copy of the previous build.

### 📋 12.2 — Change One Component at a Time

Avoid updating all of these simultaneously:

- Raspberry Pi OS.
- Iceman client.
- Proxmark3 firmware.

Working step by step makes it easier to identify the cause of a problem.

### ⚡ 12.3 — Check the Power Supply

A stable power supply is especially important during:

- Compilation.
- System updates.
- Firmware flashing.

### 🔄 12.4 — Do Not Update Without a Reason

A newer version is not automatically better suited to your hardware.

Before updating, check:

- Bug fixes.
- New features.
- Compatibility changes.
- Possible hardware limitations.

### 📚 12.5 — Keep a Maintenance Log

You can create a log file:

```bash
nano ~/maintenance.log
```

Example:

```text
Date: 2026-11-01

Raspberry Pi:
- Raspberry Pi OS Lite 64-bit
- System updated

Proxmark3:
- Generic Easy
- Platform: PM3GENERIC
- LED_ORDER: PM3EASY

Iceman:
- Git commit: to be completed
- Client tested: yes
- Firmware tested: yes

Notes:
- Installation working correctly
```

This file will help you track changes to your installation.

---

## 13. Maintenance Checklist

Before considering maintenance complete:

- [ ] The Raspberry Pi boots correctly.
- [ ] Wi-Fi connectivity works.
- [ ] SSH access works.
- [ ] Sufficient disk space is available.
- [ ] The power supply is stable.
- [ ] The Proxmark3 is detected over USB.
- [ ] The Iceman client starts.
- [ ] Client and firmware versions are compatible.
- [ ] `hw version` works.
- [ ] `hw status` works.
- [ ] An authorized RFID/NFC test works.
- [ ] The `pm` and `pm3` commands work.
- [ ] The build configuration is backed up.
- [ ] The Git commit is recorded.
- [ ] A recent backup exists on another device.
- [ ] Changes have been documented.

---

## ✅ Step 6 Complete — Installation Backed Up and Maintainable

We now know how to maintain our **Proxmark3 Easy + Raspberry Pi Zero 2 W** installation.

We have learned to:

- Check the Raspberry Pi's status.
- Identify installed versions.
- Back up important settings.
- Create a complete microSD card image.
- Update Raspberry Pi OS.
- Prepare a new Iceman build.
- Understand flashing risks.
- Preserve the bootloader.
- Restore a previous version.
- Troubleshoot common problems.

Our Proxmark3 Easy can now be operated and maintained more reliably.

### 🎯 Going Further

The project can evolve with additional features:

- Improved battery life.
- External battery operation.
- A 3D-printed enclosure.
- A dedicated Wi-Fi access point.
- Automatic Iceman startup.
- Improved SSH security.
- Automated diagnostic scripts.

These improvements may be covered in future guides.

---

## 📚 References

- [Proxmark3 Iceman — RfidResearchGroup](https://github.com/RfidResearchGroup/proxmark3)
- [Raspberry Pi — Official Documentation](https://www.raspberrypi.com/documentation/)
- [NetworkManager — Documentation](https://networkmanager.dev/docs/)
- [GNU tmux — Official Repository](https://github.com/tmux/tmux)
- [Step 2 — Raspberry Pi Installation](02-raspberry-installation.md)
- [Step 3 — Proxmark3 Iceman Installation](03-proxmark3-installation.md)
- [Step 4 — Daily Usage](04-daily-usage.md)

---

## 🧭 Navigation

| ⬅️ Previous Step | 🏠 Table of Contents | 🌐 Français |
|---|---|---|
| [Step 5 — Mobile Connection](05-mobile-connection.md) | [Back to Home](../../README.md) | [Étape 6 — Maintenance, sauvegardes et mises à jour](../fr/06-maintenance.md) |

---

📡 **Project: Proxmark3 Easy — Raspberry Pi Zero 2 W Wireless**y — Raspberry Pi Zero 2 W Wireless**
