# 📶 Step 7 — Automatic Wi-Fi Hotspot

🌐 **Language:** [🇫🇷 Français](../fr/07-hotspot-wifi.md) | 🇬🇧 English

[🏠 Back to the table of contents](../../README.md)

---

## 📖 Overview

The **Proxmark3 Easy + Raspberry Pi Zero 2 W** setup can already be accessed over Wi-Fi from a computer or smartphone. However, away from home, the Raspberry Pi may not find any saved Wi-Fi network.

This optional chapter adds an **automatic fallback Wi-Fi hotspot**:

- **Wi-Fi client mode:** connect to a saved network when one is available at boot.
- **Hotspot mode:** create a dedicated wireless network when no known network connects during startup.

This makes it possible to operate the Proxmark3 without a router or Internet connection.

> 💡 **Important**
>
> The hotspot is optional. If you always use an existing Wi-Fi network, you can keep the configuration from the previous chapters.

### 🎯 Goals

By the end of this chapter, you will be able to:

- Check whether your Wi-Fi adapter supports access-point mode.
- Set connection priorities for saved Wi-Fi networks.
- Create a password-protected hotspot with a fixed IP address.
- Configure a startup fallback script and a systemd service.
- Test both connection modes and access Iceman from a smartphone.
- Diagnose problems and disable the feature when needed.

---

## 1. How it works

### 📡 1.1 — Two Wi-Fi modes

The Raspberry Pi Zero 2 W has one built-in Wi-Fi interface. This setup uses **either client mode or hotspot mode**, not both simultaneously.

| Situation | Behavior |
|---|---|
| A known Wi-Fi network connects during startup | Keep the client connection |
| Several known networks are available | NetworkManager chooses according to its connection rules and priorities |
| No known network connects within 30 seconds | Start the hotspot |
| A known network reappears while the hotspot is running | Keep the hotspot in this configuration |
| The Raspberry Pi reboots | Make a new choice at startup |

### 🔄 1.2 — Startup flow

```text
             Raspberry Pi boots
                     |
                     v
            NetworkManager starts
                     |
                     v
          Saved Wi-Fi connections tried
                     |
                     v
             Wait up to 30 seconds
                     |
            Connected to known Wi-Fi?
                  /       \
                YES        NO
                 |          |
                 v          v
            Keep Wi-Fi   Start hotspot
                 |          |
                 v          v
               SSH      Direct SSH
                 |          |
                 +-----+----+
                       |
                       v
                      tmux
                       |
                       v
                 Proxmark3 Iceman
```

### ⚠️ 1.3 — Important limitation

The selection happens **at boot**. This version does not periodically search for known networks or automatically switch back from an active hotspot. To return to a known Wi-Fi network, make sure it is available and **reboot the Raspberry Pi**.

> 💡 **Why this design?**
>
> Avoiding background scans and repeated mode changes reduces the risk of disrupting an active SSH session. The priority is a simple, predictable setup.

---

## 2. Reference configuration

| Item | Configuration |
|---|---|
| Raspberry Pi | Zero 2 W |
| Operating system | Raspberry Pi OS Lite 64-bit |
| Debian base | Debian 13 (Trixie) |
| Network manager | NetworkManager |
| Wi-Fi interface | `wlan0` |
| Example primary network profile | `Home-WiFi` |
| Example secondary network profile | `Phone-Hotspot` |
| Hotspot SSID / profile | `Proxmark3-Wireless` |
| Hotspot IP address | `192.168.50.1/24` |
| Wireless security | WPA2-Personal |
| Connection wait | 30 seconds |
| Remote access | SSH |
| Persistent terminal | tmux |

### 📚 Prerequisites

Complete these chapters first:

- [Step 2 — Raspberry Pi setup](02-raspberry-installation.md)
- [Step 3 — Proxmark3 installation](03-proxmark3-installation.md)
- [Step 4 — Daily usage](04-daily-usage.md)

Your Raspberry Pi must boot, connect to Wi-Fi, accept SSH connections, run NetworkManager, and launch the Proxmark3 Iceman client.

