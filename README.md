# 🛡️ Arch Linux Security Cheat Sheet

**Community Edition – Defensive Security & System Hygiene**

Made by Arch Linux users who prefer verifying before trusting.

**Goal:** detect suspicious activity early, verify package integrity, and make safer decisions around AUR software.

> **Important:** None of these commands prove a system is compromised or clean. They are indicators that help you investigate.


## ⚠️ CHECK FIRST — Before Updating Anything

Run these before every update.

# List all AUR / foreign packages (your primary attack surface)
pacman -Qm

# Verify installed packages against pacman's database
sudo pacman -Qkk 2>&1 | grep warning

# Check for executables in temporary directories
find /tmp /dev/shm -type f -executable

# Show active network connections
sudo ss -tp

# Top CPU-consuming processes
ps aux --sort=-%cpu | head -20

# Recent successful logins
last | head -20

# Recent failed login attempts
sudo lastb | head -20

# Critical messages since last boot
sudo journalctl -p crit -b

**If anything looks suspicious, investigate before updating.**

## 🦠 Malware & Rootkit Scanning

### Update signatures first

sudo freshclam

### Scan critical system directories

sudo clamscan -r --infected --no-summary \
    --log=/var/log/clamscan.log \
    /usr/bin /usr/sbin /usr/lib /bin /sbin

### Scan a downloaded file before running it

sudo clamscan /path/to/file

### Scan an installed binary

sudo clamscan /usr/bin/program-name

### Scan an application's installation directory
sudo clamscan -r /usr/lib/application-name/

### Scan every file installed by a package
pacman -Ql package-name | awk '{print $2}' | grep -v '/$' | sudo xargs clamscan

> Treat findings as indicators, not proof of compromise.

## 🔍 AUR Audit — Before Building Anything

Clone without installing:
yay -G package-name

Review the build instructions:
less package-name/PKGBUILD

Inspect install hooks:
less package-name/*.install 2>/dev/null

Search for potentially risky patterns:
grep -E "(curl|wget|eval|base64|/tmp/|bash -c|python -c|nc )" \ package-name/PKGBUILD

List files installed by a package:
pacman -Ql package-name

### AUR Safety Tips

* Prefer packages with many users and active maintainers.
* Read recent comments on the AUR page.
* Understand what the PKGBUILD downloads and executes.
* Avoid installing abandoned packages unless you can audit them yourself.

## 📦 Package Integrity & Cleanup

Verify installed packages:
sudo pacman -Qkk 2>&1 | grep warning


List orphaned packages:
sudo pacman -Qtdq


Remove orphaned packages:
sudo pacman -Rns $(pacman -Qtdq)

List Flatpak applications:
flatpak list --app


## 🗂️ File System Checks

Executables in temporary locations:
find /tmp /dev/shm -type f -executable

Find SUID binaries:
find / -perm -4000 -type f 2>/dev/null

Find world-writable files:
find / -perm -0002 -type f 2>/dev/null \
    | grep -v "^/proc/"


### Recently modified system binaries
find /usr/bin /usr/sbin /bin /sbin \
    -mtime -1 -type f 2>/dev/null

> This only shows binaries modified within the last 24 hours. Updates can legitimately trigger results.

---

## 🔎 Investigating Suspicious Files

Scan it:
sudo clamscan /path/to/file


Inspect printable strings:
strings /path/to/file | head -30


View metadata:
stat /path/to/file


Check package ownership:
pacman -Qo /path/to/file


Verify package integrity:
sudo pacman -Qkk package-name

## 🌐 Network Monitoring

Listening services:
sudo ss -tulnp


Active connections:
sudo ss -tp


View firewall rules:
sudo nft list ruleset


> If no firewall rules exist, consider configuring nftables.

## 📋 Logs

Critical messages from this boot:
sudo journalctl -p crit -b

Errors from the last 24 hours:
sudo journalctl --since "24 hours ago" -p err

Follow logs live:
sudo journalctl -f

Authentication events:
sudo journalctl -u sshd


## 🛡️ Rootkit Detection

RKHunter:

sudo rkhunter --update
sudo rkhunter --check --skip-keypress


Chkrootkit:
sudo chkrootkit

> Rootkit tools often generate false positives. Review findings carefully.

## 🔄 Updating Safely

Update official repositories:
sudo pacman -Syu


Update Flatpaks:
sudo flatpak update


Update AUR packages:
yay -Syu


**Only update after completing your verification checks.**

## 💡 Best Practices

* **Check first, update second.**
* **Keep your AUR footprint small.**
* **Review PKGBUILDs before installing unfamiliar software.**
* **Prefer official repositories whenever possible.**
* **Use Flatpak selectively for applications that benefit from sandboxing.**
* **Remove software you no longer use.**
* **Follow Arch security advisories and project news.**
* **If something feels wrong, investigate before proceeding.**

---

## Remember

Security is not a single command.

It's a habit of:

* verifying,
* minimizing trust,
* keeping software updated,
* and paying attention to unusual behavior.

Stay curious. Stay cautious. Stay safe.
