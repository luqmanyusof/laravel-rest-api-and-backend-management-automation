# Day 3 — SFTP, Cron & Scheduled File Transfers

> *Part of **Practical Laravel Backend Integration & Automation** — REST & SOAP APIs, SFTP file
> transfers, and cron-scheduled jobs (Day 3 of 3).*

Today you move files to a real Linux server over **SFTP** and learn to **schedule work with
cron**. First you learn *what* SFTP (SSH File Transfer Protocol) is, then you **build an Ubuntu
24.04 Server VM** (from the files you downloaded on Day 2), turn on its SFTP server, and transfer
files two ways: with a **GUI client** (WinSCP and FileZilla, in depth) and from the **terminal**
(the `sftp` command, with plenty of examples). Finally you **schedule a task with cron** (the
Linux job scheduler) so routine work runs on its own.

**Everything today runs on the VM and standard Windows SFTP tools — no Laravel involved.** Day 1
and Day 2 stay on Laragon; Day 3 is a self-contained Linux/SFTP module.

**Stack:** Windows host (WinSCP, FileZilla, the built-in `ssh`/`sftp` client) + an Ubuntu Server
VM (VirtualBox, OpenSSH), cron.

**What you build today**
- Understand **SSH (Secure Shell) / SFTP vs FTP (File Transfer Protocol)** (the concept) before
  touching anything
- **VirtualBox installed + an Ubuntu 24.04 Server VM** built from Day 2's downloads
- **Linux command-line essentials** — navigating, editing files, packages, users and permissions
- An **SFTP server** with a dedicated user and a drop folder
- **GUI transfers** with WinSCP and FileZilla — using each tool's real features, not just drag-drop
- **Terminal transfers** with the `sftp` command — a working repertoire of everyday commands
- A **cron schedule** that runs a routine task automatically

**What is NOT in scope today:** Laravel / PHP (Hypertext Preprocessor) (Days 1–2 — Laravel runs
on Laragon and isn't part of today's VM work), SSH key-based authentication and Bash scripting
(useful next steps, but beyond today), production SSH hardening, cloud SFTP providers.

**How this day builds (prerequisites first, easy first):**
1. **Concept:** what SSH/SFTP is and why (no setup — just understand) → 2. **Build** the Ubuntu VM
(the big prerequisite) → 3. **Linux admin essentials** (find your feet on the command line) → 4.
Turn on the **SFTP server** + make a user → 5. Transfer files with a **GUI** (WinSCP, then FileZilla
— in depth) → 6. Transfer from the **terminal** (many `sftp` examples) → 7. **Schedule** a routine
task with **cron**.

> **Concept before setup, easy before advanced:** you read the "why" first, then get a quick
> visual win with a GUI before dropping to the terminal, and finish by automating a task with
> cron. Everything is plain Linux — nothing needs to be installed or hosted beyond the VM itself.

> **Carry-over from Day 2:** the only thing you need is the **VirtualBox installer + Ubuntu 24.04
> Server ISO you downloaded on Day 2** (Topic 1.3) — the ISO is a disc-image installer file —
> saved on disk. No downloads? Grab them from
> `https://www.virtualbox.org/wiki/Downloads` and `https://ubuntu.com/download/server`. (You do
> **not** need Day 1's app or Day 2's Mailpit today.)

---

## Topic 1 — Concept: SSH, and SFTP vs FTP/FTPS

**Prerequisite:** none — this is the vocabulary for everything that follows. **Why first:** you
should know *why* we build an SSH/SFTP server before you spend time building one. No VM needed
yet; just read.

**Three ways to move files.**

| | FTP | FTPS | **SFTP** |
|---|---|---|---|
| Based on | old FTP protocol | FTP + TLS | **SSH** |
| Encryption | ❌ none (plaintext!) | ✅ TLS | ✅ SSH |
| Ports | 21 + random data ports | 21 + data ports | **just 22** |
| Firewall-friendly | ✗ (two channels) | ✗ | ✅ (one channel) |

- **FTP** sends passwords and files in the clear — never use it on real data.
- **FTPS** (FTP Secure) bolts **TLS** (Transport Layer Security) onto FTP but keeps its awkward
  two-channel design.
- **SFTP** is *file transfer over SSH*: one encrypted channel on **port 22**. Today's focus and
  the modern default.

