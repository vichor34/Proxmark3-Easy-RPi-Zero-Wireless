# 📱 Step 5 — Using Proxmark3 from an iPhone, iPad, or Android Device

🌐 **Language:** [🇫🇷 Français](../fr/05-connexion-mobile.md) | 🇬🇧 English

[🏠 Back to Table of Contents](../../README.md)

---

## 📖 Introduction

In the previous chapters, we installed and configured our **Proxmark3 Easy** on a **Raspberry Pi Zero 2 W**.

We can now operate the Proxmark3 from a computer using SSH, tmux, and the Iceman client.

In this chapter, we will learn how to control this setup from a **smartphone or tablet**, without a USB connection between the mobile device and the Proxmark3.

This solution works with:

- iPhones running iOS.
- iPads running iPadOS.
- Android smartphones.
- Android tablets.

The principle is the same on every platform:

**The phone connects to the Raspberry Pi over Wi-Fi, then uses SSH to access the Iceman client.**

> 💡 **Important**
>
> The Proxmark3 Easy remains physically connected to the Raspberry Pi via USB.
>
> The smartphone does not communicate directly with the Proxmark3.
>
> It remotely controls the Iceman software running on the Raspberry Pi.

### 🎯 Objectives

By the end of this chapter, you will be able to:

- Understand how the mobile connection works.
- Install an SSH client on iPhone, iPad, or Android.
- Save a connection to the Raspberry Pi.
- Use the Proxmark3 from a mobile terminal.
- Recover an Iceman session using tmux.
- Use special keys on a touchscreen keyboard.
- Connect the Raspberry Pi to a mobile hotspot.
- Understand the limitations of different connection methods.
- Troubleshoot common connection problems.

---

## 1. Our Setup Architecture

Our system consists of three components:

1. A smartphone or tablet.
2. A Raspberry Pi Zero 2 W.
3. A Proxmark3 Easy.

### 🏠 1.1 — Using a Home Wi-Fi Network

In this configuration, the smartphone and Raspberry Pi are connected to the same local network.

```text
       ┌──────────────────────┐
       │  iPhone / Android    │
       │                      │
       │  SSH Application     │
       └──────────┬───────────┘
                  │
                  │ Wi-Fi
                  │
       ┌──────────▼───────────┐
       │  Wi-Fi Router        │
       └──────────┬───────────┘
                  │
                  │ 2.4 GHz Wi-Fi
                  │
       ┌──────────▼───────────┐
       │ Raspberry Pi Zero 2 W│
       │                      │
       │ SSH + tmux + Iceman  │
       └──────────┬───────────┘
                  │
                  │ USB
                  │
       ┌──────────▼───────────┐
       │    Proxmark3 Easy    │
       └──────────────────────┘
```

The smartphone sends SSH commands to the Raspberry Pi.

The Iceman client then communicates with the Proxmark3 over USB.

### 📡 1.2 — Using a Mobile Hotspot

It is also possible to use a smartphone as a Wi-Fi hotspot.

In this case, the Raspberry Pi connects to the Wi-Fi network created by the phone.

```text
       ┌──────────────────────┐
       │  iPhone / Android    │
       │                      │
       │  Mobile Hotspot      │
       │  + SSH Application   │
       └──────────┬───────────┘
                  │
                  │ Wi-Fi
                  │
       ┌──────────▼───────────┐
       │ Raspberry Pi Zero 2 W│
       │                      │
       │ SSH + tmux + Iceman  │
       └──────────┬───────────┘
                  │
                  │ USB
                  │
       ┌──────────▼───────────┐
       │    Proxmark3 Easy    │
       └──────────────────────┘
```

> ⚠️ **Warning**
>
> Operation depends on the smartphone's hotspot capabilities.
>
> Some devices or networks may restrict communication between connected devices.
>
> Always test this connection method before relying on it while traveling.

---

## 2. Prerequisites

Before starting, make sure that:

- The Raspberry Pi boots correctly.
- The Proxmark3 Easy is connected to the Raspberry Pi.
- The SSH service is enabled.
- The Raspberry Pi is connected to a Wi-Fi network.
- The Iceman client works.
- The `pm` and `pm3` commands are configured.

These operations are covered in the previous chapters.

