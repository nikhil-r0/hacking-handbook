**c4ptur3th3fl4g Walkthrough (2025)**

---

### 🧩 **Category:** General Skills

### 🧠 **Difficulty:** Easy

### 🏷️ **Tags:** [encoding, steganography, forensics, spectrogram]

### 💻 **Platform:** TryHackMe

### 📅 **Year:** 2025

### 📆 **Date:** 2025-10-20

---

## 📝 Description

This challenge is a collection of introductory CTF tasks designed to test a variety of fundamental skills. The challenges involve decoding various text encodings, analyzing audio files with a spectrogram, and extracting hidden data from images using steganography and file carving techniques.

---

## 🔍 Initial Recon

| Item                  | Notes                                                                                                                                                        |
| :-------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Files provided**    | A series of encoded strings, an audio file (`secretaudio_1559007588454.wav`), and two image files (`stegosteg_1559008553457.jpg`, `meme_1559010886025.jpg`). |
| **Services / Ports**  | None. All challenges are offline.                                                                                                                            |
| **Hints given**       | The task titles ("Translation and Shifting", "Spectograms", "Steganography") give direct clues about the required techniques.                                |
| **Observed behavior** | Each task presents a piece of data that must be analyzed or decoded using the correct tool or technique to find the flag.                                    |

---

## 🛠️ Approach

### **Task 1: Translation and Shifting**

This task consisted of 10 strings, each requiring a different decoding method. Tools like **CyberChef** and **decode.fr** were effective for solving these.

| Encoding                 | Method                                                                                                   | Flag                                                   |
| :----------------------- | :------------------------------------------------------------------------------------------------------- | :----------------------------------------------------- |
| **Leet Speak**           | Manually translated from leetspeak to plain English.                                                     | `can you capture the flag?`                            |
| **Binary**               | Converted from binary to ASCII using CyberChef.                                                          | `lets try some binary out!`                            |
| **Base32**               | Decoded using a Base32 decoder in CyberChef.                                                             | `base32 is super common in CTF's`                      |
| **Base64**               | Decoded using a Base64 decoder in CyberChef.                                                             | `Each Base64 digit represents exactly 6 bits of data.` |
| **Hexadecimal**          | Converted from hex values to ASCII using CyberChef.                                                      | `hexadecimal or base16?`                               |
| **ROT13 Cipher**         | Decoded using decode.fr.                                                                                 | `Rotate me 13 places!`                                 |
| **ROT47 Cipher**         | Decoded using decode.fr.                                                                                 | `You spin me right round baby right round (47 times)`  |
| **Morse Code**           | Translated from Morse code using decode.fr.                                                              | `TELECOMMUNICATION ENCODING`                           |
| **Decimal (ASCII)**      | Converted decimal values to ASCII using decode.fr.                                                       | `Unpack this BCD`                                      |
| **Multi-level Encoding** | Chained decoding steps in CyberChef: From Base64 → From Morse Code → From Binary → ROT47 → From Decimal. | `Let's make this a bit trickier...`                    |

---

### **Task 2: Spectrogram Analysis**

The audio file `secretaudio_1559007588454.wav` contained hidden text.

**Method:**
Opened in **Audacity** → switched to **Spectrogram View** → visible text message in frequency bands.

**Flag:** `Super Secret Message`

---

### **Task 3: Steganography**

Hidden message in a JPEG image.

**Method:**
Initial checks with `exiftool` and `strings` revealed nothing. Used `steghide` for data extraction.

```bash
steghide extract -sf stegosteg_1559008553457.jpg
```

**Output:** Created file `steganopayload2248.txt` containing the flag.
**Flag:** `SpaghettiSteg`

---

### **Task 4: Security Through Obscurity**

Analyzing an image for embedded files.

**Method:**
Used `binwalk` to identify and extract hidden data.

```bash
binwalk -e meme_1559010886025.jpg
```

Extracted files included `hackerchat.png` and a RAR archive `122A7.rar`.

**Further Analysis:**

```bash
strings 122A7.rar
```

Revealed a second hidden flag.

**Flags:**

* `hackerchat.png`
* `AHH_YOU_FOUND_ME!`

---

## 🚩 **Flag Summary**

| Task       | Flags                                                                                                                                                                                                                                                                                                                                            |
| :--------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Task 1** | can you capture the flag?<br>lets try some binary out!<br>base32 is super common in CTF's<br>Each Base64 digit represents exactly 6 bits of data.<br>hexadecimal or base16?<br>Rotate me 13 places!<br>You spin me right round baby right round (47 times)<br>TELECOMMUNICATION ENCODING<br>Unpack this BCD<br>Let's make this a bit trickier... |
| **Task 2** | Super Secret Message                                                                                                                                                                                                                                                                                                                             |
| **Task 3** | SpaghettiSteg                                                                                                                                                                                                                                                                                                                                    |
| **Task 4** | hackerchat.png<br>AHH_YOU_FOUND_ME!                                                                                                                                                                                                                                                                                                              |

---