**SSH in one line:** a secure, encrypted remote shell. `ssh user@host` logs you into another
machine's command line over port 22. **SFTP rides on that same SSH connection** — so once a
machine runs an SSH server, it can do SFTP too (you'll rely on this in Topic 4).

**The pattern we're building toward:** a **drop folder**. A partner connects over SFTP and drops
a file into an inbox on your server; a scheduled job on the server then picks it up and files it
away. This is one of the most common real-world integrations — no API required, just files.

**Checkpoint ✅** You can explain, in one sentence each: why SFTP beats FTP (encryption, single
port 22), and how SFTP relates to SSH (it runs over it). You'll connect to a real server for the
first time in Topic 5, once the VM exists.

---

## Topic 2 — Install VirtualBox & build the Ubuntu VM

**Prerequisite:** Day 2's two downloads on disk (VirtualBox installer + Ubuntu ISO).

**Goal:** from those downloads, install VirtualBox, create and install an Ubuntu 24.04 Server VM
(with OpenSSH), and put it on the network so Windows can reach it. This is the biggest setup step
of the course — take it slowly.

> **Shortcut:** if the trainer provided a ready-made `.ova` (Open Virtualization Appliance — a
> single-file VM export you import in one step), skip Steps 2–4: in VirtualBox use
> **File → Import Appliance**, select the `.ova`, **Import**, then jump to Step 5 (networking).

**Step 1 — install VirtualBox.** Run the installer you downloaded on Day 2 → **Next** through the
defaults → **Install** (approve any driver/network prompts) → **Finish**. VirtualBox opens.

**Step 2 — create the VM.**
1. In VirtualBox click **New**.
2. **Name:** `training-sftp`. **ISO Image:** browse to your `ubuntu-24.04.x-live-server-amd64.iso`.
   **Type:** Linux, **Version:** Ubuntu (64-bit) — VirtualBox fills these in from the ISO.
3. Tick **Skip Unattended Installation** (so you run the installer yourself and set a known
   password). Click **Next**.
4. **Base Memory:** 2048 MB (more if you can spare it). **Processors:** 2. **Next**.
5. **Disk:** create a virtual hard disk, **20 GB**, defaults. **Next → Finish**.

**Step 3 — install Ubuntu Server.**
1. Select the VM → **Start**. It boots the ISO into the text installer (use **arrow keys, Tab,
   Space, Enter** — no mouse needed). If asked, choose **Try or Install Ubuntu Server**.
2. Accept the defaults: language → **English**; keyboard → your layout; installer type →
   **Ubuntu Server** (not minimized); network → leave as-is (DHCP — Dynamic Host Configuration
   Protocol, which auto-assigns the VM an address); proxy → blank; mirror →
   default; storage → **Use an entire disk** → **Done** → **Continue** past the confirmation.
3. **Profile setup** — write these down, you'll use them all day:
   - Your name: `Trainer`
   - Server name: `training-sftp`
   - Username: `admin`
   - Password: something simple like `password` (lab only).
4. **"Install OpenSSH server"** — **press Space to tick this box.** Important: this makes the
   SFTP/SSH server available on first boot, so you don't install it separately.
5. Skip the "Featured server snaps" list (don't select any) → the install runs (a few minutes).

**Step 4 — first boot.**
1. When it says **"Reboot Now"**, select it. If it hangs on "Please remove the installation
   medium", just press **Enter** (VirtualBox already detaches the ISO).
2. At the login prompt, log in with `admin` / your password. You now have a Linux command line.

**Step 5 — put the VM on the network so Windows can reach it.** By default a new VM uses **NAT**
(Network Address Translation), which the host can't connect *into*. Switch it to **Bridged** so
it gets an address on your LAN (Local Area Network):

1. In the VM, shut down cleanly: `sudo poweroff`.
2. In VirtualBox: select the VM → **Settings → Network → Adapter 1 → Attached to: Bridged
   Adapter** → **Name:** pick your active Wi-Fi/Ethernet adapter → **OK**.
3. **Start** the VM again and log in.

**Step 6 — find the VM's IP (Internet Protocol) address** (inside the VM):

```bash
ip a
```

Look for `inet 192.168.x.x` under an adapter like `enp0s3` / `eth0` (ignore `127.0.0.1`).
**Write it down — we'll call it `<VM_IP>` all day.**

**Step 7 — prove Windows can reach it.** In a Windows terminal:

```bash
ping <VM_IP>
```

- **Replies = good.** Continue.
- **On a locked-down corporate network where Bridged gets no address:** shut the VM down, set
  Adapter 1 back to **NAT**, open **Advanced → Port Forwarding**, add a rule **Host Port 2222 →
  Guest Port 22**. Then everywhere below, connect to `127.0.0.1` on port **2222** instead of
  `<VM_IP>` on port 22.

**Checkpoint ✅ (Topic 2 complete)**
- The Ubuntu VM boots to a login and you can log in as `admin`.
- The VM is on the network; `ping <VM_IP>` replies from Windows.

**Common problems**
- *Installer complains about virtualization / "VT-x not available"* → enable **hardware
  virtualization (VT-x on Intel / AMD-V on AMD)** in your PC's **BIOS/UEFI** (the firmware setup
  screen — Basic Input/Output System / Unified Extensible Firmware Interface), and turn off
  Windows **Hyper-V** if it's on.
- *`ip a` shows only `127.0.0.1` / `10.0.2.15`* → still on NAT. Redo Step 5 (Bridged), or use the
  NAT port-forwarding fallback in Step 7.
- *Bridged shows no IPv4* → the adapter didn't get DHCP: `sudo dhclient enp0s3`, or reboot the VM.

---

## Topic 3 — Linux admin essentials

**Prerequisite:** the VM built and networked (Topic 2).

**Goal:** get comfortable with the everyday Linux commands you'll lean on for the rest of the day
— navigating, viewing and editing files, installing packages, managing users, and reading
permissions. None of this is SFTP-specific; it's the ground under Topics 4–7. Type everything
**inside the VM** (the VirtualBox `training-sftp` window). Every command here is safe to run and
repeat.

### 3a — Read the prompt, and `sudo`

The shell prompt tells you who and where you are:

```text
admin@training-sftp:~$
│     │            │ │
│     │            │ └─ $ = normal user   (# would mean root)
│     │            └─── ~ = your home directory (/home/admin)
│     └──────────────── the machine's hostname
└────────────────────── your username
```

> The hostname (`training-sftp`) confirms you're on the VM and not your Windows machine — glance
> at it before any command that matters.

Your `admin` user is a normal user. Admin tasks (installing software, editing system files,
managing other users) need **`sudo`** ("superuser do") in front — it asks for *your* password the
first time:

```bash
$ sudo whoami
root
```

> **Rule of thumb:** work as your normal user; add `sudo` only when a command needs admin rights.
> Don't log in as root directly.

### 3b — Move around the filesystem

Linux is one big tree starting at `/` (root):

```bash
$ pwd            # print working directory — where am I?
$ ls             # list files here
$ ls -l          # long listing: permissions, owner, size, date
$ ls -la         # also show hidden files (those starting with .)
$ cd /etc        # change directory to /etc
$ cd ~           # go to your home directory
$ cd ..          # go up one level
$ cd -           # go back to the previous directory
```

Directories worth knowing:

| Path | What lives there |
|------|------------------|
| `/etc` | System configuration files (e.g. `/etc/ssh/sshd_config`) |
| `/var/log` | Log files (`syslog`, `auth.log`) |
| `/home/admin` | Your personal files |
| `/home/sftpuser` | The SFTP user's home + `upload/` inbox (you create this in Topic 4) |
| `/usr/bin` | Installed programs |
| `/tmp` | Temporary files (wiped on reboot) |

📌 **Checkpoint:** `cd /var/log` then `ls` — you should see files like `syslog` and `auth.log`.

### 3c — View and edit files

```bash
$ cat /etc/hostname          # dump a whole (small) file
$ less /var/log/syslog       # scroll a big file: ↑ ↓ PgUp PgDn, q to quit
$ head -n 20 /var/log/syslog # first 20 lines
$ tail -n 20 /var/log/syslog # last 20 lines
$ tail -f /var/log/syslog    # live-follow a log (Ctrl+C to stop)
```

`tail -f` is your best friend when watching a service in real time — you'll use it to watch the
cron job fire in Topic 7.

`nano` is the beginner-friendly editor used throughout the day (you'll edit `sshd_config` and a
crontab with it later):

```bash
$ nano notes.txt        # a normal file
$ sudo nano /etc/hosts  # a system file needs sudo
```

Inside nano: type normally to edit · **Ctrl+O** then **Enter** = save (write Out) · **Ctrl+X** =
exit · **Ctrl+K** = cut a line, **Ctrl+U** = paste.

> ⚠️ Editing system files carelessly can break things. When a lab says edit a config file, change
> only the lines it names.

### 3d — Install software with APT

Ubuntu installs software from **repositories** with `apt`:

```bash
$ sudo apt update            # refresh the list of available packages
$ sudo apt upgrade           # install available updates
$ sudo apt install tree      # install a package (example: 'tree')
$ tree /etc/apt              # try the tool you just installed
$ sudo apt remove tree       # remove it
$ apt search htop            # search for a package
```

**Update the VM fully now:**

```bash
$ sudo apt update && sudo apt upgrade -y
```

`&&` means "run the second command only if the first succeeded"; `-y` auto-answers yes. (This is
the same pattern you'd use to install OpenSSH by hand if it were ever missing — see Topic 4, Step
1.)

📌 **Checkpoint:** a second `sudo apt upgrade` ends with `0 to upgrade` — the system is current.

### 3e — Users and groups

```bash
$ whoami                 # your username
$ id                     # your user id and group memberships
$ groups                 # groups you belong to (note: sudo)
```

Creating and removing a user (know the command — you'll use `adduser` for real in Topic 4):

```bash
$ sudo adduser bob            # interactive: creates user 'bob' + home dir
$ sudo usermod -aG sudo bob   # give bob admin rights (add to the sudo group)
$ sudo deluser bob            # remove a user
```

> **Why it matters here:** in Topic 4 you'll create a dedicated `sftpuser` with exactly this
> `adduser` command, and OpenSSH already runs as its own system user. Understanding users now
> makes the SFTP setup make sense.

### 3f — File permissions

Every file has an **owner**, a **group**, and permission bits. `ls -l` shows them (the two `admin`
words are the file's owner and group):

```text
-rw-r--r-- 1 admin admin 220 Sep 10 09:00 notes.txt
│ │  │  └── others:  r--   (read only)
│ │  └───── group:   r--   (read only)
│ └──────── owner:   rw-   (read + write)
└────────── type:    -     (- = file, d = directory)
```

Changing permissions and ownership:

```bash
$ chmod 640 notes.txt                    # owner rw, group r, others none
$ sudo chown admin:admin notes.txt       # set owner:group
```

The three digits are owner/group/others, where **4=read, 2=write, 1=execute** added together (so
`6 = 4+2 = rw`, `7 = rwx`). This is exactly the `chown`/`chmod 755` you'll run on the SFTP
`upload/` folder in Topic 4.

📌 **Checkpoint:** `chmod 600 notes.txt` then `ls -l notes.txt` shows `-rw-------`.

### 3g — Disk, memory, processes, and help

```bash
$ df -h               # disk space, human-readable
$ free -h             # memory usage
$ top                 # live process viewer (q to quit)
$ ps aux | grep ssh   # find running processes matching 'ssh'
$ man ls              # full manual for a command (q to quit)
$ ls --help           # quick usage summary
```

The `|` (pipe) feeds one command's output into another — here `ps aux` into `grep`. That
`ps aux | grep ssh` is a quick way to confirm the SSH server is running before you rely on it in
Topic 4.

### Command cheat-sheet

| Task | Command |
|------|---------|
| Where am I | `pwd` |
| List files | `ls -la` |
| Change dir | `cd /path` |
| View file | `less file` |
| Follow a log | `tail -f file` |
| Edit file | `nano file` |
| Update system | `sudo apt update && sudo apt upgrade` |
| Install software | `sudo apt install <name>` |
| Disk space | `df -h` |
| Memory | `free -h` |
| Processes | `top` |
| Who am I | `whoami` / `id` |

**Checkpoint ✅**
- You can read the prompt and tell you're on `training-sftp` as a normal user.
- You can move around with `cd`/`ls`/`pwd`, view a log with `less`/`tail`, and edit a file in `nano`.
- `sudo apt update && sudo apt upgrade -y` completes, and you can read owner/group/others in `ls -l`.

**Common problems**
- *"user is not in the sudoers file"* → log in as `admin` (the account you created during install),
  not a limited user.
- *`apt` errors about a lock (`Could not get lock /var/lib/dpkg/lock`)* → another update is running
  (or one crashed); wait a minute and retry, or reboot the VM.
- *Edited a system file and something broke* → reopen it with `sudo nano` and undo your change; only
  ever alter the lines a lab names.

---

## Topic 4 — Turn on the SFTP server & create a dedicated user

**Prerequisite:** the VM built and networked (Topic 2), and comfort with the shell (Topic 3).

**Goal:** confirm the VM's SSH/SFTP server is running, then add a dedicated transfer user with a
drop folder. Type these **inside the VM** (in the VirtualBox window — no remote connection needed
yet).

**Step 1 — verify OpenSSH is running.** You ticked "Install OpenSSH server" during the Ubuntu
install (Topic 2, Step 3), so it should already be active:

```bash
sudo systemctl status ssh      # expect "active (running)" — press q to exit
sudo ss -tlnp | grep :22       # expect a line with 0.0.0.0:22 or *:22
```

> **If it's missing** (you didn't tick it during install): install it now —
> `sudo apt update && sudo apt install -y openssh-server && sudo systemctl enable --now ssh`.

**Step 2 — configure the firewall (UFW) to allow SSH / SFTP / SCP.** Ubuntu's firewall is **UFW**
(Uncomplicated Firewall). A real server should run with it **on**, allowing only what's needed — so
we turn it on and explicitly allow SSH. **SFTP and SCP need no rules of their own: both travel over
the SSH connection on port 22, so one "allow SSH" rule opens all three.**

> ⚠️ **Order matters.** Allow SSH **before** enabling UFW — that is exactly what stops the firewall
> from locking you out of the VM.

```bash
sudo ufw status               # current state (often "inactive" on a fresh install)
sudo ufw allow OpenSSH        # open port 22/tcp — covers SSH, SFTP and SCP (same as: ufw allow 22/tcp)
sudo ufw enable               # type 'y' at the "may disrupt SSH" warning — your rule keeps 22 open
sudo ufw status               # confirm a line:  OpenSSH (or 22/tcp)  ALLOW  Anywhere
```

> **Using the NAT port-forward fallback (Topic 2)?** UFW still only needs port **22** allowed — the
> host→guest forward (host 2222 → guest 22) arrives on guest port 22, which `allow OpenSSH` already
> covers. No extra UFW rule is needed.

**Step 3 — create a dedicated SFTP user** (never use a personal/admin account for automated
transfers):

```bash
sudo adduser sftpuser
# set a password when prompted; accept defaults for the rest
```

**Step 4 — create a drop folder for incoming files and set ownership:**

```bash
sudo mkdir -p /home/sftpuser/upload
sudo chown -R sftpuser:sftpuser /home/sftpuser/upload
sudo chmod 755 /home/sftpuser/upload
```

**Checkpoint ✅**
- `systemctl status ssh` shows **active (running)** and port 22 is listening.
- `sudo ufw status` shows the firewall **active** with **OpenSSH / 22 ALLOW**.
- The `sftpuser` account exists and owns `/home/sftpuser/upload`.
- Pre-check reachability from Windows now: `Test-NetConnection <VM_IP> -Port 22` →
  `TcpTestSucceeded : True`. (Your first real transfer is the next topic.)

**Common problems**
- *`adduser: user already exists`* → fine, reuse it; reset the password with `sudo passwd sftpuser`.
- *Locked out right after `sudo ufw enable`* → you enabled **before** allowing SSH. Fix it from the
  **VirtualBox console window** (a local login the firewall can't block): `sudo ufw allow OpenSSH`.
- *From Windows the connection times out, but `sudo ufw status` shows SSH allowed* → it's **VM
  networking, not the firewall**: recheck Bridged vs NAT and `<VM_IP>` (Topic 2); with NAT, connect
  to `127.0.0.1` on port **2222**.
- *SSH shell works but SFTP/SCP fail* → **not** a firewall issue (they share port 22) — check the
  `upload` folder is owned by `sftpuser` (redo Step 4).

> **Optional hardening (skip if short on time):** to lock `sftpuser` to *only* SFTP and *only*
> its own folder, edit `/etc/ssh/sshd_config` (`sudo nano /etc/ssh/sshd_config`), add at the
> bottom:
> ```text
> Match User sftpuser
>     ChrootDirectory /home/sftpuser
>     ForceCommand internal-sftp
>     AllowTcpForwarding no
> ```
> The chroot dir must be root-owned (`sudo chown root:root /home/sftpuser`), with writes only
> inside `upload/`. Restart: `sudo systemctl restart ssh`. **Note:** `ChrootDirectory` (chroot =
> confining a user so `/home/sftpuser` becomes their whole visible filesystem) requires that
> folder to be **root-owned** — which then stops `sftpuser` writing anywhere in its home except
> `upload/`. Cron still runs (it doesn't go through SSH, so `ForceCommand` never affects it), but
> the Topic 7 cron job couldn't write to a `processed/` folder inside the chroot — so if you apply
> this hardening, keep any folders the cron job writes **outside** `/home/sftpuser`.

---

## Topic 5 — Transfer files with a GUI client (WinSCP & FileZilla, in depth)

**Prerequisite:** the SFTP server + `sftpuser` (Topic 4).

**Goal:** make your **first connection from Windows to the VM** and get genuinely comfortable with
the two GUI clients ops teams actually use. We go past drag-and-drop into the features you'll rely
on day to day. We use **password** auth (the simplest); key-based login is a next step beyond
today.

**Install both clients** (on Windows):
- **WinSCP:** `https://winscp.net/eng/download.php` → **Download WinSCP** → run installer →
  **Typical installation** → **Next → Install → Finish**.
- **FileZilla:** `https://filezilla-project.org/download.php?type=client` → **Download FileZilla
  Client** → run installer → **I Agree** → **decline** any bundled extra offers → **Install →
  Finish**.

**Prepare a test file.** On Windows, create `users.csv` (a CSV — Comma-Separated Values — file)
with a few rows — you'll upload it below:

```csv
name,email
Alice Tan,alice@test.com
Bob Lee,bob@test.com
Chandra Rao,chandra@test.com
```

---

### 4a — WinSCP in depth (recommended on Windows)

**Step 1 — save a reusable session.**
1. Open WinSCP → the **Login** dialog appears.
2. **New Site.** File protocol: **SFTP**. Host name: `<VM_IP>`. Port: **22**.
3. User name: `sftpuser`. Password: the one you set in Topic 4.
4. Click **Save** → name it `training-sftp` → tick **Save password** (lab only) → **OK**. The
   session is now stored in the left-hand list for one-click reconnects.
5. Select it → **Login** → on first connect accept the host key (**Yes**) to trust the VM.

**Step 2 — learn the interface.** WinSCP opens in **Commander** view: **local files on the left,
remote (VM) files on the right**, with a transfer queue along the bottom. (Prefer a single-pane,
Windows-Explorer feel? **Options → Preferences → Environment → Interface → Explorer**.)
- The address bars show the current local and remote folders — double-click into `upload/` on the
  right.
- The **Upload** / **Download** toolbar buttons, **F5**, or drag-and-drop all move files.

**Step 3 — upload and download.**
- Drag `users.csv` from the left pane into `/home/sftpuser/upload/` on the right (or select it and
  press **F5 → Copy**). It appears in the remote pane.
- Downloading is the same in reverse (remote → local, **F5**).

**Step 4 — the features worth knowing** (this is why WinSCP earns its place):
- **Edit a remote file in place — F4.** Select a remote file and press **F4**; WinSCP downloads
  it, opens it in an editor, and re-uploads on save. Ideal for quick config tweaks on the server.
- **Synchronize a whole folder — Commands → Synchronize.** Mirror a local folder to the remote (or
  the reverse); WinSCP transfers only what changed and previews the plan before doing anything.
- **Keep Remote Directory up to Date — Commands menu.** WinSCP watches a local folder and
  auto-uploads any file the moment you save it — a handy "poor man's deploy."
- **The transfer queue.** Large or many files transfer in the background; **pause, resume,
  reorder**, and set a **speed limit** (right-click the queue). Interrupted transfers **resume**
  instead of restarting.
- **File permissions.** Right-click a remote file → **Properties** to view/change its Unix
  permissions (the `chmod` bits) without a terminal.
- **Generate a script from a transfer.** In the upload/download dialog open **Transfer Settings →
  Generate Code** to turn the action into a ready-to-run WinSCP script or PowerShell snippet — a
  clean bridge from clicking to automating.
- **Bookmarks & session logging.** Bookmark deep remote folders, and enable a session log
  (**Preferences → Logging**) when you need to prove exactly what happened.

**Checkpoint ✅** `users.csv` is in `/home/sftpuser/upload` (visible in WinSCP's remote pane), you
can re-open the saved `training-sftp` session in one click, and you've edited a remote file with
**F4**.

**Common problems**
- *"Host key not verified"* → click **Yes/Accept** to trust the VM the first time.
- *`Connection refused` / timeout* → check `<VM_IP>` and that the VM + SSH are running (Topics 2 & 4).
- *Can't write to `upload/`* → the folder isn't owned by `sftpuser` (redo Topic 4, Step 4).

---

### 4b — FileZilla in depth (cross-platform alternative)

FileZilla does the same job with a different layout — worth knowing because many shops standardise
on it, and it runs on Windows, macOS and Linux.

**Step 1 — save a site.**
1. **File → Site Manager → New Site**, name it `training-sftp`.
2. Protocol: **SFTP - SSH File Transfer Protocol**. Host: `<VM_IP>`. Port: **22**.
3. Logon Type: **Ask for password** (or **Normal** to store it — lab only). User: `sftpuser`.
4. **Connect** → **OK** to trust the host key the first time.
   *(One-off instead? Use the **Quickconnect** bar at the top: Host `sftp://<VM_IP>`, Username,
   Password, Port `22` → **Quickconnect**.)*

**Step 2 — learn the interface.** FileZilla shows four areas:
- **Message log** (top) — the live SFTP conversation; your first stop when something fails.
- **Local site** (left) and **Remote site** (right) — navigate into `upload/` on the right.
- **Transfer queue** (bottom) — files grouped on **Queued / Failed / Successful** tabs.

**Step 3 — upload and download.** Drag `users.csv` from Local to Remote (or right-click →
**Upload**). Downloads go the other way. Transfers land in the queue and process automatically.

**Step 4 — the features worth knowing:**
- **Transfer queue control.** Right-click the queue to **pause/resume**; set **maximum
  simultaneous transfers** and **speed limits** (**Edit → Settings → Transfers**). Failed items
  collect on the **Failed transfers** tab so you can retry them in a batch.
- **Directory comparison.** **View → Directory Comparison** colour-codes files that differ in
  size/date between local and remote. Pair it with **View → Synchronized Browsing** so both panes
  navigate together — excellent for checking a folder tree matches.
- **Remote editing.** Right-click a remote file → **View/Edit**; FileZilla opens it locally and
  offers to re-upload on save (**Edit → Settings → File editing** sets the editor).
- **File permissions.** Right-click a remote file → **File permissions** for the `chmod` dialog
  (numeric field or checkboxes).
- **Filename filters.** **View → Filename filters** to hide/show files by pattern (e.g. only
  `*.csv`) in busy folders.
- **Export/import sites.** **File → Export** saves your Site Manager entries to share with a
  teammate or move to another machine.

**Checkpoint ✅** `users.csv` is in `/home/sftpuser/upload` via FileZilla too, you can read the
**message log**, and you've compared the local and remote folders with **Directory Comparison**.

**Common problems**
- *FileZilla defaults to plain FTP* → make sure the protocol says **SFTP**, not FTP.
- *"Connection timed out"* → wrong `<VM_IP>`, or the VM/SSH isn't running (Topics 2 & 4).
- *Edited file didn't upload* → confirm you accepted the **View/Edit** re-upload prompt.

**WinSCP or FileZilla?** For a Windows-only shop, **WinSCP** is usually smoother (F4 in-place edit,
Keep-up-to-date, code generation). Reach for **FileZilla** when you need the **same client on
Windows, macOS and Linux**. Both speak identical SFTP to the server — the choice is ergonomics.

---

## Topic 6 — Transfer files from the terminal

**Prerequisite:** a working SFTP connection (you just did it in the GUI, Topic 5).

**Goal:** do the same transfers from the command line — the mechanics a GUI hides, and the skill
that scales to servers with no desktop. You'll build a working repertoire of `sftp` commands. All
of this uses the OpenSSH `sftp` client that ships with Windows 10/11 (run it from PowerShell).

**Start an interactive session.** Create a test file on Windows first (e.g. `hello.txt`), then:

```powershell
sftp sftpuser@<VM_IP>
```

Enter the password once. You land at an `sftp>` prompt — a small shell where **plain commands act
on the remote VM** and **commands starting with `l` act on your local Windows machine**.

**The commands you'll actually use** (type them at the `sftp>` prompt):

| Command | What it does |
|---|---|
| `pwd` / `lpwd` | Print the **remote** / **local** working directory |
| `ls` / `lls` | List the **remote** / **local** directory |
| `cd upload` / `lcd Desktop` | Change the **remote** / **local** directory |
| `mkdir incoming` | Make a remote directory |
| `put hello.txt` | **Upload** a file (local → remote) |
| `put -r myfolder` | Upload a whole folder (recursive) |
| `put *.csv` | Upload every matching file (wildcards) |
| `get report.csv` | **Download** a file (remote → local) |
| `get report.csv copy.csv` | Download and rename in one step |
| `get -r logs` | Download a whole folder |
| `rename a.csv b.csv` | Rename a remote file |
| `rm old.csv` | Delete a remote file |
| `df -h` | Show remote free space |
| `!` | Drop to a **local** shell (type `exit` to come back) |
| `bye` / `exit` | Close the session |

**Worked examples.** Try these in order — each is a real task:

1. **Upload one file into the inbox:**
   ```text
   cd upload
   put hello.txt
   ls                       # confirm it's there
   ```
2. **Upload several files at once** (wildcards):
   ```text
   lcd C:\Users\<you>\Desktop
   put *.csv                 # every CSV on your Desktop → remote upload/
   ```
3. **Download a file and rename it locally:**
   ```text
   get hello.txt hello-copy.txt
   ```
4. **Upload an entire folder** (recursive):
   ```text
   put -r reports            # uploads the local 'reports' folder and everything in it
   ```
5. **Housekeeping — make a folder, rename and delete on the server.** (`rename` and `rm` act on
   the **remote** VM, so work on a throwaway copy and leave `hello.txt` in place:)
   ```text
   mkdir archive              # a new remote folder
   put hello.txt temp.txt     # upload a throwaway copy
   rename temp.txt old.txt    # rename it on the server
   rm old.txt                 # delete it again — hello.txt is untouched
   ```

**Two shortcuts beyond the interactive prompt:**

- **`scp` for a quick one-file copy** (no prompt session needed):
  ```powershell
  scp report.csv sftpuser@<VM_IP>:/home/sftpuser/upload/   # upload
  scp sftpuser@<VM_IP>:/home/sftpuser/upload/report.csv .  # download to the current folder
  scp -r reports sftpuser@<VM_IP>:/home/sftpuser/upload/   # a whole folder
  ```
- **Batch mode — run a list of commands in one go.** Put commands in a text file and feed it to
  `sftp -b`. Create `batch.txt`:
  ```text
  cd upload
  put users.csv
  ls
  bye
  ```
  then run:
  ```powershell
  sftp -b batch.txt sftpuser@<VM_IP>
  ```
  It runs the whole list and exits. *(It still prompts for the password once — running it truly
  unattended would need SSH keys, a next step beyond today.)*

**A plain remote shell.** To run commands *on* the VM rather than transfer files, use `ssh`
instead of `sftp`: `ssh sftpuser@<VM_IP>` gives you a Linux prompt on the server; `exit` to leave.

**Checkpoint ✅**
- `hello.txt` (and your `*.csv` files) appear in `/home/sftpuser/upload` on the VM.
- A downloaded file appears on Windows under its new name.
- `sftp -b batch.txt sftpuser@<VM_IP>` runs your command list end to end.

**Common problems**
- *`sftp: command not found`* → enable the Windows **OpenSSH Client**
  (**Settings → Apps → Optional features**), or use WinSCP's built-in terminal.
- *`Permission denied` on `put`* → you're not in a folder `sftpuser` owns; `cd upload` first.
- *Wildcards match nothing* → check your **local** folder with `lls` and `lpwd`.

---

## Topic 7 — Schedule a routine task with cron

**Prerequisite:** files can arrive in `/home/sftpuser/upload` over SFTP (Topics 5–6).

**Goal:** understand **cron** (Linux's built-in scheduler) and use it to run a routine task
automatically — no desktop, no manual trigger. We keep the task deliberately simple: tidy the SFTP
inbox on a schedule. Work **as `sftpuser`** in the VM window:

```bash
sudo su - sftpuser      # you are now sftpuser, in /home/sftpuser
mkdir -p ~/logs         # cron will write its logs here
```

### 6a — Cron basics

Each **crontab** ("cron table") line has **five time fields + a command**:

```text
* * * * *  command-to-run
│ │ │ │ │
│ │ │ │ └── day of week (0-6, Sun=0)
│ │ │ └──── month (1-12)
│ │ └────── day of month (1-31)
│ └──────── hour (0-23)
└────────── minute (0-59)
```

Examples: `* * * * *` = every minute · `*/5 * * * *` = every 5 minutes · `0 2 * * *` = 02:00 daily
· `30 6 * * 1` = 06:30 every Monday.

**See it work with a throwaway job first.** As `sftpuser`, run `crontab -e` (pick nano if asked),
add this line, then save (Ctrl+O, Enter) and exit (Ctrl+X):

```text
* * * * * date >> /home/sftpuser/logs/cron-test.log
```

Wait ~2 minutes, then `cat ~/logs/cron-test.log` — it gains a line every minute. Remove the line
afterwards (`crontab -e`, delete it, save).

### 6b — Schedule a real task: auto-archive the inbox

Let's have cron **tidy the SFTP inbox automatically**: every 5 minutes, move any uploaded CSV into
a `processed/` folder and note it in a log. It's a single command — no script file needed.

Prepare the folder once:

```bash
mkdir -p ~/processed
```

Then `crontab -e` and add one line:

```text
*/5 * * * * mv /home/sftpuser/upload/*.csv /home/sftpuser/processed/ 2>/dev/null && echo "archived at $(date)" >> /home/sftpuser/logs/archive.log
```

- `*/5 * * * *` — runs every 5 minutes.
- `mv .../upload/*.csv .../processed/` — moves any CSVs out of the inbox.
- `2>/dev/null` — stays silent when the inbox is empty (nothing to move is normal, not an error).
- `&& echo ... >> archive.log` — writes a log line **only** when files were actually moved.

Confirm the schedule:

```bash
crontab -l
```

**Prove it end to end:**
1. From Windows (WinSCP or `sftp`), upload a `users.csv` into `/home/sftpuser/upload/`.
2. Within 5 minutes, on the VM:
   ```bash
   ls ~/processed              # the moved file is here
   cat ~/logs/archive.log      # "archived at <date>"
   ls ~/upload                 # empty again — the inbox was tidied
   ```

**Checkpoint ✅**
- `crontab -l` shows your scheduled line.
- After an upload, the file moves from `upload/` to `processed/` on its own, and `archive.log`
  records it.

**Common problems**
- *Nothing happens* → use **full paths** in cron jobs (cron doesn't know your home shortcuts), and
  make sure `~/logs` exists.
- *`crontab -e` opens an unfamiliar editor* → run `select-editor` (or `export EDITOR=nano`) first.
- *Works when you type it, fails under cron* → cron runs with a **minimal environment** (a bare
  `PATH`, no login profile). Use full paths, and if a command isn't found add
  `PATH=/usr/local/bin:/usr/bin:/bin` at the top of the crontab.
- *Confirm cron even fired* → `grep CRON /var/log/syslog | tail` (may need `sudo`).

### 6c — How this maps to Laravel's scheduler (concept — no setup today)

You've automated with **raw Linux cron**. Laravel projects usually add a friendlier layer on top
called the **Task Scheduler** — worth recognising, since you'll meet it on any Laravel codebase
(including Days 1–2's app).

- Instead of many crontab lines, a Laravel app declares its whole schedule in **one place in
  code** (`routes/console.php`): e.g. `->everyFiveMinutes()`, `->dailyAt('02:00')`, `->weekdays()`.
- The server then needs just **one** real cron entry — `* * * * * php /path/artisan schedule:run`
  — and Laravel works out which tasks are actually due each minute.
- **Why teams prefer it:** schedules live in version control with the code, read like English
  (`->dailyAt('02:00')` vs `0 2 * * *`), and can guard against overlapping runs
  (`->withoutOverlapping()`).
- **The trade-off** (and why today is raw cron): the scheduler needs the Laravel app hosted on
  this machine. Today's module is deliberately Laravel-free, so plain cron is the right tool — but
  on Days 1–2's app, `schedule:run` is exactly how you'd automate a SOAP/REST sync on a timer.

**In one line:** raw cron schedules *a command*; the Laravel scheduler schedules *code*, driven by
a single `schedule:run` cron entry.

---

## End-of-Day 3 — final working state

You should now have:
- An Ubuntu VM running **OpenSSH SFTP** with a dedicated `sftpuser` and an `upload/` inbox.
- **GUI transfers** working in both WinSCP and FileZilla (saved sessions, remote editing, folder
  sync/compare).
- **Terminal transfers** working with the `sftp` command (`put`/`get`, wildcards, recursive, batch
  mode) and `scp`.
- A `sftpuser` **crontab** entry that auto-archives the inbox every 5 minutes into `~/processed/`,
  logging to `~/logs/archive.log`.

**Security / robustness recap**
- SFTP over SSH — encrypted, single port 22; no plaintext FTP.
- A **dedicated, least-privilege** SFTP user, not an admin account.
- The scheduled job **does nothing when the inbox is empty** (safe) and **moves** what it processes
  so files aren't handled twice.

**Course wrap-up — what you covered across three days**
1. **Day 1:** a secured REST API in Laravel (validation, JSON errors, Sanctum tokens).
2. **Day 2:** SOAP consume + expose in Laravel, with fault handling and email alerts on failure.
3. **Day 3:** a Linux **SFTP server** with GUI + terminal transfers, and a **cron-scheduled** task
   that tidies the inbox automatically.

Together they span the three integration styles you'll meet in the field: **REST**, **SOAP**, and
**file transfer / scheduled batch**.

**Stretch goals (if time remains)**
- **Key-based login:** generate an SSH key pair and install the public key so logins (and any
  future automation) need no password.
- **A real processing script:** replace the one-line cron job with a Bash script that validates
  each file, archives good ones, and quarantines bad ones with a log.
- **Process any file type**, not just `*.csv`, and add a size/format sanity check.
- **Rotate the logs** (`logrotate`) so they don't grow forever.