### 📚 Required Chapters

- [Step 2 — Raspberry Pi Installation](02-raspberry-installation.md)
- [Step 3 — Proxmark3 Iceman Installation](03-proxmark3-installation.md)
- [Step 4 — Daily Usage](04-daily-usage.md)

### 🔍 2.1 — Check the Network Address

From an existing SSH connection on a computer, run:

```bash
hostname -I
```

This command displays the IP addresses assigned to the Raspberry Pi.

To identify the IPv4 address used by the Wi-Fi interface:

```bash
ip -4 addr show wlan0
```

Example:

```text
inet 192.168.1.100/24
```

In this example, the Raspberry Pi's IP address is:

```text
192.168.1.100
```

> 💡 **Note**
>
> This address is only an example.
>
> The actual address depends on your network and may change when the Raspberry Pi connects to another access point.

---

## 3. Connect from an iPhone or iPad

On iOS and iPadOS, you need an application capable of establishing an SSH connection.

We will use **Termius** as an example.

### 🍎 3.1 — Install an SSH Client

Open the App Store.

Search for:

```text
Termius
```

Install the application.

Termius allows you to:

- Establish SSH connections.
- Save connection details.
- Open a remote terminal.
- Access additional keys for Linux commands.

> 💡 **Note**
>
> Some Termius features require a paid subscription.
>
> A basic SSH connection can be established without a subscription, subject to the features available in the installed version.
>
> Other compatible SSH applications can also be used.

### 🔧 3.2 — Add the Raspberry Pi

Open Termius.

Create a new SSH host.

Enter the following information:

| Setting | Value |
|---|---|
| Name | Proxmark3 |
| Address | `proxmark.local` |
| Port | `22` |
| Username | `piuser` |
| Authentication | Password or SSH key |

Replace `piuser` with the username configured on your Raspberry Pi.

If `proxmark.local` does not work, use the Raspberry Pi's actual IP address.

### 🔐 3.3 — Establish the Connection

Select the saved host.

The application establishes an SSH connection to the Raspberry Pi.

On the first connection, verify the server's SSH host key fingerprint before accepting it.

Then enter your password if you use password authentication.

Once connected, you should see a prompt similar to:

```text
piuser@proxmark:~ $
```

### 📡 3.4 — Launch the Proxmark3

In the terminal:

```bash
pm
```

Then, if Iceman is not already running:

```bash
pm3
```

The Iceman client should start.

You can now run a diagnostic command:

```text
hw version
```

### 📱 3.5 — Use Special Keys

Some commands require keys that are not directly available on a touchscreen keyboard.

These include:

- `Ctrl`
- `Esc`
- `Tab`
- Arrow keys

In Termius, use the terminal's additional key bar when available.

To detach a tmux session:

1. Activate the `Ctrl` key.
2. Press `B`.
3. Release the key combination.
4. Press `D`.

The tmux session remains active on the Raspberry Pi.

> 💡 **Tip**
>
> If you regularly use the Proxmark3 from an iPhone, become familiar with your SSH application's special keys.
>
> Their layout may vary depending on the application version.

---

## 4. Connect from Android

The procedure is similar to the one used on iPhone.

We can also use Termius on Android.

### 🤖 4.1 — Install an SSH Client

Open the Google Play Store.

Search for:

```text
Termius
```

Install the application.

You can also use another Android-compatible SSH client.

### 🔧 4.2 — Add the Raspberry Pi

Create a new SSH host.

Enter:

| Setting | Value |
|---|---|
| Name | Proxmark3 |
| Address | `proxmark.local` |
| Port | `22` |
| Username | `piuser` |
| Authentication | Password or SSH key |

If the `.local` hostname does not work, use the Raspberry Pi's IP address.

### 🔐 4.3 — Connect

Open the saved SSH connection.

Verify the server's identity on the first connection.

Enter the password if required.

Once connected, the Raspberry Pi terminal appears.

### 📡 4.4 — Launch Iceman

Run:

```bash
pm
```

Then:

```bash
pm3
```

You can now use Iceman commands.

For example:

```text
hw status
```

### ⌨️ 4.5 — Use the Android Keyboard

Android SSH applications generally provide tools for accessing special keys.