> ⚠️ **Warning**
>
> Changing network settings can disconnect SSH. Work somewhere you can restart the Pi or access its microSD card. Prefer having local keyboard/display access for recovery.

---

## 3. Check NetworkManager

### 🔍 3.1 — Identify the network interface

Connect by SSH:

```bash
ssh piuser@proxmark.local
```

List devices:

```bash
nmcli device status
```

Example:

```text
DEVICE  TYPE      STATE      CONNECTION
wlan0   wifi      connected  Home-WiFi
lo      loopback  connected  lo
```

This guide uses `wlan0`. If your interface has another name, adapt all relevant commands.

### 📡 3.2 — Check access-point support

```bash
nmcli -f WIFI-PROPERTIES.AP device show wlan0
```

Expected:

```text
WIFI-PROPERTIES.AP: yes
```

> ⚠️ **Important**
>
> If the result is `no`, stop and check the Wi-Fi hardware and driver before continuing.

### 🔍 3.3 — List saved connections

```bash
nmcli connection show
```

Example:

```text
NAME           TYPE      DEVICE
Home-WiFi      wifi      wlan0
Phone-Hotspot  wifi      --
lo             loopback  lo
```

The script will check the **active NetworkManager connection profile name**, which is not necessarily the same as the SSID.

---

## 4. Configure known Wi-Fi networks

### 📶 4.1 — Connection priorities

NetworkManager can store several Wi-Fi profiles. Set the preferred network to a higher autoconnect priority:

| Network profile | Priority |
|---|---:|
| `Home-WiFi` | 100 |
| `Phone-Hotspot` | 50 |

**Replace these example names with your actual NetworkManager profile names.**

### 🔧 4.2 — Primary network

```bash
sudo nmcli connection modify "Home-WiFi" \
  connection.autoconnect yes \
  connection.autoconnect-priority 100
```

### 🔧 4.3 — Secondary network

```bash
sudo nmcli connection modify "Phone-Hotspot" \
  connection.autoconnect yes \
  connection.autoconnect-priority 50
```

If you have only one known network, configure only that one.

### 🔍 4.4 — Verify priorities

```bash
nmcli -f connection.id,connection.autoconnect,connection.autoconnect-priority \
  connection show "Home-WiFi"
```

Expected:

```text
connection.id:                   Home-WiFi
connection.autoconnect:          yes
connection.autoconnect-priority: 100
```

Repeat for any additional profiles.

> 💡 **Tip**
>
> A higher priority is preferred among eligible profiles; it does not guarantee a successful connection if the network is unavailable or authentication fails.

---

## 5. Create the Wi-Fi hotspot

We will create the hotspot profile **without activating it yet**.

### 📡 5.1 — Add the connection profile

```bash
sudo nmcli connection add \
  type wifi \
  ifname wlan0 \
  con-name "Proxmark3-Wireless" \
  ssid "Proxmark3-Wireless"
```

### 🔧 5.2 — Configure access-point mode

```bash
sudo nmcli connection modify "Proxmark3-Wireless" \
  802-11-wireless.mode ap \
  802-11-wireless.band bg \
  ipv4.method shared \
  ipv4.addresses 192.168.50.1/24 \
  ipv6.method disabled \
  connection.autoconnect no
```

| Setting | Purpose |
|---|---|
| `mode ap` | Wi-Fi access-point mode |
| `band bg` | 2.4 GHz band |
| `ipv4.method shared` | Provides IPv4 addressing to connected clients |
| `192.168.50.1/24` | Raspberry Pi address on the hotspot |
| `ipv6.method disabled` | Disables IPv6 on this profile |
| `autoconnect no` | Prevents NetworkManager from starting the hotspot on its own |

> 💡 **Why `autoconnect no`?**
>
> The hotspot must not compete with saved client networks at startup. Our fallback script will explicitly activate it when necessary.

### 🔐 5.3 — Set the Wi-Fi password

Choose WPA2-Personal:

```bash
sudo nmcli connection modify "Proxmark3-Wireless" \
  wifi-sec.key-mgmt wpa-psk
```

