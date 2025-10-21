# Pickle Rick (2025)

---

### 🧩 Category: Web Exploitation

### 🧠 Difficulty: Easy

### 🏷️ Tags: [web exploitation, command injection, privilege escalation]

### 💻 Platform: TryHackMe

### 📅 Year: 2025

### 📆 Date: 2025-10-21

---

## 📝 Description

This challenge involves exploiting a web application to find three hidden "ingredients". The path to success requires enumerating the web server to find login credentials, executing commands via a vulnerable web page, and then escalating privileges on the underlying Linux system to gain root access and find the final flags.

---

## 🔍 Initial Recon

| Item                  | Notes                                                                                                                                                       |
| :-------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Files provided**    | None. The challenge starts with just an IP address.                                                                                                         |
| **Services / Ports**  | nmap scan revealed two open ports: **22/tcp (SSH)** and **80/tcp (HTTP - Apache 2.4.41)**.                                                                  |
| **Hints given**       | The website source code contained a comment with the username. `robots.txt` contained a string that turned out to be the password.                          |
| **Observed behavior** | Directory busting with gobuster revealed a `/login.php` page. After logging in, the web application provides a command panel that executes system commands. |

---

## 🛠️ Approach

### **Step 1: Web Enumeration & Credential Discovery**

* Ran an `nmap` scan to confirm services (HTTP on port 80).
* Viewed the website source; a comment revealed the username: `R1ckRul3s`.
* Performed directory busting with `gobuster` which discovered `/robots.txt` containing the string `Wubbalubbadubdub` (likely the password) and `/login.php`.

### **Step 2: Gaining Initial Access**

* Logged into `/login.php` with:

  * **Username:** `R1ckRul3s`
  * **Password:** `Wubbalubbadubdub`
* Authentication provided access to a command panel which executed system commands.
* Ran `ls` via the web shell; found `Sup3rS3cretPickl3Ingred.txt`.
* Read the file to retrieve **Flag 1**.

### **Step 3: Privilege Escalation**

* `clue.txt` hinted at exploring the filesystem for more ingredients.
* Checked sudo privileges with `sudo -l` as the web shell user (`www-data`).
* `sudo -l` output showed: `(ALL) NOPASSWD: ALL` for `www-data`.

  * This allowed easy escalation: `sudo su` to become `root`.

### **Step 4: Finding the Final Ingredients**

* As `root`, explored filesystem freely.
* Found the second ingredient in Rick’s home directory:

  * `ls -a /home/rick` → file: `second ingredients`
  * Read it: **Flag 2**
* Found the third ingredient in root’s directory:

  * `ls /root` → file: `3rd.txt`
  * Read it: **Flag 3**

---

## 🔬 Command Log (summary)

| Command                                                                                                          | Result / Purpose                                                      |
| :--------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------- |
| `nmap 10.201.90.31`                                                                                              | Discovered open ports **22** and **80**.                              |
| `gobuster dir -u http://10.201.90.31 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt` | Found `/login.php` and `/robots.txt`.                                 |
| `ls` (via web shell)                                                                                             | Listed files, revealing `Sup3rS3cretPickl3Ingred.txt` and `clue.txt`. |
| `less Sup3rS3cretPickl3Ingred.txt` (via web shell)                                                               | Revealed **Flag 1: `mr. meeseek hair`**.                              |
| `sudo -l` (via web shell)                                                                                        | Showed `www-data` has `(ALL) NOPASSWD: ALL` sudo rights.              |
| `sudo su` (via web shell)                                                                                        | Escalated to **root**.                                                |
| `ls -a /home/rick` (as root)                                                                                     | Found the hidden `second ingredients` file.                           |
| `less /home/rick/'second ingredients'` (as root)                                                                 | Revealed **Flag 2: `1 jerry tear`**.                                  |
| `sudo ls /root` (as root)                                                                                        | Found `3rd.txt`.                                                      |
| `sudo less /root/3rd.txt` (as root)                                                                              | Revealed **Flag 3: `fleeb juice`**.                                   |

---

## 💡 Lessons Learned

* **Directory Busting Wordlists:** Choosing a comprehensive wordlist (e.g., `directory-list-2.3-medium.txt`) is often more effective than smaller lists. Specifying extensions with `-x` helps find files like `.php` and `.txt`.
* **Understanding Sudo:** When `sudo -l` shows `NOPASSWD: ALL`, privilege escalation is trivial. Prefer cautious use of `sudo su` vs running single commands with `sudo` in environments like web shells — single-command `sudo <cmd>` calls can be more reliable in restricted contexts.

---

## 🚩 Flags (The three ingredients)

* **Flag 1:** `mr. meeseek hair`
* **Flag 2:** `1 jerry tear`
* **Flag 3:** `fleeb juice`