To detach a tmux session:

```text
Ctrl + B
```

Then:

```text
D
```

The exact procedure depends on the SSH client.

---

## 5. Recover a Session from Another Device

One of the main advantages of our configuration is tmux.

An Iceman session can remain active even when the smartphone disconnects.

### 🔄 5.1 — Example

Imagine you are using the Proxmark3 from your computer.

You started:

```bash
pm
```

Then:

```bash
pm3
```

You now want to continue working from your smartphone.

### 🔌 5.2 — Detach the Session on the Computer

Inside tmux:

```text
Ctrl + B
```

Then:

```text
D
```

You can close the SSH connection.

### 📱 5.3 — Resume the Session on the Smartphone

Open your SSH application.

Connect to the Raspberry Pi.

Run:

```bash
pm
```

You return to the existing session.

If Iceman is still running, its interface reappears.

> 💡 **Important**
>
> The Raspberry Pi must remain powered on.
>
> The Proxmark3 must remain connected.
>
> tmux does not preserve sessions after a Raspberry Pi reboot.

### ⚠️ 5.4 — Avoid Concurrent Connections

Avoid controlling the same session from multiple devices simultaneously.

When switching devices, detach the session on the first device before resuming it on the second.

---

## 6. Use a Smartphone's Mobile Hotspot

So far, we have used a home Wi-Fi network.

However, our goal is also to use the Proxmark3 while traveling.

For this, we can connect the Raspberry Pi to a smartphone's Wi-Fi hotspot.

### 📶 6.1 — How It Works

The smartphone creates a Wi-Fi network.

The Raspberry Pi connects to this network.

The smartphone's SSH application then connects to the Raspberry Pi using its address on that network.

This solution does not require an external router.

> ⚠️ **Limitation**
>
> The Raspberry Pi Zero 2 W supports 2.4 GHz Wi-Fi.
>
> The smartphone hotspot must therefore provide a connection compatible with this frequency band.

---

## 7. Configure an iPhone Hotspot

### 🍎 7.1 — Enable Personal Hotspot

On the iPhone:

1. Open **Settings**.
2. Go to **Personal Hotspot**.
3. Enable **Allow Others to Join**.
4. Check the network name and password.

If available, enable:

**Maximize Compatibility**

This option improves compatibility with devices using 2.4 GHz Wi-Fi.

### 🔧 7.2 — Save the Network on the Raspberry Pi

For this initial configuration, use a computer that already has SSH access to the Raspberry Pi.

On the Raspberry Pi, run:

```bash
sudo nmcli device wifi connect "IPHONE_NETWORK_NAME" --ask
```

Replace `IPHONE_NETWORK_NAME` with the network name created by your iPhone.

NetworkManager will request the password if necessary.

> ⚠️ **Warning**
>
> Your current SSH connection may be interrupted when the Raspberry Pi switches networks.
>
> Perform this operation only when you have a way to find the Raspberry Pi's address on the new network.

### 🔍 7.3 — Verify the Connection

Once the Raspberry Pi is connected to the hotspot, find its IP address.

You can use:

```bash
hostname -I
```

if you still have terminal access.

Otherwise, check the information available on the phone or use another authorized network discovery method.

### 📱 7.4 — Connect from the iPhone

In Termius:

1. Open the saved connection.
2. Use `proxmark.local` if the hostname works.
3. Otherwise, enter the Raspberry Pi's IP address on the hotspot network.
4. Establish the SSH connection.

Then:

```bash
pm
```

And, if necessary:

```bash
pm3
```

> 💡 **Note**
>
> SSH access from the same iPhone that provides the hotspot depends on iOS networking behavior.
>
> Test this configuration before relying on it away from home.
>
> If local communication is not possible, use a compatible external Wi-Fi access point or another network configuration.

---

## 8. Configure an Android Hotspot

### 🤖 8.1 — Enable the Wi-Fi Hotspot

On Android, open the network settings.

Depending on the manufacturer, the option may be called:

- Mobile Hotspot.
- Tethering.
- Wi-Fi Hotspot.

Enable the hotspot.

### 📶 8.2 — Check the Wi-Fi Band

If your phone allows you to select the Wi-Fi band, choose:

```text
2.4 GHz
```