Enter a strong, unique password without typing it directly into a command stored in shell history:

```bash
read -rsp "Hotspot password: " HOTSPOT_PASSWORD
echo
```

Apply it:

```bash
sudo nmcli connection modify "Proxmark3-Wireless" \
  wifi-sec.psk "$HOTSPOT_PASSWORD"
```

Clear the temporary shell variable:

```bash
unset HOTSPOT_PASSWORD
```

> 🔐 **Security**
>
> Never commit real Wi-Fi passwords to GitHub. Protect the Raspberry Pi's SSH account as well: clients on the hotspot can reach its network services.

### 🔍 5.4 — Verify the hotspot profile

```bash
nmcli -f 802-11-wireless.ssid,802-11-wireless.mode,ipv4.method,ipv4.addresses,connection.autoconnect \
  connection show "Proxmark3-Wireless"
```

| Setting | Expected value |
|---|---|
| SSID | `Proxmark3-Wireless` |
| Mode | `ap` |
| IPv4 method | `shared` |
| Address | `192.168.50.1/24` |
| Autoconnect | `no` |

> ⚠️ **Warning**
>
> Do **not** run `nmcli connection up "Proxmark3-Wireless"` yet. Activating the hotspot can replace your current Wi-Fi connection and terminate SSH.

---

## 6. Create the automatic fallback script

The script waits up to 30 seconds for one of the specified saved connections. If none is active, it starts the hotspot.

### 🛠️ 6.1 — Create the script

```bash
sudo nano /usr/local/sbin/proxmark-wifi-fallback.sh
```

Paste the following **complete** script:

```bash
#!/bin/bash

set -euo pipefail

INTERFACE="wlan0"
HOTSPOT="Proxmark3-Wireless"
TIMEOUT=30

logger -t proxmark-wifi "Looking for a known Wi-Fi network..."

# Wait up to 30 seconds for a known connection.
for ((i=0; i<TIMEOUT; i++)); do

    ACTIVE=$(nmcli -g GENERAL.CONNECTION device show "$INTERFACE" 2>/dev/null || true)

    if [[ "$ACTIVE" == "Home-WiFi" || "$ACTIVE" == "Phone-Hotspot" ]]; then
        logger -t proxmark-wifi "Connected to: $ACTIVE"
        exit 0
    fi

    sleep 1
done

# Final check before activating the hotspot.
ACTIVE=$(nmcli -g GENERAL.CONNECTION device show "$INTERFACE" 2>/dev/null || true)

if [[ "$ACTIVE" == "Home-WiFi" || "$ACTIVE" == "Phone-Hotspot" ]]; then
    exit 0
fi

logger -t proxmark-wifi "No known network connected: starting hotspot."

# Prevent automatic reconnection attempts on this interface
# during the hotspot session.
nmcli device set "$INTERFACE" autoconnect no

# Start the hotspot.
if nmcli connection up "$HOTSPOT" ifname "$INTERFACE"; then
    logger -t proxmark-wifi "Hotspot active: $HOTSPOT"
else
    logger -t proxmark-wifi "ERROR: hotspot activation failed."
    exit 1
fi
```

### ⚠️ 6.2 — Adapt the known network names

The condition below appears **twice** in the script:

```bash
if [[ "$ACTIVE" == "Home-WiFi" || "$ACTIVE" == "Phone-Hotspot" ]]; then
```

Replace both example names with your own **NetworkManager connection profile names**.

With only one known network, use this condition in both places:

```bash
if [[ "$ACTIVE" == "Home-WiFi" ]]; then
```

> ⚠️ **Important**
>
> The script checks the name of the **active NetworkManager profile**, not the broadcast SSID. Use `nmcli connection show` to identify the correct names. This script is intentionally designed for the explicitly listed profiles.

### 💾 6.3 — Save the file

In nano: press `Ctrl + O`, then `Enter`, then `Ctrl + X`.

### 🔐 6.4 — Make it executable

```bash
sudo chmod 755 /usr/local/sbin/proxmark-wifi-fallback.sh
```

