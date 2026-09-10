# Ubuntu Server VM — Host-Only SSH Setup (VirtualBox)

Reliable host↔VM SSH access that works regardless of hotel/public WiFi.
The VM keeps **NAT** for internet and gets a second **Host-Only** adapter
for a private connection to Windows.

---

## Step 1 — Shut the VM down cleanly

Inside the VM:

```
sudo poweroff
```

---

## Step 2 — Create the Host-Only network (VirtualBox 7+)

1. **File → Tools → Network Manager → Host-only Networks tab → Create**
2. Note the adapter name (e.g. `VirtualBox Host-Only Ethernet Adapter`)
   and its range — default host IP is `192.168.56.1`, DHCP hands out
   `192.168.56.101+`.
3. Make sure **Enable DHCP Server** is ticked for that network.

---

## Step 3 — Attach the adapters to the VM

Select the VM → **Settings → Network**:

- **Adapter 1**: keep **NAT** (gives the VM internet)
- **Adapter 2**: tick *Enable Network Adapter* →
  **Attached to: Host-only Adapter** →
  **Name:** the adapter from Step 2 → **OK**

---

## Step 4 — Start the VM and log in

Start the VM. Log in at the console with your Ubuntu username and password.

---

## Step 5 — Identify the two interfaces

```
ip a
```

You'll see two ethernet interfaces:

- `enp0s3` — NAT, already has `inet 10.0.2.15` (internet)
- `enp0s8` — Host-only, likely **no `inet`** yet

If your names differ, use whatever the second (host-only) interface is
called in the steps below.

---

## Step 6 — Configure both interfaces with netplan

Open the netplan file (name may vary — `ls /etc/netplan/` to check):

```
sudo nano /etc/netplan/network.yaml
```

Set its contents to (**use spaces, never tabs** — YAML rejects tabs):

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: true
```

Save (Ctrl+O, Enter) and exit (Ctrl+X).

Fix permissions and apply:

```
sudo chmod 600 /etc/netplan/network.yaml
sudo netplan apply
```

---

## Step 7 — Get the host-only IP

```
ip a
```

Look at `enp0s8` for `inet 192.168.56.x`. **Write it down** — this is your
`<VM_IP>` (e.g. `192.168.56.101`).

If `enp0s8` still has no IP, confirm DHCP was enabled in Step 2, or set a
static address instead (see *Troubleshooting* below).

---

## Step 8 — Confirm internet works (needed to install SSH)

```
ping -c3 8.8.8.8
```

- **Replies** → good, continue.
- **No reply** → NAT isn't working. Check Adapter 1 is NAT and that
  `enp0s3` shows `inet 10.0.2.15` in `ip a`.

---

## Step 9 — Install the SSH server

The `ssh` service does not exist until `openssh-server` is installed:

```
sudo apt update
sudo apt install -y openssh-server
```

---

## Step 10 — Enable and start SSH

```
sudo systemctl enable --now ssh
sudo systemctl status ssh
```

You should see **active (running)**. Press `q` to exit the status view.

> On some minimal installs the unit is `sshd` instead of `ssh`. If `ssh`
> isn't found, use `sudo systemctl enable --now sshd` instead.

---

## Step 11 — Enable the firewall and allow SSH

```
sudo ufw allow OpenSSH
sudo ufw enable
sudo ufw status
```

`ufw status` should list **OpenSSH  ALLOW  Anywhere**.

> Do the `allow OpenSSH` **before** `ufw enable` so you don't lock SSH out.

---

## Step 12 — Connect from Windows

Open **PowerShell** or **Command Prompt** and connect using your Ubuntu
username and the host-only IP from Step 7:

```
ssh <ubuntu_username>@<VM_IP>
```

Example — user `luqman`, IP `192.168.56.101`:

```
ssh luqman@192.168.56.101
```

- First connection asks to confirm the fingerprint — type `yes`.
- Enter your Ubuntu password.
- Port stays the default `22` (no `-p 2222` — that's only for the NAT
  port-forwarding fallback).

You're in.

---

## Troubleshooting

**`ssh` not recognised on Windows**
Settings → Apps → Optional features → Add → **OpenSSH Client**.
Or use **PuTTY** with host `<VM_IP>`, port `22`.

**Connection refused / hangs — ping first:**

```
ping <VM_IP>
```

- Ping fails → host-only adapter isn't up. Recheck Step 6 and re-run
  `sudo netplan apply`.
- Ping works but SSH refused → SSH service isn't running (Step 10) or the
  firewall is blocking it (Step 11).

**`enp0s8` never gets a DHCP address — set a static IP instead**
Edit netplan so the host-only interface is static:

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: false
      addresses:
        - 192.168.56.50/24
```

```
sudo netplan apply
```

Then connect to `192.168.56.50` from Windows. Make sure the address is
inside the host-only range and not already used.