This band is compatible with the Raspberry Pi Zero 2 W.

### 🔧 8.3 — Save the Network

On the Raspberry Pi:

```bash
sudo nmcli device wifi connect "ANDROID_NETWORK_NAME" --ask
```

Replace the network name with your Android hotspot's name.

Enter the requested password.

### 🔍 8.4 — Find the IP Address

Once the Raspberry Pi is connected, find its IP address.

Some Android smartphones display connected devices and their addresses.

You can also check the address from the Raspberry Pi when terminal access is available.

### 📡 8.5 — Use the Proxmark3

From your Android SSH application:

1. Connect to the Raspberry Pi's address.
2. Open the tmux session.
3. Launch Iceman if necessary.

Commands:

```bash
pm
```

Then:

```bash
pm3
```

> ⚠️ **Note**
>
> SSH access from the same phone hosting the hotspot may behave differently depending on the Android version and manufacturer.
>
> Test this feature on your device.

---

## 9. Switch Between Home Wi-Fi and a Mobile Hotspot

NetworkManager can save multiple Wi-Fi networks.

We already introduced this feature in Step 2.

### 📋 9.1 — List Saved Connections

On the Raspberry Pi:

```bash
nmcli connection show
```

This command displays known network connections.

### 📶 9.2 — Display the Active Network

```bash
nmcli device status
```

The Wi-Fi interface is generally:

```text
wlan0
```

### 🔄 9.3 — Automatic Connection

When several saved networks are available, NetworkManager can select a connection based on availability and priority.

You can therefore prepare the Raspberry Pi for different environments:

- Home Wi-Fi.
- Workshop Wi-Fi.
- Smartphone hotspot.

> 💡 **Tip**
>
> Save and test your networks before traveling.
>
> This avoids having to change network settings when SSH access is no longer available.

### ⚠️ 9.4 — IP Address Changes

The Raspberry Pi's IP address may change when it joins another network.

For example:

```text
Home network: 192.168.1.100
```

```text
Mobile hotspot: different address
```

These values are examples only.

If `proxmark.local` does not work, use the IP address assigned on the currently active network.

---

## 10. Mobile Connection Security

The Raspberry Pi provides access to Linux and the Proxmark3.

It is therefore important to secure these connections.

### 🔐 10.1 — Use a Strong Password

The Raspberry Pi user account should be protected by a sufficiently long and difficult-to-guess password.

Avoid reusing a password from another service.

### 🔑 10.2 — Prefer SSH Keys

For regular use, SSH key authentication is recommended.

It avoids repeatedly entering a password.

The exact configuration depends on the SSH application.

> 💡 **Tip**
>
> Do not disable password authentication until you have verified that SSH key authentication works correctly.

### 🌐 10.3 — Do Not Expose SSH Directly to the Internet

Our tutorial uses local network connections.

There is no need to expose the Raspberry Pi's SSH port to the Internet.

Avoid forwarding port `22` on your router.

For remote access over the Internet, use an appropriately configured secure solution, such as a VPN.

### 📱 10.4 — Protect Your Smartphone

Protect your phone with:

- A screen lock code.
- Biometric authentication, if available.
- Security updates.
- Controlled access to SSH credentials.

Never share your private SSH keys.

---

## 11. Troubleshoot Common Problems

### ❌ 11.1 — The Smartphone Cannot Find the Raspberry Pi

Check:

1. The Raspberry Pi is powered on.
2. It is connected to the correct Wi-Fi network.
3. The smartphone can communicate with other devices on that network.
4. The IP address is correct.

Try the IP address instead of:

```text
proxmark.local
```

### ❌ 11.2 — The Mobile Hotspot Does Not Work

Check:

- The hotspot is enabled.
- The network uses a compatible Wi-Fi band.
- The password is correct.
- The Raspberry Pi is actually connected.
- Local communication between devices is allowed.

### ❌ 11.3 — SSH Connection Refused

Check that the SSH service is running on the Raspberry Pi.

From local access or another working connection:

```bash
systemctl status ssh
```

### ❌ 11.4 — The `pm` Command Does Not Work

Check the alias:

```bash
alias pm
```

If necessary, run:

```bash
tmux new-session -A -s proxmark
```