### 🔍 6.5 — Check Bash syntax

```bash
bash -n /usr/local/sbin/proxmark-wifi-fallback.sh
```

No output means no Bash syntax error was detected. It does **not** prove the hotspot will work; the runtime test is still necessary.

---

## 7. Create the systemd service

### 🛠️ 7.1 — Create the unit file

```bash
sudo nano /etc/systemd/system/proxmark-wifi-fallback.service
```

Paste:

```ini
[Unit]
Description=Proxmark3 Wi-Fi fallback hotspot
Wants=NetworkManager.service
After=NetworkManager.service

[Service]
Type=oneshot
ExecStartPre=/usr/bin/nmcli device set wlan0 autoconnect yes
ExecStart=/usr/local/sbin/proxmark-wifi-fallback.sh
TimeoutStartSec=60

[Install]
WantedBy=multi-user.target
```

### 📖 7.2 — Unit settings explained

| Setting | Purpose |
|---|---|
| `Wants=NetworkManager.service` | Requests NetworkManager at boot |
| `After=NetworkManager.service` | Runs after NetworkManager has started |
| `Type=oneshot` | Runs a one-time startup task |
| `ExecStartPre` | Re-enables device autoconnect before the check |
| `ExecStart` | Executes the fallback script |
| `TimeoutStartSec=60` | Limits service startup time |
| `WantedBy=multi-user.target` | Enables normal boot integration |

### 🔄 7.3 — Reload systemd

```bash
sudo systemctl daemon-reload
```

### ⚡ 7.4 — Enable the service

```bash
sudo systemctl enable proxmark-wifi-fallback.service
```

### 🔍 7.5 — Verify enablement

```bash
systemctl is-enabled proxmark-wifi-fallback.service
```

Expected:

```text
enabled
```

> ⚠️ **Warning**
>
> Do not immediately run `systemctl start` over SSH. If no known network is detected, the script may switch to the hotspot and disconnect you.

---

## 8. Test with a known Wi-Fi network

### 📡 8.1 — Check the current connection

```bash
nmcli device status
```

Confirm that `wlan0` is connected to your normal Wi-Fi network.

### 🔄 8.2 — Reboot

```bash
sudo reboot
```

Wait about a minute, then reconnect:

```bash
ssh piuser@proxmark.local
```

If mDNS is unavailable, use the Raspberry Pi's LAN IP address.

### 🔍 8.3 — Check the active connection

```bash
nmcli device status
```

Expected example:

```text
wlan0  wifi  connected  Home-WiFi
```

### 📋 8.4 — Inspect the service and logs

```bash
systemctl status proxmark-wifi-fallback.service --no-pager
```

```bash
journalctl -u proxmark-wifi-fallback.service -b --no-pager
```

Example log messages:

```text
Looking for a known Wi-Fi network...
Connected to: Home-WiFi
Finished proxmark-wifi-fallback.service
```

`inactive (dead)` is normal after a successful `Type=oneshot` service without `RemainAfterExit`.

---

## 9. Test without a known Wi-Fi network

This test checks whether the hotspot starts when no saved network is available.

### ⚠️ Precautions

You may temporarily lose SSH access. Before testing, verify the known-network test passed, keep the hotspot password handy, and arrange a way to reboot or access the Raspberry Pi locally if needed.

### 🔌 9.1 — Shut down the Raspberry Pi

```bash
sudo poweroff
```

Wait until shutdown has completed.

### 📡 9.2 — Make known networks unavailable

For example:

- Turn off your home Wi-Fi router.
- Disable your phone's personal hotspot.
- Ensure no other saved network is within range.

### ⚡ 9.3 — Boot the Raspberry Pi

Power the Raspberry Pi back on. Wait approximately **45–60 seconds**. If no known profile connected during the waiting period, the fallback script should activate the hotspot.

### 📱 9.4 — Join the hotspot

On your phone or computer, look for:

```text
Proxmark3-Wireless
```

Connect with the password you set earlier.

### 🔍 9.5 — Connect over SSH