### ❌ 11.5 — The `pm3` Command Does Not Work

Check:

```bash
command -v pm3
```

If necessary, refer to Step 3 to verify the Iceman client installation.

### ❌ 11.6 — The Touchscreen Keyboard Cannot Control tmux

Make sure your SSH application provides a `Ctrl` key.

To detach the session:

```text
Ctrl + B
```

Then:

```text
D
```

If your application cannot easily send these keys, try another SSH client.

### ❌ 11.7 — The Connection Drops When the Phone Locks

Some mobile operating systems suspend network applications when they move into the background.

This may interrupt the SSH connection.

If Iceman was started inside a tmux session, it may continue running on the Raspberry Pi.

Reconnect and run:

```bash
pm
```

You will recover the session if it is still active.

---

## 12. Simplified Mobile Procedure

Here is the procedure to remember.

### 📱 Startup

**1. Power on the Raspberry Pi and Proxmark3.**

**2. Connect the smartphone to a network that can reach the Raspberry Pi.**

**3. Open the SSH application.**

**4. Connect to the Raspberry Pi.**

**5. Open the persistent session:**

```bash
pm
```

**6. Launch Iceman if necessary:**

```bash
pm3
```

### 🔍 Initial Diagnostics

Inside Iceman:

```text
hw version
```

Then:

```text
hw status
```

### 🔌 Temporary Disconnection

Detach the tmux session:

```text
Ctrl + B
```

Then:

```text
D
```

You can then close the SSH application.

### 🔄 Reconnection

Open your SSH application again.

Connect to the Raspberry Pi.

Run:

```bash
pm
```

You return to your session.

---

## 13. Final Checklist

Before considering the mobile configuration complete:

- [ ] The Raspberry Pi works correctly.
- [ ] The Proxmark3 Easy is connected and recognized.
- [ ] An SSH application is installed on the smartphone.
- [ ] SSH access to the Raspberry Pi works.
- [ ] The `pm` command opens a tmux session.
- [ ] The `pm3` command launches Iceman.
- [ ] Diagnostic commands work.
- [ ] Special keys are accessible on the mobile keyboard.
- [ ] A tmux session can be detached.
- [ ] The session can be recovered after reconnecting.
- [ ] The mobile hotspot has been tested if you plan to use it.
- [ ] You know how to find the Raspberry Pi's IP address.
- [ ] SSH access is secured.

---

## ✅ Step 5 Complete — Proxmark3 Accessible from a Smartphone

Our installation can now be used from:

- A Windows computer.
- A macOS computer.
- A Linux computer.
- An iPhone.
- An iPad.
- An Android smartphone.
- An Android tablet.

Thanks to the Raspberry Pi Zero 2 W, the Proxmark3 Easy can be controlled remotely over Wi-Fi.

tmux sessions provide a persistent working environment and make it possible to switch between devices.

### 🚀 In the Next Step

We will cover installation maintenance:

- Backing up important files.
- Updating Raspberry Pi OS.
- Updating the Iceman client.
- Preparing a firmware update.
- Checking versions and compatibility.
- Diagnosing advanced problems.
- Restoring a previous installation when necessary.

---

## 📚 References

- [Raspberry Pi — Official Documentation](https://www.raspberrypi.com/documentation/)
- [Termius — Official Website](https://termius.com/)
- [Proxmark3 Iceman — RfidResearchGroup](https://github.com/RfidResearchGroup/proxmark3)
- [Step 2 — Raspberry Pi Installation](02-raspberry-installation.md)
- [Step 3 — Proxmark3 Iceman Installation](03-proxmark3-installation.md)
- [Step 4 — Daily Usage](04-daily-usage.md)

---

## 🧭 Navigation

| ⬅️ Previous Step | 🏠 Table of Contents | 🌐 Français |
|---|---|---|
| [Step 4 — Daily Usage](04-daily-usage.md) | [Back to Home](../../README.md) | [Étape 5 — Connexion mobile](../fr/05-connexion-mobile.md) |

> 💡 **Temporary navigation**
>
> Navigation links will be harmonized once all tutorial pages have been created.

---

📡 **Project: Proxmark3 Easy — Raspberry Pi Zero 2 W Wireless**