| Setting | Value |
|---|---|
| Host | `192.168.50.1` |
| Port | `22` |
| Username | `piuser` |
| Authentication | SSH password or key |

From a terminal:

```bash
ssh piuser@192.168.50.1
```

> 💡 **Note**
>
> Your smartphone may warn that this Wi-Fi network has no Internet access. This is expected: the hotspot provides local connectivity to the Raspberry Pi.

### 📡 9.6 — Verify hotspot mode

```bash
nmcli device status
```

Expected:

```text
wlan0  wifi  connected  Proxmark3-Wireless
```

Check the interface address:

```bash
ip -4 addr show wlan0
```

Expected address:

```text
192.168.50.1/24
```

### 📋 9.7 — Check startup logs

```bash
journalctl -u proxmark-wifi-fallback.service -b --no-pager
```

Look for messages reporting hotspot activation.

> ⚠️ **Important**
>
> A successful test validates your specific hardware, driver, and NetworkManager version. Behavior can differ on other configurations.

---

## 10. Use Proxmark3 through the hotspot

The hotspot changes only **how you reach the Raspberry Pi**; Iceman commands remain the same.

### 📱 10.1 — Connect over SSH

From a phone joined to `Proxmark3-Wireless`:

```bash
ssh piuser@192.168.50.1
```

### 🖥️ 10.2 — Open the tmux session

```bash
pm
```

This creates or reattaches to the `proxmark` tmux session, as configured in the earlier chapters.

### 📡 10.3 — Start Iceman

If the client is not already running:

```bash
pm3
```

### 🔍 10.4 — Check the Proxmark3

In the Iceman client:

```text
hw version
```

Then:

```text
hw status
```

The device should respond normally.

### 🔌 10.5 — Detach without closing tmux

```text
Ctrl + B, then D
```

You can reconnect to the session over SSH while the Raspberry Pi remains powered on.

> 💡 **Remember**
>
> The hotspot is simply another route to the Raspberry Pi. It does not change the Proxmark3 firmware or Iceman workflow.

---

## 11. Return to normal Wi-Fi

### 🔄 11.1 — Reboot when your saved network is available

Turn your usual Wi-Fi network back on. While connected to the hotspot over SSH, run:

```bash
sudo reboot
```

NetworkManager will try saved connections again during startup. If a known profile connects, the fallback script leaves it in place.

### 🔍 11.2 — Confirm the connection

Reconnect through your LAN and run:

```bash
nmcli device status
```

The usual Wi-Fi profile should be active.

> ⚠️ **Important**
>
> Merely turning the router back on does **not** automatically switch an active hotspot back to client mode. This version makes the decision at boot.

---

## 12. Troubleshooting

### ❌ 12.1 — The hotspot does not appear

Check access-point support:

```bash
nmcli -f WIFI-PROPERTIES.AP device show wlan0
```

Check the saved hotspot profile:

```bash
nmcli connection show "Proxmark3-Wireless"
```

Inspect the fallback service log:

```bash
journalctl -u proxmark-wifi-fallback.service -b --no-pager
```

Inspect NetworkManager logs:

```bash
sudo journalctl -u NetworkManager -b --no-pager
```

### ❌ 12.2 — The service fails

```bash
systemctl status proxmark-wifi-fallback.service --no-pager
```

Check script syntax and permissions:

```bash
bash -n /usr/local/sbin/proxmark-wifi-fallback.sh
```

```bash
ls -l /usr/local/sbin/proxmark-wifi-fallback.sh
```

### ❌ 12.3 — The phone connects, but SSH fails

With local access, verify SSH:

```bash
systemctl status ssh
```

Verify the Wi-Fi IP address:

```bash
ip -4 addr show wlan0
```

Use `192.168.50.1` as the SSH host and confirm that the phone is still connected to `Proxmark3-Wireless`.

### ❌ 12.4 — The Raspberry Pi does not return to known Wi-Fi

List profiles:

```bash
nmcli connection show
```

Check a saved network's autoconnect settings:

```bash
nmcli -f connection.id,connection.autoconnect,connection.autoconnect-priority \
  connection show "Home-WiFi"
```

Check device autoconnect:

```bash
nmcli -f GENERAL.AUTOCONNECT device show wlan0
```

If needed, from a working network or local console:

```bash
sudo nmcli device set wlan0 autoconnect yes
```

Then reboot while your normal network is available.

### ❌ 12.5 — The hotspot starts too soon

The script uses:

```bash
TIMEOUT=30
```

If your normal network needs more time to become available, increase it, for example:

```bash
TIMEOUT=60
```

Save the script. The change applies the next time the service runs. A longer timeout also delays hotspot availability when no known network exists.

---

## 13. Disable automatic hotspot fallback

The feature can be disabled without removing your normal Wi-Fi profiles.

### 🛑 13.1 — Disable the startup service

```bash
sudo systemctl disable proxmark-wifi-fallback.service
```

### 🔄 13.2 — Re-enable device autoconnect

```bash
sudo nmcli device set wlan0 autoconnect yes
```

Also ensure that saved Wi-Fi profiles have `connection.autoconnect: yes`.

### 🗑️ 13.3 — Optionally remove the hotspot profile

```bash
sudo nmcli connection delete "Proxmark3-Wireless"
```

> ⚠️ **Warning**
>
> Removing an active Wi-Fi connection can immediately disconnect SSH. Perform this step from another working network or a local console.

### 🔄 13.4 — Re-enable fallback later

If the script, unit, and hotspot profile still exist:

```bash
sudo systemctl enable proxmark-wifi-fallback.service
```

It will run again on the next boot.

---

## 14. Final checklist

- [ ] NetworkManager is running.
- [ ] `wlan0` supports access-point mode.
- [ ] Known Wi-Fi profiles are saved.
- [ ] Connection priorities are configured.
- [ ] The `Proxmark3-Wireless` profile exists.
- [ ] The hotspot address is `192.168.50.1/24`.
- [ ] A strong hotspot password is configured.
- [ ] The fallback script is executable.
- [ ] The systemd service is enabled.
- [ ] A known network reconnects after reboot.
- [ ] The hotspot appears when no known network is available.
- [ ] A smartphone can join the hotspot.
- [ ] SSH works through the hotspot.
- [ ] The `pm` command works.
- [ ] The `pm3` client starts.
- [ ] A reboot restores normal Wi-Fi when a known network is available.

---

## ✅ Step 7 complete — Standalone Proxmark3 Wireless

The **Proxmark3 Easy + Raspberry Pi Zero 2 W** can now be used on an existing Wi-Fi network or through its own hotspot when no saved network connects at startup.

You have configured NetworkManager priorities, a secure hotspot, a fixed local IP address, a startup fallback script, and a systemd service. You have also tested SSH, tmux, and Iceman without an external network.

### 🎯 Possible future improvements

- Periodic detection of known networks.
- Automatic switching back from hotspot to client mode.
- A local web interface.
- A Wi-Fi setup assistant.
- Automated network recovery.

These enhancements are **not required** for the current version.

---

## 📚 References

- [NetworkManager — Official documentation](https://networkmanager.dev/docs/)
- [Raspberry Pi — Official documentation](https://www.raspberrypi.com/documentation/)
- [systemd — Service unit documentation](https://www.freedesktop.org/software/systemd/man/latest/systemd.service.html)
- [Step 2 — Raspberry Pi setup](02-raspberry-installation.md)
- [Step 4 — Daily usage](04-daily-usage.md)
- [Step 5 — Mobile connection](05-mobile-connection.md)
- [Step 6 — Maintenance, backups and updates](06-maintenance.md)

---

## 🧭 Navigation

| ⬅️ Previous step | 🏠 Contents | 🌐 Français |
|---|---|---|
| [Step 6 — Maintenance, backups and updates](06-maintenance.md) | [Back to home](../../README.md) | [Étape 7 — Hotspot Wi-Fi automatique](../fr/07-hotspot-wifi.md) |

---

📡 **Project: Proxmark3 Easy — Raspberry Pi Zero 2 W Wireless**
